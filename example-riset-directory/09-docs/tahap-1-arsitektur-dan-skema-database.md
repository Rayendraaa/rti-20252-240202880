# Tahap 1 — Perancangan Arsitektur & Skema Database

**Status:** Selesai

---

## 1. Komponen Sistem

1. **API Gateway (Go, Echo)** — menerima request, mem-parsing header JWT untuk mengambil `kid`, lalu meresolusi JWK terkait sebelum verifikasi signature.
2. **Redis (L1 Cache, murni cache JWKS)**
   - *Positive cache*: `jwks:kid:<kid>` → JWK (TTL pendek, mis. 5 menit) untuk kunci valid.
   - *Negative cache*: `jwks:negative:<kid>` → marker (TTL pendek, mis. 60 detik) untuk `kid` yang tidak ditemukan — inti mitigasi flooding.
   - Tidak menyimpan state rate-limit (lihat poin 3).
3. **PostgreSQL (L2 / Source of Truth + Rate Limit Counter Permanen)** — menyimpan metadata kunci signing (`signing_keys`) dan counter rate-limit permanen (`rate_limit_counters`).

## 2. Alur Resolusi Kunci (Mitigasi)

```
Request masuk → Gateway parsing header JWT → ambil `kid`
  │
  ├─ Cek Redis positive cache (jwks:kid:<kid>)
  │     ├─ HIT  → verifikasi signature → lanjut
  │     └─ MISS ↓
  │
  ├─ Cek Redis negative cache (jwks:negative:<kid>)
  │     ├─ HIT  → tolak langsung (401), tanpa query DB
  │     └─ MISS ↓
  │
  ├─ UPSERT & cek rate_limit_counters di PostgreSQL (atomic, per client_ip + window)
  │     ├─ EXCEEDED → tolak (429) + set Redis negative cache
  │     └─ OK ↓
  │
  └─ Query PostgreSQL (signing_keys WHERE kid = ? AND is_active)
        ├─ FOUND     → isi Redis positive cache → verifikasi signature
        └─ NOT FOUND → set Redis negative cache → tolak (401)
```

Catatan: pada mode `CACHE_MODE=none` (baseline), langkah cek Redis dan rate-limit dilewati — setiap request langsung query `signing_keys` di PostgreSQL, mensimulasikan gateway tanpa mitigasi.

Mekanisme **fail-closed**: jika Redis tidak dapat diakses, gateway tetap melanjutkan ke PostgreSQL (rate-limit counter tetap berfungsi karena bersumber dari PostgreSQL); jika PostgreSQL tidak dapat diakses, request ditolak (bukan diloloskan tanpa verifikasi).

## 3. Skema Database (PostgreSQL)

```sql
CREATE TABLE signing_keys (
    kid             VARCHAR(255) PRIMARY KEY,
    kty             VARCHAR(10)  NOT NULL DEFAULT 'RSA',
    alg             VARCHAR(10)  NOT NULL DEFAULT 'RS256',
    use_type        VARCHAR(10)  NOT NULL DEFAULT 'sig',
    n               TEXT         NOT NULL,   -- modulus, base64url
    e               TEXT         NOT NULL,   -- exponent, base64url
    is_active       BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT now(),
    expires_at      TIMESTAMPTZ,
    revoked_at      TIMESTAMPTZ
);

CREATE INDEX idx_signing_keys_active ON signing_keys (kid) WHERE is_active = TRUE;

-- Counter rate-limit permanen (source of truth di PostgreSQL)
CREATE TABLE rate_limit_counters (
    client_ip       INET        NOT NULL,
    window_start    TIMESTAMPTZ NOT NULL,
    request_count   INTEGER     NOT NULL DEFAULT 0,
    blocked_count   INTEGER     NOT NULL DEFAULT 0,
    PRIMARY KEY (client_ip, window_start)
);
```

Upsert atomik untuk increment counter per request (window tetap, mis. 1 detik):

```sql
INSERT INTO rate_limit_counters (client_ip, window_start, request_count)
VALUES ($1, $2, 1)
ON CONFLICT (client_ip, window_start)
DO UPDATE SET request_count = rate_limit_counters.request_count + 1
RETURNING request_count;
```

Jika `request_count` melebihi ambang batas, request ditolak dan `blocked_count` di-increment pada baris yang sama. Data ini bersifat permanen (tidak di-TTL) sehingga dapat dipakai langsung untuk analisis pola serangan pada Tahap 4.

Tabel log lookup tambahan (untuk cache hit/miss ratio) akan ditentukan pada Tahap 2 setelah skenario k6 lebih jelas.

## 4. Skema Redis (Murni L1 Cache JWKS)

| Key Pattern | Tipe | TTL | Tujuan |
|---|---|---|---|
| `jwks:kid:<kid>` | STRING (JSON JWK) | ~300s | Cache positif untuk kunci valid |
| `jwks:negative:<kid>` | STRING (`"1"`) | ~60s | Cache negatif untuk `kid` tak dikenal |

## 5. Keputusan Teknis (Final)

1. **Mode eksperimen**: satu binary gateway dengan toggle `CACHE_MODE=none|hybrid` — `none` = baseline tanpa cache/rate-limit, `hybrid` = arsitektur mitigasi penuh. Memastikan perbandingan baseline vs mitigated apple-to-apple untuk perhitungan $D_{perf}$.
2. **Framework Gateway**: **Echo** (Go web framework).
3. **Rate limiting**: counter permanen di **PostgreSQL** (`rate_limit_counters`, atomic UPSERT per `client_ip` + window). **Redis murni sebagai L1 cache JWKS** (positive & negative cache), tidak menyimpan state rate-limit.
4. **Identity Service**: **PostgreSQL `signing_keys` langsung sebagai backing store** — tidak ada microservice tambahan; fokus eksperimen pada lapisan caching/rate-limit di Gateway.
5. **Redis client**: `go-redis/redis/v9` (default standar Go ekosistem).
6. **PostgreSQL driver**: `pgx` (native driver, performa baik, mendukung connection pooling via `pgxpool`).
7. **Skenario issuer**: single issuer (disederhanakan) — dapat diperluas ke multi-issuer di penelitian lanjutan jika diperlukan.

# Tahap 3 — Skrip Pengujian k6 (Legitimate vs Attack Traffic)

**Status:** Selesai — matrix 400 run (40 replikasi) sudah dijalankan, data tersedia di `04-data/` (matrix awal 50 run/5 replikasi diarsipkan di `04-data/_archive-50run-20260612/`)
**Bergantung pada:** [tahap-2-implementasi-gateway.md](tahap-2-implementasi-gateway.md)
**Lokasi kode:** [../05-kode/k6](../05-kode/k6)

---

## Tujuan

Menyusun skenario k6 untuk membandingkan gateway pada mode `CACHE_MODE=none` (baseline) vs `CACHE_MODE=hybrid` (mitigasi), dengan tiga jenis traffic:

- **Legitimate traffic** — request dengan JWT valid (`kid` dikenal), mensimulasikan beban normal.
- **Attack traffic** — request dengan JWT ber-`kid` acak/tidak terdaftar, mensimulasikan JWKS Endpoint Flooding (CVE-2026-48524).
- **Mixed traffic** — legitimate + attack berjalan bersamaan, untuk mengukur dampak mitigasi terhadap pengalaman user legit saat diserang.

## Deliverable

- [x] Skrip k6 `legitimate.js` (steady load dengan `kid` valid)
- [x] Skrip k6 `attack.js` (flooding dengan `kid` acak/pool, `KID_STRATEGY=unique|pool`)
- [x] Skrip k6 `mixed.js` (kombinasi legitimate + attack secara bersamaan, dengan Trend custom per scenario)
- [x] Konfigurasi skenario (VUs, durasi, ramping) untuk tiap kombinasi mode × traffic
- [x] Output metrics k6 + snapshot `/metrics` gateway dalam format JSON/CSV untuk Tahap 4
- [x] Smoke test (kalibrasi sebelum matrix penuh)
- [x] Matrix penuh 400 run (2 cache_mode x 5 traffic_variant x 40 replikasi)

## Desain yang Diimplementasikan

### Struktur kode (`05-kode/k6/`)

```
05-kode/k6/
├── lib/
│   ├── config.js              # BASE_URL, durasi, VU, KID_STRATEGY (env-driven)
│   ├── tokens.js               # SharedArray token legit + pool kid attack
│   ├── legit-tokens.json       # JWT valid (hasil gen-legit-tokens.sh)
│   └── gen-legit-tokens.sh      # regenerasi legit-tokens.json dari seed Tahap 2
├── legitimate.js                # constant-vus, JWT valid
├── attack.js                    # ramping-vus 0->200, JWT kid acak/invalid
├── mixed.js                     # legitimate + attack berjalan bersamaan
├── monitor-resources.sh         # docker stats polling -> resources.csv
├── run-scenario.sh               # runner 1 kombinasi -> 04-data/<run-id>/
└── README.md
```

### Skrip & skenario

| Skrip | Executor | Default durasi | Env relevan |
|---|---|---|---|
| `legitimate.js` | `constant-vus` | 5 VU x 60s | `LEGIT_VUS`, `LEGIT_DURATION` |
| `attack.js` | `ramping-vus` 0→200 | ramp 10s + hold 50s | `ATTACK_RAMP_DURATION`, `ATTACK_HOLD_DURATION`, `ATTACK_MAX_VUS`, `KID_STRATEGY` |
| `mixed.js` | `legitimate` + `attack` sebagai dua k6 scenario bersamaan, masing-masing ditag `scenario` | sama seperti di atas | semua env di atas |

`KID_STRATEGY`:
- `unique` — kid acak baru tiap request → menguji jalur **rate-limit**.
- `pool` — kid dari pool ~50 nilai (dibuat sekali via `SharedArray`, dipakai berulang) → menguji **negative cache** + rate-limit, lebih representatif pola CVE.

Token legitimate: 1 JWT valid (`kid: seed-key-01`, exp +24h) di-generate sekali dari seed Tahap 2 (`gen-legit-tokens.sh`), dipakai berulang via `SharedArray` — tidak ada signing dinamis di k6.

### Matrix eksperimen

| Dimensi | Nilai |
|---|---|
| `CACHE_MODE` | `none`, `hybrid` |
| Traffic variant | `legitimate`, `attack-unique`, `attack-pool`, `mixed-unique`, `mixed-pool` |
| Replikasi | 40 |

Total: **2 × 5 × 40 = 400 run**, dijalankan via loop `run-matrix.sh` (membungkus `run-scenario.sh`, lihat [README](../05-kode/k6/README.md)).

### Runner (`run-scenario.sh`)

Untuk setiap kombinasi `<cache_mode> <traffic_variant> <replication>`:

1. `CACHE_MODE=<mode> docker compose up -d --force-recreate gateway` (di `05-kode/gateway/`).
2. Poll `GET /healthz` sampai sehat (timeout 30s).
3. Start `monitor-resources.sh` di background → `resources.csv`.
4. Snapshot `GET /metrics` gateway → `gateway-metrics-before.txt`.
5. Jalankan skrip k6 via `docker run --rm --network gateway_default ... grafana/k6 run --summary-export ...`.
6. Snapshot `GET /metrics` gateway → `gateway-metrics-after.txt`.
7. Stop resource monitor, tulis `meta.json` (cache_mode, traffic_variant, kid_strategy, replication, waktu mulai/selesai, parameter rate-limit & TTL cache).

`<run-id>` = `<cache_mode>__<traffic_variant>__rep<N>__<timestamp>`.

### Output per run (`04-data/<run-id>/`)

```
04-data/<cache_mode>__<traffic_variant>__rep<N>__<timestamp>/
├── k6-summary.json            # ringkasan agregat k6 (--summary-export)
├── gateway-metrics-before.txt # snapshot /metrics gateway sebelum run
├── gateway-metrics-after.txt  # snapshot /metrics gateway sesudah run
├── resources.csv               # timestamp,container,cpu_pct,mem_usage,mem_pct (~3s interval)
└── meta.json                    # cache_mode, traffic_variant, kid_strategy, replication, waktu mulai/selesai
```

`k6-summary.json` mencakup `metrics.http_req_duration` (semua scenario) serta,
untuk `mixed.js`, `metrics.legitimate_req_duration` dan
`metrics.attack_req_duration` (Trend custom per scenario, di-tag via
`res.timings.duration`) — dipakai Tahap 4 untuk menghitung D_perf traffic
legitimate saat mixed (hybrid vs none).

`gateway-metrics-*.txt` adalah scrape Prometheus (`jwksgw_*`) — delta
before/after memberi angka eksak `jwksgw_db_queries_total`,
`jwksgw_cache_requests_total`, `jwksgw_rate_limit_blocked_total`,
`jwksgw_auth_requests_total` per run, untuk metrik "efektivitas mitigasi" di
Tahap 4.

`resources.csv` interval nominal 1s, tapi `docker stats --no-stream` untuk 3
container butuh ~2-3s di Windows Docker Desktop sehingga interval aktual ~3s
— cukup untuk tren CPU/memori pada window 60s.

State Postgres/Redis **tidak** direset antar run — `window_start` per-detik
pada rate limiter membuat data antar run tetap terisolasi.

## Hasil Smoke Test

Smoke test (`./run-scenario.sh hybrid legitimate smoke -e LEGIT_DURATION=15s -e LEGIT_VUS=2`)
dijalankan untuk kalibrasi sebelum commit ke matrix 50-run.

**Iterasi pertama** memakai `--out json=...` (raw per-request metrics):
menghasilkan `k6-output.json` **139MB / 571.414 baris** hanya dari 15 detik,
2 VU, ~2.900 req/s. Diekstrapolasi ke matrix penuh (60s, attack.js ramping ke
200 VU, 50 run) → volume data tidak terkelola (puluhan GB, risiko disk penuh).

**Perbaikan**: ganti `--out json=...` → `--summary-export=...` (statistik
agregat per metrik), tambah snapshot `/metrics` gateway before/after, dan
tambah `Trend` custom per scenario di `mixed.js`.

**Iterasi kedua** (setelah perbaikan), hasil untuk 15s/2VU/~2.900 req/s
(43.531 requests, 100% checks lolos, `http_req_duration` avg ≈ 463µs):

| File | Ukuran |
|---|---|
| `k6-summary.json` | ~3.3 KB |
| `gateway-metrics-before.txt` | ~165 B |
| `gateway-metrics-after.txt` | ~2.3 KB |
| `resources.csv` (15s @ ~3s interval) | ~1.2 KB |

Total per run < 10 KB — aman untuk matrix 50-run.

## Hasil Matrix Penuh (awal, 50 run — diarsipkan)

Matrix awal 50 run (5 replikasi) dijalankan via loop `run-scenario.sh` (lihat
di atas), total durasi run 2026-06-12T18:05Z – 2026-06-12T18:59Z (~54 menit
untuk 50 run, lebih cepat dari estimasi karena overhead restart gateway/health
check kecil pada mesin lokal). Semua 50 run selesai dengan `k6_exit_code: 0`.

Output: `04-data/<cache_mode>__<traffic_variant>__rep<N>__<timestamp>/`,
total ukuran seluruh matrix **~1.7 MB** (vs. 139 MB untuk 1 smoke test 15
detik sebelum perbaikan output strategy) — jauh lebih terkelola.

| cache_mode | traffic_variant | replikasi |
|---|---|---|
| none, hybrid | legitimate, attack-unique, attack-pool, mixed-unique, mixed-pool | 1-5 |

Dataset ini kemudian dipindahkan ke `04-data/_archive-50run-20260612/` setelah
matrix 400-run (lihat bawah) dijalankan sebagai pengganti.

## Hasil Matrix Penuh (400 run / 40 replikasi)

Untuk memperbesar ukuran sampel statistik, matrix diperluas dari 5 menjadi 40
replikasi per kombinasi (total 2 × 5 × 40 = 400 run). Loop baru
`run-matrix.sh` (lihat [README](../05-kode/k6/README.md)) menjalankan
replikasi 1..40 secara *interleaved* (loop replikasi di luar, loop
mode/variant di dalam), sehingga jika proses berhenti di tengah jalan, setiap
kombinasi tetap memiliki jumlah replikasi yang sama.

Sebelum menjalankan matrix, token JWT legitimate (`lib/legit-tokens.json`)
yang sebelumnya sudah *expired* (dibuat 2026-06-12, `exp +24h`) diregenerasi
ulang via skrip seed Tahap 2 dan `gen-legit-tokens.sh`, serta cache Redis
di-*flush* agar matrix dimulai dari kondisi cache dingin (konsisten dengan
metodologi awal).

Matrix 400 run dijalankan 2026-06-15, seluruhnya selesai dengan
`k6_exit_code: 0` (0 `FAILED` pada `04-data/matrix-40run.log`), menghasilkan
struktur `04-data/<cache_mode>__<traffic_variant>__rep<N>__<timestamp>/` yang
sama seperti di atas, dengan replikasi 1-40 untuk tiap 10 kombinasi
`(cache_mode, traffic_variant)`.

Data 400-run ini menjadi input Tahap 4 (analisis & visualisasi), menggantikan
dataset 50-run sebelumnya.

## Catatan Lingkungan

- **MSYS_NO_PATHCONV=1** diperlukan pada perintah `docker run` di Git Bash
  (Windows) agar path container (`/scripts/...`, `/data/...`) tidak diubah
  Git Bash/MSYS menjadi path Windows sebelum diteruskan ke `docker`.
- Direktori `04-data/<run-id>/` kadang tidak bisa langsung dihapus
  (`Device or resource busy`) tepat setelah `docker run --rm` dengan bind
  mount selesai — ini transient lock Docker Desktop/WSL2 pada Windows, hilang
  sendiri setelah beberapa saat.

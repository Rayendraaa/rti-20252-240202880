# Tahap 2 — Implementasi API Gateway (Go)

**Status:** Selesai
**Acuan arsitektur:** [tahap-1-arsitektur-dan-skema-database.md](tahap-1-arsitektur-dan-skema-database.md)
**Lokasi kode:** [../05-kode/gateway/](../05-kode/gateway/)

---

## Tujuan

Mengimplementasikan API Gateway (Go + Echo) yang mendukung dua mode operasi melalui `CACHE_MODE`:

- `none` — baseline, setiap request langsung query `signing_keys` di PostgreSQL.
- `hybrid` — mitigasi penuh: Redis L1 cache (positive/negative) + rate-limit counter permanen di PostgreSQL.

## Deliverable

- [x] Struktur project Go (`cmd/gateway`, `internal/...`) — DDD-lite per bounded-context (`jwks`, `ratelimit`, `jwtauth`, `httpapi`, `platform`, `metrics`)
- [x] `docker-compose.yml` (gateway, postgres, redis) dengan healthcheck & `depends_on: condition: service_healthy`
- [x] Migration SQL via Sqitch (`signing_keys`, `rate_limit_counters`, `upsert_rate_limit_counter` function)
- [x] Skrip seed (`scripts/seed`): generate RSA-2048 keypair, insert ke `signing_keys`, cetak contoh JWT valid (exp +24h)
- [x] Middleware verifikasi JWT (RS256) + resolusi `kid` (mode `none` dan `hybrid`, fail-closed pada Postgres down, fail-open pada Redis down)
- [x] Endpoint `/metrics` (Prometheus, prefix `jwksgw_`): cache hit/miss, db query count, rate-limit blocked count, auth outcome, request duration
- [x] Konfigurasi via environment variable (`.env.example`)
- [x] `/healthz` (dipakai healthcheck compose & runner Tahap 3)
- [x] `README.md` dengan command mentah (sqitch deploy, seed, run, docker compose, switch `CACHE_MODE`)

## Hasil Verifikasi End-to-End

Diverifikasi manual via `docker compose` + curl (lihat [../05-kode/gateway/README.md](../05-kode/gateway/README.md) bagian "Verifikasi end-to-end"):

- **Hybrid**: valid kid → 200 (cache miss → DB → fill cache) → 200 (cache hit); unknown kid → 401 `invalid_kid` (negative cache) tanpa query DB berulang; flood concurrent dengan `kid` unik → sebagian `429 rate_limited` setelah >20 req/s per `client_ip`.
- **None**: valid kid selalu 200 dengan `jwksgw_db_queries_total{resolve_key}` naik 1:1 per request; tidak pernah `429`.
- **Fail-closed**: Postgres down → `503 service_unavailable` (kedua mode). Redis down (hybrid) → kid yang sudah ter-cache tetap `200` (fallback Postgres), `/healthz` melaporkan `redis:false`.

## Catatan Lingkungan

- PostgreSQL container di-expose ke host pada port **5433** (bukan 5432) untuk menghindari konflik dengan instance PostgreSQL lokal di mesin development. Di dalam jaringan Docker, gateway tetap mengakses `postgres:5432`.
- Sqitch project (`migrations/`) adalah dokumentasi migrasi resmi (deploy/revert/verify), namun di mesin development saat ini `sqitch` CLI tidak punya driver `DBD::Pg` — migrasi diverifikasi dengan menjalankan file `deploy/*.sql` langsung via `psql`. Pastikan environment dengan `DBD::Pg` terpasang untuk `sqitch deploy` penuh.

# Laporan Penelitian

**Judul:** Performance and Security Evaluation of Mitigating JWKS Endpoint Flooding on Microservices Gateway Using Redis-PostgreSQL Hybrid Caching

**Peneliti:** Helmi Bahar Alim
**Target Publikasi:** Sinta 2 (Jurnal RESTI/Telematika) atau Scopus Q3–Q4
**Status Penelitian:** Tahap 1–4 selesai; Tahap 5 (draf naskah jurnal) sedang berjalan ([../07-manuskrip/](../07-manuskrip/))

---

## 1. Ringkasan Eksekutif

Penelitian ini merancang, mengimplementasikan, dan mengevaluasi secara empiris mekanisme **Redis-PostgreSQL Hybrid Caching** sebagai mitigasi kerentanan **JWKS Endpoint Flooding** pada API Gateway berbasis Go (Echo). Evaluasi dilakukan melalui eksperimen terkontrol: satu gateway dengan dua mode operasi (`CACHE_MODE=none` sebagai baseline dan `CACHE_MODE=hybrid` sebagai mitigasi), diuji terhadap 5 varian traffic (legitimate, dua varian serangan, dan dua varian campuran) masing-masing 40 replikasi — total **400 pengujian beban** menggunakan k6, dengan pengukuran latensi, throughput, metrik internal gateway (Prometheus), dan penggunaan resource container (CPU/memori).

**Temuan utama:**

- Mitigasi **tidak menambah overhead** pada kondisi normal (latensi hybrid sedikit lebih rendah dari baseline).
- Mitigasi **menurunkan beban query PostgreSQL sebesar 93,2%–99,997%** dan **CPU PostgreSQL dari 64–154% menjadi <2,5%** pada mayoritas skenario.
- Mitigasi **melindungi latensi traffic legitimate** saat sistem diserang ($D_{perf}$ p95 = -92,9% pada `mixed-unique`, -39,5% pada `mixed-pool`).
- Ditemukan **trade-off**: pada pola serangan dengan `kid` selalu baru (`*-unique`), rate-limiting berbasis UPSERT per `client_ip` di PostgreSQL menjadi titik kontensi *lock*, sehingga CPU PostgreSQL tetap tinggi (103–124%) dan latensi traffic penyerang pada mode hybrid justru lebih buruk daripada baseline.

Seluruh kode sumber, data eksperimen, skrip analisis, tabel, dan figure tersedia di repository ini (lihat §7 Lampiran untuk peta artefak).

---

## 2. Latar Belakang dan Rumusan Masalah

### 2.1 Latar Belakang

API Gateway pada arsitektur microservices umumnya memvalidasi JSON Web Token (JWT) dengan mengambil kunci publik penandatangan dari *JSON Web Key Set* (JWKS) berdasarkan *Key ID* (`kid`) pada header token. Pada implementasi naif, setiap `kid` yang belum dikenal memicu *lookup* baru ke backing store (database/Identity Service). Penyerang dapat mengeksploitasi pola ini — yang dalam penelitian ini disebut **JWKS Endpoint Flooding** (selaras dengan kelas kerentanan CVE-2026-48524, perlu diverifikasi — lihat [../02-literatur/matriks-literatur.md](../02-literatur/matriks-literatur.md)) — dengan membanjiri gateway menggunakan JWT ber-`kid` acak, sehingga beban *lookup* ke database bertumbuh linear terhadap *request rate* penyerang dan berpotensi menyebabkan *resource exhaustion* yang menurunkan kualitas layanan bagi pengguna sah.

### 2.2 Rumusan Masalah

1. Bagaimana merancang mekanisme caching pada API Gateway yang membatasi dampak JWKS Endpoint Flooding terhadap beban database backend, tanpa menambah latensi signifikan pada traffic legitimate?
2. Seberapa besar efektivitas skema Redis-PostgreSQL Hybrid Caching (positive cache, negative cache, rate limiting berbasis PostgreSQL) dalam menurunkan beban query database dan penggunaan CPU selama serangan?
3. Bagaimana dampak ($D_{perf}$) mitigasi terhadap latensi traffic legitimate, baik pada kondisi normal maupun saat berjalan bersamaan dengan traffic serangan?
4. Apakah strategi serangan `kid` selalu baru (`unique`) vs `kid` berulang dari pool kecil (`pool`) menghasilkan efektivitas dan trade-off mitigasi yang berbeda?

### 2.3 Tujuan Penelitian

Detail tujuan & kontribusi: lihat [../01-proposal/proposal-penelitian.md](../01-proposal/proposal-penelitian.md) §3 dan §5, serta [../07-manuskrip/02-pendahuluan.md](../07-manuskrip/02-pendahuluan.md).

---

## 3. Metodologi dan Pelaksanaan

Penelitian dilaksanakan dalam 5 tahap. Bagian ini merangkum implementasi dan verifikasi setiap tahap; detail teknis lengkap ada pada dokumen `09-docs/tahap-N-*.md` yang dirujuk.

### 3.1 Tahap 1 — Perancangan Arsitektur & Skema Database

**Status: Selesai.** Dirancang arsitektur tiga komponen (Gateway Go/Echo, Redis sebagai L1 cache murni, PostgreSQL sebagai L2/*source of truth*), alur resolusi kunci (positive cache → negative cache → rate-limit PostgreSQL → query `signing_keys`), skema tabel `signing_keys` dan `rate_limit_counters` (dengan *stored procedure* `upsert_rate_limit_counter` untuk UPSERT atomik), dan skema key Redis (`jwks:kid:<kid>`, `jwks:negative:<kid>`). Mode eksperimen `CACHE_MODE=none|hybrid` dirancang sejak tahap ini agar perbandingan baseline-vs-mitigated dapat dilakukan pada infrastruktur identik.

Detail & diagram: [../09-docs/tahap-1-arsitektur-dan-skema-database.md](../09-docs/tahap-1-arsitektur-dan-skema-database.md), [../03-teori/arsitektur-dan-skema.md](../03-teori/arsitektur-dan-skema.md).

### 3.2 Tahap 2 — Implementasi API Gateway (Go)

**Status: Selesai.** Gateway diimplementasikan dengan struktur *clean architecture* per *bounded context* (`internal/jwks`, `internal/ratelimit`, `internal/jwtauth`, `internal/httpapi`, `internal/platform`, `internal/metrics`), menggunakan Echo, `pgx`/`pgxpool`, `go-redis/redis/v9`, `golang-jwt/jwt/v5`, dan `prometheus/client_golang`. Deliverable: migrasi SQL (Sqitch), skrip seed (generate RSA-2048 keypair + sample JWT), middleware verifikasi JWT dengan resolusi `kid` untuk kedua mode, endpoint `/api/resource`, `/healthz`, `/metrics`, serta `docker-compose.yml` dengan healthcheck.

**Verifikasi end-to-end** (manual via curl, kedua mode):
- *Hybrid*: kid valid → `200` (cache miss → DB → fill cache → cache hit pada request berikutnya); kid tidak dikenal → `401 invalid_kid` (negative cache, tidak ada query DB berulang); flood concurrent kid unik → sebagian `429 rate_limited` setelah >20 req/detik per `client_ip`.
- *None*: kid valid selalu `200` dengan `jwksgw_db_queries_total{resolve_key}` naik 1:1 per request; tidak pernah `429`.
- *Fail-closed/fail-open*: PostgreSQL down → `503` (kedua mode); Redis down (hybrid) → kid ter-cache tetap `200` (fallback PostgreSQL), `/healthz` melaporkan `redis:false`.

Catatan lingkungan: PostgreSQL container di-expose ke host pada port 5433 (hindari konflik port lokal); migrasi diverifikasi via `psql` langsung (Sqitch CLI di mesin dev tidak memiliki driver `DBD::Pg`).

Detail: [../09-docs/tahap-2-implementasi-gateway.md](../09-docs/tahap-2-implementasi-gateway.md), kode: [../05-kode/gateway/](../05-kode/gateway/).

### 3.3 Tahap 3 — Pengujian Beban k6

**Status: Selesai — matrix 400 run (40 replikasi) telah dijalankan.** Disusun 3 skrip k6 (`legitimate.js`, `attack.js` dengan `KID_STRATEGY=unique|pool`, `mixed.js` yang menjalankan keduanya secara paralel dengan Trend custom per skenario), runner `run-scenario.sh` (restart gateway sesuai mode, health check, snapshot `/metrics` sebelum/sesudah, jalankan k6, monitor resource), `run-matrix.sh` (loop replikasi × kombinasi mode/varian), dan `monitor-resources.sh` (`docker stats` polling ~3s).

**Iterasi desain penting**: percobaan awal menggunakan `k6 run --out json=...` menghasilkan **139 MB** data mentah hanya untuk 15 detik pengujian — tidak layak untuk matrix penuh. Solusi: ganti ke `--summary-export` (ringkasan agregat) + snapshot `/metrics` gateway before/after (delta = ground truth jumlah query/cache/rate-limit) + Trend custom di `mixed.js`. Hasil: total ukuran matrix awal 50 run **~1,7 MB**.

**Matrix awal (5 replikasi, diarsipkan)**: `CACHE_MODE` ∈ {none, hybrid} × traffic_variant ∈ {legitimate, attack-unique, attack-pool, mixed-unique, mixed-pool} × replikasi 1–5 = **50 run**, dijalankan ~54 menit (2026-06-12T18:05Z–18:59Z), seluruhnya `k6_exit_code = 0`. Dataset ini kemudian diarsipkan ke `04-data/_archive-50run-20260612/`.

**Matrix final (40 replikasi)**: untuk memperbesar sampel statistik, replikasi diperluas menjadi 40 per kombinasi — `CACHE_MODE` ∈ {none, hybrid} × traffic_variant (5 varian) × replikasi 1–40 = **400 run**, dijalankan via `run-matrix.sh` pada 2026-06-15 (selesai `2026-06-15T09:53:24Z`), seluruhnya `k6_exit_code = 0`. Sebelum eksekusi, token JWT legitimate yang sebelumnya *expired* diregenerasi dan cache Redis di-*flush* agar matrix dimulai dari kondisi cache dingin. Dataset 400 run inilah yang menjadi sumber statistik final pada §4.

Output per run: `k6-summary.json`, `gateway-metrics-{before,after}.txt`, `resources.csv`, `meta.json`, disimpan di `04-data/<cache_mode>__<traffic_variant>__rep<N>__<timestamp>/` (tidak disertakan dalam repository git — lihat `.gitignore` — namun seluruh skrip pembangkit tersedia untuk reproduksi).

Detail: [../09-docs/tahap-3-pengujian-k6.md](../09-docs/tahap-3-pengujian-k6.md), kode: [../05-kode/k6/](../05-kode/k6/).

### 3.4 Tahap 4 — Ekstraksi Data & Visualisasi

**Status: Selesai.** Dibangun *pipeline* analisis Python (`05-kode/analysis/`, dijalankan via `python run_all.py`) terdiri dari:

| Modul | Fungsi |
|---|---|
| `common.py` | Helper baca artefak `04-data/<run-id>/` (k6 summary, meta, `/metrics`, `resources.csv`) |
| `load_runs.py` | Bangun DataFrame tidy: ringkasan k6 per run, ringkasan resource, delta `/metrics` gateway |
| `descriptive_stats.py` | Statistik deskriptif latensi/RPS per (cache_mode, traffic_variant) + breakdown legit vs attack pada mixed |
| `compute_dperf.py` | Hitung $D_{perf}$ |
| `resource_stats.py` | CPU%/memori per (cache_mode, traffic_variant, container) |
| `gateway_metrics.py` | Metrik efektivitas mitigasi dari delta `jwksgw_*` |
| `charts.py` | 5 figure PNG |

Output: 6 tabel CSV ([../06-output/tables/](../06-output/tables/)) dan 5 figure PNG ([../06-output/figures/](../06-output/figures/)). Detail & hasil: [../09-docs/tahap-4-analisis-data.md](../09-docs/tahap-4-analisis-data.md).

### 3.5 Tahap 5 — Draf Naskah Jurnal

**Status: Sedang berjalan.** Draf konten per bagian naskah (Abstrak, Pendahuluan, Tinjauan Pustaka, Metodologi, Hasil & Analisis, Kesimpulan, Daftar Pustaka) telah disusun di [../07-manuskrip/](../07-manuskrip/), siap dipindahkan ke template jurnal tujuan. Bagian yang masih perlu dilengkapi: Tinjauan Pustaka (*related work*, lihat [../02-literatur/matriks-literatur.md](../02-literatur/matriks-literatur.md)), verifikasi nomor CVE, dan keputusan bahasa final naskah.

---

## 4. Hasil Penelitian

Ringkasan hasil (detail lengkap & interpretasi: [../07-manuskrip/05-hasil-analisis.md](../07-manuskrip/05-hasil-analisis.md) dan [../09-docs/tahap-4-analisis-data.md](../09-docs/tahap-4-analisis-data.md)).

### 4.1 D_perf — Dampak Mitigasi terhadap Traffic Legitimate

| Kondisi | Metrik | T_none (ms) | T_hybrid (ms) | $D_{perf}$ |
|---|---|---|---|---|
| `legitimate` (tanpa serangan) | avg | 0,6905 | 0,6301 | -8,8% |
| `legitimate` (tanpa serangan) | p95 | 1,0384 | 1,0063 | -3,1% |
| Traffic legit dalam `mixed-unique` | avg | 10,4183 | 0,7721 | -92,6% |
| Traffic legit dalam `mixed-unique` | p95 | 19,4384 | 1,3839 | -92,9% |
| Traffic legit dalam `mixed-pool` | avg | 10,7468 | 5,7595 | -46,4% |
| Traffic legit dalam `mixed-pool` | p95 | 20,5135 | 12,4138 | -39,5% |

### 4.2 Penurunan Beban Query PostgreSQL

| traffic_variant | db_queries `none` (mean) | db_queries `hybrid` (mean) | Reduction |
|---|---|---|---|
| legitimate | 300.114,7 | 10,0 | 99,997% |
| attack-unique | 907.845,5 | 61.894,1 | 93,182% |
| attack-pool | 879.271,7 | 73,1 | 99,992% |
| mixed-unique | 880.678,3 | 57.957,1 | 93,419% |
| mixed-pool | 849.226,3 | 74,6 | 99,991% |

### 4.3 Penggunaan CPU PostgreSQL

| traffic_variant | CPU postgres `none` (mean%) | CPU postgres `hybrid` (mean%) |
|---|---|---|
| legitimate | 64,1 | 2,2 |
| attack-unique | 158,3 | 124,4 |
| attack-pool | 153,9 | 2,2 |
| mixed-unique | 152,5 | 103,0 |
| mixed-pool | 149,9 | 2,2 |

### 4.4 Figure

| File | Isi |
|---|---|
| [`fig_latency_p95.png`](../06-output/figures/fig_latency_p95.png) | Latensi p95 per traffic_variant: none vs hybrid |
| [`fig_dperf.png`](../06-output/figures/fig_dperf.png) | $D_{perf}$ (avg & p95) untuk 3 perbandingan |
| [`fig_db_queries_reduction.png`](../06-output/figures/fig_db_queries_reduction.png) | Total query PostgreSQL per run (log scale) |
| [`fig_postgres_cpu.png`](../06-output/figures/fig_postgres_cpu.png) | CPU% rata-rata container PostgreSQL |
| [`fig_resource_timeseries.png`](../06-output/figures/fig_resource_timeseries.png) | Time-series CPU PostgreSQL selama `mixed-pool` rep1 |

### 4.5 Interpretasi Singkat

1. Mitigasi tidak menambah overhead pada kondisi normal — bahkan sedikit lebih cepat (positive cache hit ratio ≈ 99,997%).
2. Mitigasi melindungi pengalaman pengguna sah secara signifikan saat sistem diserang (D_perf p95 hingga -92,9%).
3. Reduction beban query PostgreSQL 93,2%–99,997% dan CPU PostgreSQL turun ke <2,5% pada skenario `legitimate`, `attack-pool`, `mixed-pool`.
4. **Trade-off**: pada `*-unique`, rate-limiting berbasis UPSERT per `client_ip` menjadi titik kontensi *lock* — CPU PostgreSQL hybrid tetap 103–124% dan latensi traffic penyerang pada hybrid lebih buruk dibanding `none`. Traffic legitimate tetap terlindungi.

---

## 5. Kendala dan Catatan Lingkungan

- **Output k6 mentah (`--out json=`) tidak skalabel** (139 MB/15s) — diatasi dengan `--summary-export` + snapshot `/metrics` + Trend custom (lihat §3.3).
- **Direktori run data kadang terkunci sementara** (`Device or resource busy`) pada Windows/Docker Desktop setelah `docker run --rm` dengan bind mount — transient, hilang sendiri setelah beberapa saat, tidak memerlukan penanganan kode.
- **`MSYS_NO_PATHCONV=1`** diperlukan pada `docker run` via Git Bash (Windows) agar path container tidak diterjemahkan ke path Windows oleh MSYS.
- **Sqitch CLI** di mesin development tidak memiliki driver `DBD::Pg` — migrasi diverifikasi via `psql` langsung; `migrations/` tetap menjadi dokumentasi resmi deploy/revert/verify.
- **PostgreSQL container** di-expose pada port 5433 (bukan 5432 default) untuk menghindari konflik dengan instance PostgreSQL lokal.

---

## 6. Kesimpulan dan Saran

Ringkasan kesimpulan & saran penelitian lanjutan: lihat [../07-manuskrip/06-kesimpulan.md](../07-manuskrip/06-kesimpulan.md).

Inti kesimpulan: skema **Redis-PostgreSQL Hybrid Caching** efektif memitigasi JWKS Endpoint Flooding — tanpa overhead pada kondisi normal, melindungi traffic legitimate secara signifikan saat diserang, dan memangkas beban PostgreSQL 93–99,997% pada mayoritas skenario — dengan satu trade-off teridentifikasi pada desain rate-limiting berbasis baris counter tunggal per klien saat pola serangan menggunakan `kid` yang selalu baru.

---

## 7. Lampiran — Peta Artefak Penelitian

| Folder | Isi | Status |
|---|---|---|
| [01-proposal/](../01-proposal/) | Proposal penelitian | Selesai |
| [02-literatur/](../02-literatur/) | Matriks literatur (kerangka, perlu dilengkapi) | Kerangka tersedia |
| [03-teori/](../03-teori/) | Diagram arsitektur & skema (Tahap 1) | Selesai |
| [04-data/](../04-data/) | Data mentah 400 run/40 replikasi (tidak di-commit, lihat `.gitignore`; matrix awal 50 run/5 replikasi diarsipkan di `_archive-50run-20260612/`) | Tersedia lokal |
| [05-kode/gateway/](../05-kode/gateway/) | Source code API Gateway (Go) | Selesai |
| [05-kode/k6/](../05-kode/k6/) | Skrip pengujian beban k6 | Selesai |
| [05-kode/analysis/](../05-kode/analysis/) | Pipeline analisis Python | Selesai |
| [06-output/](../06-output/) | Tabel & figure hasil analisis | Selesai |
| [07-manuskrip/](../07-manuskrip/) | Draf naskah jurnal (Tahap 5) | Sedang berjalan |
| [08-laporan/](../08-laporan/) | Laporan penelitian (dokumen ini) | Selesai |
| [09-docs/](../09-docs/) | Dokumen rencana & status tiap tahap | Selesai |

**Cara reproduksi penuh:**

```bash
# Tahap 2: jalankan gateway (lihat 05-kode/gateway/README.md)
cd 05-kode/gateway && docker compose up -d

# Tahap 3: jalankan matrix 400 run / 40 replikasi (lihat 05-kode/k6/README.md)
cd 05-kode/k6 && ./run-matrix.sh

# Tahap 4: jalankan pipeline analisis
cd 05-kode/analysis && python run_all.py
```

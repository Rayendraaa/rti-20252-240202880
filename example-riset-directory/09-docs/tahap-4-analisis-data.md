# Tahap 4 — Ekstraksi Data & Visualisasi

**Status:** Selesai — pipeline analisis sudah dijalankan atas matrix 400 run (40 replikasi), tabel & figure tersedia di `06-output/`
**Bergantung pada:** [tahap-3-pengujian-k6.md](tahap-3-pengujian-k6.md)
**Lokasi kode:** [../05-kode/analysis](../05-kode/analysis)

---

## Tujuan

Mengolah data mentah hasil pengujian k6 (`04-data/`) — ringkasan k6, snapshot `/metrics` gateway, dan `resources.csv` — menjadi statistik deskriptif, perhitungan $D_{perf}$, metrik efektivitas mitigasi, dan visualisasi untuk Tahap 5.

## Deliverable

- [x] Skrip pengolahan `k6-summary.json` + `meta.json` → DataFrame tidy (`load_runs.py`)
- [x] Statistik deskriptif (mean/std latensi avg/p90/p95/max, RPS, failed/checks rate) per (cache_mode, traffic_variant)
- [x] Pengumpulan metrik resource (CPU%, memori) container gateway/postgres/redis dari `resources.csv`
- [x] Perhitungan $D_{perf}$ = (T_hybrid − T_none) / T_none × 100% untuk traffic legitimate (baseline & dalam mixed)
- [x] Metrik efektivitas mitigasi dari delta `/metrics` gateway (db queries, cache hit ratio, rate-limit blocked, auth outcome)
- [x] Visualisasi grafik perbandingan (none vs hybrid) per traffic variant
- [x] Ringkasan tabel hasil untuk Tahap 5 (`06-output/tables/`)
- [x] Orkestrator `run_all.py` menjalankan seluruh pipeline sekali jalan

## Desain yang Diimplementasikan

### Struktur kode (`05-kode/analysis/`)

```
05-kode/analysis/
├── requirements.txt        # pandas, numpy, scipy, matplotlib
├── common.py                # helper baca 04-data/<run-id>/ (k6 summary, meta, /metrics, resources.csv)
├── load_runs.py              # build_run_summary / build_resource_summary / build_gateway_metrics
├── descriptive_stats.py       # statistik deskriptif latensi/RPS + breakdown legit vs attack pada mixed
├── compute_dperf.py           # D_perf legitimate (baseline & dalam mixed-unique/mixed-pool)
├── resource_stats.py          # CPU%/memori mean & max per (cache_mode, traffic_variant, container)
├── gateway_metrics.py         # metrik efektivitas mitigasi dari delta /metrics (jwksgw_*)
├── charts.py                  # 5 figure PNG -> 06-output/figures/
└── run_all.py                 # jalankan semua modul di atas berurutan
```

Setiap run dengan `meta.json.k6_exit_code != 0` dilewati (tidak ada pada matrix 400 run saat ini — semua exit 0).

### Modul

| Modul | Fungsi utama | Output |
|---|---|---|
| `load_runs.py` | `build_run_summary()`, `build_resource_summary()`, `build_gateway_metrics()` | DataFrame tidy (dipakai modul lain, tidak menulis file) |
| `descriptive_stats.py` | `build_descriptive_stats()`, `build_mixed_scenario_stats()` | `descriptive_stats.csv`, `descriptive_stats_mixed_scenarios.csv` |
| `compute_dperf.py` | `build_dperf()` | `dperf.csv` |
| `resource_stats.py` | `build_resource_usage()` | `resource_usage.csv` |
| `gateway_metrics.py` | `build_derived_metrics()`, `build_mitigation_effectiveness()`, `build_db_query_reduction()` | `mitigation_effectiveness.csv`, `db_query_reduction.csv` |
| `charts.py` | `fig_latency_p95`, `fig_dperf`, `fig_db_queries_reduction`, `fig_postgres_cpu`, `fig_resource_timeseries` | 5x PNG di `06-output/figures/` |

Cara jalankan (dari `05-kode/analysis/`, environment Python dengan `requirements.txt` terinstal):

```
python run_all.py
```

Atau jalankan modul satu per satu (`python descriptive_stats.py`, dst.) untuk debug.

### Definisi $D_{perf}$

```
D_perf = (T_hybrid - T_none) / T_none * 100%
```

Negatif = `hybrid` lebih cepat (membaik) dibanding `none`; positif = overhead. Dihitung untuk:

1. **`legitimate` (tanpa attack)** — overhead cache hybrid pada kondisi normal, pakai `http_req_duration` avg/p95 keseluruhan.
2. **`mixed-unique` / `mixed-pool`** — dampak mitigasi terhadap pengalaman user legit saat diserang, pakai Trend custom `legitimate_req_duration` (avg/p95) dari `mixed.js`.

### Metrik efektivitas mitigasi

Dari delta `gateway-metrics-{before,after}.txt` (Prometheus `jwksgw_*`):

- `db_queries_total` = `jwksgw_db_queries_total{query_type="resolve_key"}` + `{query_type="rate_limit_upsert"}` — beban Postgres per run.
- `cache_hit_ratio` = hit / (hit+miss) dari `jwksgw_cache_requests_total`.
- `rate_limit_blocked_total` = `jwksgw_rate_limit_blocked_total`.
- `auth_<outcome>` = `jwksgw_auth_requests_total{outcome=...}` untuk `ok|invalid_kid|rate_limited|unavailable|invalid_token`.

`build_db_query_reduction()` menghitung penurunan `db_queries_total` (none → hybrid) per traffic_variant — metrik utama "efektivitas mitigasi" (lebih besar = lebih baik, karena beban Postgres turun drastis saat diserang).

## Hasil

### D_perf (`dperf.csv`, lihat juga `fig_dperf.png`)

| traffic_variant | label | metric | T_none (ms) | T_hybrid (ms) | D_perf |
|---|---|---|---|---|---|
| legitimate | tanpa attack | avg | 0.6905 | 0.6301 | **-8.8%** |
| legitimate | tanpa attack | p95 | 1.0384 | 1.0063 | **-3.1%** |
| mixed-unique | legit traffic dalam mixed-unique | avg | 10.4183 | 0.7721 | **-92.6%** |
| mixed-unique | legit traffic dalam mixed-unique | p95 | 19.4384 | 1.3839 | **-92.9%** |
| mixed-pool | legit traffic dalam mixed-pool | avg | 10.7468 | 5.7595 | **-46.4%** |
| mixed-pool | legit traffic dalam mixed-pool | p95 | 20.5135 | 12.4138 | **-39.5%** |

Hybrid caching **tidak menambah overhead** pada kondisi normal (legitimate tanpa attack justru sedikit lebih cepat, kemungkinan karena Postgres pada `none` masih menanggung query JWKS untuk setiap request). Saat traffic legitimate berjalan bersamaan dengan attack (`mixed-*`), hybrid **melindungi pengalaman user legit secara signifikan** — latensi p95 turun 93% (mixed-unique) dan 39% (mixed-pool) dibanding baseline.

### Penurunan beban query Postgres (`db_query_reduction.csv`, `fig_db_queries_reduction.png`)

| traffic_variant | db_queries none (mean) | db_queries hybrid (mean) | reduction |
|---|---|---|---|
| legitimate | 300.114,7 | 10,0 | **99.997%** |
| attack-unique | 907.845,5 | 61.894,1 | **93.182%** |
| attack-pool | 879.271,7 | 73,1 | **99.992%** |
| mixed-unique | 880.678,3 | 57.957,1 | **93.419%** |
| mixed-pool | 849.226,3 | 74,6 | **99.991%** |

Hybrid caching (positive + negative cache di Redis) memangkas query ke Postgres **93-99.997%** di semua traffic variant. Pada `*-pool` (kid attacker berulang dari pool kecil), negative cache sangat efektif (~99.99% reduction) karena `kid` yang sama langsung ditolak dari Redis tanpa hit Postgres. Pada `*-unique` (kid attacker selalu baru), reduction lebih rendah (~93%) karena setiap `kid` baru tetap memicu satu kali `rate_limit_upsert` ke Postgres sebelum diblokir.

### Beban CPU Postgres (`resource_usage.csv`, `fig_postgres_cpu.png`, `fig_resource_timeseries.png`)

| traffic_variant | CPU postgres none (mean%) | CPU postgres hybrid (mean%) |
|---|---|---|
| legitimate | 64.1 | 2.2 |
| attack-unique | 158.3 | 124.4 |
| attack-pool | 153.9 | 2.2 |
| mixed-unique | 152.5 | 103.0 |
| mixed-pool | 149.9 | 2.2 |

Untuk `legitimate`, `attack-pool`, dan `mixed-pool`, hybrid menurunkan CPU Postgres dari 64-154% menjadi <2.5% — sejalan dengan penurunan query di atas. Untuk `*-unique`, CPU Postgres pada hybrid tetap tinggi (103-124%) karena setiap `kid` baru memicu `upsert_rate_limit_counter` (UPSERT per client_ip+window_start); dengan 200 VU dari satu IP, ini menjadi **lock-contention bottleneck** pada baris counter yang sama — terlihat juga pada `fig_latency_p95.png` di mana `attack-unique` hybrid p95 jauh lebih tinggi dari `none`. Ini adalah trade-off penting: hybrid memperlambat *traffic attacker itu sendiri* pada skenario `*-unique`, namun (lihat D_perf di atas) tetap melindungi traffic legitimate yang berjalan bersamaan pada `mixed-unique`.

### Catatan untuk Tahap 5

- Trade-off `*-unique` vs `*-pool` di atas adalah temuan penting: pola CVE realistis (kid attacker dari pool kecil, `*-pool`) menunjukkan hybrid sangat efektif di semua dimensi (latensi, query Postgres, CPU). Pola `*-unique` (kid selalu baru) adalah edge case yang mengekspos bottleneck pada implementasi rate-limit per-IP berbasis UPSERT row tunggal — relevan untuk bagian "Keterbatasan"/"Future Work" paper.
- Semua angka di atas adalah mean dari 40 replikasi; lihat `descriptive_stats.csv` dan `mitigation_effectiveness.csv` untuk std dev (error bar pada figure).

# 06-output

Hasil olahan data & visualisasi — **Tahap 4** (lihat [../09-docs/tahap-4-analisis-data.md](../09-docs/tahap-4-analisis-data.md)).

Dihasilkan oleh `05-kode/analysis/run_all.py` dari data mentah `04-data/` (matrix 400 run, 40 replikasi).

## tables/

| File | Isi |
|---|---|
| `descriptive_stats.csv` | Statistik deskriptif (latensi avg/p90/p95/max, RPS, failed/checks rate) per (cache_mode, traffic_variant), mean±std atas 40 replikasi |
| `descriptive_stats_mixed_scenarios.csv` | Breakdown latensi legitimate vs attack untuk traffic_variant `mixed-unique`/`mixed-pool` |
| `dperf.csv` | $D_{perf}$ = (T_hybrid − T_none) / T_none × 100% untuk traffic legitimate (baseline & dalam mixed) |
| `resource_usage.csv` | CPU% & memori (MiB) mean/max per (cache_mode, traffic_variant, container) |
| `mitigation_effectiveness.csv` | Metrik efektivitas mitigasi dari delta `/metrics` gateway (db queries, cache hit ratio, rate-limit blocked, auth outcome) |
| `db_query_reduction.csv` | Penurunan total query Postgres hybrid vs none per traffic_variant |

## figures/

| File | Isi |
|---|---|
| `fig_latency_p95.png` | Bar chart `http_req_duration` p95 per traffic_variant: none vs hybrid (mean±std, log scale) |
| `fig_dperf.png` | Bar chart $D_{perf}$ (avg & p95) untuk 3 perbandingan traffic legitimate |
| `fig_db_queries_reduction.png` | Bar chart total query Postgres per run: none vs hybrid (log scale) |
| `fig_postgres_cpu.png` | Bar chart CPU% rata-rata container `gateway-postgres-1`: none vs hybrid |
| `fig_resource_timeseries.png` | Time-series CPU% `gateway-postgres-1` selama `mixed-pool` rep1: none vs hybrid |

## Acuan

[../09-docs/tahap-4-analisis-data.md](../09-docs/tahap-4-analisis-data.md)

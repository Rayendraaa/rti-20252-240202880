# [EXAMPLE] JWKS Flooding — Penelitian Mitigasi JWKS Endpoint Flooding

**Judul:** Performance and Security Evaluation of Mitigating JWKS Endpoint Flooding on Microservices Gateway Using Redis-PostgreSQL Hybrid Caching

**Target publikasi:** Sinta 2 (Jurnal RESTI/Telematika) atau Scopus Q3-Q4

## Ringkasan

Penelitian ini mengevaluasi mitigasi celah keamanan **JWKS Endpoint Flooding** (mirip CVE-2026-48524) — di mana penyerang membanjiri API Gateway dengan JWT ber-`kid` (Key ID) acak sehingga resolver JWKS melakukan kueri tak terbatas ke Identity Service/database, menyebabkan resource exhaustion. Solusi yang diuji adalah skema **Redis-PostgreSQL Hybrid Caching** (positive & negative cache) dengan rate-limiting pada lookup kunci, diimplementasikan pada API Gateway berbasis Go (Echo).

Detail lengkap topik & roadmap: [09-docs/rencana-penelitian.md](09-docs/rencana-penelitian.md)

## Struktur Direktori

| Folder | Isi |
|---|---|
| [00-admin/](00-admin/) | Administrasi penelitian (jadwal, korespondensi) |
| [01-proposal/](01-proposal/) | Proposal penelitian |
| [02-literatur/](02-literatur/) | Referensi & paper terkait (Tinjauan Pustaka) |
| [03-teori/](03-teori/) | Arsitektur & desain sistem (Tahap 1) |
| [04-data/](04-data/) | Data mentah hasil pengujian k6 & metrik container |
| [05-kode/](05-kode/) | Source code: API Gateway (Go) & skrip k6 (Tahap 2 & 3) |
| [06-output/](06-output/) | Statistik & visualisasi hasil pengujian (Tahap 4) |
| [07-manuskrip/](07-manuskrip/) | Draf naskah jurnal (Tahap 5) |
| [08-laporan/](08-laporan/) | Laporan progres/akhir penelitian |
| [09-docs/](09-docs/) | Dokumen perencanaan & roadmap tahap-tahap penelitian |

## Status Tahapan

- [x] **Tahap 1** — Perancangan Arsitektur & Skema Database — *Selesai* ([detail](09-docs/tahap-1-arsitektur-dan-skema-database.md))
- [x] **Tahap 2** — Implementasi API Gateway (Go) — *Selesai* ([detail](09-docs/tahap-2-implementasi-gateway.md))
- [x] **Tahap 3** — Skrip Pengujian k6 (Legitimate vs Attack Traffic) — *Selesai* ([detail](09-docs/tahap-3-pengujian-k6.md))
- [x] **Tahap 4** — Ekstraksi Data & Visualisasi — *Selesai* ([detail](09-docs/tahap-4-analisis-data.md))
- [ ] **Tahap 5** — Draf Paper Jurnal — *Sedang berjalan* ([detail](09-docs/tahap-5-draf-paper.md))

## Laporan Penelitian

Laporan penelitian komprehensif (ringkasan eksekutif, metodologi per tahap, hasil, kendala, kesimpulan): [08-laporan/laporan-penelitian.md](08-laporan/laporan-penelitian.md)

## Author

Helmi Bahar

# Rencana Penelitian: Mitigasi JWKS Endpoint Flooding dengan Redis-PostgreSQL Hybrid Caching

## 1. Ringkasan

| Item | Keterangan |
|---|---|
| Judul | Performance and Security Evaluation of Mitigating JWKS Endpoint Flooding on Microservices Gateway Using Redis-PostgreSQL Hybrid Caching |
| Target Publikasi | Sinta 2 (Jurnal RESTI/Telematika) atau Scopus Q3-Q4 |
| Stack | Docker, PostgreSQL, Redis, API Gateway (Go), k6 |
| Masalah | JWKS Endpoint Flooding via `kid` acak → kueri tak terbatas ke Identity Service/DB → resource exhaustion |
| Solusi | Hybrid cache (Redis L1 + PostgreSQL L2) + negative caching + rate-limiting pada lookup kunci |

## 2. Alur Kerja (Roadmap)

Setiap tahap memiliki file rencana detail tersendiri agar lebih rapi:

- [x] **Tahap 1** — [Perancangan Arsitektur & Skema Database](tahap-1-arsitektur-dan-skema-database.md) — *Selesai*
- [x] **Tahap 2** — [Implementasi API Gateway (Go)](tahap-2-implementasi-gateway.md) — *Selesai*
- [x] **Tahap 3** — [Skrip Pengujian k6 (Legitimate vs Attack Traffic)](tahap-3-pengujian-k6.md) — *Selesai*
- [x] **Tahap 4** — [Ekstraksi Data & Visualisasi](tahap-4-analisis-data.md) — *Selesai*
- [ ] **Tahap 5** — [Draf Paper Jurnal](tahap-5-draf-paper.md) — *Berikutnya*

---

## 3. Catatan

Dokumen ini adalah indeks utama. Detail teknis, skema, dan keputusan masing-masing tahap dicatat pada file `tahap-N-*.md` terkait dan diperbarui seiring progres pengerjaan.

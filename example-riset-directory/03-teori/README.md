# 03-teori

Arsitektur, desain, dan landasan teori sistem — hasil **Tahap 1**.

## Isi yang diharapkan

- Diagram alur resolusi kunci (mitigasi JWKS flooding)
- Skema database PostgreSQL (`signing_keys`, `rate_limit_counters`)
- Skema Redis (positive/negative cache)
- Diagram arsitektur komponen (Gateway, Redis, PostgreSQL)

## Berkas

- [arsitektur-dan-skema.md](arsitektur-dan-skema.md) — diagram Mermaid (arsitektur komponen, alur resolusi kunci, fail-closed/fail-open, ERD database), skema Redis, dan pemetaan ke implementasi kode

## Acuan

Detail teknis lengkap Tahap 1 (status: selesai): [../09-docs/tahap-1-arsitektur-dan-skema-database.md](../09-docs/tahap-1-arsitektur-dan-skema-database.md)

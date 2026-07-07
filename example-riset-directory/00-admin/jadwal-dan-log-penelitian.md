# Jadwal & Log Pelaksanaan Penelitian

Catatan kronologis pelaksanaan tiap tahap (sumber: riwayat commit git & dokumen `09-docs/tahap-N-*.md`). Tanggal mengikuti `git log`.

## Log Pelaksanaan

| Tanggal | Tahap | Aktivitas | Referensi |
|---|---|---|---|
| 2026-06-12 s.d. 2026-06-13 (commit 01:05) | Tahap 1 & 2 | Perancangan arsitektur/skema database; implementasi API Gateway Go (Echo) — clean architecture, migrasi Sqitch, seed script, docker-compose, verifikasi end-to-end (`CACHE_MODE=none`/`hybrid`, fail-closed/fail-open) | [09-docs/tahap-1-arsitektur-dan-skema-database.md](../09-docs/tahap-1-arsitektur-dan-skema-database.md), [09-docs/tahap-2-implementasi-gateway.md](../09-docs/tahap-2-implementasi-gateway.md) |
| 2026-06-13 01:05 | Tahap 3 | Implementasi skrip k6 (`legitimate.js`, `attack.js`, `mixed.js`), runner & monitor resource | [09-docs/tahap-3-pengujian-k6.md](../09-docs/tahap-3-pengujian-k6.md) |
| 2026-06-12 18:05–18:59 (≈54 menit) | Tahap 3 | Eksekusi matrix penuh 50 run (2 `CACHE_MODE` × 5 `traffic_variant` × 5 replikasi), seluruhnya `k6_exit_code = 0` | commit "Mark Tahap 3 complete after running full 50-run k6 matrix" (2026-06-13 02:00) |
| 2026-06-13 07:41 | Tahap 4 | Pipeline analisis Python (`run_all.py`), 6 tabel CSV + 5 figure PNG, dokumen Tahap 4 diperbarui ke status Selesai | [09-docs/tahap-4-analisis-data.md](../09-docs/tahap-4-analisis-data.md), [06-output/](../06-output/) |
| 2026-06-13 | Tahap 5 | Draf konten naskah (8 bagian) di `07-manuskrip/`; pelengkapan `01-proposal/`, `02-literatur/`, `03-teori/`, dan laporan penelitian `08-laporan/` | [09-docs/tahap-5-draf-paper.md](../09-docs/tahap-5-draf-paper.md), [08-laporan/laporan-penelitian.md](../08-laporan/laporan-penelitian.md) |
| 2026-06-13 | Tahap 5 | Verifikasi CVE-2026-48524 (terkonfirmasi via GHSA-fhv5-28vv-h8m8); pencarian 18 referensi literatur nyata & penyusunan bibliografi Mendeley; pelengkapan §2.4 *Related Work* di `03-tinjauan-pustaka.md` dan `07-daftar-pustaka.md`; penyusunan naskah konsolidasi `naskah-jurnal.md`/`.docx` | [02-literatur/matriks-literatur.md](../02-literatur/matriks-literatur.md), [02-literatur/daftar-pustaka.bib](../02-literatur/daftar-pustaka.bib), [07-manuskrip/naskah-jurnal.md](../07-manuskrip/naskah-jurnal.md) |
| 2026-06-15 | Tahap 3 & 4 | Perluasan replikasi dari 5 menjadi 40 per kombinasi: regenerasi token JWT legitimate (sebelumnya *expired*), flush cache Redis, eksekusi matrix penuh 400 run (2 `CACHE_MODE` × 5 `traffic_variant` × 40 replikasi) via `run-matrix.sh`, seluruhnya `k6_exit_code = 0` (selesai 2026-06-15T09:53:24Z); dataset 50-run lama diarsipkan ke `04-data/_archive-50run-20260612/`; pipeline analisis (`run_all.py`) dijalankan ulang atas dataset baru; seluruh statistik di `naskah-jurnal.md`/`.docx`, `00-outline.md`, dan dokumen `09-docs/`/`08-laporan/`/`01-proposal/` diperbarui ke n=40 | [09-docs/tahap-3-pengujian-k6.md](../09-docs/tahap-3-pengujian-k6.md), [09-docs/tahap-4-analisis-data.md](../09-docs/tahap-4-analisis-data.md), [04-data/matrix-40run.log](../04-data/matrix-40run.log) |

## Status Ringkas

- **Tahap 1–4**: Selesai (dataset final: matrix 400 run / 40 replikasi per kombinasi, 2026-06-15).
- **Tahap 5**: Konten naskah selesai dengan statistik n=40 (termasuk tinjauan pustaka & verifikasi CVE-2026-48524); menyisakan keputusan bahasa final dan pemindahan ke template jurnal tujuan (dilakukan oleh peneliti).

## Item Tindak Lanjut (Checklist Sebelum Submission)

- [x] Lengkapi matriks literatur dengan paper *related work* nyata ([02-literatur/matriks-literatur.md](../02-literatur/matriks-literatur.md)) — 18 referensi terverifikasi
- [x] Verifikasi CVE-2026-48524 terhadap basis data NVD/MITRE — terkonfirmasi via GHSA-fhv5-28vv-h8m8 (PyJWT, CVSS 3.7)
- [ ] Tetapkan bahasa final naskah (Indonesia/Inggris) sesuai jurnal tujuan
- [ ] Pindahkan konten [07-manuskrip/naskah-jurnal.md](../07-manuskrip/naskah-jurnal.md)/`.docx` ke template jurnal tujuan
- [ ] Finalisasi penempatan figure/tabel sesuai gaya jurnal
- [ ] Review akhir seluruh klaim numerik agar konsisten antar dokumen (lihat daftar pada [07-manuskrip/00-outline.md](../07-manuskrip/00-outline.md))

## Korespondensi

*(belum ada — tambahkan catatan korespondensi dengan pembimbing/editor jurnal di sini saat tersedia)*

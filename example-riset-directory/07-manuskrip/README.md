# 07-manuskrip

Draf naskah ilmiah — **Tahap 5**, target publikasi Sinta 2 (Jurnal RESTI/Telematika) atau Scopus Q3-Q4.

## Naskah konsolidasi

- [naskah-jurnal.md](naskah-jurnal.md) — naskah lengkap dalam template jurnal standar (Judul/Penulis, Abstrak ID+EN, §1 Pendahuluan – §5 Kesimpulan, Daftar Pustaka)
- [naskah-jurnal.docx](naskah-jurnal.docx) — hasil konversi ke .docx (dibangun via `python build_docx.py`, lihat [build_docx.py](build_docx.py); tidak memerlukan pandoc)

## Struktur naskah per bagian (sumber/draf kerja)

- [00-outline.md](00-outline.md) — outline, peta sumber, dan daftar klaim kunci yang harus konsisten
- [01-abstrak.md](01-abstrak.md) — Abstrak (ID & EN)
- [02-pendahuluan.md](02-pendahuluan.md) — Pendahuluan (latar belakang, rumusan masalah, tujuan, kontribusi)
- [03-tinjauan-pustaka.md](03-tinjauan-pustaka.md) — Tinjauan Pustaka (JWT/JWKS, mitigasi, *related work*; lihat [../02-literatur/](../02-literatur/))
- [04-metodologi.md](04-metodologi.md) — Metodologi (arsitektur eksperimen, skema hybrid caching, skenario k6)
- [05-hasil-analisis.md](05-hasil-analisis.md) — Hasil & Analisis (mengacu pada [../06-output/](../06-output/))
- [06-kesimpulan.md](06-kesimpulan.md) — Kesimpulan & Saran Penelitian Lanjutan
- [07-daftar-pustaka.md](07-daftar-pustaka.md) — Daftar Pustaka (18 referensi, format IEEE; BibTeX di [../02-literatur/daftar-pustaka.bib](../02-literatur/daftar-pustaka.bib))

> `naskah-jurnal.md`/`.docx` adalah gabungan final dari bagian-bagian di atas. Pemindahan ke template jurnal tujuan (margin, sitasi, kolom spesifik) dilakukan oleh peneliti.

## Acuan

[../09-docs/tahap-5-draf-paper.md](../09-docs/tahap-5-draf-paper.md)

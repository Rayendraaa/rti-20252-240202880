# WS-12: Result Presentation & Visualization

> **Bab 12 — Penyajian Hasil & Visualisasi**
> **Tema Eksperimen (lanjutan WS-09/10/11): Penyajian Hasil E2EE Hybrid RSA + AES-256-CBC**

---

## Konteks Eksperimen

Data yang sudah divalidasi di WS-11 (`data_log.csv`, 21 run, dinyatakan "siap analisis") sekarang disajikan dalam bentuk tabel dan visualisasi untuk menjawab pertanyaan riset: **"Bagaimana waktu enkripsi/dekripsi AES-256-CBC berskala terhadap ukuran pesan?"**

---

## Template A.12 — Result Presentation Plan (terisi)

```
RESULT PRESENTATION PLAN

Research Question : Bagaimana waktu enkripsi & dekripsi AES-256-CBC (dalam skema
                     hybrid dengan RSA-2048) berskala terhadap ukuran pesan?
Metrik Utama      : Waktu enkripsi (ms), waktu dekripsi (ms)

Tabel Hasil:
| Skenario | Waktu Enkripsi (mean ± std, ms) | Waktu Dekripsi (mean ± std, ms) | n |
|----------|----------------------------------|----------------------------------|---|
| 100 B    | 0.588 ± 0.015                    | 2.226 ± 0.444                    | 5 |
| 1 KB     | 0.818 ± 0.442                    | 2.125 ± 0.216                    | 5 |
| 10 KB    | 0.724 ± 0.073                    | 2.014 ± 0.053                    | 5 |
| 100 KB   | 1.864 ± 0.673                    | 2.667 ± 0.281                    | 5 |

Visualisasi yang Direncanakan:
| # | Jenis Grafik | Pesan Utama | Metrik |
|---|-------------|-------------|--------|
| 1 | Bar chart + error bar | Perbandingan rata-rata waktu enkripsi antar-skenario | Mean enc time ± std |
| 2 | Box plot | Sebaran/variabilitas waktu enkripsi tiap skenario (termasuk outlier run-001) | Semua 5 run per skenario |
| 3 | Scatter plot (log-x) | Trade-off ukuran pesan vs waktu enkripsi, termasuk titik anomali 5MB | Semua 21 run |

Bias Check:
  [x] Y-axis mulai dari 0 (atau dijustifikasi)
  [x] Error bar/CI ditampilkan
  [x] Semua data disertakan (tidak cherry-picked) — termasuk run-001 & run-021 yang anomali
  [x] Tidak menggunakan 3D tanpa alasan
```

---

## Latihan 1 — Tabel Hasil

| Skenario | Waktu Enkripsi (mean ± std, ms) | Waktu Dekripsi (mean ± std, ms) | n |
|----------|----------------------------------|----------------------------------|---|
| 100 B  | 0.588 ± 0.015 | 2.226 ± 0.444 | 5 |
| 10 KB  | 0.724 ± 0.073 | 2.014 ± 0.053 | 5 |
| 1 KB   | 0.818 ± 0.442 | 2.125 ± 0.216 | 5 |
| 100 KB | 1.864 ± 0.673 | 2.667 ± 0.281 | 5 |

*Diurutkan berdasarkan waktu enkripsi (metrik utama). N=5 per skenario. Mean ± std dalam milidetik. AES-256-CBC + RSA-2048, PHP 8.3.6/OpenSSL 3.0.13.*
*Catatan: skenario 1 KB mengandung run-001 (1.701 ms) yang teridentifikasi sebagai contextual anomaly di WS-11 — nilai tetap disertakan sesuai keputusan validasi (data siap analisis, dianotasi bukan dihapus), sehingga std 1 KB tampak lebih tinggi dari 10 KB meski ukuran pesannya lebih kecil.*

**Checklist tabel:**
- [x] Self-contained (judul jelas, satuan ada, N tercantum)
- [x] Mean ± std (bukan single number)
- [x] Diurutkan berdasarkan metrik utama
- [x] Format konsisten di semua baris

---

## Latihan 2 — Rencana Visualisasi

| # | Jenis Grafik | Pesan | Data yang Digunakan |
|---|-------------|-------|---------------------|
| 1 | Bar chart + error bar | Waktu enkripsi naik seiring ukuran pesan, dengan ketidakpastian (std) ditampilkan eksplisit | Mean & std enc time, 4 skenario utama |
| 2 | Box plot | Skenario 1 KB punya sebaran (spread) jauh lebih lebar dibanding 100 B/10 KB — mengungkap outlier run-001 secara visual | Semua 5 run per skenario |
| 3 | Scatter plot (skala log) | Hubungan ukuran pesan vs waktu enkripsi tidak linear sederhana; titik 5 MB menunjukkan lonjakan tajam di luar tren skenario kecil | Semua 21 run, termasuk anomali 5 MB |

### Grafik 1 — Bar chart perbandingan waktu enkripsi antar-skenario

<img width="980" height="700" alt="bar_enc_time" src="https://github.com/user-attachments/assets/e400eb79-3d75-4b98-8f72-6c12bad832cb" />


**Interpretasi:** Waktu enkripsi rata-rata naik dari 100 B (0.588 ms) ke 100 KB (1.864 ms). Error bar pada 1 KB terlihat jauh lebih panjang dibanding 100 B dan 10 KB — sinyal visual bahwa skenario 1 KB punya variabilitas tidak wajar (dijelaskan di Grafik 2).

### Grafik 2 — Box plot distribusi waktu enkripsi per skenario

<img width="980" height="700" alt="boxplot_enc_time" src="https://github.com/user-attachments/assets/52d1fce4-b999-4cf9-9ca8-3dff603dfb03" />


**Interpretasi:** Box plot skenario 1 KB menunjukkan satu titik jauh di atas box (whisker atas memanjang) — ini adalah run-001 (1.701 ms) yang sudah diinvestigasi sebagai contextual anomaly di WS-11 (dugaan cold-start OPcache). Box plot 100 B dan 10 KB jauh lebih rapat, konsisten dengan std kecil di tabel Latihan 1.

### Grafik 3 — Scatter plot trade-off ukuran pesan vs waktu enkripsi
<img width="980" height="700" alt="scatter_size_vs_time" src="https://github.com/user-attachments/assets/c730d787-974a-49a0-88bb-b4038d01c433" />


**Interpretasi:** Pada skala log, empat skenario utama (100 B–100 KB) menunjukkan kenaikan waktu enkripsi yang relatif landai, sementara titik 5 MB (run-021, kanan atas) melonjak jauh dari tren tersebut — konsisten dengan status `ANOMALY_DOCUMENTED` yang diberikan di WS-10/WS-11, bukan mengikuti pola linear sederhana skenario kecil.

---

## Latihan 3 — Bias Detection

**Skenario:** Metode A = 91.2%, Metode B = 90.8%. Bar chart dengan Y-axis mulai dari 90%.

| Pertanyaan | Jawaban |
|-----------|---------|
| Apakah Y-axis menyesatkan? | Ya — dengan sumbu Y mulai dari 90%, selisih 0.4% terlihat seperti perbedaan besar (bar A terlihat jauh lebih tinggi dari B), padahal secara absolut perbedaannya kecil |
| Apakah error bar ditampilkan? | Tidak disebutkan dalam skenario — ini pelanggaran tambahan: tanpa error bar/std, tidak bisa dinilai apakah selisih 0.4% signifikan atau hanya noise antar-run |
| Apakah semua kondisi ditampilkan? | Tidak diketahui dari deskripsi skenario — perlu dicek apakah hanya 2 metode "terbaik" yang ditampilkan sementara metode lain yang berkinerja lebih rendah disembunyikan (cherry-picking) |
| Apa solusinya? | Set Y-axis mulai dari 0% (atau jika perlu zoom, beri catatan eksplisit + tunjukkan juga versi Y dari 0%), tambahkan error bar (mean ± std, n run), dan tampilkan semua metode/kondisi yang diuji, bukan hanya yang "menang" |

**Evaluasi grafik sendiri dari Latihan 2:**
- [x] Semua bias check lulus
- [ ] Ada yang perlu diperbaiki: ____

**Justifikasi kelulusan bias check:**
> - Y-axis ketiga grafik (bar, box plot) mulai dari 0 — tidak ada truncated axis
> - Grafik 1 menampilkan error bar (std) secara eksplisit di atas tiap bar
> - Grafik 2 (box plot) dan Grafik 3 (scatter) menampilkan SEMUA titik data, termasuk run-001 dan run-021 yang anomali — tidak cherry-picked
> - Scatter plot memakai skala log pada sumbu X (bukan 3D atau distorsi visual lain) karena rentang ukuran pesan (100 B hingga 5 MB) sangat lebar — pemakaian skala log dijustifikasi, bukan untuk menyembunyikan pola

---

## Refleksi

> Mengapa tabel dan grafik keduanya diperlukan — tidak cukup salah satu saja? Pernahkah Anda membuat grafik yang (tanpa sengaja) menyesatkan?

> Tabel di Latihan 1 memberi angka presisi (mis. 100 KB = 1.864 ± 0.673 ms) yang bisa dikutip langsung di laporan atau dibandingkan baris per baris. Tapi tabel saja tidak menunjukkan bahwa skenario 1 KB punya SATU titik outlier yang mendorong std-nya tinggi — informasi itu baru terlihat jelas begitu data divisualisasikan sebagai box plot (Grafik 2). Sebaliknya, grafik saja (tanpa tabel) tidak memberi angka pasti untuk dikutip atau dibandingkan secara numerik dengan eksperimen lain. Kombinasi keduanya — tabel untuk presisi, grafik untuk pola — memastikan tidak ada informasi penting yang tersembunyi hanya karena dipilih satu bentuk penyajian saja.
>
> Pada percobaan awal (sebelum menyusun WS-12 ini), sempat terpikir untuk langsung membuat bar chart mean waktu enkripsi tanpa error bar, karena hasilnya "terlihat rapi" dan bar-nya berurutan naik sesuai ukuran pesan. Baru setelah menambahkan error bar dan box plot, terlihat bahwa skenario 1 KB sebenarnya punya variabilitas tinggi (bukan angka yang stabil) — kalau grafik awal tanpa error bar itu dipakai, orang yang membaca bisa salah menyimpulkan bahwa 1 KB "selalu" memakan waktu ~0.818 ms, padahal 4 dari 5 run-nya sebenarnya di bawah 0.65 ms.

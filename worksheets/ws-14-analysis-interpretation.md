# WS-14: Analysis, Interpretation & Failure Analysis

> **Bab 14 — Analisis Data, Interpretasi & Failure Analysis**
> **Tema Eksperimen (lanjutan WS-09/10/11/12/13): Analisis Waktu Enkripsi E2EE Hybrid RSA-2048 + AES-256-CBC**

---

## Konteks Eksperimen

Dataset `data_log.csv` (21 run, tervalidasi di WS-11, divisualisasikan di WS-12, dipreprocessing di WS-13) sekarang dianalisis secara formal untuk menjawab dua pertanyaan:

1. **RQ-utama:** Apakah ukuran pesan memengaruhi waktu enkripsi AES-256-CBC secara **signifikan** (bukan sekadar "terlihat naik" di tabel WS-12)? → diuji pada 20 data skenario utama (100 B–100 KB, n=5/skenario).
2. **RQ-sekunder (dari model WS-13):** Apakah model regresi linear (`enc_time ~ message_size`) yang dilatih di WS-13 benar-benar **generalisasi** dengan baik, atau justru gagal — dan jika gagal, apa yang bisa dipelajari? → di sinilah *failure analysis* worksheet ini berperan, karena hasil model WS-13 pada test set ternyata **jauh lebih buruk daripada terlihat di R² training**.

Skenario 5 MB (run-021) tetap dikeluarkan dari uji hipotesis utama (konsisten dengan keputusan WS-13), tapi dipakai ulang di sini sebagai kasus **boundary condition** untuk failure analysis.

---

## Template A.14 — Analysis & Interpretation Report (terisi)

```
ANALYSIS & INTERPRETATION

1. Statistik Deskriptif (enc_time_ms, n=5/skenario, dari data_log.csv):
   | Skenario | Mean  | Std   | Median | Min   | Max   | n |
   |----------|-------|-------|--------|-------|-------|---|
   | 100 B    | 0.588 | 0.017 | 0.593  | 0.568 | 0.604 | 5 |
   | 1 KB     | 0.818 | 0.494 | 0.591  | 0.569 | 1.701 | 5 |
   | 10 KB    | 0.724 | 0.082 | 0.715  | 0.647 | 0.855 | 5 |
   | 100 KB   | 1.864 | 0.753 | 1.448  | 1.298 | 3.015 | 5 |

2. Uji Hipotesis:
   Uji yang digunakan  : Kruskal-Wallis H test (bukan one-way ANOVA)
   Justifikasi          : Shapiro-Wilk pada gabungan 20 data (W=0.697, p<0.001) menolak
                           normalitas; grup 1 KB sendiri sangat non-normal (W=0.599, p=0.0006)
                           akibat outlier run-001 (1.701 ms). Sesuai tabel pemilihan uji
                           materi (">2 grup, non-normal → Kruskal-Wallis + post-hoc").
   Hasil: H(3) = 12.65, p = 0.0055, effect size (eta-squared) = 0.60
   CI 95% (bootstrap, selisih median 100B vs 100KB) : [0.705, 2.422] ms

3. Keputusan:
   [x] H0 ditolak → H1 diterima (ukuran pesan memengaruhi waktu enkripsi secara signifikan)
   [ ] H0 tidak ditolak

4. Interpretasi:
   Hubungan ke RQ       : Waktu enkripsi AES-256-CBC berbeda signifikan antar-skenario
                           ukuran pesan (p=0.0055 < 0.05), dengan effect size BESAR (η²=0.60,
                           di atas ambang "large" 0.14). Post-hoc Mann-Whitney+Bonferroni
                           (α_koreksi=0.0083) menunjukkan perbedaan signifikan hanya pada
                           pasangan yang rentang ukurannya lebar: 100B vs 10KB (p=0.008),
                           100B vs 100KB (p=0.008), 10KB vs 100KB (p=0.008). Pasangan yang
                           berdekatan atau melibatkan grup 1KB (100B vs 1KB, 1KB vs 10KB,
                           1KB vs 100KB) TIDAK signifikan setelah koreksi — konsisten dengan
                           varians tinggi grup 1KB akibat outlier run-001.
   Practical significance: Signifikan secara statistik DAN praktis — kenaikan median dari
                           100B (0.593 ms) ke 100KB (1.448 ms) adalah >2x lipat, cukup besar
                           untuk relevan dalam desain sistem chat ber-volume tinggi.
   Perbandingan literatur: Konsisten dengan kompleksitas teoritis AES-CBC yang linear O(n)
                           terhadap ukuran plaintext, namun kenaikan yang terukur di sini
                           TIDAK murni linear pada rentang kecil (100B→10KB relatif datar)
                           karena didominasi overhead RSA (konstan per panggilan) — pola ini
                           baru terlihat proporsional pada rentang besar (100KB ke atas),
                           lihat Failure Analysis di bawah.

5. Limitation:
   | Jenis | Ancaman | Dampak | Mitigasi |
   |-------|---------|--------|----------|
   | Statistical | n=5/skenario (kecil), 1 outlier (run-001) mendominasi grup 1KB | Post-hoc pada pasangan yang melibatkan 1KB kehilangan power, berisiko Type II error | Tambah run (n≥10), atau ulangi analisis dengan 1KB tanpa run-001 sebagai sensitivity check |
   | Internal validity | Faktor sistem tak terkontrol penuh (cold-start OPcache, thermal, background process — dicatat di WS-09/11) | Variasi enc_time bisa sebagian berasal dari noise sistem, bukan murni ukuran pesan | Tambahkan warm-up run yang dibuang, isolasi proses saat pengukuran |
   | External validity | Hanya diuji pada 1 mesin, 1 versi PHP/OpenSSL, rentang 100B–100KB | Generalisasi ke hardware/OS lain atau ukuran pesan sangat besar (>1MB) belum terjamin | Replikasi di environment lain; uji rentang ukuran lebih luas (lihat boundary condition 5MB) |
   | Construct validity | `enc_time_ms` diukur di level aplikasi (microtime PHP), bukan di level cipher murni | Angka mencakup overhead PHP call stack, bukan cost AES-CBC "murni" | Untuk klaim performa cipher murni, ukur di level C/OpenSSL benchmark terpisah |

6. Failure Analysis:
   RQ utama (perbedaan enc_time antar skenario) TIDAK gagal — H0 ditolak dengan effect
   size besar. Namun ada kegagalan yang justru lebih informatif: MODEL REGRESI dari WS-13.

   Penyebab potensial : Model dilatih (train, n=16) menghasilkan R² = 0.75 (r=0.866, p<0.001
                         untuk slope) — terlihat kuat. Tapi saat dievaluasi di test set (n=4,
                         split seed=2026 dari WS-13), R² = -1.18 (LEBIH BURUK dari sekadar
                         menebak rata-rata). Penyebabnya: run-001 (1KB, contextual anomaly
                         yang sudah diidentifikasi di WS-11) kebetulan JATUH di test set,
                         dengan residual +1.104 ms — sendirian mendominasi total error pada
                         test set yang hanya berisi 4 titik data.
   Boundary condition   : Saat model diekstrapolasi ke skenario 5MB (di luar rentang
                         training 100B–100KB), model memprediksi 73.4 ms, padahal aktualnya
                         127.3 ms — error relatif 42%. Model linear valid untuk rentang kecil
                         (100B–100KB, di mana overhead RSA konstan mendominasi), tapi GAGAL
                         di luar rentang itu karena pada ukuran sangat besar, biaya alokasi
                         memori & operasi AES per-blok mulai mendominasi secara non-linear —
                         asumsi "hubungan linear sederhana" dilanggar.
   Insight              : (1) Dengan n test set sekecil 4, SATU titik outlier bisa membalik
                         kesimpulan model dari "baik" (R² train=0.75) menjadi "gagal total"
                         (R² test=-1.18) — evaluasi model pada sampel kecil sangat rapuh
                         terhadap outlier tunggal, bukan cuma soal ukuran sampel training.
                         (2) Model linear sederhana punya boundary condition eksplisit:
                         valid hanya di rentang 100B–100KB; direkomendasikan pendekatan
                         piecewise/non-linear (mis. tambahkan fitur log(size) atau size²)
                         jika model perlu dipakai untuk rentang pesan yang lebih lebar.
```

---

## Latihan 1 — Pemilihan Uji Statistik

| Pertanyaan | Jawaban |
|-----------|---------|
| Berapa grup yang dibandingkan? | 4 (skenario ukuran pesan: 100 B, 1 KB, 10 KB, 100 KB) |
| Apakah data berpasangan (paired)? | Tidak — setiap grup adalah 5 run independen dengan seed berbeda, bukan pengukuran berulang pada subjek yang sama |
| Apakah distribusi normal? (uji normalitas) | Beragam per grup (Shapiro-Wilk): 100B p=0.261 (normal), 10KB p=0.444 (normal), 100KB p=0.124 (normal), **1KB p=0.0006 (TIDAK normal**, akibat outlier run-001). Gabungan 20 data: p<0.001 (tidak normal) |
| **Uji yang dipilih:** | Kruskal-Wallis H test + post-hoc Mann-Whitney U dengan koreksi Bonferroni |
| **Justifikasi:** | Karena minimal satu dari 4 grup (1KB) melanggar asumsi normalitas secara signifikan, dan pelanggaran ini disebabkan outlier yang sudah terdokumentasi (bukan kesalahan data) — menggunakan uji non-parametrik lebih aman daripada ANOVA yang mengasumsikan normalitas. Sebagai pembanding, ANOVA tetap dihitung (F=8.36, p=0.0014) dan hasilnya SAMA-SAMA signifikan, jadi kesimpulan robust terhadap pilihan uji — tapi Kruskal-Wallis dipilih sebagai laporan utama karena lebih sesuai asumsi data. |

**Effect size yang akan dilaporkan:** [x] Eta-squared (untuk Kruskal-Wallis, η² = (H−k+1)/(n−k)) / [ ] Cohen's d / [ ] Lainnya

---

## Latihan 2 — Interpretasi Hasil

**Data (dari `data_log.csv`, 20 skenario utama):**

| Skenario | Waktu Enkripsi (mean ± std, ms) | n |
|----------|----------------------------------|---|
| 100 B    | 0.588 ± 0.017 | 5 |
| 1 KB     | 0.818 ± 0.494 | 5 |
| 10 KB    | 0.724 ± 0.082 | 5 |
| 100 KB   | 1.864 ± 0.753 | 5 |

H(3) = 12.65, p = 0.0055, η² = 0.60, CI 95% (selisih median 100B→100KB, bootstrap) = [0.705, 2.422] ms

| Aspek | Interpretasi |
|-------|-------------|
| Signifikansi statistik | p = 0.0055 < 0.05 → signifikan pada α=0.05, bahkan pada α=0.01. Ukuran pesan memengaruhi waktu enkripsi secara statistik meyakinkan. |
| Effect size | η² = 0.60 → **large effect** (jauh di atas ambang umum 0.14 untuk "large" dalam konteks ANOVA/Kruskal-Wallis). Artinya ~60% variasi waktu enkripsi antar-run dapat "dijelaskan" oleh perbedaan skenario ukuran pesan. |
| Practical significance | Signifikan secara praktis: median waktu enkripsi 100KB (1.448 ms) lebih dari 2× median 100B (0.593 ms). Untuk sistem chat dengan volume pesan besar/berbasis file, perbedaan ini terasa di skala agregat meski di level milidetik tunggal tampak kecil. |
| Hubungan ke RQ | Menjawab RQ WS-12 ("Bagaimana waktu enkripsi berskala terhadap ukuran pesan?") secara formal: bukan hanya "terlihat naik" di grafik, tapi terbukti signifikan secara statistik dengan effect size besar — grafik WS-12 dan uji WS-14 saling menguatkan (bukan bertentangan). |
| Perbandingan literatur | AES-CBC secara teoritis O(n) terhadap ukuran plaintext. Post-hoc menunjukkan pola ini baru "kelihatan" pada rentang lebar (100B vs 10KB/100KB signifikan), sementara pada rentang sempit (100B vs 1KB, 1KB vs 10KB) TIDAK signifikan — konsisten dengan literatur bahwa pada ukuran pesan kecil, overhead RSA key-exchange (konstan per panggilan) mendominasi dibanding biaya AES itu sendiri, sehingga efek ukuran pesan "tenggelam" oleh noise/overhead pada rentang kecil. |

---

## Latihan 3 — Failure Analysis

**Skenario:** Model regresi linear WS-13 (`enc_time_ms ~ message_size_bytes`, dilatih pada 16 data train) mencatat R² = 0.75 saat training (r = 0.866, p < 0.001 untuk koefisien slope) — terlihat sangat baik. Tapi saat dievaluasi ulang di sini pada 4 data test (split identik dengan WS-13, seed=2026), **R² = -1.18** — lebih buruk daripada model "dummy" yang selalu menebak rata-rata test set.

| Pertanyaan | Jawaban |
|-----------|---------|
| Apakah ini "gagal"? | Ya, secara teknis model gagal generalisasi pada test set — tapi ini BUKAN kegagalan riset. Justru ini temuan bernilai tentang kerapuhan evaluasi model pada sampel test yang sangat kecil (n=4). |
| Kemungkinan penyebab? | Saat data displit (WS-13, seed=2026), run-001 — skenario 1KB dengan waktu enkripsi 1.701 ms yang sudah teridentifikasi sebagai *contextual anomaly* sejak WS-11 (diduga cold-start OPcache) — kebetulan jatuh ke test set, bukan train set. Residual run-001 sendiri (+1.104 ms) sudah lebih besar dari total variasi 3 titik test lainnya gabungan, sehingga menghancurkan R² test meski model pada dasarnya menangkap tren yang benar (terbukti dari 3 titik test lain yang residualnya kecil: 0.009, 0.013, -0.707). |
| Boundary condition? | Model hanya valid untuk rentang ukuran pesan 100B–100KB (rentang training). Saat diekstrapolasi ke 5MB (run-021, di luar rentang training), model memprediksi 73.4 ms sementara aktualnya 127.3 ms (error relatif 42%) — model linear meremehkan waktu enkripsi pada ukuran sangat besar karena hubungan size↔time tidak lagi murni linear di luar rentang kecil-menengah (efek non-linear seperti overhead alokasi memori untuk buffer 5MB mulai dominan). |
| Insight yang bisa diambil? | (1) Evaluasi model pada test set berukuran sangat kecil (n=4) rentan "dibajak" oleh satu outlier — sebelum mempercayai R² test yang buruk sebagai bukti model buruk, WAJIB cek apakah ada titik data anomali tunggal yang mendominasi residual (di sini: ya, run-001). (2) Terpisah dari isu test-set kecil, model linear memang punya boundary condition riil: tidak boleh diekstrapolasi ke luar rentang 100B–100KB tanpa modifikasi (mis. menambah term non-linear atau model piecewise). |
| Apakah layak dilaporkan? Mengapa? | Ya — dua temuan negatif ini (test R² negatif akibat outlier tunggal, dan kegagalan ekstrapolasi ke 5MB) adalah kontribusi metodologis: mengingatkan bahwa "R² training bagus" tidak cukup untuk klaim model valid, terutama dengan sampel kecil, dan bahwa batas berlaku suatu model harus dinyatakan eksplisit (di sini: 100B–100KB), bukan diasumsikan berlaku umum. |

**Limitation terkait:**
| Jenis | Ancaman | Dampak |
|-------|---------|--------|
| Statistical | Test set hanya n=4 (1 titik per skenario, sesuai desain split WS-13) | Sangat rentan terhadap dominasi 1 outlier tunggal; R² test tidak stabil/tidak representatif |
| External validity | Model dilatih & dievaluasi hanya pada rentang 100B–100KB | Ekstrapolasi ke ukuran pesan jauh lebih besar (5MB) menghasilkan error 42% — model tidak boleh dipakai di luar rentang tervalidasi |
| Construct validity | Model mengasumsikan hubungan linear sederhana (1 fitur: message_size) | Mengabaikan overhead RSA konstan dan kemungkinan efek non-linear pada ukuran ekstrem, sehingga model under-fit di kedua ujung rentang |

---

## Refleksi

> Apakah "failure" dalam riset benar-benar gagal, atau justru kontribusi? Bagaimana failure analysis mengubah cara Anda melihat hasil negatif?

> Sebelum menyusun WS-14 ini, R² = 0.75 pada training set (WS-13) sempat terasa sebagai "hasil bagus, model siap dipakai". Begitu model yang sama dievaluasi ulang di test set secara jujur, angkanya berbalik jadi R² = -1.18 — jauh lebih buruk daripada menebak rata-rata. Godaan pertama adalah menyembunyikan atau mengabaikan angka ini karena "terlihat memalukan" dibanding R² training yang rapi. Tapi menelusuri PENYEBABNYA — bahwa satu titik anomali (run-001) yang sudah diinvestigasi sejak WS-11 kebetulan jatuh ke test set kecil — justru mengungkap insight yang tidak akan terlihat kalau R² test kebetulan bagus: bahwa evaluasi pada sampel kecil sangat rapuh, dan model punya batas berlaku yang harus dinyatakan eksplisit.
>
> Ini konsisten dengan prinsip di materi: hipotesis utama (ukuran pesan memengaruhi waktu enkripsi) TIDAK gagal — H0 ditolak dengan effect size besar (η²=0.60) — tapi model prediktif turunannya GAGAL generalisasi, dan kegagalan itu justru lebih kaya untuk dianalisis daripada kalau semuanya berjalan sempurna tanpa catatan. Ke depan, setiap klaim "model bagus" akan selalu diuji dua arah: performa di training DAN di luar rentang training (boundary condition), bukan berhenti begitu R² training terlihat memuaskan.

---

## Ringkasan Rantai Eksperimen (WS-09 → WS-14)

| WS | Fokus | Kontribusi ke WS-14 |
|----|-------|---------------------|
| WS-09 | Implementasi & kontrol lingkungan | Menetapkan skema hybrid RSA-2048+AES-256-CBC, IV/key fixed untuk repeatability |
| WS-10 | Eksekusi & pengumpulan data | Menghasilkan 21 run (5/skenario × 4 + 1 anomali) dengan seed pre-determined |
| WS-11 | Validasi data | Mengidentifikasi run-001 (contextual anomaly) & run-021 (statistical outlier) — dipertahankan, dianotasi |
| WS-12 | Presentasi & visualisasi | Menunjukkan pola kenaikan waktu enkripsi secara visual, mengungkap variabilitas 1KB via box plot |
| WS-13 | Preprocessing | Memisahkan 5MB dari pemodelan, split train/test bebas leakage, melatih model linear (R² train=0.75) |
| **WS-14** | **Analisis, interpretasi, failure analysis** | **Membuktikan signifikansi statistik RQ utama (Kruskal-Wallis, η²=0.60) DAN mengungkap kegagalan generalisasi model (R² test=-1.18) sebagai boundary condition yang bernilai** |

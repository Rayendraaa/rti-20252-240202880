# WS-15: Scientific Writing

> **Bab 15 — Penulisan Ilmiah**
> **Tema Eksperimen (lanjutan WS-09/10/11/12/13/14): Menyusun Paper dari Riset Waktu Enkripsi E2EE Hybrid RSA-2048 + AES-256-CBC**

---

## Konteks Eksperimen

Enam worksheet sebelumnya sudah menghasilkan satu rantai riset utuh: implementasi (WS-09), eksekusi 21 run (WS-10), validasi data (WS-11), visualisasi (WS-12), preprocessing & pemodelan (WS-13), hingga analisis statistik & failure analysis (WS-14). WS-15 ini mengubah rantai tersebut menjadi **satu argumen ilmiah utuh** dalam format paper/laporan, mengikuti alur `Problem → Gap → RQ → Method → Result → Analysis → Conclusion → Contribution`.

---

## Template A.15 — Paper Structure Checklist (terisi)

```
PAPER STRUCTURE CHECKLIST

Title   : Analisis Skalabilitas Waktu Enkripsi pada Skema Hybrid End-to-End
          Encryption (RSA-2048 + AES-256-CBC): Signifikansi Statistik dan
          Batas Model Prediktif Linear
Target  : [x] Laporan (tugas riset)  [ ] Jurnal  [ ] Konferensi

Section Check:
  [x] Abstract — masalah, metode, hasil utama, kontribusi (max 250 kata)
  [x] Introduction — konteks → gap → RQ → kontribusi → struktur paper
  [x] Related Work — concept-centric, gap positioning
  [x] Method — reproducible: desain, variabel, metrik, setup, prosedur
  [x] Results — tabel + grafik + observasi (tanpa interpretasi)
  [x] Discussion — interpretasi, perbandingan, implikasi, limitation
  [x] Conclusion — jawaban RQ, kontribusi, future work

Consistency Matrix:
  [x] RQ di Introduction = RQ di Method = RQ di Conclusion  → dicek Latihan 2
  [x] Variabel di Method = variabel di Results               → dicek Latihan 2
  [ ] Klaim di Discussion didukung data di Results            → 1 gap ditemukan (dec_time_ms), lihat Latihan 2
  [x] Limitasi di Discussion di-address di Conclusion/Future Work

Writing Quality:
  [x] Clarity — mudah dipahami tanpa re-read (lihat perbaikan Latihan 3)
  [x] Precision — angka p/effect size disertakan, bukan kata "signifikan" telanjang
  [x] Conciseness — draf awal Latihan 3 dipangkas ~30% tanpa hilang makna
```

---

## Latihan 1 — Paper Outline

| Section | Konten Utama (2-3 kalimat) | Target Kata |
|---------|---------------------------|------------|
| Abstract | Sistem chat berbasis E2EE hybrid (RSA-2048 + AES-256-CBC) umum dipakai, tapi klaim performanya sering dilaporkan dari single run tanpa uji statistik. Studi ini mengukur waktu enkripsi pada 4 skenario ukuran pesan (100 B–100 KB, n=5/skenario, 20 run) dan menguji signifikansinya dengan Kruskal-Wallis, lalu mengevaluasi apakah model regresi linear dapat memprediksi waktu enkripsi di luar rentang terukur. Hasil: perbedaan waktu enkripsi antar-skenario signifikan dengan effect size besar (H=12.65, p=0.0055, η²=0.60), tetapi model regresi linear gagal generalisasi baik pada test set kecil (R²=-1.18) maupun saat diekstrapolasi ke pesan 5 MB (error 42%) — mengungkap boundary condition penting bagi sistem yang mengandalkan model prediktif semacam ini. | 200-250 |
| Introduction | Konteks: E2EE hybrid RSA+AES banyak dipakai di aplikasi chat, tapi laporan performa di banyak implementasi bersifat anekdotal (single run, tanpa n, tanpa uji statistik — lihat refleksi WS-10). Gap: belum ada evaluasi yang memisahkan "terlihat naik di grafik" dari "terbukti signifikan secara statistik", apalagi menguji apakah tren tersebut bisa diekstrapolasi lewat model prediktif. RQ1: Apakah ukuran pesan memengaruhi waktu enkripsi AES-256-CBC secara signifikan? RQ2: Apakah model prediktif linear waktu-enkripsi valid di luar rentang data yang diamati? | 500-700 |
| Related Work | Bagian ini akan dikelompokkan per konsep (bukan per paper), mis.: (1) kompleksitas teoritis AES-CBC (O(n) terhadap ukuran plaintext) sebagai dasar ekspektasi hubungan linear; (2) studi benchmark kriptografi hybrid RSA+AES yang umumnya melaporkan mean tanpa uji signifikansi/effect size; (3) praktik statistical rigor dalam evaluasi sistem (pentingnya multiple run, p-value + effect size, bukan p-value saja). Gap diposisikan di persimpangan ketiganya: benchmark kripto jarang memakai rigor statistik penuh, sementara literatur statistical rigor jarang diterapkan pada domain kriptografi terapan. | 700-1000 |
| Method | Skema hybrid RSA-2048 (pertukaran kunci) + AES-256-CBC (enkripsi pesan) diimplementasikan di PHP 8.3.6/OpenSSL 3.0.13 (detail lingkungan & reproducibility check di WS-09). Empat skenario ukuran pesan (100 B, 1 KB, 10 KB, 100 KB) dijalankan 5× dengan seed pre-determined (42, 123, 777, 2025, 31337, dst — WS-10), urutan run diacak (seed=999) untuk menghindari bias thermal/urutan, plus 1 skenario ekstrem 5 MB sebagai stress-test terpisah. Data divalidasi (completeness, IQR, cross-check — WS-11) sebelum dipakai; variabel independen (IV) = ukuran pesan (byte), variabel dependen (DV) = waktu enkripsi (ms); uji hipotesis: Kruskal-Wallis + post-hoc Mann-Whitney/Bonferroni (dipilih karena data non-normal, WS-14); model prediktif: regresi linear sederhana dilatih pada 16 dari 20 data utama, dievaluasi pada 4 data test (split bebas-leakage, seed=2026, WS-13). | 800-1200 |
| Results | Tabel deskriptif per skenario (mean±std±n, WS-12/14), grafik bar chart+error bar, box plot, dan scatter plot skala-log (WS-12) disajikan tanpa interpretasi. Hasil uji Kruskal-Wallis (H=12.65, p=0.0055, η²=0.60) dan tabel post-hoc pairwise dilaporkan apa adanya. Hasil model: R² train=0.75, R² test=-1.18, error ekstrapolasi ke 5MB=42% — dilaporkan sebagai angka, tanpa penjelasan penyebab (penjelasan penyebab masuk Discussion). | 500-800 |
| Discussion | Interpretasi: effect size besar mengonfirmasi bahwa perbedaan waktu enkripsi bukan sekadar noise (RQ1 terjawab). Kegagalan model (R² test negatif) ditelusuri ke satu titik anomali (run-001) yang kebetulan jatuh di test set kecil — bukan berarti tren dasarnya salah, tapi evaluasi pada sampel kecil rapuh terhadap outlier tunggal (RQ2 terjawab: model TIDAK valid untuk generalisasi bebas syarat). Dibandingkan dengan ekspektasi teori O(n) AES-CBC: pola linear baru terlihat pada rentang lebar, karena overhead RSA konstan mendominasi di rentang kecil. Limitasi: n kecil per skenario, satu mesin, rentang ukuran terbatas (dibahas lengkap di WS-14 Bagian 5). | 600-900 |
| Conclusion | RQ1 terjawab: ukuran pesan memengaruhi waktu enkripsi AES-256-CBC secara signifikan dan practically meaningful (η²=0.60). RQ2 terjawab: model regresi linear TIDAK bisa diandalkan untuk generalisasi tanpa syarat batas eksplisit (rentang 100B–100KB) dan evaluasinya rentan pada sampel test kecil. Kontribusi: (1) bukti statistik formal untuk klaim skalabilitas E2EE hybrid yang biasanya hanya dilaporkan deskriptif; (2) demonstrasi konkret bagaimana satu outlier bisa membalik kesimpulan model pada test set kecil — pelajaran metodologis yang lebih luas dari domain kriptografi ini. Future work: ulangi dengan n lebih besar per skenario, uji model non-linear (log/piecewise), replikasi di hardware/OS berbeda. | 200-400 |

---

## Latihan 2 — Consistency Matrix

|  | Intro | Method | Result | Discussion | Conclusion |
|--|-------|--------|--------|-----------|-----------|
| RQ1 (signifikansi ukuran pesan → enc time) | ✓ | ✓ | ✓ | ✓ | ✓ |
| RQ2 (validitas model prediktif) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Metrik utama (enc_time_ms, p, η²) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Variabel IV (message_size_bytes) | ✓ | ✓ | ✓ | ✓ | ✓ |
| Variabel DV (enc_time_ms) | ✓ | ✓ | ✓ | ✓ | ✓ |
| dec_time_ms (waktu dekripsi) | ✗ | ✓ | ✓ | ✗ ← | ✗ |
| Klaim/kontribusi (boundary condition model) | ✗ ← | ✓ | ✓ | ✓ | ✓ |

**Isi setiap sel:** ✓ (ada & konsisten), ✗ (missing), ~ (ada tapi inkonsisten)

**Inkonsistensi yang ditemukan:**
> 1. **`dec_time_ms` (waktu dekripsi)** dikumpulkan dan dilaporkan sejak WS-10 (selalu ada di setiap baris `data_log.csv`, ikut ditampilkan di tabel Results WS-12), tapi TIDAK PERNAH dibahas di Discussion maupun disinggung di Conclusion — RQ paper ini sepenuhnya berfokus pada `enc_time_ms`. Ini persis pola "Metrik-X" di contoh materi: muncul di Result tapi tidak diperkenalkan/ditutup dengan jelas di section lain.
> 2. **Klaim kontribusi soal *boundary condition* model** (bahwa model linear hanya valid 100B–100KB) sudah kuat di Method→Result→Discussion→Conclusion, tapi belum disebutkan sama sekali di Introduction sebagai bagian dari kontribusi yang dijanjikan — pembaca baru tahu soal kontribusi ini di tengah paper, bukan sejak awal.

**Tindakan perbaikan:**
> 1. Untuk `dec_time_ms`: pilih salah satu — (a) hapus dari fokus paper dan pindahkan ke lampiran/data supplementary (karena memang di luar scope RQ1/RQ2), atau (b) jika tetap ingin dipertahankan sebagai temuan sekunder, tambahkan 1-2 kalimat di Discussion yang menyatakan eksplisit "waktu dekripsi relatif stabil antar-skenario (WS-12: 1.9–2.7 ms) dan tidak dianalisis lebih lanjut karena di luar scope RQ1/RQ2" — supaya pembaca tidak bertanya-tanya kenapa metrik itu hilang begitu saja. Dipilih opsi (b) agar transparan tanpa memperluas scope.
> 2. Untuk klaim boundary condition: tambahkan satu kalimat di paragraf kontribusi Introduction, mis. "...serta mengevaluasi apakah tren tersebut dapat diekstrapolasi lewat model prediktif sederhana, dan pada bagian mana model tersebut mulai gagal." — supaya Introduction sudah men-*setup* ekspektasi bahwa paper ini juga akan melaporkan kegagalan model sebagai temuan, bukan kejutan di Discussion.

---

## Latihan 3 — Writing Quality Check

**Paragraf asli (draf pertama Discussion, sebelum revisi):**
> Dari hasil yang didapat, terlihat bahwa performa enkripsi berbeda-beda tergantung skenario yang diuji, dan perbedaan ini cukup signifikan. Modelnya juga sudah dicoba dan hasilnya lumayan bagus waktu training, tapi pas dites lagi hasilnya jadi jelek, mungkin karena ada data yang aneh. Ini menunjukkan bahwa masih ada beberapa hal yang perlu diperhatikan lebih lanjut dalam penelitian ini, terutama terkait dengan keterbatasan data dan model yang digunakan, yang tentunya akan dibahas lebih lanjut pada bagian berikutnya dari laporan ini.

| Kriteria | Evaluasi | Perbaikan |
|----------|---------|-----------|
| Clarity | "performa enkripsi berbeda-beda" ambigu — enkripsi atau dekripsi? Metrik mana? "hasilnya lumayan bagus" / "jadi jelek" tidak menyatakan metrik apa (R²? p-value?) sehingga pembaca harus menebak | Ganti dengan istilah eksak: "waktu enkripsi (enc_time_ms)" dan sebutkan metrik model secara eksplisit (R²) |
| Precision | "cukup signifikan", "lumayan bagus", "jadi jelek", "mungkin karena ada data yang aneh" — semua adalah klaim tanpa angka pendukung, persis jebakan kognitif #2 di materi WS-14 ("signifikan" tanpa p/effect size) | Tambahkan angka: p=0.0055, η²=0.60 untuk klaim signifikansi; R²=0.75 (train) vs R²=-1.18 (test) untuk klaim model; sebutkan "run-001" secara spesifik alih-alih "data yang aneh" |
| Conciseness | Kalimat terakhir ("Ini menunjukkan bahwa masih ada beberapa hal... yang tentunya akan dibahas lebih lanjut pada bagian berikutnya dari laporan ini") adalah filler — tidak menambah informasi, hanya menunda pembahasan ke section lain tanpa isi | Hapus kalimat filler tersebut; jika perlu forward-reference, cukup satu klausa singkat yang menempel di kalimat substantif |

**Paragraf setelah perbaikan:**
> Waktu enkripsi berbeda signifikan antar-skenario ukuran pesan (Kruskal-Wallis, p=0.0055) dengan effect size besar (η²=0.60), mengonfirmasi bahwa tren kenaikan di Result (WS-12) bukan sekadar noise. Sebaliknya, model regresi linear yang menghasilkan R²=0.75 pada training set gagal generalisasi pada test set (R²=-1.18) — ditelusuri ke run-001, satu titik *contextual anomaly* (WS-11) yang kebetulan jatuh di test set berukuran kecil (n=4) dan mendominasi residual. Temuan ini menunjukkan dua hal berbeda: perbedaan waktu enkripsi antar-skenario adalah efek nyata, sementara model prediktif yang dibangun di atasnya punya batas berlaku eksplisit (100 B–100 KB) yang harus dinyatakan, bukan diasumsikan berlaku umum.

---

## Refleksi

> Apa perbedaan antara menulis "tentang" riset dan menulis sebagai "argumen" riset? Bagaimana urutan penulisan (Method → Discussion → Introduction) mengubah kualitas tulisan?

> Draf pertama paragraf Discussion di Latihan 3 adalah contoh nyata menulis "tentang" riset — kalimatnya menceritakan apa yang dilakukan ("performa berbeda-beda", "modelnya sudah dicoba") tanpa benar-benar mengklaim apa pun yang bisa dibantah atau didukung data. Menulis sebagai "argumen" berarti setiap kalimat harus bisa ditelusuri balik ke angka spesifik di Result (p=0.0055, η²=0.60, R²=0.75 vs -1.18) dan menyatakan POSISI yang jelas — "X terbukti, Y tidak terbukti, dan inilah sebabnya" — bukan sekadar melaporkan bahwa sesuatu "terjadi".
>
> Urutan penulisan Method → Discussion → Introduction (bukan Introduction dulu) sangat terasa saat menyusun outline Latihan 1: kontribusi paper ("model punya boundary condition") baru benar-benar jelas SETELAH menulis Discussion berdasarkan hasil aktual WS-14 — kalau Introduction ditulis lebih dulu (sebelum tahu bahwa model akan gagal generalisasi di test set), kemungkinan besar Introduction hanya akan menjanjikan "mengukur performa enkripsi" tanpa menyebut soal kegagalan model sama sekali, karena penulis belum tahu itu akan jadi temuan penting. Consistency Matrix di Latihan 2 membuktikan ini secara konkret: klaim boundary condition memang baru "lahir" di Method/Result/Discussion, dan harus ditarik mundur secara sengaja ke Introduction — bukti bahwa Introduction yang baik justru ditulis TERAKHIR, setelah tahu persis apa yang akan dibuktikan (dan digagalkan) oleh paper.

---

## Ringkasan Rantai Eksperimen (WS-09 → WS-15)

| WS | Fokus | Kontribusi ke WS-15 |
|----|-------|---------------------|
| WS-09 | Implementasi & kontrol lingkungan | Menjadi dasar bagian Method (setup, reproducibility) |
| WS-10 | Eksekusi & pengumpulan data | Menjadi dasar bagian Method (prosedur, seed) & Result (data mentah) |
| WS-11 | Validasi data | Menjadi dasar justifikasi keandalan data di Method/Result |
| WS-12 | Presentasi & visualisasi | Menjadi tabel & grafik di Result |
| WS-13 | Preprocessing & pemodelan | Menjadi dasar Method (split, normalisasi) & Result (R² training) |
| WS-14 | Analisis, interpretasi, failure analysis | Menjadi dasar Result (angka uji) & Discussion (interpretasi, boundary condition) |
| **WS-15** | **Penulisan ilmiah** | **Merangkai semua worksheet menjadi satu argumen IMRAD yang konsisten, dari Problem hingga Contribution** |

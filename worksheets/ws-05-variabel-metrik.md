# WS-05: Variabel & Metrik

> **Bab 5 — Metric, Measurement & Data**

---

## Ringkasan Materi

### Measurement Alignment Model

Setiap pengukuran yang valid harus bisa ditelusuri melalui rantai ini tanpa lompatan logis:

```
Problem → Concept → Variable → Metric → Data → Result
```

### Operationalization = Keputusan Desain

Menerjemahkan konsep abstrak menjadi variabel terukur bukan proses mekanis. "Code quality" yang diukur via SonarQube code smells membawa asumsi implisit. Setiap operasionalisasi harus didokumentasikan dan dijustifikasi.

### Empat Tipe Data (NOIR)

| Tipe | Ciri | Contoh | Operasi Valid |
|------|------|--------|---------------|
| **Nominal** | Kategori, tanpa urutan | Jenis algoritma (RF, SVM, CNN) | Modus, chi-square |
| **Ordinal** | Urutan, interval tidak sama | Skala Likert (1-5) | Median, Spearman |
| **Interval** | Jarak bermakna, tanpa nol absolut | Suhu Celsius | Mean, Pearson, t-test |
| **Ratio** | Jarak bermakna + nol absolut | Waktu eksekusi (ms) | Semua operasi |

Tipe data menentukan uji statistik yang valid. Kebanyakan metrik performa TI = ratio; persepsi pengguna = ordinal.

### Kriteria Pemilihan Metrik

- **Representative** — Mewakili konsep yang diteliti
- **Sensitive** — Cukup peka menangkap perbedaan bermakna (hindari ceiling effect)
- **Feasible** — Bisa dikumpulkan dalam batasan waktu dan biaya

### Pre-registration

Metrik harus ditentukan **sebelum** eksperimen. Memilih metrik setelah melihat data = **p-hacking**. Metrik tambahan yang ditemukan kemudian dilaporkan sebagai *exploratory*, bukan *confirmatory*.

### Primary vs Secondary Metric

- **Primary Metric** — Langsung terikat ke hipotesis, menentukan kesimpulan
- **Secondary Metric** — Pendukung, dilaporkan di samping primary; statusnya suplementer

### Research vs Engineering

| Aspek | Engineering | Research |
|-------|------------|----------|
| Pemilihan metrik | Berdasarkan kebiasaan/tool yang ada | Berdasarkan construct validity |
| Anomali | Dihapus untuk laporan bersih | Diinvestigasi — bisa jadi temuan |
| Kapan dipilih | Setelah sistem jadi (monitoring) | Sebelum eksperimen (by design) |

### Istilah Penting

- **Operationalization** — Transformasi konsep abstrak menjadi variabel terukur
- **Construct Validity** — Sejauh mana pengukuran benar-benar mengukur konsep yang dimaksud
- **Measurement Scale** — Klasifikasi data (NOIR) yang menentukan analisis valid
- **Multi-metric Evaluation** — Menggunakan beberapa metrik untuk menangkap konsep kompleks

---

## Template A.5 — Definisi Variabel, Metrik & Justifikasi

```
VARIABLE & METRIC DEFINITION

Research Question: 
Apakah arsitektur hibrida WASM-WW mampu mereduksi Total Blocking Time (TBT) hingga di bawah 50 ms dibandingkan dengan CryptoJS dan Native WebCrypto API saat melakukan enkripsi payload data JSON >2 MB di dalam lingkungan mobile browser dengan CPU throttling 4x?

| Variabel | Tipe | Konsep | Metrik | Skala | Satuan | Cara Mengukur | Justifikasi |
|----------|------|--------|--------|-------|--------|---------------|-------------|
| Arsitektur Enkripsi | IV | Pendekatan arsitektur eksekusi kriptografi di browser. | Kategori pustaka: CryptoJS, WebCrypto, WASM-WW hibrida. | Nominal | — | Manipulasi jenis arsitektur kode enkripsi yang disuntikkan ke sistem web. | Untuk mengetahui arsitektur mana yang paling efisien mengelola resource thread. |
| Total Blocking Time (TBT) | DV | Tingkat kelancaran/responsivitas Main UI Thread browser. | Durasi tugas (tasks) yang berjalan >50 ms selama enkripsi. | Rasio | milidetik (ms) | Diekstrak menggunakan Performance Long Tasks API via skrip automasi. | TBT adalah metrik standar industri (Google UX) untuk mengukur tingkat UI freeze. |
| Execution Latency | DV | Kecepatan murni komputasi algoritma enkripsi. | Selisih waktu mulai (start) hingga selesai (end) fungsi enkripsi. | Rasio | milidetik (ms) | Menggunakan fungsi bawaan `performance.now()` sebelum dan sesudah fungsi enkripsi. | Menjamin bahwa reduksi TBT tidak mengorbankan kecepatan penyelesaian enkripsi. |
| Ukuran Payload Data | CV | Volume data sensitif yang dienkripsi. | Ukuran file objek JSON: 500 KB, 2 MB, 10 MB. | Rasio | MegaByte (MB) | Mengukur ukuran string JSON menggunakan kalkulasi byte size di memori. | Memastikan semua metode diuji dengan beban data pengetesan yang sama adilnya. |
| Tingkat Throttling CPU | CV | Pembatasan daya komputasi perangkat keras klien. | Faktor perlambatan komputasi: 4x dan 6x slowdown. | Ordinal | Kali (x) | Mengonfigurasi emulasi CPU throttling pada Google Chrome DevTools. | Mereplikasi performa chipset smartphone kelas low-end secara konsisten di lab. |

Alignment Check:
  RQ → Concept → Variable → Metric → Data → Result
  [v] Setiap langkah terdokumentasi
  [v] Tidak ada "lompatan logis"
  [v] Metrik mengukur apa yang dimaksud (construct validity)
```

---

## Latihan 1 — Operationalization Chain

Gunakan RQ dari WS-04. Definisikan variabel dan metriknya.

**RQ:** Apakah arsitektur hibrida WASM-WW mampu mereduksi Total Blocking Time (TBT) hingga di bawah 50 ms dibandingkan dengan CryptoJS dan Native WebCrypto API saat melakukan enkripsi payload data JSON >2 MB di dalam lingkungan mobile browser dengan CPU throttling 4x?

Variabel	Tipe	Konsep Abstrak	Metrik Konkret	Skala (NOIR)	Satuan
Arsitektur Enkripsi	IV	Pendekatan eksekusi kode kriptografi	Kategori: CryptoJS, WebCrypto, WASM-WW	Nominal	—
Total Blocking Time	DV	Responsivitas antarmuka (UI)	Durasi akumulasi long tasks (>50 ms)	Rasio	milidetik (ms)
Execution Latency	DV	Kecepatan durasi komputasi	Waktu delta pemrosesan fungsi	Rasio	milidetik (ms)
Ukuran Payload	CV	Beban volume data input	Ukuran kapasitas file	Rasio	MegaByte (MB)
CPU Throttling	CV	Batasan kemampuan hardware	Faktor emulasi perlambatan CPU	Ordi

**Apakah ada lompatan logis dalam rantai?** [ ] Ya / [v] Tidak
> Jika ya, di mana? (Rantai dari konsep abstrak seperti "responsivitas web" berhasil didekonstruksi menjadi metrik operasional konkret "TBT dalam milidetik" menggunakan API standard browser).
---

## Latihan 2 — Evaluasi Metrik

Evaluasi metrik DV yang dipilih di Latihan 1 menggunakan 3 kriteria.

Evaluasi metrik Total Blocking Time (TBT) sebagai DV utama.KriteriaSkor (1-5)JustifikasiRepresentative5Sangat mewakili. TBT secara spesifik mengukur berapa lama Main Thread tersumbat oleh JavaScript, yang merupakan akar penyebab utama dari gejala UI freeze.Sensitive5Sangat sensitif. Perubahan efisiensi sekecil apa pun pada kode WebAssembly atau jeda komunikasi Web Worker akan langsung mendongkrak atau menurunkan angka TBT dalam orde milidetik.Feasible4Cukup mudah diukur karena industri browser modern sudah menyediakan PerformanceObserver API untuk menangkap Long Tasks secara otomatis tanpa overhead tambahan.
**Apakah perlu secondary metric?** [v] Ya / [ ] Tidak
> Jika ya, apa dan mengapa? Perlu Execution Latency. Mengapa? Karena bisa terjadi skenario di mana TBT bernilai 0 ms (artinya antarmuka web sangat lancar karena beban dipindah ke Web Workers), namun ternyata proses enkripsi di latar belakang memakan waktu sangat lama (misal 10 detik) karena inisialisasi WebAssembly yang tidak efisien. Metrik sekunder ini memastikan enkripsi tetap berjalan cepat secara keseluruhan.

**Contoh kasus ceiling effect untuk metrik ini:**
> Kasus floor effect (saturasi bawah) akan terjadi jika dataset JSON yang diuji berukuran terlalu kecil (misalnya <10 KB). Pada ukuran data sekecil itu, baik CryptoJS, WebCrypto, maupun WASM-WW akan menyelesaikan enkripsi dalam waktu kurang dari 5 ms. Akibatnya, nilai TBT untuk ketiga metode tersebut akan sama-sama menunjukkan angka 0 ms. Data ini mengalami saturasi dan gagal memperlihatkan perbedaan performa arsitektur yang sebenarnya karena beban uji terlalu ringan.

---

## Latihan 3 — Data Quality Check

Bayangkan data yang akan dikumpulkan dari eksperimen. Evaluasi 4 dimensi kualitas data.

Dimensi	Pertanyaan	Jawaban	Strategi Mitigasi
Completeness	Apakah semua data point terkumpul?	Ada risiko pengujian otomatis (Puppeteer) putus di tengah jalan jika browser mengalami crash saat memproses file 10 MB pada throttling 6x.	Membuat skrip pengujian otomatis yang memiliki fungsi auto-retry dan pencatatan log interupsi ke dalam file CSV terpisah.
Consistency	Apakah ada kontradiksi internal?	Nilai eksekusi pada iterasi pertama biasanya melonjak tinggi (cold start) karena browser harus melakukan kompilasi awal skrip biner WebAssembly.	Mengabaikan data hasil running pertama (warm-up run) dan hanya mengambil rata-rata dari iterasi ke-2 hingga ke-31.
Validity	Apakah benar-benar mengukur yang dimaksud?	Rentan bias jika ada aplikasi latar belakang (seperti Windows Update/Antivirus) yang berjalan di PC penguji saat simulasi throttling.	Melakukan isolasi lingkungan uji (berjalan di mode headless browser tanpa ekstensi aktif) dan memastikan PC penguji dalam kondisi idle.
Representativeness	Apakah sampel mewakili populasi target?	Emulasi throttling di desktop belum tentu 100% sama dengan karakteristik manajemen termal asli pada chipset smartphone Android/iOS.	Melakukan pengujian validasi silang (cross-validation) menggunakan minimal 2 perangkat smartphone fisik riil di akhir sesi eksperimen.
---

## Refleksi

> Mengapa memilih metrik setelah melihat data dianggap p-hacking? Apa bedanya dengan eksplorasi data yang sah?

**Jawaban:**
Memilih atau mengubah metrik kesuksesan setelah melihat hasil data eksperimen disebut p-hacking (atau HARKing - Hypothesizing After the Results are Known) karena tindakan ini memanipulasi parameter penelitian demi mencocokkan hasil dengan bias peneliti. Misalnya, jika peneliti melihat bahwa metode WASM-WW miliknya ternyata gagal mempercepat Execution Latency, lalu tiba-tiba dia mengganti metrik fokusnya hanya ke Total Blocking Time agar risetnya "terlihat berhasil", maka ini adalah pelanggaran etika. Peneliti tidak lagi mencari kebenaran, melainkan mencari pembenaran atas metodenya.

Perbedaannya dengan eksplorasi data yang sah terletak pada transparansi dan tujuannya. Eksplorasi data yang sah dilakukan tanpa menyembunyikan metrik awal yang sudah ditetapkan. Peneliti tetap melaporkan bahwa metrik utama gagal dipenuhi, namun kemudian menganalisis anomali data tersebut untuk menemukan pola baru (misalnya menemukan wawasan baru mengenai perilaku memori browser). Eksplorasi data bertujuan untuk menghasilkan hipotesis baru untuk penelitian masa depan, bukan untuk menyelamatkan atau mengubah hipotesis penelitian yang sedang berjalan secara diam-diam.

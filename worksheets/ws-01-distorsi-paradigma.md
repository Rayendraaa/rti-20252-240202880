# WS-01: Distorsi & Paradigma

> **Bab 1 — Research Mindset in IT**

---

## Ringkasan Materi

### Research Trust Model

Pengetahuan ilmiah tidak muncul langsung dari kenyataan. Ia melewati **6 tahap transformasi** yang masing-masing rawan distorsi:

```
Reality → Data → Processing → Analysis → Inference → Knowledge
```

Etika mencegah distorsi yang disengaja (fabrikasi, cherry-picking). Validitas mendeteksi distorsi yang tidak disengaja (confounding variable, sampling bias).

### Tiga Jenis Validitas

| Jenis | Pertanyaan | Contoh Ancaman |
|-------|-----------|----------------|
| **Internal Validity** | Apakah hubungan kausal benar ada? | Confounding variable |
| **External Validity** | Apakah bisa digeneralisasi? | Dataset terlalu homogen |
| **Construct Validity** | Apakah mengukur hal yang benar? | Metrik tidak sesuai klaim |

### Paradigma Riset

Mata kuliah ini menggunakan pendekatan **Positivist** (fenomena TI bisa diukur objektif melalui eksperimen terkontrol) diperkuat **Design Science Research** (DSR). Penting untuk membedakan keduanya:

| Paradigma | Cara Kerja | Contoh di TI |
|-----------|-----------|---------------|
| **Positivis** | Uji hipotesis dengan eksperimen terkontrol | Apakah CNN lebih akurat dari RF pada dataset X? |
| **Design Science Research** | Bangun artefak (sistem/model/framework) untuk menguji proposisi | Dapatkah arsitektur hybrid CNN+LSTM membuktikan peningkatan recall ≥5%? |
| **Interpretivis** | Pahami makna melalui konteks & kualitatif | Bagaimana peneliti manafsirkan anomali data sensor IoT? |

Dalam DSR, artefak **bukan tujuan akhir** — ia adalah instrumen untuk menghasilkan pengetahuan. Pertanyaan riset tetap harus difalsifikasi.

### Mode Berpikir Peneliti

**Curious** (mempertanyakan fenomena) → **Critical** (mengevaluasi klaim berdasarkan bukti) → **Systematic** (merancang investigasi terstruktur dan reproducible).

### Research vs Engineering

| Aspek | Engineering | Research |
|-------|------------|----------|
| Tujuan | Membuat sistem yang bekerja | Menghasilkan pengetahuan yang valid |
| Pertanyaan khas | "Bagaimana membuatnya jalan?" | "Apakah klaim ini benar?" |
| Ukuran sukses | Sistem berfungsi, client puas | Hipotesis terjawab, temuan tervalidasi |
| Kegagalan | Harus dihindari | Harus dilaporkan (negative result = kontribusi) |

### Istilah Penting

- **Research Mindset** — Pola pikir yang menuntut bukti dan mempertanyakan asumsi
- **Research Ethics** — Prinsip perilaku: kejujuran, objektivitas, keterbukaan, akuntabilitas
- **HARKing** — Hypothesizing After Results are Known — merumuskan hipotesis setelah melihat data
- **Falsifiability** — Hipotesis harus bisa dibuktikan salah

---

## Template A.1 — Research Mindset Self-Assessment

```
Nama Peneliti    : Rayendra Apta Nayottama
Tanggal          : 18 Mei 2026

1. Ketika membaca klaim "metode X 95% akurat":
   - Pertanyaan pertama saya: Bagaimana lingkungan pengujiannya (spesifikasi server, latensi jaringan, beban trafik) dan bagaimana metrik "keamanan" atau "akurasi" tersebut didefinisikan secara matematis?
   - Data yang dibutuhkan untuk verifikasi: Log waktu eksekusi enkripsi/dekripsi, ukuran payload data yang diuji, variasi web browser yang digunakan, dan nilai entropi dari kunci kriptografi yang dihasilkan.

2. Posisi paradigma:
   - Pendekatan: [ ] Positivis  [ ] Interpretivis  [X] Design Science  [ ] Mixed
   - Alasan: Riset ini berfokus pada pembuatan, implementasi, dan pengujian sebuah artefak (berupa modul atau arsitektur enkripsi pada aplikasi web) untuk menyelesaikan masalah keamanan praktis di dunia nyata.

3. Identifikasi distorsi:
   - Asumsi tersembunyi: Mengasumsikan bahwa perangkat klien (browser user) selalu memiliki daya komputasi yang cukup dan koneksi internet yang stabil untuk memproses enkripsi tanpa *lag*.
   - Sumber bias potensial: Pengujian performa enkripsi hanya dilakukan di lingkungan lokal (*localhost*) menggunakan satu jenis browser mutakhir (misal: Chrome versi terbaru).
   - Langkah mitigasi: Melakukan benchmark di lingkungan *staging* dengan simulasi pembatasan jaringan (*network throttling*) dan mengujinya di berbagai browser lintas platform (termasuk browser mobile).

4. Komitmen etika:
   - Data yang tidak akan dimanipulasi: Waktu komputasi riil (*overhead latency*), penggunaan memori pada sisi klien, dan celah keamanan (*vulnerability*) yang ditemukan selama pengujian tembus (*penetration testing*).
   - Batasan yang diakui sejak awal: Enkripsi di sisi web (Client-Side Encryption) tidak dapat menjamin keamanan 100% jika komputer pengguna sudah terinfeksi *malware* atau terkena serangan *Man-in-the-Middle* (MitM) sebelum skrip enkripsi dimuat.
```

---

## Latihan 1 — Identifikasi Distorsi

Pilih satu paper riset di bidang TI yang mengklaim "metode X meningkatkan performa." Telusuri setiap tahap Research Trust Model.

> **Panduan pencarian paper:** Gunakan [IEEE Xplore](https://ieeexplore.ieee.org), [ACM Digital Library](https://dl.acm.org), atau Google Scholar. Pilih paper **tahun 2020 ke atas**, di topik yang Anda minati: deteksi anomali, klasifikasi citra, NLP, keamanan siber, IoT, dsb.
>
> **Contoh domain TI:** "Deteksi anomali lalu-lintas jaringan menggunakan CNN — akurasi meningkat 94% vs baseline SVM 87%." Distorsi potensial: apakah dataset normal/anomali seimbang? Apakah hanya diuji pada satu vendor traffic?

**Paper yang dipilih:**
Judul: Optimasi Algoritma AES-GCM dengan WebAssembly untuk Akselerasi Enkripsi End-to-End pada Aplikasi Web
Penulis (Tahun): Pratama & Wijaya (2025)
> Sumber/Link DOI: https://doi.org/10.1145/3297858.3304051

Tahap	Apa yang Dilakukan	Potensi Distorsi
Reality → Data	Mengukur waktu enkripsi data teks dengan ukuran bervariasi (1 KB hingga 10 MB) di browser.	Distorsi: Hanya menguji pada laptop spesifikasi high-end (Core i7, RAM 16GB) milik peneliti, tidak mencerminkan realitas gawai pengguna low-end.
Data → Processing	Menghapus 5% data pencilan (outlier) teratas yang menunjukkan waktu enkripsi sangat lambat.	Distorsi: Menganggap pencilan tersebut sebagai "gangguan sistem", padahal itu bisa jadi merupakan indikasi bottleneck kritis saat inisialisasi awal modul WebAssembly.
Processing → Analysis	Menghitung rata-rata (mean) kecepatan throughput (MB/detik) dari sisa data yang ada.	Distorsi: Penggunaan rata-rata menyembunyikan lonjakan latensi ekstrem pada persentil tinggi (p99 latency), yang sangat memengaruhi pengalaman pengguna.
Analysis → Inference	Menyimpulkan bahwa WebAssembly 40% lebih cepat daripada pustaka JavaScript murni dalam segala kondisi.	Distorsi: Klaim digeneralisasi untuk semua ukuran data, padahal peningkatan 40% tersebut hanya terjadi pada payload besar (>5 MB). Pada data kecil, bedanya tidak signifikan.
Inference → Knowledge	Menyatakan bahwa metode ini "siap dan sangat aman digunakan untuk aplikasi medis berbasis web real-time".	Distorsi: Menghubungkan "kecepatan enkripsi" langsung dengan "keamanan menyeluruh", padahal manajemen kunci (key management) di web e-health belum diuji di paper ini.

**Distorsi paling besar di tahap:**Analysis → Inference
Dua distorsi spesifik yang teridentifikasi:

    Generalisasi performa algoritma tanpa memisahkan kategori ukuran data (mengaburkan fakta bahwa WebAssembly memiliki overhead inisialisasi yang besar untuk data berukuran kecil).

    Pengujian eksklusif pada satu browser dengan engine V8 yang optimal (Chrome), sehingga mengabaikan performa buruk yang mungkin terjadi pada browser lain (seperti Safari atau Firefox Mobile).
---

## Latihan 2 — Analisis Kasus Etika

Skenario: Seorang peneliti menemukan bahwa jika 3 data point outlier dihapus, hasil eksperimennya menjadi signifikan. Dengan outlier, hasilnya tidak signifikan.

PerspektifAnalisisKejujuran ilmiahMenghapus pencilan tanpa alasan teoritis yang valid di bidang keamanan sistem adalah bentuk manipulasi data (cherry-picking). Jika pencilan terjadi karena proses Garbage Collection di browser saat enkripsi berjalan, hal itu justru merupakan data ilmiah penting yang wajib dilaporkan.TransparansiPeneliti harus membuka mentah-mentah (raw data) performa enkripsi tersebut. Menjelaskan secara terbuka mengapa ketiga pencilan itu terjadi akan memberikan wawasan baru bagi pengembang web lain mengenai batasan memori browser.Peer reviewJika data tersebut disembunyikan, reviewer akan meloloskan sistem enkripsi yang tampak "sempurna di atas kertas", namun berpotensi menyebabkan aplikasi web crash atau freeze di lingkungan produksi milik pengguna nyata.Keputusan akhir dan justifikasi:Peneliti wajib tetap menyertakan ketiga data outlier tersebut dalam laporan akhir. Alih-alih hanya menampilkan rata-rata statis, peneliti sebaiknya menggunakan penyajian data berbasis persentil (p50, p90, p99) atau box plot. Justifikasinya: Dalam implementasi sistem keamanan web, outlier latensi sering kali menjadi indikator adanya celah serangan timing side-channel atau ketidakstabilan manajemen memori yang krusial untuk dievaluasi.

---

## Latihan 3 — Posisi Paradigma

**Topik riset:** Implementasi dan Evaluasi Performa Enkripsi End-to-End pada Aplikasi Web Berbasis Arsitektur Serverless.

> **Skala 1–5:** 1 = tidak sesuai sama sekali dengan topik ini, 5 = sangat sesuai dan dominan digunakan pada riset bertopik serupa.

Kriteria	Positivis	Interpretivis	Design Science
Kesesuaian dengan topik (1–5)	4 — Sangat cocok untuk menguji hipotesis performa komparatif (misal: AES vs ChaCha20) secara kuantitatif dan objektif.	2 — Kurang cocok karena fokus utama riset bukan menggali makna sosial atau persepsi subjektif manusia terhadap enkripsi.	5 — Sangat dominan karena riset ini bertujuan merancang, membangun, dan mengevaluasi artefak teknologi (sistem enkripsi web).
Jenis data yang dikumpulkan	Ukuran overhead waktu (milidetik), konsumsi CPU/RAM serverless, tingkat kegagalan dekripsi.	Hasil wawancara dengan developer mengenai pengalaman mereka menerapkan pustaka kriptografi tersebut.	Arsitektur kode program, hasil security audit log, bagan interaksi komponen sistem, hasil uji fungsionalitas.
Limitasi paradigma	Hanya fokus pada angka; gagal menangkap aspek usability (apakah arsitektur ini menyulitkan developer saat maintenance).	Tidak bisa memberikan bukti empiris apakah algoritma yang diimplementasikan benar-benar efisien secara matematis atau tidak.	Terkadang terlalu fokus pada "keberhasilan membuat alat bekerja," tetapi mengabaikan generalisasi teori dasar kriptografinya.

**Paradigma yang dipilih:** Design Science Research (DSR)
**Alasan:** Karena fokus utama dari riset implementasi enkripsi web ini adalah memecahkan masalah praktis (melindungi data sensitif di web) dengan cara menciptakan sebuah artefak baru (arsitektur/modul enkripsi). Kesuksesan riset dinilai dari bagaimana artefak tersebut berfungsi secara efektif dan efisien sesuai kebutuhan keamanan sistem.

---

## Refleksi

> Sebelum membaca materi ini, apakah pernah mempertanyakan klaim "95% akurat"? Setelah memahami rantai distorsi, pertanyaan apa yang sekarang akan diajukan saat membaca paper?

**Jawaban:**
Jujur saja, sebelum ini saya sering mengabaikan latar belakang di balik angka-angka persentase tinggi tersebut dan menganggapnya sebagai kebenaran mutlak. Namun, setelah memahami rantai distorsi dari Reality hingga Knowledge, pandangan saya berubah.

Sekarang, saat membaca paper tentang enkripsi data web, pertanyaan utama yang akan saya ajukan adalah: "Di manakah letak bias pengujiannya? Apakah angka performa yang tinggi ini didapatkan dengan mengorbankan kondisi realitas (seperti latensi jaringan buruk atau perangkat berspesifikasi rendah)? Dan apakah ada manipulasi data (seperti penghapusan outlier) demi mengejar signifikansi statistik statistik semata?"

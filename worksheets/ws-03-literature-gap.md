# WS-03: Literature Mapping & Gap

> **Bab 3 — Literature Review, Research Gap & Baseline**

---

## Ringkasan Materi

### Literature Review = Positioning, Bukan Ringkasan

Literature review bukan merangkum paper satu per satu. Pendekatan yang benar adalah **concept-centric** — organisasi berdasarkan tema, metode, atau variabel. Tujuan: menemukan **pola, kontradiksi, dan gap**.

**Perbandingan pendekatan Author-centric vs Concept-centric:**

| Aspek | Author-centric (Hindari) | Concept-centric (Gunakan) |
|-------|--------------------------|---------------------------|
| Struktur | Per penulis/paper ("Rahman et al. menyatakan...") | Per konsep/metode ("Pendekatan berbasis transformer") |
| Tujuan | Ringkasan isi paper | Perbandingan metode & identifikasi gap |
| Contoh paragraph | "Rahman (2023) pakai CNN. Lee (2022) pakai LSTM. Zhang (2021) pakai RF." | "Tiga pendekatan dominan: CNN digunakan oleh 4 paper untuk representasi fitur visual; LSTM untuk data sekuensial; RF sebagai baseline klasik." |
| Hasil akhir | Daftar paper | Peta pengetahuan + gap yang teridentifikasi |

### Empat Jenis Research Gap

| Jenis Gap | Deskripsi | Contoh |
|-----------|----------|--------|
| **Performance Gap** | Performa belum memadai | Akurasi deteksi hanya 78% pada kasus tertentu |
| **Method Gap** | Pendekatan belum diterapkan | Belum ada yang pakai transformer untuk task ini |
| **Data Gap** | Dataset terbatas/tidak representatif | Semua studi pakai dataset sintetis |
| **Context Gap** | Belum diuji pada konteks berbeda | Belum ada evaluasi di negara berkembang |

Gap terkuat = kombinasi 2+ jenis.

### Systematic Search Strategy

1. **Database utama**: IEEE Xplore, ACM DL, Scopus
   - Akses IEEE/ACM melalui jaringan kampus atau VPN institusi
   - Alternatif bebas biaya: Google Scholar, ResearchGate ([researchgate.net](https://www.researchgate.net)), arXiv ([arxiv.org](https://arxiv.org))
2. **Boolean query** yang terdokumentasi eksplisit
   - Contoh: `("anomaly detection" OR "intrusion detection") AND ("deep learning" OR "neural network") NOT ("medical imaging")`
   - Gunakan tanda kutip untuk frasa eksak; AND/OR/NOT mengontrol scope
3. **Snowballing** — dua arah:
   - **Backward snowballing**: buka daftar referensi di paper kunci → telusuri paper yang dikutip
   - **Forward snowballing**: di Google Scholar, klik "Cited by" di bawah paper kunci → temukan paper yang mengutipnya
   - Ulangi 1–2 tingkat untuk membangun cakupan komprehensif
4. Klaim "belum ada penelitian" harus didukung **bukti pencarian**

### Baseline Selection — 3 Kriteria

| Kriteria | Pertanyaan |
|----------|-----------|
| **Relevan** | Apakah menyelesaikan masalah yang sama? |
| **Representatif** | Apakah mewakili common practice? |
| **State-of-the-Art** | Apakah terbaru/terbaik? |

Membandingkan deep learning 2024 dengan decision tree sederhana tanpa justifikasi = **straw man comparison** (perbandingan tidak jujur).

### Research vs Engineering

| Aspek | Engineering | Research |
|-------|------------|----------|
| Tujuan baca literatur | Mencari solusi yang sudah ada | Memahami apa yang belum terjawab |
| Cara membaca paper | Tutorial, how-to | Metode, limitasi, gap |
| Baseline | Framework terpopuler | State-of-the-art yang rigorous |
| Dokumentasi pencarian | Tidak diperlukan | Wajib (reproducible) |

### Istilah Penting

- **Concept-centric** — Organisasi literatur berdasarkan konsep/metode, bukan per penulis
- **Snowballing** — Backward (telusuri referensi) + Forward (cari yang mengutip paper kunci)
- **Research Position** — Pernyataan eksplisit posisi riset terhadap studi sebelumnya
- **Straw man comparison** — Memilih baseline lemah agar metode sendiri terlihat lebih baik

---

## Template A.3 — Literature Mapping & Gap Identification

```
LITERATURE MAPPING

Topik      : Optimasi Performa Enkripsi Client-Side pada Aplikasi Web Berkinerja Tinggi
Database   : IEEE Xplore, ACM Digital Library, Google Scholar
Query      : ("web application" OR "browser") AND ("client-side encryption" OR "end-to-end encryption") AND ("performance" OR "latency") AND ("WebAssembly" OR "Web Workers")
Tahun      : 2020 - 2026
Hasil awal : 142 paper → Screening → 5 paper final

Literature Matrix (concept-centric):

| Study               | Tahun | Method                           | Data                 | Result                                               | Limitation                                          |
|---------------------|-------|----------------------------------|----------------------|------------------------------------------------------|-----------------------------------------------------|
| De Almeida & Inácio | 2020  | WebCrypto API vs WebAssembly    | Text payload 1KB-50MB| WebCrypto unggul di native API, WASM di custom algo. | Hanya diuji pada desktop browser tunggal (Chrome).  |
| Jangda et al.       | 2021  | WebAssembly Benchmark Suite      | Kriptografi & Math v8| WASM 1.2x-2x lebih lambat dari native, jauh > JS.    | Mengabaikan UI thread locking di perangkat mobile.   |
| Kim & Lee           | 2022  | Multithreading Web Workers (AES) | Multimedia files     | Mengurangi UI freeze, tapi thread spawn overhead.     | Overhead postMessage menurunkan throughput data kecil.|
| Singh et al.        | 2024  | Pure JS Cryptography (CryptoJS)  | JSON E-health data   | Latensi tinggi (>3000 ms) di RAM <4GB.               | Tidak menggunakan akselerasi hardware binary.       |
| Zhao et al.         | 2025  | Streaming WebCrypto API          | IoT Data Stream      | Throughput stabil pada koneksi desktop berkecepatan. | Bottleneck memori saat network throttling di mobile.|

Pola yang ditemukan:
  Metode dominan     : Web Cryptography API standar browser dan WebAssembly untuk algoritma kustom.
  Dataset umum       : Data teks statis berukuran modular (1 KB hingga 10 MB) atau file biner buatan.
  Limitasi berulang  : Pengujian dieksekusi secara eksklusif pada lingkungan desktop (High-End PC) dan mengabaikan degradasi performa pada browser mobile.

GAP IDENTIFICATION

Gap 1: [Jenis: method]
  Deskripsi    : Sebagian besar penelitian mengevaluasi WebAssembly kriptografi pada single-thread eksekusi JavaScript, belum mengintegrasikannya secara hibrida dengan arsitektur multi-thread (Web Workers) untuk menangani enkripsi berbasis stream payload besar di web.
  Bukti        : Studi Jangda et al. (2021) dan De Almeida (2020) fokus pada perbandingan komputasi linear murni tanpa memisahkan Main UI thread.
  Signifikansi : Tanpa integrasi Web Workers, modul WebAssembly yang cepat sekalipun akan tetap mengunci (freeze) antarmuka pengguna saat mengenkripsi payload berukuran besar di browser mobile.

Gap 2: [Jenis: context]
  Deskripsi    : Minimnya metrik evaluasi performa enkripsi client-side web di lingkungan mobile browser berdaya rendah (low-end devices) dengan batasan memori sandbox yang ketat.
  Bukti        : Studi Zhao et al. (2025) mengakui adanya kendala memori di mobile, namun pengujian empiris mereka tetap didominasi infrastruktur PC.
  Signifikansi : Solusi enkripsi aplikasi web saat ini rentan gagal (crash atau hang) ketika diakses oleh pengguna kelas menengah ke bawah yang memiliki keterbatasan resource hardware.

Baseline Selection:
| Baseline                           | Relevansi                                     | Representatif                               | Source               |
|------------------------------------|-----------------------------------------------|---------------------------------------------|----------------------|
| Pure JS Encryption (CryptoJS)      | Membandingkan dengan arsitektur tradisional.   | Digunakan oleh >60% sistem warisan (legacy).| Singh et al., 2024   |
| Standar Native WebCrypto API       | Mengukur batas performa maksimum browser.     | Standar industri enkripsi web saat ini.     | W3C Recommendation   |

---

## Latihan 1 — Concept-Centric Literature Table

Gunakan topik riset dari WS-02. Cari minimal 5 paper relevan menggunakan database akademik.

> **Panduan pencarian:**
> - Database: IEEE Xplore, ACM DL, Google Scholar, atau ResearchGate
> - Tulis query Boolean yang digunakan: contoh `("object detection" OR "image classification") AND ("edge computing") NOT ("medical")`. Dokumentasikan query secara eksplisit.
> - Akses gratis: buka Google Scholar → cari judul paper → klik [PDF] jika tersedia, atau akses lewat campus VPN

**Topik riset:** Optimasi Performa Enkripsi End-to-End di Sisi Klien menggunakan WebAssembly dan Web Workers.
**Query pencarian:** ("web application" OR "browser") AND ("client-side encryption" OR "end-to-end") AND ("WebAssembly" OR "Web Workers")
**Database:** IEEE Xplore & Google Scholar
#	Study	Tahun	Method	Dataset	Result	Limitasi
1	De Almeida & Inácio	2020	WebCrypto vs WebAssembly	Payload teks bervariasi (1KB-50MB)	WebCrypto unggul pada fungsi native, WASM unggul pada kustomisasi algoritma.	Pengujian terbatas pada PC desktop berkinerja stabil.
2	Jangda et al.	2021	WASM Compilation Benchmark	Paket kode biner komputasi matematika	WebAssembly memangkas waktu eksekusi hingga 50% dibanding JS murni.	Tidak mengukur dampak nyata terhadap kelancaran UI frame rate (FPS).
3	Kim & Lee	2022	Parallel Web Workers (AES)	Berkas dokumen dan gambar acak	Berhasil menghilangkan gejala UI freeze pada file berukuran sedang.	Mengalami overhead komunikasi data (postMessage) pada file di bawah 500 KB.
4	Singh et al.	2024	Pure JS Cryptography	Log data rekam medis terstruktur	Mudah diimplementasikan di semua browser tanpa konfigurasi khusus.	Menghasilkan latensi ekstrem (>3000 milidetik) untuk enkripsi enkapsulasi besar.
5	Zhao et al.	2025	Streaming WebCrypto API

**Pola yang terlihat — Metode dominan:** Penggunaan WebAssembly sebagai akselerator komputasi tingkat rendah (low-level binary speed) dan Web Cryptography API untuk pemanggilan fungsi primitif kriptografi bawaan browser.
**Limitasi yang berulang:** Mayoritas riset mengabaikan pengujian pada kondisi ekstrem ekosistem mobile (misal: fluktuasi RAM tersedia, pelambatan CPU gawai, dan overhead serialisasi data lintas thread).

---

## Latihan 2 — Gap Identification

Berdasarkan tabel di Latihan 1, identifikasi gap.

| Jenis Gap | Ditemukan? | Gap Statement |
|-----------|-----------|---------------|
| Performance Gap | [v] Ya / [ ] Tidak | *Contoh: Akurasi turun di bawah 80% untuk kelas minoritas* |
| Method Gap | [v] Ya / [ ] Tidak | |
| Data Gap | [ ] Ya / [v] Tidak | |
| Context Gap | [v] Ya / [ ] Tidak | |

**Gap utama yang dipilih:** Method & Context Gap (Hibridisasi WebAssembly-Web Workers pada Browser Mobile).
**Mengapa gap ini penting (bukan sekadar "belum ada yang meneliti")?**
> Gap ini krusial karena jika tidak diselesaikan, visi Privacy by Design (di mana data dienkripsi langsung di tangan pengguna sebelum menyentuh cloud) akan gagal diadopsi secara massal. Pengguna gawai mobile berspesifikasi rendah akan dipaksa memilih antara mengorbankan privasi data mereka (mengirim plaintext) atau mengorbankan fungsionalitas aplikasi (aplikasi macet/freeze akibat beban enkripsi). Menyelesaikan gap metode hibrida ini akan mewujudkan sistem keamanan web yang inklusif dan efisien di semua kelas perangkat komputer.

---

## Latihan 3 — Baseline Selection

Pilih 2 baseline dari literatur yang sudah dibaca.

#	Baseline	Mengapa Relevan	Mengapa Representatif	Apakah SOTA?	Sumber
1	CryptoJS running on Main JavaScript Thread	Algoritma dan tujuannya sama, yaitu memproses enkripsi data di sisi klien.	Merupakan pustaka paling populer dan standar praktis (common practice) yang paling banyak digunakan oleh developer web saat ini.	Bukan. Ini adalah pendekatan legacy yang tidak efisien untuk beban data modern.	Singh et al., 2024
2	Native Web Cryptography API (Single Thread)	Menguji algoritma enkripsi bawaan yang sudah dioptimalkan oleh vendor browser (C++ level).	Mewakili batas kecepatan tertinggi standar industri web modern saat ini (state-of-the-art untuk API bawaan).	Ya, untuk kategori eksekusi asinkron bawaan (native browser code).	W3C Recommendation / De Almeida & Inácio, 2020

**Apakah pemilihan baseline ini bisa dianggap straw man?** [ ] Ya / [ ] Tidak
> Justifikasi: Pemilihan baseline ini adil dan valid karena mencakup dua spektrum nyata dalam dunia web development: CryptoJS mewakili realitas ekosistem aplikasi web lama yang mendominasi pasar produksi saat ini (market representative), sedangkan Web Cryptography API mewakili performa batas atas teoretis dari kapabilitas bawaan browser modern (SOTA representative). Membandingkan metode baru dengan kedua baseline ini akan membuktikan apakah inovasi kita benar-benar memberikan lompatan efisiensi yang signifikan atau tidak.

---

## Refleksi

> Apa perbedaan antara "belum ada yang meneliti ini" (klaim tanpa bukti) dengan research gap yang valid? Bagaimana cara membuktikan bahwa sebuah gap benar-benar ada?

**Jawaban:**
Klaim "belum ada yang meneliti ini" sering kali merupakan bentuk ketidaktahuan peneliti (argument from ignorance) akibat kurangnya kedalaman proses literatur review. Klaim semacam ini berbahaya karena bisa jadi topik tersebut memang tidak diteliti karena tidak memiliki nilai urgensi ilmiah, tidak logis secara arsitektur komputer, atau solusinya sudah dianggap selesai di industri.

Sebaliknya, research gap yang valid adalah sebuah kesenjangan yang ditemukan secara sadar melalui proses dekomposisi dan pemetaan matriks konsep literatur saat ini. Gap valid lahir ketika kita berhasil membuktikan adanya kontradiksi, keterbatasan metode terdahulu, atau ketidakmampuan solusi mutakhir saat ini dalam menyelesaikan masalah baru di kondisi dunia nyata (realitas sistem).

Cara membuktikannya: Kita harus menyajikan bukti dokumentasi ilmiah yang kuat di bab pendahuluan atau kajian pustaka. Kita tunjukkan secara eksplisit: "Peneliti A telah mencoba menyelesaikan masalah X dengan metode Y, namun mereka mengabaikan batasan Z. Di sisi lain, Peneliti B mencoba menyelesaikan batasan Z di bidang yang berbeda, namun tidak cocok untuk domain X. Oleh karena itu, terdapat celah (gap) integrasi antara Y dan Z untuk mengatasi masalah sistem ini." Bukti ini diperkuat dengan visualisasi matriks literatur yang menunjukkan kekosongan pada koordinat irisan variabel tersebut.

# WS-02: Problem Statement

> **Bab 2 — Problem Formulation & System Context**

---

## Ringkasan Materi

### Problem Formation Model

Masalah riset melewati 5 tahap transformasi. Melompat langsung dari Reality ke Variable adalah kesalahan paling umum.

```
Reality → Observed Issue (Symptom) → Diagnosed Problem (Root Cause)
→ Researchable Problem (Scoped) → Measurable Variable (Operationalized)
```

### Topic ≠ Problem ≠ Research Problem

| Level | Contoh | Status |
|-------|--------|--------|
| **Topik** | Keamanan IoT | Terlalu luas, tidak bisa diuji |
| **Problem** | MQTT tidak terenkripsi | Spesifik tapi belum riset |
| **Research Problem** | Belum ada studi membandingkan overhead TLS 1.3 vs DTLS pada MQTT di IoT RAM < 64KB | Bisa dirancang eksperimennya |

### Symptom vs Root Cause

Apa yang diamati (gejala) ≠ mengapa terjadi (akar masalah). Gunakan **5 Whys** atau **Fishbone Diagram** untuk menggali.

Contoh: "User meninggalkan checkout" (symptom) → "Waktu loading > 8 detik karena API call sequential" (root cause).

### System Thinking

Setiap masalah riset TI harus terikat pada komponen sistem: **Input → Process → Output → Outcome → Constraints → Stakeholders**.

### Problem Quality Check

Masalah riset yang layak harus memenuhi 5 kriteria:
- **Clarity** — Satu orang membaca akan paham
- **Measurability** — Ada metrik kuantitatif
- **Relevance** — Penting untuk domain
- **Testability** — Bisa gagal (falsifiable)
- **Impact** — Ada kontribusi jika terjawab

### Research vs Engineering

| Aspek | Engineering | Research |
|-------|------------|----------|
| Tujuan | Menyelesaikan masalah (*solve*) | Memahami dan membuktikan (*understand & prove*) |
| Masalah | Bug, error, fitur belum ada | Gap dalam pengetahuan |
| Scope | Selesaikan semua yang perlu | Batasi agar bisa dibuktikan |
| Output | Working system | Evidence, paper, replicable findings |

### Istilah Penting

- **Problem Statement** — Formulasi tertulis: konteks sistem + gap + dampak + justifikasi
- **System Context** — Deskripsi lengkap: input, proses, output, outcome, constraints, stakeholders
- **Problem Drift** — Masalah "bermutasi" dari pendahuluan ke metodologi karena statement awal tidak presisi
- **Solution-First Thinking** — Memulai dari solusi tanpa masalah yang jelas — berbahaya dalam riset
- **Operational Definition** — Definisi variabel yang cukup jelas agar peneliti lain bisa mengukur hal yang sama

---

## Template A.2 — Problem Statement Builder

```
Domain & Konteks
  Domain   : Keamanan Aplikasi Web / Kriptografi Terapan
  Konteks  : Enkripsi data di sisi klien (Client-Side Encryption) pada aplikasi web 
             yang menangani data medis atau finansial sensitif.

System Context
  Input       : Data sensitif mentah (plaintext), kunci enkripsi (cryptographic key), 
                dan pustaka/modul kriptografi.
  Process     : Proses komputasi matematis untuk mengubah plaintext menjadi ciphertext 
                di dalam runtime browser sebelum data dikirim melalui jaringan.
  Output      : Payload data terenkripsi aman (ciphertext).
  Outcome     : Kerahasiaan data terjaga sejak dari perangkat pengguna (End-to-End Privacy), 
                meminimalkan risiko kebocoran data akibat serangan Man-in-the-Middle (MitM).
  Constraints : Keterbatasan daya komputasi CPU pada gawai mobile low-end, batasan memori 
                sandbox browser, dan lingkungan JavaScript yang bersifat single-threaded.
  Stakeholders: Pengembang web (Web Developer), Tim Keamanan Siber, dan Pengguna Aplikasi.

Fenomena → Problem
  Fenomena yang diamati             : Aplikasi web mengalami perlambatan performa atau 
                                      membeku (freeze) sesaat ketika memproses enkripsi data.
  Gejala (symptom) yang terukur     : Latensi pemrosesan enkripsi mencapai >2.500 ms untuk 
                                      file/payload berukuran >2 MB pada perangkat mobile.
  Masalah yang didiagnosis          : Operasi matematika intensif algoritma kriptografi 
                                      menghabiskan resource Main UI Thread pada JavaScript.
  Masalah riset (researchable)      : Bagaimana meminimalkan overhead latensi enkripsi data 
                                      client-side melalui arsitektur komputasi paralel atau 
                                      eksekusi berbasis low-level binary di browser?
  Variabel yang terukur             : Waktu eksekusi enkripsi (ms), konsumsi kapasitas CPU (%), 
                                      dan frame rate UI (FPS) selama proses enkripsi.

Problem Quality Check
  [X] Clarity — Apakah satu orang membaca akan paham?
  [X] Measurability — Apakah ada metrik kuantitatif?
  [X] Relevance — Apakah penting untuk domain?
  [X] Testability — Apakah bisa gagal?
  [X] Impact — Apakah ada kontribusi jika terjawab?

Problem Statement (1 paragraf):
  Implementasi enkripsi data client-side pada aplikasi web sering kali memicu latensi 
  komputasi yang tinggi (mencapai >2.500 ms untuk payload >2 MB) khususnya pada perangkat 
  mobile, karena operasi kriptografi intensif berjalan di atas Main UI Thread JavaScript 
  yang bersifat single-threaded. Akibatnya, aplikasi mengalami pembekuan antarmuka 
  (UI freeze) dan penurunan responsivitas yang merusak user experience. Penelitian ini 
  berfokus pada optimalisasi performa komputasi enkripsi di sisi klien dengan menguji 
  efisiensi arsitektur paralel berbasis Web Workers dan WebAssembly untuk mereduksi latensi 
  eksekusi tanpa menurunkan standar kekuatan algoritma enkripsi yang digunakan.
```

---

## Latihan 1 — Dari Topik ke Masalah Riset

Pilih satu topik di bidang TI yang diminati. Transformasikan melalui 5 tahap Problem Formation Model.

**Topik awal:** Implementasi enkripsi data pada aplikasi web

Tahap	Hasil
Reality	Pengembang ingin mengamankan data formulir sensitif langsung dari browser pengguna sebelum data tersebut dikirim ke server cloud.
Observed Issue (Symptom)	Pengguna mengeluhkan tombol "Kirim/Simpan" menjadi tidak responsif selama beberapa detik, dengan metrik freeze rata-rata 2,5 detik pada perangkat mobile.
Diagnosed Problem (Root Cause)	Pustaka kriptografi yang digunakan ditulis dalam JavaScript murni (pure JS library) yang mengeksekusi algoritma berat (seperti AES-256-GCM) langsung di main thread browser.
Researchable Problem	Memanfaatkan teknologi low-level binary (WebAssembly) untuk menggeser beban komputasi berat keluar dari eksekusi JavaScript standar di browser.
Measurable Variable	Latensi enkripsi (milidetik), Throughput data (MB/detik), dan responsivitas antarmuka aplikasi (Frame Per Second / FPS).

**Apakah terjebak solution-first thinking?** [ ] Ya / [v] Tidak
> Jika ya, kembali ke tahap mana? Sudah fokus pada penyelesaian bottleneck performa berdasarkan akar masalah main thread block

---

## Latihan 2 — System Context Decomposition

Gambarkan konteks sistem dari masalah riset di Latihan 1.

Komponen	Deskripsi
Input	Data isian pengguna (plaintext) dan kunci rahasia (session key) yang di-generate di browser.
Process	Eksekusi fungsi matematika algoritma enkripsi menggunakan pustaka berbasis WebAssembly.
Output	String terenkripsi (ciphertext) berformat Base64 atau ArrayBuffer.
Outcome	Data terkirim ke server dengan aman tanpa bisa diintip oleh penyedia jaringan (ISP) atau penyerang lokal.
Constraints	Keterbatasan dukungan spesifikasi WebAssembly pada browser versi lama dan keterbatasan daya baterai gawai mobile akibat komputasi intensif.
Stakeholders	Pengguna akhir aplikasi web (pemilik data) dan Pengembang Front-End.

**Komponen mana yang paling relevan dengan masalah riset?** Process dan Constraints

---

## Latihan 3 — Problem Quality Check

Evaluasi problem statement yang sudah dibuat menggunakan 5 kriteria.

Kriteria	Skor (1-5)	Justifikasi
Clarity	5	Masalah sangat spesifik: latensi enkripsi web, penyebabnya (single-threaded JS), dan dampaknya (UI freeze).
Measurability	5	Memiliki batasan angka konkret (>2.500 ms untuk file >2 MB) serta metrik pembanding (ms, FPS).
Relevance	4	Sangat relevan di era privasi data ketat (Privacy by Design), namun terbatas pada ekosistem web modern.
Testability	5	Dapat diuji dengan mudah melalui skenario eksperimen lab: jika WebAssembly tidak memangkas waktu eksekusi, hipotesis dinyatakan gagal.
Impact	4	Memberikan solusi nyata bagi arsitektur aplikasi web berkinerja tinggi yang aman tanpa mengorbankan kenyamanan pengguna.

**Skor total:** 23/25

**Problem statement versi final (1 paragraf):**
Implementasi enkripsi data client-side berbasis JavaScript murni pada aplikasi web sering kali memicu latensi tinggi (mencapai >2.500 ms untuk payload berukuran >2 MB) yang memblokir Main UI Thread browser. Hambatan ini mengakibatkan penurunan responsivitas halaman web secara drastis saat memproses data sensitif pengguna pada gawai mobile menengah ke bawah. Penelitian ini ditujukan untuk mengatasi masalah latensi tersebut dengan merancang dan menguji modul enkripsi berbasis WebAssembly guna memindahkan komputasi berat kriptografi ke tingkat low-level binary execution, sehingga diharapkan mampu mereduksi overhead latensi tanpa mengurangi integritas keamanan data.
---

## Refleksi

> Bandingkan "masalah" yang biasa ditemui saat coding (bug, error) dengan masalah riset. Apa perbedaan fundamental dalam cara mendefinisikan dan mendekati keduanya?

**Jawaban:**
Perbedaan utamanya terletak pada sifat kepastian solusi dan cakupannya.

Saat coding, masalah biasanya berupa bug atau error (seperti TypeError: crypto.encrypt is not a function). Masalah ini bersifat deterministik, lokasinya jelas pada baris kode tertentu, dan solusinya sudah pasti ada (misalnya memperbaiki sintaksis atau melengkapi pustaka yang kurang). Cara pendekatannya adalah pelacakan kesalahan (debugging) berdasarkan dokumentasi teknis yang ada.

Sebaliknya, masalah riset bersifat multi-dimensi dan tidak memiliki jawaban tunggal yang instan. Masalah riset muncul dari adanya gap performa atau batasan sistem di dunia nyata (seperti "Mengapa enkripsi aman selalu membuat web lambat di HP murah?"). Pendekatannya membutuhkan dekomposisi variabel, eksperimen komparatif, dan pembuktian empiris. Solusi dari masalah riset adalah sebuah kontribusi pengetahuan baru yang menyeimbangkan berbagai kompromi teknis (trade-off), bukan sekadar membuat kode program berjalan tanpa error error di konsol browser.

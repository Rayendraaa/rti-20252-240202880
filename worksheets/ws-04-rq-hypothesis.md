# WS-04: Research Question & Hypothesis

> **Bab 4 — Research Question, Contribution & Hypothesis**

---

## Ringkasan Materi

### RQ Bukan Pertanyaan Biasa

Research Question yang baik secara implisit mengandung cetak biru eksperimen: subjek, baseline, metrik, domain, dataset.

| Kualitas | Contoh |
|----------|--------|
| **Buruk** | "Bagaimana pengaruh deep learning terhadap deteksi malware?" |
| **Baik** | "Apakah CNN menghasilkan F1-Score lebih tinggi dari RF pada CIC-MalMem-2022?" |

Perbedaan: RQ yang baik menyebutkan **metode spesifik**, **metrik terukur**, **baseline**, dan **dataset**.

### Tiga Jenis RQ

| Jenis | Pola | Kebutuhan |
|-------|------|-----------|
| **Comparison** | A vs B → mana lebih baik? | ≥ 2 metode, metrik sama |
| **Improvement** | A' vs A → modifikasi lebih baik? | Pre/post, bukti perbaikan |
| **Exploratory** | Faktor X₁...Xₙ → pengaruh terhadap Y? | Multi-variabel, korelasi/regresi |

### Contribution Statement

Tiga jenis kontribusi: **Improvement** (metode terbukti lebih baik), **Comparison** (perbandingan sistematis yang belum ada), **Novel Approach** (pendekatan baru). Kontribusi harus terhubung langsung dengan gap — kontribusi tanpa gap = klaim tanpa justifikasi.

### Hypothesis H₀ / H₁

- **H₀** (Null) = Tidak ada perbedaan signifikan — asumsi default, harus dibuktikan salah
- **H₁** (Alternative) = Ada perbedaan signifikan — diterima hanya jika H₀ ditolak
- Harus **falsifiable**, mengandung **metrik terukur**, dirumuskan **SEBELUM eksperimen**

### Rantai Operasionalisasi

```
RQ → Variable → Metric → Data → Analysis
```

Jika rantai ini tidak lengkap, RQ belum mature. Bi-directional: RQ yang tidak bisa jadi hipotesis testable harus direvisi mundur.

### Research vs Engineering

| Aspek | Engineering | Research |
|-------|------------|----------|
| Tujuan pertanyaan | Apa yang harus dibangun? | Apa yang harus dibuktikan? |
| Bentuk jawaban | Sistem yang berfungsi | Bukti empiris terukur |
| Sukses diukur oleh | User satisfaction, uptime | Signifikansi statistik, effect size |
| Jika gagal | Debug dan perbaiki | Laporkan, analisis mengapa |

### Istilah Penting

- **Research Question (RQ)** — Pertanyaan spesifik: variabel terukur + metrik + konteks
- **Contribution Statement** — Apa yang diketahui setelah riset selesai yang sebelumnya belum ada
- **H₀ / H₁** — Null vs Alternative Hypothesis
- **Falsifiability** — Kondisi hipotesis ditolak harus bisa didefinisikan sebelum eksperimen
- **Operationalization** — Proses mewujudkan konsep abstrak menjadi variabel terukur

---

## Template A.4 — RQ-Contribution-Hypothesis

```
RQ-CONTRIBUTION-HYPOTHESIS

Gap Statement  : Sebagian besar metode enkripsi client-side pada aplikasi web memicu UI freeze (antarmuka membeku) pada browser mobile karena komputasi berjalan di atas single main thread JavaScript, serta belum adanya skema hibrida yang mengombinasikan WebAssembly dan Web Workers untuk menangani data stream berukuran besar secara asinkron.

Research Question:
  Tipe         : [ ] Comparison  [v] Improvement  [ ] Exploratory
  Formulasi    : Seberapa besar penurunan Total Blocking Time (TBT) dan Execution Latency yang dihasilkan oleh arsitektur enkripsi hibrida WebAssembly-Web Workers (WASM-WW) dibandingkan dengan CryptoJS dan native WebCrypto API saat memproses enkripsi data di lingkungan mobile browser dengan kondisi CPU terbatasi?
  Variabel IV  : Arsitektur/Pustaka Enkripsi (CryptoJS vs Native WebCrypto API vs Hibrida WASM-WW).
  Variabel DV  : Total Blocking Time (TBT) dan Execution Latency.
  Metrik       : Milidetik (ms) untuk waktu dan Frame per Second (FPS) untuk stabilitas UI.
  Dataset      : Dataset JSON terstruktur (data rekam medis simulasi) berukuran 500 KB, 2 MB, dan 10 MB.
  Baseline     : 1. CryptoJS (mewakili pure JavaScript framework)
                 2. Native Web Cryptography API (mewakili standard single-thread browser API)

Quality Check RQ:
  [v] Variabel spesifik
  [v] Metrik jelas
  [v] Baseline ada
  [v] Konteks disebutkan
  [v] Memerlukan eksperimen (bukan hanya survei literatur)

Contribution Statement:
  Apa yang baru diketahui : Karakteristik efisiensi performa dan batas optimal dari hibridasi low-level binary (WebAssembly) di dalam background thread (Web Workers) dalam mereduksi interupsi UI thread pada perangkat mobile berdaya rendah.
  Jenis kontribusi        : [v] Improvement  [ ] Comparison  [ ] Novel approach
  Gap yang diisi          : Mengatasi masalah UI freeze dan degradasi throughput enkripsi client-side pada konteks ekosistem mobile browser.

Hypothesis Pair:
  H₀ : Tidak ada penurunan signifikan pada Total Blocking Time (TBT) dan Execution Latency antara arsitektur hibrida WASM-WW dengan native WebCrypto API ketika mengeksekusi enkripsi data pada mobile browser dengan CPU throttling. ($H_0: \mu_{WASM-WW} \ge \mu_{WebCrypto}$)
  H₁ : Arsitektur hibrida WASM-WW menurunkan Total Blocking Time (TBT) secara signifikan hingga di bawah ambang batas 50 ms dan mempertahankan UI frame rate di atas 50 FPS dibandingkan native WebCrypto API. ($H_1: \mu_{WASM-WW} < \mu_{WebCrypto}$)
  Threshold              : Tingkat signifikansi statistik $\alpha = 0.05$ melalui Paired T-Test, serta batas teknis TBT < 50 ms.
  Justifikasi threshold  : Nilai $\alpha = 0.05$ adalah standar pengujian hipotesis kuantitatif ilmu komputer, sedangkan TBT < 50 ms merujuk pada Google RAIL Model (User-Centric Performance) agar aplikasi dipersepsikan "instan" dan bebas lag oleh manusia.
```

---

## Latihan 1 — Dari Gap ke RQ

Gunakan gap yang ditemukan di WS-03. Transformasikan menjadi Research Question.

**Gap dari WS-03:** Belum adanya skema enkripsi hibrida yang mengintegrasikan WebAssembly ke dalam klaster Web Workers untuk mengatasi UI freeze akibat beban komputasi kriptografi berat pada browser mobile.

**RQ versi pertama (tulis bebas):**
> Bagaimana cara membuat enkripsi di aplikasi web menjadi lebih cepat menggunakan WebAssembly dan Web Workers agar tidak membuat handphone hang atau lambat?

**Evaluasi RQ:**

Evaluasi RQ:KomponenAda?IsiMetode spesifikYaMenggunakan integrasi WebAssembly dan Web Workers.Metrik terukurTidakKata "lebih cepat" dan "hang" terlalu abstrak/kualitatif, belum berupa metrik teknis.BaselineTidakBelum disebutkan sistem apa yang akan dijadikan pembanding keberhasilannya.Dataset/konteksParsialMenyebutkan konteks handphone (mobile), tetapi belum mendefinisikan batasan lingkungan ujinya.
**Tipe RQ:** [ ] Comparison / [v] Improvement / [ ] Exploratory

**RQ versi revisi (setelah evaluasi):**
> Apakah arsitektur hibrida WASM-WW mampu mereduksi Total Blocking Time (TBT) hingga di bawah 50 ms dibandingkan dengan CryptoJS dan Native WebCrypto API saat melakukan enkripsi payload data JSON >2 MB di dalam lingkungan mobile browser dengan CPU throttling 4x?

---

## Latihan 2 — Hypothesis Pair

Rumuskan pasangan hipotesis dari RQ di Latihan 1.

KomponenIsiH₀Arsitektur hibrida WASM-WW tidak memberikan penurunan yang signifikan pada Total Blocking Time (TBT) dibandingkan dengan Native WebCrypto API saat memproses data >2 MB pada mobile browser dengan CPU throttling ($p \ge 0.05$).H₁Arsitektur hibrida WASM-WW memberikan penurunan yang signifikan pada Total Blocking Time (TBT) hingga mencapai rata-rata <50 ms dibandingkan dengan Native WebCrypto API ($p < 0.05$).MetrikTotal Blocking Time (TBT) dalam milidetik (ms) dan tingkat signifikansi statistik ($p\text{-value}$).Threshold$p < 0.05$ untuk aspek statistik, dan $\le 50\text{ ms}$ untuk aspek performa teknis.Justifikasi thresholdBatas p-value 50 ms diadopsi dari standardisasi optimasi Core Web Vitals industri web modern untuk memastikan kelancaran interaksi antarmuka.

**Apakah hipotesis ini falsifiable?** [v] Ya / [ ] Tidak
> Bagaimana cara membuktikannya salah? Hipotesis $H_1$ akan otomatis gugur (salah) jika setelah eksperimen dilakukan sebanyak 30 kali iterasi, data empiris menunjukkan bahwa TBT dari metode hibrida WASM-WW tetap berada di atas 50 ms, atau hasil uji statistik Paired T-Test menghasilkan $p \ge 0.05$ (yang berarti perbedaan performa hanya terjadi karena kebetulan/faktor noise).

---

## Latihan 3 — Rantai Operasionalisasi

Lengkapi rantai dari RQ hingga metode analisis.

Tahap	Isi
RQ	Apakah arsitektur hibrida WASM-WW mampu mereduksi Total Blocking Time (TBT) secara signifikan dibandingkan dengan CryptoJS dan Native WebCrypto API pada mobile browser?
Variable (IV)	Jenis arsitektur eksekusi enkripsi (CryptoJS vs Native WebCrypto API vs Hibrida WASM-WW).
Variable (DV)	Total Blocking Time (TBT) dan Execution Latency aplikasi web.
Metric	Durasi waktu dalam milidetik (ms) yang dicatat melalui PerformanceObserver API bawaan browser.
Data source	Log telemetri browser otomatis yang diekstrak menggunakan pustaka pengujian otomatis (Puppeteer/Selenium) pada skenario pengujian beban data terstruktur bervariasi.
Analysis method	Analisis deskriptif menggunakan Box-Plot (untuk melihat persebaran pencilan) dilanjutkan dengan uji inferensial One-Way ANOVA atau Paired T-Test untuk membuktikan signifikansi perbedaan antar-metode.

**Apakah rantai lengkap?** [v] Ya / [ ] Tidak
> Jika tidak, tahap mana yang perlu direvisi? (Rantai sudah konsisten secara linier dari perancangan variabel hingga metode statistik pengujiannya).

---

## Refleksi

> Ambil satu judul skripsi/paper yang pernah dibaca. Coba ekstrak RQ-nya. Apakah RQ tersebut memenuhi semua komponen (metode, metrik, baseline, konteks)? Jika tidak, apa yang hilang?

Judul: Implementasi Algoritma Advanced Encryption Standard (AES) untuk Mengamankan Data Rekam Medis pada Aplikasi Web Rumah Sakit X.

RQ yang diekstrak: Bagaimana cara mengimplementasikan algoritma AES pada aplikasi web untuk mengamankan data rekam medis di Rumah Sakit X?

Komponen yang hilang:

    Metrik Terukur: Tidak ada target performa atau keamanan yang jelas (misal: berapa kecepatan yang ingin dicapai atau parameter ketahanan apa yang diuji).

    Baseline Pembanding: Tidak menyertakan pembanding (apakah AES ini lebih baik dari metode enkripsi lama di rumah sakit tersebut atau tidak).

    Kebutuhan Eksperimen: Masalah ini cenderung berupa proyek rekayasa perangkat lunak standar (software engineering project) biasa, bukan riset. Pertanyaannya bisa dijawab hanya dengan membaca dokumentasi pustaka coding tanpa memerlukan pembuktian hipotesis ilmiah melalui eksperimen lab.

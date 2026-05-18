# WS-06: System-Experiment Mapping

> **Bab 6 — System Design sebagai Experimental Artifact**

---

## Ringkasan Materi

### Sistem = Instrumen Pengujian, Bukan Produk

Seorang engineer bertanya "apakah sistem bekerja?" — seorang peneliti bertanya "apa yang bisa dibuktikan sistem ini?" Sistem dalam riset adalah **artifact** — objek yang sengaja dibuat untuk menguji klaim spesifik.

### System as Experiment Model

```
RQ → Variable → System Component → Experimental Setup → Output
```

Setiap komponen sistem harus bisa ditelusuri ke variabel riset (top-down), dan setiap pengukuran harus menjawab RQ (bottom-up).

### Mapping Variabel ke Komponen

| Tipe Variabel | Peran di Sistem | Contoh |
|---------------|----------------|--------|
| **IV** (Independent) | Modul yang bisa di-toggle/swap | Algoritma A vs B |
| **DV** (Dependent) | Modul pengukuran | Logger, metrics collector |
| **CV** (Control) | Config yang dikunci | Dataset, parameter tetap |

Jika variabel tidak bisa di-map ke komponen apapun → arsitektur perlu didesain ulang.

### 4 Prinsip Desain Eksperimental

| Prinsip | Pertanyaan Kunci |
|---------|-----------------|
| **Traceability** | Komponen ini melayani variabel yang mana? |
| **Modularity** | Bisakah IV diubah tanpa memengaruhi yang lain? |
| **Controllability** | Apakah CV dieksternalisasi ke config file? |
| **Measurability** | Apakah sistem otomatis menghasilkan data yang dibutuhkan? |

### Variable Isolation melalui Arsitektur

- **Modular architecture** — Pisahkan berdasarkan variabel
- **Configuration-driven** — Ubah config (YAML/JSON), bukan code
- **Feature toggles** — On/off flag untuk ablation study

  Contoh config YAML dengan feature toggles:
  ```yaml
  model:
    type: cnn          # IV: ganti "rf" untuk kondisi baseline
  features:
    use_temporal: true  # toggle komponen temporal
    use_normalization: true  # toggle preprocessing
  experiment:
    seed: 42
    runs: 5
  ```
  Dengan pendekatan ini, berbeda kondisi eksperimen = berbeda satu baris config, **tanpa mengubah kode**.

### Research vs Engineering

| Aspek | Engineering | Research |
|-------|------------|----------|
| Tujuan sistem | Memenuhi kebutuhan user | Menguji hipotesis, menghasilkan bukti |
| Arsitektur | Optimasi performa & skalabilitas | Optimasi isolasi variabel & reprodusibilitas |
| Konfigurasi | Sering hardcoded | Dieksternalisasi ke config file |
| Fitur tambahan | Menambah nilai user | Menambah noise jika tidak terkait RQ |

### Istilah Penting

- **Artifact** — Objek yang sengaja dibuat untuk memecahkan masalah atau menguji proposisi
- **Traceability** — Kemampuan menelusuri hubungan RQ → variabel → komponen → output
- **Variable Isolation** — Mengubah hanya satu variabel sambil menahan yang lain konstan
- **Ablation Study** — Menguji kontribusi tiap komponen dengan melepasnya satu per satu
- **Configuration-driven Execution** — Semua parameter di config file, bukan hardcoded

---

## Template A.6 — Mapping RQ ke Arsitektur Sistem

```
SYSTEM-EXPERIMENT MAPPING
Research Question: 
Apakah arsitektur hibrida WASM-WW mampu mereduksi Total Blocking Time (TBT) hingga di bawah 50 ms dibandingkan dengan CryptoJS dan Native WebCrypto API saat melakukan enkripsi payload data JSON >2 MB di dalam lingkungan mobile browser dengan CPU throttling 4x?

Variable → Component Mapping:
| Variabel | Tipe | Komponen Sistem | Cara Manipulasi/Pengukuran |
|----------|------|-----------------|---------------------------|
| Arsitektur Enkripsi | IV | Modul Kriptografi (*Crypto Engine Driver*) | Mengubah konfigurasi injeksi modul (`Config.ENGINE_TYPE`) menjadi 'WASM_WW', 'CRYPTO_JS', atau 'NATIVE_WEB_CRYPTO'. |
| Total Blocking Time (TBT) | DV | Pengamat Performa (*Performance Observer API*) | *Built-in listener* mendengarkan tipe 'longtask' di Main Thread dan mengakumulasikan durasi tugas yang melebihi 50 ms. |
| Execution Latency | DV | Pencatat Waktu (*High-Resolution Timer*) | Memasang penanda waktu berbasis `performance.now()` tepat sebelum dan sesudah fungsi enkripsi dieksekusi. |
| Ukuran Payload Data | CV | Generator Data Uji (*Data Mock Provider*) | Menyuntikkan file JSON tiruan dengan opsi ukuran statis yang diatur secara presisi pada 500 KB, 2 MB, dan 10 MB. |
| Tingkat Throttling CPU | CV | Profiler Perangkat (*Browser DevTools Throttling*) | Mengontrol emulasi performa perangkat melalui protokol otomatisasi Puppeteer (`Emulation.setCPUThrottlingRate`). |

4 Prinsip Desain:
  [X] Traceability — Setiap komponen bisa ditelusuri ke variabel
  [X] Variable Isolation — IV bisa diubah tanpa mengubah CV
  [X] Measurement Integration — Pengukuran DV built-in
  [X] Reproducibility — Setup bisa direkonstruksi

Experimental Setup:
  Input data     : File data terstruktur JSON (Simulasi Rekam Medis Elektronik) bervariasi ukuran.
  Parameter      : Iterasi = 30x per skenario, Algoritma Utama = AES-256-GCM, Mode Browser = Headless Chrome.
  Output format  : File CSV berisi matriks: [Skenario_ID, IV_Type, Payload_Size, CPU_Throttle, Latency_ms, TBT_ms].
```

---

## Latihan 1 — Variable-to-Component Mapping

Gunakan RQ dan variabel dari WS-05. Petakan ke komponen sistem.

**RQ:** Apakah arsitektur hibrida WASM-WW mampu mereduksi Total Blocking Time (TBT) hingga di bawah 50 ms dibandingkan dengan CryptoJS dan Native WebCrypto API saat melakukan enkripsi payload data JSON >2 MB di dalam lingkungan mobile browser dengan CPU throttling 4x?

VariabelTipeKomponen SistemCara Manipulasi / PengukuranArsitektur EnkripsiIVCrypto Factory Wrapper (Modul Penukar Mesin Enkripsi)Mengganti dependency injection pustaka enkripsi lewat parameter inisialisasi aplikasi tanpa mengubah logika UI.Total Blocking TimeDVMain-Thread Long Task TrackerMenggunakan web API bawaan browser untuk memantau pemblokiran UI thread selama fungsi kriptografi berjalan.Execution LatencyDVCore Crypto TimerMengukur waktu Delta ($\Delta t$) pemrosesan internal pada modul yang aktif.Ukuran PayloadCVMemory Data BufferMemuat string JSON ke memori virtual browser dengan kapasitas beban yang dikunci secara statis.CPU ThrottlingCVChromium Headless ControllerMemanipulasi rate throttling (4x/6x) lewat script execution bridge dari luar sistem sandbox browser.

**Apakah semua variabel bisa di-map?** [v] Ya / [ ] Tidak
> Jika tidak, komponen apa yang perlu ditambahkan? _________

---

## Latihan 2 — 4 Prinsip Desain

Evaluasi desain arsitektur sistem pengujian enkripsi web terhadap 4 prinsip dasar riset:PrinsipStatusBukti / Penjelasan
Traceability✅Setiap komponen arsitektur mewakili variabel secara jernih. Crypto Factory Wrapper merepresentasikan IV, sementara Tracker dan Timer merepresentasikan DV.
Modularity✅Kode enkripsi dipisahkan secara ketat dari kode antarmuka. Komponen Web Worker dibuat dalam berkas terpisah (worker.js) sehingga mudah dilepas atau dipasang.
Controllability✅Peneliti memegang kendali penuh. Ukuran payload dan tingkat pelambatan CPU diatur secara programatik lewat skrip automasi luar, bebas dari intervensi manual.
Measurability✅Pengukuran bersifat built-in dan non-intrusif menggunakan API standar resmi browser (PerformanceObserver), sehingga meminimalkan efek observer's paradox (alat ukur memperlambat sistem).

**Prinsip mana yang paling sulit dipenuhi?** Controllability (Isolasi Variabel terhadap Browser Noise)
**Strategi untuk mengatasinya:**
> Mesin browser memiliki subsistem otomatis yang tidak bisa dikontrol sepenuhnya oleh peneliti, seperti Garbage Collection (GC) memori dan optimasi optimistik Just-In-Time (JIT) kompiler. Untuk mengatasinya, strategi mitigasi yang dilakukan adalah mengeksekusi warm-up run (pemotongan iterasi pertama) agar JIT selesai bekerja, melakukan force garbage collection di sela-sela perpindahan skenario, serta meningkatkan jumlah pengujian hingga 30 kali iterasi lalu mengambil nilai median untuk mengeliminasi pencilan data akibat noise sistem.

---

## Latihan 3 — Ablation Study Planning

Jika sistem memiliki 3 komponen utama, rencanakan ablation study.

> **Panduan jumlah kondisi:** Untuk 3 komponen (A, B, C), kondisi minimal yang direkomendasikan:
> Full + (-A) + (-B) + (-C) = **4 kondisi dasar**. Jika waktu memungkinkan, tambahkan kombinasi ganda: (-A,-B), (-A,-C), (-B,-C) = **7 kondisi**. Sesuaikan dengan *computational cost* dan tenggat waktu penelitian.

Kondisi	Komponen A (WASM)	Komponen B (Web Workers)	Komponen C (Streaming)	Hasil yang Diharapkan
Full	✅ Rust-compiled WASM      	✅ Terisolasi di Background      	✅ Data dipotong per 64 KB	Performa Optimal: TBT < 50 ms dan Latency minimum pada payload 10 MB.
– A	❌ JavaScript murni	      ✅ Terisolasi di Background	      ✅ Data dipotong per 64 KB	Latensi komputasi membengkak karena JS murni lambat, tetapi TBT tetap aman (<50 ms) karena berada di background thread.
– B	✅ Rust-compiled WASM	      ❌ Berjalan di Main UI Thread      	✅ Data dipotong per 64 KB	Latensi komputasi sangat cepat, namun TBT hancur (>500 ms) karena Main Thread terkunci selama proses enkripsi.
– C	✅ Rust-compiled WASM	      ✅ Terisolasi di Background      	❌ Data dimuat utuh (Bulk memory)	TBT aman, tetapi terjadi lonjakan drastis pada penggunaan RAM b

**Komponen mana yang diprediksi paling berkontribusi?** Komponen B (Web Workers)
**Mengapa?**
> Karena esensi dari metrik Total Blocking Time (TBT) adalah mengukur durasi hambatan pada Main UI Thread. Secepat apa pun fungsi matematika enkripsi dipercepat oleh WebAssembly (Komponen A), jika komputasi tersebut tetap dieksekusi di atas Main Thread, browser akan tetap menganggapnya sebagai Long Task yang memicu UI freeze. Memindahkan beban kerja keluar dari Main Thread melalui Web Workers (Komponen B) adalah kunci utama yang secara langsung mengeliminasi nilai TBT hingga mendekati 0 ms.

---

## Refleksi

> Apa risiko jika sistem dibangun seperti produk (monolitik, fitur lengkap) lalu baru dilakukan eksperimen? Mengapa arsitektur modular penting untuk riset?

**Jawaban:**
Jika sistem dibangun seperti produk monolitik yang langsung jadi dengan segala fiturnya (misal: langsung digabung dengan sistem autentikasi, penyimpanan database local, dan animasi UI yang kompleks), peneliti akan menghadapi masalah Confounding Variables (Variabel Pengganggu). Ketika hasil eksperimen menunjukkan performa web melambat, peneliti tidak akan bisa mendiagnosis dengan pasti apakah perlambatan tersebut murni disebabkan oleh algoritma enkripsinya, akibat proses baca-tulis database, atau karena beban rendering komponen UI.

    Di sinilah letak perbedaan fundamental arsitektur riset dan arsitektur produk. Arsitektur modular sangat krusial dalam penelitian karena memungkinkan kita melakukan isolasi variabel secara ketat. Kita bisa mempreteli atau menukar satu komponen spesifik (seperti dalam Ablation Study atau penggantian mesin IV) tanpa mengganggu fungsi komponen lainnya. Sifat modular ini menjamin tingkat validitas internal yang tinggi, karena kita dapat membuktikan secara empiris hubungan sebab-akibat langsung antara perubahan arsitektur dengan variasi metrik performa yang dihasilkan.

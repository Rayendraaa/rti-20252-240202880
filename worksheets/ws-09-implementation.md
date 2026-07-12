# WS-09: Implementation & Environment

> **Bab 9 — Implementasi Riset & Kontrol Lingkungan**
> **Tema Eksperimen: Implementasi End-to-End Encryption (E2EE) pada Sistem Chat/Pesan Berbasis PHP**

---

## Konteks Eksperimen

Eksperimen ini mengukur **konsistensi performa** implementasi End-to-End Encryption (E2EE) menggunakan skema hybrid **RSA (pertukaran kunci) + AES-256-CBC (enkripsi pesan)** pada aplikasi berbasis PHP. Variabel yang diukur: waktu enkripsi, waktu dekripsi, dan integritas pesan (hasil dekripsi harus identik dengan plaintext asli) pada ukuran pesan yang bervariasi.

---

## Template A.9 — Dokumentasi Setup Eksperimen

```
EXPERIMENT SETUP DOCUMENTATION

Hardware:
  CPU     : (isi sesuai laptop Anda, mis. Intel Core i5 generasi 10+)
  RAM     : (isi sesuai perangkat, mis. 8 GB)
  GPU     : Tidak relevan (proses kriptografi berbasis CPU)
  Storage : (isi sesuai perangkat, mis. SSD 256 GB)

Software:
  OS        : Windows 10/11
  Runtime   : PHP 8.x (bundled XAMPP)
  Framework : CodeIgniter (dengan ekstensi openssl aktif)

Dependencies:
| Library         | Version        | Sumber                  | Hash/Checksum        |
|-----------------|----------------|--------------------------|-----------------------|
| PHP OpenSSL ext | bundled PHP 8.x| XAMPP installer          | (sesuai installer XAMPP) |
| CodeIgniter     | 3.x / 4.x      | codeigniter.com          | (sesuai rilis resmi)  |
| Composer (opsional) | 2.x        | getcomposer.org          | (sesuai installer)    |

Konfigurasi:
  Config file     : config/encryption.php (kunci RSA, ukuran blok AES, mode CBC)
  Random seed     : openssl_random_pseudo_bytes() untuk IV — dicatat per-run agar dapat direplikasi
  Hyperparameters : Panjang kunci RSA = 2048 bit, algoritma AES = AES-256-CBC

Reproducibility Check:
  [x] Dependency terdokumentasi (requirements.txt / lock file) → dicatat versi PHP & ekstensi OpenSSL
  [x] Seed ditetapkan di semua level (Python, NumPy, framework) → IV dan keypair RSA disimpan per-run
  [x] Config di version control → config/encryption.php di-commit ke Git
  [ ] README instruksi reproduksi lengkap → belum ditulis, lihat Latihan 3
```

---

## Latihan 1 — Environment Specification

| Komponen | Spesifikasi |
|----------|------------|
| CPU | *(isi sesuai laptop Anda)* |
| RAM | *(isi sesuai laptop Anda)* |
| GPU | Tidak digunakan — proses enkripsi/dekripsi murni CPU-bound |
| OS | Windows 10/11 |
| Runtime | PHP 8.x (via XAMPP) |
| Framework | CodeIgniter + ekstensi OpenSSL bawaan PHP |
| Random Seed | IV (Initialization Vector) 16-byte dari `openssl_random_pseudo_bytes()`, dicatat per-run |

**Dependencies (minimal 5):**

| Library | Version | Alasan Dibutuhkan |
|---------|---------|-------------------|
| PHP openssl extension | bundled PHP 8.x | Fungsi inti enkripsi AES & RSA (`openssl_encrypt`, `openssl_public_encrypt`) |
| CodeIgniter | 3.x/4.x | Kerangka kerja aplikasi tempat modul E2EE diintegrasikan |
| PHP hash extension | bundled PHP 8.x | Verifikasi integritas pesan (HMAC-SHA256) |
| XAMPP (Apache + MySQL) | versi terpasang | Lingkungan server lokal untuk menjalankan eksperimen |
| Composer (opsional) | 2.x | Manajemen dependency tambahan jika memakai library kripto pihak ketiga |

---

## Latihan 2 — Repeatability Test Plan

Rancangan: jalankan fungsi enkripsi-dekripsi pesan yang sama (misal pesan 1 KB) sebanyak 3× pada environment yang sama, dengan IV yang **sama** (bukan random) agar hasil ciphertext dapat dibandingkan secara identik.

| Run | Seed (IV) | Metrik Utama | Hasil Sama? |
|-----|------|-------------|-------------|
| 1 | IV tetap: `a1b2c3...` (16 byte fixed) | Waktu enkripsi (ms) + waktu dekripsi (ms) + kecocokan plaintext hasil dekripsi | — |
| 2 | IV tetap sama dengan Run 1 | Waktu enkripsi (ms) + waktu dekripsi (ms) + kecocokan plaintext hasil dekripsi | [ ] Ya / [ ] Tidak |
| 3 | IV tetap sama dengan Run 1 | Waktu enkripsi (ms) + waktu dekripsi (ms) + kecocokan plaintext hasil dekripsi | [ ] Ya / [ ] Tidak |

**Jika hasil berbeda, kemungkinan penyebab (khusus konteks E2EE):**

> - **Thermal throttling** — enkripsi RSA berulang pada laptop tanpa pendingin memadai bisa memperlambat run berikutnya
> - **Background process** — Apache/MySQL XAMPP, antivirus, atau browser lain yang aktif bersamaan memengaruhi waktu eksekusi
> - **IV/random tidak dikunci** — jika IV dibiarkan `openssl_random_pseudo_bytes()` tanpa dicatat, ciphertext akan selalu berbeda meski plaintext & key sama → bukan berarti tidak repeatable, tapi harus didokumentasikan sebagai *by design*
> - **Cache OPcache PHP** — request pertama ke script lebih lambat karena kompilasi, request berikutnya lebih cepat karena OPcache

**Checklist kontrol yang sudah diterapkan:**
- [x] Random seed (IV) di-set tetap untuk keperluan pengujian repeatability
- [ ] Tidak ada background process yang mengganggu → perlu tutup aplikasi lain saat pengujian
- [ ] Cache dibersihkan antar-run → restart Apache/OPcache sebelum tiap run
- [x] Config file yang sama (`config/encryption.php`) untuk semua run

---

## Latihan 3 — README Eksperimen

```
# Judul Eksperimen: Pengujian Konsistensi Performa End-to-End Encryption (E2EE) Hybrid RSA+AES pada Aplikasi PHP

## 1. Environment
> Windows 10/11, PHP 8.x (XAMPP), CodeIgniter, ekstensi OpenSSL aktif.
> (Salin detail lengkap dari Latihan 1)

## 2. Installation
> 1. Install XAMPP, aktifkan Apache & ekstensi openssl di php.ini
> 2. Clone/copy project ke folder htdocs
> 3. Jalankan `composer install` jika ada dependency tambahan

## 3. Data
> Pesan uji berupa teks dengan variasi ukuran: 100 byte, 1 KB, 10 KB, 100 KB.
> Disimpan sebagai file .txt di folder /test-data agar identik untuk setiap run.

## 4. Execution
> Jalankan script `test_encryption.php` melalui browser (localhost/project/test_encryption.php)
> atau via CLI: `php test_encryption.php`

## 5. Configuration
> File: config/encryption.php
> Parameter kunci: RSA key size = 2048 bit, algoritma AES = AES-256-CBC, IV = fixed untuk uji repeatability

## 6. Expected Output
> Output berupa tabel: ukuran pesan | waktu enkripsi (ms) | waktu dekripsi (ms) | ciphertext (base64) | status kecocokan plaintext (match/mismatch)
```

---

## Refleksi

> Apakah eksperimen Anda saat ini bisa direproduksi oleh orang lain tanpa bantuan Anda? Komponen apa yang masih hilang?

**Contoh jawaban:**
Eksperimen E2EE ini sudah mencapai tahap **repeatability** karena dijalankan pada environment, kode, dan IV yang sama menghasilkan ciphertext identik. Namun belum sepenuhnya **reproducible** oleh orang lain karena beberapa komponen masih perlu dilengkapi.

**Level saat ini:** [x] Repeatability / [ ] Reproducibility / [ ] Belum keduanya

**Komponen yang belum terdokumentasi:**
> - Versi PHP dan ekstensi openssl yang persis digunakan belum dicatat dengan nomor versi eksplisit
> - Belum ada `composer.lock` atau daftar versi library pihak ketiga jika digunakan
> - Prosedur pembersihan cache (OPcache) antar-run belum distandarkan dalam script otomatis
> - README belum menyertakan contoh output nyata (angka waktu enkripsi/dekripsi aktual) sebagai pembanding

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
  Runtime   : PHP 8.3.6 (bundled XAMPP)
  Framework : CodeIgniter (dengan ekstensi openssl aktif)

Dependencies:
| Library         | Version        | Sumber                  | Hash/Checksum        |
|-----------------|----------------|--------------------------|-----------------------|
| PHP OpenSSL ext | OpenSSL 3.0.13 | XAMPP installer          | (sesuai installer XAMPP) |
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
| Runtime | PHP 8.3.6 (via XAMPP) |
| Framework | CodeIgniter + ekstensi OpenSSL bawaan PHP (OpenSSL 3.0.13) |
| Random Seed | IV & AES key dikunci (fixed) khusus untuk uji repeatability — lihat Latihan 2 |

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

Rancangan: jalankan fungsi enkripsi-dekripsi pesan yang sama (ukuran 100 B, 1 KB, 10 KB, 100 KB) sebanyak 3× pada environment yang sama, dengan IV dan AES key yang **dikunci** (fixed, bukan random) agar hasil ciphertext dapat dibandingkan secara identik. Dijalankan menggunakan `test_encryption.php` (lampiran di bawah).

**Hasil aktual (contoh run — angka akan sedikit berbeda di perangkat Anda):**

| Ukuran Pesan | Run | Enkripsi (ms) | Dekripsi (ms) | Integritas | Ciphertext Identik dgn Run #1? |
|---|---|---|---|---|---|
| 100 B  | 1 | 0.916 | 2.056 | OK | — |
| 100 B  | 2 | 0.674 | 2.012 | OK | Ya |
| 100 B  | 3 | 0.616 | 1.901 | OK | Ya |
| 1 KB   | 1 | 0.633 | 1.955 | OK | — |
| 1 KB   | 2 | 0.607 | 1.966 | OK | Ya |
| 1 KB   | 3 | 0.617 | 1.943 | OK | Ya |
| 10 KB  | 1 | 0.685 | 2.079 | OK | — |
| 10 KB  | 2 | 0.680 | 2.105 | OK | Ya |
| 10 KB  | 3 | 0.677 | 2.083 | OK | Ya |
| 100 KB | 1 | 1.478 | 2.461 | OK | — |
| 100 KB | 2 | 1.447 | 2.434 | OK | Ya |
| 100 KB | 3 | 1.440 | 2.521 | OK | Ya |

**Kesimpulan run di atas:** [x] Ya — repeatable / [ ] Tidak

Ringkasan sederhana (pesan 1 KB, sesuai format tabel template asli):

| Run | Seed | Metrik Utama | Hasil Sama? |
|-----|------|-------------|-------------|
| 1 | IV & AES key fixed | Enkripsi 0.633 ms / Dekripsi 1.955 ms | — |
| 2 | IV & AES key fixed (sama dgn Run 1) | Enkripsi 0.607 ms / Dekripsi 1.966 ms | [x] Ya |
| 3 | IV & AES key fixed (sama dgn Run 1) | Enkripsi 0.617 ms / Dekripsi 1.943 ms | [x] Ya |

**Jika hasil berbeda, kemungkinan penyebab (khusus konteks E2EE):**

> - **Thermal throttling** — enkripsi RSA berulang pada laptop tanpa pendingin memadai bisa memperlambat run berikutnya
> - **Background process** — Apache/MySQL XAMPP, antivirus, atau browser lain yang aktif bersamaan memengaruhi waktu eksekusi
> - **IV/random tidak dikunci** — jika IV dibiarkan `openssl_random_pseudo_bytes()` tanpa dicatat, ciphertext akan selalu berbeda meski plaintext & key sama → bukan berarti tidak repeatable, tapi harus didokumentasikan sebagai *by design*
> - **Cache OPcache PHP** — request pertama ke script lebih lambat karena kompilasi, request berikutnya lebih cepat karena OPcache

**Checklist kontrol yang sudah diterapkan:**
- [x] Random seed (IV & AES key) di-set tetap untuk keperluan pengujian repeatability
- [x] RSA keypair disimpan ke `rsa_keypair.json` agar run ke-2/ke-3 memakai keypair yang sama
- [ ] Tidak ada background process yang mengganggu → perlu tutup aplikasi lain saat pengujian ulang di perangkat sendiri
- [ ] Cache dibersihkan antar-run → restart Apache/OPcache sebelum tiap run
- [x] Config file yang sama (konstanta di `EncryptionHelper.php`) untuk semua run

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
> Output berupa tabel di terminal/browser seperti berikut (contoh aktual):
>
> ```
> PHP Version   : 8.3.6
> OpenSSL       : OpenSSL 3.0.13 30 Jan 2024
> AES Method    : aes-256-cbc
> RSA Key Bits  : 2048
>
> Ukuran  Run  Enkripsi(ms)  Dekripsi(ms)  Integritas  Ciphertext identik antar-run?
> --------------------------------------------------------------------------------
> 100 B   #1   0.916         2.056         OK          -
> 100 B   #2   0.674         2.012         OK          Ya
> 100 B   #3   0.616         1.901         OK          Ya
> ...
> ```
> Kolom "Integritas" harus selalu "OK" (HMAC cocok & plaintext hasil dekripsi = plaintext asli).
> Kolom "Ciphertext identik antar-run?" harus selalu "Ya" karena IV & AES key dikunci.
```

---

## Lampiran — Kode Program

### EncryptionHelper.php

```php
<?php
/**
 * EncryptionHelper.php
 * ---------------------------------------------------------
 * Modul E2EE hybrid: RSA (pertukaran kunci) + AES-256-CBC (enkripsi pesan).
 * Dibuat config-driven agar sesuai prinsip riset di WS-09
 * (parameter eksplisit, bukan hardcode acak, seed/IV dapat dikunci).
 * ---------------------------------------------------------
 */

class EncryptionHelper
{
    // ====== KONFIGURASI (setara config/encryption.php di worksheet) ======
    const AES_METHOD   = 'aes-256-cbc';
    const RSA_KEY_BITS = 2048;

    /**
     * Generate pasangan kunci RSA.
     * Dalam riset sungguhan, key ini idealnya dibuat sekali lalu disimpan
     * ke file (private.pem / public.pem) agar run berikutnya memakai
     * key yang sama -> mendukung repeatability.
     */
    public static function generateRsaKeyPair(): array
    {
        $res = openssl_pkey_new([
            'private_key_bits' => self::RSA_KEY_BITS,
            'private_key_type' => OPENSSL_KEYTYPE_RSA,
        ]);

        openssl_pkey_export($res, $privateKey);
        $publicKey = openssl_pkey_get_details($res)['key'];

        return [
            'private' => $privateKey,
            'public'  => $publicKey,
        ];
    }

    /**
     * Enkripsi pesan (hybrid):
     * 1. Buat AES key acak (session key) + IV
     * 2. Enkripsi pesan dengan AES-256-CBC
     * 3. Enkripsi AES key dengan RSA public key (simulasi "kirim kunci ke penerima")
     *
     * $fixedIv: jika diisi (16 byte), IV tidak di-random -> dipakai khusus
     * untuk uji REPEATABILITY (Latihan 2), supaya ciphertext bisa
     * dibandingkan identik antar-run.
     */
    public static function encryptMessage(string $plaintext, string $rsaPublicKey, ?string $fixedIv = null, ?string $fixedAesKey = null): array
    {
        $aesKey = $fixedAesKey ?? openssl_random_pseudo_bytes(32); // 256-bit
        $iv     = $fixedIv ?? openssl_random_pseudo_bytes(openssl_cipher_iv_length(self::AES_METHOD));

        $ciphertext = openssl_encrypt($plaintext, self::AES_METHOD, $aesKey, OPENSSL_RAW_DATA, $iv);

        // HMAC untuk verifikasi integritas pesan
        $hmac = hash_hmac('sha256', $ciphertext, $aesKey);

        // Enkripsi AES key dengan RSA public key penerima
        openssl_public_encrypt($aesKey, $encryptedAesKey, $rsaPublicKey);

        return [
            'ciphertext'      => base64_encode($ciphertext),
            'iv'              => base64_encode($iv),
            'hmac'            => $hmac,
            'encrypted_key'   => base64_encode($encryptedAesKey),
        ];
    }

    /**
     * Dekripsi pesan: kebalikan dari encryptMessage().
     * Mengembalikan plaintext, dan status kecocokan HMAC (integritas).
     */
    public static function decryptMessage(array $payload, string $rsaPrivateKey): array
    {
        $ciphertext    = base64_decode($payload['ciphertext']);
        $iv            = base64_decode($payload['iv']);
        $encryptedKey  = base64_decode($payload['encrypted_key']);

        openssl_private_decrypt($encryptedKey, $aesKey, $rsaPrivateKey);

        $plaintext = openssl_decrypt($ciphertext, self::AES_METHOD, $aesKey, OPENSSL_RAW_DATA, $iv);

        $expectedHmac = hash_hmac('sha256', $ciphertext, $aesKey);
        $integrityOk  = hash_equals($expectedHmac, $payload['hmac']);

        return [
            'plaintext'    => $plaintext,
            'integrity_ok' => $integrityOk,
        ];
    }
}
```

### test_encryption.php

```php
<?php
/**
 * test_encryption.php
 * ---------------------------------------------------------
 * Runner untuk WS-09 Latihan 2 & 3.
 * Bisa dijalankan via CLI:   php test_encryption.php
 * atau via browser:          http://localhost/<folder-project>/test_encryption.php
 *
 * Yang diukur:
 *  - Waktu enkripsi & dekripsi (ms) untuk beberapa ukuran pesan
 *  - Repeatability: 3x run dengan IV & AES key TETAP (dikunci) supaya
 *    ciphertext bisa dibandingkan identik antar-run
 * ---------------------------------------------------------
 */

require_once __DIR__ . '/EncryptionHelper.php';

$isCli = (php_sapi_name() === 'cli');
if (!$isCli) {
    header('Content-Type: text/plain; charset=utf-8');
}

echo "===========================================================\n";
echo " WS-09 - Uji Performa & Repeatability E2EE (RSA + AES-256-CBC)\n";
echo "===========================================================\n\n";

// ---------------------------------------------------------------
// 1. Setup environment (dicatat agar sesuai Template A.9)
// ---------------------------------------------------------------
echo "PHP Version   : " . PHP_VERSION . "\n";
echo "OpenSSL       : " . (extension_loaded('openssl') ? OPENSSL_VERSION_TEXT : 'TIDAK AKTIF') . "\n";
echo "AES Method    : " . EncryptionHelper::AES_METHOD . "\n";
echo "RSA Key Bits  : " . EncryptionHelper::RSA_KEY_BITS . "\n\n";

// ---------------------------------------------------------------
// 2. Generate RSA keypair SEKALI, lalu simpan ke file supaya
//    bisa dipakai ulang di run berikutnya (mendukung reproducibility).
// ---------------------------------------------------------------
$keyFile = __DIR__ . '/rsa_keypair.json';

if (file_exists($keyFile)) {
    $keys = json_decode(file_get_contents($keyFile), true);
    echo "[Keypair] Menggunakan keypair RSA yang sudah tersimpan (rsa_keypair.json)\n\n";
} else {
    $keys = EncryptionHelper::generateRsaKeyPair();
    file_put_contents($keyFile, json_encode($keys));
    echo "[Keypair] Keypair RSA baru dibuat & disimpan ke rsa_keypair.json\n\n";
}

// ---------------------------------------------------------------
// 3. Latihan 1 & 2: uji dengan beberapa ukuran pesan
//    IV & AES key DIKUNCI (fixed) supaya hasil bisa dibandingkan
//    antar-run secara identik -> uji REPEATABILITY.
// ---------------------------------------------------------------
$fixedIv     = str_repeat("\x01", 16); // 16 byte fixed IV (contoh sederhana, TIDAK untuk produksi)
$fixedAesKey = str_repeat("\x02", 32); // 32 byte fixed AES key (contoh sederhana, TIDAK untuk produksi)

$messageSizes = [
    '100 B'  => str_repeat('A', 100),
    '1 KB'   => str_repeat('A', 1024),
    '10 KB'  => str_repeat('A', 10 * 1024),
    '100 KB' => str_repeat('A', 100 * 1024),
];

$numRuns = 3; // sesuai Latihan 2: jalankan 3x

echo str_pad("Ukuran", 8) . str_pad("Run", 5) . str_pad("Enkripsi(ms)", 14) .
     str_pad("Dekripsi(ms)", 14) . str_pad("Integritas", 12) . "Ciphertext identik antar-run?\n";
echo str_repeat('-', 80) . "\n";

foreach ($messageSizes as $label => $message) {
    $ciphertexts = [];

    for ($run = 1; $run <= $numRuns; $run++) {

        $t0 = microtime(true);
        $payload = EncryptionHelper::encryptMessage($message, $keys['public'], $fixedIv, $fixedAesKey);
        $encMs = round((microtime(true) - $t0) * 1000, 3);

        $t0 = microtime(true);
        $result = EncryptionHelper::decryptMessage($payload, $keys['private']);
        $decMs = round((microtime(true) - $t0) * 1000, 3);

        $integrityOk = $result['integrity_ok'] && ($result['plaintext'] === $message);
        $ciphertexts[$run] = $payload['ciphertext'];

        $sameAsRun1 = ($run === 1) ? '-' : (($ciphertexts[$run] === $ciphertexts[1]) ? 'Ya' : 'Tidak');

        echo str_pad($label, 8) . str_pad("#$run", 5) . str_pad($encMs, 14) .
             str_pad($decMs, 14) . str_pad($integrityOk ? 'OK' : 'GAGAL', 12) .
             $sameAsRun1 . "\n";
    }
    echo str_repeat('-', 80) . "\n";
}

echo "\nCatatan:\n";
echo "- 'Ciphertext identik antar-run' seharusnya SELALU 'Ya' karena IV & AES key dikunci.\n";
echo "  Jika 'Tidak' muncul, ada sumber non-determinisme yang perlu diselidiki\n";
echo "  (lihat daftar kemungkinan penyebab di Latihan 2 worksheet).\n";
echo "- Waktu enkripsi/dekripsi bisa sedikit bervariasi antar-run karena\n";
echo "  faktor sistem (background process, thermal throttling, dll),\n";
echo "  walau ciphertext-nya tetap identik.\n";
echo "- Untuk pemakaian PRODUKSI (bukan riset), JANGAN PERNAH memakai IV/key\n";
echo "  yang fixed seperti di script ini — gunakan openssl_random_pseudo_bytes()\n";
echo "  murni acak setiap pesan.\n";
```

---

## Refleksi

> Apakah eksperimen Anda saat ini bisa direproduksi oleh orang lain tanpa bantuan Anda? Komponen apa yang masih hilang?

**Contoh jawaban:**
Eksperimen E2EE ini sudah mencapai tahap **repeatability** — dibuktikan dari hasil run di Latihan 2, di mana ciphertext Run #2 dan #3 identik dengan Run #1 di semua ukuran pesan (100 B hingga 100 KB), dan integritas pesan selalu "OK". Namun belum sepenuhnya **reproducible** oleh orang lain karena beberapa komponen masih perlu dilengkapi.

**Level saat ini:** [x] Repeatability / [ ] Reproducibility / [ ] Belum keduanya

**Komponen yang belum terdokumentasi:**
> - Versi PHP dan ekstensi openssl yang persis digunakan belum dicatat dengan nomor versi eksplisit
> - Belum ada `composer.lock` atau daftar versi library pihak ketiga jika digunakan
> - Prosedur pembersihan cache (OPcache) antar-run belum distandarkan dalam script otomatis
> - README belum menyertakan contoh output nyata (angka waktu enkripsi/dekripsi aktual) sebagai pembanding

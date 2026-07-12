# WS-10: Experiment Execution & Data Collection

> **Bab 10 — Eksekusi Eksperimen & Pengumpulan Data**
> **Tema Eksperimen (lanjutan WS-09): Multiple-Run Execution untuk E2EE Hybrid RSA + AES-256-CBC**

---

## Konteks Eksperimen

Melanjutkan WS-09, eksperimen ini menjalankan implementasi E2EE **berulang kali (multiple run)** untuk 4 skenario ukuran pesan (100 B, 1 KB, 10 KB, 100 KB), masing-masing **5 run dengan seed berbeda yang sudah ditentukan sebelum eksekusi**, ditambah 1 skenario ekstrem (5 MB) untuk mendemonstrasikan protokol anomali. Semua run dicatat ke file `data_log.csv` — bukan sekadar output di layar.

---

## Template A.10 — Execution Plan & Data Log (terisi)

```
EXECUTION PLAN

| Run # | Skenario | Seed | Parameter | Status | Waktu Enc/Dec (ms) | Output File |
|-------|----------|------|-----------|--------|---------------------|-------------|
| run-001 | 1KB   | 42    | AES-256-CBC, RSA-2048 | OK | 1.701 / 2.535 | data_log.csv |
| run-002 | 10KB  | 123   | AES-256-CBC, RSA-2048 | OK | 0.855 / 2.075 | data_log.csv |
| run-003 | 100B  | 777   | AES-256-CBC, RSA-2048 | OK | 0.603 / 3.113 | data_log.csv |
| ...     | ...   | ...   | ...                   | ...| ...           | ...         |
| run-021 | 5MB (anomali) | 42 | AES-256-CBC, RSA-2048 | ANOMALY_DOCUMENTED | 127.308 / 68.699 | data_log.csv |

Jumlah runs per skenario : 5 (skenario utama) + 1 (skenario anomali khusus)
Total runs               : 21

DATA LOG (contoh 1 run — run-001):
  Run ID    : run-001
  Timestamp : 2026-07-12T11:24:16+00:00
  Skenario  : 1KB
  Input     : pesan teks 1024 byte
  Output    : ciphertext base64 (AES-256-CBC) + encrypted_key (RSA-2048), integrity_ok=true
  Anomali   : (tidak ada)
  Catatan   : run normal, digunakan untuk hitung mean/std skenario 1KB
```

---

## Latihan 1 — Execution Plan

| Run # | Skenario | Seed | Parameter Kunci | Status |
|-------|----------|------|----------------|--------|
| 1 | Pesan 100 B, RSA-2048 + AES-256-CBC | 777 | key_size=2048, aes=256-cbc | Selesai (OK) |
| 2 | Pesan 1 KB, RSA-2048 + AES-256-CBC | 42 | key_size=2048, aes=256-cbc | Selesai (OK) |
| 3 | Pesan 10 KB, RSA-2048 + AES-256-CBC | 123 | key_size=2048, aes=256-cbc | Selesai (OK) |
| 4 | Pesan 100 KB, RSA-2048 + AES-256-CBC | 31337 | key_size=2048, aes=256-cbc | Selesai (OK) |
| 5 | Pesan 5 MB (skenario anomali/stress test) | 42 | key_size=2048, aes=256-cbc | Selesai (ANOMALY_DOCUMENTED) |

**Total skenario:** 5 (100 B, 1 KB, 10 KB, 100 KB, 5 MB-anomali)
**Run per skenario:** 5 run untuk 4 skenario utama, 1 run untuk skenario anomali khusus
**Total run keseluruhan:** 21 run

**Urutan eksekusi:** Urutan 20 run skenario utama **diacak (shuffle)** dengan seed urutan terpisah (999) sebelum dijalankan — bukan berurutan per skenario — untuk menghindari bias sistematis (mis. thermal throttling yang selalu menimpa skenario yang dijalankan terakhir). Skenario anomali (5 MB) sengaja ditempatkan di akhir sebagai stress-test terpisah.

**Daftar seed pre-determined yang dipakai (digilir per run):** `42, 123, 777, 2025, 31337, 8, 99, 555, 71, 2024`

---

## Latihan 2 — Data Log Terstruktur

**Identitas:**
| Field | Contoh (dari run nyata) |
|-------|--------|
| Run ID | `run-001` |
| Timestamp | `2026-07-12T11:24:16+00:00` |
| Skenario | `1KB` |

**Konfigurasi:**
| Field | Contoh (dari run nyata) |
|-------|--------|
| Seed | `42` |
| Code version | `e2ee-experiment-v1.0` |
| Parameter kripto | `AES-256-CBC, RSA-2048` |

**Hasil:**
| Metrik | Tipe Data | Range Valid |
|--------|----------|-------------|
| enc_time_ms | float | > 0 |
| dec_time_ms | float | > 0 |
| integrity_ok | boolean | true / false |
| message_size_bytes | integer | > 0 |
| status | string (enum) | OK / ANOMALY / ANOMALY_DOCUMENTED / CRASH |

**Format output:** [x] CSV / [ ] JSON / [ ] Database / [ ] Lainnya: ____

**Ringkasan statistik hasil (mean & std waktu enkripsi, n=5 per skenario):**

| Skenario | n | Mean Enc (ms) | Std Enc (ms) |
|---|---|---|---|
| 100 B | 5 | 0.588 | 0.015 |
| 1 KB | 5 | 0.818 | 0.442 |
| 10 KB | 5 | 0.724 | 0.073 |
| 100 KB | 5 | 1.864 | 0.673 |
| 5 MB (anomali, n=1) | 1 | 127.308 | 0.000 |

---

## Latihan 3 — Anomaly Protocol

| Jenis Anomali | Contoh | Tindakan |
|---------------|--------|----------|
| Run gagal (crash) | RSA gagal dekripsi karena `encrypted_key` korup/tidak cocok dengan private key | Tangkap dengan try-catch, status dicatat `CRASH`, catat pesan exception di kolom `anomaly_note`, JANGAN dihapus dari log, lakukan re-run dengan parameter sama untuk cek apakah konsisten |
| Hasil ekstrem | run-021: pesan 5 MB menghasilkan enc=127.308 ms, dec=68.699 ms — jauh di atas skenario lain (100 KB hanya ~1.9 ms) | Ditandai `ANOMALY_DOCUMENTED`, TETAP disimpan di `data_log.csv` sebagai temuan valid (batas skalabilitas AES-CBC pada pesan besar), bukan dihapus sebagai outlier |
| Waktu eksekusi anomali | Salah satu run 1KB (run-001, seed 42) enc=1.701 ms — jauh di atas rata-rata skenario 1KB lain (~0.6 ms) | Investigasi: kemungkinan warm-up/first-run overhead (OPcache belum "panas"). Didokumentasikan di catatan, TIDAK dihapus, tapi ditandai sebagai kandidat outlier untuk dianalisis terpisah |
| Inkonsistensi dengan run lain | Jika suatu saat `integrity_ok=false` muncul pada run manapun (di eksperimen ini semua 21 run integrity_ok=true) | Prioritas tinggi: hentikan run berikutnya, cek apakah bug pada IV/key generation, dokumentasikan seed & kondisi persis sebelum re-run |

**Prinsip:** Detect → Investigate → Document → Decide

**Catatan penerapan nyata:** Pada eksperimen ini, run-021 (5 MB) sengaja disisipkan sebagai stress-test dan **tidak dihapus** dari `data_log.csv` meskipun hasilnya jauh berbeda dari skenario lain — ini konsisten dengan prinsip "anomali = dokumentasi, bukan hapus".

---

## Refleksi

> Pernahkah Anda melaporkan hasil riset/tugas dari single run? Apa risikonya? Bagaimana multiple run mengubah kepercayaan terhadap hasil?

**Pengalaman sebelumnya:**
> Pada WS-09, pengujian awal E2EE hanya dijalankan sekali per ukuran pesan (single run) untuk mengecek fungsi enkripsi/dekripsi bekerja. Risikonya: waktu eksekusi run-001 skenario 1 KB (1.701 ms) jauh lebih tinggi dari rata-rata skenario 1 KB sesungguhnya (0.818 ms, std 0.442 ms) — kalau hanya dilaporkan dari 1 run, kesimpulan performa bisa menyesatkan.

**Yang akan dilakukan berbeda:**
> Ke depan, setiap klaim performa (mis. "enkripsi pesan 1 KB memakan waktu X ms") akan selalu disertai jumlah run (n=5 atau lebih), mean, dan standar deviasi — bukan angka tunggal. Seed untuk tiap run juga akan ditentukan di awal (pre-determined) dan dicatat di log CSV, serta anomali seperti skenario 5 MB akan tetap disimpan dan didokumentasikan, bukan dibuang sebagai outlier.

---

## Lampiran — Kode Program

### execution_runner.php

```php
<?php
/**
 * execution_runner.php
 * ---------------------------------------------------------
 * WS-10 - Execution Plan & Data Collection
 * Tema: End-to-End Encryption (RSA + AES-256-CBC)
 *
 * Menjalankan MULTIPLE RUN (bukan single run) per skenario,
 * dengan seed yang PRE-DETERMINED (ditentukan sebelum eksekusi,
 * bukan diacak saat run), lalu mencatat semua run ke file CSV
 * terstruktur (data_log.csv) -- bukan sekadar stdout.
 * ---------------------------------------------------------
 */

require_once __DIR__ . '/EncryptionHelper.php';

$isCli = (php_sapi_name() === 'cli');
if (!$isCli) {
    header('Content-Type: text/plain; charset=utf-8');
}

// ---------------------------------------------------------------
// 1. EXECUTION PLAN — ditentukan SEBELUM eksekusi (bukan on-the-fly)
// ---------------------------------------------------------------
$scenarios = [
    '100B'  => str_repeat('A', 100),
    '1KB'   => str_repeat('A', 1024),
    '10KB'  => str_repeat('A', 10 * 1024),
    '100KB' => str_repeat('A', 100 * 1024),
];

$runsPerScenario = 5; // minimum 5-10 run per skenario sesuai materi
$seeds = [42, 123, 777, 2025, 31337, 8, 99, 555, 71, 2024]; // pre-determined, dipakai berurutan

$codeVersion = 'e2ee-experiment-v1.0'; // representasi "commit hash" untuk konteks ini

// Urutan eksekusi di-randomisasi (counterbalancing) untuk menghindari bias
// urutan (mis. efek thermal throttling selalu mengenai skenario yang sama).
$executionQueue = [];
foreach ($scenarios as $scenarioName => $message) {
    for ($i = 0; $i < $runsPerScenario; $i++) {
        $executionQueue[] = ['scenario' => $scenarioName, 'message' => $message, 'run_index' => $i];
    }
}

mt_srand(999); // seed untuk randomisasi URUTAN saja (bukan seed kripto), agar urutan run juga reproducible
shuffle($executionQueue);

// ---------------------------------------------------------------
// 2. Siapkan RSA keypair (dipakai sama untuk semua run -> reproducibility)
// ---------------------------------------------------------------
$keyFile = __DIR__ . '/rsa_keypair.json';
if (file_exists($keyFile)) {
    $keys = json_decode(file_get_contents($keyFile), true);
} else {
    $keys = EncryptionHelper::generateRsaKeyPair();
    file_put_contents($keyFile, json_encode($keys));
}

// ---------------------------------------------------------------
// 3. Siapkan file log CSV terstruktur
// ---------------------------------------------------------------
$logFile = __DIR__ . '/data_log.csv';
$csv = fopen($logFile, 'w');
fputcsv($csv, [
    'run_id', 'timestamp', 'scenario', 'seed', 'code_version',
    'message_size_bytes', 'enc_time_ms', 'dec_time_ms',
    'integrity_ok', 'status', 'anomaly_note'
]);

// Injeksi 1 anomali BUATAN secara sengaja untuk mendemonstrasikan
// protokol anomali (Latihan 3): pesan berukuran ekstrem (5 MB) sekali,
// TIDAK dihapus dari log meski hasilnya lambat / berbeda.
$executionQueue[] = [
    'scenario'  => '5MB_ANOMALY_TEST',
    'message'   => str_repeat('A', 5 * 1024 * 1024),
    'run_index' => 0,
];

$runCounter = 0;
$seedIndex  = 0;
$results    = []; // dikumpulkan untuk ringkasan statistik per skenario

foreach ($executionQueue as $job) {
    $runCounter++;
    $runId    = sprintf('run-%03d', $runCounter);
    $scenario = $job['scenario'];
    $message  = $job['message'];

    $seed = $seeds[$seedIndex % count($seeds)];
    $seedIndex++;

    // Seed dipakai untuk membangkitkan IV & AES key secara DETERMINISTIK
    // per run (bukan openssl_random_pseudo_bytes murni), supaya tiap run
    // punya seed yang tercatat & bisa direplikasi persis.
    mt_srand($seed);
    $iv     = '';
    $aesKey = '';
    for ($i = 0; $i < 16; $i++) $iv     .= chr(mt_rand(0, 255));
    for ($i = 0; $i < 32; $i++) $aesKey .= chr(mt_rand(0, 255));

    $timestamp = date('c');
    $status = 'OK';
    $anomalyNote = '';

    try {
        $t0 = microtime(true);
        $payload = EncryptionHelper::encryptMessage($message, $keys['public'], $iv, $aesKey);
        $encMs = round((microtime(true) - $t0) * 1000, 3);

        $t0 = microtime(true);
        $result = EncryptionHelper::decryptMessage($payload, $keys['private']);
        $decMs = round((microtime(true) - $t0) * 1000, 3);

        $integrityOk = $result['integrity_ok'] && ($result['plaintext'] === $message);

        if (!$integrityOk) {
            $status = 'ANOMALY';
            $anomalyNote = 'Integritas gagal - plaintext hasil dekripsi tidak cocok';
        }
        if ($scenario === '5MB_ANOMALY_TEST') {
            $status = 'ANOMALY_DOCUMENTED';
            $anomalyNote = 'Skenario ekstrem sengaja diuji (5MB) - waktu eksekusi jauh lebih tinggi, DIPERTAHANKAN di log, bukan dihapus';
        }

    } catch (\Throwable $e) {
        $status = 'CRASH';
        $encMs = null;
        $decMs = null;
        $integrityOk = false;
        $anomalyNote = 'Exception: ' . $e->getMessage();
    }

    fputcsv($csv, [
        $runId, $timestamp, $scenario, $seed, $codeVersion,
        strlen($message), $encMs, $decMs,
        $integrityOk ? 'true' : 'false', $status, $anomalyNote
    ]);

    $results[$scenario][] = ['enc' => $encMs, 'dec' => $decMs];

    echo "$runId | $scenario | seed=$seed | enc={$encMs}ms | dec={$decMs}ms | $status\n";
}

fclose($csv);

// ---------------------------------------------------------------
// 4. Ringkasan statistik per skenario (mean, std) — Latihan 1/refleksi
// ---------------------------------------------------------------
echo "\n=== RINGKASAN STATISTIK PER SKENARIO ===\n";
foreach ($results as $scenario => $runs) {
    $encTimes = array_column($runs, 'enc');
    $encTimes = array_filter($encTimes, fn($v) => $v !== null);
    if (empty($encTimes)) continue;

    $mean = array_sum($encTimes) / count($encTimes);
    $variance = array_sum(array_map(fn($x) => ($x - $mean) ** 2, $encTimes)) / count($encTimes);
    $std = sqrt($variance);

    printf("%-18s n=%-3d mean_enc=%.3fms std_enc=%.3fms\n", $scenario, count($encTimes), $mean, $std);
}

echo "\nLog lengkap tersimpan di: data_log.csv\n";
```

> Script ini bergantung pada `EncryptionHelper.php` dari WS-09 — pastikan kedua file berada di folder yang sama.

### data_log.csv (hasil eksekusi nyata, 21 run)

```csv
run_id,timestamp,scenario,seed,code_version,message_size_bytes,enc_time_ms,dec_time_ms,integrity_ok,status,anomaly_note
run-001,2026-07-12T11:24:16+00:00,1KB,42,e2ee-experiment-v1.0,1024,1.701,2.535,true,OK,
run-002,2026-07-12T11:24:16+00:00,10KB,123,e2ee-experiment-v1.0,10240,0.855,2.075,true,OK,
run-003,2026-07-12T11:24:16+00:00,100B,777,e2ee-experiment-v1.0,100,0.603,3.113,true,OK,
run-004,2026-07-12T11:24:16+00:00,100B,2025,e2ee-experiment-v1.0,100,0.593,2.004,true,OK,
run-005,2026-07-12T11:24:16+00:00,100KB,31337,e2ee-experiment-v1.0,102400,3.015,3.212,true,OK,
run-006,2026-07-12T11:24:16+00:00,1KB,8,e2ee-experiment-v1.0,1024,0.641,2.039,true,OK,
run-007,2026-07-12T11:24:16+00:00,10KB,99,e2ee-experiment-v1.0,10240,0.738,1.988,true,OK,
run-008,2026-07-12T11:24:16+00:00,1KB,555,e2ee-experiment-v1.0,1024,0.569,2.039,true,OK,
run-009,2026-07-12T11:24:16+00:00,10KB,71,e2ee-experiment-v1.0,10240,0.715,1.926,true,OK,
run-010,2026-07-12T11:24:16+00:00,100KB,2024,e2ee-experiment-v1.0,102400,1.448,2.648,true,OK,
run-011,2026-07-12T11:24:16+00:00,10KB,42,e2ee-experiment-v1.0,10240,0.647,2.054,true,OK,
run-012,2026-07-12T11:24:16+00:00,1KB,123,e2ee-experiment-v1.0,1024,0.591,2.109,true,OK,
run-013,2026-07-12T11:24:16+00:00,100KB,777,e2ee-experiment-v1.0,102400,1.298,2.46,true,OK,
run-014,2026-07-12T11:24:16+00:00,100KB,2025,e2ee-experiment-v1.0,102400,2.246,2.458,true,OK,
run-015,2026-07-12T11:24:16+00:00,100B,31337,e2ee-experiment-v1.0,100,0.568,1.982,true,OK,
run-016,2026-07-12T11:24:16+00:00,1KB,8,e2ee-experiment-v1.0,1024,0.589,1.901,true,OK,
run-017,2026-07-12T11:24:16+00:00,100KB,99,e2ee-experiment-v1.0,102400,1.314,2.559,true,OK,
run-018,2026-07-12T11:24:16+00:00,100B,555,e2ee-experiment-v1.0,100,0.604,1.987,true,OK,
run-019,2026-07-12T11:24:16+00:00,100B,71,e2ee-experiment-v1.0,100,0.574,2.043,true,OK,
run-020,2026-07-12T11:24:16+00:00,10KB,2024,e2ee-experiment-v1.0,10240,0.666,2.025,true,OK,
run-021,2026-07-12T11:24:16+00:00,5MB_ANOMALY_TEST,42,e2ee-experiment-v1.0,5242880,127.308,68.699,true,ANOMALY_DOCUMENTED,"Skenario ekstrem sengaja diuji (5MB) - waktu eksekusi jauh lebih tinggi, DIPERTAHANKAN di log, bukan dihapus"
```

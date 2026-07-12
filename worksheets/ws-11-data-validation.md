# WS-11: Data Validation & Integrity

> **Bab 11 — Validasi Data & Integritas**
> **Tema Eksperimen (lanjutan WS-09 & WS-10): Validasi Dataset E2EE Hybrid RSA + AES-256-CBC**

---

## Konteks Eksperimen

Worksheet ini memvalidasi `data_log.csv` (21 run) yang dihasilkan di WS-10 — sebelum data tersebut dipakai untuk klaim performa (mis. "enkripsi 1 KB butuh ~0.8 ms").

---

## Template A.11 — Data Validation Checklist (terisi)

```
DATA VALIDATION CHECKLIST

Completeness:
  [x] Semua skenario tercakup (100B, 1KB, 10KB, 100KB, 5MB-anomali)
  [x] Jumlah run sesuai rencana (20 run utama + 1 run anomali = 21)
  [x] Tidak ada file output hilang (semua tercatat di data_log.csv)
  Missing: 0 dari 21 data points

Format Consistency:
  [x] Semua file format sama (CSV, satu file data_log.csv)
  [x] Header konsisten (run_id, timestamp, scenario, seed, code_version,
      message_size_bytes, enc_time_ms, dec_time_ms, integrity_ok, status, anomaly_note)
  [x] Tipe data konsisten (waktu = float, seed = integer, integrity_ok = boolean string)

Range & Logic:
  [x] Nilai dalam range masuk akal (semua enc/dec_time_ms > 0)
  [x] Tidak ada waktu negatif
  [x] integrity_ok semua "true" (21/21) — tidak ada di luar {true, false}
  Anomali ditemukan: run-021 (5MB, waktu enc 127.308ms jauh di atas skenario lain)
                     — statistical outlier TIDAK terdeteksi pada skenario 1KB via IQR,
                       tapi run-001 (1KB, 1.701ms) adalah contextual anomaly (lihat Latihan 2)

Cross-Validation:
  [x] Run identik → hasil mendekati (4 dari 5 run 1KB berkisar 0.569–0.641 ms)
  [x] Trend konsisten dengan ekspektasi teori (waktu enkripsi naik seiring ukuran pesan:
      100B < 10KB < 1KB* < 100KB < 5MB, *1KB sedikit lebih tinggi karena outlier run-001)

Keputusan:
  [x] Data siap analisis (dengan catatan: run-001 & run-021 diberi anotasi khusus, bukan dihapus)
  [ ] Perlu cleaning
  [ ] Perlu re-run (skenario: ____)
```

---

## Latihan 1 — Completeness Check

| Skenario | Run Direncanakan | Run Tercatat | Missing | Alasan |
|----------|-----------------|-------------|---------|--------|
| 100 B | 5 | 5 | 0 | — |
| 1 KB | 5 | 5 | 0 | — |
| 10 KB | 5 | 5 | 0 | — |
| 100 KB | 5 | 5 | 0 | — |
| 5 MB (anomali/stress-test) | 1 | 1 | 0 | — |

**Total expected:** 21 | **Total actual:** 21 | **Missing:** 0

**Keputusan untuk data missing:**
> Tidak ada data yang hilang pada eksekusi ini (completeness 100%). Namun protokol tetap didokumentasikan untuk jaga-jaga: jika di run mendatang ada run berstatus `CRASH` (lihat kolom `status` di `data_log.csv`), run tersebut TETAP dihitung sebagai "tercatat" (bukan hilang) karena masih ada baris lognya dengan `anomaly_note` berisi pesan error — yang perlu di-re-run hanya skenario dengan run yang benar-benar tidak menghasilkan baris log sama sekali.

---

## Latihan 2 — Anomaly Investigation

Dataset diambil dari skenario **1 KB** (5 run nyata dari `data_log.csv`), karena ini skenario dengan variasi terbesar di antara skenario ukuran kecil-menengah.

| Run | Enc Time (ms) |
|-----|-------------|
| run-001 (seed 42) | 1.701 |
| run-006 (seed 8) | 0.641 |
| run-008 (seed 555) | 0.569 |
| run-012 (seed 123) | 0.591 |
| run-016 (seed 8) | 0.589 |

**Deteksi outlier (metode IQR, data diurutkan: 0.569, 0.589, 0.591, 0.641, 1.701):**
- Q1 = 0.579 | Q3 = 1.171 | IQR = 0.592
- Batas bawah (Q1 - 1.5×IQR) = -0.309
- Batas atas (Q3 + 1.5×IQR) = 2.059
- Outlier terdeteksi (murni IQR): **Tidak ada** — run-001 (1.701 ms) masih di bawah batas atas 2.059 ms

**Investigasi (untuk setiap nilai yang mencurigakan, meski lolos IQR):**

| Nilai Mencurigakan | Nilai | Kemungkinan Penyebab | Keputusan |
|---------|-------|---------------------|-----------|
| run-001 | 1.701 ms | Ini adalah run PERTAMA yang dieksekusi (setelah shuffle, run-001 tetap kebetulan berjalan lebih awal) — kemungkinan **cold-start**: OPcache PHP belum "panas", kelas `EncryptionHelper` baru pertama kali dimuat & dikompilasi | **Bukan dihapus.** Ditandai sebagai *contextual anomaly* (normal secara statistik/IQR, tapi mencolok dibanding 4 run 1KB lainnya yang konsisten di 0.57–0.64 ms). Direkomendasikan: tambahkan 1 "warm-up run" yang dibuang sebelum mulai pencatatan resmi di eksperimen berikutnya |
| run-021 (5MB) | 127.308 ms | Bukan outlier dalam skenario 1KB (beda skenario), tapi merupakan **statistical outlier ekstrem** dibanding seluruh dataset gabungan (semua skenario) — sesuai desain, karena ukuran pesan 5 MB memang jauh lebih besar dari skenario lain | **Dipertahankan** sebagai temuan valid tentang skalabilitas AES-CBC, bukan dihapus. Diberi status `ANOMALY_DOCUMENTED` di kolom `status`, bukan `OK` biasa, agar mudah difilter saat analisis |

**Catatan penting (sesuai Jebakan Kognitif #4 di materi):** Contoh ini menunjukkan bahwa **IQR murni tidak selalu menangkap semua anomali** — run-001 lolos uji IQR statistik pada skenario 1KB, tapi tetap merupakan *contextual anomaly* yang layak diinvestigasi karena menyimpang jauh dari 4 run sejenisnya.

---

## Latihan 3 — Validation Report

**1. Completeness:** 100% data terkumpul (21 dari 21 run direncanakan)

**2. Format:** [x] Konsisten / [ ] Ada inkonsistensi

**3. Range check (anomali):**
> - run-021 (5MB): waktu enc 127.308 ms — statistical outlier ekstrem terhadap seluruh dataset, DIDOKUMENTASIKAN sebagai stress-test, bukan dihapus
> - run-001 (1KB): waktu enc 1.701 ms — contextual anomaly (lolos IQR tapi mencolok dibanding 4 run sejenisnya), diduga cold-start OPcache, DIDOKUMENTASIKAN untuk investigasi lanjutan
> - Tidak ditemukan nilai negatif atau `integrity_ok=false` di seluruh 21 baris

**4. Logic check:** [x] Parameter sesuai plan / [ ] Ada ketidaksesuaian
> Semua run memakai `AES-256-CBC` + `RSA-2048` sesuai desain WS-09, seed sesuai daftar pre-determined di WS-10, dan trend waktu enkripsi naik seiring ukuran pesan (kecuali anomali run-001 yang sudah diinvestigasi terpisah).

**Kesimpulan:** [x] Data siap analisis / [ ] Perlu tindakan
> Dataset `data_log.csv` (21 run) dinyatakan layak untuk analisis statistik (mean, std per skenario di WS-10), dengan catatan run-001 dan run-021 diberi anotasi khusus saat interpretasi hasil, bukan dikeluarkan dari dataset.

---

## Refleksi

> Apa perbedaan antara "data yang benar" dan "data yang dipercaya"? Mengapa proses validasi formal diperlukan meskipun data dikumpulkan secara otomatis?

> Pada eksperimen ini, semua 21 baris `data_log.csv` "benar" dalam arti dihasilkan langsung oleh script tanpa dimanipulasi — tidak ada nilai yang diketik manual atau diedit. Tapi "benar" secara teknis tidak otomatis berarti "dipercaya": run-001 (1.701 ms) dan run-021 (127.308 ms) adalah data yang sah dan akurat, namun kalau langsung dirata-rata tanpa investigasi, keduanya bisa membuat kesimpulan performa E2EE menyesatkan (mis. skenario 1KB terlihat "lebih lambat dari 10KB" kalau run-001 tidak dianotasi terpisah).
>
> Proses validasi formal tetap diperlukan meski data dikumpulkan otomatis oleh `execution_runner.php`, karena logger otomatis hanya menjamin data **tercatat dengan konsisten** — bukan menjamin setiap nilai **bebas dari gangguan eksternal** (cold-start, thermal throttling, background process) yang tidak terlihat dari angka itu sendiri. Validasi (completeness, IQR, cross-check trend) adalah langkah yang mengubah "data mentah yang benar" menjadi "data yang layak dipercaya untuk membuat klaim ilmiah".

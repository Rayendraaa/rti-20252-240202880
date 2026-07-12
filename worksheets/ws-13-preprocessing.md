# WS-13: Data Preprocessing

> **Bab 13 — Preprocessing & Persiapan Data untuk Analisis**
> **Tema Eksperimen (lanjutan WS-09/10/11/12): Preprocessing Dataset E2EE untuk Pemodelan**

---

## Konteks Eksperimen

Dataset `data_log.csv` (21 run, sudah divalidasi di WS-11) akan dipakai untuk tugas baru: **memodelkan hubungan antara ukuran pesan (byte) dan waktu enkripsi (ms)** menggunakan regresi sederhana. Karena ini melibatkan train/test split, preprocessing harus dilakukan dengan hati-hati agar tidak terjadi *data leakage*.

---

## Template A.13 — Preprocessing Documentation Log (terisi)

```
PREPROCESSING LOG

Dataset           : data_log.csv (WS-10/11), 21 run E2EE hybrid RSA-2048 + AES-256-CBC
Jumlah data awal  : 21 records, 11 kolom

Cleaning:
| Masalah | Jumlah Kasus | Penanganan | Justifikasi |
|---------|-------------|------------|-------------|
| Missing | 0 dari 21   | Tidak ada tindakan | Semua kolom enc_time_ms/dec_time_ms/integrity_ok terisi (dicek WS-11) |
| Duplikat| 0 dari 21   | Tidak ada tindakan | run_id 100% unik, dicek dengan set() |
| Error   | 0 dari 21   | Tidak ada tindakan | Semua timestamp format ISO 8601 konsisten (2026-07-12), status hanya berisi 2 nilai valid: OK/ANOMALY_DOCUMENTED |
| Skenario ekstrem | 1 dari 21 (run-021, 5MB) | Flag & separate — dikeluarkan dari dataset pemodelan skala-kecil, TIDAK dihapus dari data_log.csv mentah | Ukurannya 51× lebih besar dari skenario terbesar berikutnya (100KB); jika diikutsertakan akan mendominasi regresi linear dan mendistorsi hubungan pada rentang 100B-100KB yang jadi fokus pertanyaan riset |

Transformation:
| Transformasi | Variabel | Detail | Alasan |
|-------------|----------|--------|--------|
| (tidak ada transformasi non-linear diterapkan pada dataset final) | — | Sempat dipertimbangkan log-transform pada message_size_bytes (seperti di WS-12 untuk visualisasi), tapi TIDAK diterapkan di sini karena rentang 100B-100KB pada 20 data utama masih cukup wajar untuk regresi linear langsung | Prinsip Minimal Distortion — tidak transformasi jika tidak benar-benar diperlukan |

Normalization:
  Metode    : Z-score ((x - mean) / std)
  Alasan    : message_size_bytes (100 - 102.400) dan enc_time_ms (0.568 - 3.015) berada
              di skala sangat berbeda; jika dipakai bersama dalam model berbasis jarak
              atau gradient descent, variabel skala besar akan mendominasi
  Parameter : dihitung dari TRAINING SET SAJA (16 dari 20 data utama)
              mean_size=28441.00, std_size=42883.90
              mean_enc=0.9778,   std_enc=0.6881

Leakage Check:
  [x] Parameter normalisasi dari training set saja (16 run, seed split=2026)
  [x] Tidak ada informasi test set dalam preprocessing (mean/std dihitung SEBELUM melihat test set)
  [x] Cross-validation/split dilakukan sebelum normalisasi dihitung

Jumlah data akhir : 20 records dipakai untuk pemodelan (16 train + 4 test),
                     1 record (5MB) dikeluarkan dari pemodelan tapi tetap ada di data_log.csv mentah
Script tersedia   : [x] Ya → path: preprocessing.py (lampiran) | [ ] Belum
```

---

## Latihan 1 — Cleaning Plan

| Masalah | Jumlah Kasus | Penanganan | Justifikasi |
|---------|-------------|------------|-------------|
| Missing values (enc/dec_time_ms, integrity_ok) | 0 dari 21 (0%) | Tidak ada tindakan diperlukan | Dicek langsung di WS-11: completeness 100%, tidak ada sel kosong |
| Duplikat run_id | 0 dari 21 (0%) | Tidak ada tindakan | `len(run_ids) == len(set(run_ids))` → True, semua unik |
| Inkonsistensi format timestamp/status | 0 dari 21 (0%) | Tidak ada tindakan | Semua timestamp ISO 8601 tanggal sama, status hanya 2 kategori valid |
| Skenario ekstrem di luar rentang analisis (5MB) | 1 dari 21 (4.8%) | Flag & separate — dipisah ke subset "skenario anomali", tidak dipakai di model regresi utama | Sesuai materi: bukan missing/duplikat/error, tapi outlier substantif by design yang perlu ditangani terpisah, bukan dihapus dari dataset mentah |

**Jumlah data sebelum cleaning:** 21
**Jumlah data setelah cleaning (untuk pemodelan):** 20
**Persentase data yang dikeluarkan dari pemodelan:** 4.8% (1 dari 21, tetap didokumentasikan di `data_log.csv`)

---

## Latihan 2 — Normalisasi Decision

| Variabel | Range Asli | Distribusi | Outlier? | Metode Normalisasi | Alasan |
|----------|-----------|-----------|----------|-------------------|--------|
| message_size_bytes | 100 – 102.400 byte | Right-skewed (4 titik cluster: 100, 1.024, 10.240, 102.400) | Tidak (dalam 20 data utama; run-021 5MB sudah dipisah di Latihan 1) | Z-score | Skala jauh lebih besar dari enc_time_ms; diperlukan agar kedua variabel sebanding untuk model regresi/jarak |
| enc_time_ms | 0.568 – 3.015 ms | Mendekati normal per-skenario, sedikit right-skewed karena run-001 (1.701 ms pada 1KB) | Ya (run-001, terdokumentasi sebagai contextual anomaly di WS-11) | Z-score | Robust scaling dipertimbangkan karena ada 1 outlier ringan, tapi Z-score tetap dipilih karena outlier tsb sudah didokumentasikan & bukan error, serta n masih kecil (20) sehingga robust scaling tidak memberi keuntungan signifikan |
| dec_time_ms | 1.901 – 3.212 ms | Relatif stabil, variasi kecil antar-skenario | Tidak signifikan | Tidak perlu (tidak dipakai sebagai fitur di model ini) | Model hanya fokus pada enc_time_ms sebagai target; dec_time_ms disimpan tapi tidak dinormalisasi karena di luar scope pertanyaan riset saat ini |

**Apakah normalisasi diperlukan?** [x] Ya / [ ] Tidak

**Justifikasi:**
> Diperlukan karena `message_size_bytes` (skala ribuan-ratusan ribu) dan `enc_time_ms` (skala satuan) memiliki magnitudo yang sangat berbeda. Jika kelak model dikembangkan lebih jauh (mis. regresi multivariat dengan fitur tambahan, atau clustering skenario), perbedaan skala ini bisa membuat satu variabel secara tidak adil mendominasi perhitungan jarak/gradient — z-score menyamakan kontribusi kedua variabel.

**Leakage check:**
- [x] Parameter dihitung dari training set saja (16 run: 4 per skenario, split seed=2026)
- [x] Normalisasi diterapkan setelah train-test split (bukan sebelum split lalu dipecah)

**Contoh hasil normalisasi (train vs test, parameter dari train saja):**

| Set | Run | message_size (z) | enc_time (z) |
|---|---|---|---|
| Train | run-019 (100B) | -0.661 | -0.587 |
| Train | run-005 (100KB) | 1.725 | 2.961 |
| Test  | run-004 (100B) | -0.661 | -0.559 |
| Test  | run-001 (1KB, anomali) | -0.639 | **1.051** |
| Test  | run-013 (100KB) | 1.725 | 0.465 |

*Catatan: run-001 (kandidat contextual anomaly dari WS-11) kebetulan masuk ke test set pada split ini — nilai z-score-nya (1.051) mengonfirmasi ulang bahwa titik ini menyimpang cukup jauh (>1 std) dari pola normal skenario 1KB, konsisten dengan temuan WS-11.*

---

## Latihan 3 — Preprocessing Report

```
PREPROCESSING SUMMARY

1. Dataset: data_log.csv (eksperimen E2EE RSA-2048 + AES-256-CBC, WS-09/10)
2. Data awal: 21 records, 11 kolom (fitur utama dipakai: message_size_bytes, enc_time_ms)
3. Cleaning:
   - Missing values: 0 kasus, metode: tidak ada tindakan (data lengkap)
   - Duplikat: 0 kasus, tindakan: tidak ada tindakan (run_id unik)
   - Error format: 0 kasus, tindakan: tidak ada tindakan
   - Tambahan: 1 kasus skenario ekstrem (5MB), tindakan: flag & separate (dikeluarkan
     dari dataset pemodelan, tetap ada di data_log.csv mentah)
4. Transformation: tidak ada transformasi non-linear diterapkan (prinsip minimal distortion)
5. Normalisasi: Z-score, parameter dari TRAINING SET (16 dari 20 data utama, split seed=2026)
   mean_size=28441.00 std_size=42883.90 | mean_enc=0.9778 std_enc=0.6881
6. Data akhir: 20 records dipakai untuk pemodelan (16 train / 4 test), 2 kolom fitur
7. Leakage check: [x] Lulus / [ ] Ada masalah
```

---

## Refleksi

> Apakah Anda pernah melakukan normalisasi "karena biasa dilakukan" tanpa mempertimbangkan apakah benar-benar diperlukan? Apa risiko over-preprocessing?

> Pada tahap awal menyiapkan dataset ini, ada dorongan untuk langsung menerapkan log-transform pada `message_size_bytes` (meniru skala log yang dipakai untuk visualisasi di WS-12) sebelum benar-benar mengecek apakah itu perlu untuk regresi linear sederhana pada 20 data utama. Setelah dicek ulang, rentang 100B-100KB pada 20 data utama (tanpa skenario 5MB) sebenarnya masih cukup wajar untuk dipakai langsung tanpa transformasi non-linear — log-transform baru relevan kalau skenario 5MB ikut disertakan.
>
> Risiko over-preprocessing yang paling nyata di sini adalah: kalau normalisasi/transformasi diterapkan ke SELURUH dataset (termasuk test set) sebelum split, atau kalau skenario 5MB tetap diikutsertakan tanpa dipisah, hasil z-score dan model regresi bisa menyembunyikan pola sebenarnya pada rentang kecil-menengah (100B-100KB) — persis jenis distorsi yang diperingatkan di prinsip "Minimal Distortion": ubah sesedikit mungkin, jangan preprocessing "karena biasa dilakukan".

---

## Lampiran — Kode Program

### preprocessing.py

```python
"""
preprocessing.py
---------------------------------------------------------
WS-13 - Preprocessing dataset E2EE (data_log.csv)
Tujuan: menyiapkan message_size_bytes & enc_time_ms untuk
regresi sederhana, dengan train/test split & normalisasi
yang TIDAK bocor (leakage-free).
---------------------------------------------------------
"""
import csv
import statistics
import random

rows = list(csv.DictReader(open('data_log.csv')))

# 1. CLEANING ------------------------------------------------
run_ids = [r['run_id'] for r in rows]
assert len(run_ids) == len(set(run_ids)), "Ditemukan duplikat run_id!"
assert all(r['enc_time_ms'] != '' for r in rows), "Ditemukan missing value!"

# Flag & separate skenario ekstrem (5MB) - bukan dihapus dari file mentah,
# hanya dikeluarkan dari dataset pemodelan skala kecil-menengah ini.
main = [r for r in rows if r['scenario'] != '5MB_ANOMALY_TEST']
anomaly_flagged = [r for r in rows if r['scenario'] == '5MB_ANOMALY_TEST']
print(f"Main dataset: {len(main)} records | Flagged & separated: {len(anomaly_flagged)} record")

# 2. TRAIN/TEST SPLIT (SEBELUM normalisasi -> mencegah leakage) ----
scenarios = ['100B', '1KB', '10KB', '100KB']
by_scenario = {s: [r for r in main if r['scenario'] == s] for s in scenarios}

random.seed(2026)  # seed split PRE-DETERMINED & didokumentasikan
train, test = [], []
for s in scenarios:
    runs = by_scenario[s][:]
    random.shuffle(runs)
    test.append(runs[0])       # 1 run/skenario -> test set (4 total)
    train.extend(runs[1:])     # 4 run/skenario -> train set (16 total)

# 3. NORMALIZATION -- parameter dari TRAIN SET SAJA -----------------
train_sizes = [float(r['message_size_bytes']) for r in train]
train_enc   = [float(r['enc_time_ms']) for r in train]

mean_size, std_size = statistics.mean(train_sizes), statistics.pstdev(train_sizes)
mean_enc, std_enc   = statistics.mean(train_enc), statistics.pstdev(train_enc)


def zscore(x, mean, std):
    return (x - mean) / std


# Terapkan ke train & test memakai parameter YANG SAMA (dari train)
for label, dataset in [('TRAIN', train), ('TEST', test)]:
    print(f"\n{label} SET (n={len(dataset)}):")
    for r in dataset:
        size = float(r['message_size_bytes'])
        enc = float(r['enc_time_ms'])
        z_size = zscore(size, mean_size, std_size)
        z_enc = zscore(enc, mean_enc, std_enc)
        print(f"  {r['run_id']:8s} scenario={r['scenario']:6s} "
              f"size_z={z_size:+.3f} enc_z={z_enc:+.3f}")

# 4. LEAKAGE CHECK (self-test) ---------------------------------------
test_run_ids = {r['run_id'] for r in test}
train_run_ids = {r['run_id'] for r in train}
assert test_run_ids.isdisjoint(train_run_ids), "Leakage: ada run yang duplikat train/test!"
print("\nLeakage check: LULUS - parameter normalisasi hanya dari train, "
      "train & test tidak beririsan.")

hasilnya:
<img width="1918" height="1198" alt="gambar" src="https://github.com/user-attachments/assets/62035507-0761-49f3-9298-99da1c36ebbf" />

```

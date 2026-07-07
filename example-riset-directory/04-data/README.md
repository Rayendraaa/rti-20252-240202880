# 04-data

Data mentah hasil pengujian — output dari **Tahap 3**, input untuk **Tahap 4**.

## Isi yang diharapkan

- Hasil pengujian k6 dalam format CSV/JSON, per kombinasi mode (`CACHE_MODE=none|hybrid`) × jenis traffic (legitimate/attack/mixed)
- Metrik resource container (CPU, memori) PostgreSQL & Redis selama pengujian
- Metadata eksekusi tiap run (timestamp, konfigurasi, durasi)

## Catatan

Data di folder ini bersifat mentah (raw) dan belum diolah. Hasil olahan (statistik, grafik) disimpan di [../06-output/](../06-output/).

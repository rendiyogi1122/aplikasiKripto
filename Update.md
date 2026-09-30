Act as an Expert Full-Stack Developer (Next.js, TypeScript, FastAPI, Python, Supabase).
Tugas Anda adalah membuat fitur "Download Excel" yang meng-generate file .xlsx secara on-the-fly di backend (FastAPI) menggunakan library `xlsxwriter` dan mengembalikannya ke frontend (Next.js) via `StreamingResponse`.

Terdapat 3 jenis pengunduhan Excel dengan alur dan format yang berbeda. Perhatikan aturan dan skema datanya dengan sangat teliti:

---
### 1. ALUR DAN SKEMA DATABASE
Untuk Teks dan File, Frontend akan mengambil data dari Supabase lalu mengirimkan array JSON data tersebut ke endpoint FastAPI. Backend tidak perlu melakukan query ke database, cukup menerima JSON array, melooping datanya, dan merender Excel.
Gambar histogram akan dikirim dalam format Base64 (tanpa prefix 'data:image/png;base64,'). Backend harus men-decode Base64 menggunakan `io.BytesIO()` sebelum memasukkannya ke sel Excel dengan `worksheet.insert_image()`.

---
### 2. FORMAT EXCEL 1: ENKRIPSI TEKS (Multi-Row)
- Endpoint: `POST /api/download-excel/teks`
- Frontend mengirim array of object berisi riwayat teks dari tabel `text_encryption_history`.
- Kolom Excel yang harus dibuat secara berurutan:
  1. ID Uji
  2. Ciphertext
  3. Password (Ciphertext)
  4. Tipe Algoritma (method)
  5. Waktu Komputasi Enkripsi (waktu_komputasi_ms)
  6. Entropi Shannon (entropi_shannon)
  7. Avalanche Effect (avalanche_effect)
  8. Histogram Data (Diisi dengan gambar dari `histogram_base64`)
- Aturan Baris: Looping data dari request. Set default tinggi baris (row height) menjadi 100 agar gambar histogram tidak gepeng. Set lebar kolom Histogram menjadi 35.

---
### 3. FORMAT EXCEL 2: ENKRIPSI FILE (Multi-Row)
- Endpoint: `POST /api/download-excel/file`
- Frontend mengirim array of object berisi riwayat file dari tabel `file_encryption_history`.
- Kolom Excel yang harus dibuat secara berurutan:
  1. ID Uji
  2. Nama File (original_filename)
  3. Password (Ciphertext)
  4. Tipe File (Ekstensi)
  5. Ukuran File (file_size)
  6. Tipe Algoritma (method)
  7. Waktu Komputasi Enkripsi (waktu_komputasi_ms)
  8. Entropi Shannon (entropi_shannon)
  9. Avalanche Effect (avalanche_effect)
  10. Histogram Data (Diisi dengan gambar dari `histogram_base64`)
- Aturan Baris: Sama seperti pengujian teks, lakukan looping, atur tinggi baris 100, dan atur scale gambar menggunakan opsi `{'x_scale': 0.5, 'y_scale': 0.5}` (atau sesuaikan agar muat di dalam sel).

---
### 4. FORMAT EXCEL 3: VISUALISASI ECB vs AES-GCM (Single-Row / Tanpa Database)
- Endpoint: `POST /api/download-excel/visualisasi`
- Fitur ini tidak mengambil data dari database. Frontend akan mengirimkan 1 object payload yang berisi State saat ini di layar, yaitu: Nama File, Ukuran File, Waktu Komputasi, gambar_original (Base64), gambar_ecb (Base64), dan gambar_gcm (Base64).
- Kolom Excel yang harus dibuat secara berurutan:
  1. ID Uji (Generate random/timestamp saja)
  2. Nama File
  3. Ukuran File
  4. Waktu Komputasi
  5. Gambar Awal (Original) -> Insert image dari gambar_original
  6. Gambar Hasil Mode ECB -> Insert image dari gambar_ecb
  7. Gambar Hasil Mode AES-GCM -> Insert image dari gambar_gcm
- Aturan Baris: Hanya buat 1 baris data di bawah header. Atur tinggi baris menjadi 150 dan lebar ke-3 kolom gambar tersebut menjadi minimal 40 agar gambar perbandingan terlihat sangat jelas berdampingan.

---
### INSTRUKSI KODE:
1. Buatkan kode `main.py` (FastAPI) lengkap dengan Pydantic Models untuk menerima request tersebut, dan fungsi `xlsxwriter` yang membungkus gambar ke `io.BytesIO()`. Return menggunakan `StreamingResponse` dengan media type `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`.
2. Buatkan contoh fungsi handler di `frontend` (Next.js TypeScript) untuk men-trigger fetch ke endpoint backend tersebut dan memicu proses download file `.xlsx` di browser user.
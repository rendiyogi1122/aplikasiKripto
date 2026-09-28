Nama Fitur: Panel Analisis Metrik Kriptografi Komprehensif (Waktu, Entropi, Histogram, dan Avalanche Effect)

Konteks & Tujuan:
Ekstensikan fungsionalitas enkripsi dan dekripsi yang sudah ada dengan memunculkan dashboard analitik secara otomatis tepat setelah proses kriptografi selesai. Karena sistem ini akan digunakan untuk memenuhi evaluasi teknis tugas keamanan informasi yang ketat, tampilan harus dibuat sekompak mungkin (compact), profesional, dan hasil perhitungan statisik (seperti akurasi Hamming distance) harus dieksekusi dengan presisi di sisi server.

1. Pemrosesan Analitik di Backend (Python FastAPI)
Modifikasi endpoint enkripsi dan dekripsi saat ini untuk tidak hanya mengembalikan teks/file hasil, tetapi juga objek JSON berisi metrik berikut:

    Waktu Eksekusi (Performance): Gunakan time.perf_counter() untuk mengukur selisih waktu sebelum dan sesudah algoritma enkripsi/dekripsi berjalan. Kembalikan nilainya dalam format milidetik (ms).

    Entropi Shannon: Buat fungsi matematis untuk menghitung tingkat keacakan (randomness) data dari ciphertext. Nilai harus dikembalikan dalam bentuk float (ideal mendekati 8.0).

    Avalanche Effect (Khusus Enkripsi): Buat alur simulasi terisolasi di dalam endpoint.

        Ambil plaintext asli dan ubah (flip) tepat 1 bit secara acak.

        Enkripsi plaintext yang dimodifikasi tersebut menggunakan kunci (key) dan Initialization Vector (IV) yang persis sama.

        Bandingkan ciphertext asli dengan ciphertext modifikasi menggunakan Hamming Distance.

        Hitung dan kembalikan persentase perubahannya (ideal berada di kisaran 50%).

    Data Array Histogram: Hitung frekuensi kemunculan setiap nilai byte (0 hingga 255) dari plaintext dan ciphertext. Kembalikan dua array integer panjang 256 agar frontend tidak perlu membebani browser untuk melakukan ekstraksi frekuensi byte dari file besar.

2. Desain Antarmuka di Frontend (Next.js & Tailwind CSS)
Buat komponen baru bernama <CryptoAnalyticsPanel/> yang langsung dirender (muncul dengan animasi fade-in) di bawah kotak hasil enkripsi/dekripsi.

    Grid Layout Kompak: Gunakan grid-cols-3 dari Tailwind untuk menyusun metrik angka agar berdampingan dan tidak menyita ruang vertikal layar.

    Kartu Indikator Metrik (Metric Cards):

        Card 1 (Waktu): Tampilkan label "Waktu Komputasi" dengan angka tebal (misal: 42.5 ms).

        Card 2 (Entropi): Tampilkan nilai entropi (misal: 7.98). Tambahkan indikator warna (teks hijau jika > 7.9, merah jika di bawahnya).

        Card 3 (Avalanche Effect): Tampilkan persentase (misal: 50.2%). Beri warna hijau jika angkanya berkisar antara 45% - 55%.

    Visualisasi Histogram Ringkas:

        Gunakan library grafik modern seperti recharts atau chart.js.

        Gunakan tipe Bar Chart yang menampilkan distribusi plaintext (warna biru/transparan) di- overlay dengan distribusi ciphertext (warna oranye/merah solid).

        Tempatkan grafik ini di bawah ketiga kartu metrik. Untuk menjaga UI tetap rapi, masukkan grafik ini ke dalam Accordion (tombol dropdown) berjudul "Lihat Perbandingan Histogram Data" sehingga grafik hanya merender memakan ruang layar jika pengguna secara eksplisit membukanya.

3. Penanganan State dan Loading
Pastikan komponen analitik memiliki skeleton loader yang berjalan selagi menunggu respons kalkulasi dari FastAPI, untuk memberikan umpan balik visual bahwa perhitungan metrik sedang berlangsung di background.
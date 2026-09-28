Konteks Masalah:
Simulasi Avalanche Effect saat ini menghasilkan nilai 0.00% karena algoritma AES-GCM dan ChaCha20-Poly1305 beroperasi dengan prinsip stream cipher (menggunakan keystream). Membalikkan (flip) 1 bit pada plaintext menggunakan kunci dan IV yang sama hanya akan mengubah 1 bit pada ciphertext. Untuk mendapatkan evaluasi yang valid pada mode ini, modifikasi (bit-flip) harus dilakukan pada Kunci (Key) atau IV, bukan pada plaintext.

Kebutuhan Pembaruan Kode:
Tulis ulang fungsi penghitung avalanche_effect di backend dengan alur logika berikut:

    Pertahankan Plaintext: Jangan lakukan modifikasi apa pun pada data plaintext asli.

    Enkripsi Baseline: Lakukan enkripsi menggunakan Plaintext, Key asli, dan IV asli. Ekstrak data ciphertext (tidak termasuk tag autentikasi jika memungkinkan, atau gunakan keseluruhan payload). Simpan sebagai ciphertext_A.

    Injeksi Perubahan (Bit-Flipping):

        Buat salinan dari Key asli menjadi modified_key (berupa bytearray).

        Ubah (XOR) tepat 1 bit pada byte pertama dari modified_key.

        Contoh implementasi Python: modified_key[0] ^= 0x01

    Enkripsi Sekunder: Lakukan enkripsi kedua pada Plaintext yang sama menggunakan modified_key dan IV asli. Simpan hasilnya sebagai ciphertext_B.

    Kalkulasi Hamming Distance:

        Lakukan iterasi per byte (dan per bit) antara ciphertext_A dan ciphertext_B.

        Hitung jumlah total bit yang berbeda di posisi yang sama.

    Kalkulasi Persentase: Hitung rasio perubahan menggunakan rumus: (Hamming Distance / Total Bit Panjang Ciphertext) * 100.

    Pengembalian Data: Pastikan API sekarang mengembalikan persentase yang berada di kisaran ~50% (jika algoritmanya ideal) ke antarmuka Next.js.

Catatan Kriptografi: Jangan gunakan modified_key ini untuk operasi penyimpanan atau dekripsi yang sesungguhnya. Operasi ini harus diisolasi murni di dalam service analitik atau blok kode metrik evaluasi sesaat setelah enkripsi data asli selesai dilakukan.
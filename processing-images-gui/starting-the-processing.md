# Memulai Pemrosesan

Setelah Anda mengimpor gambar, menandai target kalibrasi, dan mengonfigurasi pengaturan proyek, Anda siap untuk memulai pemrosesan. Halaman ini akan memandu Anda melalui proses memulai pipeline pemrosesan Chloros.

## Daftar Periksa Pra-Pemrosesan

Sebelum mengklik tombol Mulai, pastikan semuanya siap:

* [ ] **Berkas diimpor** - Semua gambar muncul di File Browser
* [ ] **Gambar target ditandai** - Kolom Target dicentang untuk gambar kalibrasi
* [ ] **Model kamera terdeteksi** - Kolom Model Kamera menampilkan kamera yang benar
* [ ] **Pengaturan dikonfigurasi** - Pengaturan Proyek telah ditinjau dan disesuaikan
* [ ] **Indeks dipilih** - Indeks multispektral yang diinginkan ditambahkan (jika diperlukan)
* [ ] **Format ekspor dipilih** - Format output sesuai dengan alur kerja Anda

{% hint style=&quot;info&quot; %}
**Tips**: Klik beberapa gambar di File Browser untuk memverifikasi bahwa gambar-gambar tersebut dimuat dengan benar sebelum pemrosesan.
{% endhint %}

***

## Memulai Pemrosesan

### Temukan Tombol Mulai

Tombol Mulai/Putar terletak di bilah header atas Chloros:

* Posisi: Tengah atas jendela
* Ikon: **Tombol Putar/Mulai** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Status: Tombol aktif (terang) saat siap diproses

### Klik untuk Memulai

1. Klik tombol **Play/Start** di bilah header atas
2. Pemrosesan dimulai segera
3. Tombol menjadi tidak aktif (abu-abu) selama pemrosesan
4. Bilah kemajuan diperbarui, menampilkan status pemrosesan

{% hint style=&quot;success&quot; %}
**Pemrosesan Dimulai**: Setelah diklik, Chloros secara otomatis menangani semua langkah pemrosesan - deteksi target, debayering, kalibrasi, perhitungan indeks, dan ekspor.
{% endhint %}

***

## Memahami Mode Pemrosesan

Chloros beroperasi dalam dua mode pemrosesan berbeda tergantung pada lisensi Anda:

### Mode Gratis (Pemrosesan Berurutan)

**Tersedia untuk semua pengguna**

**Cara kerjanya:**

* Memproses gambar satu per satu secara berurutan
* Operasi single-threaded
* Penggunaan memori lebih rendah

**Bar kemajuan menampilkan 2 tahap:**

1. **Deteksi Target** - Pemindaian target kalibrasi
2. **Pemrosesan** - Penerapan kalibrasi dan ekspor gambar

**Waktu pemrosesan:**

* Jauh lebih lambat daripada mode paralel Chloros+
* Cocok untuk dataset kecil hingga menengah (&lt; 200 gambar)

### Mode Chloros+ (Pemrosesan Paralel)

**Membutuhkan lisensi Chloros+**

**Cara kerjanya:**

* Memproses beberapa gambar secara bersamaan
* Operasi multi-threaded (hingga 16 pekerja paralel)
* Memanfaatkan beberapa inti CPU
* Akselerasi GPU (CUDA) opsional dengan kartu grafis NVIDIA

**Bar kemajuan menampilkan 4 tahap:**

1. **Deteksi** - Mencari target kalibrasi
2. **Analisis** - Memeriksa metadata gambar dan mempersiapkan pipeline
3. **Kalibrasi** - Menerapkan koreksi dan kalibrasi
4. **Ekspor** - Menyimpan gambar yang diproses dan indeks

**Interaksi bilah kemajuan:**

* **Arahkan kursor mouse** ke bilah untuk melihat panel dropdown 4 tahap yang detail
* **Klik** bilah kemajuan untuk membekukan panel dropdown di tempat
* **Klik lagi** untuk membatalkan pembekuan dan menyembunyikan panel

**Waktu pemrosesan:**

* Jauh lebih cepat daripada mode gratis
* Berbanding lurus dengan jumlah inti CPU
* Akselerasi GPU lebih meningkatkan kecepatan

{% hint style=&quot;info&quot; %}
**Chloros+ Kecepatan**: Pemrosesan paralel dapat 5-10 kali lebih cepat daripada mode berurutan untuk dataset besar. Proyek 500 gambar yang memakan waktu 2 jam dalam mode gratis dapat selesai dalam 15-20 menit dengan Chloros+.
{% endhint %}

***

## Apa yang Terjadi Selama Pemrosesan

### Tahap 1: Deteksi Target

**Apa yang dilakukan Chloros:**

* Memindai gambar target yang ditandai (atau semua gambar jika tidak ada yang ditandai)
* Mengidentifikasi 4 panel kalibrasi di setiap target
* Mengekstrak nilai reflektansi dari panel target
* Merekam cap waktu target untuk penjadwalan kalibrasi

**Durasi:** 1-30 detik (dengan target yang ditandai), 5-30+ menit (tanpa tanda)

### Tahap 2: Debayering (Konversi RAW)

**Apa yang dilakukan Chloros:**

* Mengonversi data pola Bayer RAW menjadi gambar RGB penuh
* Menerapkan algoritma demosaicing berkualitas tinggi
* Mempertahankan kualitas dan detail gambar maksimal

**Durasi:** Bervariasi tergantung jumlah gambar dan kecepatan CPU

### Tahap 3: Kalibrasi

**Apa yang dilakukan Chloros:**

* **Koreksi vignette**: Menghapus gelapnya lensa di tepi
* **Kalibrasi reflektansi**: Menormalkan menggunakan nilai reflektansi target
* Menerapkan koreksi di semua band/saluran
* Menggunakan target kalibrasi yang sesuai untuk setiap gambar berdasarkan cap waktu

**Durasi:** Sebagian besar waktu pemrosesan

### Tahap 4: Perhitungan Indeks

**Apa yang dilakukan Chloros:**

* Menghitung indeks multispektral yang dikonfigurasi (NDVI, NDRE, dll.)
* Menerapkan perhitungan band pada gambar yang dikalibrasi
* Menghasilkan gambar indeks untuk setiap indeks yang dipilih

**Durasi:** Beberapa detik per gambar

### Tahap 5: Ekspor

**Apa yang dilakukan Chloros:**

* Menyimpan gambar yang dikalibrasi dalam format yang dipilih
* Mengekspor gambar indeks dengan warna LUT yang dikonfigurasi
* Menulis file ke subfolder model kamera
* Mempertahankan nama file asli dengan sufiks

**Durasi:** Bervariasi tergantung format ekspor dan ukuran file

***

## Perilaku Pemrosesan

### Alur Pemrosesan Otomatis

Setelah dimulai, seluruh alur pemrosesan berjalan secara otomatis:

* Tidak memerlukan interaksi pengguna
* Semua langkah yang dikonfigurasi dieksekusi secara berurutan
* Pembaruan progres ditampilkan secara real-time

### Penggunaan Komputer Selama Pemrosesan

**Mode Bebas:**

* Penggunaan CPU relatif rendah (single-threaded)
* Komputer tetap responsif untuk tugas lain
* Aman untuk meminimalkan Chloros dan bekerja di aplikasi lain

**Chloros+ Mode Paralel:**

* Penggunaan CPU tinggi (multi-threaded, hingga 16 inti)
* Dengan akselerasi GPU: Penggunaan GPU tinggi
* Komputer mungkin kurang responsif selama pemrosesan
* Hindari memulai tugas CPU-intensif lainnya

{% hint style=&quot;warning&quot; %}
**Tips Kinerja**: Untuk kinerja optimal Chloros+, tutup aplikasi lain dan biarkan Chloros menggunakan sumber daya sistem penuh.
{% endhint %}

### Pemrosesan Tidak Dapat Dihentikan

**Batasan penting:**

* Setelah dimulai, pemrosesan tidak dapat dihentikan
* Anda dapat membatalkan pemrosesan, tetapi kemajuan akan hilang
* Hasil parsial tidak disimpan
* Harus memulai dari awal jika dibatalkan

**Tips perencanaan:** Untuk proyek yang sangat besar, pertimbangkan untuk memproses dalam batch atau menggunakan CLI untuk kontrol yang lebih baik.

***

## Memantau Pemrosesan Anda

Selama pemrosesan berjalan, Anda dapat:

* **Melihat bilah kemajuan** - Lihat persentase penyelesaian keseluruhan
* **Melihat tahap saat ini** - Deteksi, Analisis, Kalibrasi, atau Ekspor
* **Periksa tab log** - Lihat pesan dan peringatan pemrosesan secara detail
* **Pratinjau gambar yang telah selesai** - Beberapa file ekspor mungkin muncul selama pemrosesan

Untuk informasi detail tentang pemantauan, lihat [Monitoring the Processing](monitoring-the-processing.md).

***

## Membatalkan Pemrosesan

Jika Anda perlu menghentikan pemrosesan:

### Cara Membatalkan

1. Temukan tombol **Stop/Cancel** (menggantikan tombol Start selama pemrosesan)
2. Klik tombol Stop
3. Pemrosesan berhenti segera
4. Hasil sebagian akan dihapus

### Kapan Membatalkan

**Alasan yang sah untuk membatalkan:**

* Menyadari pengaturan yang salah digunakan
* Lupa menandai gambar target
* Gambar yang salah diimpor
* Sistem berjalan terlalu lambat atau tidak responsif

**Setelah membatalkan:**

* Periksa dan perbaiki masalah apa pun
* Sesuaikan pengaturan sesuai kebutuhan
* Mulai ulang pemrosesan dari awal
* Untuk pengalaman terbaik, tutup sepenuhnya Chloros dan mulai ulang

{% hint style=&quot;warning&quot; %}
**Tidak Ada Hasil Parsial**: Membatalkan akan membuang semua kemajuan. Chloros tidak menyimpan gambar yang diproses sebagian.
{% endhint %}

***

## Perkiraan Waktu Pemrosesan

Waktu pemrosesan aktual bervariasi secara signifikan berdasarkan:

* Jumlah gambar
* Resolusi gambar
* Format input RAW vs JPG
* Mode pemrosesan (Free vs Chloros+)
* Kecepatan CPU dan jumlah inti
* Ketersediaan GPU (hanya Chloros+)
* Jumlah indeks yang dihitung
* Kompleksitas format ekspor

### Perkiraan Kasar (Chloros+, gambar 12MP, CPU modern)

| Jumlah Gambar | Mode Gratis | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 gambar   | 15-20 menit | 5-8 menit        | 3-5 menit        |
| 100 gambar  | 30-40 menit | 10-15 menit      | 5-8 menit        |
| 200 gambar  | 1-1,5 jam | 20-30 menit      | 10-15 menit      |
| 500 gambar  | 2-3 jam   | 45-60 menit      | 20-30 menit      |
| 1000 gambar | 4-6 jam   | 1,5-2 jam      | 40-60 menit      |

{% hint style=&quot;info&quot; %}
**Pertama Kali Dijalankan**: Pemrosesan awal mungkin memakan waktu lebih lama karena Chloros membangun cache dan profil. Pemrosesan dataset serupa selanjutnya akan lebih cepat.
{% endhint %}

***

## Masalah Umum Saat Memulai

### Tombol Mulai Dinonaktifkan (Berwarna Abu-abu)

**Penyebab Mungkin:**

* Tidak ada gambar yang diimpor
* Backend belum sepenuhnya dimulai
* Pemrosesan sebelumnya masih berjalan
* Proyek belum sepenuhnya dimuat

**Solusi:**

1. Tunggu hingga backend sepenuhnya diinisialisasi (periksa ikon menu utama)
2. Verifikasi gambar telah diimpor di File Browser
3. Restart Chloros jika tombol tetap dinonaktifkan
4. Periksa Debug Log untuk pesan kesalahan

### Pemrosesan Dimulai Lalu Segera Gagal

**Penyebab kemungkinan:**

* Tidak ada gambar yang valid dalam proyek
* File gambar rusak
* Ruang disk tidak mencukupi
* Memori (RAM) tidak mencukupi

**Solusi:**

1. Periksa Debug Log <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> untuk pesan kesalahan
2. Verifikasi ruang disk yang tersedia
3. Coba proses subset gambar yang lebih kecil
4. Verifikasi gambar tidak rusak

### Peringatan &quot;Tidak Ada Target yang Terdeteksi&quot;

**Penyebab kemungkinan:**

* Lupa menandai gambar target
* Gambar target tidak mengandung target yang terlihat
* Pengaturan deteksi target terlalu ketat

**Solusi:**

1. Periksa [Memilih Gambar Target](choosing-target-images.md)
2. Tandai gambar yang sesuai di kolom Target
3. Verifikasi target terlihat di gambar yang ditandai
4. Sesuaikan pengaturan deteksi target jika diperlukan

***

## Tips untuk Pemrosesan yang Sukses

### Sebelum Memulai

1. **Uji dengan subset kecil terlebih dahulu** - Proses 10-20 gambar untuk memverifikasi pengaturan
2. **Periksa ruang disk yang tersedia** - Pastikan ada ruang kosong 2-3 kali ukuran dataset
3. **Tutup aplikasi yang tidak diperlukan** - Bebaskan sumber daya sistem
4. **Verifikasi gambar target** - Pratinjau target yang ditandai untuk memastikan kualitas
5. **Simpan proyek** - Proyek disimpan otomatis, tetapi disarankan untuk menyimpan secara manual

### Selama Pemrosesan

1. **Hindari mode tidur sistem** - Nonaktifkan mode hemat daya
2. **Jaga Chloros di latar depan** - Atau setidaknya terlihat di bilah tugas
3. **Pantau kemajuan sesekali** - Periksa peringatan atau kesalahan
4. **Jangan memuat aplikasi berat lainnya** - Terutama saat menggunakan mode paralel Chloros+

### Akselerasi GPU Chloros+

Jika menggunakan akselerasi GPU NVIDIA:

1. Perbarui driver NVIDIA ke versi terbaru
2. Pastikan GPU memiliki VRAM 4GB+
3. Tutup aplikasi yang membutuhkan GPU tinggi (game, pengeditan video)
4. Pantau suhu GPU (pastikan pendinginan yang memadai)

***

## Langkah Selanjutnya

Setelah pemrosesan dimulai:

1. **Pantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)
2. **Tunggu hingga selesai** - Pemrosesan berjalan secara otomatis
3. **Periksa hasil** - Lihat [Menyelesaikan Pemrosesan](finishing-the-processing.md)

Untuk informasi tentang apa yang harus dilakukan selama pemrosesan, lihat [Memantau Pemrosesan](monitoring-the-processing.md).

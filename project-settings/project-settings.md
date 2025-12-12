# Pengaturan Proyek

Pengaturan Proyek <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> di Chloros memungkinkan Anda untuk mengonfigurasi semua aspek pemrosesan gambar, deteksi target kalibrasi, perhitungan indeks multispektral, dan opsi ekspor untuk proyek Anda. Pengaturan ini disimpan bersama proyek Anda dan dapat disimpan sebagai templat untuk digunakan kembali di proyek-proyek lain.

## Mengakses Pengaturan Proyek

Untuk mengakses Pengaturan Proyek:

1. Buka proyek di Chloros
2. Klik tab **Pengaturan Proyek**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> di bilah sisi kiri
3. Panel pengaturan akan menampilkan semua opsi konfigurasi yang tersedia, disusun berdasarkan kategori

***

## Deteksi Target

Pengaturan ini mengontrol cara Chloros mendeteksi dan memproses target kalibrasi dalam gambar Anda.

### Area sampel kalibrasi minimum (px)

* **Tipe**: Angka
* **Rentang**: 0 hingga 10.000 piksel
* **Default**: 25 piksel
* **Deskripsi**: Menentukan area minimum (dalam piksel) yang diperlukan agar suatu wilayah terdeteksi dianggap sebagai sampel target kalibrasi yang valid. Nilai yang lebih kecil akan mendeteksi target yang lebih kecil tetapi dapat meningkatkan deteksi palsu. Nilai yang lebih besar memerlukan wilayah target yang lebih besar dan jelas untuk deteksi.
* **Kapan menyesuaikan**:
  * Tingkatkan jika Anda mendapatkan deteksi palsu pada artefak gambar kecil
  * Kurangi jika target kalibrasi Anda terlihat kecil dalam gambar dan tidak terdeteksi

### Minimum Pengelompokan Target (0-100)

* **Tipe**: Bilangan
* **Rentang**: 0 hingga 100
* **Default**: 60
* **Deskripsi**: Mengontrol ambang batas pengelompokan untuk menggabungkan wilayah berwarna serupa saat mendeteksi target kalibrasi. Nilai yang lebih tinggi memerlukan warna yang lebih serupa untuk digabungkan, menghasilkan deteksi target yang lebih konservatif. Nilai yang lebih rendah memungkinkan variasi warna yang lebih besar dalam kelompok target.
* **Kapan menyesuaikan**:
  * Tingkatkan jika target kalibrasi terbagi menjadi beberapa deteksi
  * Kurangi jika target kalibrasi dengan variasi warna tidak terdeteksi sepenuhnya

***

## Pemrosesan

Pengaturan ini mengontrol cara Chloros memproses dan mengkalibrasi gambar Anda.

### Koreksi vignette

* **Tipe**: Kotak centang
* **Default**: Diaktifkan (dicentang)
* **Deskripsi**: Menerapkan koreksi vignette untuk mengkompensasi penggelapan lensa di tepi gambar. Vignetting adalah fenomena optik umum di mana sudut dan tepi gambar tampak lebih gelap daripada pusatnya akibat karakteristik lensa.
* **Kapan menonaktifkan**: Hanya nonaktifkan jika kombinasi kamera/lensa Anda sudah menerapkan koreksi vignette, atau jika Anda ingin melakukan koreksi vignette secara manual dalam pengolahan pasca.

### Kalibrasi reflektansi / keseimbangan putih

* **Tipe**: Kotak centang
* **Default**: Diaktifkan (dicentang)
* **Deskripsi**: Mengaktifkan kalibrasi reflektansi otomatis menggunakan target kalibrasi yang terdeteksi dalam gambar Anda. Ini menormalisasi nilai reflektansi di seluruh dataset Anda dan memastikan pengukuran yang konsisten terlepas dari kondisi pencahayaan.
* **Kapan menonaktifkan**: Nonaktifkan hanya jika Anda ingin memproses gambar mentah yang belum dikalibrasi atau jika Anda menggunakan alur kerja kalibrasi yang berbeda.

### Metode Debayer

* **Tipe**: Pilihan dropdown
* **Opsi**:
  * Kualitas Tinggi (Lebih Cepat) - Saat ini opsi satu-satunya yang tersedia
* **Default**: Kualitas Tinggi (Lebih Cepat)
* **Deskripsi**: Memilih algoritma demosaicing yang digunakan untuk mengonversi data sensor pola Bayer mentah menjadi gambar berwarna penuh. Metode &quot;Kualitas Tinggi (Lebih Cepat)&quot; memberikan keseimbangan optimal antara kecepatan pemrosesan dan kualitas gambar.
* **Catatan**: Metode debayer tambahan mungkin ditambahkan pada versi mendatang dari Chloros.

### Interval kalibrasi ulang minimum

* **Tipe**: Angka
* **Rentang**: 0 hingga 3.600 detik
* **Default**: 0 detik
* **Deskripsi**: Menentukan interval waktu minimum (dalam detik) antara penggunaan target kalibrasi. Jika disetel ke 0, Chloros akan menggunakan semua target kalibrasi yang terdeteksi. Jika disetel ke nilai yang lebih tinggi, Chloros hanya akan menggunakan target kalibrasi yang terpisah setidaknya selama ini detik, mengurangi waktu pemrosesan untuk dataset dengan penangkapan target kalibrasi yang sering.
* **Kapan menyesuaikan**:
  * Atur ke 0 untuk akurasi kalibrasi maksimum saat kondisi pencahayaan bervariasi
  * Tingkatkan (misalnya, menjadi 60-300 detik) untuk pemrosesan lebih cepat saat pencahayaan konsisten dan Anda memiliki gambar target kalibrasi yang sering

### Pergeseran zona waktu sensor cahaya

* **Tipe**: Angka
* **Rentang**: -12 hingga +12 jam
* **Default**: 0 jam
* **Deskripsi**: Menentukan offset zona waktu (dalam jam dari UTC) untuk cap waktu data sensor cahaya. Ini digunakan saat memproses file data PPK (Post-Processed Kinematic) untuk memastikan sinkronisasi waktu yang benar antara penangkapan gambar dan data GPS.
* **Kapan disesuaikan**: Atur ini sesuai dengan pergeseran zona waktu lokal Anda jika data PPK Anda menggunakan waktu lokal alih-alih UTC. Contoh:
  * Waktu Pasifik: -8 atau -7 (tergantung pada DST)
  * Waktu Timur: -5 atau -4 (tergantung pada DST)
  * Waktu Eropa Tengah: +1 atau +2 (tergantung pada DST)

### Terapkan koreksi PPK

* **Tipe**: Kotak centang
* **Default**: Dinonaktifkan (tidak dicentang)
* **Deskripsi**: Mengaktifkan penggunaan koreksi Kinematik Pasca-Proses (PPK) dari perekam DAQ MAPIR yang mengandung GPS (GNSS). Saat diaktifkan, Chloros akan menggunakan file log .daq yang mengandung data pin eksposur di direktori proyek Anda dan menerapkan koreksi geolokasi presisi pada gambar Anda.
* **Persyaratan**: File log .daq dengan entri pin eksposur harus ada di direktori proyek Anda
* **Kapan mengaktifkan**: Disarankan untuk selalu mengaktifkan koreksi PPK jika Anda memiliki entri umpan balik eksposur di file log .daq Anda.

### Pin Eksposur 1

* **Tipe**: Pilihan dropdown
* **Visibilitas**: Hanya terlihat saat &quot;Terapkan koreksi PPK&quot; diaktifkan DAN data eksposur tersedia untuk Pin 1
* **Opsi**:
  * Nama model kamera yang terdeteksi dalam proyek
  * &quot;Jangan Gunakan&quot; - Abaikan pin eksposur ini
* **Default**: Dipilih otomatis berdasarkan konfigurasi proyek
* **Deskripsi**: Menugaskan kamera spesifik ke Pin Eksposur 1 untuk sinkronisasi waktu PPK. Pin eksposur mencatat waktu tepat saat rana kamera dipicu, yang kritis untuk geolokasi PPK yang akurat.
* **Perilaku pemilihan otomatis**:
  * Satu kamera + satu pin: Kamera dipilih secara otomatis
  * Satu kamera + dua pin: Pin 1 secara otomatis ditugaskan ke kamera
  * Beberapa kamera: Pemilihan manual diperlukan

### Pin Eksposur 2

* **Tipe**: Pilihan dropdown
* **Visibilitas**: Hanya terlihat saat &quot;Terapkan koreksi PPK&quot; diaktifkan DAN data eksposur tersedia untuk Pin 2
* **Opsi**:
  * Nama model kamera yang terdeteksi dalam proyek
  * &quot;Jangan Digunakan&quot; - Abaikan pin eksposur ini
* **Default**: Dipilih otomatis berdasarkan konfigurasi proyek
* **Deskripsi**: Menugaskan kamera spesifik ke Pin Eksposur 2 untuk sinkronisasi waktu PPK saat menggunakan pengaturan dua kamera.
* **Perilaku pemilihan otomatis**:
  * Satu kamera + satu pin: Pin 2 secara otomatis diatur ke &quot;Jangan Digunakan&quot;
  * Satu kamera + dua pin: Pin 2 secara otomatis diatur ke &quot;Jangan Digunakan&quot;
  * Beberapa kamera: Pemilihan manual diperlukan
* **Catatan**: Kamera yang sama tidak dapat ditugaskan ke Pin 1 dan Pin 2 secara bersamaan.

***

## Indeks

Pengaturan ini memungkinkan Anda mengonfigurasi indeks multispektral untuk analisis dan visualisasi.

### Tambahkan indeks

* **Tipe**: Panel konfigurasi indeks khusus
* **Deskripsi**: Membuka panel interaktif di mana Anda dapat memilih dan mengonfigurasi indeks vegetasi multispektral (NDVI, NDRE, EVI, dll.) untuk dihitung selama pemrosesan gambar. Anda dapat menambahkan beberapa indeks, masing-masing dengan pengaturan visualisasi sendiri.
* **Indeks yang tersedia**: Sistem ini mencakup lebih dari 30 indeks multispektral yang telah didefinisikan sebelumnya, termasuk:
  * NDVI (Indeks Vegetasi Perbedaan Normal)
  * NDRE (Indeks Perbedaan Normal RedEdge)
  * EVI (Indeks Vegetasi Terperbaiki)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Dan banyak lagi (lihat [Rumus Indeks Multispektral](multispectral-index-formulas.md) untuk daftar lengkap)
* **Fitur**:
  * Pilih dari rumus indeks yang telah ditentukan
  * Konfigurasi gradien warna visualisasi (LUT - Tabel Pencarian)
  * Tetapkan nilai ambang batas untuk analisis
  * Buat rumus indeks kustom

### Rumus Kustom (Fitur Chloros+)

* **Tipe**: Array definisi rumus kustom
* **Deskripsi**: Memungkinkan Anda membuat dan menyimpan rumus indeks multispektral kustom menggunakan matematika band. Rumus kustom disimpan bersama pengaturan proyek Anda dan dapat digunakan seperti indeks bawaan.
* **Cara membuat**:
  1. Di panel konfigurasi indeks, cari opsi rumus kustom
  2. Tentukan rumus Anda menggunakan identifikasi band (misalnya, NIR, Red, Green, Blue)
  3. Simpan rumus dengan nama yang deskriptif
* **Sintaks rumus**: Operasi matematika standar didukung, termasuk:  
  * Aritmatika: `+`, `-`, `*`, `/`
  * Kurung untuk urutan operasi
  * Referensi band: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Ekspor

Pengaturan ini mengontrol format dan kualitas gambar yang diekspor.

### Format gambar yang dikalibrasi

* **Tipe**: Pilihan dropdown
* **Opsi**:
  * **TIFF (16-bit)** - Format TIFF 16-bit tanpa kompresi
  * **TIFF (32-bit, Persentase)** - Format TIFF 32-bit floating-point dengan nilai reflektansi sebagai persentase
  * **PNG (8-bit)** - Format PNG terkompresi 8-bit
  * **JPG (8-bit)** - Format JPEG terkompresi 8-bit
* **Default**: TIFF (16-bit)
* **Deskripsi**: Memilih format file untuk menyimpan gambar yang telah diproses dan dikalibrasi.
* **Rekomendasi format**:
  * **TIFF (16-bit)**: Direkomendasikan untuk analisis ilmiah dan alur kerja profesional. Menjaga kualitas data maksimum tanpa artefak kompresi. Terbaik untuk analisis multispektral dan pemrosesan lebih lanjut di perangkat lunak GIS.
  * **TIFF (32-bit, Persentase)**: Terbaik untuk alur kerja yang memerlukan nilai reflektansi sebagai persentase (0-100%). Menawarkan presisi maksimum untuk pengukuran radiometrik.
  * **PNG (8-bit)**: Cocok untuk tampilan web dan visualisasi umum. Ukuran file lebih kecil dengan kompresi tanpa kehilangan data, tetapi rentang dinamis berkurang.  
  * **JPG (8-bit)**: Ukuran file terkecil, terbaik untuk pratinjau dan tampilan web saja. Menggunakan kompresi dengan kehilangan data yang tidak cocok untuk analisis ilmiah.  

***  

## Simpan Template Proyek

Fitur ini memungkinkan Anda menyimpan pengaturan proyek saat ini sebagai templat yang dapat digunakan kembali.

* **Tipe**: Masukan teks + Tombol Simpan
* **Deskripsi**: Masukkan nama deskriptif untuk templat pengaturan Anda dan klik ikon simpan. Templat akan menyimpan semua pengaturan proyek saat ini (deteksi target, opsi pemrosesan, indeks, dan format ekspor) untuk penggunaan ulang yang mudah di proyek mendatang.
* **Kasus penggunaan**:
  * Buat templat untuk sistem kamera yang berbeda (RGB, multispektral, NIR)
  * Simpan konfigurasi standar untuk jenis tanaman tertentu atau alur kerja analisis
  * Bagikan pengaturan yang konsisten di seluruh tim
* **Cara penggunaan**:
  1. Konfigurasikan semua pengaturan proyek yang diinginkan
  2. Masukkan nama templat (misalnya, &quot;RedEdge Survey3 NDVI Standard&quot;)
  3. Klik ikon simpan
  4. Templat kini dapat dimuat saat membuat proyek baru

***

## Folder Penyimpanan Proyek

Pengaturan ini menentukan lokasi penyimpanan default untuk proyek baru.

* **Tipe**: Tampilan jalur direktori + Tombol Edit
* **Default**: `C:\Users\[Username]\Chloros Projects`
* **Deskripsi**: Menampilkan direktori default saat ini tempat proyek Chloros baru dibuat. Klik ikon edit untuk memilih direktori lain.
* **Kapan mengubah**:
  * Atur ke drive jaringan untuk kolaborasi tim
  * Ubah ke drive dengan ruang penyimpanan lebih besar untuk dataset besar
  * Organisasikan proyek berdasarkan tahun, klien, atau jenis proyek di folder yang berbeda
* **Catatan**: Mengubah pengaturan ini hanya memengaruhi PROYEK BARU. Proyek yang sudah ada tetap berada di lokasi aslinya.

***

## Persisten Pengaturan

Semua pengaturan proyek disimpan secara otomatis bersama berkas proyek Anda (format proyek `.mapir`). Saat Anda membuka kembali proyek, semua pengaturan akan dipulihkan persis seperti saat Anda meninggalkannya.

### Hierarki Pengaturan

Pengaturan diterapkan dalam urutan berikut:

1. **Pengaturan default sistem** - Pengaturan default bawaan yang didefinisikan oleh Chloros
2. **Pengaturan templat** - Jika Anda memuat templat saat membuat proyek
3. **Pengaturan proyek yang disimpan** - Pengaturan yang disimpan bersama berkas proyek
4. **Penyesuaian manual** - Perubahan apa pun yang Anda buat selama sesi saat ini

### Pengaturan dan Pemrosesan Gambar

Sebagian besar perubahan pengaturan (terutama di kategori Pemrosesan dan Ekspor) akan memicu pemrosesan ulang gambar untuk mencerminkan pengaturan baru. Namun, beberapa pengaturan bersifat &quot;hanya ekspor&quot; dan tidak memerlukan pemrosesan ulang segera:

* Simpan Template Proyek
* Direktori Kerja
* Format gambar yang dikalibrasi (berlaku saat ekspor)

***

## Praktik Terbaik

1. **Mulai dengan pengaturan default**: Pengaturan default bekerja dengan baik untuk sebagian besar sistem kamera MAPIR dan alur kerja tipikal.
2. **Buat templat**: Setelah mengoptimalkan pengaturan untuk alur kerja atau kamera tertentu, simpan sebagai templat untuk memastikan konsistensi di seluruh proyek.
3. **Uji coba sebelum pemrosesan penuh**: Saat mencoba pengaturan baru, uji coba pada subset kecil gambar sebelum memproses seluruh dataset.
4. **Dokumentasikan pengaturan Anda**: Gunakan nama template yang deskriptif yang menunjukkan sistem kamera, jenis pemrosesan, dan tujuan penggunaan (misalnya, &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Pemilihan format ekspor**: Pilih format ekspor berdasarkan penggunaan akhir:
   * Analisis ilmiah → TIFF (16-bit atau 32-bit)
   * Pengolahan GIS → TIFF (16-bit)
   * Visualisasi cepat → PNG (8-bit)
   * Berbagi di web → JPG (8-bit)

***

Untuk informasi lebih lanjut tentang indeks multispektral di Chloros, lihat halaman [Rumus Indeks Multispektral](multispectral-index-formulas.md).

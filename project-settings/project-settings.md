# Pengaturan Proyek

Panel samping Pengaturan Proyek <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> di Chloros memungkinkan Anda mengonfigurasi semua aspek pemrosesan gambar, deteksi target kalibrasi, perhitungan indeks multispektral, dan opsi ekspor untuk proyek Anda. Pengaturan ini disimpan bersama proyek Anda dan dapat disimpan sebagai templat untuk digunakan kembali di berbagai proyek.

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
* **Deskripsi**: Menentukan area minimum (dalam piksel) yang diperlukan agar suatu wilayah yang terdeteksi dianggap sebagai sampel target kalibrasi yang valid. Nilai yang lebih kecil akan mendeteksi target yang lebih kecil tetapi dapat meningkatkan false positive. Nilai yang lebih besar memerlukan wilayah target yang lebih besar dan jelas untuk deteksi.
* **Kapan harus disesuaikan**:
  * Tingkatkan jika Anda mendapatkan deteksi palsu pada artefak gambar kecil
  * Kurangi jika target kalibrasi Anda tampak kecil dalam gambar dan tidak terdeteksi

### Pengelompokan Target Minimum (0-100)

* **Jenis**: Angka
* **Rentang**: 0 hingga 100
* **Default**: 60
* **Deskripsi**: Mengontrol ambang batas pengelompokan untuk menggabungkan wilayah berwarna serupa saat mendeteksi target kalibrasi. Nilai yang lebih tinggi memerlukan lebih banyak warna serupa untuk digabungkan, menghasilkan deteksi target yang lebih konservatif. Nilai yang lebih rendah memungkinkan variasi warna yang lebih besar dalam kelompok target.
* **Kapan harus disesuaikan**:
  * Tingkatkan jika target kalibrasi terpecah menjadi beberapa deteksi
  * Kurangi jika target kalibrasi dengan variasi warna tidak terdeteksi sepenuhnya

***

## Pemrosesan

Pengaturan ini mengontrol cara Chloros memproses dan mengkalibrasi gambar Anda.

### Koreksi Vignette

* **Jenis**: Kotak centang
* **Default**: Diaktifkan (dicentang)
* **Deskripsi**: Menerapkan koreksi vignette untuk mengkompensasi penggelapan lensa di tepi gambar. Vignetting adalah fenomena optik umum di mana sudut dan tepi gambar tampak lebih gelap daripada bagian tengahnya akibat karakteristik lensa.
* **Kapan dinonaktifkan**: Hanya nonaktifkan jika kombinasi kamera/lensa Anda sudah menerapkan koreksi vignette, atau jika Anda ingin mengoreksi vignetting secara manual dalam pengolahan pasca-pemotretan.

### Kalibrasi reflektansi / keseimbangan putih

* **Jenis**: Kotak centang
* **Default**: Diaktifkan (dicentang)
* **Deskripsi**: Mengaktifkan kalibrasi reflektansi otomatis menggunakan target kalibrasi yang terdeteksi dalam gambar Anda. Ini menormalisasi nilai reflektansi di seluruh dataset Anda dan memastikan pengukuran yang konsisten terlepas dari kondisi pencahayaan.
* **Kapan harus dinonaktifkan**: Nonaktifkan hanya jika Anda ingin memproses gambar mentah yang belum dikalibrasi atau jika Anda menggunakan alur kerja kalibrasi yang berbeda.

### Metode Debayer

* **Tipe**: Pilihan dropdown
* **Opsi**:
  * Standar (Cepat, Kualitas Sedang)
  * Texture Aware (Lambat, Kualitas Tertinggi) \[Chloros+]
* **Default**: Standar (Cepat, Kualitas Sedang)
* **Deskripsi**: Memilih algoritma demosaicing yang digunakan untuk mengubah data sensor pola Bayer mentah menjadi gambar berwarna penuh. Metode &quot;Standar (Cepat, Kualitas Sedang)&quot; memberikan keseimbangan optimal antara kecepatan pemrosesan dan kualitas gambar. Metode &quot;Texture Aware (Lambat, Kualitas Tertinggi)&quot; \[Chloros+] menggunakan algoritma demosaicing berkualitas tinggi yang peka terhadap tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir seluruh noise demosaicing. Model Texture Aware memerlukan memori GPU (VRAM) untuk dijalankan. Kami merekomendasikan penggunaannya jika Anda memiliki &gt;4GB VRAM yang tersedia untuk pemrosesan yang lebih cepat.
* **Catatan**: Metode debayer tambahan mungkin ditambahkan pada versi mendatang dari Chloros.

### Interval kalibrasi ulang minimum

* **Tipe**: Angka
* **Rentang**: 0 hingga 3.600 detik
* **Default**: 0 detik
* **Deskripsi**: Menentukan interval waktu minimum (dalam detik) antara penggunaan target kalibrasi. Jika diatur ke 0, Chloros akan menggunakan setiap target kalibrasi yang terdeteksi. Jika diatur ke nilai yang lebih tinggi, Chloros hanya akan menggunakan target kalibrasi yang terpisah setidaknya sebanyak detik tersebut, sehingga mengurangi waktu pemrosesan untuk dataset dengan penangkapan target kalibrasi yang sering.
* **Kapan harus disesuaikan**:
  * Atur ke 0 untuk akurasi kalibrasi maksimal saat kondisi pencahayaan bervariasi
  * Tingkatkan (misalnya, menjadi 60–300 detik) untuk pemrosesan lebih cepat saat pencahayaan konsisten dan Anda memiliki gambar target kalibrasi yang sering

### Pergeseran zona waktu sensor cahaya

* **Tipe**: Angka
* **Rentang**: -12 hingga +12 jam
* **Default**: 0 jam
* **Deskripsi**: Menentukan selisih zona waktu (dalam jam dari UTC) untuk cap waktu data sensor cahaya. Ini digunakan saat memproses berkas data PPK (Post-Processed Kinematic) untuk memastikan sinkronisasi waktu yang benar antara pengambilan gambar dan data GPS.
* **Kapan harus disesuaikan**: Atur ini ke selisih zona waktu lokal Anda jika data PPK Anda menggunakan waktu lokal alih-alih UTC. Contoh:
  * Waktu Pasifik: -8 atau -7 (tergantung DST)
  * Waktu Timur: -5 atau -4 (tergantung DST)
  * Waktu Eropa Tengah: +1 atau +2 (tergantung DST)

### Terapkan koreksi PPK

* **Tipe**: Kotak centang
* **Default**: Dinonaktifkan (tidak dicentang)
* **Deskripsi**: Mengaktifkan penggunaan koreksi Post-Processed Kinematic (PPK) dari perekam DAQ MAPIR yang dilengkapi GPS (GNSS). Jika diaktifkan, Chloros akan menggunakan file log .daq apa pun yang berisi data pin eksposur di direktori proyek Anda dan menerapkan koreksi geolokasi yang akurat pada gambar Anda.
* **Persyaratan**: File log .daq dengan entri pin eksposur harus ada di direktori proyek Anda
* **Kapan harus diaktifkan**: Disarankan untuk selalu mengaktifkan koreksi PPK jika Anda memiliki entri umpan balik eksposur di file log .daq Anda.

### Pin Eksposur 1

* **Tipe**: Pilihan dropdown
* **Visibilitas**: Hanya terlihat saat &quot;Terapkan koreksi PPK&quot; diaktifkan DAN data eksposur tersedia untuk Pin 1
* **Opsi**:
  * Nama model kamera yang terdeteksi dalam proyek
  * &quot;Jangan Gunakan&quot; - Abaikan pin eksposur ini
* **Default**: Dipilih otomatis berdasarkan konfigurasi proyek
* **Deskripsi**: Menetapkan kamera tertentu ke Pin Eksposur 1 untuk sinkronisasi waktu PPK. Pin eksposur mencatat waktu tepat saat rana kamera dipicu, yang sangat penting untuk geolokasi PPK yang akurat.
* **Perilaku pemilihan otomatis**:
  * Satu kamera + satu pin: Secara otomatis memilih kamera
  * Satu kamera + dua pin: Pin 1 secara otomatis ditetapkan ke kamera
  * Beberapa kamera: Diperlukan pemilihan manual

### Pin Eksposur 2

* **Tipe**: Pilihan dropdown
* **Visibilitas**: Hanya terlihat saat &quot;Terapkan koreksi PPK&quot; diaktifkan DAN data eksposur tersedia untuk Pin 2
* **Opsi**:
  * Nama model kamera yang terdeteksi dalam proyek
  * &quot;Jangan Gunakan&quot; - Abaikan pin eksposur ini
* **Default**: Dipilih otomatis berdasarkan konfigurasi proyek
* **Deskripsi**: Menetapkan kamera tertentu ke Pin Eksposur 2 untuk sinkronisasi waktu PPK saat menggunakan pengaturan dua kamera.
* **Perilaku pemilihan otomatis**:
  * Satu kamera + satu pin: Pin 2 secara otomatis disetel ke &quot;Jangan Gunakan&quot;
  * Satu kamera + dua pin: Pin 2 secara otomatis disetel ke &quot;Jangan Gunakan&quot;
  * Beberapa kamera: Diperlukan pemilihan manual
* **Catatan**: Kamera yang sama tidak dapat ditugaskan ke Pin 1 dan Pin 2 secara bersamaan.***

## Indeks

Pengaturan ini memungkinkan Anda mengonfigurasi indeks multispektral untuk analisis dan visualisasi.

### Tambahkan indeks

* **Tipe**: Panel konfigurasi indeks khusus
* **Deskripsi**: Membuka panel interaktif di mana Anda dapat memilih dan mengonfigurasi indeks vegetasi multispektral (NDVI, NDRE, EVI, dll.) untuk dihitung selama pemrosesan gambar. Anda dapat menambahkan beberapa indeks, masing-masing dengan pengaturan visualisasinya sendiri.
* **Indeks yang tersedia**: Sistem ini mencakup lebih dari 30 indeks multispektral yang telah ditentukan sebelumnya, termasuk:
  * NDVI (Indeks Vegetasi Perbedaan Terstandarisasi)
  * NDRE (Indeks Perbedaan Terstandarisasi RedEdge)
  * EVI (Indeks Vegetasi yang Ditingkatkan)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * Dan masih banyak lagi (lihat [Rumus Indeks Multispektral](multispectral-index-formulas.md) untuk daftar lengkap)
* **Fitur**:
  * Pilih dari rumus indeks yang telah ditentukan
  * Konfigurasi gradien warna visualisasi (LUT - Tabel Pencarian)
  * Tetapkan nilai ambang batas untuk analisis
  * Buat rumus indeks kustom

### Rumus Kustom (Fitur Chloros+)

* **Jenis**: Kumpulan definisi rumus kustom
* **Deskripsi**: Memungkinkan Anda membuat dan menyimpan rumus indeks multispektral kustom menggunakan perhitungan band. Rumus kustom disimpan bersama pengaturan proyek Anda dan dapat digunakan seperti indeks bawaan.
* **Cara membuat**:
  1. Di panel konfigurasi Indeks, cari opsi rumus kustom
  2. Tentukan rumus Anda menggunakan pengenal band (misalnya, NIR, Red, Green, Blue)
  3. Simpan rumus dengan nama yang deskriptif
* **Sintaks rumus**: Operasi matematika standar didukung, termasuk:
  * Aritmatika: `+`, `-`, `*`, `/`
  * Tanda kurung untuk urutan operasi
  * Referensi rentang: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Ekspor

Pengaturan ini mengontrol format dan kualitas gambar yang telah diproses dan diekspor.

### Format gambar yang dikalibrasi

* **Jenis**: Pilihan dropdown
* **Opsi**:
  * **TIFF (16-bit)** - Format TIFF 16-bit tanpa kompresi
  * **TIFF (32-bit, Persen)** - Format TIFF 32-bit floating-point dengan nilai reflektansi sebagai persentase
  * **PNG (8-bit)** - Format PNG 8-bit terkompresi
  * **JPG (8-bit)** - Format JPEG 8-bit terkompresi
* **Default**: TIFF (16-bit)
* **Deskripsi**: Memilih format file untuk menyimpan gambar yang telah diproses dan dikalibrasi.
* **Rekomendasi format**:
  * **TIFF (16-bit)**: Direkomendasikan untuk analisis ilmiah dan alur kerja profesional. Mempertahankan kualitas data maksimum tanpa artefak kompresi. Terbaik untuk analisis multispektral dan pemrosesan lebih lanjut dalam perangkat lunak GIS.
  * **TIFF (32-bit, Persen)**: Terbaik untuk alur kerja yang memerlukan nilai reflektansi dalam bentuk persentase (0-100%). Menawarkan presisi maksimum untuk pengukuran radiometrik.
  * **PNG (8-bit)**: Cocok untuk tampilan web dan visualisasi umum. Ukuran file lebih kecil dengan kompresi tanpa kehilangan data, namun rentang dinamis berkurang.
  * **JPG (8-bit)**: Ukuran file terkecil, paling cocok untuk pratinjau dan tampilan web saja. Menggunakan kompresi dengan kehilangan data yang tidak cocok untuk analisis ilmiah.***

## Simpan Template Proyek

Fitur ini memungkinkan Anda menyimpan pengaturan proyek saat ini sebagai template yang dapat digunakan kembali.

* **Tipe**: Masukan teks + Tombol Simpan
* **Deskripsi**: Masukkan nama deskriptif untuk template pengaturan Anda dan klik ikon simpan. Template ini akan menyimpan semua pengaturan proyek saat ini (deteksi target, opsi pemrosesan, indeks, dan format ekspor) untuk penggunaan ulang yang mudah pada proyek-proyek mendatang.
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
* **Default (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Default (Linux)**: `~/.local/share/chloros/projects`
* **Deskripsi**: Menampilkan direktori default saat ini tempat proyek-proyek baru dibuat. Klik ikon edit untuk memilih direktori yang berbeda.
* **Kapan harus mengubah**:
  * Atur ke drive jaringan untuk kolaborasi tim
  * Ubah ke drive dengan ruang penyimpanan lebih besar untuk dataset besar
  * Atur proyek berdasarkan tahun, klien, atau jenis proyek dalam folder yang berbeda
* **Catatan**: Mengubah pengaturan ini hanya memengaruhi proyek BARU. Proyek yang sudah ada tetap berada di lokasi aslinya.***

## Persisten Pengaturan

Semua pengaturan proyek secara otomatis disimpan bersama berkas proyek Anda (format proyek `.mapir`). Saat Anda membuka kembali proyek, semua pengaturan akan dipulihkan persis seperti yang Anda tinggalkan.

### Hierarki Pengaturan

Pengaturan diterapkan dalam urutan berikut:

1. **Pengaturan default sistem** - Pengaturan default bawaan yang didefinisikan oleh Chloros
2. **Pengaturan templat** - Jika Anda memuat templat saat membuat proyek
3. **Pengaturan proyek yang disimpan** - Pengaturan yang disimpan bersama berkas proyek
4. **Penyesuaian manual** - Perubahan apa pun yang Anda lakukan selama sesi saat ini

### Pengaturan dan Pemrosesan Gambar

Sebagian besar perubahan pengaturan (terutama di kategori Pemrosesan dan Ekspor) akan memicu pemrosesan ulang gambar untuk mencerminkan pengaturan baru. Namun, beberapa pengaturan bersifat &quot;hanya ekspor&quot; dan tidak memerlukan pemrosesan ulang segera:

* Simpan Template Proyek
* Direktori Kerja
* Format gambar yang dikalibrasi (berlaku saat mengekspor)

***

## Praktik Terbaik

1. **Mulailah dengan pengaturan default**: Pengaturan default berfungsi dengan baik untuk sebagian besar sistem kamera MAPIR dan alur kerja umum.
2. **Buat templat**: Setelah Anda mengoptimalkan pengaturan untuk alur kerja atau kamera tertentu, simpan sebagai templat untuk memastikan konsistensi di seluruh proyek.
3. **Uji coba sebelum pemrosesan penuh**: Saat bereksperimen dengan pengaturan baru, uji coba pada subset kecil gambar sebelum memproses seluruh dataset Anda.
4. **Dokumentasikan pengaturan Anda**: Gunakan nama templat yang deskriptif yang menunjukkan sistem kamera, jenis pemrosesan, dan tujuan penggunaan (misalnya, &quot;Survey3\_RGB\_NDVI\_Agriculture&quot;).
5. **Pemilihan format ekspor**: Pilih format ekspor berdasarkan tujuan akhir Anda:
   * Analisis ilmiah → TIFF (16-bit atau 32-bit)
   * Pemrosesan GIS → TIFF (16-bit)
   * Visualisasi cepat → PNG (8-bit)
   * Berbagi di web → JPG (8-bit)

***

Untuk informasi lebih lanjut mengenai indeks multispektral dalam Chloros, lihat halaman [Rumus Indeks Multispektral](multispectral-index-formulas.md).

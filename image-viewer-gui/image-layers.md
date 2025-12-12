# Lapisan Gambar

Menu tarik-turun Lapisan Gambar di Chloros Image Viewer memungkinkan Anda beralih dengan cepat antara berbagai versi gambar yang sama - mulai dari tangkapan asli hingga keluaran reflektansi yang diproses dan gambar indeks yang dihitung.

## Apa itu Lapisan Gambar?

Dalam Chloros, **lapisan** merujuk pada berbagai output gambar yang tersedia untuk satu gambar sumber. Saat memproses gambar, Chloros menciptakan beberapa versi:

* **Gambar asli** (berkas JPG dan RAW dari kamera Anda)
* **Output reflektansi yang dikalibrasi** (jika kalibrasi reflektansi diaktifkan)
* **Gambar target** (jika gambar mengandung target kalibrasi)
* **Gambar indeks** (NDVI, NDRE, GNDVI, dll. jika indeks telah dikonfigurasi)

Menu tarik-turun **Layer Selector** di pojok kanan atas Image Viewer memungkinkan Anda beralih antara versi-versi ini tanpa meninggalkan viewer.

***

## Jenis Lapisan yang Tersedia

### JPG

* Gambar pratinjau JPG asli dari kamera Anda
* Selalu tersedia untuk semua gambar
* Belum diproses, sesuai dengan yang diambil oleh kamera
* Paling cepat untuk dimuat dan ditampilkan

**Kapan harus dilihat:**

* Pratinjau cepat dari tangkapan asli
* Memeriksa komposisi dan bingkai gambar
* Memverifikasi kualitas tangkapan sebelum pemrosesan

### RAW (Asli)

* Data sensor RAW asli dari kamera Anda
* Telah di-debayered tanpa pemrosesan pasca
* Kedalaman bit lebih tinggi daripada JPG (biasanya data sensor 12-bit atau 14-bit)

**Kapan digunakan:**

* Memeriksa kualitas data sensor asli
* Memeriksa masalah sensor atau artefak
* Membandingkan hasil sebelum/sesudah pemrosesan

### RAW (Target)

* Hanya muncul untuk gambar yang teridentifikasi mengandung target kalibrasi
* Menampilkan gambar RAW asli dengan target yang terdeteksi
* Digunakan untuk memverifikasi deteksi target berhasil

**Kapan harus dilihat:**

* Memastikan target kalibrasi terdeteksi dengan benar
* Memeriksa kualitas gambar target
* Memecahkan masalah kalibrasi

{% hint style=&quot;info&quot; %}
**Lapisan Target**: Lapisan ini hanya muncul di menu dropdown untuk gambar yang mengandung target kalibrasi. Gambar tangkapan biasa tidak akan memiliki opsi ini.
{% endhint %}

### RAW (Reflektansi)

* Gambar keluaran reflektansi yang dikalibrasi
* Koreksi vignette (jika diaktifkan dalam pemrosesan)
* Reflektansi dikalibrasi menggunakan data target (jika diaktifkan)
* Multi-band TIFF dengan semua saluran kamera
* Nilai piksel mewakili persentase reflektansi (saat menggunakan mode persentase)
* Siap untuk dimanipulasi dengan [Index/LUT Sandbox](index-lut-sandbox.md)

**Kapan harus dilihat:**

* Memeriksa hasil yang dikalibrasi
* Memverifikasi kualitas kalibrasi
* Memeriksa nilai piksel untuk akurasi ilmiah
* Membandingkan dengan aslinya untuk melihat efek kalibrasi

{% hint style=&quot;success&quot; %}
**Disarankan**: Gunakan lapisan RAW (Reflectance) saat memeriksa nilai piksel untuk pengukuran dan analisis ilmiah.
{% endhint %}

### RAW (NDVI Index)... dan sejenisnya

* Gambar indeks vegetasi yang dihitung (NDVI dalam contoh ini)
* Nama indeks berubah tergantung pada indeks yang dikonfigurasi selama pemrosesan
* Contoh: RAW (NDVI Indeks), RAW (NDRE Indeks), RAW (GNDVI Indeks), dll.
* Gambar grayscale satu band yang menampilkan hasil perhitungan indeks
* Satu lapisan muncul untuk setiap indeks yang dikonfigurasi di Pengaturan Proyek

**Nama indeks yang mungkin:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* Dan banyak lagi... (lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md))

**Kapan harus dilihat:**

* Memeriksa hasil perhitungan indeks
* Memeriksa rentang nilai indeks
* Mengidentifikasi area yang menarik
* Memverifikasi gambar indeks sebelum digunakan dalam GIS atau analisis

***

## Menggunakan Selektor Lapisan

### Membuka Menu Tarik-Turun

1. Buka gambar dalam mode layar penuh (klik thumbnail mana pun di Image Viewer)
2. Temukan **menu tarik-turun lapisan** di pojok kanan atas pemutar gambar
3. Menu tarik-turun menampilkan lapisan yang saat ini dipilih (misalnya, &quot;JPG&quot;)
4. Klik menu tarik-turun untuk melihat semua lapisan yang tersedia

### Mengganti Lapisan

1. Klik menu tarik-turun lapisan untuk membuka daftar
2. Semua lapisan yang tersedia untuk gambar saat ini ditampilkan
3. Klik nama lapisan apa pun untuk beralih ke versi tersebut
4. Gambar akan diperbarui secara instan untuk menampilkan lapisan yang dipilih

**Peralihan cepat:**

* Menu dropdown mengingat pilihan terakhir Anda
* Saat beralih ke gambar berikutnya, Chloros mencoba menampilkan jenis lapisan yang sama
* Jika lapisan tersebut tidak tersedia di gambar berikutnya, secara default akan menggunakan JPG

### Ketersediaan Lapisan

Tidak semua lapisan tersedia untuk setiap gambar:

**Selalu tersedia:**

* ✅ JPG (setiap gambar memiliki pratinjau JPG)

**Tersedia secara kondisional:**

* ⚠️ RAW (Asli) - Hanya jika gambar diambil dalam mode RAW atau RAW+JPG
* ⚠️ RAW (Target) - Hanya jika gambar mengandung target kalibrasi yang terdeteksi
* ⚠️ RAW (Reflektansi) - Hanya setelah diproses dengan kalibrasi reflektansi diaktifkan
* ⚠️ RAW (\[Indeks] Indeks) - Hanya setelah diproses dengan indeks yang dikonfigurasi

***

## Ketahanan Lapisan

### Berpindah Antara Gambar

Saat Anda berpindah ke gambar lain (menggunakan tombol panah atau mengklik thumbnail):

**Preferensi lapisan tetap terjaga:**

* Jika melihat &quot;RAW (Reflektansi)&quot;, gambar berikutnya menampilkan &quot;RAW (Reflektansi)&quot; (jika tersedia)
* Jika melihat &quot;RAW (NDVI Indeks)&quot;, gambar berikutnya menampilkan &quot;RAW (NDVI Indeks)&quot; (jika tersedia)
* Jika lapisan yang sama tidak ada, default ke JPG

**Contoh alur kerja:**

1. Buka Gambar 1, beralih ke RAW (NDVI Index)
2. Tekan → untuk melihat Gambar 2
3. Gambar 2 secara otomatis menampilkan lapisan RAW (NDVI Index)
4. Lanjutkan navigasi - semua gambar menampilkan lapisan NDVI
5. Sangat efisien untuk meninjau hasil indeks di banyak gambar

***

## Alur Kerja Umum

### Alur Kerja 1: Perbandingan Sebelum/Sesudah

**Tujuan**: Membandingkan gambar asli dengan gambar yang telah dikalibrasi

1. Buka gambar yang telah diproses di Image Viewer
2. Pilih **RAW (Asli)** dari menu dropdown
3. Catat efek vignetting dan nilai yang belum dikalibrasi
4. Beralih ke **RAW (Reflektansi)** dari menu dropdown
5. Bandingkan - efek vignetting hilang, nilai telah dikalibrasi

### Alur Kerja 2: Peninjauan Indeks

**Tujuan**: Meninjau hasil NDVI secara cepat di seluruh dataset

1. Buka gambar yang telah diproses pertama
2. Pilih **RAW (NDVI Index)** dari menu dropdown
3. Gunakan tombol panah → untuk berpindah ke gambar berikutnya
4. Lapisan NDVI tetap aktif secara otomatis
5. Lanjutkan melalui semua gambar, periksa pola NDVI
6. Beralih ke **RAW (NDRE Index)** untuk membandingkan

### Alur Kerja 3: Verifikasi Target

**Tujuan**: Memverifikasi semua gambar target terdeteksi dengan benar

1. Navigasi ke gambar target
2. Pilih **RAW (Target)** dari menu dropdown
3. Verifikasi target kalibrasi terlihat jelas dan terdeteksi
4. Navigasi ke gambar target berikutnya
5. Ulangi verifikasi untuk semua target

### Alur Kerja 4: Pemeriksaan Nilai Piksel

**Tujuan**: Memeriksa nilai reflektansi untuk akurasi ilmiah

1. Buka gambar yang telah diproses
2. Pilih lapisan **RAW (Reflektansi)**
3. Aktifkan mode **Persentase Piksel** (tombol di bilah alat kanan atas)
4. Pindahkan kursor ke area vegetasi
5. Verifikasi nilai piksel berada dalam rentang yang diharapkan (30-70% untuk NIR, 5-15% untuk Red)
6. Periksa area tanah dan air untuk nilai yang sesuai

***

## Memahami Nilai Piksel Berdasarkan Lapisan

Lapisan yang berbeda menampilkan rentang nilai piksel yang berbeda:

### Lapisan JPG

* **Rentang**: 0-255 (8-bit)
* **Arti**: Nilai tampilan, dikoreksi gamma
* **Penggunaan**: Pemeriksaan visual saja, tidak untuk pengukuran ilmiah

### RAW (Asli)

* **Rentang**: 0-65535 (16-bit)
* **Arti**: Angka digital sensor mentah
* **Penggunaan**: Memeriksa kinerja sensor, belum dikalibrasi

### RAW (Reflektansi)

* **Rentang**: 0-65.535 (16-bit TIFF) atau 0,0-1,0 (32-bit Persen)
* **Arti**: Persentase reflektansi yang dikalibrasi
* **Penggunaan**: Pengukuran dan analisis ilmiah

**Untuk 16-bit TIFF:** Bagi dengan 65.535 untuk mendapatkan persentase reflektansi **Untuk 32-bit Persentase:** Nilai langsung mewakili persentase (0,5 = 50% reflektansi)

### RAW (Gambar Indeks)

* **Rentang**: Bervariasi tergantung indeks (biasanya -1,0 hingga +1,0 untuk indeks yang dinormalisasi)
* **Arti**: Hasil perhitungan indeks
* **Contoh**:
  * NDVI: -1 hingga +1 (vegetasi biasanya 0,4 hingga 0,9)
  * NDRE: -1 hingga +1 (deteksi stres)
  * EVI: 0 hingga 1 (vegetasi yang ditingkatkan)

***

## Tips dan Praktik Terbaik

### Pengalihan Lapisan yang Efisien

* **Kesadaran pintasan keyboard**: Meskipun tidak ada pintasan keyboard untuk lapisan, tombol panah navigasi (←/→) berfungsi di semua lapisan
* **Alur kerja yang konsisten**: Pilih satu lapisan (misalnya, NDVI) dan tinjau seluruh dataset sebelum beralih ke lapisan lain
* **Perbandingan cepat**: Beralih antara Original dan Reflectance untuk memverifikasi kualitas pemrosesan

### Pertimbangan Kinerja

* **JPG memuat paling cepat**: Gunakan untuk navigasi cepat melalui banyak gambar
* **Lapisan RAW memuat lebih lambat**: Resolusi dan kedalaman bit yang lebih tinggi
* **Lapisan indeks**: Kecepatan serupa dengan lapisan Reflectance
* **Pemuatan pertama paling lambat**: Tampilan selanjutnya dari lapisan yang sama disimpan dalam cache dan lebih cepat

### Verifikasi Kualitas

* **Selalu periksa RAW (Asli)**: Verifikasi kualitas data sumber sebelum mempercayai output yang diproses
* **Bandingkan lapisan**: Gunakan pergantian lapisan untuk memvalidasi pemrosesan berfungsi dengan benar
* **Periksa rentang indeks**: Gunakan mode Persentase Piksel dengan lapisan indeks untuk memverifikasi nilai-nilai yang wajar

***

## Pemecahan Masalah

### Lapisan Tidak Tersedia

**Masalah**: Lapisan yang diharapkan tidak muncul di menu dropdown

**Penyebab kemungkinan:**

* Gambar belum diproses (hanya JPG dan RAW (Asli) yang tersedia)
* Kalibrasi Reflectance dinonaktifkan selama pemrosesan
* Indeks spesifik tidak dikonfigurasi di Pengaturan Proyek
* Gambar adalah gambar target saja (tidak ada indeks yang dihasilkan untuk target)

**Solusi:**

1. Verifikasi gambar telah diproses (periksa folder output untuk file yang diproses)
2. Periksa Pengaturan Proyek untuk memastikan indeks telah dikonfigurasi
3. Proses ulang dengan indeks yang diinginkan diaktifkan

### Lapisan yang Ditampilkan Salah

**Masalah**: Gambar terbuka di lapisan yang tidak diharapkan

**Penyebab**: Preferensi lapisan dari gambar sebelumnya dibawa ke gambar saat ini, tetapi lapisan tersebut tidak ada di gambar saat ini

**Solusi:** Chloros secara otomatis beralih ke JPG saat lapisan yang disukai tidak tersedia - ini adalah perilaku normal

### Tidak Dapat Melihat Target Kalibrasi

**Masalah:** Lapisan RAW (Target) tidak menampilkan deteksi target

**Penyebab Mungkin:**

* Target tidak terdeteksi selama pemrosesan
* Gambar sebenarnya tidak mengandung target
* Pengaturan deteksi target terlalu ketat

**Solusi:**

1. Periksa Log Debug untuk pesan &quot;Target ditemukan&quot;
2. Verifikasi gambar sebenarnya mengandung target kalibrasi yang terlihat
3. Sesuaikan pengaturan deteksi target di Pengaturan Proyek
4. Lihat [Memilih Gambar Target](../processing-images-gui/choosing-target-images.md)

***

## Fitur Terkait

### Alat Pemirsa Gambar

Saat melihat lapisan apa pun, Anda dapat menggunakan:

* **Kontrol zoom**: Perbesar untuk memeriksa detail
* **Pan**: Klik dan seret untuk berpindah di gambar yang diperbesar
* **Pemeriksaan nilai piksel**: Lihat nilai di lokasi kursor
* **Panah navigasi**: Pindah antar gambar sambil mempertahankan lapisan
* **Mode Persentase Piksel**: Beralih antara tampilan DN dan persentase

Lihat [Membuka Gambar Penuh Layar](opening-an-image-full-screen.md) untuk dokumentasi lengkap Penampil Gambar.

### Sandbox Indeks/LUT

Untuk pengujian dan visualisasi indeks interaktif:

* **Perhitungan indeks real-time**: Uji rumus indeks yang berbeda
* **Pemetaan warna LUT**: Terapkan gradien warna pada indeks abu-abu
* **Ekspor visualisasi**: Simpan gambar indeks berwarna

Lihat [Index/LUT Sandbox](index-lut-sandbox.md) untuk detail.

***

## Langkah Selanjutnya

Sekarang setelah Anda memahami lapisan gambar:

* [**Membuka Gambar Penuh Layar**](opening-an-image-full-screen.md) - Panduan lengkap pemirsa gambar
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualisasi indeks interaktif
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) - Referensi indeks yang tersedia
* [**Menyelesaikan Pemrosesan**](../processing-images-gui/finishing-the-processing.md) - Memahami output yang diproses

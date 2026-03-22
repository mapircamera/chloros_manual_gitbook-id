# Menyesuaikan Pengaturan Proyek

Sebelum memproses gambar Anda, penting untuk mengonfigurasi pengaturan proyek agar sesuai dengan kebutuhan alur kerja Anda. Panel Pengaturan Proyek <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> memberikan kontrol menyeluruh atas kalibrasi, opsi pemrosesan, indeks multispektral, dan format ekspor.

## Mengakses Pengaturan Proyek

1. Buka proyek Anda di Chloros
2. Klik ikon **Pengaturan Proyek** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> di bilah sisi kiri
3. Panel Pengaturan Proyek menampilkan semua opsi konfigurasi

{% hint style="info" %}
**Pengaturan disimpan secara otomatis** bersama proyek Anda. Saat Anda membuka kembali proyek, semua pengaturan akan dipulihkan.
{% endhint %}

***

## Pengaturan Cepat untuk Alur Kerja Umum

### Pengaturan Default (Disarankan untuk Sebagian Besar Pengguna)

Untuk alur kerja kamera MAPIR dan Survey3 yang umum, pengaturan default berfungsi dengan baik:

* ✅ **Koreksi vignette**: Diaktifkan
* ✅ **Kalibrasi reflektansi**: Diaktifkan (membutuhkan gambar target MAPIR)
* ✅ **Metode debayer**: Standar (Cepat, Kualitas Sedang)
* ✅ **Format ekspor**: TIFF (16-bit)

Cukup impor gambar Anda dan mulai pemrosesan dengan pengaturan default ini.

***

## Ringkasan Pengaturan Proyek

Panel Pengaturan Proyek dibagi menjadi beberapa kategori. Berikut adalah ringkasan setiap bagian. Untuk dokumentasi lengkap, lihat [Pengaturan Proyek](../project-settings/project-settings.md).

### Deteksi Target

Mengontrol cara Chloros mengidentifikasi target kalibrasi dalam gambar Anda.

**Pengaturan utama:*** **Area sampel kalibrasi minimum**: Ambang batas ukuran untuk deteksi target (default: 25 piksel)
* **Pengelompokan target minimum**: Ambang batas kesamaan untuk mengelompokkan wilayah target (default: 60)**Kapan harus menyesuaikan:**

* Tingkatkan area sampel jika terjadi deteksi palsu
* Kurangi jika target tidak terdeteksi
* Sesuaikan pengelompokan jika target terpecah menjadi beberapa deteksi

### Pemrosesan

Opsi pemrosesan gambar utama dan kalibrasi.

**Pengaturan utama:*** **Koreksi vignette**: Mengkompensasi penggelapan lensa di tepi ✅ Disarankan
* **Kalibrasi reflektansi**: Menormalisasi nilai menggunakan target kalibrasi ✅ Disarankan
* **Metode Debayer**: Algoritma untuk mengonversi RAW menjadi multi-spektral 3-saluran
* **Interval kalibrasi ulang minimum**: Waktu antara penggunaan target kalibrasi (0 = gunakan semua)**Pengaturan lanjutan:*** **Pergeseran zona waktu sensor cahaya**: Untuk sinkronisasi waktu PPK (default: 0)
* **Terapkan koreksi PPK**: Menggunakan data GPS/pin eksposur dari berkas .daq
* **Pin Eksposur 1/2**: Menetapkan kamera ke pin eksposur untuk pengaturan dua kamera

### Metode Debayer

Saat ini kami menawarkan 2 metode debayering di Chloros:

#### Standar (Cepat, Kualitas Sedang)

Metode debayer Standar memproses dengan cepat tetapi menampilkan noise warna akibat debayering, sehingga menghasilkan gambar yang kurang akurat dan lebih berisik.

#### Texture Aware (Lambat, Kualitas Tertinggi) \[Hanya di Chloros+]

Texture Aware menggunakan metode debayer berkualitas tinggi yang peka tepi, dikombinasikan dengan model denoising AI/ML yang menghilangkan hampir seluruh noise debayering. Model Texture Aware memerlukan memori GPU (VRAM) untuk dijalankan. Kami merekomendasikan penggunaannya jika Anda memiliki &gt;4GB VRAM yang tersedia untuk pemrosesan yang lebih cepat.

### Indeks (Indeks Multispektral)

Konfigurasikan indeks vegetasi mana yang akan dihitung dan diekspor.

**Cara menambahkan indeks:**

1. Klik tombol**&quot;Tambahkan indeks&quot;**

2. Pilih indeks dari menu dropdown (NDVI, NDRE, GNDVI, dll.)
3. Konfigurasikan pengaturan visualisasi (warna LUT, rentang nilai)
4. Tambahkan beberapa indeks sesuai kebutuhan

**Indeks populer:*** **NDVI**: Kesehatan vegetasi umum (paling umum)
* **NDRE**: Deteksi stres dini bersama RedEdge
* **GNDVI**: Sensitif terhadap konsentrasi klorofil
* **OSAVI**: Berfungsi baik pada tanah yang terlihat
* **EVI**: Wilayah dengan indeks luas daun tinggi (LAI)**Rumus kustom (hanya untuk Chloros+):**

* Buat rumus indeks multispektral kustom
* Gunakan perhitungan band dengan semua kanal gambar
* Simpan rumus kustom untuk digunakan kembali

Untuk semua indeks dan rumus yang tersedia, lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md).

### Ekspor

Mengontrol format dan kualitas file keluaran.

**Format yang tersedia:*** **TIFF (16-bit)**: Direkomendasikan untuk analisis GIS dan ilmiah (rentang 0-65.535)
* **TIFF (32-bit, Persentase)**: Nilai reflektansi bilangan floating-point (rentang 0,0-1,0)
* **PNG (8-bit)**: Kompresi tanpa kehilangan data untuk visualisasi (rentang 0-255)
* **JPG (8-bit)**: Ukuran file terkecil, kompresi dengan kehilangan data (rentang 0-255)***

## Menyimpan dan Memuat Pengaturan

### Simpan Template Proyek

Buat templat yang dapat digunakan ulang untuk alur kerja yang konsisten:

1. Konfigurasikan semua pengaturan yang diinginkan di panel Pengaturan Proyek
2. Gulir ke bagian **&quot;Simpan Templat Proyek&quot;** di bagian bawah
3. Masukkan nama templat yang deskriptif (misalnya, &quot;Survey3N\_RGN\_Agriculture&quot;)
4. Klik ikon simpan

**Manfaat:**

* Terapkan pengaturan yang sama di berbagai proyek
* Bagikan konfigurasi dengan anggota tim
* Pertahankan konsistensi untuk survei berulang

### Memuat Template pada Proyek Baru

Saat membuat proyek baru:

1. Pilih **&quot;Proyek Baru&quot;** dari menu utama
2. Pilih opsi **&quot;Muat dari template&quot;**

3. Pilih template yang telah Anda simpan
4. Semua pengaturan akan diterapkan secara otomatis

### Direktori Kerja

Pengaturan **&quot;Folder Simpan Proyek&quot;** menentukan lokasi default pembuatan proyek baru:

* **Lokasi default**: `C:\Users\[Username]\Chloros Projects`
* **Ubah lokasi**: Klik ikon edit dan pilih folder baru
* **Kapan harus diubah**:
  * Drive jaringan untuk kolaborasi tim
  * Drive lain dengan ruang penyimpanan lebih besar
  * Struktur folder terorganisir berdasarkan tahun/klien

***

## Pengaturan PPK (Post-Processed Kinematic)

Jika menggunakan perekam DAQ MAPIR dengan GPS untuk geolokasi yang akurat:

### Persyaratan

* MAPIR DAQ dengan modul GPS (GNSS)
* Berkas log .daq dengan entri pin eksposur
* Kamera terhubung ke pin eksposur DAQ selama sesi perekaman

### Langkah Konfigurasi

1. Letakkan berkas log .daq di folder proyek Anda
2. Di Pengaturan Proyek, aktifkan kotak centang **&quot;Terapkan koreksi PPK&quot;**

3. Atur**&quot;Light sensor timezone offset&quot;** jika diperlukan (default: 0 untuk UTC)
4. Tetapkan kamera ke pin eksposur:
   * **Kamera tunggal**: Secara otomatis ditugaskan ke Pin 1
   * **Kamera ganda**: Tetapkan masing-masing kamera ke pin yang benar secara manual**Penetapan Pin Eksposur:*** **Pin Eksposur 1**: Pilih model kamera dari menu tarik-turun
* **Pin Eksposur 2**: Pilih kamera kedua atau &quot;Jangan Digunakan&quot;
* Kamera yang sama tidak dapat ditugaskan ke kedua pin

{% hint style="warning" %}
**Penting**: Pin eksposur harus ditugaskan dengan benar ke kamera masing-masing. Penugasan yang salah akan mengakibatkan data geolokasi yang salah.
{% endhint %}

***

## Skenario Lanjutan

### Proyek Multi-Kamera

Saat memproses gambar dari beberapa kamera MAPIR dalam satu proyek:

1. Chloros secara otomatis mendeteksi model setiap kamera
2. Setiap kamera mendapatkan profil pemrosesan yang sesuai
3. PPK: Tetapkan secara manual setiap kamera ke pin eksposur yang benar
4. Semua kamera menggunakan format ekspor dan indeks yang sama

**Contoh**: Rig kamera ganda Survey3W RGN + Survey3N OCN

### Survei Time-Lapse atau Multi-Tanggal

Untuk survei berulang di area yang sama sepanjang waktu:

1. Buat templat dengan pengaturan standar Anda
2. Gunakan pengaturan target kalibrasi yang konsisten pada setiap sesi
3. Olah setiap tanggal sebagai proyek terpisah
4. Gunakan pengaturan yang identik untuk hasil yang dapat dibandingkan
5. Ekspor dalam format yang sama untuk analisis temporal

### Kumpulan Data Besar

Untuk proyek dengan banyak gambar (500+):

* Pertimbangkan untuk membagi menjadi proyek-proyek lebih kecil berdasarkan tanggal atau area
* Gunakan pemrosesan paralel Chloros+ untuk hasil yang lebih cepat
* Pertimbangkan CLI atau API untuk otomatisasi batch
* Sesuaikan interval kalibrasi ulang minimum untuk mengurangi waktu deteksi target

***

## Memverifikasi Pengaturan Anda

Sebelum memulai pemrosesan, periksa pengaturan kunci berikut:

* [ ] Model kamera terdeteksi dengan benar di File Browser
* [ ] Koreksi vignette diaktifkan
* [ ] Kalibrasi reflektansi diaktifkan
* [ ] Setidaknya satu gambar target kalibrasi telah diimpor
* [ ] Indeks multispektral yang diinginkan telah ditambahkan
* [ ] Format ekspor sesuai dengan alur kerja Anda
* [ ] Pengaturan PPK telah dikonfigurasi (jika menggunakan file .daq dengan peristiwa eksposur)

***

## Langkah Selanjutnya

Setelah pengaturan Anda dikonfigurasi:

1. **Tandai gambar target kalibrasi** - Lihat [Memilih Gambar Target](choosing-target-images.md)
2. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
3. **Pantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)

Untuk detail lengkap mengenai semua pengaturan yang tersedia, lihat dokumentasi referensi [Pengaturan Proyek](../project-settings/project-settings.md).

# Mengatur Pengaturan Proyek

Sebelum memproses gambar Anda, penting untuk mengonfigurasi pengaturan proyek agar sesuai dengan kebutuhan alur kerja Anda. Panel Pengaturan Proyek <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> memberikan kontrol komprehensif atas kalibrasi, opsi pemrosesan, indeks multispektral, dan format ekspor.

## Mengakses Pengaturan Proyek

1. Buka proyek Anda di Chloros
2. Klik ikon **Pengaturan Proyek** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> di bilah sisi kiri
3. Panel Pengaturan Proyek menampilkan semua opsi konfigurasi

{% hint style=&quot;info&quot; %}
**Pengaturan disimpan secara otomatis** bersama proyek Anda. Saat Anda membuka kembali proyek, semua pengaturan akan dipulihkan.
{% endhint %}

***

## Pengaturan Cepat untuk Alur Kerja Umum

### Pengaturan Default (Disarankan untuk Kebanyakan Pengguna)

Untuk alur kerja kamera MAPIR Survey3 yang umum, pengaturan default berfungsi dengan baik:

* ✅ **Koreksi vignette**: Diaktifkan
* ✅ **Kalibrasi reflektansi**: Diaktifkan (membutuhkan gambar target MAPIR)
* ✅ **Metode Debayer**: Kualitas Tinggi (Lebih Cepat)
* ✅ **Format Ekspor**: TIFF (16-bit)

Cukup impor gambar Anda dan mulai pemrosesan dengan pengaturan default ini.

***

## Ringkasan Pengaturan Proyek

Panel Pengaturan Proyek dibagi menjadi beberapa kategori. Berikut adalah ringkasan setiap bagian. Untuk dokumentasi lengkap, lihat [Pengaturan Proyek](../project-settings/project-settings.md).

### Deteksi Target

Mengontrol cara Chloros mengidentifikasi target kalibrasi dalam gambar Anda.

**Pengaturan utama:**

* **Area sampel kalibrasi minimum**: Ambang batas ukuran untuk deteksi target (default: 25 piksel)
* **Pengelompokan target minimum**: Ambang batas kesamaan untuk mengelompokkan wilayah target (default: 60)

**Kapan menyesuaikan:**

* Tingkatkan area sampel jika terjadi deteksi palsu
* Kurangi jika target tidak terdeteksi
* Sesuaikan pengelompokan jika target terbagi menjadi beberapa deteksi

### Pemrosesan

Opsi pemrosesan dan kalibrasi gambar utama.

**Pengaturan utama:**

* **Koreksi vignette**: Mengkompensasi penggelapan lensa di tepi ✅ Direkomendasikan
* **Kalibrasi reflektansi**: Menormalkan nilai menggunakan target kalibrasi ✅ Direkomendasikan
* **Metode debayer**: Algoritma untuk mengonversi RAW menjadi multi-spektral 3-saluran
* **Interval kalibrasi minimum**: Waktu antara penggunaan target kalibrasi (0 = gunakan semua)

**Pengaturan lanjutan:**

* **Pergeseran zona waktu sensor cahaya**: Untuk sinkronisasi waktu PPK (default: 0)
* **Terapkan koreksi PPK**: Menggunakan data GPS/pin eksposur dari file .daq
* **Pin Eksposur 1/2**: Menugaskan kamera ke pin eksposur untuk pengaturan dua kamera

### Indeks (Indeks Multispektral)

Konfigurasikan indeks vegetasi mana yang akan dihitung dan diekspor.

**Cara menambahkan indeks:**

1. Klik tombol **&quot;Tambah indeks&quot;**
2. Pilih indeks dari menu dropdown (NDVI, NDRE, GNDVI, dll.)
3. Konfigurasikan pengaturan visualisasi (warna LUT, rentang nilai)
4. Tambahkan indeks tambahan sesuai kebutuhan

**Indeks populer:**

* **NDVI**: Kesehatan vegetasi umum (paling umum)
* **NDRE**: Deteksi stres dini dengan RedEdge
* **GNDVI**: Sensitif terhadap konsentrasi klorofil
* **OSAVI**: Berfungsi baik dengan tanah yang terlihat
* **EVI**: Wilayah dengan indeks area daun tinggi (LAI)

**Rumus kustom (hanya Chloros+):**

* Buat formula indeks multispektral kustom
* Gunakan perhitungan band dengan semua kanal gambar
* Simpan formula kustom untuk penggunaan ulang

Untuk semua indeks dan formula yang tersedia, lihat [Formula Indeks Multispektral](../project-settings/multispectral-index-formulas.md).

### Ekspor

Mengontrol format dan kualitas file output.

**Format yang tersedia:**

* **TIFF (16-bit)**: Direkomendasikan untuk analisis GIS dan ilmiah (rentang 0-65.535)
* **TIFF (32-bit, Persentase)**: Nilai reflektansi bilangan floating-point (rentang 0,0-1,0)
* **PNG (8-bit)**: Kompresi tanpa kehilangan data untuk visualisasi (rentang 0-255)
* **JPG (8-bit)**: Ukuran file terkecil, kompresi dengan kehilangan data (rentang 0-255)

***

## Menyimpan dan Memuat Pengaturan

### Simpan Template Proyek

Buat templat yang dapat digunakan ulang untuk alur kerja yang konsisten:

1. Konfigurasikan semua pengaturan yang diinginkan di panel Pengaturan Proyek
2. Gulir ke bagian **&quot;Simpan Templat Proyek&quot;** di bagian bawah
3. Masukkan nama templat yang deskriptif (misalnya, &quot;Survey3N\_RGN\_Agriculture&quot;)
4. Klik ikon simpan

**Manfaat:**

* Terapkan pengaturan yang sama di beberapa proyek
* Bagikan konfigurasi dengan anggota tim
* Pertahankan konsistensi untuk survei berulang

### Memuat Template pada Proyek Baru

Saat membuat proyek baru:

1. Pilih **&quot;Proyek Baru&quot;** dari menu utama
2. Pilih opsi **&quot;Memuat dari template&quot;**
3. Pilih template yang telah disimpan
4. Semua pengaturan diterapkan secara otomatis

### Direktori Kerja

Pengaturan **&quot;Folder Penyimpanan Proyek&quot;** menentukan lokasi default untuk membuat proyek baru:

* **Lokasi default**: `C:\Users\[Username]\Chloros Projects`
* **Ubah lokasi**: Klik ikon edit dan pilih folder baru
* **Kapan mengubah**:
  * Drive jaringan untuk kolaborasi tim
  * Drive lain dengan ruang penyimpanan lebih besar
  * Struktur folder terorganisir berdasarkan tahun/klien

***

## Pengaturan PPK (Post-Processed Kinematic)

Jika menggunakan perekam DAQ MAPIR dengan GPS untuk penentuan lokasi geografis yang presisi:

### Persyaratan

* Perekam DAQ MAPIR dengan modul GPS (GNSS)
* Berkas log .daq dengan entri pin eksposur
* Kamera terhubung ke pin eksposur DAQ selama sesi perekaman

### Langkah Konfigurasi

1. Letakkan berkas log .daq di folder proyek Anda
2. Di Pengaturan Proyek, aktifkan kotak centang **&quot;Terapkan koreksi PPK&quot;**
3. Atur **&quot;Pergeseran zona waktu sensor cahaya&quot;** jika diperlukan (default: 0 untuk UTC)
4. Atur kamera ke pin eksposur:
   * **Kamera tunggal**: Ditetapkan secara otomatis ke Pin 1
   * **Dua kamera**: Tetapkan setiap kamera ke pin yang benar secara manual

**Penetapan Pin Eksposur:**

* **Pin Eksposur 1**: Pilih model kamera dari daftar dropdown
* **Pin Eksposur 2**: Pilih kamera kedua atau &quot;Jangan Digunakan&quot;
* Kamera yang sama tidak dapat ditugaskan ke kedua pin

{% hint style=&quot;warning&quot; %}
**Penting**: Pin eksposur harus ditugaskan dengan benar ke kamera masing-masing. Penugasan yang salah akan menghasilkan data geolokasi yang salah.
{% endhint %}

***

## Skenario Lanjutan

### Proyek Multi-Kamera

Saat memproses gambar dari beberapa kamera MAPIR dalam satu proyek:

1. Chloros secara otomatis mendeteksi model setiap kamera
2. Setiap kamera mendapatkan profil pemrosesan yang sesuai
3. PPK: Tetapkan secara manual setiap kamera ke pin eksposur yang benar
4. Semua kamera menggunakan format ekspor dan indeks yang sama

**Contoh**: Survey3W RGN + Survey3N OCN rig kamera ganda

### Survei Time-Lapse atau Multi-Tanggal

Untuk survei berulang di area yang sama sepanjang waktu:

1. Buat templat dengan pengaturan standar Anda
2. Gunakan pengaturan target kalibrasi yang konsisten setiap sesi
3. Proses setiap tanggal sebagai proyek terpisah
4. Gunakan pengaturan yang identik untuk hasil yang dapat dibandingkan
5. Ekspor dalam format yang sama untuk analisis temporal

### Set Data Besar

Untuk proyek dengan banyak gambar (500+):

* Pertimbangkan untuk membagi menjadi proyek-proyek kecil berdasarkan tanggal atau area
* Gunakan pemrosesan paralel Chloros+ untuk hasil lebih cepat
* Pertimbangkan CLI atau API untuk otomatisasi batch
* Sesuaikan interval kalibrasi minimum untuk mengurangi waktu deteksi target

***

## Memverifikasi Pengaturan Anda

Sebelum memulai pemrosesan, periksa pengaturan kunci berikut:

* [ ] Model kamera terdeteksi dengan benar di File Browser
* [ ] Koreksi vignette diaktifkan
* [ ] Kalibrasi reflektansi diaktifkan
* [ ] Setidaknya satu gambar target kalibrasi telah diimpor
* [ ] Indeks multispektral yang diinginkan telah ditambahkan
* [ ] Format ekspor sesuai dengan alur kerja Anda
* [ ] Pengaturan PPK dikonfigurasi (jika menggunakan .daq dengan peristiwa eksposur)

***

## Langkah Selanjutnya

Setelah pengaturan Anda dikonfigurasi:

1. **Tandai gambar target kalibrasi** - Lihat [Memilih Gambar Target](choosing-target-images.md)
2. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
3. **Pantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)

Untuk detail lengkap tentang semua pengaturan yang tersedia, lihat dokumentasi referensi [Pengaturan Proyek](../project-settings/project-settings.md).

# Membuka Gambar dalam Layar Penuh

Chloros Image Viewer menyediakan antarmuka layar penuh khusus untuk melihat, menganalisis, dan memanipulasi gambar multispektral Anda. Baik saat melihat gambar asli maupun hasil pemrosesan, Image Viewer menawarkan alat-alat canggih untuk pemeriksaan dan analisis.

## Mengakses Image Viewer

### Dari File Browser

Cara paling umum untuk membuka gambar di Image Viewer:

1. Pastikan Anda berada di tab **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik **thumbnail gambar** mana pun di grid gambar
3. Gambar akan terbuka di **area pratinjau utama** (tengah layar)
4. Gambar kini telah dimuat dan siap untuk ditampilkan dalam mode layar penuh

### Membuka Tab Image Viewer

Setelah gambar dimuat di area pratinjau:

1. Klik ikon **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di bilah sisi kiri
2. Tab Penampil Gambar akan terbuka, menampilkan gambar yang dipilih dalam mode layar penuh
3. Alat tampilan dan analisis lanjutan akan tersedia di bilah sisi kiri

***

## Gambaran Umum Antarmuka Penampil Gambar

### Area Tampilan Utama

Bagian terbesar layar menampilkan gambar Anda:

* **Resolusi penuh**: Gambar ditampilkan pada resolusi asli
* **Dapat diperbesar**: Gunakan kontrol atau roda mouse untuk memperbesar
* **Dapat digeser**: Klik dan seret untuk berpindah saat diperbesar
* **Perbandingan aspek terjaga**: Gambar diskalakan secara proporsional***

## Opsi Tampilan

### Navigasi Gambar Dasar

#### Menelusuri Gambar

Navigasi melalui kumpulan gambar Anda menggunakan pintasan keyboard atau tombol:

* **Gambar berikutnya**: Klik tombol → atau tekan tombol**→** (Panah Kanan)
* **Gambar sebelumnya**: Klik tombol ← atau tekan tombol**←** (Panah Kiri)
* **Lompat ke gambar tertentu**: Kembali ke Penjelajah File dan klik thumbnail yang diinginkan

#### Kontrol Perbesaran

Sesuaikan perbesaran untuk memeriksa detail gambar:

**Perbesar:*** Klik tombol **+** (Plus)
* Tekan tombol **+**atau**=*** Gulir roda mouse **ke atas**

**Perkecil:*** Klik tombol **−** (Minus)
* Tekan tombol **−** (Minus)
* Gulir roda mouse **ke bawah**

#### Geser Saat Diperbesar

Saat diperbesar melebihi ukuran layar:

1. Pindahkan kursor mouse di atas gambar
2. Klik dan **tahan tombol kiri mouse**

3.**Seret** untuk menggeser gambar
4. Lepaskan untuk menghentikan penggeseran

**Alternatif**: Gunakan tombol panah untuk menggeser dalam langkah kecil***

## Pemeriksaan Nilai Piksel

### Melihat Nilai Piksel di Kursor

Saat Anda menggerakkan kursor mouse di atas gambar, nilai piksel ditampilkan secara real-time:**Lokasi tampilan nilai:*** **Angka mengambang dan garis merah di legenda gradien LUT sisi kanan*** **Saat diperbesar lebih jauh, nilai mengambang di dekat kursor dan piksel yang disorot*** Menampilkan nilai untuk piksel **di bawah kursor atau yang disorot*** Diperbarui saat Anda menggerakkan mouse

***

## Jenis Gambar yang Dapat Anda Lihat

### JPG

**Gambar JPG dari kamera:**

* Menampilkan data JPG seperti pratinjau
* Menampilkan nilai asli yang belum dikoreksi
* Berguna untuk memeriksa kualitas gambar sebelum pemrosesan

### RAW (Asli)

### RAW (Reflektansi)

**Setelah pemrosesan:**

* Vignette telah dikoreksi
* Reflektansi telah dikalibrasi
* Multi-band TIFF (Red, Green, NIR, dll.)
* Data ilmiah siap untuk analisis

### RAW (Indeks)

**NDVI, NDRE, GNDVI, dll. (berkas \_NDVI.tif):**

* Gambar skala abu-abu satu kanal
* Nilai piksel mewakili hasil perhitungan indeks
* Rentang biasanya -1 hingga +1 untuk indeks yang dinormalisasi
* Dapat menerapkan LUT warna untuk visualisasi

***

## Penerapan Indeks dan LUT

Terapkan indeks multispektral dan Tabel Pencarian Warna (LUT):

1. Temukan **Index/LUT Sandbox**di**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sidebar
2. Pilih indeks vegetasi (NDVI, NDRE, dll.)
3. Pilih rumus multispektral, atau buat rumus kustom Anda sendiri (hanya Chloros+)
4. Terapkan gradien LUT warna untuk visualisasi
5. Sesuaikan rentang nilai dan ambang batas

Lihat [Index/LUT Sandbox](index-lut-sandbox.md) untuk petunjuk terperinci.

***

## Pintasan Keyboard

### Navigasi

* **→** (Panah Kanan): Gambar berikutnya
* **←** (Panah Kiri): Gambar sebelumnya
* **Home**: Gambar pertama dalam daftar
* **End**: Gambar terakhir dalam daftar

### Zoom

* **+**atau**=**: Perbesar
* **−**: Perkecil
* **Roda Mouse**: Perbesar/perkecil***

### Memverifikasi Perhitungan Indeks

Pastikan indeks dihitung dengan benar:

1. Buka NDVI atau gambar indeks lainnya
2. Periksa area vegetasi:
   * **NDVI**: Harus menunjukkan nilai 0,4–0,9 untuk tanaman sehat
   * **NDRE**: Nilai lebih tinggi untuk pertumbuhan yang subur
   * **GNDVI**: Mirip dengan NDVI tetapi sensitif terhadap klorofil
3. Periksa non-vegetasi:
   * **Tanah**: Dekat 0 atau sedikit negatif
   * **Air**: Nilai negatif (-0,5 hingga 0)***

## Pemecahan Masalah Tampilan

### Gambar Tidak Bisa Dibuka

**Penyebab yang mungkin:**

* File rusak selama pemrosesan
* Format file tidak didukung
* Memori tidak cukup untuk gambar besar

**Solusi:**

1. Coba buka di penampil eksternal untuk memverifikasi integritas file
2. Periksa apakah format file sesuai dengan tipe yang diharapkan
3. Tutup aplikasi lain untuk membebaskan memori
4. Coba gambar yang lebih kecil/berbeda

### Tampilan Gambar Hitam atau Putih

**Penyebab yang mungkin:**

* Rentang nilai di luar kemampuan tampilan
* Gambar float 32-bit dengan nilai yang tidak biasa
* Kesalahan perhitungan indeks

**Solusi:**

1. Periksa nilai piksel - jika semuanya sangat rendah atau sangat tinggi, sesuaikan rentang tampilan
2. Coba buka di QGIS atau aplikasi serupa dengan penyesuaian rentang otomatis
3. Periksa Log Debug dari proses untuk kesalahan

### Nilai Piksel Tampak Salah

**Penyebab yang mungkin:**

* Melihat gambar yang salah (asli vs yang diproses)
* Kalibrasi tidak diterapkan dengan benar
* Data sensor cahaya tidak disertakan dalam input
* Mode persentase diaktifkan/dinonaktifkan secara salah

**Solusi:**

1. Pastikan Anda melihat output yang telah diproses (periksa sufiks nama file)
2. Periksa status tombol mode persentase
3. Bandingkan dengan gambar yang diketahui baik dari dataset yang sama

***

## Langkah Selanjutnya

Sekarang Anda dapat melihat gambar dalam layar penuh:

* [**Lapisan Gambar**](image-layers.md) - Pelajari tentang visualisasi multi-band
* [**Sandbox Indeks/LUT**](index-lut-sandbox.md) - Terapkan indeks kustom dan pemetaan warna
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) - Pahami indeks yang tersedia

Untuk alur kerja pemrosesan, lihat:

* [**Pemrosesan Gambar (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Panduan pemrosesan lengkap

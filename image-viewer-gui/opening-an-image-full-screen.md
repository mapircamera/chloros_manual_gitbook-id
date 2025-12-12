# Membuka Gambar dalam Mode Layar Penuh

Chloros Image Viewer menyediakan antarmuka layar penuh khusus untuk melihat, menganalisis, dan memanipulasi gambar multispektral Anda. Baik saat melihat gambar asli maupun hasil pemrosesan, Image Viewer menawarkan alat-alat canggih untuk inspeksi dan analisis.

## Akses ke Image Viewer

### Dari File Browser

Cara paling umum untuk membuka gambar di Image Viewer:

1. Pastikan Anda berada di tab **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik thumbnail gambar apa pun di grid gambar
3. Gambar terbuka di area pratinjau utama (tengah layar)
4. Gambar sekarang dimuat dan siap untuk ditampilkan dalam mode layar penuh

### Membuka Tab Image Viewer

Setelah gambar dimuat di area pratinjau:

1. Klik ikon **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di bilah sisi kiri
2. Tab Image Viewer terbuka, menampilkan gambar yang dipilih dalam mode layar penuh
3. Alat tampilan dan analisis lanjutan tersedia di bilah sisi kiri

***

## Ringkasan Antarmuka Image Viewer

### Area Tampilan Utama

Bagian terbesar layar menampilkan gambar Anda:

* **Resolusi penuh**: Gambar ditampilkan pada resolusi asli
* **Dapat diperbesar/perkecil**: Gunakan kontrol atau roda mouse untuk memperbesar/perkecil
* **Dapat digeser**: Klik dan seret untuk berpindah saat diperbesar
* **Perbandingan aspek terjaga**: Gambar diskalakan secara proporsional

***

## Opsi Tampilan

### Navigasi Gambar Dasar

#### Menjelajahi Gambar

Navigasi melalui set gambar Anda menggunakan pintasan keyboard atau tombol:

* **Gambar berikutnya**: Klik tombol → atau tekan tombol **→** (Panah Kanan)
* **Gambar sebelumnya**: Klik tombol ← atau tekan tombol **←** (Panah Kiri)
* **Lompat ke gambar tertentu**: Kembali ke File Browser dan klik thumbnail yang diinginkan

#### Kontrol Perbesaran

Sesuaikan perbesaran untuk memeriksa detail gambar:

**Perbesar:**

* Klik tombol **+** (Plus)
* Tekan tombol **+** atau **=**
* Gulir roda mouse ke atas

**Perkecil:**

* Klik tombol **−** (Minus)
* Tekan tombol **−** (Minus)
* Gulir roda mouse ke bawah

**Sesuaikan dengan Layar:**

* Klik tombol **↔** (Sesuaikan)
* Tekan tombol **0** (Nol)
* Klik ganda pada gambar

#### Geser Saat Diperbesar

Saat diperbesar melebihi ukuran layar:

1. Pindahkan kursor mouse di atas gambar
2. Klik dan **tahan tombol mouse kiri**
3. **Seret** untuk memindahkan gambar
4. Lepaskan untuk menghentikan pergerakan

**Alternatif**: Gunakan tombol panah untuk memindahkan gambar dalam increment kecil

***

## Pemeriksaan Nilai Piksel

### Menampilkan Nilai Piksel di Kursor

Saat Anda menggerakkan kursor mouse di atas gambar, nilai piksel ditampilkan secara real-time:

**Lokasi tampilan nilai:**

* **Angka mengambang dan garis merah di legenda gradien LUT indeks sisi kanan**
* **Saat diperbesar lebih lanjut, nilai mengambang di dekat kursor dan piksel yang ditandai**
* Menampilkan nilai untuk piksel **di bawah kursor atau yang ditandai**
* Diperbarui saat Anda menggerakkan mouse

***

## Jenis Gambar yang Dapat Ditampilkan

### Gambar Asli (Pra-Pemrosesan)

**Gambar RAW + JPG dari kamera:**

* Menampilkan data RAW sebagai pratinjau
* Menampilkan nilai asli yang belum dikoreksi
* Berguna untuk memeriksa kualitas gambar sebelum pemrosesan

### Gambar Reflektansi yang Dikalibrasi

**Setelah pemrosesan:**

* Koreksi vignette
* Reflektansi dikalibrasi
* Multi-band TIFF (Red, Green, NIR, dll.)
* Data ilmiah siap untuk analisis

### Gambar Indeks

**NDVI, NDRE, GNDVI, dll. (berkas \_NDVI.tif):**

* Gambar monokrom tunggal
* Nilai piksel mewakili hasil perhitungan indeks
* Rentang biasanya -1 hingga +1 untuk indeks yang dinormalisasi
* Dapat menerapkan tabel warna LUT untuk visualisasi

***

## Aplikasi Indeks dan LUT

Terapkan indeks multispektral dan tabel warna LUT:

1. Temukan **Index/LUT Sandbox** di **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di bilah sisi
2. Pilih indeks vegetasi (NDVI, NDRE, dll.)
3. Pilih rumus multispektral, atau buat rumus kustom Anda sendiri (hanya Chloros+)
4. Terapkan gradien LUT warna untuk visualisasi
5. Sesuaikan rentang nilai dan ambang batas

Lihat [Index/LUT Sandbox](index-lut-sandbox.md) untuk petunjuk detail.

***

## Pintasan Keyboard

### Navigasi

* **→** (Panah Kanan): Gambar berikutnya
* **←** (Panah Kiri): Gambar sebelumnya
* **Home**: Gambar pertama dalam daftar
* **End**: Gambar terakhir dalam daftar

### Perbesaran

* **+** atau **=**: Perbesar
* **−**: Perkecil
* **0** (Nol): Sesuaikan dengan layar
* **Roda Mouse**: Perbesar/perkecil

### Kontrol Tampilan

* **P**: Aktifkan/nonaktifkan mode persentase piksel
* **L**: Aktifkan/nonaktifkan panel lapisan
* **Esc**: Tutup mode layar penuh atau kembali ke Penjelajah File

### Lainnya

* **Ctrl+S**: Simpan gambar saat ini
* **F**: Mode layar penuh (jika tersedia)

***

### Verifikasi Perhitungan Indeks

Pastikan indeks dihitung dengan benar:

1. Buka NDVI atau gambar indeks lainnya
2. Periksa area vegetasi:
   * **NDVI**: Harus menampilkan 0,4-0,9 untuk tanaman sehat
   * **NDRE**: Nilai lebih tinggi untuk pertumbuhan yang vigor
   * **GNDVI**: Mirip dengan NDVI tetapi sensitif terhadap klorofil
3. Periksa area non-vegetasi:
   * **Tanah**: Dekat 0 atau sedikit negatif
   * **Air**: Nilai negatif (-0,5 hingga 0)

***

## Pemecahan Masalah Masalah Tampilan

### Gambar Tidak Bisa Dibuka

**Penyebab kemungkinan:**

* File rusak selama pemrosesan
* Format file tidak didukung
* Memori tidak cukup untuk gambar besar

**Solusi:**

1. Coba buka di pemutar eksternal untuk memverifikasi integritas file
2. Periksa format file sesuai dengan jenis yang diharapkan
3. Tutup aplikasi lain untuk membebaskan memori
4. Coba gambar yang lebih kecil/berbeda

### Tampilan Gambar Hitam atau Putih

**Penyebab kemungkinan:**

* Rentang nilai di luar kemampuan tampilan
* Gambar 32-bit float dengan nilai yang tidak biasa
* Kesalahan perhitungan indeks

**Solusi:**

1. Periksa nilai piksel - jika semua sangat rendah atau sangat tinggi, sesuaikan rentang tampilan
2. Coba buka di QGIS atau aplikasi serupa dengan penyesuaian rentang otomatis
3. Periksa Log Debug dari pemrosesan untuk kesalahan

### Nilai Piksel Tampak Salah

**Penyebab Mungkin:**

* Melihat gambar yang salah (asli vs yang diproses)
* Kalibrasi tidak diterapkan dengan benar
* Data sensor cahaya tidak termasuk dalam input
* Mode persentase diaktifkan secara salah

**Solusi:**

1. Verifikasi bahwa Anda melihat output yang diproses (periksa sufiks nama file)
2. Periksa status tombol mode persentase
3. Bandingkan dengan gambar yang diketahui baik dari dataset yang sama

***

## Langkah Selanjutnya

Sekarang Anda dapat melihat gambar dalam mode layar penuh:

* [**Lapisan Gambar**](image-layers.md) - Pelajari tentang visualisasi multi-band
* [**Sandbox Indeks/LUT**](index-lut-sandbox.md) - Terapkan indeks kustom dan pemetaan warna
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) - Pahami indeks yang tersedia

Untuk alur kerja pemrosesan, lihat:

* [**Pemrosesan Gambar (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Panduan pemrosesan lengkap

# Menyelesaikan Pemrosesan

Setelah Chloros menyelesaikan pemrosesan, saatnya meninjau hasil Anda, memeriksa kualitas keluaran, dan menyiapkan gambar yang telah diproses untuk digunakan dalam alur kerja Anda. Halaman ini akan memandu Anda melalui langkah-langkah terakhir dan tindakan selanjutnya.

## Indikasi Pemrosesan Selesai

Saat pemrosesan berhasil diselesaikan, Anda akan melihat beberapa indikator:

* ✅ **Bilah kemajuan**: Mencapai 100% penyelesaian
* ✅ **Log Debug**: Menampilkan pesan &quot;Pemrosesan Selesai&quot;
* ✅ **Tombol Mulai**: Menjadi aktif kembali (siap untuk pemrosesan berikutnya)
* ✅ **File keluaran**: Semua gambar yang telah diproses disimpan ke subfolder model kamera***

## Menemukan Gambar yang Telah Diproses

### Membuka Folder Keluaran

1. Klik ikon **Menu Utama** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (pojok kiri atas)
2. Pilih **&quot;Buka Folder Proyek&quot;**

3. Penjelajah file Anda akan terbuka ke direktori proyek
4. Temukan proyek Anda berdasarkan namanya

***

## Meninjau Gambar yang Telah Diproses

### Pratinjau Cepat di Penjelajah File

**Pratinjau bawaan Windows:**

1. Navigasi ke subfolder model kamera
2. Pilih berkas gambar
3. Pratinjau muncul di panel pratinjau Windows Explorer
4. Gunakan tombol panah untuk menelusuri gambar

### Pratinjau di Penampil Gambar Eksternal

**Penampil yang direkomendasikan:*** **QGIS** - Perangkat lunak GIS gratis (terbaik untuk analisis multispektral yang direferensikan secara geografis)
* **IrfanView** - Penampil gambar yang cepat dan ringan (mendukung TIFF)
* **Adobe Photoshop** - Pengeditan profesional (mendukung TIFF)
* **GIMP** - Alternatif gratis untuk Photoshop
* **Windows Photos** - Tampilan dasar (mungkin tidak mendukung TIFF 16-bit)

### Pratinjau di Chloros Image Viewer

Gunakan Image Viewer bawaan Chloros untuk visualisasi lanjutan:

1. Klik thumbnail gambar di File Browser
2. Gambar terbuka di area pratinjau utama
3. Klik tab **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di bilah sisi kiri
4. Gunakan [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) untuk analisis interaktif

Lihat [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) untuk petunjuk terperinci.

***

## Meninjau Log Debug

### Periksa Peringatan atau Kesalahan

1. Buka tab **Log Debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> tab
2. Gulir melalui pesan
3. Cari peringatan berwarna kuning atau kesalahan berwarna merah
4. Periksa masalah yang tercatat
5. Hubungi dukungan MAPIR untuk bantuan

### Menyimpan Log

Untuk menyimpan catatan pemrosesan atau mengirim ke Dukungan MAPIR:

1. Klik tombol **&quot;Salin&quot;**atau**&quot;Unduh&quot;**

2. Simpan sebagai berkas teks di folder proyek
3. Sertakan dalam dokumentasi proyek
4. Kirim ke dukungan MAPIR jika menemui masalah

***

## Masalah Keluaran Umum dan Solusinya

### Masalah: Berkas Keluaran Hilang

**Penyebab yang mungkin:**

* File tidak memenuhi kriteria pemrosesan
* Gambar khusus target (dikecualikan dari ekspor)
* Ruang disk habis selama ekspor
* File rusak selama pemrosesan

**Solusi:**

1. Periksa Log Debug untuk pesan skip/kesalahan
2. Pastikan ruang disk mencukupi
3. Hitung file: Harus sesuai dengan (jumlah asli - jumlah target) × (indeks + 1)
4. Impor ulang dan olah ulang file yang hilang

### Masalah: Tepi Gelap atau Terang (Vignetting Masih Terlihat)

**Penyebab kemungkinan:**

* Koreksi vignetting dinonaktifkan
* Kamera/lensa tidak terdaftar dalam basis data profil Chloros
* Vignetting ekstrem melebihi kemampuan koreksi

**Solusi:**

1. Pastikan koreksi vignetting diaktifkan di Pengaturan Proyek
2. Periksa apakah model kamera terdeteksi dengan benar
3. Hubungi dukungan MAPIR jika vignetting tetap terjadi

### Masalah: Warna atau Nilai yang Salah

**Penyebab yang mungkin:**

* Tidak ada target kalibrasi yang terdeteksi
* Model target kalibrasi yang salah dipilih
* Kalibrasi reflektansi dinonaktifkan
* Kualitas gambar target buruk

**Solusi:**

1. Pastikan kalibrasi reflektansi telah diaktifkan
2. Periksa pesan &quot;Target ditemukan&quot; di Log Debug
3. Periksa kualitas gambar target
4. Proses ulang dengan menandai target yang tepat

### Masalah: Nilai NDVI Tampak Salah

**Rentang nilai NDVI yang diharapkan:*** **Air, batu, tanah**: -0,1 hingga 0,2
* **Vegetasi jarang/tidak sehat**: 0,2 hingga 0,4
* **Vegetasi sedang**: 0,4 hingga 0,6
* **Vegetasi sehat dan padat**: 0,6 hingga 0,9**Jika nilai berada di luar rentang ini:**

1. Pastikan kalibrasi reflektansi telah diterapkan
2. Pastikan log sensor cahaya telah disertakan
3. Periksa apakah target kalibrasi terdeteksi
4. Pastikan model kamera yang benar terdeteksi
5. Periksa waktu dan kondisi pengambilan gambar target

***

## Menggunakan Gambar yang Telah Diproses

### Untuk Fotogrametri / Pembuatan Orthomosaik

**Alur kerja yang direkomendasikan:**

1.**Impor gambar reflektansi yang telah dikalibrasi** ke perangkat lunak fotogrametri:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Pertahankan metadata EXIF**: Pastikan data GPS tetap terjaga untuk penandaan geolokasi
3. **Alur kerja yang dikalibrasi**: Gunakan gambar reflektansi untuk akurasi ilmiah
4. **Proses mosaik indeks**: Buat ortomosaik NDVI dari gambar indeks individual
5. **Ekspor GeoTIFF yang telah direferensikan secara geografis**: Untuk digunakan dalam aplikasi GIS

### Untuk Analisis GIS

**Alur kerja yang direkomendasikan:**

1.**Muat ke QGIS, ArcGIS, atau yang serupa**

2.**Gunakan gambar reflektansi 16-bit TIFF** untuk analisis multi-band
3. **Gunakan gambar indeks** (NDVI, NDRE) sebagai lapisan vegetasi siap pakai
4. **Kalkulator raster**: Gabungkan band untuk analisis kustom
5. **Ekspor**: Buat peta klasifikasi, deteksi perubahan, peta kesehatan vegetasi

### Untuk Analisis Langsung / Pelaporan

**Alur kerja yang direkomendasikan:**

1.**Gunakan gambar indeks dengan warna LUT** untuk laporan visual
2. **Ekstrak statistik**: Rata-rata NDVI per bidang/plot
3. **Rangkaian waktu**: Bandingkan indeks di berbagai sesi
4. **Buat laporan**: Sertakan peta, statistik, dan visualisasi***

## Pengarsipan dan Cadangan

### Strategi Cadangan yang Direkomendasikan

**Apa yang harus disimpan:*** ✅ **Gambar RAW/JPG asli** - Arsipkan di drive terpisah/cloud
* ✅ **Hasil pemrosesan** - Simpan gambar yang telah dikalibrasi dan indeks
* ✅ **Berkas proyek** - Berisi semua pengaturan untuk pemrosesan ulang jika diperlukan
* ✅ **Log Debug** - Mencatat detail pemrosesan
* ✅ **Gambar target kalibrasi** - Untuk verifikasi dan pemrosesan ulang**Rekomendasi penyimpanan:*** **Cadangan segera**: Hard drive eksternal
* **Arsip jangka panjang**: Penyimpanan cloud (Google Drive, Dropbox, dll.)
* **Data kritis**: Simpan 2-3 salinan di lokasi yang berbeda***

## Proses Pengolahan Berikutnya

### Menggunakan Kembali Pengaturan Proyek

Jika akan memproses dataset serupa di masa mendatang:

1. **Simpan Template Proyek** (jika belum dilakukan)
2. **Buat proyek baru** menggunakan template yang disimpan
3. **Impor gambar baru**

4.**Proses**dengan pengaturan yang sama untuk konsistensi

### Pemrosesan Batch untuk Beberapa Sesi

Untuk beberapa sesi/kumpulan data:**Opsi 1: GUI - Beberapa Proyek**

* Buat proyek terpisah untuk setiap sesi
* Gunakan pengaturan templat yang konsisten
* Proses satu per satu

**Opsi 2: Chloros CLI (hanya Chloros+)**

* Otomatisasi pemrosesan batch
* Proses beberapa folder dengan skrip
* Lihat [Dokumentasi CLI](../CLI.md)

**Opsi 3: Python SDK (hanya untuk Chloros ke atas)**

* Kontrol programatik
* Integrasi dengan alur kerja analisis
* Lihat [Dokumentasi API](../api-python-sdk.md)

***

## Pemecahan Masalah Pasca-Pemrosesan

### Memproses Ulang dengan Pengaturan Berbeda

Jika hasilnya tidak memuaskan:

1. Simpan gambar asli (jangan pernah dihapus)
2. Buka proyek yang sama di Chloros
3. Sesuaikan pengaturan di panel Pengaturan Proyek
4. Proses ulang - hasil baru akan menggantikan hasil sebelumnya

### Memproses Subset Gambar

Untuk memproses ulang hanya gambar tertentu:

1. Buat proyek baru
2. Impor hanya gambar yang perlu diproses ulang
3. Gunakan templat pengaturan yang sama
4. Proses dataset yang lebih kecil

### Mendapatkan Bantuan

Jika Anda mengalami masalah:

* 📧 **Email**: info@mapir.camera (sertakan Log Debug)
* 🌐 **Dukungan**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Pertanyaan yang Sering Diajukan](../faq.md)
* 📖 **Dokumentasi**: [Manual Chloros](../)***

## Ringkasan: Alur Kerja Lengkap

Anda kini telah menyelesaikan alur kerja pemrosesan Chloros secara lengkap:

1. ✅ **Membuat proyek** - Lihat [Proyek](../projects.md)
2. ✅ **Menambahkan file** - Lihat [Menambahkan File](adding-files-to-a-project.md)
3. ✅ **Menyesuaikan pengaturan** - Lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md)
4. ✅ **Menandai target** - Lihat [Memilih Gambar Target](choosing-target-images.md)
5. ✅ **Memulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
6. ✅ **Memantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)
7. ✅ **Meninjau hasil** - Halaman ini**Gambar multispektral Anda yang telah dikalibrasi dan dikoreksi reflektansinya siap untuk dianalisis!**

***

## Sumber Daya Tambahan

### Fitur Lanjutan

* [**Penerjemah Gambar**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualisasi dan analisis interaktif
* [**Sandbox Indeks/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Pengujian indeks kustom
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) - Referensi indeks lengkap

### Otomatisasi &amp; Integrasi

* [**Dokumentasi CLI**](../CLI.md) - Pemrosesan batch baris perintah
* [**Python SDK**](../api-python-sdk.md) - Otomatisasi programatik
* [**Fitur Chloros+**](../#chloros) - Kemampuan pemrosesan lanjutan

### Dukungan &amp; Pembelajaran

* [**FAQ**](../faq.md) - Jawaban atas pertanyaan umum
* [**Target Kalibrasi**](../calibration-targets.md) - Memahami kalibrasi reflektansi
* [**Kamera yang Didukung**](../supported-cameras.md) - Perangkat keras yang kompatibel

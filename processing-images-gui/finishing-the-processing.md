# Menyelesaikan Pemrosesan

Setelah Chloros menyelesaikan pemrosesan, saatnya untuk meninjau hasil Anda, memverifikasi kualitas output, dan menyiapkan gambar yang telah diproses untuk digunakan dalam alur kerja Anda. Halaman ini memandu Anda melalui langkah-langkah akhir dan tindakan selanjutnya.

## Indikasi Pengolahan Selesai

Ketika pengolahan selesai dengan sukses, Anda akan melihat beberapa indikator:

* ✅ **Progress bar**: Mencapai 100% penyelesaian
* ✅ **Debug Log**: Menampilkan pesan &quot;Processing Complete&quot;
* ✅ **Tombol Mulai**: Menjadi aktif kembali (siap untuk pengolahan berikutnya)
* ✅ **File output**: Semua gambar yang diproses disimpan di subfolder model kamera

***

## Menemukan Gambar yang Telah Diproses

### Membuka Folder Output

1. Klik ikon **Menu Utama** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> (pojok kiri atas)
2. Pilih **&quot;Buka Folder Proyek&quot;**
3. Penjelajah file Anda akan terbuka ke direktori proyek
4. Temukan proyek Anda berdasarkan nama

***

## Meninjau Gambar yang Diolah

### Pratinjau Cepat di File Explorer

**Pratinjau bawaan Windows:**

1. Navigasi ke subfolder model kamera
2. Pilih file gambar
3. Pratinjau muncul di panel pratinjau Windows Explorer
4. Gunakan tombol panah untuk menjelajahi gambar

### Pratinjau di Penampil Gambar Eksternal

**Penampil yang direkomendasikan:**

* **QGIS** - Perangkat lunak GIS gratis (terbaik untuk analisis multispektral yang digeoreferensikan)
* **IrfanView** - Penampil gambar cepat dan ringan (mendukung TIFF)
* **Adobe Photoshop** - Pengeditan profesional (mendukung TIFF)
* **GIMP** - Alternatif gratis untuk Photoshop
* **Windows Photos** - Tampilan dasar (mungkin tidak mendukung TIFF 16-bit)

### Pratinjau di Chloros Image Viewer

Gunakan Image Viewer bawaan Chloros untuk visualisasi lanjutan:

1. Klik thumbnail gambar di File Browser
2. Gambar terbuka di area pratinjau utama
3. Klik tab **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di bilah sisi kiri
4. Gunakan [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) untuk analisis interaktif

Lihat [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) untuk petunjuk detail.

***

## Memeriksa Log Debug

### Periksa Peringatan atau Kesalahan

1. Buka tab **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> tab
2. Gulir melalui pesan
3. Cari peringatan kuning atau kesalahan merah
4. Periksa masalah yang tercatat
5. Hubungi dukungan MAPIR untuk bantuan

### Menyimpan Log

Untuk menyimpan catatan pemrosesan atau mengirim ke dukungan MAPIR:

1. Klik tombol **&quot;Salin&quot;** atau **&quot;Unduh&quot;**
2. Simpan sebagai file teks di folder proyek
3. Sertakan dalam dokumentasi proyek
4. Kirim ke dukungan MAPIR jika mengalami masalah

***

## Masalah Keluaran Umum dan Solusinya

### Masalah: File Keluaran Hilang

**Penyebab kemungkinan:**

* File tidak memenuhi kriteria pemrosesan
* Gambar target saja (dieksepsi dari ekspor)
* Ruang disk habis selama ekspor
* Korupsi file selama pemrosesan

**Solusi:**

1. Periksa Log Debug untuk pesan skip/error
2. Pastikan ruang disk cukup
3. Hitung file: Harus sesuai (jumlah asli - jumlah target) × (indeks + 1)
4. Impor ulang dan olah ulang file yang hilang

### Masalah: Tepi Gelap atau Terang (Vignetting Masih Terlihat)

**Penyebab Mungkin:**

* Koreksi vignetting dinonaktifkan
* Kamera/lensa tidak terdaftar dalam basis data profil Chloros
* Vignetting ekstrem melebihi kemampuan koreksi

**Solusi:**

1. Verifikasi koreksi vignetting diaktifkan di Pengaturan Proyek
2. Periksa model kamera terdeteksi dengan benar
3. Hubungi dukungan MAPIR jika vignetting tetap terjadi

### Masalah: Warna atau Nilai yang Salah

**Penyebab kemungkinan:**

* Tidak ada target kalibrasi yang terdeteksi
* Model target kalibrasi yang salah dipilih
* Kalibrasi reflektansi dinonaktifkan
* Gambar target berkualitas buruk

**Solusi:**

1. Pastikan kalibrasi reflektansi diaktifkan
2. Periksa pesan &quot;Target ditemukan&quot; di Log Debug
3. Periksa kualitas gambar target
4. Olah ulang dengan target yang benar ditandai

### Masalah: Nilai NDVI Tampak Salah

**Rentang nilai NDVI yang diharapkan:**

* **Air, batu, tanah**: -0,1 hingga 0,2
* **Vegetasi jarang/tidak sehat**: 0,2 hingga 0,4
* **Vegetasi sedang**: 0,4 hingga 0,6
* **Vegetasi sehat dan padat**: 0,6 hingga 0,9

**Jika nilai berada di luar rentang ini:**

1. Verifikasi kalibrasi reflektansi telah diterapkan
2. Verifikasi log sensor cahaya telah disertakan
3. Periksa target kalibrasi terdeteksi
4. Pastikan model kamera yang benar terdeteksi
5. Review waktu dan kondisi pengambilan gambar target

***

## Menggunakan Gambar yang Telah Diproses

### Untuk Fotogrametri / Pembuatan Orthomosaik

**Alur kerja yang direkomendasikan:**

1. **Impor gambar reflektansi yang dikalibrasi** ke perangkat lunak fotogrametri:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Pertahankan metadata EXIF**: Pastikan data GPS terjaga untuk geotagging
3. **Alur kerja yang dikalibrasi**: Gunakan gambar reflektansi untuk akurasi ilmiah
4. **Proses mosaik indeks**: Buat mosaik orto NDVI dari gambar indeks individu
5. **Ekspor mosaik georeferensi GeoTIFF**: Untuk penggunaan dalam aplikasi GIS

### Untuk Analisis GIS

**Alur kerja yang direkomendasikan:**

1. **Muat ke QGIS, ArcGIS, atau sejenisnya**
2. **Gunakan gambar reflektansi 16-bit TIFF** untuk analisis multi-band
3. **Gunakan gambar indeks** (NDVI, NDRE) sebagai lapisan vegetasi siap pakai
4. **Penghitung raster**: Gabungkan band untuk analisis kustom
5. **Ekspor**: Buat peta klasifikasi, deteksi perubahan, dan peta kesehatan vegetasi

### Untuk Analisis Langsung / Pelaporan

**Alur kerja yang direkomendasikan:**

1. **Gunakan gambar indeks dengan warna LUT** untuk laporan visual
2. **Ekstrak statistik**: Rata-rata NDVI per bidang/plot
3. **Seri waktu**: Bandingkan indeks di beberapa sesi
4. **Buat laporan**: Sertakan peta, statistik, dan visualisasi

***

## Arsip dan Cadangan

### Strategi Cadangan yang Direkomendasikan

**Apa yang harus disimpan:**

* ✅ **Gambar RAW/JPG asli** - Arsipkan di drive terpisah/cloud
* ✅ **Hasil pemrosesan** - Simpan gambar yang dikalibrasi dan indeks
* ✅ **Berkas proyek** - Mengandung semua pengaturan untuk pemrosesan ulang jika diperlukan
* ✅ **Catatan debug** - Mendokumentasikan detail pemrosesan
* ✅ **Gambar target kalibrasi** - Untuk verifikasi dan pemrosesan ulang

**Rekomendasi penyimpanan:**

* **Cadangan segera**: Drive eksternal
* **Arsip jangka panjang**: Penyimpanan awan (Google Drive, Dropbox, dll.)
* **Data kritis**: Simpan 2-3 salinan di lokasi yang berbeda

***

## Pengolahan Berikutnya

### Menggunakan Ulang Pengaturan Proyek

Jika mengolah dataset serupa di masa depan:

1. **Simpan Template Proyek** (jika belum dilakukan)
2. **Buat proyek baru** menggunakan templat yang disimpan
3. **Impor gambar baru**
4. **Proses** dengan pengaturan yang sama untuk konsistensi

### Pemrosesan Berkelompok untuk Beberapa Sesi

Untuk beberapa sesi/dataset:

**Opsi 1: GUI - Beberapa Proyek**

* Buat proyek terpisah untuk setiap sesi
* Gunakan pengaturan templat yang konsisten
* Proses satu per satu

**Opsi 2: Chloros CLI (hanya Chloros+)**

* Otomatisasi pemrosesan batch
* Proses folder multiple dengan skrip
* Lihat [Dokumentasi CLI](../CLI.md)

**Opsi 3: Python SDK (hanya Chloros+)**

* Kontrol programatik
* Integrasi dengan alur kerja analisis
* Lihat [Dokumentasi API](../api-python-sdk.md)

***

## Pemecahan Masalah Pasca-Pemrosesan

### Memproses Ulang dengan Pengaturan Berbeda

Jika hasil tidak memuaskan:

1. Simpan gambar asli (jangan dihapus)
2. Buka proyek yang sama di Chloros
3. Sesuaikan pengaturan di panel Pengaturan Proyek
4. Proses ulang - hasil akan menggantikan hasil sebelumnya

### Memproses Subset Gambar

Untuk memproses ulang hanya gambar tertentu:

1. Buat proyek baru
2. Impor hanya gambar yang memerlukan pemrosesan ulang
3. Gunakan templat pengaturan yang sama
4. Proses dataset yang lebih kecil

### Mendapatkan Bantuan

Jika Anda mengalami masalah:

* 📧 **Email**: info@mapir.camera (sertakan Log Debug)
* 🌐 **Dukungan**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Pertanyaan yang Sering Diajukan](../faq.md)
* 📖 **Dokumentasi**: [Manual Chloros](../)

***

## Ringkasan: Alur Kerja Lengkap

Anda telah menyelesaikan alur kerja pemrosesan Chloros secara lengkap:

1. ✅ **Proyek dibuat** - Lihat [Proyek](../projects.md)
2. ✅ **Menambahkan file** - Lihat [Menambahkan File](adding-files-to-a-project.md)
3. ✅ **Menyesuaikan pengaturan** - Lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md)
4. ✅ **Menandai target** - Lihat [Memilih Gambar Target](choosing-target-images.md)
5. ✅ **Memulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
6. ✅ **Pemantauan kemajuan** - Lihat [Pemantauan Pemrosesan](monitoring-the-processing.md)
7. ✅ **Pemeriksaan hasil** - Halaman ini

**Gambar multispektral yang telah dikalibrasi dan dikoreksi reflektansi Anda siap untuk dianalisis!**

***

## Sumber Daya Tambahan

### Fitur Lanjutan

* [**Penerima Gambar**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualisasi dan analisis interaktif
* [**Sandbox Indeks/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Pengujian indeks kustom
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) - Referensi indeks lengkap

### Otomatisasi &amp; Integrasi

* [**Dokumentasi CLI**](../CLI.md) - Pemrosesan batch baris perintah
* [**Python SDK**](../api-python-sdk.md) - Otomatisasi programatik
* [**Fitur Chloros+**](../#chloros) - Kemampuan pemrosesan lanjutan

### Dukungan &amp; Pembelajaran

* [**FAQ**](../faq.md) - Pertanyaan umum yang dijawab
* [**Target Kalibrasi**](../calibration-targets.md) - Memahami kalibrasi reflektansi
* [**Kamera yang Didukung**](../supported-cameras.md) - Perangkat keras yang kompatibel

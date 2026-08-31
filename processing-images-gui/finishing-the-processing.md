# Menyelesaikan Pemrosesan

Setelah Chloros menyelesaikan pemrosesan, kini saatnya meninjau hasil Anda, memverifikasi kualitas keluaran, dan mempersiapkan gambar yang telah diproses untuk digunakan dalam alur kerja Anda. Halaman ini akan memandu Anda melalui langkah-langkah akhir dan tindakan selanjutnya.

## Indikasi Pemrosesan Selesai

Saat pemrosesan berhasil selesai, Anda akan melihat beberapa indikator:

* ✅ **Bilah kemajuan**: Mencapai 100% penyelesaian
* ✅ **Log Debug**: Menampilkan baris-baris terakhir `[RUN-SUMMARY]` beserta jumlahnya (gambar, grup kamera, target, gambar yang dikalibrasi, file yang ditulis)
* ✅ **Tombol Mulai**: Menjadi aktif kembali (siap untuk proses berikutnya)
* ✅ **File keluaran**: Semua gambar yang telah diproses disimpan ke dalam pohon keluaran proyek (di bawah)

{% hint style="warning" %}
**Proses yang tidak menulis gambar apa pun dianggap gagal.** Jika Anda meminta produk gambar dan proses tersebut tidak menulis satupun, Chloros akan melaporkannya sebagai kegagalan — `[RUN-SUMMARY]` memberikan petunjuk dalam nama log mengenai penyebab yang mungkin (tidak ada yang diimpor, tidak ada target yang terdeteksi, atau setiap produk yang diminta dilewati karena tidak berlaku). Ekivalen CLI akan keluar dengan nilai non-nol. Proses yang sengaja hanya menghasilkan metadata (semua produk ekspor dinonaktifkan, tanpa indeks) tetap dianggap berhasil. Lihat [Referensi CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Menemukan Gambar yang Telah Diproses

### Membuka Folder Keluaran

1. Klik ikon **Menu Utama** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (pojok kiri atas)
2. Pilih **&quot;Buka Folder Proyek&quot;**

3. Penjelajah file Anda akan terbuka ke direktori proyek
4. Temukan proyek Anda berdasarkan namanya

### Pohon Output

Hasil pengolahan disimpan **di bawah folder proyek, dikelompokkan berdasarkan kamera dan kemudian berdasarkan format file**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Folder kamera**: `LATT-<sensor>-<lens>-F<filter>` untuk LATTICE (sesuai dengan EXIF pengambilan gambar `Model`), `<model>_<filter>` untuk Survey3 (misalnya `Survey3N_RGN`). Dua kamera yang berbagi sensor dan filter tetapi memiliki lensa yang berbeda akan memiliki struktur folder terpisah — vignette, bidang pandang, dan distorsi berbeda.
* **Format folder**: mengikuti pengaturan format ekspor Anda — `tiff16`, `tiff8`, `png8`, `jpg8`, atau `tiff32` hingga TIFF (32-bit, Persen). Radiance selalu berformat float32 dan selalu disimpan di bawah `tiff32`.
* **Folder produk**:
  * `Reflectance_Calibrated_Images/` — reflektansi yang dikalibrasi
  * `Debayered_Images/` — de-bayering linier (LATTICE)
  * `Preview_Images/` — pratinjau tampilan (LATTICE)
  * `Radiance_Images/` — radian spektral float32, W/m²/sr/nm (LATTICE multispektral)
  * `Vignette_Corrected_Images/` **atau** `Sensor_Response_Images/` — nilai cadangan yang belum dikalibrasi untuk bingkai tanpa referensi reflektansi; hanya salah satu dari keduanya yang ada per proses, dipilih berdasarkan pengaturan koreksi Vignette
  * `<INDEX>_Index_Images/` — satu folder per indeks yang dipilih (misalnya `NDVI_Index_Images`)

{% hint style="info" %}
**Setiap produk yang diekspor tetap menggunakan nama file SUMBER.**Ekspor radiance dari `capture_..._raw.tif` tetap bernama `capture_..._raw.tif` — hanya saja berada di dalam `tiff32/Radiance_Images/`.**Folderlah yang mengidentifikasi produk, bukan nama file**, jadi pencarian terhadap `*radiance*.tif` tidak akan menemukan apa pun; gunakan direktori sebagai acuan pencarian.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Berapa Banyak File yang Seharusnya Ada?

Jangan menghitung berdasarkan rumus — jumlah keluaran bergantung pada produk mana yang diaktifkan dan mana yang berlaku untuk masing-masing kamera (misalnya kamera RGB tidak mendapatkan data radiance/reflectance). Jumlah yang akurat tercantum dalam log: baris terakhir `[RUN-SUMMARY]` melaporkan dengan tepat berapa banyak file yang ditulis, dan baris petunjuk menjelaskan apa saja yang dilewati.

***

## Meninjau Gambar yang Telah Diproses

### Pratinjau Cepat di File Explorer

**Pratinjau bawaan Windows:**

1. Buka folder produk (misalnya `tiff16/Reflectance_Calibrated_Images/`)
2. Pilih file gambar
3. Pratinjau akan muncul di panel pratinjau Windows Explorer
4. Gunakan tombol panah untuk menelusuri gambar

### Pratinjau di Penampil Gambar Eksternal

**Penampil yang direkomendasikan:*** **QGIS** - Perangkat lunak GIS gratis (paling cocok untuk analisis multispektral yang direferensikan secara geografis)
* **IrfanView** - Penampil gambar yang cepat dan ringan (mendukung TIFF)
* **Adobe Photoshop** - Pengeditan profesional (mendukung TIFF)
* **GIMP** - Alternatif gratis untuk Photoshop
* **Windows Photos** - Penayangan dasar (mungkin tidak mendukung format 16-bit TIFF)

### Pratinjau di Penampil Gambar Chloros

Gunakan Penampil Gambar bawaan Chloros untuk visualisasi lanjutan:

1. Klik thumbnail gambar di Penjelajah File
2. Gambar terbuka di area pratinjau utama
3. Klik tab **Penampil Gambar** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di bilah sisi kiri
4. Gunakan [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) untuk analisis interaktif

Lihat [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) untuk petunjuk terperinci.

***

## Membaca Nilai Piksel Reflektansi (GIS / Pix4D / Skrip)

Reflektansi disimpan sebagai bilangan bulat DN, dan **nilai DN yang berarti ρ = 1,0 bergantung pada kamera sumber**:

| Sumber          | ρ = 1,0 adalah | Cara mengetahuinya                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (ruang kosong hingga ρ 2,0) | Tag XMP `Chloros:PixelScale=32768` tercantum pada file |
| Survey3         | **65535** (dipotong pada ρ 1,0)     | Tidak ada tag XMP `Chloros:*` — ketidakhadiran itulah yang menjadi petunjuk |

**Baca tag `Chloros:PixelScale` dan bagi dengan nilainya** daripada mengasumsikan nilai 65535 secara seragam — membagi reflektansi LATTICE dengan 65535 secara diam-diam akan membagi dua setiap nilai. Satu kasus khusus tidak memiliki skala sesuai desain: tangkapan sumber 8-bit yang ditulis sebagai keluaran 8-bit akan dipotong, bukan diskalakan ulang, dan sengaja tidak diberi tag skala — ekspor ulang pada 16-bit atau 32-bit alih-alih membaginya. Lihat [Format Gambar Keluaran](../output-image-formats.md) untuk penjelasan selengkapnya.***

## Metadata yang Dibawa ke Ekspor

Setiap produk mempertahankan **blok GPS**dan**sub-IFD EXIF** dari tangkapan sumber, sehingga
ekspor tersebut membawa `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal`, dan
`CameraSerialNumber`, serta data georeferensi.

{% hint style="warning" %}
**Jika ortomosaik dihasilkan dengan skala yang tidak masuk akal, periksa `FocalLength` terlebih dahulu.**
Pix4D menghitung jarak sampel tanah (GSD) berdasarkan panjang fokus ditambah ketinggian. Tanpa tag tersebut,
program akan menggunakan skala yang sangat salah — pada satu penerbangan yang diukur dengan 49 pengambilan gambar, kebun jeruk seluas 411 m × 160 m
direkonstruksi menjadi 47,8 km × 13 km, menghasilkan ortofoto 455 megapiksel yang sebagian besar berisi
ruang kosong. Proses pembagian ubin yang lambat dan ukuran file yang sangat besar secara tak terduga merupakan gejala dari masalah ini, bukan
masalah terpisah.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Tidak *semua* tag disalin. Tag struktural IFD0 sengaja tidak disertakan (menyalin
tag tersebut akan merusak keluaran LATTICE), dan `ExifImageWidth` / `ExifImageHeight` dikecualikan
karena tag tersebut mendeskripsikan tangkapan asli — ekspor yang diubah ukurannya akan
mengklaim dimensi yang bertentangan dengan rasternya sendiri.

***

## Meninjau Log Debug

### Periksa Peringatan atau Kesalahan

1. Buka tab **Log Debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Gulir pesan-pesan tersebut
3. Cari peringatan berwarna kuning atau kesalahan berwarna merah
4. Baca baris-baris `[RUN-SUMMARY]` dan petunjuk apa pun
5. Hubungi dukungan MAPIR untuk bantuan

### Menyimpan Log

Untuk menyimpan catatan pemrosesan atau mengirimkannya ke Dukungan MAPIR:

1. Klik tombol **&quot;Salin&quot;**atau**&quot;Unduh&quot;**

2. Simpan sebagai berkas teks di folder proyek
3. Sertakan dalam dokumentasi proyek
4. Kirim ke tim dukungan MAPIR jika menemui masalah

***

## Masalah Keluaran Umum dan Solusinya

### Masalah: Berkas Keluaran Hilang

**Penyebab yang mungkin:**

* Produk tersebut tidak berlaku untuk kamera tersebut (misalnya, radiance/reflectance untuk kamera RGB — log menunjukkan hal tersebut)
* Referensi yang diperlukan tidak ada (misalnya, reflektansi tanpa target dan tanpa sinar turun dari `.daq`)
* Kotak centang ekspor produk dinonaktifkan di Pengaturan Proyek
* Ruang disk habis selama proses ekspor

**Solusi:**

1. Periksa petunjuk `[RUN-SUMMARY]` dan baris `[EXPORT-CHECK]` di Log Debug — keduanya menjelaskan alasan terlewatnya data per kamera
2. Verifikasi kotak centang produk ekspor di [Pengaturan Proyek](adjusting-project-settings.md)
3. Pastikan ruang disk mencukupi
4. Lakukan pemrosesan ulang setelah memperbaiki penyebabnya

### Masalah: Tepi Gelap atau Terang (Vignetting Masih Terlihat)

**Kemungkinan penyebab:**

* Koreksi vignetting dinonaktifkan
* Kamera/lensa tidak ada dalam basis data profil Chloros
* Vignetting ekstrem yang melampaui kemampuan koreksi

**Solusi:**

1. Pastikan koreksi vignetting diaktifkan di Pengaturan Proyek
2. Pastikan model kamera terdeteksi dengan benar
3. Hubungi dukungan MAPIR jika vignetting masih terjadi

### Masalah: Warna atau Nilai yang Salah

**Penyebab yang mungkin:**

* Tidak terdeteksi target kalibrasi
* Model target kalibrasi yang dipilih salah
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
* **Vegetasi sehat dan lebat**: 0,6 hingga 0,9**Jika nilai berada di luar rentang ini:**

1. Pastikan kalibrasi reflektansi telah diterapkan
2. Pastikan log sensor cahaya telah disertakan
3. Pastikan target kalibrasi terdeteksi
4. Pastikan model kamera yang benar terdeteksi
5. Periksa waktu dan kondisi pengambilan gambar target
6. Jika menghitung indeks sendiri dari file reflektansi, pastikan Anda membaginya dengan nilai `Chloros:PixelScale` pada file tersebut (lihat di atas)

***

## Menggunakan Gambar yang Telah Diproses

### Untuk Fotogrametri / Pembuatan Ortomosaik

**Alur kerja yang direkomendasikan:**

1.**Impor gambar reflektansi yang telah dikalibrasi** ke perangkat lunak fotogrametri:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Pertahankan metadata EXIF**: Pastikan data GPS tetap tersimpan untuk penandaan geografis
3. **Alur kerja yang dikalibrasi**: Gunakan gambar reflektansi untuk akurasi ilmiah — reflektansi LATTICE membawa tag kalibrasi XMP yang dibaca oleh Pix4D
4. **Proses mosaik indeks**: Buat ortomosaik NDVI dari gambar indeks individual
5. **Ekspor GeoTIFF yang telah direferensikan secara geografis**: Untuk digunakan dalam aplikasi GIS

### Untuk Analisis GIS

**Alur kerja yang direkomendasikan:**

1.**Muat ke dalam QGIS, ArcGIS, atau aplikasi serupa**

2.**Gunakan citra reflektansi 16-bit TIFF** untuk analisis multi-band (bagilah dengan nilai `Chloros:PixelScale` pada file tersebut)
3. **Gunakan citra indeks** (NDVI, NDRE) sebagai lapisan vegetasi yang siap pakai
4. **Kalkulator raster**: Gabungkan pita untuk analisis khusus
5. **Ekspor**: Buat peta klasifikasi, deteksi perubahan, peta kesehatan vegetasi

### Untuk Analisis Langsung / Pelaporan

**Alur kerja yang direkomendasikan:**

1.**Gunakan citra indeks dengan warna LUT** untuk laporan visual
2. **Ekstrak statistik**: Rata-rata NDVI per bidang/plot
3. **Rangkaian waktu**: Bandingkan indeks di berbagai sesi
4. **Buat laporan**: Sertakan peta, statistik, dan visualisasi***

## Pengarsipan dan Pencadangan

### Strategi Pencadangan yang Direkomendasikan

**Yang perlu disimpan:*** ✅ **Gambar RAW/JPG asli atau tangkapan mentah LATTICE** - Arsipkan di drive/cloud terpisah; data mentah adalah sumber pipeline dan semua yang lain dapat dihasilkan kembali darinya
* ✅ **File sensor cahaya `.daq` / `.csv`** - Diperlukan untuk menghitung kembali reflektansi di kemudian hari
* ✅ **Hasil pemrosesan** - Simpan gambar yang telah dikalibrasi dan indeksnya
* ✅ **Folder proyek** (`project.json` dan file terkait) - Berisi semua pengaturan untuk pemrosesan ulang jika diperlukan
* ✅ **Log Debug** - Mendokumentasikan detail pemrosesan
* ✅ **Gambar target kalibrasi** - Untuk verifikasi dan pemrosesan ulang**Rekomendasi penyimpanan:*** **Pencadangan segera**: Hard drive eksternal
* **Arsip jangka panjang**: Penyimpanan cloud (Google Drive, Dropbox, dll.)
* **Data penting**: Simpan 2–3 salinan di lokasi yang berbeda***

## Proses Pengolahan Berikutnya

### Menggunakan Kembali Pengaturan Proyek

Jika akan memproses kumpulan data serupa di masa mendatang:

1. **Simpan Template Proyek** (jika belum dilakukan)
2. **Buat proyek baru** menggunakan template yang disimpan
3. **Impor gambar baru**

4.**Proses**dengan pengaturan yang sama untuk konsistensi

### Pemrosesan Batch Beberapa Sesi

Untuk beberapa sesi/kumpulan data:**Opsi 1: GUI - Beberapa Proyek**

* Buat proyek terpisah untuk setiap sesi
* Gunakan pengaturan templat yang konsisten
* Olah satu per satu

**Opsi 2: Chloros CLI (hanya untuk versi Chloros ke atas)**

* Otomatiskan pemrosesan batch
* Proses beberapa folder dengan skrip
* Lihat [Dokumentasi CLI](../CLI.md) dan [Referensi CLI](../reference/cli-reference.md)

**Opsi 3: Python SDK (hanya Chloros+ )**

* Kontrol terprogram
* Integrasi dengan alur kerja analisis
* Lihat [Dokumentasi API](../api-python-sdk.md) dan [Referensi SDK](../reference/sdk-reference.md)

***

## Pemecahan Masalah Pasca-Pemrosesan

### Memproses Ulang dengan Pengaturan yang Berbeda

Jika hasilnya tidak memuaskan:

1. Simpan gambar asli (jangan pernah dihapus)
2. Buka proyek yang sama di Chloros
3. Sesuaikan pengaturan di panel Pengaturan Proyek
4. Lakukan pemrosesan ulang — hasil akan disimpan di folder produk yang sama, sehingga file dengan nama yang sama dari pemrosesan sebelumnya akan diganti

### Memproses Sebagian Gambar

Untuk memproses ulang hanya gambar tertentu:

1. Buat proyek baru
2. Impor hanya gambar yang perlu diproses ulang
3. Gunakan templat pengaturan yang sama
4. Proses kumpulan data yang lebih kecil

### Mendapatkan Bantuan

Jika Anda menemui masalah:

* 📧 **Email**: info@mapir.camera (sertakan Log Debug)
* 🌐 **Dukungan**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **FAQ**: [Pertanyaan yang Sering Diajukan](../faq.md)
* 📖 **Dokumentasi**: [Panduan Chloros](../)***

## Ringkasan: Alur Kerja Lengkap

Anda kini telah menyelesaikan seluruh alur kerja pemrosesan Chloros:

1. ✅ **Membuat proyek** - Lihat [Proyek](../projects.md)
2. ✅ **Menambahkan berkas** - Lihat [Menambahkan Berkas](adding-files-to-a-project.md)
3. ✅ **Menyesuaikan pengaturan** - Lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md)
4. ✅ **Menandai target** - Lihat [Memilih Gambar Target](choosing-target-images.md)
5. ✅ **Memulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
6. ✅ **Memantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)
7. ✅ **Hasil telah ditinjau** - Halaman ini**Gambar multispektral Anda yang telah dikalibrasi dan dikoreksi reflektansinya siap untuk dianalisis!**

***

## Sumber Daya Tambahan

### Fitur Lanjutan

* [**Penampil Gambar**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualisasi dan analisis interaktif
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Pengujian indeks kustom
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) - Referensi indeks lengkap

### Otomatisasi &amp; Integrasi

* [**Dokumentasi CLI**](../CLI.md) - Pemrosesan batch melalui baris perintah
* [**Python SDK**](../api-python-sdk.md) - Otomatisasi berbasis program
* [**Fitur Chloros+**](../#chloros) - Kemampuan pemrosesan tingkat lanjut

### Dukungan &amp; Pembelajaran

* [**FAQ**](../faq.md) - Jawaban atas pertanyaan umum
* [**Target Kalibrasi**](../calibration-targets.md) - Memahami kalibrasi reflektansi
* [**Kamera yang Didukung**](../supported-cameras.md) - Perangkat keras yang kompatibel

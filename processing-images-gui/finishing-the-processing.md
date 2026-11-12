# Menyelesaikan Pemrosesan

Setelah Chloros menyelesaikan pemrosesan, kini saatnya meninjau hasil Anda, memverifikasi kualitas keluaran, dan mempersiapkan gambar yang telah diproses untuk digunakan dalam alur kerja Anda. Halaman ini akan memandu Anda melalui langkah-langkah akhir dan tindakan selanjutnya.

## Indikasi Pemrosesan Selesai

Saat pemrosesan berhasil diselesaikan, Anda akan melihat beberapa indikator:

* ✅ **Bilah kemajuan**: Mencapai 100% penyelesaian
* ✅ **Log Debug**: Menampilkan baris-baris terakhir `[RUN-SUMMARY]` beserta jumlahnya (gambar, grup kamera, target, gambar yang dikalibrasi, file yang ditulis)
* ✅ **Tombol Mulai**: Kembali aktif (siap untuk proses berikutnya)
* ✅ **File keluaran**: Semua gambar yang telah diproses disimpan ke dalam pohon keluaran proyek (di bawah)

{% hint style="warning" %}
**Proses yang tidak menulis gambar apa pun dianggap gagal.** Jika Anda meminta produk gambar dan proses tersebut tidak menulis satupun, Chloros melaporkannya sebagai kegagalan — `[RUN-SUMMARY]` memberikan petunjuk dalam nama log mengenai penyebab yang mungkin (tidak ada yang diimpor, tidak ada target yang terdeteksi, atau setiap produk yang diminta dilewati karena tidak berlaku). Ekivalen CLI-nya keluar dengan nilai non-nol. Proses yang sengaja hanya menghasilkan metadata (semua produk ekspor dinonaktifkan, tanpa indeks) tetap dianggap berhasil. Lihat [Referensi CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Menemukan Gambar yang Telah Diproses

### Membuka Folder Keluaran

1. Klik ikon **Menu Utama** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(kiri atas)
2. Pilih **"Buka Folder Proyek"**

3. Penjelajah file Anda akan terbuka ke direktori proyek
4. Temukan proyek Anda berdasarkan namanya

### Pohon Output

Hasil diarsipkan **di bawah folder proyek, dikelompokkan berdasarkan kamera dan kemudian berdasarkan format file**:

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
* **Format folder**: mengikuti pengaturan format ekspor Anda — `tiff16`, `tiff8`, `png8`, `jpg8`, atau `tiff32` untuk TIFF (32-bit, Persen). Radiance selalu berjenis float32 dan selalu disimpan di bawah `tiff32`.
* **Folder produk**:
  * `Reflectance_Calibrated_Images/` — reflektansi yang dikalibrasi
  * `Debayered_Images/` — hasil debayering linier (LATTICE): offset gelap dihilangkan dan demosaicing dilakukan, tanpa flat field atau unmix, dalam orientasi asli sensor
  * `Preview_Images/` — pratinjau tampilan (LATTICE)
  * `Radiance_Images/` — radian spektral float32, W/m²/sr/nm (multispektral LATTICE)
  * `Vignette_Corrected_Images/` **atau** `Sensor_Response_Images/` — opsi cadangan yang belum dikalibrasi untuk bingkai tanpa referensi reflektansi; hanya salah satu dari keduanya yang ada per sesi, dipilih berdasarkan pengaturan koreksi Vignette
  * `<INDEX>_Index_Images/` — satu folder per indeks yang dipilih (misalnya `NDVI_Index_Images`)
* **`clamp_masks/`** (baru di versi 1.3.0, khusus untuk kamera multispektral LATTICE M3C): muncul di dalam folder format, di samping folder produk, ketika proses pemisahan pita memotong nilai negatif menjadi nol dalam produk radian atau reflektansi. Folder ini berisi satu masker per piksel untuk setiap produk yang terpengaruh, misalnya `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. Masker tidak pernah berada di dalam folder produk, sehingga folder produk dapat diserahkan ke perangkat lunak fotogrametri apa adanya. Lihat [Masker Clamp](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Setiap produk yang diekspor tetap menggunakan nama berkas SUMBER.**Ekspor radian dari `capture_..._raw.tif` tetap bernama `capture_..._raw.tif` — hanya saja berkas tersebut berada di dalam `tiff32/Radiance_Images/`.**Folderlah yang mengidentifikasi produk, bukan nama berkas**, sehingga pencarian terhadap `*radiance*.tif` tidak akan menemukan apa pun; gunakan direktori sebagai acuan pencarian.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Berapa Banyak Berkas yang Harus Ada?

Jangan menghitung berdasarkan rumus — jumlah keluaran bergantung pada produk mana yang diaktifkan dan mana yang berlaku untuk setiap kamera (misalnya, kamera RGB tidak mendapatkan radiance/reflectance). Jumlah yang pasti tercantum dalam log: baris terakhir `[RUN-SUMMARY]` melaporkan dengan tepat berapa banyak file yang ditulis, dan baris petunjuk menjelaskan apa saja yang dilewati.

***

## Meninjau Gambar yang Telah Diproses

### Pratinjau Cepat di File Explorer

**Pratinjau bawaan Windows:**

1. Buka folder produk (misalnya `tiff16/Reflectance_Calibrated_Images/`)
2. Pilih file gambar
3. Pratinjau akan muncul di panel pratinjau Windows Explorer
4. Gunakan tombol panah untuk menelusuri gambar

### Pratinjau di Penampil Gambar Eksternal

**Penampil yang direkomendasikan:**

* **QGIS** - Perangkat lunak GIS gratis (paling cocok untuk analisis multispektral yang direferensikan secara geografis)
* **IrfanView** - Penampil gambar yang cepat dan ringan (mendukung TIFF)
* **Adobe Photoshop** - Pengeditan profesional (mendukung TIFF)
* **GIMP** - Alternatif gratis untuk Photoshop
* **Windows Photos** - Penayangan dasar (mungkin tidak mendukung TIFF 16-bit)

### Pratinjau di Penampil Gambar Chloros


Gunakan Penampil Gambar bawaan Chloros untuk visualisasi lanjutan:

1. Klik thumbnail gambar di Penjelajah File
2. Gambar terbuka di area pratinjau utama
3. Klik tab **Penampil Gambar** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
di bilah sisi kiri
4. Gunakan [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) untuk analisis interaktif

Lihat [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) untuk petunjuk terperinci.

***

## Membaca Nilai Piksel Reflektansi (GIS / Pix4D / Skrip)

Reflektansi disimpan sebagai bilangan bulat DN, dan **nilai DN yang berarti ρ = 1,0 bergantung pada kamera sumber**:

| Sumber          | ρ = 1,0 adalah | Cara mengetahuinya                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (ruang kosong hingga ρ 2,0) | Tag XMP `Chloros:PixelScale=32768` tercantum pada berkas |
| Survey3          | **65535** (dipotong pada ρ 1,0)     | Tidak ada tag `Chloros:PixelScale` — ketiadaan tersebut adalah petunjuknya (produk Survey3 memang memiliki tag pemrosesan `Chloros:*` lainnya) |

**Baca tag `Chloros:PixelScale` dan bagi dengan nilainya** daripada mengasumsikan nilai 65535 secara seragam — membagi reflektansi LATTICE dengan 65535 secara diam-diam akan membagi dua setiap nilai. File reflektansi dari versi sebelum 1.3.0 yang ditulis sebagai 8-bit telah dipotong (clipped), bukan diskalakan ulang, dan tidak memiliki tag skala: ekspor ulang file tersebut dalam format 16-bit atau 32-bit alih-alih membaginya. Lihat [Format Gambar Keluaran](../output-image-formats.md) untuk penjelasan selengkapnya.

***

## Metadata yang Dibawa ke dalam Ekspor

Setiap produk mempertahankan **blok GPS**dan**sub-IFD EXIF** dari pengambilan sumber, sehingga
ekspor membawa `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal`, dan
`CameraSerialNumber` serta georeferensi.

{% hint style="warning" %}
**Jika ortomosaik dihasilkan dengan skala yang tidak masuk akal, periksa `FocalLength` terlebih dahulu.**
Pix4D menghitung jarak sampel tanah (GSD) berdasarkan panjang fokus ditambah ketinggian. Tanpa tag tersebut,
program akan kembali ke skala yang sangat salah — pada satu penerbangan yang diukur dengan 49 pengambilan gambar, sebuah kebun jeruk seluas 411 m × 160 m
kebun jeruk direkonstruksi menjadi 47,8 km × 13 km, menghasilkan ortofoto 455 megapiksel yang sebagian besar berisi
ruang kosong. Proses tiling yang lambat dan ukuran file yang sangat besar secara tak terduga merupakan gejala dari hal ini, bukan
masalah terpisah.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Tidak *semua* tag disalin. Tag struktural IFD0 sengaja tidak disertakan (menyalin
tag tersebut akan merusak keluaran LATTICE), dan `ExifImageWidth` / `ExifImageHeight` dikecualikan
karena tag-tag tersebut mendeskripsikan tangkapan asli — ekspor yang ukurannya diubah akan
mengklaim dimensi yang bertentangan dengan rasternya sendiri.

Chloros juga menulis tag XMP `Chloros:*`-nya sendiri yang menjelaskan apa yang telah dilakukan pada setiap produk
(misalnya `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Mulai versi 1.3.0, produk-produk Survey3 menyertakannya di setiap mode pemrosesan, produk indeks mendeskripsikan indeks itu sendiri (`Chloros:ProcessingLevel=index`,
rumusnya, dan apakah dihitung dari reflektansi atau radian) alih-alih mengulang
tag data mentah sumber, dan produk dalam folder yang kedalaman namanya melebihi 260 karakter tetap mempertahankan metadata
mereka. Lihat [Format Gambar Keluaran](../output-image-formats.md#index-products).

***

## Meninjau Log Debug

### Memeriksa Peringatan atau Kesalahan

1. Buka tab **Log Debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Gulir melalui pesan-pesan
3. Cari peringatan berwarna kuning atau kesalahan berwarna merah
4. Baca baris-baris `[RUN-SUMMARY]` dan petunjuk apa pun
5. Hubungi dukungan MAPIR untuk bantuan

### Menyimpan Log

Untuk menyimpan catatan pemrosesan atau mengirimkannya ke Dukungan MAPIR:

1. Klik tombol **"Salin"**atau**"Unduh"**

2. Simpan sebagai berkas teks di folder proyek
3. Sertakan dalam dokumentasi proyek
4. Kirim ke dukungan MAPIR jika menemui masalah

Untuk melaporkan masalah langsung ke MAPIR, gunakan ikon bug (**Laporkan bug**) di bagian atas tab Log Debug — lihat [Melaporkan bug dari tab log](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Masalah Keluaran Umum dan Solusinya

### Masalah: Berkas Keluaran Hilang

**Penyebab yang mungkin:**

* Produk tersebut tidak berlaku untuk kamera tersebut (misalnya, radiance/reflectance untuk kamera RGB — log menunjukkan hal tersebut)
* Referensi yang diperlukan tidak ada (misalnya, reflektansi tanpa target dan tanpa `.daq` downwelling)
* Kotak centang ekspor produk dinonaktifkan di Pengaturan Proyek
* Ruang disk habis selama proses ekspor

**Solusi:**

1. Periksa petunjuk `[RUN-SUMMARY]` dan baris `[EXPORT-CHECK]` di Log Debug — keduanya menjelaskan alasan pengabaian per kamera
2. Pastikan kotak centang ekspor produk di [Pengaturan Proyek](adjusting-project-settings.md) telah dicentang
3. Pastikan ruang disk mencukupi
4. Lakukan pemrosesan ulang setelah memperbaiki penyebabnya

### Masalah: Tepi Gelap atau Terang (Vignetting Masih Terlihat)

**Kemungkinan penyebab:**

* Koreksi vignetting dinonaktifkan
* Kamera/lensa tidak ada dalam basis data profil Chloros

* Vignetting ekstrem yang melampaui kemampuan koreksi

**Solusi:**

1. Pastikan koreksi vignetting diaktifkan di Pengaturan Proyek
2. Periksa apakah model kamera terdeteksi dengan benar
3. Hubungi dukungan MAPIR jika vignetting masih terjadi

### Masalah: Warna atau Nilai yang Salah

**Kemungkinan penyebab:**

* Tidak ada target kalibrasi yang terdeteksi
* Model target kalibrasi yang dipilih salah
* Kalibrasi reflektansi dinonaktifkan
* Kualitas gambar target buruk

**Solusi:**

1. Pastikan kalibrasi reflektansi telah diaktifkan
2. Periksa pesan "Target ditemukan" di Log Debug
3. Periksa kualitas gambar target
4. Proses ulang dengan menandai target yang tepat

### Masalah: Nilai NDVI Tampak Salah

**Rentang NDVI yang diharapkan:**

* **Air, batu, tanah**: -0,1 hingga 0,2
* **Vegetasi jarang/tidak sehat**: 0,2 hingga 0,4
* **Vegetasi sedang**: 0,4 hingga 0,6
* **Vegetasi sehat dan lebat**: 0,6 hingga 0,9**Jika nilai berada di luar rentang ini:**

1. Pastikan kalibrasi reflektansi telah diterapkan
2. Pastikan log sensor cahaya telah disertakan
3. Periksa apakah target kalibrasi terdeteksi
4. Pastikan model kamera yang benar telah terdeteksi
5. Tinjau waktu dan kondisi pengambilan gambar target
6. Jika Anda menghitung indeks sendiri dari berkas reflektansi, pastikan Anda membaginya dengan nilai `Chloros:PixelScale` pada berkas tersebut (lihat di atas)

***

## Menggunakan Gambar yang Telah Diproses

### Untuk Fotogrametri / Pembuatan Ortomosaik

**Alur kerja yang direkomendasikan:**

1.**Impor gambar reflektansi yang telah dikalibrasi** ke perangkat lunak fotogrametri:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Pertahankan metadata EXIF**: Pastikan data GPS tetap terjaga untuk penandaan geolokasi
3. **Alur kerja yang dikalibrasi**: Gunakan gambar reflektansi untuk akurasi ilmiah — gambar reflektansi LATTICE dilengkapi dengan tag kalibrasi XMP yang dapat dibaca oleh Pix4D
4. **Proses mosaik indeks**: Buat ortomosaik NDVI dari gambar indeks individual
5. **Ekspor GeoTIFF yang telah direferensikan secara geografis**: Untuk digunakan dalam aplikasi GIS

### Untuk Analisis GIS

**Alur kerja yang direkomendasikan:**

1.**Muat ke QGIS, ArcGIS, atau aplikasi serupa**

2.**Gunakan gambar reflektansi TIFF 16-bit** untuk analisis multi-band (bagilah dengan nilai `Chloros:PixelScale` pada file)
3. **Gunakan gambar indeks** (NDVI, NDRE) sebagai lapisan vegetasi siap pakai
4. **Kalkulator raster**: Gabungkan pita untuk analisis khusus
5. **Ekspor**: Buat peta klasifikasi, deteksi perubahan, dan peta kesehatan vegetasi

### Untuk Analisis Langsung / Pelaporan

**Alur kerja yang direkomendasikan:**

1.**Gunakan gambar indeks dengan warna LUT** untuk laporan visual
2. **Ekstrak statistik**: Nilai rata-rata NDVI per bidang/plot
3. **Seri waktu**: Bandingkan indeks antar sesi
4. **Buat laporan**: Sertakan peta, statistik, dan visualisasi***

## Pengarsipan dan Pencadangan

### Strategi Pencadangan yang Direkomendasikan

**Yang perlu disimpan:**

* ✅ **Gambar RAW/JPG asli atau tangkapan mentah LATTICE** - Arsipkan di drive terpisah/cloud; data mentah adalah sumber alur kerja dan semua yang lain dapat dihasilkan kembali darinya
* ✅ **File sensor cahaya `.daq` / `.csv`** - Diperlukan untuk menghitung ulang reflektansi di kemudian hari
* ✅ **Hasil pemrosesan** - Simpan gambar yang telah dikalibrasi dan indeks
* ✅ **Folder proyek** (`project.json` dan berkas terkait) - Berisi semua pengaturan untuk pemrosesan ulang jika diperlukan
* ✅ **Log Debug** - Mendokumentasikan detail pemrosesan
* ✅ **Gambar target kalibrasi** - Untuk verifikasi dan pemrosesan ulang

**Rekomendasi penyimpanan:**

* **Pencadangan segera**: Hard disk eksternal
* **Arsip jangka panjang**: Penyimpanan awan (Google Drive, Dropbox, dll.)
* **Data penting**: Simpan 2–3 salinan di lokasi yang berbeda***

## Proses Pengolahan Berikutnya

### Menggunakan Kembali Pengaturan Proyek

Jika akan memproses kumpulan data serupa di masa mendatang:

1. **Simpan Template Proyek** (jika belum dilakukan)
2. **Buat proyek baru** menggunakan template yang telah disimpan
3. **Impor gambar baru**

4.**Proses**dengan pengaturan yang sama untuk konsistensi

### Pemrosesan Batch untuk Beberapa Sesi

Untuk beberapa sesi/kumpulan data:**Opsi 1: GUI - Beberapa Proyek**

* Buat proyek terpisah untuk setiap sesi
* Gunakan pengaturan templat yang konsisten
* Proses satu per satu

**Opsi 2: Chloros CLI (hanya Chloros+)**

* Otomatiskan pemrosesan massal
* Proses beberapa folder dengan skrip
* Lihat [Dokumentasi CLI](../CLI.md) dan [Referensi CLI](../reference/cli-reference.md)

**Opsi 3: Python SDK (hanya Chloros+)**

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
4. Proses ulang — hasil akan disimpan di folder produk yang sama, sehingga file dengan nama yang sama dari proses sebelumnya akan diganti

### Memproses Sebagian Gambar

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
* 📖 **Dokumentasi**: [Panduan Chloros](../)

***

## Ringkasan: Alur Kerja Lengkap

Anda kini telah menyelesaikan seluruh alur kerja pemrosesan Chloros:

1. ✅ **Proyek telah dibuat** - Lihat [Proyek](../projects.md)
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

### Otomatisasi & Integrasi

* [**Dokumentasi CLI**](../CLI.md) - Pemrosesan batch melalui baris perintah
* [**Python SDK**](../api-python-sdk.md) - Otomatisasi berbasis pemrograman
* [**Fitur Chloros+**](../#chloros) - Kemampuan pemrosesan tingkat lanjut

### Dukungan & Pembelajaran

* [**FAQ**](../faq.md) - Jawaban atas pertanyaan umum
* [**Target Kalibrasi**](../calibration-targets.md) - Memahami kalibrasi reflektansi
* [**Kamera yang Didukung**](../supported-cameras.md) - Perangkat keras yang kompatibel

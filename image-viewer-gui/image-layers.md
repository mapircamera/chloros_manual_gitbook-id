# Lapisan Gambar

**Menu tarik-turun lapisan** di pojok kanan atas Penampil Gambar memungkinkan Anda beralih di antara setiap versi gambar yang sedang Anda lihat — mulai dari tangkapan sumber, melalui setiap produk yang telah diproses, hingga gambar indeks yang dihitung — tanpa perlu keluar dari penampil.

## Apa itu Lapisan Gambar?

"Lapisan" di Chloros adalah satu **file produk**yang didaftarkan berdasarkan satu gambar sumber. Proses impor memberikan file-file sumber; pemrosesan menambahkan satu lapisan untuk setiap produk yang dihasilkan oleh proses tersebut. File yang diekspor tetap menggunakan nama file sumber —**folder**-lah yang mengidentifikasi produk, dan nama lapisan adalah label yang diberikan Chloros untuk folder tersebut.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Daftar lapisan

### Selalu ada

| Lapisan | Apa itu |
| --- | --- |
| **JPG**(atau**PNG**/**TIFF**) | File dasar yang masuk bersama hasil pengambilan. Survey3 mengimpor file `.JPG` di samping setiap file `.RAW`; hasil pengambilan LATTICE menyertakan pratinjau tampilan dalam format PNG atau TIFF. Dilabeli sesuai dengan apa yang sebenarnya diimpor |
| **RAW (Asli)** | Frame mentah sumber, telah melalui proses debayering untuk tampilan tanpa koreksi apa pun. Tersedia sejak saat impor — tidak memerlukan pemrosesan |

Sebuah tangkapan LATTICE yang berkas dasarnya **adalah** bingkai mentahnya tidak memiliki entri dasar terpisah: `RAW (Original)` sudah mencakupnya.

### Produk pemrosesan Survey3



| Lapisan | Ditulis ke | Ada saat |
| --- | --- | --- |
| **RAW (Target)** | — | Frame ini teridentifikasi mengandung target kalibrasi |
| **RAW (Reflektansi)** | `Reflectance_Calibrated_Images/` | Kalibrasi reflektansi berhasil dijalankan pada frame ini |
| **Vignette Corrected**| `Vignette_Corrected_Images/` | Frame tersebut tidak dapat dikalibrasi reflektansinya**dan** *koreksi vignette* diaktifkan |
| **Respon Sensor**| `Sensor_Response_Images/` | Bingkai ini tidak dapat dikalibrasi reflektansinya**dan** *koreksi vignette* dinonaktifkan |
| **Keseimbangan Putih** | `White_Balanced_Images/` | Produk dengan keseimbangan putih telah disimpan |

{% hint style="info" %}
**Koreksi Vignette dan Respons Sensor adalah alternatif, tidak pernah keduanya sekaligus.** Terdapat tepat satu produk cadangan yang tidak dikalibrasi per proses, untuk setiap model kamera, dan tombol *Koreksi Vignette* menentukan produk mana yang dipilih. Lihat [Pengaturan Proyek](../project-settings/project-settings.md).
{% endhint %}

### Tingkat LATTICE

LATTICE menangkap fan-out ke tingkat-tingkat ini dalam satu tahap pemrosesan. Tingkat mana saja yang tersedia bergantung pada tombol ekspor per-produk di Pengaturan Proyek dan pada apa yang berlaku untuk kamera tersebut.

| Lapisan | Ditulis ke | Berlaku untuk |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB, dan multispektral |
| **RAW (Pratinjau)** | `Preview_Images/` | Multispektral (perluasan warna palsu) |
| **White Balanced** | `Preview_Images/` | Kamera utama RGB — pratinjau RGB terdaftar di bawah nama ini agar selaras dengan lapisan Survey3 dengan nama yang sama |
| **RAW (Radiance)** | `Radiance_Images/` | Hanya multispektral |
| **RAW (Reflektansi)** | `Reflectance_Calibrated_Images/` | Hanya multispektral, dan hanya jika ada catatan downwelling `.daq` yang cocok atau target dalam bingkai yang lulus QA yang mencakup bingkai tersebut |

Kamera utama RGB tidak memiliki radiometri per-band, sehingga radiance dan reflectance dilewati untuk kamera tersebut sebagai **tidak berlaku** — log akan mencatat hal ini daripada gagal tanpa pemberitahuan.

{% hint style="info" %}
**Lapisan mentah dan yang telah di-debayering selaras dengan produk-produk (fitur baru di versi 1.3.0).**Berkas mentah dan yang telah di-debayering LATTICE disimpan dalam orientasi asli sensor, dan mencatat orientasi yang sesuai dalam file XMP; produk-produk lainnya disimpan dengan orientasi tersebut sudah diterapkan. Penampil dan kisi gambar menyesuaikan tampilan**RAW (Asli)**dan**RAW (Debayered)** berdasarkan catatan tersebut, sehingga bingkai tidak lagi terbalik saat Anda beralih antara lapisan produk dan lapisan mentah. Ini hanya untuk tampilan — berkas-berkas tersebut tidak pernah ditimpa. File tanpa catatan orientasi (yang diambil sebelum versi 1.3.0, atau ditulis oleh perangkat lunak lain) ditampilkan sesuai dengan cara penyimpanannya. Lihat [Format Gambar Keluaran](../output-image-formats.md#orientation).
{% endhint %}

Masker clamp (`*_clampmask.tif` dalam folder `clamp_masks/`) bukanlah lapisan: Chloros mengabaikannya saat impor. Lihat [Format Gambar Keluaran](../output-image-formats.md#clamp-masks).

### Lapisan indeks, LUT, dan sandbox

| Pola lapisan | Contoh | Asal |
| --- | --- | --- |
| **RAW (Indeks `<INDEX>`)** | `RAW (NDVI Index)` | Satu per indeks yang dikonfigurasi di Pengaturan Proyek, dihitung selama pemrosesan |
| **`<INDEX>` LUT** | `NDVI LUT` | Versi indeks yang telah dipetakan warnanya |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Satu per proses ekspor [Indeks/LUT Sandbox](index-lut-sandbox.md) proses ekspor |

Jika nama indeks yang sama dikonfigurasi lebih dari sekali dengan pengaturan yang berbeda, yang kedua dan seterusnya akan diberi nomor pada namanya (`RAW (NDVI2 Index)`) agar lapisan-lapisan tersebut tetap dapat dibedakan.

***

## Menggunakan pemilih lapisan

1. Buka gambar dalam layar penuh dengan mengklik thumbnail di kisi
2. Klik **menu tarik-turun lapisan** di pojok kanan atas penampil
3. Pilih lapisan — gambar akan diperbarui seketika

Menu dropdown menampilkan **JPG, RAW (Asli), RAW (Target), RAW (Reflektansi)** terlebih dahulu, dalam urutan tersebut, dan mencantumkan semua yang lain setelahnya sesuai urutan pendaftaran produk.

### Preferensi lapisan saat menavigasi

Menekan tombol **←**/**→** akan berpindah ke gambar berikutnya dan berusaha mempertahankan lapisan yang sama:

1. **Kecocokan persis didahulukan** — jika gambar berikutnya memiliki lapisan dengan nama yang sama, lapisan tersebut yang akan ditampilkan. Inilah yang membuat Anda tetap berada di lapisan `RAW (NDVI Index)` saat menelusuri seluruh set
2. **Kemudian pencocokan berdasarkan jenis** — lapisan indeks mencari lapisan indeks apa pun, LUT mencari LUT apa pun, reflektansi mencari reflektansi, target mencari target, asli mencari asli, dasar mencari dasar
3. **Kemudian, khusus untuk lapisan ekspor** — nama tetap dipertahankan meskipun daftar lapisan belum diperbarui, karena berkas tersebut sudah ada di disk. Inilah yang memungkinkan Anda meninjau hasil sementara proses masih sedang menyimpannya
4. **Jika tidak** — lapisan pertama yang tersedia, yang biasanya adalah gambar dasar

File sidecar `.daq` dan `.csv` dalam proyek dilewati saat navigasi menggunakan tombol panah, sehingga saat menelusuri gambar, Anda tidak akan pernah sampai pada rekaman sensor cahaya.

Fitur zoom dan pan juga berlaku di seluruh gambar, sehingga perbandingan sebelum/sesudah pada posisi bidang yang sama menjadi mudah.

***

## Memahami nilai piksel per lapisan

[Panel Nilai Kursor](opening-an-image-full-screen.md#cursor-values) menampilkan nilai aktual per kanal di bawah kursor Anda, dalam satuan yang digunakan untuk menyimpan lapisan tersebut. Kolom-kolomnya berubah sesuai dengan lapisan:

| Lapisan | Satuan yang ditampilkan | Catatan |
| --- | --- | --- |
| Dasar (Pratinjau JPG / PNG / TIFF) | DN, 0–255 | Nilai tampilan, yang telah dikoreksi gamma pada RGB. Hanya untuk pemeriksaan visual |
| RAW (Asli) | DN | Angka digital mentah dari sensor. Sumbu histogram menunjukkan kedalaman: 255 (8-bit), 4095 (12-bit), atau 65535 (16-bit) |
| RAW (Debayered) | DN | Linear, tanpa peregangan tampilan |
| RAW (Pratinjau) / Keseimbangan Putih | DN | Hasil tampilan — diregangkan atau dikoreksi gamma. Bukan untuk pengukuran |
| RAW (Radiance) | **W/m²/sr/nm** | Radian fisik Float32. Tidak ada kolom DN |
| RAW (Reflectance) | DN **dan %** | Persentase dihitung berdasarkan skala file tersebut — lihat di bawah |
| Ekspor indeks / LUT / sandbox | Nilai indeks, atau komponen RGB | File indeks saluran tunggal melaporkan nilai indeks; file LUT yang dipetakan warna melaporkan komponen Red /Green /Blue |

### Reflektansi: skalanya per file

{% hint style="warning" %}
**"Bagi dengan 65.535" hanya benar untuk Survey3.** Reflektansi LATTICE disimpan pada skala yang berbeda, dan mencampurkan kedua pembagi tersebut adalah cara paling umum untuk mendapatkan nilai reflektansi yang tepat setengah dari yang seharusnya.
{% endhint %}

| Sumber | DN yang setara dengan reflektansi 1,0 | Diidentifikasi oleh |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Tag XMP `Chloros:PixelScale=32768` yang tercantum pada setiap ekspor reflektansi LATTICE. Ruang headroom 2× berarti nilai ρ di atas 1,0 dapat direpresentasikan alih-alih dipotong |
| **Survey3**| **65535** | Tidak ada tag skala XMP Chloros — kalibrasi Survey3 menulis ρ × dtype-max dan memotong pada 1,0 |

Untuk GIS dan skrip: baca `Chloros:PixelScale` dari berkas dan bagi dengan nilainya. Jika tag tersebut tidak ada, berkas tersebut menggunakan skala Survey3 (65535). Penampil, kotak pasir indeks/LUT, dan ekspor indeks semuanya menentukan skala ini dengan cara yang sama, sehingga angka yang Anda baca di kursor adalah angka yang digunakan dalam perhitungan indeks.

Penyimpanan khusus format di atas skala tersebut:

* **TIFF (32-bit, Persen)** menyimpan DN / 65535 sebagai bilangan float
* **PNG (8-bit)**dan**JPG (8-bit)** menyimpan DN × 255 / 65535
* **Hasil perkalian DN 8-bit dari pengambilan gambar 8-bit**, atau berkas reflektansi yang ditulis sebelum versi 1.3.0, dipotong menjadi 0–255 dan tidak memiliki tag skala. Panel hanya menampilkan DN untuk berkas-berkas tersebut, tanpa kolom persentase

### Rentang nilai indeks

| Keluarga indeks | Rentang umum | Nilai |
| --- | --- | --- |
| Perbedaan ternormalisasi (NDVI, GNDVI, NDRE, ENDVI…) | −1 hingga +1 | Vegetasi sehat umumnya 0,4–0,9; tanah gundul mendekati 0; air bernilai negatif |
| Disesuaikan dengan tanah (SAVI, OSAVI, MSAVI2…) | kira-kira −1 hingga +1,5 | Nilai serupa dengan NDVI namun latar belakang tanah ditekan |
| Rasio (GRVI, GCI, MSR, CIRE…) | tak terbatas ke atas | Rasio meningkat tanpa batas saat pita pembilang mendekati nol |
| EVI / LAI | 0 hingga ~1, 0 hingga ~3,5 | Awan dan piksel jenuh lainnya membuat keduanya keluar dari rentang — tutupi terlebih dahulu |

Lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md) untuk rumus tepat di balik setiap preset.

***

## Alur kerja umum

### Perbandingan sebelum / sesudah

1. Pilih **RAW (Asli)** dan perhatikan efek vignetting serta nilai-nilai yang belum dikalibrasi
2. Beralih ke **RAW (Reflektansi)**

3. Bandingkan — vignetting telah dihilangkan, nilai-nilai telah dikalibrasi. Perbesar dan geser tampilan tetap, sehingga Anda melihat area permukaan yang sama

### Tinjau satu indeks di seluruh rangkaian gambar

1. Buka gambar pertama yang telah diproses dan pilih lapisan indeks
2. Tekan **→** berulang kali — lapisan indeks akan mengikuti pergerakan Anda dari gambar ke gambar
3. Perhatikan histogram di bilah samping saat Anda menelusuri: bingkai yang distribusinya melonjak perlu diperiksa lebih lanjut

### Verifikasi target kalibrasi

1. Pilih **RAW (Target)** pada bingkai target
2. Pastikan target terlihat jelas dan terdeteksi
3. Pindah ke bingkai target berikutnya — lapisan target akan mengikuti

### Periksa keakuratan nilai reflektansi

1. Pilih **RAW (Reflectance)**

2. Baca kolom**%** di panel Nilai Kursor — nilainya sudah diskalakan dengan benar untuk file tersebut
3. Lakukan pemeriksaan validasi terhadap material yang diketahui dalam bingkai: vegetasi yang sehat memiliki nilai tinggi di NIR dan rendah di merah; target kalibrasi seharusnya menunjukkan nilai yang mendekati nilai reflektansi yang dipublikasikan

***

## Pemecahan Masalah

### Lapisan yang saya harapkan tidak ada di menu dropdown

**Kemungkinan penyebab**

* Gambar tersebut belum pernah diproses — hanya lapisan dasar dan `RAW (Original)` yang ada
* Tombol pengaturan ekspor produk tidak dicentang di Pengaturan Proyek
* Produk tersebut tidak berlaku untuk kamera tersebut (radian dan reflektansi pada master RGB; indeks apa pun pada kamera mono M3M pita tunggal)
* Kalibrasi reflektansi tidak memiliki data yang dapat diproses — tidak ada cakupan downwelling `.daq` dan tidak ada target dalam bingkai yang lulus QA — sehingga bingkai tersebut kembali ke Vignette Corrected atau Sensor Response

**Apa yang harus dilakukan**

1. Periksa log proses: Chloros akan menunjukkan kapan produk ekspor yang diminta tidak dapat dihasilkan dan alasannya
2. Periksa opsi ekspor per produk di [Pengaturan Proyek](../project-settings/project-settings.md)
3. Pastikan folder produk tersebut ada di pohon output proyek
4. Proses ulang dengan produk tersebut diaktifkan

### Daftar lapisan tampak kedaluwarsa

Chloros memindai ulang folder produk proyek saat proses berjalan dan memperbaiki pendaftaran lapisan yang hilang berdasarkan data yang sebenarnya ada di disk, sehingga lapisan yang telah selesai diekspor biasanya muncul dengan sendirinya dalam satu pemindaian. Beralih dari gambar dan kembali ke tampilan tersebut akan memaksa pembaruan resolusi.

### Nilai reflektansi tampak setengah dari yang seharusnya

Anda hampir pasti membagi berkas LATTICE dengan 65535. Gunakan `Chloros:PixelScale` (32768), atau periksa kolom **%**, yang telah menerapkan nilai tersebut.

### Lapisan indeks ada tetapi gambarnya kosong

Indeks tersebut memerlukan pita yang tidak dimiliki oleh lapisan Anda — misalnya, indeks yang membaca saluran ketiga yang diterapkan pada berkas satu atau dua saluran. Beralihlah ke lapisan multi-pita (reflektansi atau debayered), atau pilih indeks yang sesuai dengan filter kamera.

***

## Langkah Selanjutnya

* [**Membuka Gambar dalam Layar Penuh**](opening-an-image-full-screen.md) — pembacaan kursor, histogram, dan pengaturan GSD
* [**Sandbox Indeks/LUT**](index-lut-sandbox.md) — visualisasi indeks interaktif dan ekspor
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) — referensi indeks
* [**Menyelesaikan Pemrosesan**](../processing-images-gui/finishing-the-processing.md) — struktur folder keluaran yang dituju oleh lapisan-lapisan ini

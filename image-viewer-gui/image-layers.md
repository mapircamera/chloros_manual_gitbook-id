# Lapisan Gambar

**Menu tarik-turun lapisan** di pojok kanan atas Penampil Gambar memungkinkan Anda beralih di antara setiap versi gambar yang sedang Anda lihat — mulai dari tangkapan sumber, melalui setiap hasil pemrosesan, hingga gambar indeks yang dihitung — tanpa perlu keluar dari penampil.

## Apa Itu Lapisan Gambar?

Sebuah &quot;lapisan&quot; di Chloros adalah satu **berkas produk**yang didaftarkan terhadap satu gambar sumber. Proses impor memberikan berkas-berkas sumber; pemrosesan menambahkan satu lapisan untuk setiap produk yang dihasilkan oleh proses tersebut. File yang diekspor tetap menggunakan nama file sumber —**folder**-lah yang mengidentifikasi produk, dan nama lapisan adalah label yang diberikan Chloros untuk folder tersebut.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## Daftar lapisan

### Selalu ada

| Lapisan | Apa itu |
| --- | --- |
| **JPG**(atau**PNG**/**TIFF**) | File dasar yang disertakan bersama tangkapan. Survey3 mengimpor file `.JPG` di samping setiap file `.RAW`; Perekaman LATTICE menampilkan pratinjau layar berupa PNG atau TIFF. Dilabeli sesuai dengan apa yang sebenarnya diimpor |
| **RAW (Asli)** | Frame mentah sumber, yang telah di-debayering untuk ditampilkan tanpa koreksi apa pun. Tersedia sejak saat impor — tidak memerlukan pemrosesan |

Tangkapan LATTICE yang file dasarnya **adalah** bingkai mentahnya tidak memiliki entri dasar terpisah: `RAW (Original)` sudah mencakupnya.

### Produk pemrosesan Survey3

| Lapisan | Ditulis ke | Ada saat |
| --- | --- | --- |
| **RAW (Target)** | — | Frame tersebut diidentifikasi mengandung target kalibrasi |
| **RAW (Reflektansi)** | `Reflectance_Calibrated_Images/` | Kalibrasi reflektansi berhasil dijalankan pada frame ini |
| **Vignette Corrected**| `Vignette_Corrected_Images/` | Frame tersebut tidak dapat dikalibrasi reflektansinya**dan** *koreksi vignette* diaktifkan |
| **Respon Sensor**| `Sensor_Response_Images/` | Bingkai ini tidak dapat dikalibrasi reflektansinya**dan** *koreksi vignette* dinonaktifkan |
| **Keseimbangan Putih** | `White_Balanced_Images/` | Produk dengan keseimbangan putih telah disimpan |

{% hint style="info" %}
**Koreksi Vignette dan Respons Sensor adalah alternatif, tidak pernah keduanya sekaligus.** Terdapat tepat satu produk cadangan yang tidak dikalibrasi per proses, untuk setiap model kamera, dan tombol *Koreksi Vignette* menentukan mana yang dipilih. Lihat [Pengaturan Proyek](../project-settings/project-settings.md).
{% endhint %}

### Tingkat LATTICE

LATTICE menangkap penyebaran data ke tingkat-tingkat ini dalam satu tahap pemrosesan. Tingkat mana saja yang tersedia bergantung pada pengaturan ekspor per-produk di Pengaturan Proyek dan pada spesifikasi kamera yang bersangkutan.

| Lapisan | Ditulis ke | Berlaku untuk |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB dan multispektral |
| **RAW (Pratinjau)** | `Preview_Images/` | Multispektral (perluasan warna palsu) |
| **Keseimbangan Putih** | `Preview_Images/` | Kamera utama RGB — pratinjau RGB terdaftar dengan nama ini agar sejajar dengan lapisan Survey3 yang memiliki nama sama |
| **RAW (Radiance)** | `Radiance_Images/` | Hanya multispektral |
| **RAW (Reflektansi)** | `Reflectance_Calibrated_Images/` | Hanya multispektral, dan hanya jika ada catatan downwelling `.daq` yang sesuai atau target dalam bingkai yang lulus QA yang mencakup bingkai tersebut |

Kamera utama RGB tidak memiliki radiometri per-pita, sehingga radiance dan reflectance dilewati untuk kamera tersebut sebagai **tidak berlaku** — log akan mencatat hal ini daripada gagal tanpa pemberitahuan.

### Lapisan indeks, LUT, dan sandbox

| Pola lapisan | Contoh | Asal |
| --- | --- | --- |
| **RAW (`<INDEX>` Indeks)** | `RAW (NDVI Index)` | Satu per indeks yang dikonfigurasi di Pengaturan Proyek, dihitung selama pemrosesan |
| **`<INDEX>` LUT** | `NDVI LUT` | Versi yang telah dipetakan warnanya dari sebuah indeks |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Satu per proses ekspor [Indeks/LUT Sandbox](index-lut-sandbox.md) proses ekspor |

Jika nama indeks yang sama dikonfigurasi lebih dari sekali dengan pengaturan yang berbeda, indeks kedua dan seterusnya akan diberi nomor pada namanya (`RAW (NDVI2 Index)`) agar lapisan-lapisan tersebut tetap dapat dibedakan.

***

## Menggunakan pemilih lapisan

1. Buka gambar dalam tampilan layar penuh dengan mengklik thumbnail di kisi
2. Klik **menu tarik-turun lapisan** di pojok kanan atas penampil
3. Pilih lapisan — gambar akan diperbarui seketika

Menu tarik-turun menampilkan **JPG, RAW (Asli), RAW (Target), RAW (Reflektansi)** terlebih dahulu, dalam urutan tersebut, dan mencantumkan semua yang lain setelahnya sesuai urutan pendaftaran produk.

### Preferensi lapisan saat menavigasi

Menekan **←**/**→** akan berpindah ke gambar berikutnya dan berusaha mempertahankan lapisan yang sama:

1. **Kecocokan persis didahulukan** — jika gambar berikutnya memiliki lapisan dengan nama yang sama, Anda akan beralih ke lapisan tersebut. Inilah yang membuat Anda tetap berada di lapisan `RAW (NDVI Index)` saat menelusuri seluruh set
2. **Kemudian pencocokan berdasarkan jenis** — lapisan indeks mencari lapisan indeks apa pun, LUT mencari LUT apa pun, reflektansi mencari reflektansi, target mencari target, asli mencari asli, dan dasar mencari dasar
3. **Kemudian, khusus untuk lapisan ekspor** — nama tetap dipertahankan meskipun daftar lapisan belum sepenuhnya terperbarui, karena berkas tersebut sudah ada di disk. Inilah yang memungkinkan Anda meninjau hasil sementara proses masih sedang menulisnya
4. **Jika tidak** — lapisan pertama yang tersedia, yang biasanya adalah gambar dasar

File sidecar `.daq` dan `.csv` dalam proyek dilewati saat navigasi menggunakan tombol panah, sehingga saat menelusuri gambar, Anda tidak akan pernah mendarat pada rekaman sensor cahaya.

Fitur zoom dan pan juga berlaku di seluruh gambar, sehingga perbandingan sebelum/sesudah pada posisi bidang yang sama menjadi mudah.

***

## Memahami nilai piksel per lapisan

[Panel Nilai Kursor](opening-an-image-full-screen.md#cursor-values) menampilkan nilai sebenarnya per kanal di bawah kursor Anda, dalam satuan yang digunakan untuk menyimpan lapisan tersebut. Kolom-kolomnya berubah sesuai dengan lapisan:

| Lapisan | Satuan yang ditampilkan | Catatan |
| --- | --- | --- |
| Dasar (JPG / PNG / pratinjau TIFF) | DN, 0–255 | Nilai tampilan, telah dikoreksi gamma pada RGB. Hanya untuk pemeriksaan visual |
| RAW (Asli) | DN | Angka digital mentah dari sensor. Sumbu histogram menunjukkan kedalaman: 255 (8-bit), 4095 (12-bit), atau 65535 (16-bit) |
| RAW (Debayered) | DN | Linear, tanpa peregangan tampilan |
| RAW (Pratinjau) / Keseimbangan Putih | DN | Produk tampilan — diregangkan atau dikoreksi gamma. Bukan untuk pengukuran |
| RAW (Radian) | **W/m²/sr/nm** | Radian fisik Float32. Tanpa kolom DN |
| RAW (Reflektansi) | DN **dan %** | Persentase dihitung berdasarkan skala file tersebut — lihat di bawah |
| Ekspor indeks / LUT / sandbox | Nilai indeks, atau komponen RGB | File indeks saluran tunggal melaporkan nilai indeks; file LUT yang dipetakan warna melaporkan komponen Red/Green/Blue |

### Reflektansi: skalanya per berkas

{% hint style="warning" %}
**&quot;Bagi dengan 65.535&quot; hanya benar untuk Survey3.** Reflektansi LATTICE disimpan dalam skala yang berbeda, dan mencampurkan kedua pembagi tersebut adalah cara paling umum untuk mendapatkan nilai reflektansi yang tepat setengah dari yang seharusnya.
{% endhint %}

| Sumber | DN yang setara dengan reflektansi 1,0 | Diidentifikasi oleh |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Tag XMP `Chloros:PixelScale=32768` yang dicantumkan pada setiap ekspor reflektansi LATTICE. Ruang kosong 2× berarti ρ di atas 1,0 dapat direpresentasikan alih-alih dipotong |
| **Survey3**|**65535** | Tidak ada tag skala XMP Chloros — kalibrasi Survey3 menulis ρ × dtype-max dan memotong pada 1,0 |

Untuk GIS dan skrip: baca nilai dari berkas dan bagi dengan nilai tersebut. Jika tag tidak ada, berkas menggunakan skala (65535). Penampil, kotak pasir indeks/LUT, dan ekspor indeks semuanya menentukan skala dengan cara yang sama, sehingga angka yang Anda baca di kursor adalah angka yang digunakan dalam perhitungan indeks.

Penyimpanan khusus format di atas skala tersebut:

* **TIFF (32-bit, Persen)** menyimpan DN / 65535 sebagai bilangan float
* **PNG (8-bit)**dan**JPG (8-bit)** menyimpan DN × 255 / 65535
* **Ekspor 8-bit TIFF dari tangkapan sumber 8-bit** dipotong menjadi 0–255 alih-alih diskalakan ulang, dan sengaja tidak menyertakan tag skala. Panel hanya menampilkan DN untuk file-file tersebut, tanpa kolom persentase

### Rentang nilai indeks

| Keluarga indeks | Rentang umum | Pembacaan |
| --- | --- | --- |
| Perbedaan ternormalisasi (NDVI, GNDVI, NDRE, ENDVI…) | −1 hingga +1 | Vegetasi sehat umumnya 0,4–0,9; tanah telanjang mendekati 0; air bernilai negatif |
| Disesuaikan dengan tanah (SAVI, OSAVI, MSAVI2…) | kira-kira −1 hingga +1,5 | Nilai yang serupa dengan NDVI dengan latar belakang tanah yang ditekan |
| Rasio (GRVI, GCI, MSR, CIRE…) | tak terbatas ke atas | Rasio terus meningkat tanpa batas saat rentang pembilang mendekati nol |
| EVI / LAI | 0 hingga ~1, 0 hingga ~3,5 | Awan dan piksel jenuh lainnya membuat keduanya keluar dari rentang — tutupi terlebih dahulu |

Lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md) untuk rumus tepat di balik setiap preset.

***

## Alur kerja umum

### Perbandingan sebelum / sesudah

1. Pilih **RAW (Asli)** dan perhatikan efek vignetting serta nilai-nilai yang belum dikalibrasi
2. Beralih ke **RAW (Reflektansi)**

3. Bandingkan — efek vignetting telah dihilangkan, nilai-nilai telah dikalibrasi. Perbesar dan geser tampilan tetap terkunci, sehingga Anda melihat area tanah yang sama

### Meninjau satu indeks di seluruh rangkaian gambar

1. Buka gambar pertama yang telah diproses dan pilih lapisan indeks
2. Tekan **→** berulang kali — lapisan indeks akan mengikuti Anda dari gambar ke gambar
3. Perhatikan histogram di bilah samping saat Anda menelusuri: bingkai yang distribusinya melonjak layak untuk diperiksa lebih dekat

### Verifikasi target kalibrasi

1. Pilih **RAW (Target)** pada bingkai target
2. Pastikan target terlihat jelas dan terdeteksi
3. Pindah ke bingkai target berikutnya — lapisan target akan mengikuti

### Periksa keakuratan nilai reflektansi

1. Pilih **RAW (Reflectance)**

2. Baca kolom**%** di panel Nilai Kursor — nilainya sudah diskalakan dengan benar untuk file tersebut
3. Lakukan pemeriksaan validasi terhadap bahan yang diketahui dalam bingkai: vegetasi yang sehat memiliki nilai NIR yang tinggi dan nilai merah yang rendah; target kalibrasi seharusnya menunjukkan nilai reflektansi yang mendekati nilai yang dipublikasikan

***

## Pemecahan Masalah

### Lapisan yang saya harapkan tidak ada dalam menu tarik-turun

**Kemungkinan penyebab**

* Gambar tersebut belum pernah diproses — hanya lapisan dasar dan `RAW (Original)` yang ada
* Tombol ekspor produk tidak dicentang di Pengaturan Proyek
* Produk tersebut tidak berlaku untuk kamera tersebut (radiance dan reflektansi pada master RGB; indeks apa pun pada kamera mono M3M single-band)
* Kalibrasi reflektansi tidak memiliki data yang dapat diproses — tidak ada cakupan sinar turun (downwelling) pada `.daq` dan tidak ada target dalam bingkai yang lulus QA — sehingga bingkai tersebut kembali menggunakan Vignette Corrected atau Sensor Response

**Apa yang harus dilakukan**

1. Periksa log proses: Chloros menunjukkan kapan produk ekspor yang diminta tidak dapat dihasilkan dan alasannya
2. Periksa tombol pengaturan ekspor per produk di [Pengaturan Proyek](../project-settings/project-settings.md)
3. Pastikan folder produk tersebut ada di pohon output proyek
4. Proses ulang dengan produk tersebut diaktifkan

### Daftar lapisan terlihat sudah kedaluwarsa

Chloros memindai ulang folder produk proyek saat proses berjalan dan memperbaiki pendaftaran lapisan yang hilang berdasarkan data yang sebenarnya ada di disk, sehingga lapisan yang telah selesai diekspor biasanya muncul dengan sendirinya dalam satu siklus pemindaian. Beralih dari gambar dan kembali ke proyek akan memaksa pembaruan resolusi.

### Nilai reflektansi tampak setengah dari yang seharusnya

Anda hampir pasti membagi berkas LATTICE dengan 65535. Gunakan `Chloros:PixelScale` (32768), atau periksa kolom **%**, yang telah menerapkan nilai tersebut.

### Lapisan indeks ada tetapi gambarnya kosong

Indeks tersebut memerlukan band yang tidak dimiliki oleh lapisan Anda — misalnya, indeks yang membaca saluran ketiga yang diterapkan pada berkas satu atau dua saluran. Beralihlah ke lapisan multi-band (reflektansi atau debayered), atau pilih indeks yang sesuai dengan filter kamera.

***

## Langkah Selanjutnya

* [**Membuka Gambar dalam Layar Penuh**](opening-an-image-full-screen.md) — pembacaan kursor, histogram, dan pengaturan GSD
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — visualisasi indeks interaktif dan ekspor
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) — referensi indeks
* [**Menyelesaikan Pemrosesan**](../processing-images-gui/finishing-the-processing.md) — struktur folder keluaran yang dituju oleh lapisan-lapisan ini

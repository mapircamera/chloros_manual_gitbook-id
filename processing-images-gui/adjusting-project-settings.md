# Menyesuaikan Pengaturan Proyek

Sebelum memproses gambar Anda, penting untuk mengonfigurasi pengaturan proyek agar sesuai dengan kebutuhan alur kerja Anda. Panel Pengaturan Proyek <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> menyediakan kontrol menyeluruh atas kalibrasi, opsi pemrosesan, indeks multispektral, dan format ekspor.

## Mengakses Pengaturan Proyek

1. Buka proyek Anda di Chloros
2. Klik ikon **Pengaturan Proyek** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> di bilah sisi kiri
3. Panel Pengaturan Proyek menampilkan semua opsi konfigurasi

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>Panel Pengaturan Proyek — Tampilan, Deteksi Target, dan Pemrosesan</p></figcaption></figure>

{% hint style="info" %}
**Pengaturan disimpan secara otomatis** bersama proyek Anda. Saat Anda membuka kembali proyek, semua pengaturan akan dipulihkan.
{% endhint %}

***

## Pengaturan Cepat untuk Alur Kerja Umum

### Pengaturan Default (Direkomendasikan untuk Sebagian Besar Pengguna)

Pengaturan default berfungsi dengan baik untuk alur kerja Survey3 dan LATTICE yang umum:

* ✅ **Koreksi vignette**: Diaktifkan
* ✅ **Kalibrasi reflektansi / keseimbangan putih**: Diaktifkan (menggunakan target MAPIR dan/atau data sensor cahaya DAQ)
* ✅ **Metode debayering**: Standar (Cepat, Kualitas Sedang)
* ✅ **Format ekspor**: TIFF (16-bit) (TIFF (8-bit) untuk proyek yang seluruhnya 8-bit)
* ✅ **Semua produk ekspor**: Diaktifkan (LATTICE secara otomatis menghasilkan file debayered, pratinjau, radiance, dan reflektansi)

Cukup impor gambar Anda dan mulailah pemrosesan dengan pengaturan default ini.

***

## Ikhtisar Pengaturan Proyek

Panel Pengaturan Proyek disusun ke dalam bagian-bagian di bawah ini. Dua bagian tambahan — **Sensor Cahaya DAQ**dan**Penyelarasan Array** — muncul secara otomatis jika proyek Anda berisi file yang relevan. Untuk dokumentasi lengkap, lihat [Pengaturan Proyek](../project-settings/project-settings.md).

### Tampilan

* **Resolusi Gambar Miniatur**: Resolusi gambar miniatur kisi gambar. Opsi:**Default (512 px)**,**1024 px**,**2048 px**,**Resolusi penuh**. Hanya untuk tampilan — tidak pernah memengaruhi pemrosesan. Nilai yang lebih tinggi terlihat lebih tajam saat diperbesar, tetapi memuat lebih lambat.

### Deteksi Target

Mengontrol cara Chloros mengidentifikasi target kalibrasi dalam gambar Anda.

**Pengaturan utama:*** **Luas sampel kalibrasi minimum (px)**: Ambang batas ukuran untuk deteksi target (default:**25**, rentang 0–10.000)
* **Pengelompokan Target Minimum (0–100)**: Ambang batas kemiripan untuk mengelompokkan wilayah target (default:**60**)**Kapan harus menyesuaikan:**

* Tingkatkan luas sampel jika terjadi deteksi palsu
* Kurangi jika target tidak terdeteksi
* Sesuaikan pengelompokan jika target terpecah menjadi beberapa deteksi

{% hint style="info" %}
Pengaturan ini tidak dapat diubah (berwarna abu-abu) saat **Kalibrasi reflektansi / keseimbangan putih** dinonaktifkan — jika dinonaktifkan, deteksi target tidak akan berjalan sama sekali.
{% endhint %}

### Pemrosesan

Opsi pemrosesan gambar utama dan kalibrasi.

**Pengaturan utama:*** **Koreksi vignette**: Mengkompensasi penggelapan lensa di tepi ✅ Direkomendasikan
* **Kalibrasi reflektansi / keseimbangan putih**: Mengkalibrasi gambar menggunakan target yang terdeteksi (Survey3) dan/atau data sensor cahaya DAQ (LATTICE) ✅ Direkomendasikan
* **Metode debayer**: Algoritma untuk mengonversi RAW menjadi multispektral 3-saluran
* **Interval kalibrasi ulang minimum**: Waktu minimum dalam detik antara penggunaan target kalibrasi (default:**0** = gunakan semua, rentang 0–3600)**Produk cadangan yang tidak dikalibrasi:**Ketika sebuah bingkai tidak dapat dikalibrasi reflektansinya (tidak ada target yang tersedia, atau kalibrasi dinonaktifkan), bingkai tersebut diekspor sebagai salah satu dari dua produk cadangan —**hanya satu dari pasangan tersebut yang ada per proses**, dipilih oleh tombol koreksi Vignette:

* **Ekspor respons sensor**: menulis `Sensor_Response_Images` — digunakan saat koreksi Vignette**dinonaktifkan*** **Ekspor dengan koreksi vignette**: menulis `Vignette_Corrected_Images` — digunakan saat koreksi Vignette**aktif**Kotak centang yang tidak aktif akan berwarna abu-abu. Mencabut centang pada kotak yang aktif akan menghentikan penulisan file tersebut sama sekali.**Produk ekspor LATTICE** (ditampilkan untuk setiap proyek; berlaku untuk pengambilan gambar LATTICE):

* **Ekspor gambar yang telah di-debayered**: gambar yang telah melalui proses debayering linier (`Debayered_Images`): offset gelap dari paket kalibrasi dihilangkan, kemudian dilakukan demosaicing, tanpa flat field atau unmix — file yang sama dengan yang disimpan oleh penangkapan pada tingkat debayeringnya — dalam orientasi asli sensor. Berlaku untuk modul RGB dan multispektral.
* **Ekspor pratinjau**: pratinjau tampilan (`Preview_Images`). RGB = keseimbangan putih (sumber cahaya DAQ jika tersedia, jika tidak maka gray-world) + gamma; multispektral = peregangan warna palsu.
* **Ekspor radian**: radian spektral float32 (`Radiance_Images`, W/m²/sr/nm). Hanya untuk modul multispektral — tidak berlaku untuk master RGB.
* **Ekspor reflektansi**: reflektansi tipe uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) jika pembacaan sinar turun (downwelling) `.daq` atau target dalam bingkai mencakup seluruh bingkai. Hanya modul multispektral.

Keempatnya **aktif secara default**— satu bingkai mentah LATTICE yang diimpor akan dibagi ke setiap produk yang diaktifkan dan berlaku dalam satu proses pemrosesan. Kotak centang**Ekspor reflektansi** akan dinonaktifkan (berwarna abu-abu) jika kalibrasi reflektansi dimatikan. Pengaturan yang tidak dapat diaktifkan karena sakelar induknya selalu dinonaktifkan dengan tooltip yang menyebutkan sakelar yang harus diubah.**Pengaturan lanjutan:*** **Pergeseran zona waktu sensor cahaya**: Jam dari UTC untuk penyesuaian waktu sensor cahaya (default: 0, rentang −12 hingga +12)
* **Terapkan koreksi PPK**: Menggunakan data GPS/pin eksposur dari berkas `.daq` (default: nonaktif)
* **Pin Eksposur 1/2**: Menetapkan kamera ke pin eksposur untuk konfigurasi dua kamera

{% hint style="info" %}
**Tingkat masukan LATTICE bersifat otomatis.** Rekaman LATTICE menyimpan tingkat pemrosesannya dalam metadata XMP, dan pemrosesan selalu masuk ke alur kerja pada bingkai mentah — tidak ada yang perlu dikonfigurasi di antarmuka pengguna grafis (GUI). (Bendera CLI `--input-level` tersedia sebagai opsi penggantian untuk pengguna tingkat lanjut pada tangkapan dengan metadata yang hilang; lihat [Referensi CLI](../reference/cli-reference.md).)
{% endhint %}

### Metode Debayer

Saat ini kami menawarkan 2 metode debayering di Chloros:

#### Standar (Cepat, Kualitas Sedang)

Debayer Standar memproses dengan cepat tetapi menampilkan noise warna akibat debayering, sehingga menghasilkan gambar yang kurang akurat dan lebih berisik.

#### Texture Aware (Lambat, Kualitas Tertinggi) \[Hanya Chloros+]

Texture Aware menggunakan metode debayering berkualitas tinggi yang peka terhadap tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir seluruh noise debayering. Model ini memerlukan memori GPU (VRAM) untuk berjalan: dengan **7 GB atau lebih VRAM**, model ini dapat memproses beberapa gambar secara bersamaan; di bawah 7 GB, model ini hanya dapat memproses satu gambar pada satu waktu (jauh lebih lambat). Lihat [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**Pengambilan gambar LATTICE selalu menggunakan demosaic Standar.** Tidak ada model Texture Aware yang dilatih untuk LATTICE, sehingga opsi ini tidak ditawarkan untuk citra LATTICE — citra Survey3 dalam proyek yang sama masih dapat menggunakannya.
{% endhint %}

### Indeks (Indeks Multispektral)

Konfigurasikan indeks vegetasi mana yang akan dihitung dan diekspor. Menu tarik-turun antarmuka pengguna (GUI) menawarkan **27 rumus indeks yang telah ditentukan sebelumnya**.**Cara menambahkan indeks:**

1. Klik tombol**"Tambahkan indeks"**

2. Pilih indeks dari menu dropdown (NDVI, NDRE, GNDVI, dll.)
3. Konfigurasikan pengaturan visualisasi (warna LUT, rentang nilai)
4. Tambahkan beberapa indeks sesuai kebutuhan

**Indeks populer:*** **NDVI**: Kesehatan vegetasi secara umum (paling umum)
* **NDRE**: Deteksi stres dini dengan RedEdge
* **GNDVI**: Sensitif terhadap konsentrasi klorofil
* **OSAVI**: Berfungsi baik pada tanah yang terlihat
* **EVI**: Wilayah dengan indeks luas daun (LAI) tinggi

**Rumus kustom:**

* Buat rumus indeks multispektral kustom dengan perhitungan band pada semua saluran gambar
* Simpan rumus kustom untuk digunakan kembali
* Rumus kustom merupakan fitur Chloros+; ketersediaannya bergantung pada tingkatan paket langganan Anda

Untuk semua indeks dan rumus yang tersedia — termasuk nama-nama yang hanya tersedia di antarmuka pengguna grafis (GUI) dan yang juga berfungsi di antarmuka baris perintah (CLI) atau SDK — lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md).

### Ekspor

Mengontrol format file keluaran.

**Format yang tersedia**(pengaturan:**Format gambar yang dikalibrasi**, default**TIFF (16-bit)**):

* **TIFF (16-bit)**: Direkomendasikan untuk analisis GIS dan ilmiah
* **TIFF (8-bit)**: TIFF 8-bit* **TIFF (32-bit, Persen)**: Nilai floating-point
* **PNG (8-bit)**: Kompresi tanpa kehilangan data untuk visualisasi
* **JPG (8-bit)**: Ukuran berkas terkecil, kompresi dengan kehilangan data

Daftar ini menunjukkan format yang dapat digunakan untuk gambar dalam proyek: Chloros tidak meningkatkan kedalaman bit. Gambar yang semuanya 8-bit hanya menawarkan TIFF (8-bit), PNG (8-bit), dan JPG (8-bit), serta secara default menggunakan TIFF (8-bit). Dalam proyek campuran, setiap pengambilan gambar diekspor pada kedalaman aslinya atau yang lebih rendah. Radiance tetap dalam format float 32-bit.

Hasil ekspor disimpan di dalam folder proyek, dikelompokkan berdasarkan kamera dan format: `<project>/<camera>/<format>/<Product>_Images/`. Radiance **selalu** disimpan sebagai float32 ke dalam folder `tiff32` terlepas dari pengaturan ini. File yang diekspor mempertahankan nama file sumber — folderlah yang mengidentifikasi produk. Ekspor multispektral LATTICE juga dapat menulis masker pembatas per piksel ke dalam folder `clamp_masks/` di samping folder-folder produk. Lihat [Menyelesaikan Pemrosesan](finishing-the-processing.md) untuk melihat struktur output lengkap.

{% hint style="warning" %}
**Membaca nilai reflektansi**: nilai DN yang berarti ρ = 1,0 bergantung pada kamera sumber — LATTICE menggunakan 32768 (ditandai sebagai XMP `Chloros:PixelScale`), sedangkan Survey3 menggunakan 65535. Baca tag tersebut daripada mengasumsikan nilai konstan. Lihat [Format Gambar Keluaran](../output-image-formats.md).
{% endhint %}

### Sensor Cahaya DAQ

Bagian ini mencantumkan setiap berkas DAQ downwelling (`.daq` / `.csv`) dalam proyek Anda, satu baris per berkas, yang menampilkan model sensor, nama berkas, dan **penutup** difuser yang berlaku untuk berkas tersebut.

* **Penggantian nilai cap (semua berkas)**: menu tarik-turun tunggal yang berlaku untuk seluruh proyek.**Otomatis** (default) menggunakan nilai batas yang terekam pada setiap berkas — sinar matahari diasumsikan jika tidak ada data yang terekam, namun DAQ-U, DAQ-M, dan DAQ-E dikirimkan tanpa nilai batas, dan Chloros tidak dapat mendeteksi nilai yang telah disesuaikan, sehingga pemrosesan akan memberikan peringatan jika menggunakan asumsi nilai batas: pilih nilai batas yang telah disesuaikan. Memilih batas akan menggantikan pengaturan pada setiap berkas kecuali berkas DAQ-E-S, di mana difusornya merupakan bagian dari kalibrasinya: rekaman mentah dikoreksi dengannya, dan rekaman yang sudah memiliki batas akan direferensikan ulang (koreksi yang terekam dibatalkan, batas yang dipilih diterapkan).
* Baris-baris tersebut akan menampilkan peringatan jika penutup yang direkam merupakan asumsi default hub dan bukan yang dikonfirmasi oleh operator, serta jika penutup yang dipilih tidak memiliki profil untuk model perangkat tersebut (penggantian akan ditolak untuk berkas tersebut).

Rekaman DAQ yang dibuat di tab Light Sensors ditambahkan ke proyek yang terbuka secara otomatis, dan berkas `.daq` / `.csv` yang diimpor akan muncul di sini segera setelah ditambahkan.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Pengaturan Proyek Bawah — Indeks, Format Ekspor, bagian Sensor Cahaya DAQ, dan kontrol templat/folder proyek</p></figcaption></figure>### Penyelarasan Array

Bagian ini **hanya** muncul jika setidaknya satu gambar dalam proyek memiliki transformasi penyelarasan antar-modul yang ditandai oleh array LATTICE saat pengambilan gambar (`Chloros:Alignment*` XMP). Bagian ini menunjukkan berapa banyak gambar yang memiliki tag dan kamera mana yang menjadi referensi, dengan pengaturan berikut:

* **Terapkan penyelarasan array** (default: aktif): menyesuaikan setiap produk yang diproses (debayered / pratinjau / radiance / reflektansi / indeks) ke dalam geometri referensi bersama array. Nonaktif = mengekspor dalam geometri sensor asli.
* **Pangkas ke tumpang tindih bersama** (default: aktif): memotong ekspor yang telah diselaraskan ke wilayah yang dimiliki bersama oleh semua modul, sehingga setiap pita memiliki area cakupan yang sama. Nonaktif mempertahankan kanvas sensor penuh (diisi hitam di luar sumber).
* **Resampling**:**Bilinear (halus, default)**,**Nearest (mempertahankan nilai tepat)**— tanpa pencampuran antar-piksel, untuk analisis radiometrik yang ketat — atau**Cubic (paling tajam)**.

***

## Menyimpan dan Memuat Pengaturan

### Simpan Template Proyek

Buat template yang dapat digunakan kembali untuk alur kerja yang konsisten:

1. Konfigurasikan semua pengaturan yang diinginkan di panel Pengaturan Proyek
2. Gulir ke bagian **"Simpan Template Proyek"** di bagian bawah
3. Masukkan nama templat yang deskriptif (misalnya, "Survey3N\_RGN\_Agriculture")
4. Klik ikon simpan

**Manfaat:**

* Menerapkan pengaturan yang identik di berbagai proyek
* Berbagi konfigurasi dengan anggota tim
* Menjaga konsistensi untuk survei berulang

### Memuat Templat pada Proyek Baru

Saat membuat proyek baru:

1. Pilih **"Proyek Baru"** dari menu utama
2. Pilih templat proyek di pemilih templat opsional
3. Semua pengaturan dari templat tersebut akan diterapkan secara otomatis

### Direktori Kerja

Pengaturan **"Direktori Kerja"** menentukan lokasi default pembuatan proyek baru:

* **Lokasi default**: `Documents\MAPIR\Chloros Projects` di folder Dokumen Windows Anda (biasanya `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; lokasi ini mengikuti folder Dokumen jika OneDrive atau profil roaming memindahkannya)
* **Ubah lokasi**: Klik ikon edit dan pilih folder baru
* **Dibagikan dengan CLI**: `chloros-cli` menggunakan pengaturan folder proyek default yang sama
* **Gunakan nama folder yang singkat di Windows**: kecuali jika jalur panjang Windows diaktifkan, folder proyek yang panjangnya melebihi sekitar 130 karakter akan ditolak sebelum pemrosesan — lihat [Memulai Pemrosesan](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Kapan harus mengubah**:
  * Drive jaringan untuk kolaborasi tim
  * Drive berbeda dengan ruang penyimpanan lebih besar
  * Struktur folder yang terorganisir berdasarkan tahun/klien

***

## Pengaturan PPK (Post-Processed Kinematic)

Jika menggunakan perekam DAQ MAPIR dengan GPS untuk geolokasi yang akurat:

### Persyaratan

* Perekam DAQ MAPIR dengan modul GPS (GNSS)
* Berkas log .daq dengan entri pin eksposur
* Kamera terhubung ke pin eksposur DAQ selama sesi perekaman

### Langkah-langkah Konfigurasi

1. Letakkan berkas log .daq di folder proyek Anda
2. Di Pengaturan Proyek, centang kotak **"Terapkan koreksi PPK"**

3. Atur**"Light sensor timezone offset"** jika diperlukan (default: 0 untuk UTC)
4. Tetapkan kamera ke pin eksposur:
   * **Kamera tunggal**: Secara otomatis ditetapkan ke Pin 1
   * **Kamera ganda**: Tetapkan setiap kamera secara manual ke pin yang benar**Penetapan Pin Eksposur:*** **Pin Eksposur 1**: Pilih model kamera dari menu tarik-turun
* **Pin Eksposur 2**: Pilih kamera kedua atau "Jangan Digunakan"
* Kamera yang sama tidak dapat ditetapkan ke kedua pin

{% hint style="warning" %}
**Penting**: Pin eksposur harus ditugaskan dengan benar ke kamera masing-masing. Penugasan yang salah akan mengakibatkan data geolokasi yang salah.
{% endhint %}

***

## Skenario Lanjutan

### Proyek Multi-Kamera

Saat memproses gambar dari beberapa kamera MAPIR dalam satu proyek:

1. Chloros secara otomatis mendeteksi setiap model kamera (baik Survey3 maupun LATTICE)
2. Setiap kamera mendapatkan profil pemrosesan yang sesuai, dan setiap kamera memiliki struktur folder keluaran tersendiri
3. PPK: Tetapkan secara manual setiap kamera Survey3 ke pin eksposur yang benar
4. Semua kamera menggunakan format ekspor dan indeks yang sama

**Contoh**: Rig kamera ganda Survey3W RGN + Survey3N OCN, atau susunan LATTICE yang menggabungkan kamera utama RGB dengan modul pita sempit

### Survei Time-Lapse atau Multi-Tanggal

Untuk survei berulang di area yang sama dari waktu ke waktu:

1. Buat templat dengan pengaturan standar Anda
2. Gunakan pengaturan target kalibrasi yang konsisten di setiap sesi
3. Proses setiap tanggal sebagai proyek terpisah
4. Gunakan pengaturan yang identik untuk hasil yang dapat dibandingkan
5. Ekspor dalam format yang sama untuk analisis temporal

### Kumpulan Data Besar

Untuk proyek dengan banyak gambar (500+):

* Pertimbangkan untuk membagi menjadi proyek-proyek yang lebih kecil berdasarkan tanggal atau area
* Gunakan pemrosesan paralel Chloros+ untuk hasil yang lebih cepat
* Pertimbangkan penggunaan CLI atau API untuk otomatisasi batch
* Sesuaikan interval kalibrasi ulang minimum untuk mengurangi waktu deteksi target

***

## Memverifikasi Pengaturan Anda

Sebelum memulai pemrosesan, periksa pengaturan kunci berikut:

* [ ] Model kamera terdeteksi dengan benar di File Browser
* [ ] Koreksi vignette diaktifkan
* [ ] Kalibrasi reflektansi diaktifkan
* [ ] Untuk Survey3: setidaknya satu gambar target kalibrasi telah diimpor dan diperiksa; untuk LATTICE: target dan/atau rekaman downwelling `.daq` tersedia
* [ ] Indeks multispektral yang diinginkan telah ditambahkan
* [ ] Format ekspor sesuai dengan alur kerja Anda
* [ ] Pengaturan PPK telah dikonfigurasi (jika menggunakan berkas .daq dengan peristiwa eksposur)

***

## Langkah Selanjutnya

Setelah pengaturan Anda dikonfigurasi:

1. **Tandai gambar target kalibrasi** - Lihat [Memilih Gambar Target](choosing-target-images.md)
2. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
3. **Pantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)

Untuk detail lengkap mengenai semua pengaturan yang tersedia, lihat dokumentasi referensi [Pengaturan Proyek](../project-settings/project-settings.md).

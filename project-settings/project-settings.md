# Pengaturan Proyek

Panel samping Pengaturan Proyek <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

di Chloros

memungkinkan Anda mengonfigurasi semua aspek pemrosesan gambar, deteksi target kalibrasi, perhitungan indeks multispektral, dan opsi ekspor untuk proyek Anda. Pengaturan ini disimpan bersama proyek Anda dan dapat disimpan sebagai templat untuk digunakan kembali di berbagai proyek.

## Mengakses Pengaturan Proyek

Untuk mengakses Pengaturan Proyek:

1. Buka proyek di Chloros


2. Klik tab **Pengaturan Proyek** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

di bilah samping kiri
3. Panel pengaturan akan menampilkan semua opsi konfigurasi yang tersedia, disusun berdasarkan kategori

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->



{% hint style="info" %}
**Pengaturan yang bergantung pada pengaturan lain akan ditampilkan abu-abu.** Ketika sebuah pengaturan induk membuat suatu pengaturan tidak dapat dilakukan (misalnya, menghapus centang pada *Kalibrasi reflektansi / keseimbangan putih* membuat *Ekspor reflektansi* tidak dapat dilakukan), kontrol yang bergantung akan dinonaktifkan dan tooltip-nya akan menyebutkan pengaturan induk yang harus diubah.
{% endhint %}

***

## Tampilan

### Resolusi Gambar Mini

* **Jenis**: Pilihan menu tarik-turun
* **Opsi**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Default**: Default (512 px)
* **Deskripsi**: Resolusi (sisi terpanjang, dalam piksel) yang digunakan untuk menampilkan thumbnail dalam kisi gambar. Nilai yang lebih tinggi akan terlihat lebih tajam saat diperbesar, tetapi memuat lebih lambat dan menggunakan lebih banyak memori. Resolusi penuh menampilkan ukuran gambar asli.
* **Catatan**: Hanya untuk tampilan — ini tidak pernah memengaruhi pemrosesan atau file yang diekspor.***

## Deteksi Target

Pengaturan ini mengontrol cara Chloros

mendeteksi dan memproses target kalibrasi dalam gambar Anda. Keduanya hanya aktif saat **Kalibrasi reflektansi / keseimbangan putih** diaktifkan (jika tidak, keduanya akan dinonaktifkan, karena deteksi target dilewati sepenuhnya).

### Luas sampel kalibrasi minimum (px)

* **Jenis**: Angka
* **Rentang**: 0 hingga 10.000 piksel
* **Default**: 25 piksel
* **Deskripsi**: Menentukan luas minimum (dalam piksel) yang diperlukan agar suatu wilayah yang terdeteksi dianggap sebagai sampel target kalibrasi yang valid. Nilai yang lebih kecil akan mendeteksi target yang lebih kecil, tetapi dapat meningkatkan jumlah hasil positif palsu. Nilai yang lebih besar memerlukan wilayah target yang lebih besar dan lebih jelas untuk dideteksi.
* **Kapan harus disesuaikan**:
  * Tingkatkan jika Anda mendapatkan deteksi palsu pada artefak gambar kecil
  * Kurangi jika target kalibrasi Anda tampak kecil dalam gambar dan tidak terdeteksi

### Pengelompokan Target Minimum (0-100)

* **Jenis**: Bilangan
* **Rentang**: 0 hingga 100
* **Default**: 60
* **Deskripsi**: Mengontrol ambang batas pengelompokan untuk menggabungkan wilayah berwarna serupa saat mendeteksi target kalibrasi. Nilai yang lebih tinggi memerlukan lebih banyak warna serupa untuk digabungkan, sehingga menghasilkan deteksi target yang lebih konservatif. Nilai yang lebih rendah memungkinkan variasi warna yang lebih banyak dalam satu kelompok target.
* **Kapan harus menyesuaikan**:
  * Naikkan jika target kalibrasi terpecah menjadi beberapa deteksi
  * Turunkan jika target kalibrasi dengan variasi warna tidak terdeteksi sepenuhnya

***

## Pemrosesan

Pengaturan ini mengontrol cara Chloros

memproses dan mengkalibrasi gambar Anda.

### Koreksi Vignette

* **Jenis**: Kotak centang
* **Default**: Diaktifkan (dicentang)
* **Deskripsi**: Menerapkan koreksi vignette untuk mengkompensasi penggelapan lensa di tepi gambar. Vignetting adalah fenomena optik umum di mana sudut dan tepi gambar tampak lebih gelap daripada bagian tengahnya akibat karakteristik lensa.
* **Efek samping**: Tombol ini juga menentukan *produk cadangan yang belum dikalibrasi* mana yang akan ditulis oleh proses (lihat di bawah).

### Kalibrasi reflektansi / keseimbangan putih

* **Jenis**: Kotak centang
* **Default**: Diaktifkan (dicentang)
* **Deskripsi**: Mengaktifkan kalibrasi reflektansi — dari target kalibrasi yang terdeteksi dalam bingkai dan/atau data cahaya yang ditangkap oleh sensor cahaya DAQ, tergantung pada kamera dan apa yang tersedia. Ini menormalisasi nilai reflektansi di seluruh dataset Anda dan memastikan pengukuran yang konsisten terlepas dari kondisi pencahayaan.
* **Saat dinonaktifkan**: Deteksi target dilewati sepenuhnya, dan**tidak ada produk reflektansi yang dapat dihasilkan oleh kamera mana pun** — baik yang digerakkan oleh target Survey3

maupun yang digerakkan oleh DAQ LATTICE. Pengaturan yang bergantung (*Ekspor reflektansi*, *Interval kalibrasi ulang minimum*, dan ambang batas Deteksi Target) akan dinonaktifkan (berwarna abu-abu).

### Produk cadangan tanpa kalibrasi: Ekspor respons sensor / Ekspor yang telah dikoreksi vignette

* **Jenis**: Dua kotak centang
* **Pengaturan default**: Keduanya diaktifkan (dicentang)
* **Deskripsi**: Ketika sebuah frame tidak dapat dikalibrasi reflektansinya (tidak ditemukan target kalibrasi, atau kalibrasi reflektansi dinonaktifkan), frame tersebut akan disimpan sebagai *produk cadangan yang belum dikalibrasi*. **Tepat satu dari dua produk cadangan tersebut ada per pengambilan gambar, untuk setiap model kamera**, yang dipilih oleh tombol *Koreksi vignette*:
  * Koreksi vignette **aktif**→ `Vignette_Corrected_Images/` (diatur oleh**Ekspor dengan koreksi vignette**)
  * Koreksi vignette **nonaktif**→ `Sensor_Response_Images/` (diatur oleh**Ekspor respons sensor**)
* Produk cadangan yang tidak aktif akan ditampilkan abu-abu. Mencabut centang pada produk yang aktif akan menghentikan penulisan file tersebut sama sekali.

### Produk ekspor LATTICE

Untuk proyek yang berisi tangkapan LATTICE, setiap bingkai LATTICE yang diimpor akan dibagi ke dalam setiap produk yang diaktifkan **dan berlaku**dalam satu proses pemrosesan. Empat kotak centang mengontrol penyebaran (semua default**aktif**):

| Pengaturan | Folder keluaran | Apa yang diekspor |
| --- | --- | --- |
| **Ekspor debayered** | `Debayered_Images/` | Gambar debayered linier: offset gelap dari paket kalibrasi telah dihilangkan, kemudian dilakukan demosaik yang memperhatikan tepi, tanpa koreksi linearitas, flat field, atau unmix — file yang sama dengan yang disimpan oleh pengambilan gambar pada tingkat debayered-nya. Tetap dalam orientasi asli sensor (tercatat dalam XMP). Berlaku untuk kamera RGB

dan multispektral. |
| **Ekspor pratinjau** | `Preview_Images/` | Pratinjau tampilan. RGB

= keseimbangan putih (sumber cahaya DAQ jika tersedia, jika tidak menggunakan skala abu-abu) + gamma; multispektral = peregangan warna palsu. |
| **Ekspor radian** | `Radiance_Images/` | Radian spektral Float32 dalam W/m²/sr/nm. Hanya multispektral (M3C/M3M) — tidak berlaku untuk master RGB

. Selalu ditulis sebagai TIFF

32-bit terlepas dari pengaturan *Format gambar yang dikalibrasi*. |
| **Ekspor reflektansi**| `Reflectance_Calibrated_Images/` | Reflektansi Uint16, diskalakan sehingga**32768 = reflektansi 1,0** (ditandai sebagai XMP `Chloros:PixelScale`). Hanya multispektral, ditulis ketika catatan downwelling `.daq` yang cocok (atau target dalam bingkai yang lulus QA) mencakup bingkai tersebut. |

* Kamera utama RGB

memancarkan data yang telah di-debayer + pratinjau; radian dan reflektansi dilewati untuk kamera ini karena tidak berlaku.
* Kedalaman bit data yang telah di-debayer/pratinjau mengikuti pengaturan *Format gambar yang dikalibrasi*; radian selalu berupa float32.
* Pemrosesan Survey3

tidak terpengaruh oleh keempat tombol pengaturan ini.
* Produk radian dan reflektansi dari kamera multispektral M3C juga dapat menulis **masker clamp** per piksel ke dalam folder `clamp_masks/` di samping folder produk, mencatat di mana pemisahan band memotong nilai negatif menjadi nol. Lihat [Format Gambar Keluaran](../output-image-formats.md#clamp-masks).

Keempat opsi tersebut juga tersedia dalam mode headless sebagai `chloros-cli process --debayered / --preview / --radiance / --reflectance`. Opsi-opsi ini menggantikan bendera CLI

lama `--radiometric-output`, yang kini tidak ada lagi.

{% hint style="warning" %}
**Menonaktifkan semua produk yang berlaku akan menyebabkan proses gagal.** Mulai versi 1.2.0, proses pengolahan yang diminta untuk menghasilkan produk tetapi tidak menulis laporan produk gambar akan melaporkan kegagalan, dan CLI

akan keluar dengan nilai non-nol, alih-alih melaporkan keberhasilan tanpa pemberitahuan. Log akan menyebutkan produk yang tidak dapat ditulis beserta alasannya. Proses yang sengaja hanya menghasilkan metadata (tidak ada yang diminta) tetap dianggap berhasil.
{% endhint %}

### Sumber reflektansi (pengaturan proyek, diatur melalui CLI

)

Proyek ini juga menyimpan **referensi reflektansi** mana yang digunakan oleh produk reflektansi LATTICE. Tidak ada kontrol khusus di panel pengaturan; nilainya disimpan dalam konfigurasi proyek sebagai `Processing → "Target reflectance source"` dan diatur dengan `chloros-cli process --reflectance-source {auto,target,daq}` (`configure()` di SDK

tidak memiliki kata kunci untuk itu):

* **`auto`** (default): target kalibrasi dalam bingkai yang lulus QA menjadi referensi absolut, dan akan beralih ke pembagian downwelling DAQ (ρ = πL/E) jika tidak ada target atau QA gagal. Target yang lulus QA dengan bendera (panel yang jatuh, atau pita yang tidak berfungsi) tidak secara otomatis diutamakan: rantai target, target+DAQ, dan DAQ-saja dicoba sesuai urutan skor yang diukur pada panel yang disisihkan.
* **`target`**: reflektansi yang didorong oleh target secara ketat — tanpa penggantian DAQ.
* **`daq`**: reflektansi yang ditentukan oleh DAQ; target dalam bingkai tidak digunakan sebagai referensi.

Nilai yang disimpan dicocokkan tanpa membedakan huruf besar-kecil, dan beberapa ejaan diterima sebagai alias: `target`, `target_image`, `empirical` dan `empirical_line` semuanya berarti **target**; `daq`, `dls`, `light_sensor`, dan `sensor` semuanya berarti**daq**. Yang lainnya — termasuk kunci yang tidak ada — akan diterjemahkan menjadi**auto**.

Pemindaian target **terukur** per unit dicari berdasarkan nomor seri/QR unit target, seperti `<serial>.csv`, di tiga tempat: direktori yang ditentukan dengan `--target-reflectance-dir` (disimpan sebagai `Processing → "Target reflectance dir"`), folder `target_reflectance/` milik proyek itu sendiri, dan jalur dalam variabel lingkungan `CHLOROS_TARGET_REFLECTANCE_DIR`. Jika tidak ada pemindaian terukur untuk unit tersebut, kurva nominal yang dipublikasikan untuk model target akan digunakan sebagai gantinya.

### Metode Debayer

* **Jenis**: Pilihan dropdown
* **Opsi**:
  * Standar (Cepat, Kualitas Sedang)
  * Sensitif Tekstur (Lambat, Kualitas Tertinggi) \[Chloros

+]
* **Default**: Standar (Cepat, Kualitas Sedang)
* **Deskripsi**: Memilih algoritma demosaicing yang digunakan untuk mengubah data mentah sensor pola Bayer menjadi gambar berwarna penuh. Metode &quot;Standar (Cepat, Kualitas Sedang)&quot; memberikan keseimbangan optimal antara kecepatan pemrosesan dan kualitas gambar. Metode &quot;Texture Aware (Lambat, Kualitas Tertinggi)&quot; \[Chloros

+] menggunakan debayer berkualitas tinggi yang peka tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir seluruh noise debayering. Model Texture Aware memerlukan memori GPU (VRAM) untuk dijalankan. Kami merekomendasikan penggunaannya jika Anda memiliki VRAM &gt;4GB yang tersedia untuk pemrosesan yang lebih cepat.
* **Apabila baris tersebut berupa menu tarik-turun**: menu tarik-turun dengan dua opsi hanya akan muncul jika**keduanya** benar — Anda telah masuk dengan langganan Chloros

+ yang memenuhi syarat, **dan** proyek tersebut tidak mengandung tangkapan LATTICE. Jika tidak, baris tersebut akan ditampilkan sebagai teks biasa yang berbunyi `Standard (Fast, Medium Quality)` tanpa opsi yang dapat dipilih.
* **Catatan LATTICE**: Tidak ada model Texture Aware yang dilatih untuk LATTICE, dan alur kerja memaksa penggunaan demosaic standar untuk bingkai LATTICE terlepas dari nilai yang disimpan. Jika Anda menambahkan folder LATTICE ke proyek yang sudah memilih Texture Aware, Chloros

akan mengubah pengaturan kembali ke Standar alih-alih membiarkan nilai yang sudah kadaluwarsa di `project.json`.

### Interval kalibrasi ulang minimum

* **Jenis**: Angka
* **Rentang**: 0 hingga 3.600 detik
* **Default**: 0 detik
* **Deskripsi**: Menentukan interval waktu minimum (dalam detik) antara penggunaan target kalibrasi. Jika diatur ke 0, Chloros

akan menggunakan setiap target kalibrasi yang terdeteksi. Jika diatur ke nilai yang lebih tinggi, Chloros

hanya akan menggunakan target kalibrasi yang terpisah setidaknya sebanyak detik tersebut, sehingga mengurangi waktu pemrosesan untuk dataset dengan penangkapan target kalibrasi yang sering.
* **Kapan harus menyesuaikan**:
  * Atur ke 0 untuk akurasi kalibrasi maksimum saat kondisi pencahayaan bervariasi
  * Tingkatkan (misalnya, menjadi 60–300 detik) untuk pemrosesan yang lebih cepat saat pencahayaan konsisten dan Anda memiliki gambar target kalibrasi yang sering

### Selisih zona waktu sensor cahaya

* **Jenis**: Angka
* **Rentang**: -12 hingga +12 jam
* **Default**: 0 jam
* **Deskripsi**: Menentukan selisih zona waktu (dalam jam dari UTC) untuk cap waktu data sensor cahaya, yang digunakan saat mencocokkan log sensor cahaya dengan waktu pengambilan gambar. Rekaman `.daq` yang lebih baru memiliki asal-usul zona waktu sendiri, sehingga pengaturan ini terutama diperlukan untuk log lama yang direkam dalam waktu lokal.

### Terapkan koreksi PPK

* **Jenis**: Kotak centang
* **Default**: Dinonaktifkan (tidak dicentang)
* **Deskripsi**: Mengaktifkan penggunaan koreksi Post-Processed Kinematic (PPK) dari perekam DAQ MAPIR

yang dilengkapi GPS (GNSS). Jika diaktifkan, Chloros

akan menggunakan file log .daq apa pun yang berisi data pin eksposur di direktori proyek Anda dan menerapkan koreksi geolokasi yang akurat pada gambar Anda.
* **Persyaratan**: File log .daq dengan entri pin eksposur harus ada di direktori proyek Anda
* **Kapan harus diaktifkan**: Disarankan untuk selalu mengaktifkan koreksi PPK jika Anda memiliki entri umpan balik eksposur dalam file log .daq Anda.

### Pin Eksposur 1

* **Jenis**: Pilihan dropdown
* **Visibilitas**: Hanya terlihat ketika &quot;Terapkan koreksi PPK&quot; diaktifkan DAN data eksposur tersedia untuk Pin 1
* **Opsi**:
  * Nama model kamera yang terdeteksi dalam proyek
  * &quot;Jangan Gunakan&quot; - Abaikan pin eksposur ini
* **Default**: Dipilih secara otomatis berdasarkan konfigurasi proyek
* **Deskripsi**: Menetapkan kamera tertentu ke Pin Eksposur 1 untuk sinkronisasi waktu PPK. Pin eksposur mencatat waktu yang tepat saat rana kamera dipicu, yang sangat penting untuk geolokasi PPK yang akurat.
* **Perilaku pemilihan otomatis**:
  * Satu kamera + satu pin: Secara otomatis memilih kamera
  * Satu kamera + dua pin: Pin 1 secara otomatis ditetapkan ke kamera
  * Beberapa kamera: Diperlukan pemilihan manual

### Pin Eksposur 2

* **Jenis**: Pilihan dropdown
* **Visibilitas**: Hanya terlihat jika &quot;Terapkan koreksi PPK&quot; diaktifkan DAN data eksposur tersedia untuk Pin 2
* **Opsi**:
  * Nama model kamera yang terdeteksi dalam proyek
  * &quot;Jangan Gunakan&quot; - Abaikan pin eksposur ini
* **Default**: Dipilih secara otomatis berdasarkan konfigurasi proyek
* **Deskripsi**: Menetapkan kamera tertentu ke Pin Eksposur 2 untuk sinkronisasi waktu PPK saat menggunakan pengaturan dua kamera.
* **Perilaku pemilihan otomatis**:
  * Satu kamera + satu pin: Pin 2 secara otomatis diatur ke &quot;Jangan Gunakan&quot;
  * Satu kamera + dua pin: Pin 2 secara otomatis diatur ke &quot;Jangan Gunakan&quot;
  * Beberapa kamera: Diperlukan pemilihan manual
* **Catatan**: Kamera yang sama tidak dapat ditetapkan ke Pin 1 dan Pin 2 secara bersamaan.***

## Sensor Cahaya DAQ

Bagian ini muncul di Pengaturan Proyek dan mencantumkan setiap file downwelling DAQ dalam proyek — rekaman `.daq` dan log downwelling DAQ-M `.csv`. Rekaman yang dibuat di tab Sensor Cahaya secara otomatis ditambahkan ke proyek yang terbuka.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Setiap baris menampilkan berkas, model sensor, dan koreksi penutup difuser yang sebenarnya berlaku untuk berkas tersebut. Di atas baris-baris tersebut terdapat satu pengaturan yang berlaku untuk seluruh proyek:

### Penggantian pengaturan penutup (semua berkas)

* **Jenis**: Pilihan menu tarik-turun
* **Opsi**: `Auto` ditambah profil koreksi penutup yang berlaku untuk jenis-jenis sensor yang ada dalam proyek
* **Default**: Otomatis
* **Disimpan sebagai**: `Processing → "DAQ cap id"` (default `auto`)
* **Deskripsi**: `Auto` menggunakan penutup yang tercatat pada setiap berkas (penutup Sunshine diasumsikan jika tidak ada yang tercatat, namun DAQ-U, DAQ-M, dan DAQ-E dikirim tanpa penutup, dan Chloros

tidak dapat mendeteksi penutup yang terpasang, sehingga pemrosesan akan memberikan peringatan terkait batas yang diasumsikan: pilih batas yang dipasang). Memilih batas tertentu akan menggantikan **semua** file downwelling dalam proyek kecuali milik DAQ-E-S, di mana difusornya merupakan bagian dari kalibrasinya; proyek yang hanya berisi DAQ-E-S tidak menampilkan menu dropdown: rekaman mentah dikoreksi menggunakan batas tersebut, dan rekaman yang sudah memiliki penutup akan direferensikan ulang (koreksi yang terekam dibatalkan dan yang dipilih diterapkan).
* **Penting**: Penutup yang dipilih harus sesuai dengan penutup yang secara fisik terpasang selama perekaman. Baik sensor maupun perangkat lunak tidak dapat mendeteksi penutup fisik — ID penutup yang tidak cocok akan mengoreksi spektrum secara salah.

Secara sengaja disediakan **satu** pengaturan yang berlaku untuk seluruh proyek, bukan menu dropdown per file: pengaturan ini berlaku untuk setiap sumber downwelling dalam proyek.***

## Penyelarasan Array

Bagian ini **hanya** muncul jika setidaknya satu gambar dalam proyek memiliki transformasi penyelarasan antar-modul yang ditandai oleh array LATTICE pada saat pengambilan gambar (tag XMP `Chloros:Alignment*`). Bagian ini menunjukkan berapa banyak gambar yang memiliki tag penyelarasan, kamera mana yang menjadi referensi (lambang `REF`), serta tabel jumlah gambar per kamera.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Terapkan penyelarasan array

* **Jenis**: Kotak centang
* **Default**: Diaktifkan (dicentang)
* **Disimpan sebagai**: `Processing → "Array alignment"`
* **Deskripsi**: Melakukan warping pada setiap produk yang telah diproses (debayered / pratinjau / radiance / reflektansi / indeks) ke dalam geometri referensi bersama array tersebut menggunakan transformasi yang dicatat pada saat pengambilan gambar. Nonaktif = mengekspor dalam geometri asli per-sensor.

### Potong ke tumpang tindih bersama

* **Jenis**: Kotak centang (hanya aktif saat *Terapkan penyelarasan array* diaktifkan)
* **Default**: Diaktifkan (dicentang)
* **Disimpan sebagai**: `Processing → "Array alignment crop"`
* **Deskripsi**: Memotong ekspor yang telah diselaraskan ke wilayah yang dimiliki bersama oleh semua modul kamera, sehingga setiap pita memiliki area cakupan yang sama. Nonaktif mempertahankan kanvas sensor penuh (diisi hitam di luar sumber).

### Resampling

* **Jenis**: Pilihan menu tarik-turun (hanya aktif saat *Terapkan penyelarasan array* diaktifkan)
* **Opsi**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Default**: Bilinear
* **Disimpan sebagai**: `Processing → "Array alignment interpolation"`
* **Deskripsi**: Interpolasi yang digunakan oleh warp penyelarasan. Nearest mempertahankan nilai sumber yang tepat (tanpa pencampuran antar-piksel) untuk analisis radiometrik yang ketat; Bilinear paling baik untuk pemetaan dan penggunaan visual.

Tiga opsi yang sama juga tersedia tanpa nama sebagai `chloros-cli process --array-alignment`, `--array-alignment-crop`, dan `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Indeks

Pengaturan ini memungkinkan Anda mengonfigurasi indeks multispektral untuk analisis dan visualisasi.

### Tambahkan indeks

* **Jenis**: Panel konfigurasi indeks khusus
* **Deskripsi**: Membuka panel interaktif tempat Anda dapat memilih dan mengonfigurasi indeks vegetasi multispektral (NDVI

, NDRE

, EVI

, dll.) untuk dihitung selama pemrosesan citra. Anda dapat menambahkan beberapa indeks, masing-masing dengan pengaturan visualisasinya sendiri.
* **Indeks yang tersedia**: Menu tarik-turun antarmuka pengguna (GUI) mencakup**27** rumus indeks multispektral yang telah ditentukan sebelumnya (lihat [Rumus Indeks Multispektral](multispectral-index-formulas.md) untuk daftar lengkapnya, termasuk nama-nama yang juga diterima oleh opsi CLI

/SDK

`--indices`).
* **Fitur**:
  * Pilih dari rumus indeks yang telah ditentukan sebelumnya
  * Seret saluran filter kamera Anda ke slot pita pada rumus
  * Konfigurasikan gradien warna visualisasi (LUT - Tabel Pencarian)
  * Tetapkan nilai ambang batas dan mode pemotongan
  * Buat rumus indeks khusus
* **Catatan**: Indeks tidak dihitung untuk kamera mono LATTICE M3M single-band — indeks multi-band tidak terdefinisi pada satu band. Survey3

dan LATTICE M3C tidak terpengaruh.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Setiap indeks yang Anda tambahkan menampilkan rumusnya sebagai persamaan matematika, dengan lingkaran berwarna per slot pita: merah = Red

, hijau = Green

, biru = Blue

, oranye = Orange

, cyan = Cyan

, ungu = NIR

, magenta = RE. Seret lingkaran dari baris di atas rumus ke slot untuk mengikatnya; klik dua kali slot yang terikat untuk menghapusnya. Indeks hanya dihitung jika setiap slot yang digunakan rumus tersebut memiliki kanal.

### Rumus Kustom (Fitur Chloros

+)

* **Jenis**: Kumpulan definisi rumus kustom
* **Ketersediaan**: Memerlukan masuk dengan langganan Chloros

+ yang memenuhi syarat.
* **Deskripsi**: Memungkinkan Anda membuat dan menyimpan rumus indeks multispektral kustom menggunakan perhitungan band. Rumus kustom disimpan bersama pengaturan proyek Anda dan dapat digunakan seperti indeks bawaan.
* **Cara membuat**:
  1. Di panel konfigurasi Indeks, buka kalkulator rumus kustom
  2. Tulis rumus menggunakan **simbol band-slot**, bukan nama band
  3. Simpan rumus dengan nama yang deskriptif — rumus tersebut kemudian akan muncul di bagian bawah menu tarik-turun rumus, dan Anda dapat menyeret lingkaran saluran kamera ke slotnya persis seperti preset bawaan
* **Sintaks rumus**:
  * Slot band: `x`, `y`, `z`, `a`, `b`, `c` — enam posisi yang Anda petakan ke saluran nyata dengan menyeret
  * Operator: `+`, `-`, `*`, `/`, `^`, dan `()` untuk pengelompokan
  * Fungsi: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Mengapa menggunakan simbol, bukan nama band**: rumus yang ditulis sebagai `(y-x)/(y+x)` berfungsi pada kamera apa pun, karena pemetaan seret-dan-lepas menentukan apakah `y` merupakan NIR

850 nm dari filter RGN

atau NIR

808 nm dari filter OCN

. preset bawaan disimpan dengan cara yang sama — lihat [Rumus Indeks Multispektral](multispectral-index-formulas.md) untuk bentuk simbol yang tepat dari ke-27 rumus tersebut.
* **Di mana rumus ini berfungsi**: rumus kustom disimpan bersama pengaturan proyek dan dapat digunakan di [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) serta dalam proses pengolahan. Rumus-rumus ini**tidak** didukung oleh daftar nama CLI

/SDK

`--indices`, yang hanya menampilkan 22 nama preset bawaan.

***

## Ekspor

Pengaturan ini mengontrol format dan kualitas gambar yang telah diproses dan diekspor.

### Format gambar yang dikalibrasi

* **Jenis**: Pilihan menu tarik-turun
* **Opsi**:
  * **TIFF

(16-bit)** - Format TIFF

16-bit tanpa kompresi
  * **TIFF

(8-bit)** - Format TIFF

8-bit tanpa kompresi
  * **TIFF

(32-bit, Persen)** - TIFF

32-bit floating-point dengan nilai reflektansi dalam bentuk persentase
  * **PNG

(8-bit)** - Format PNG

8-bit terkompresi
  * **JPG (8-bit)** - Format JPEG

8-bit yang terkompresi
* **Default**: TIFF

(16-bit); proyek yang seluruhnya 8-bit menampilkan TIFF

(8-bit), dan hanya menawarkan format 8-bit
* **Deskripsi**: Memilih format file untuk menyimpan gambar yang telah diproses dan dikalibrasi. Hasil ekspor disimpan dalam subfolder per-format di dalam folder masing-masing kamera (`tiff16`, `tiff32`, `png8`, `jpg8`), dengan satu folder `<Product>_Images/` per produk. File yang diekspor mempertahankan nama file aslinya — folder, bukan sufiks nama file, yang mengidentifikasi produk.
* **Rekomendasi format**:
  * **TIFF

(16-bit)**: Direkomendasikan untuk analisis ilmiah dan alur kerja profesional. Menjaga kualitas data maksimal tanpa artefak kompresi. Paling cocok untuk analisis multispektral dan pemrosesan lanjutan dalam perangkat lunak GIS.
  * **TIFF

(32-bit, Persen)**: Paling cocok untuk alur kerja yang memerlukan nilai reflektansi dalam bentuk persentase (0–100%). Menawarkan presisi maksimum untuk pengukuran radiometrik.
  * **PNG

(8-bit)**: Cocok untuk tampilan web dan visualisasi umum. Ukuran file lebih kecil dengan kompresi tanpa kehilangan data, namun rentang dinamis berkurang.
  * **JPG (8-bit)**: Ukuran file terkecil, paling cocok hanya untuk pratinjau dan tampilan web. Menggunakan kompresi lossy yang tidak cocok untuk analisis ilmiah.
* **Catatan**: Radian LATTICE selalu diekspor sebagai TIFF

float 32-bit terlepas dari pengaturan ini.

***

## Simpan Template Proyek

Fitur ini memungkinkan Anda menyimpan pengaturan proyek saat ini sebagai template yang dapat digunakan kembali.

* **Jenis**: Kotak teks + Tombol Simpan
* **Deskripsi**: Masukkan nama deskriptif untuk template pengaturan Anda dan klik ikon simpan. Template ini akan menyimpan semua pengaturan proyek Anda saat ini (deteksi target, opsi pemrosesan, indeks, dan format ekspor) agar mudah digunakan kembali pada proyek-proyek mendatang. Templat disimpan di folder `Project Templates/` di dalam folder penyimpanan proyek Anda, dan juga dapat dipilih atau diekspor dari menu utama (*Pilih Templat* / *Simpan Templat* / *Ekspor Templat*).
* **Contoh penggunaan**:
  * Buat templat untuk sistem kamera yang berbeda (RGB

, multispektral, NIR

)
  * Simpan konfigurasi standar untuk jenis tanaman tertentu atau alur kerja analisis
  * Bagikan pengaturan yang konsisten di seluruh tim
* **Cara menggunakan**:
  1. Konfigurasikan semua pengaturan proyek yang Anda inginkan
  2. Masukkan nama templat (misalnya, &quot;RedEdge

Survey3

NDVI

Standard&quot;)
  3. Klik ikon simpan
  4. Templat tersebut kini dapat dimuat saat membuat proyek baru

***

## Folder Penyimpanan Proyek

Pengaturan ini menentukan lokasi penyimpanan default untuk proyek baru.

* **Jenis**: Tampilan jalur direktori + tombol Edit
* **Default (WindowsWindows

)**: `Documents\MAPIR\Chloros Projects` di folder Dokumen Anda — biasanya `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, atau di mana pun OneDrive atau profil roaming telah memindahkan folder Dokumen
* **Default (Linux

)**: `~/Documents/MAPIR/Chloros Projects`
* **Deskripsi**: Menampilkan direktori default saat ini tempat proyek Chloros

baru dibuat. Klik ikon edit untuk memilih direktori yang berbeda. Pengaturan pengganti ini disimpan sebagai satu baris teks di `~/.chloros/working_directory.txt` — di Windows

, file tersebut berada di `C:\Users\<Username>\.chloros\working_directory.txt`. Jika berkas tersebut tidak ada, atau menunjuk ke jalur yang sudah tidak ada lagi, Chloros

akan kembali menggunakan pengaturan default di atas. Antarmuka Baris Perintah (CLI

) membaca dan menulis berkas yang sama, sehingga `chloros-cli` dan antarmuka grafis (GUI) selalu sepakat mengenai lokasi proyek.
* **Template Proyek** berada di subfolder `Project Templates/` di dalam direktori ini.
* **Kapan harus mengubah**:
  * Atur ke drive jaringan untuk kolaborasi tim
  * Ubah ke drive dengan ruang penyimpanan lebih besar untuk dataset besar
  * Atur proyek berdasarkan tahun, klien, atau jenis proyek dalam folder yang berbeda
* **Catatan**: Mengubah pengaturan ini hanya memengaruhi proyek BARU. Proyek yang sudah ada tetap berada di lokasi aslinya.
* **Panjang jalur Windows**: kecuali jika jalur panjang Windows

diaktifkan (opsi **Aktifkan jalur panjang Windows** pada penginstal), folder proyek yang panjangnya melebihi sekitar 130 karakter akan ditolak sebelum diproses, karena hasilnya akan melampaui batas jalur Windows

sebesar 260 karakter. Lihat [Memulai Pemrosesan](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## Penyimpanan Pengaturan

Proyek Chloros

adalah sebuah **folder**. Semua pengaturan proyek disimpan di dalam folder `project.json`; perangkat keras yang terhubung juga disimpan bersamanya di `cameras.json` dan `sensors.json`, sehingga saat membuka kembali proyek, kamera dan sensor cahayanya akan terhubung kembali. Saat Anda membuka kembali proyek, semua pengaturan akan dipulihkan persis seperti saat Anda meninggalkannya. Proyek yang disimpan juga dapat dijalankan tanpa antarmuka pengguna (headless) menggunakan `chloros-cli project` atau `open_project` dari SDK

.

### Hierarki Pengaturan

Pengaturan diterapkan dalam urutan berikut:

1. **Pengaturan default sistem** - Pengaturan default bawaan yang ditentukan oleh Chloros

2. **Pengaturan templat** - Jika Anda memuat templat saat membuat proyek
3. **Pengaturan proyek yang disimpan** - Pengaturan yang disimpan bersama berkas proyek
4. **Penyesuaian manual** - Setiap perubahan yang Anda lakukan selama sesi saat ini

### Pengaturan dan Pemrosesan Gambar

Pengaturan pemrosesan dibaca saat proses pemrosesan dimulai. Mengubah pengaturan tidak akan memengaruhi hasil yang sudah tersimpan di disk — jalankan pemrosesan kembali untuk menerapkan pengaturan baru. Beberapa pengaturan sama sekali tidak memengaruhi pemrosesan:

* Resolusi Gambar Mini (hanya untuk tampilan)
* Simpan Template Proyek
* Simpan Folder Proyek

***

## Referensi kunci konfigurasi

Untuk otomatisasi (CLI

`--config`, SDK

`configure`, atau membaca `project.json` secara langsung), berikut adalah kunci-kunci yang tepat di bawah `Project Settings`:

| Jalur kunci | Tipe | Default |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | angka 0-10000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | angka 0-100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standar |
| `Processing → Minimum recalibration interval` | angka 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | bilangan -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | ID profil batas atau `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | daftar konfigurasi indeks | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Kunci-kunci `Array alignment` ditulis saat bagian Array Alignment dirender untuk pertama kalinya atau saat panggilan otomatisasi menetapkannya. Selama kunci-kunci tersebut tidak ada, pipeline menggunakan nilai-nilai yang sama seperti yang ditunjukkan di atas (`true`, `true`, bilinear), sehingga proyek .json tanpa kunci-kunci tersebut berperilaku sama persis dengan yang memilikinya.

### Kunci yang disimpan di `project.json` tanpa kontrol di panel pengaturan

Kunci-kunci ini berada di bawah pohon `Project Settings` yang sama dan dibaca oleh proses, tetapi Anda tidak akan menemukan widget untuknya di bilah samping:

| Jalur kunci | Tipe | Default | Ditetapkan oleh |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK

`input_level=`. Mengganti cara interpretasi file TIFF

masukan LATTICE; `auto` menyimpulkan dari tag XMP setiap berkas `Chloros:ProcessingLevel` ditambah jumlah saluran. Diabaikan untuk pengambilan data `.raw` dari Survey3

. Secara sengaja bukan pengaturan GUI — pengaturan &quot;auto&quot; selalu benar dalam setiap kasus normal. |
| `Processing → Target reflectance dir` | string jalur | `""` | `chloros-cli process --target-reflectance-dir`, atau target proyek API

|
| `Processing → Target reflectance config` | kamus yang diindeks berdasarkan nomor seri kamera | `{}` | Mendaftarkan target dalam bingkai (mode `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | string jalur | `""` | SDK

`process_folder(daq_log_path=…)`. Menunjuk ke rekaman `.daq` atau folder yang berisi rekaman tersebut |
| `Target Detection → Minimum calibration target squares` | angka | `4` | Default lama; tanpa kontrol dan tanpa bendera CLI

|
| `UI → Grid thumbnail size` | angka | `160` | Penggeser zoom thumbnail milik kisi gambar |

Dua preferensi penampil disimpan **di tingkat atas dalam `project.json`**, sepenuhnya di luar `Project Settings`, karena keduanya merupakan status tampilan, bukan pengaturan pemrosesan:

| Jalur kunci | Tipe | Default | Ditetapkan oleh |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | bilangan bulat 1–256 | `1` | Kontrol GSD (px) pada tab gambar — lihat [Membuka Gambar dalam Layar Penuh](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Praktik Terbaik

1. **Mulailah dengan pengaturan default**: Pengaturan default berfungsi dengan baik untuk sebagian besar sistem kamera MAPIR

dan alur kerja umum.
2. **Buat templat**: Setelah Anda mengoptimalkan pengaturan untuk alur kerja atau kamera tertentu, simpan pengaturan tersebut sebagai templat untuk memastikan konsistensi di seluruh proyek.
3. **Uji coba sebelum pemrosesan penuh**: Saat bereksperimen dengan pengaturan baru, uji coba pada sebagian kecil gambar sebelum memproses seluruh dataset Anda.
4. **Dokumentasikan pengaturan Anda**: Gunakan nama templat yang deskriptif yang menunjukkan sistem kamera, jenis pemrosesan, dan tujuan penggunaannya (misalnya, &quot;Survey3

\_RGB\_NDVI\_Agriculture&quot;).
5. **Pemilihan format ekspor**: Pilih format ekspor berdasarkan tujuan akhir Anda:
   * Analisis ilmiah → TIFF (16-bit atau 32-bit)
   * Pemrosesan GIS → TIFF

(16-bit)
   * Visualisasi cepat → PNG

(8-bit)
   * Berbagi di web → JPG (8-bit)

***

Untuk informasi lebih lanjut mengenai indeks multispektral di Chloros

, lihat halaman [Rumus Indeks Multispektral](multispectral-index-formulas.md).

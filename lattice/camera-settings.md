# Pengaturan Kamera

Tab **Kamera** adalah panel kontrol langsung Chloros

untuk kamera LATTICE: area umpan utama yang menampilkan setiap kamera yang terhubung sebagai ubin langsung, serta bilah samping yang dapat digeser di antara tiga halaman — **daftar kamera**,**panel pengaturan**(pengaturan per kamera, array, atau pengambilan — satu per satu), serta**Kalkulator Indeks**. Halaman ini mendokumentasikan setiap kontrol dalam daftar kamera, panel pengaturan per kamera, dan panel pengaturan array. Mode pengambilan, pemilihan jenis ekspor, dan alur Capture All terdapat di halaman terkait [Pengaturan &amp; Mode Perekaman](capture.md).

Tab &quot;Kamera&quot; muncul di bilah samping setelah backend Chloros

siap. Semua kontrol di bawah ini berkomunikasi dengan backend lokal melalui `127.0.0.1:5000`; perubahan akan langsung diterapkan pada tampilan langsung kamera kecuali disebutkan lain.

## Jenis kamera yang digunakan di halaman ini

Kontrol akan muncul atau tersembunyi tergantung pada jenis kamera yang dipilih. Panduan ini menggunakan istilah-istilah berikut secara konsisten:

| Istilah | Arti | Saluran filter |
| --- | --- | --- |
| **Kamera RGB** | LATTICE M3C dengan filter FRGB (model berisi `-FRGB`) | Red

/ Green

/ Blue

|
| **Multispektral Bayer** | LATTICE M3C dengan FRGN, FOCN, atau FNGB | FRGN: Red

/ Green

/ NIR

· FOCN: Orange

/ Cyan

/ NIR

· FNGB: NIR

/ Green

/ Blue

|
| **Mono (M3M)** | LATTICE M3M — satu filter pita sempit, satu pita yang dikalibrasi | Pita tunggal |
| **Anggota array** | Kamera yang terhubung sebagai bagian dari array yang disinkronkan (tampilan gabungan atau terpisah) | Sesuai filternya |

Kamera RGB

mendapatkan pemrosesan fotometrik (keseimbangan putih, profil warna, gamma); kamera multispektral dan mono mendapatkan rantai radiometrik dan melewati pengaturan fotometrik. Anggota array mengelola pengaturan tingkat aliran (format piksel, resolusi, binning, pemicu, laju bingkai) ke array — baris-baris tersebut menjadi hanya-baca di panel per-kamera dan dipindahkan ke panel pengaturan array.

## Area umpan utama



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Jika tidak ada kamera yang terhubung, area umpan menampilkan pesan pembuka **&quot;Hubungkan kamera untuk memulai&quot;**dengan dua tombol:**Hubungkan Kamera**(hijau, membuka dialog koneksi kamera tunggal) dan**Hubungkan Array** (biru, membuka dialog koneksi array). Dialog koneksi tersebut didokumentasikan dalam [Menghubungkan Kamera](connecting.md); konsep array (sinkronisasi, tingkatan, bandwidth) dalam [Array Multi-Kamera](arrays.md). Saat Anda membuka proyek yang telah disimpan dan berisi kamera, layar pembuka akan menampilkan ikon spinner dengan tulisan &quot;Membuka kembali N kamera yang disimpan…&quot; sementara Chloros

memulihkan aliran video dari sesi terakhir.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Bilah atas

| Kontrol | Fungsinya |
| --- | --- |
| **Tombol beralih mode tampilan**| Beralih antara**tampilan kisi**(semua ubin sebagai sel) dan**tampilan daftar**(baris lebar penuh: setiap susunan gabungan sebagai satu baris gabungan, setiap anggota susunan**terpisah** sebagai baris tersendiri dengan umpan dan tabel Target Kalibrasi masing-masing, lalu SATU kamera mandiri — yang aktif — di bawahnya). Keterangan: &quot;Beralih ke tampilan kisi&quot; / &quot;Beralih ke tampilan daftar&quot;. |
| **Kunci kisi**(gembok) | Secara default**terkunci** — ubin tetap pada posisinya. Buka kunci untuk menyeret dan mengatur ulang ubin ke slot mana pun (jarak antar ubin tetap terjaga). Kisi akan terkunci kembali secara otomatis setiap kali kamera baru terhubung. Keterangan: &quot;Buka kunci kisi (aktifkan seret ubin)&quot; / &quot;Kunci kisi (bekukan ubin di tempatnya)&quot;. |
| Penggeser **Zoom Umpan** | Ukuran ubin, mulai dari 60 px hingga lebar wadah penuh. Sel mempertahankan rasio aspek 4:3. Di bawah lebar sel 200 px, overlay nama dan fps akan tersembunyi agar ubin tetap rapi. |

### Ubin umpan

Setiap kamera menampilkan ubin langsung gabungan; sebuah kamera juga dapat menampilkan tiga ubin **pemisahan per saluran** skala abu-abu (lihat [Pemisahan Saluran](#display-overlays-drawn-over-the-live-feed)), sedangkan array menampilkan ubin gabungan. Ubin aktif menampilkan cincin seleksi dengan warna kamera (atau array) tersebut.

Saat mengarahkan kursor ke ubin, tombol tutup **X** akan muncul:

* Menutup ubin **gabungan** sementara pemisahan salurannya tetap aktif hanya akan menyembunyikan ubin gabungan tersebut.
* Menutup **ubin terakhir yang terlihat dari kamera mandiri** akan memutuskan koneksi kamera tersebut.
* **Ubin pemisahan anggota array gabungan tidak pernah memutuskan koneksi** kamera — ubin tersebut hanya disembunyikan.

Dengan kisi yang tidak terkunci, seret ubin mana pun ke slot mana pun; tata letak akan disimpan bersama proyek.

## Bilah Samping — daftar

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamera Halaman pertama bilah samping mencantumkan setiap kamera dan array yang terhubung:

* **Hubungkan Kamera**(hijau) /**Hubungkan Array** (biru, menampilkan &quot;Mendeteksi...&quot; saat memindai). Keduanya dinonaktifkan saat dialog penghubungan terbuka.
* **Rekam Semua** (merah) — merekam setiap kamera yang terdaftar dengan jenis ekspor yang dipilih di Pengaturan Perekaman. Membutuhkan proyek yang terbuka. Didokumentasikan secara lengkap di [Pengaturan &amp; Mode Pengambilan](capture.md).
* **Gigi Pengaturan Pengambilan** (di sebelah Ambil Semua) — membuka [panel Pengaturan Pengambilan](capture.md#the-capture-settings-pane). Dinonaktifkan jika tidak ada proyek atau saat sedang merekam.

### Baris Kamera

Setiap baris kamera menampilkan batas berwarna (warna khusus kamera), label &quot;CAM&quot; — dengan huruf peran **M**(master) atau huruf peran**S** (slave) berwarna hijau untuk anggota array — serta nama tampilan. Nama defaultnya adalah `LATTICE-MODEL (serial)`; ganti namanya dari panel pengaturan per kamera. Tombol baris:

| Tombol | Efek |
| --- | --- |
| **Mata**| Mengaktifkan/menonaktifkan tampilan. Kamera yang disembunyikan akan keluar dari kisi dan**tidak termasuk dalam Capture All**. |
| **Gear** | Buka panel pengaturan per-kamera (bagian berikutnya). |
| **Jeda / Putar**| Membekukan pratinjau langsung**hanya di sisi tampilan** — proses pengambilan di backend tetap berjalan. Kamera yang dijeda tidak dapat merekam. |
| **X** | Putus koneksi. Antarmuka pengguna diperbarui seketika (asumsi terbaik); proses pemutusan koneksi di backend sendiri bisa memakan waktu 10–30 detik untuk selesai. |

### Baris array

Sebuah baris array menampilkan lencana &quot;ARRAY&quot; dengan warna array tersebut, nama array (dapat diganti di pengaturan array), dan tombol **DAQ · on/off**—**on** ketika Sensor Cahaya tingkat array diaktifkan *atau* salah satu anggotanya memiliki sensor per-kamera; tooltip-nya mencantumkan secara tepat sensor mana yang menyalurkan data ke mana. Kamera-kamera anggota terdaftar dengan indentasi di bawahnya dalam baris masing-masing. Tombol-tombol baris array: **mata**(menyembunyikan/menampilkan SEMUA anggota sekaligus),**roda gigi**(panel pengaturan array),**X**(memutuskan koneksi seluruh array).

Status sensor cahaya (DLS) yang digunakan di baris array dan panel pengaturan array memiliki empat keadaan:**off**,**menunggu**(belum ada spektrum),**aktif**(spektrum diterima dalam 3 detik terakhir), dan**kadaluwarsa** — tidak ada spektrum baru dalam 3 detik, tetapi pembacaan terakhir *masih digunakan* (pembacaan DAQ tidak pernah kedaluwarsa pada jalur penangkapan).

Sampai Anda menyusun ulang urutannya, daftar tersebut akan menampilkan array terlebih dahulu, kemudian kamera mandiri, dengan kamera (dan setiap anggota array) dalam **urutan kamera**: M3C (Bayer) sebelum M3M (mono); di antara M3C, FRGB, FRGN, FOCN, lalu FNGB; M3M berdasarkan panjang gelombang filter, dimulai dari yang terendah. Tata letak default kisi mengikuti urutan yang sama. Anda dapat menyeret kamera mandiri dan seluruh kelompok array melewati satu sama lain di bilah samping untuk mengatur ulang urutan daftar; anggota array tidak dapat diseret secara terpisah.

## Panel pengaturan per-kamera

Buka dengan mengklik ikon **roda gigi** pada baris kamera. Panel ini akan muncul di atas daftar kamera.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Header**:**swatch warna**kamera (klik untuk membuka pemilih warna bawaan — mengatur warna batas bilah samping dan cincin pemilihan ubin),**nama**dengan tombol pensil**Ubah Nama**(menyimpan nama kosong akan mengembalikan ke default `MODEL (serial)`), dan**×** untuk menutup.

### Histogram langsung

Di bagian atas panel terdapat histogram luma langsung yang dihitung dari pratinjau JPEG

pada ~8 Hz. Nilai rata-ratanya diberi bobot Bayer — (R+2G+B)/4 — agar sesuai dengan pengukuran AE kamera itu sendiri.

* **Garis putus-putus Orange**= target AE. **Seret secara horizontal untuk mengubah target** — satu perintah dikirim saat dilepas, dan menyeret akan mengubah mode target AE menjadi Manual.
* **Garis solid Green** = nilai rata-rata luma aktual (hasil yang saat ini diberikan oleh AE).
* **Tombol RGB** (kanan atas): mengaktifkan/menonaktifkan histogram overlay per-band yang diwarnai sesuai filter kamera (misalnya pada FRGN: abu-abu untuk NIR

, hijau, merah). Pada kamera mono (M3M), tombol ini bertuliskan &quot;MONO&quot; dan dinonaktifkan — mode mono selalu menampilkan histogram luma satu band.
* Label sumbu X mengikuti kedalaman bit sensor dari format piksel saat ini: 0..255, 0..1023, 0..4095, atau 0..65535.

### Baris

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

informasi kamera

| Baris | Perilaku |
| --- | --- |
| **Model** | Hanya baca (mis. `LATT-M3C-L87-FRGN`). |
| **Kalibrasi Radiometrik**| Lencana**&quot;Aktif&quot;** berwarna Green

dengan keterangan yang menampilkan tingkatan kalibrasi, hash, tanggal kalibrasi, dan daftar band, yang dimuat dari paket kalibrasi kamera (lihat [Kalibrasi Radiometrik Pabrik](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)). **Tersembunyi untuk kamera RGB** — kamera ini menggunakan kalibrasi keseimbangan putih fotometrik, bukan radian per-pita. |
| **Laporan Kalibrasi**| Tombol**Unduh** — membuka PDF sertifikat kalibrasi NIST per nomor seri kamera di penampil sistem operasi Anda. Jika sertifikat belum disimpan dalam cache, Chloros

akan menampilkan petunjuk sebagai gantinya. |
| **Nomor Seri** | Hanya baca. |
| **Firmware**| Menampilkan versi saat ini, lalu menentukan versi yang tersedia untuk model ini (disimpan dalam cache per model — susunan N kamera memeriksa server satu kali). Status: &quot;Memeriksa…&quot; → tombol**&quot;Perbarui ke X&quot;**→ &quot;Sedang mem-flash…&quot; → &quot;Diperbarui dari A ke B&quot; / &quot;Gagal: …&quot; / &quot;Dilewati: …&quot; / hijau**&quot;Sudah terbaru&quot;**. Keterangan tombol pembaruan: &quot;Reset pabrik + pembaruan firmware + pemrograman ulang UserSet1. ~2–3 menit; jangan putuskan koneksi.&quot; |
| **IP** | Hanya baca. |
| **Suhu** | Hanya baca, diperbarui setiap 3 detik. Berubah menjadi oranye pada ≥65 °C dan merah dengan tanda ⚠ pada ≥75 °C. |
| **Target Kalibrasi** kotak centang | Mengaktifkan deteksi target reflektansi ArUco dengan tabel validasi NDVI

per panel di bawah umpan langsung (tampilan daftar) — lihat [Tabel Target Kalibrasi](#the-calibration-target-table). Hanya untuk sesi — selalu terbuka dalam keadaan nonaktif. |
| Menu tarik-turun **Sensor Cahaya** | Menghubungkan sensor cahaya DAQ (DAQ-E/M/U, dari daftar tab Sensor Cahaya) ke kamera ini untuk koreksi pencahayaan cahaya turun (DLS) dan eksposur otomatis prediktif. &quot;&quot;Tidak Ada&quot; menghapus pengikatan. Jika tidak ada sensor yang terhubung, menu tarik-turun akan menampilkan &quot;(tidak ada sensor yang terhubung — buka tab DAQ)&quot;. Pengikatan ini disimpan bersama proyek. |

#### Tabel Target Kalibrasi

Tabel ini **hanya menampilkan hasil validasi** (&quot;hanya validasi — tidak mengubah kalibrasi&quot;). Judul kolomnya mencantumkan nomor seri kamera, filter, nomor target, dan ukuran penanda dalam piksel, serta memiliki satu baris per panel:

* **Panel**, kemudian satu kolom per pita dengan nilai panel tersebut — kamera Bayer menampilkan tiga pita berdasarkan nama dan panjang gelombang (mis. FRGN: Red

660 nm / Green

550 nm / NIR

850 nm); kamera **mono (M3M) menampilkan satu pita** (mis. `F750`) dan tidak memiliki kolom NDVI

.
* **Sat** — persentase piksel panel yang jenuh (kuning mulai dari 1 %, merah mulai dari 5 %).
* **NDVI

Terukur**,**NDVI

Teoritis** (NDVI

reflektansi referensi panel) dan **Δ**— hanya untuk kamera multispektral Bayer. Tanda kecil setelah &quot;NDVI

Terukur&quot; menunjukkan dari mana NDVI

terukur dihitung:**ρ**(reflektansi — pembacaan sensor cahaya tersedia),**L**(radian — tidak ada pembacaan sensor cahaya) atau**DN**(nilai mentah kamera). Hanya**ρ** NDVI

yang dapat dibandingkan dengan NDVI

Theo: untuk L dan DN, kolom Δ dinonaktifkan dan terdapat catatan yang menyatakan hal tersebut (&quot;Tidak ada pembacaan sensor cahaya: NDVI

berasal dari radian, sehingga Δ bukan pemeriksaan kalibrasi&quot; / &quot;NDVI

berasal dari nilai mentah kamera, sehingga Δ bukan pemeriksaan kalibrasi&quot;). Untuk ρ, Δ berwarna hijau jika nilainya di bawah 0,02, kuning jika di bawah 0,05, dan merah jika sebaliknya.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Eksposur &amp; Penguatan

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Semua masukan numerik di sini menggunakan spinner yang dipercepat saat ditahan: ketuk = ±1, tahan &gt;1,5 detik = ±10, tahan &gt;3 detik = ±100. Nilai tersebut dikirim ke kamera saat Anda melepaskannya.

| Kontrol | Rentang / pilihan | Default | Berlaku untuk | Fungsinya |
| --- | --- | --- | --- | --- |
| **Pencahayaan (us)**| Nilai min/maks real-time kamera | Otomatis | Semua | Waktu pencahayaan dalam mikrodetik, dengan tombol**Otomatis/Manual**. Otomatis = eksposur otomatis berkelanjutan dari sisi kamera. Mengatur eksposur manual akan membiarkan**Gain** tetap pada Otomatis jika sebelumnya demikian, sehingga Anda bebas menggunakan seluruh rentang gain kamera, hingga Anda mengedit gain atau mengaturnya ke Manual. |
| **Gain (dB)**| Nilai minimum/maksimum langsung kamera (mis. hingga 48 dB) | Manual (mati) | Semua | Penguatan analog/digital dengan tombol pengalih**Otomatis/Manual** tersendiri. |
| **Kecerahan Target AE**| 0–255 | 80, mode**Otomatis**| Semua (dapat diedit saat AE atau penguatan otomatis aktif) | Kecerahan yang menjadi target AE. Dalam**Otomatis**(pengaturan default), pengontrol backend berbasis histogram akan memilih target tersebut sendiri, menjaga eksposur pada 60–75% dari maksimum sensor. Mengetikkan nilai atau menyeret garis oranye pada histogram akan mengubahnya ke**Manual**. |
| **AE Smoothing** | 0,5–40, langkah 0,1 | 8,0 | Semua | Peredaman AE. Keterangan: &quot;Semakin rendah = AE bereaksi lebih cepat (dapat berkedip pada fps tinggi). Semakin tinggi = lebih halus / lebih lambat.&quot; Nilai yang jauh di bawah default dapat menyebabkan AE berkedip dan mengganggu stabilitas streaming pada frame rate tinggi; 8,0 adalah nilai default yang stabil. |
| **AE Region of Interest**| Kotak centang Aktifkan + tombol**Aim**| Nonaktif | Semua | Saat diaktifkan, AE hanya mengukur wilayah bergaris putus-putus hijau, bukan seluruh bingkai.**Aim**mengaktifkan fitur klik-untuk-menempatkan pada siaran langsung: satu klik akan memusatkan wilayah pada 30% dari bingkai; klik-seret akan menggambar persegi panjang kustom (minimal 5% × 5%). Fitur**Aim** akan nonaktif secara otomatis setelah satu penempatan. Wilayah tersebut dipetakan kembali ke koordinat asli kamera sesuai rotasi/cermin yang Anda atur, dan disimpan bersama proyek. |
| **Kecepatan Penyesuaian AE** | 0,1–5, langkah 0,1 | 1,0 | Khusus anggota array | Seberapa cepat target AE otomatis melacak perubahan kecerahan adegan; 1,0× memeriksa ulang setiap 2,5 detik. |
| **Perlindungan Sorotan** | Ketat (1 %) / Normal (5 %) / Longgar (15 %) | Ketat | Kamera yang menampilkan pengaturan ini | Seberapa banyak bagian bingkai yang boleh terpotong menjadi putih sebelum AE menggelapkan gambar. |

{% hint style="info" %}
**Persyaratan cahaya untuk kamera multispektral Bayer (RGN

/ OCN

/ NGB

):** adegan harus memiliki cahaya yang cukup di ketiga kanal atau kalibrasi tidak akan berfungsi dengan benar — satu eksposur sensor mencakup ketiga spektrum tersebut. Gunakan sensor cahaya DAQ untuk mengukur cahaya Anda, atau gunakan mode all-mono (M3M) agar setiap band mendapatkan eksposurnya sendiri. Jika pengambilan gambar melanggar ketentuan ini, Chloros

akan mendeteksinya dan memberi peringatan kepada Anda (notifikasi unmix-clamp). Secara langsung, ubin pemisahan saluran akan menampilkan **&quot;Data saluran dengan tingkat kepercayaan rendah · N%&quot;** ketika unmix membatasi saluran tersebut pada 25% atau lebih dari pikselnya.
{% endhint %}

{% hint style="info" %}
**Bagaimana pengaturan eksposur otomatis mengatur eksposur dan gain (1.3.0).** Dengan Gain pada mode Otomatis, Chloros

mempertahankan gain pada 0 dB hingga eksposur mencapai nilai terpanjang yang diizinkan (batas atas AE, periode pemicu array, atau batas atas eksposur DAQ); baru setelah itu gain dinaikkan. Dalam sebuah array, laju pemicu menjadi prioritas utama: auto-exposure mengisi periode yang ditentukan oleh laju tersebut, lalu menaikkan gain, sehingga pemandangan yang semakin gelap tidak lagi menurunkan laju pemicu Otomatis (pada versi 1.2.0 hal itu terjadi). Eksposur **manual**pada anggota array yang durasinya lebih lama dari periode pemicu akan memperlambat laju pemicu array agar sesuai dengannya; laju pemicu otomatis akan kembali normal setelah setiap anggota yang memperlambatnya kembali ke eksposur otomatis. Dengan**Eksposur**dan**Kecerahan Target AE** keduanya dalam mode Otomatis serta sensor cahaya DAQ yang memberikan pembacaan, eksposur setiap kamera dipertahankan setelah eksposur otomatis stabil, selama cahaya yang diukur tetap berada dalam kisaran ±25 %; perubahan cahaya yang lebih besar dan berkelanjutan, pengaturan yang dimulai ulang (misalnya perubahan ROI), atau perubahan eksposur yang Anda lakukan sendiri akan mengaktifkannya kembali.
{% endhint %}

{% hint style="info" %}
**Bagaimana perilaku eksposur otomatis (1.3.0).**

* Eksposur manual akan mempertahankan gain pada mode Otomatis hingga Anda mengubah gain.
* Sorotan kecil tidak menggelapkan adegan: sorotan spekular hingga 1% dari bingkai diabaikan, dan pada panel target kalibrasi yang diketahui, hingga 0,5% dari satu panel.
* Batas atas eksposur DAQ hanya berlaku saat target kalibrasi terlihat; jika tidak ada panel yang terlihat, kamera menyesuaikan eksposur sesuai adegan.
* Eksposur tetap stabil pada cahaya konstan dan stabil dalam beberapa detik setelah perubahan besar. Klip tidak lagi membatasi eksposur secara permanen, dan perubahan ROI akan menyesuaikan eksposur kembali.
* Target kecerahan tidak melampaui jangkauan kamera, sehingga cahaya yang kembali tidak menyebabkan flare pada bingkai.
* Kamera yang eksposur otomatisnya berhenti merespons di bawah sinar matahari terik akan dipertahankan pada tingkat sesuai cahaya di sekitarnya, yang dibaca oleh sensor cahaya atau, jika tidak terhubung, oleh bingkai kamera itu sendiri, dan dilepaskan saat cahaya berubah. Sensor cahaya yang gelap atau tertutup tidak mengganggu hal ini.
{% endhint %}

### Format Piksel &amp; Resolusi

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Anggota array** menampilkan baris &quot;Saat Ini&quot; (format + WxH) dan &quot;Binning&quot; yang hanya dapat dibaca dengan catatan &quot;Ditetapkan di pengaturan array&quot; — memulai ulang aliran pada satu anggota akan merusak sinkronisasi, sehingga hal ini dikelola di [panel pengaturan array](#array-settings-pane).**Kamera mandiri** memiliki:

| Kontrol | Pilihan | Fungsinya |
| --- | --- | --- |
| **Format Piksel** | 8-bit / 10-bit / 12-bit / 16-bit | Kedalaman bit sensor; keluarga formatnya mengikuti sensor. Standarnya adalah 12-bit, dan sebuah proyek akan menyimpan pengaturan yang Anda tentukan, termasuk 8-bit, serta akan terhubung kembali sesuai pengaturan tersebut. |
| **Resolusi** | Penuh / Setengah / Seperempat | Relatif terhadap penggabungan piksel saat ini: Penuh = 2048/N × 1536/N untuk penggabungan N×N. |
| **Binning** | 1x1 (tidak ada) / 2x2 / 4x4 | Binning N×N berbasis perangkat keras — nilai yang lebih besar menurunkan resolusi tetapi meningkatkan SNR dan laju bingkai. Mengubahnya akan memulai ulang aliran dan mengatur ulang ROI apa pun ke bidang pandang penuh yang baru. |
| **Saat Ini** | hanya baca | Ukuran WxH aktual dan offset (x, y) yang berlaku. |

### Pratinjau Langsung

Semua yang ada di bagian ini **hanya berlaku di sisi tampilan**— ini mengubah apa yang Anda lihat di siaran langsung, sementara tangkapan yang disimpan tetap linier dan tidak dimodifikasi — dengan satu pengecualian:**Vignette** bersifat radiometrik dan juga memengaruhi ekspor (seperti yang dijelaskan di bawah).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Kontrol | Rentang / pilihan | Default | Berlaku untuk | Fungsinya |
| --- | --- | --- | --- | --- |
| **Resolusi render** | 360p (tercepat) / 480p / 720p / 1080p / Resolusi sensor asli (paling lambat) | 720p | Semua | Tinggi di mana backend menjalankan rantai pratinjau radiometrik. Nilai yang lebih rendah meningkatkan laju bingkai tanpa mengubah bidang pandang. |
| **Indeks**| Kotak centang Aktifkan + roda gigi | Nonaktif | Hanya multispektral Bayer,**bukan** anggota array gabungan | Pratinjau indeks vegetasi langsung. Ikon roda gigi membuka [Kalkulator Indeks](#index-calculator-pane) bersama yang telah dimuat sebelumnya dengan pita alami filter kamera (mis. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Ekspresi khusus ditambah LUT (aktif/nonaktif, tingkat default 7-stop, min default 0,2, maks default 1) dihitung pada setiap bingkai pratinjau. Anggota array gabungan menyembunyikan baris ini — array tersebut memiliki satu indeks bersama. |
| **Keseimbangan Putih** | Mati / Sekali / Berkelanjutan + tombol pengambilan ulang | Berkelanjutan | Hanya RGB

| Keseimbangan putih langsung. Tombol refresh mengambil ulang keseimbangan putih dari spektrum DLS saat ini (dinonaktifkan saat mode dalam posisi Mati). |
| **Gamma** | Aktif / Nonaktif | Aktif | Hanya RGB

| Menampilkan gamma (γ = 2,2 LUT) pada pratinjau langsung. Hasil tangkapan yang disimpan tetap linier. |
| **Penghilang Bising** | Kotak centang + kekuatan 0–100 | Mati / 50 | Semua (per kamera, bahkan di dalam array) | Filter bilateral pada pratinjau langsung. Nilai lebih tinggi = hasil lebih halus tetapi detail lebih lembut. |
| **Ketajaman** | Kotak centang + kekuatan 0–100 | Mati / 30 | Semua | Masker ketajaman pada pratinjau langsung, diterapkan terakhir. Dapat memperkuat noise. Hanya untuk pratinjau. |
| **Vignette**| Kotak centang + intensitas 0–100 | Nonaktif / 0 | Semua | Penghilangan vignette sisa secara manual (mencerahkan sudut-sudut), ditumpangkan di atas perkiraan Smart Vignette dari array.**Radiometrik — memengaruhi tampilan langsung DAN ekspor**, tidak seperti Denoise/Sharpness. |
| **Profil Warna** | Raw / Linear / Natural / Enhanced / Suhu Kustom | Natural | Hanya RGB

| Lihat di bawah. |
| **Suhu Warna** | 2000–10000 K, langkah 100 | 5500 K | Hanya RGB

, profil Suhu Kustom | Menetapkan keseimbangan putih ke suhu warna terkorelasi tetap (masukan DLS diabaikan). Nilai Kelvin terakhir yang dipilih akan diingat saat berpindah profil. |
| **Sorotan** | −100 hingga 0, langkah 1 | 0 (mati) | Hanya RGB

| Baru di versi 1.3.0. Megelapkan area paling terang dan paling putih pada gambar (0 = tidak berubah, −100 = paling kuat). Berlaku untuk pratinjau langsung **dan tangkapan pada tingkat tampilan (Pratinjau)**, yang merekam pengaturan tersebut (`Chloros:DisplayHighlights`); ekspor radian dan reflektansi tidak pernah terpengaruh. Fitur ini tidak dapat memulihkan data sensor yang terpotong — kurangi eksposur untuk mengatasi hal tersebut. Overlay zebra mengikuti gambar yang telah digelapkan. Dinonaktifkan dengan profil warna Raw (&quot;Tidak diterapkan dengan profil warna Raw.&quot;). |
| **Saturasi** | 0–200 (100 = netral) | 100 | Hanya RGB

| Saturasi HSV pada pratinjau langsung. |
| **Kontras** | 0–200 (100 = netral) | 100 | Hanya RGB

| Kontras linier di sekitar abu-abu tengah pada pratinjau langsung. |
| **Cerminkan H / Cerminkan V** | Kotak centang | Nonaktif | Semua | Membalik pratinjau secara horizontal / vertikal. |
| **Rotasi**| 0° / 90° / 180° / 270° | 0° | Semua | Memutar pratinjau. Orientasi diterapkan pada akhir rantai pratinjau backend.**Tangkapan yang disimpan (1.3.0):**produk Preview, Radiance, Reflectance, dan Index dari kamera mandiri diputar oleh Mirror/Rotation, sedangkan**Raw dan Debayered tetap dalam orientasi asli kamera**; setiap file tangkapan merekam pengaturan tersebut dalam XMP-nya, sehingga saat memproses file raw nanti, pengaturan tersebut diterapkan dengan cara yang sama.**File anggota array tetap dalam orientasi asli** (pengaturan direkam, tidak diterapkan), dan tampilan komposit array mengabaikannya. |**Semantik profil warna** (kamera RGB

):

* **Raw** — melewati rantai pemrosesan sepenuhnya.
* **Linear** — sinyal gelap + flat-field + keseimbangan putih; tanpa matriks warna, tanpa gamma.
* **Natural** *(default)* — linear ditambah matriks koreksi warna yang diukur dan kurva nada yang beradaptasi dengan pemandangan.
* **Enhanced**— Natural ditambah vibrance dan kontras lokal CLAHE. Biaya tambahan ini hanya berlaku untuk**pratinjau langsung** — hasil jepretan yang disimpan selalu mendapatkan hasil akhir penuh terlepas dari profil yang digunakan.
* **Suhu Kustom** — Natural dengan keseimbangan putih dikunci pada nilai Kelvin yang Anda pilih.

{% hint style="warning" %}
Untuk Natural, Enhanced, dan Custom Temperature, panel menampilkan catatan nada: bingkai dipercerah sesuai adegan masing-masing, sehingga gambar *tampilan* yang disimpan tidak dapat dibandingkan antar-bingkai. **Ekspor radian atau reflektansi untuk pengukuran.**
{% endhint %}

### Overlay tampilan (digambar di atas siaran langsung)

Fitur ini hanya berlaku di antarmuka pengguna — digambar di atas video, tidak pernah menyentuh siaran langsung atau rekaman.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlay | Kontrol | Default | Fungsinya |
| --- | --- | --- | --- |
| **Zebra** | Kotak centang + ambang batas 200–255 | Nonaktif / 250 | Garis-garis diagonal berwarna magenta pada piksel yang terpotong. |
| **Tanda silang** | Kotak centang | Nonaktif | Tanda pusat bingkai. |
| **Grid** | Nonaktif / 3 × 3 / 9 × 9 | Nonaktif | Kisi komposisi. |
| **Histogram** | Kotak centang + lebar 0,10–0,90 bingkai | Nonaktif / 0,25 | Garis histogram pada siaran langsung. |
| **Focus Peak** | Kotak centang + ambang batas 20–200 + contoh warna | Nonaktif / 80 / `#ff5722` | Penyorotan tepi Sobel untuk pemfokusan. |
| **Pemisahan Saluran** | &quot;Tampilkan Pemisahan (Red

/ Green

/ NIR

)&quot; / tombol &quot;Sembunyikan Pembagian&quot; | Tersembunyi | Menambahkan tiga ubin skala abu-abu independen per saluran di samping gambar gabungan (label tombol mengikuti saluran filter kamera). Setiap ubin pembagian dapat diseret dan memiliki warna batas yang sama dengan kamera. Tidak tersedia pada kamera monokrom. Disimpan bersama proyek. |

### Pengukur Titik

* Kotak centang **Klik untuk Mengambil Sampel**: klik umpan langsung untuk mengambil sampel satu piksel (ditandai dengan retikel berbentuk salib), atau klik dan seret suatu area untuk mendapatkan rata-rata piksel.**Hapus**menghapus sampel dan retikel. Tidak dapat digunakan bersamaan dengan mode**Aim** AE-ROI.
* **Menu tarik-turun**Tampilkan**:**Raw (kedalaman bit)**— angka digital asli pada kedalaman bit sensor (mis. 12-bit → 0..4095) — atau**Tampilan (8-bit)** (default). Saat indeks langsung aktif, Display akan menampilkan nilai indeks yang dihitung (mis. NDVI

) sebagai gantinya.
* Panel pembacaan mencantumkan koordinat piksel, ukuran bingkai, format piksel, kedalaman bit, dan tabel saluran (Chan / Nilai / %) dengan label pita dan panjang gelombang; pasangan hijau Bayer dirata-ratakan; sampel wilayah menampilkan &quot;N px avg&quot;.

Status pengukur titik hanya berlaku selama sesi.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Eksposur Otomatis Prediktif (didorong oleh DLS)

Bagian ini hanya muncul jika **setidaknya satu sensor cahaya DAQ terhubung** — pemecah masalah memerlukan spektrum cahaya yang turun secara langsung untuk menggerakkannya.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Kontrol | Rentang | Default | Fungsinya |
| --- | --- | --- | --- |
| **Aktifkan** | Kotak centang | Aktif (kamera mandiri) | Solver bentuk tertutup menggunakan spektrum DLS ditambah skalar paket kalibrasi kamera untuk menempatkan rentang kecerahan tertinggi mendekati saturasi sambil menjaga rentang kecerahan terendah di atas batas bawah SNR — satu penulisan eksposur per pemecahan, tanpa loop penyetelan. Dirancang untuk timelapse bertenaga surya di mana setiap bidikan harus diekspos dengan benar. Backend secara otomatis beralih ke AE reaktif setiap kali pembacaan DLS kedaluwarsa/hilang atau paket kalibrasi tidak dimuat. |
| **Pengehalusan (α)** | 0,05–1,0, langkah 0,05 | 0,3 | Pengehalusan solusi prediktif berturut-turut (semakin rendah = semakin halus). |
| **Reflektansi adegan**| Tombol**Kalibrasi Ulang ρ** | — | Memperkirakan ulang faktor reflektansi adegan yang digunakan pemecah masalah. |

{% hint style="info" %}
**Fitur &#x27;Array connect&#x27; menonaktifkan AE prediktif secara default** — untuk array, AE cerdas Chloros

ditambah pengaturan eksposur otomatis di sisi kamera menangani eksposur (dengan perlindungan saturasi), dan perkiraan reflektansi adegan tunggal dari AE prediktiftidak aman untuk adegan campuran. Anda dapat mengaktifkannya kembali per kamera di sini jika secara khusus menginginkan eksposur radiometrik yang digerakkan oleh DLS.
{% endhint %}

**Batas eksposur yang digerakkan oleh DAQ dan AE yang dipatok pada insiden.** Terlepas dari kotak centang di atas, ketika sensor cahaya DAQ ditugaskan ke kamera RGB

, Chloros

menghitung — dari irradiansi absolut yang mengarah ke bawah yang diukur — eksposur maksimum × gain di mana permukaan dengan reflektansi 100% tetap di bawah batas clipping, dan menerapkannya sebagai **batas atas**pada eksposur otomatis. Sejak versi 1.3.0, batas atas ini hanya berlaku selama target kalibrasi berada dalam jangkauan kamera (terlihat dalam bingkai kamera dalam 10 detik terakhir); jika tidak ada target dalam jangkauan, eksposur otomatis biasa kamera, dengan perlindungan highlight-nya sendiri, akan berjalan sebagai gantinya, sehingga bagian lain dari pemandangan tidak dibuat lebih gelap dari yang seharusnya. Selama batas atas aktif, kamera beroperasi dalam mode**incident-pinned**: kamera beroperasi dalam mode loop terbuka pada eksposur yang diukur berdasarkan cahaya yang masuk dengan gain pada 0 dB — eksposur mengikuti cahaya yang terukur, bukan konten pemandangan. Karena batas atas hanya dapat memperpendek durasi eksposur, batas atas itu sendiri tidak dapat menyebabkan terjadinya clipping. Sejak versi 1.3.0, batas atas dihitung menggunakan konstanta eksposur kamera itu sendiri dari paket kalibrasinya (nilai rata-rata armada hanya jika paket tersebut tidak menyertakan konstanta tersebut). Batas atas akan dinonaktifkan secara otomatis — dan eksposur otomatis (AE) adegan normal akan dilanjutkan — setiap kali tidak ada target kalibrasi dalam jangkauan, pembacaan DAQ hilang, usang (&gt;30 detik), gelap, atau jika ≥15% bingkai mengalami clipping pada eksposur yang ditetapkan (artinya sensor dan kamera mendeteksi pencahayaan yang berbeda). Tidak ada tombol pengaturan di antarmuka pengguna; ini merupakan perilaku standar setiap kali kamera RGB

terhubung ke DAQ.

### Akuisisi &amp; Pemicu

Anggota<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

array juga menampilkan baris **Peran**(hanya baca) (Master berwarna biru / Slave berwarna hijau),**Garis Sinkronisasi**, dan**Rekan**.

| Kontrol | Pilihan | Default | Catatan |
| --- | --- | --- | --- |
| **Mode Pemicu** | Mati / Aktif | Aktif | Dinonaktifkan untuk anggota array (array mengelola pemicu). |
| **Sumber Pemicu** | Perangkat Lunak / Line0 (M8) / Line1 / Line2 | Line0 | Tersembunyi saat Mode Pemicu dalam keadaan Off; dinonaktifkan untuk anggota array. Line0 adalah input pemicu eksternal M8 yang diisolasi optik. |
| **Kecepatan Bingkai**| Otomatis / Manual + nilai | Otomatis |**Otomatis**: batas laju bingkai kamera dinonaktifkan — eksposur menentukan fps, dan kotak tersebut menampilkan laju aktual secara langsung.**Manual**: Anda membatasi fps dengan penggeser (1 hingga maksimum yang dibatasi bandwidth), yang dimulai dari laju aktual saat ini. Anggota array melihat &quot;N fps (langsung)&quot; yang hanya dapat dibaca dengan keterangan &quot;Ditetapkan dalam pengaturan array&quot;. |

### Jaringan / Transportasi

| Baris | Perilaku |
| --- | --- |
| **Ukuran Paket**| 1500 (Standar) / 9000 (Jumbo) — default**Jumbo**. |
| **Throughput** | Batas throughput tautan yang hanya dapat dibaca dalam MB/s. Backend akan menyeimbangkan nilai ini di antara semua kamera yang terhubung setiap kali terjadi koneksi atau pemutusan koneksi. |
| **Penanganan Buffer** | Mode penanganan buffer yang hanya dapat dibaca. |

### Pengambilan Gambar

Panel ini diakhiri dengan tombol **&quot;Buka Pengaturan Pengambilan Gambar…&quot;** yang akan mengarahkan Anda ke [panel Pengaturan Pengambilan Gambar](capture.md#the-capture-settings-pane) (dinonaktifkan hingga proyek dibuka — &quot;Buat atau buka proyek untuk menyimpan hasil perekaman&quot;). Jika kamera disembunyikan atau dijeda, petunjuk akan mengingatkan Anda untuk menampilkan kembali/melanjutkan sebelum melakukan perekaman.

## Panel pengaturan array

Buka dengan mengklik ikon **roda gigi**pada baris ARRAY. Header: nama array dengan ikon pensil untuk mengganti nama, dan**×** untuk menutup. Bagian di bawah yang bertanda *hanya gabungan* hanya muncul untuk array yang terhubung dalam mode tampilan gabungan.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sinkronisasi

Baris **Master**,**Slaves**, dan**Lini Sinkronisasi** hanya dapat dibaca.

### Sensor Cahaya Sekitar

Ditampilkan untuk array gabungan maupun terpisah:

* Menu tarik-turun **Sensor Cahaya**— mengikat satu DAQ ke seluruh array. Pilihan ini langsung berlaku, menyebar ke menu tarik-turun**Sensor Cahaya** masing-masing kamera anggota (Anda masih dapat mengganti pengaturan per kamera), dan mulai meneruskan spektrum ke array.
* Baris **Status** langsung: Mati · &quot;Menunggu spektrum pertama…&quot; · &quot;Aktif — semua kamera dalam susunan telah dikoreksi pencahayaannya&quot; · &quot;Tidak ada spektrum baru dalam 3 detik terakhir — masih menggunakan pembacaan terakhir (tidak ada batas waktu data kadaluwarsa)…&quot;.
* Catatan di panel: &quot;Koreksi radiometrik untuk seluruh array. Pengaturan per kamera akan menggantikan pengaturan ini.&quot;

### Target Kalibrasi

Kelompoknya sendiri tepat setelah Sensor Cahaya Sekitar, baik untuk array gabungan maupun terpisah:

* Kotak centang **Aktifkan** — &quot;Deteksi target ArUco MAPIR

dan validasi NDVI

vs LUT reflektansi panel&quot;; mengendalikan overlay target dan tabel validasi ubin gabungan. Pada array **terpisah**, ini mengaktifkan [Tabel Target Kalibrasi](#the-calibration-target-table) (satu per baris kamera dalam tampilan daftar).

### Pengambilan — pengaturan sensor seragam

Ditampilkan untuk susunan gabungan maupun terpisah (judul grup bertuliskan &quot;Pengambilan (seragam di seluruh kamera)&quot;). Pengaturan ini berlaku seragam untuk setiap anggota (perubahan per anggota akan merusak sinkronisasi). Pengeditan disimpan sementara dan diterapkan bersama-sama dengan tombol **Terapkan**— kecuali tombol pengalih**ROI aktif / ROI nonaktif**, yang langsung diterapkan.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Kontrol | Pilihan / rentang | Fungsinya |
| --- | --- | --- |
| **Format Piksel** | 8-bit / 10-bit / 12-bit / 16-bit | Kedalaman bit yang seragam untuk semua anggota; setiap kamera menggunakan format kedalaman tersebut untuk sensornya masing-masing. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning perangkat keras — mempertahankan bidang pandang penuh sekaligus meningkatkan SNR dan laju bingkai. Mengubahnya akan mengatur ulang bidang ROI ke bidang pandang penuh yang baru. |
| **Preset Resolusi** | Penuh / Setengah / Seperempat | Relatif terhadap binning; mengisi bidang ROI dengan potongan yang terpusat. |
| **Potongan ROI (px)**| Tombol pengalih**ROI on / ROI off**, tombol**Pilih**, bidang W / H / X / Y, peta sensor penuh | Potongan sensor — lihat [Editor ROI](#the-roi-editor). Lebar/tinggi disesuaikan ke kelipatan 16 (minimal 64); offset menyesuaikan ke kelipatan 4. Petunjuk &quot;max WxH&quot; menunjukkan batas atas dan**Reset** mengembalikan bidang ke bidang pandang penuh. Saat mengedit, kotak pemotongan oranye melacak setiap perubahan bidang pada peta dan pada ubin array. |
| **Laju Pemicu**| Tombol pengalih Otomatis / Manual + fps 0,5–10, langkah 0,5 |**Otomatis**(default): backend menghitung laju pemicu berdasarkan resolusi dan bandwidth — input dinonaktifkan dan menampilkan nilai yang dihitung.**Manual**: mengunci nilai Anda saat menekan Apply. Laju yang dikunci yang tidak dapat didukung oleh ukuran bingkai baru akan disesuaikan ke laju tercepat yang diizinkan oleh ukuran tersebut, dan kembali ke nilai semula saat ukuran bingkai mengizinkannya lagi. |

Catatan di panel: &quot;Perubahan format/resolusi akan me-restart semua kamera sebentar. Laju pemicu berlaku secara langsung.&quot; Tombol **Terapkan / Batal** terletak di bagian bawah panel.

#### Editor ROI

Baru di versi 1.3.0. Bar pemotongan ROI dan peta di bawahnya bekerja bersama:

* **ROI Aktif / ROI Nonaktif**— mengaktifkan atau menonaktifkan pemotongan sensor**secara instan**(kamera akan restart sebentar; ubin array menampilkan &quot;Menerapkan pengaturan…&quot;).**Mati**membaca seluruh sensor dan**mengingat pemotongan Anda**— kolom-kolom tetap menampilkannya;**Aktif** menerapkannya kembali. Mengaktifkan ROI tanpa pemotongan yang ditetapkan akan menerapkan pemotongan setengah ukuran yang terpusat (&quot;Tidak ada pemotongan yang ditetapkan, jadi pemotongan setengah ukuran yang terpusat diterapkan. Sesuaikan dengan kolom-kolom.&quot;).
* **Pilih**— menggambar pemotongan pada peta: seret melintasi area tersebut, atau klik dua sudut yang berlawanan (Esc untuk membatalkan). Persegi panjang tersebut akan tercatat di kolom W/H/X/Y, sesuai dengan langkah-langkah di atas; tidak ada yang diterapkan hingga Anda menekan**Terapkan**, sehingga Anda dapat menyempurnakannya terlebih dahulu dengan panah bidang. Satu persegi panjang per penekanan tombol Pilih.
* **Peta**menampilkan pemotongan pada seluruh sensor, sehingga Anda dapat memperluasnya kembali: &quot;Sensor penuh, langsung&quot; saat kamera membaca sensor penuh, atau &quot;Cuplikan bingkai penuh dari {waktu}. Kamera hanya membaca area yang dipotong.&quot; saat area tersebut dipotong. Jika array hanya pernah melakukan streaming dalam kondisi terpotong, peta akan menampilkan &quot;Belum ada tampilan bingkai penuh: array ini hanya melakukan streaming dalam kondisi terpotong.&quot; dengan tombol**Tampilkan bingkai penuh**, yang mematikan ROI. Label potongan `W×H @ X,Y` terletak di luar kotak (di atasnya, atau di bawahnya jika kotak menyentuh bagian atas), pada peta dan pada ubin langsung.
* **Terapkan**mengonfirmasi pengaturan bidang: pemotongan mengaktifkan ROI; bingkai penuh (setelah**Reset**) menonaktifkannya dengan pemotongan sebelumnya tetap diingat; ROI yang dinonaktifkan dan bidang-bidangnya masih menampilkan pemotongan yang diingat akan tetap dinonaktifkan saat Anda menerapkan perubahan lain (format, binning, laju).
* **Pada susunan yang telah diselaraskan, jendela pemotongan setiap anggota ditempatkan berdasarkan penyelarasan**, sehingga setiap kamera membingkai bagian yang sama dari pemandangan. Susunan yang dipotong sebelum diselaraskan akan diselaraskan terlebih dahulu pada sensor penuh (pemotongan dicabut, penyelarasan diselesaikan, dan pemotongan dipulihkan dengan jendela per kamera).
* Setelah perubahan pemotongan, eksposur otomatis akan menyesuaikan kembali untuk bingkai baru. Di bawah pemotongan, eksposur suatu anggota masih dapat berjalan hingga periode pemicu.
* Status aktif/nonaktif ROI dan pengaturan pemotongan yang tersimpan disimpan bersama proyek.

### Penyelarasan (ko-registrasi)

Ditampilkan untuk **baik array gabungan maupun terpisah** (versi 1.2.0 hanya menampilkannya untuk array gabungan); judul grup bertuliskan &quot;Penyelarasan&quot;.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Pill **Residual**: &quot;RMS x.xx px&quot; — hijau jika di bawah 1 px, kuning jika di bawah 3 px, merah jika melebihi itu atau jika ada kamera yang gagal; &quot;no profile&quot; sebelum penyelesaian pertama.
* Baris ringkasan: &quot;✓ Semua kamera telah diselaraskan (N)&quot; / &quot;⚠ p/N kamera telah diselaraskan —  <serial (filter)="">gagal&quot; / &quot;Pangkas aktif — Kalibrasi ulang untuk menyelaraskan (menggunakan sensor penuh)&quot; / &quot;Menunggu eksposur stabil…&quot; / &quot;Menerapkan pengaturan…&quot; (kamera sedang dimulai ulang) / &quot;Array berhenti mengirimkan bingkai — sambungkan kembali array&quot;.
* Tabel per kamera: kamera (4 digit terakhir nomor seri + filter), kesalahan reproyeksi dalam piksel dengan jumlah kecocokan (&quot;ref&quot; untuk kamera utama), dan skor korelasi silang yang dinormalisasi dari tumpang tindih terhadap batas lulus 0,35.
* **Kalibrasi ulang penyelarasan** tombol (bertuliskan &quot;Kalibrasi penyelarasan&quot; sebelum profil pertama) — menjalankan kembali pendaftaran bersama pada bingkai baru.
* **Kotak centang &quot;Atur eksposur otomatis kamera untuk penyelarasan&quot;** (dicentang secara default) — untuk sementara mencerahkan kamera yang gelap atau datar (eksposur terlebih dahulu, kemudian penguatan) sehingga memiliki tekstur yang dapat dicocokkan, kemudian mengembalikan AE.

Chloros secara otomatis menyelaraskan setiap array setelah terhubung, di kedua mode tampilan, setelah eksposur otomatis anggota array stabil (proses ini terus dicoba hingga 5 menit); pratinjau gabungan juga diselaraskan saat dibuka. Array terpisah tidak memiliki ubin gabungan, sehingga bagian ini memeriksa status penyelarasan dan menjalankan Recalibrate secara otomatis. Lakukan kalibrasi ulang jika fokus atau kedalaman adegan berubah. Penyelarasan **dirancang hanya berlaku per sesi** — tidak pernah disimpan ke profil, karena bergantung pada jarak adegan saat itu. Rekaman tetap dapat diekspor dengan registrasi piksel (lihat [Ekspor yang Diselaraskan](capture.md#per-array-controls)).

### Vignette Cerdas

* Kotak centang **Aktifkan koreksi**— menerapkan perkiraan vignette per kamera ke rantai radiometrik (secara langsung**dan** pada ekspor).
* **Kalibrasi dari tampilan saat ini**— arahkan array ke target yang seragam (panel datar, dinding, atau langit) terlebih dahulu; setiap kamera diratakan secara individual dan status melaporkan peningkatan kerataan &quot;n/N kamera · −x,x %&quot;.**Hapus** menghapus estimasi tersebut.
* Status: &quot;Telah dikalibrasi&quot;, &quot;Belum dikalibrasi&quot;, atau &quot;Terkalibrasi sebagian — jalankan ulang&quot; jika hanya perkiraan beberapa anggota yang masih berlaku pada bingkai saat ini (misalnya setelah fungsi &quot;Kalibrasi Ulang&quot; memindahkan jendela potongan suatu anggota) — jalankan &quot;Kalibrasi&quot; lagi.
* Lakukan penyempurnaan per kamera dengan penggeser **Vignette** per kamera di [Pratinjau Langsung](#live-preview).

### Pratinjau Langsung

* Kotak centang **Histogram**+ penggeser lebar (0,10–0,90 dari bingkai) — menampilkan strip histogram pada umpan langsung di setiap ubin anggota sekaligus (termasuk ubin saluran terpisah), serta [Overlay histogram](#display-overlays-drawn-over-the-live-feed) yang diatur untuk seluruh array. Statusnya akan terlihat aktif jika setiap anggota sudah memilikinya. Pada array**terpisah**, ini adalah baris satu-satunya dalam grup.
* *Hanya untuk gabungan:* **Indeks**: kotak centang aktif + ikon roda gigi — membuka [Kalkulator Indeks](#index-calculator-pane) bersama dengan rentang yang digambar dari**semua** kamera anggota. Baris pratinjau ekspresi di bawahnya menampilkan ekspresi saat ini (&quot;Belum ada ekspresi yang ditetapkan — buka kalkulator untuk membuatnya&quot;), yang diperbarui setiap detik.
* *Hanya gabungan:* Menu tarik-turun **Resolusi render**(preset yang sama seperti per kamera, default 720p): tinggi aliran tampilan langsung**dan** ukuran ekspor komposit yang disimpan. Catatan di panel: &quot;Ukuran pratinjau + komposit yang disimpan. Gambar per kamera selalu diekspor dalam resolusi penuh.&quot;

### Lapisan Tampilan *(hanya gabungan)** Kotak centang **Aktifkan** (default nonaktif — kamera utama ditampilkan langsung; aktif = komposit berlapis).
* Menu tarik-turun **Menu tarik-turun**Latar Depan**/**Latar Belakang**: setiap kamera anggota (berdasarkan nama) atau**Indeks**. Saat Latar Depan diatur ke Indeks, piksel di luar batas Min/Maks LUT akan menampilkan lapisan Latar Belakang.

### Tampilan Terpisah *(hanya gabungan)*

**&quot;Tampilkan kamera anggota&quot;**— tombol**Pisahkan / Sembunyikan kamera anggota** yang menambahkan umpan langsung masing-masing kamera anggota sebagai ubin kisi terpisah di samping komposisi. Ubin-ubin tersebut membaca buffer bingkai yang ada pada array (tanpa koneksi kamera tambahan). Hanya tampilan kisi; disimpan per array bersama proyek.

### Kemampuan

Panel baca-saja yang diperbarui setiap 5 detik:

* **Label tingkatan**: &quot;Perekaman simultan&quot; (hijau) · &quot;Perekaman simultan (emisi FTD-staggered)&quot; (hijau) · &quot;Perekaman staggered (drift 100 ms)&quot; (kuning) · &quot;Konfigurasi terlalu besar&quot; (merah).
* **Kondisi bingkai**: &quot;x.xx % belum lengkap&quot; — hijau di bawah 1 %, kuning di bawah 5 %, merah pada 5 % atau lebih.
* ****Baris tautan**: &quot;NIC {mbps} Mbps - berkelanjutan {MB/s} MB/s&quot;.

Ini adalah anggaran bandwidth real-time dari array. Untuk fps dasar dan model jaringan — serta apa yang harus diubah saat tingkatan berubah menjadi kuning atau merah — lihat [Array Multi-Kamera](arrays.md) dan [Referensi CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Panel Kalkulator Indeks

Halaman bilah samping ketiga, yang digunakan bersama oleh pengaturan Indeks per kamera dan pengaturan Indeks array gabungan (satu per satu — judulnya bertuliskan &quot;Kalkulator Indeks — <camera name="">&quot; atau &quot;Kalkulator Indeks — <array name="">&quot;). Panel ini menerima daftar band (band alami filter kamera-band alami, atau semua band di seluruh anggota array), ekspresi saat ini, dan konfigurasi LUT (aktif/nonaktif, level — default 7-stop, min — default 0,2, max — default 1), ditambah histogram indeks langsung. Tombol **Terapkan** akan menerapkan ekspresi; perubahan LUT langsung diterapkan pada pratinjau.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Pengaturan per-kamera vs pengaturan yang dikelola array

Referensi cepat mengenai letak pengaturan saat sebuah kamera merupakan anggota array:

| Dikelola oleh array (hanya baca di panel kamera) | Masih per-kamera di dalam array |
| --- | --- |
| Format piksel, resolusi, binning | Eksposur otomatis (eksposur, penguatan, target, perhalusan, ROI) |
| Mode/sumber pemicu, laju bingkai | Pengurangan noise, ketajaman, vignette |
| | Orientasi (cermin/rotasi), overlay tampilan (pengaturan array → Pratinjau Langsung mengatur histogram pada setiap anggota sekaligus), pengukur titik |
| | Indeks (array tampilan terpisah), pengikatan sensor cahaya |

Perilaku lintas-fungsi lainnya:

* **Tampilan gabungan vs terpisah** dipilih saat menghubungkan array: gabungan = satu ubin komposit yang sejajar (umpan anggota hanya melalui Tampilan Terpisah); terpisah = setiap anggota menampilkan ubinnya sendiri yang tersinkronisasi. Sebuah kamera tidak pernah menampilkan umpan mandiri dan ubin array secara bersamaan.
* ****Koneksi Otomatis**: membuka proyek yang disimpan akan memulihkan kamera dan array-nya serta menerapkan kembali setiap pengaturan yang disimpan ke backend sebelum aliran dilanjutkan.
* **Pembatasan perekaman**: kamera yang disembunyikan atau dijeda dikecualikan dari Capture All; sebuah array sepenuhnya diblokir hanya jika SEMUA anggota disembunyikan/dijeda. Lihat [Pengaturan &amp; Mode Capture](capture.md).

## Bagaimana pengaturan disimpan

Status tab kamera disimpan **bersama proyek**, bukan di browser:

* Setiap perubahan reaktif akan mengambil snapshot kamera dan array ke dalam file `cameras.json` proyek (dibounce 500 ms). Hal ini mencakup nama dan warna kamera, pengaturan eksposur/gain/AE, format piksel/resolusi/binning, laju pemicu, pengaturan pratinjau (resolusi render, penghilang noise, ketajaman, vignette, profil warna, Highlights, saturasi/kontras), orientasi, overlay, pemisahan saluran, konfigurasi indeks, pengaturan AE prediktif, ROI AE, nama array, mode tampilan, pengaturan pengambilan gambar array (termasuk pemotongan ROI, apakah ROI aktif, dan pemotongan yang diingat saat ROI nonaktif), Anggaran Kabel array, serta blok kisi (zoom umpan, mode tampilan, penguncian kisi, urutan ubin manual, kamera tersembunyi, ubin tertutup, kamera aktif).
* Pengikatan sensor cahaya disimpan dalam berkas `sensors.json` proyek.
* Membuka kembali proyek akan menghubungkan kembali perangkat keras dan menerapkan kembali semua pengaturan tersebut.
* **Tidak ada proyek yang terbuka = hanya sesi**: tanpa proyek, tidak ada pengaturan yang tersimpan setelah Chloros ditutup.
* Hanya berlaku untuk sesi terlepas dari proyek: status jeda, sampel pengukur titik, kotak centang Target Kalibrasi per kamera (selalu terbuka dalam keadaan nonaktif), dan profil penyelarasan susunan (dihitung ulang per sesi sesuai desain).
* Satu pengecualian: pilihan ekspor **Pengaturan Pengambilan** dan mode pengambilan tetap tersimpan per proyek di penyimpanan aplikasi lokal, bukan di `cameras.json` — lihat [Pengaturan &amp; Mode Pengambilan](capture.md).</array></camera></serial>

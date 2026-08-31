# Pengaturan Kamera

Tab **Kamera**adalah panel kontrol langsung Chloros untuk kamera LATTICE: area tampilan utama yang menampilkan setiap kamera yang terhubung sebagai ubin langsung, serta bilah samping yang dapat digeser di antara tiga halaman —**daftar kamera**,**panel pengaturan**(pengaturan per-kamera, array, atau perekaman — satu per satu), dan**Kalkulator Indeks**. Halaman ini mendokumentasikan setiap kontrol dalam daftar kamera, panel pengaturan per kamera, dan panel pengaturan array. Mode perekaman, pemilihan jenis ekspor, dan alur Capture All terdapat di halaman terkait [Pengaturan &amp; Mode Perekaman](capture.md).

Tab tab Kamera akan muncul di bilah samping setelah backend Chloros siap. Semua kontrol di bawah ini berkomunikasi dengan backend lokal melalui `127.0.0.1:5000`; perubahan akan langsung diterapkan ke kamera langsung kecuali disebutkan lain.

## Jenis kamera yang digunakan di halaman ini

Kontrol akan muncul atau tersembunyi tergantung pada jenis kamera yang dipilih. Panduan ini menggunakan istilah-istilah berikut secara konsisten:

| Istilah | Arti | Saluran filter |
| --- | --- | --- |
| **Kamera RGB** | LATTICE M3C dengan filter FRGB (model mencakup `-FRGB`) | Red / Green / Blue |
| **Multispektral Bayer** | LATTICE M3C dengan FRGN, FOCN, atau FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — satu filter pita sempit, satu pita yang dikalibrasi | Pita tunggal |
| **Anggota array** | Kamera yang terhubung sebagai bagian dari array yang disinkronkan (tampilan gabungan atau terpisah) | Sesuai filternya |

Kamera RGB menjalani pemrosesan fotometrik (keseimbangan putih, profil warna, gamma); kamera multispektral dan mono menjalani rantai radiometrik dan melewati pengaturan fotometrik. Anggota array menyerahkan pengaturan tingkat aliran (format piksel, resolusi, binning, pemicu, laju bingkai) ke array — baris-baris tersebut menjadi hanya-baca di panel per-kamera dan dipindahkan ke panel pengaturan array.

## Area

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

umpan utama Jika tidak ada kamera yang terhubung, area umpan menampilkan pesan pembuka **&quot;Hubungkan kamera untuk memulai&quot;**dengan dua tombol:**Hubungkan Kamera**(hijau, membuka dialog penghubungan kamera tunggal) dan**Hubungkan Array** (biru, membuka dialog penghubungan array). Dialog koneksi tersebut didokumentasikan dalam [Menghubungkan Kamera](connecting.md); konsep array (sinkronisasi, tingkatan, bandwidth) dalam [Multi-Kamera](arrays.md). Saat Anda membuka proyek tersimpan yang berisi kamera, layar pembuka akan menampilkan ikon spinner dengan tulisan &quot;Membuka kembali N kamera yang tersimpan…&quot; sementara Chloros memulihkan aliran video dari sesi terakhir.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Bilah atas

| Kontrol | Fungsinya |
| --- | --- |
| **Tombol beralih mode tampilan**| Beralih antara**tampilan kisi**(semua ubin sebagai sel) dan**tampilan daftar** (susunan kamera menempati lebar penuh di bagian atas, SATU kamera aktif di bawahnya). Keterangan: &quot;Beralih ke tampilan kisi&quot; / &quot;Beralih ke tampilan daftar&quot;. |
| **Kunci kisi**(gembok) | Secara default**terkunci** — ubin tetap pada posisinya. Buka kunci untuk menyeret dan mengatur ulang ubin ke slot mana pun (jarak antar ubin tetap terjaga). Kisi akan terkunci kembali secara otomatis setiap kali kamera baru terhubung. Keterangan: &quot;Buka kunci kisi (aktifkan seret ubin)&quot; / &quot;Kunci kisi (bekukan ubin di tempat)&quot;. |
| Penggeser **Zoom Umpan** | Ukuran ubin, mulai dari 60 piksel hingga lebar wadah penuh. Sel mempertahankan rasio aspek 4:3. Di bawah lebar sel 200 px, overlay nama dan fps akan tersembunyi agar ubin tetap rapi. |

### Ubin feed

Setiap kamera menampilkan ubin langsung komposit; sebuah kamera juga dapat menampilkan tiga ubin **pemisahan per saluran** skala abu-abu (lihat [Pemisahan Saluran](#display-overlays-drawn-over-the-live-feed)), dan array menampilkan ubin gabungan. Ubin yang aktif menampilkan cincin seleksi dengan warna kamera (atau array) tersebut.

Saat kursor diarahkan ke ubin, tombol tutup **X** akan muncul:

* Menutup ubin **gabungan** sementara pemisahan salurannya tetap aktif hanya akan menyembunyikan ubin gabungan tersebut.
* Menutup **ubin terakhir yang terlihat dari kamera mandiri** akan memutuskan koneksi kamera tersebut.
* **Ubin terpisah dari anggota susunan gabungan tidak akan pernah memutuskan koneksi** kamera — ubin tersebut hanya akan disembunyikan.

Dengan kisi yang tidak terkunci, seret ubin mana pun ke slot mana pun; tata letak akan disimpan bersama proyek.

## Panel Samping — daftar

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

kamera Halaman pertama panel samping mencantumkan setiap kamera dan array yang terhubung:

* **Hubungkan Kamera**(hijau) /**Hubungkan Array** (biru, menampilkan &quot;Mendeteksi...&quot; saat memindai). Keduanya dinonaktifkan saat dialog koneksi terbuka.
* **Capture All** (merah) — merekam semua kamera yang terdaftar dengan jenis ekspor yang dipilih di Pengaturan Perekaman. Membutuhkan proyek yang terbuka. Didokumentasikan secara lengkap di [Pengaturan &amp; Mode Perekaman](capture.md).
* **Gigi Pengaturan Pengambilan**(di sebelah**Ambil Semua**) — membuka [panel Pengaturan Pengambilan](capture.md#the-capture-settings-pane). Dinonaktifkan jika tidak ada proyek atau saat sedang melakukan pengambilan.

### Baris Kamera

Setiap baris kamera menampilkan batas berwarna (warna khusus kamera), label &quot;CAM&quot; — dengan huruf **M**(master) berwarna biru atau**S** (slave) untuk anggota array — serta nama tampilan. Nama defaultnya adalah `LATTICE-MODEL (serial)`; ganti namanya dari panel pengaturan per kamera. Tombol baris:

| Tombol | Efek |
| --- | --- |
| **Mata**| Mengaktifkan/menonaktifkan tampilan. Kamera yang disembunyikan akan keluar dari kisi dan**tidak termasuk dalam Capture All**. |
| **Roda Gigi** | Membuka panel pengaturan per kamera (bagian berikutnya). |
| **Jeda / Putar**| Membekukan pratinjau langsung**hanya di sisi tampilan** — proses perekaman di backend tetap berjalan. Kamera yang dijeda tidak dapat merekam. |
| **X** | Putus koneksi. Antarmuka pengguna diperbarui seketika (optimis); proses pemutusan koneksi di backend sendiri dapat memakan waktu 10–30 detik untuk selesai. |

### Baris array

Sebuah baris array menampilkan lencana &quot;ARRAY&quot; dengan warna array tersebut, nama array (dapat diganti di pengaturan array), dan tombol **DAQ · on/off**—**on** saat Sensor Cahaya tingkat array diaktifkan *atau* ada anggota yang memiliki sensor per kamera; tooltip-nya mencantumkan secara tepat sensor mana yang menyalurkan data ke mana. Kamera-kamera anggota tercantum dengan indentasi di bawahnya dalam baris masing-masing. Tombol-tombol baris array: **mata**(menyembunyikan/menampilkan SEMUA anggota sekaligus),**roda gigi**(panel pengaturan array),**X**(memutuskan koneksi seluruh array).

Status sensor cahaya (DLS) yang digunakan di baris array dan panel pengaturan array memiliki empat status:**mati**,**menunggu**(belum ada spektrum),**aktif**(spektrum diterima dalam 3 detik terakhir), dan**kadaluwarsa** — tidak ada spektrum baru dalam 3 detik, tetapi pembacaan terakhir *masih digunakan* (pembacaan DAQ tidak pernah kedaluwarsa di jalur penangkapan).

Anda dapat menyeret kamera mandiri dan seluruh grup array melewati satu sama lain di bilah samping untuk mengatur ulang urutan daftar; anggota array tidak dapat diseret secara terpisah.

## Panel pengaturan per kamera

Buka dengan mengklik ikon **roda gigi** pada baris kamera. Panel tersebut akan muncul di atas daftar kamera.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Header**:**sampel warna**kamera (klik untuk membuka pemilih warna bawaan — mengatur warna batas bilah samping dan warna cincin pemilihan ubin),**nama**dengan tombol pensil**Ubah Nama**(menyimpan nama kosong akan mengembalikan ke default `MODEL (serial)`), dan**×** untuk menutup.

### Histogram langsung

Di bagian atas panel terdapat histogram luma langsung yang dihitung dari pratinjau JPEG pada ~8 Hz. Nilai rata-ratanya diberi bobot Bayer — (R+2G+B)/4 — agar sesuai dengan pengukuran AE kamera itu sendiri.

* **Garis putus-putus Orange**= target AE.**Seret secara horizontal untuk mengubah target** — satu perintah dikirim saat dilepas, dan menyeret akan mengubah mode target AE menjadi Manual.
* **Garis penuh Green** = nilai rata-rata luma aktual (hasil yang saat ini diberikan oleh AE).
* **Tombol RGB** (kanan atas): mengaktifkan/menonaktifkan histogram overlay per-band yang diwarnai sesuai filter kamera (misalnya pada FRGN: abu-abu NIR, hijau, merah). Pada kamera mono (M3M), tombol tersebut bertuliskan &quot;MONO&quot; dan dinonaktifkan — mode mono selalu menampilkan histogram luma satu band.
* Label sumbu-X mengikuti kedalaman bit sensor dari format piksel saat ini: 0..255, 0..1023, 0..4095, atau 0..65535.

### Baris

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

informasi kamera | Baris | Perilaku |
| --- | --- |
| **Model** | Hanya baca (mis. `LATT-M3C-L87-FRGN`). |
| **Kalibrasi Radiometrik**| Green Lencana**&quot;Aktif&quot;**dengan keterangan yang menampilkan tingkatan kalibrasi, hash, tanggal kalibrasi, dan daftar band, yang dimuat dari paket kalibrasi kamera (lihat [Kalibrasi Radiometrik Pabrik](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Disembunyikan untuk kamera RGB** — kamera ini menggunakan kalibrasi keseimbangan putih fotometrik, bukan radian per-pita. |
| Tombol **Laporan Kalibrasi**|**Unduh** — membuka file PDF sertifikat kalibrasi NIST per nomor seri kamera di penampil bawaan sistem operasi Anda. Jika sertifikat belum disimpan dalam cache, Chloros akan menampilkan petunjuk sebagai gantinya. |
| **Nomor Seri** | Hanya baca. |
| **Firmware**| Menampilkan versi saat ini, kemudian menentukan versi yang tersedia untuk model ini (disimpan dalam cache per model — sebuah susunan N kamera akan memeriksa server satu kali). Status: &quot;Sedang memeriksa…&quot; → tombol**&quot;Perbarui ke X&quot;**→ &quot;Sedang mem-flash…&quot; → &quot;Diperbarui dari A ke B&quot; / &quot;Gagal: …&quot; / &quot;Dilewati: …&quot; /**&quot;Sudah terbaru&quot;** berwarna hijau. Keterangan pada tombol pembaruan: &quot;Reset pabrik + pembaruan firmware + pemrograman ulang UserSet1. ~2–3 menit; jangan putuskan koneksi.&quot; |
| **IP** | Hanya baca. |
| **Suhu** | Hanya baca, diperbarui setiap 3 detik. Berubah menjadi oranye pada ≥65 °C dan merah dengan ⚠ pada ≥75 °C. |
| Kotak centang **Target Kalibrasi** | Mengaktifkan deteksi target reflektansi ArUco dengan tabel validasi per panel NDVI di bawah umpan langsung (tampilan daftar). Hanya untuk sesi — selalu terbuka dalam keadaan mati. |
| Menu tarik-turun **Sensor Cahaya** | Menghubungkan sensor cahaya DAQ (DAQ-E/M/U, dari daftar tab Sensor Cahaya) ke kamera ini untuk koreksi pencahayaan cahaya turun (DLS) dan eksposur otomatis prediktif. Pilihan &quot;None&quot; menghapus pengikatan tersebut. Jika tidak ada sensor yang terhubung, menu tarik-turun akan menampilkan &quot;(tidak ada sensor yang terhubung — buka tab DAQ)&quot;. Pengikatan ini disimpan bersama proyek. |

### Eksposur &amp; Penguatan

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Semua input numerik di sini menggunakan spinner &quot;tahan untuk mempercepat&quot;: ketuk = ±1, tahan &gt;1,5 detik = ±10, tahan &gt;3 detik = ±100. Nilai tersebut dikirim ke kamera saat Anda melepaskannya.

| Kontrol | Rentang / pilihan | Default | Berlaku untuk | Fungsinya |
| --- | --- | --- | --- | --- |
| **Pencahayaan (us)**| Nilai minimum/maksimum real-time kamera | Otomatis | Semua | Waktu pencahayaan dalam mikrodetik, dengan tombol beralih**Otomatis/Manual**. Otomatis = pencahayaan otomatis berkelanjutan dari sisi kamera. |
| **Gain (dB)**| Nilai minimum/maksimum langsung dari kamera (mis. hingga 48 dB) | Manual (mati) | Semua | Penguatan analog/digital dengan tombol pengalih**Otomatis/Manual** tersendiri. |
| **Kecerahan Target AE**| 0–255 | 80, mode**Otomatis**| Semua (dapat diedit saat AE atau gain otomatis aktif) | Kecerahan yang menjadi target AE. Dalam**Otomatis**(pengaturan default), pengontrol latar belakang berbasis histogram akan memilih target tersebut sendiri, menjaga eksposur pada 60–75 % dari nilai maksimum sensor. Mengetikkan nilai atau menyeret garis oranye pada histogram akan mengubahnya menjadi**Manual**. |
| **Pengehalusan AE** | 0,5–40, langkah 0,1 | 8,0 | Semua | Pengehalusan AE. Keterangan: &quot;Semakin rendah = AE bereaksi lebih cepat (dapat berdenyut pada fps tinggi). Semakin tinggi = lebih halus / lebih lambat.&quot; Nilai yang jauh di bawah default dapat menyebabkan AE berkedip dan mengganggu stabilitas streaming pada frame rate tinggi; 8,0 adalah nilai default yang stabil. |
| **AE Region of Interest**| Kotak centang Aktifkan + tombol**Aim**| Nonaktif | Semua | Saat diaktifkan, AE hanya mengukur wilayah bergaris putus-putus hijau, bukan seluruh bingkai.**Aim** mengaktifkan fitur klik-untuk-menempatkan pada siaran langsung: satu klik akan memusatkan wilayah pada 30% dari bingkai; klik-seret akan menggambar persegi panjang kustom (minimal 5% × 5%). Fitur Aim akan dinonaktifkan secara otomatis setelah satu penempatan. Wilayah tersebut dipetakan kembali ke koordinat asli kamera sesuai rotasi/pencerminan yang Anda tetapkan, dan disimpan bersama proyek. |
| **Kecepatan Penyesuaian AE** | 0,1–5, langkah 0,1 | 1,0 | Hanya anggota array | Seberapa cepat target AE otomatis melacak perubahan kecerahan adegan; 1,0× melakukan pemeriksaan ulang setiap 2,5 detik. |
| **Highlight Protection** | Ketat (1%) / Normal (5%) / Longgar (15%) | Ketat | Kamera yang mendukung pengaturan ini | Seberapa besar bagian bingkai yang boleh terpotong menjadi putih sebelum AE menggelapkan gambar. |

{% hint style="info" %}
**Persyaratan cahaya untuk kamera multispektral Bayer (RGN / OCN / NGB):** adegan harus memiliki cahaya yang cukup di ketiga kanal atau kalibrasi tidak akan berfungsi dengan benar — satu eksposur sensor mencakup ketiga spektrum tersebut. Gunakan sensor cahaya DAQ untuk mengukur cahaya Anda, atau gunakan mode all-mono (M3M) agar setiap band mendapatkan eksposur tersendiri. Jika pengambilan gambar melanggar ketentuan ini, Chloros akan mendeteksinya dan memberi peringatan kepada Anda (notifikasi &quot;unmix-clamp&quot;).
{% endhint %}

### Format Piksel &amp;

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

Resolusi**Anggota array** menampilkan baris &quot;Current&quot; (format + WxH) dan &quot;Binning&quot; yang hanya dapat dibaca dengan catatan &quot;Ditetapkan di pengaturan array&quot; — memulai ulang aliran pada satu anggota akan merusak sinkronisasi, sehingga hal ini dikelola di [panel pengaturan array](#array-settings-pane).**Kamera mandiri** memiliki:

| Kontrol | Pilihan | Fungsinya |
| --- | --- | --- |
| **Format Piksel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Format piksel sensor (kedalaman bit). |
| **Resolusi** | Penuh / Setengah / Seperempat | Relatif terhadap penggabungan piksel saat ini: Penuh = 2048/N × 1536/N untuk penggabungan piksel N×N. |
| **Penggabungan Piksel** | 1x1 (tidak ada) / 2x2 / 4x4 | Binning N×N berbasis perangkat keras — nilai yang lebih besar menurunkan resolusi tetapi meningkatkan SNR dan laju bingkai. Mengubahnya akan memulai ulang streaming dan mengatur ulang ROI apa pun ke bidang pandang penuh yang baru. |
| **Saat Ini** | hanya baca | WxH aktual dan offset (x, y) yang berlaku. |

### Pratinjau Langsung

Semua yang ada di bagian ini bersifat **hanya di sisi tampilan**— pengaturan ini mengubah apa yang Anda lihat di siaran langsung, sementara rekaman yang disimpan tetap linier dan tidak dimodifikasi — dengan satu pengecualian:**Vignette** bersifat radiometrik dan memengaruhi ekspor juga (dijelaskan di bawah).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Kontrol | Rentang / pilihan | Default | Berlaku untuk | Fungsinya |
| --- | --- | --- | --- | --- |
| **Resolusi render** | 360p (tercepat) / 480p / 720p / 1080p / Resolusi sensor asli (paling lambat) | 720p | Semua | Tinggi di mana backend menjalankan rantai pratinjau radiometrik. Nilai yang lebih rendah meningkatkan laju bingkai tanpa mengubah bidang pandang. |
| **Indeks**| Kotak centang Aktifkan + roda gigi | Nonaktif | Hanya multispektral Bayer,**bukan** anggota susunan gabungan | Pratinjau indeks vegetasi langsung. Roda gigi membuka [Kalkulator Indeks](#index-calculator-pane) bersama yang telah dimuat sebelumnya dengan pita alami filter kamera (misalnya `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Ekspresi khusus ditambah LUT (aktif/nonaktif, tingkat default 3, min default 0,2, maks default 1) dihitung pada setiap bingkai pratinjau. Anggota array gabungan menyembunyikan baris ini — array tersebut memiliki satu indeks bersama. |
| **Keseimbangan Putih** | Mati / Sekali / Berkelanjutan + tombol pengambilan ulang | Berkelanjutan | Hanya RGB | Keseimbangan putih langsung. Tombol penyegaran mengambil ulang keseimbangan putih dari spektrum DLS saat ini (dinonaktifkan saat mode dalam keadaan Mati). |
| **Gamma** | Aktif / Nonaktif | Aktif | Hanya RGB | Menampilkan gamma (γ = 2,2 LUT) pada pratinjau langsung. Rekaman yang disimpan tetap linier. |
| **Pengurangan Noise** | Kotak centang + kekuatan 0–100 | Nonaktif / 50 | Semua (per kamera, bahkan di dalam array) | Filter bilateral pada pratinjau langsung. Nilai lebih tinggi = hasil lebih halus namun detailnya lebih lembut. |
| **Ketajaman** | Kotak centang + kekuatan 0–100 | Mati / 30 | Semua | Masker ketajaman pada pratinjau langsung, diterapkan terakhir. Dapat memperkuat noise. Hanya untuk pratinjau. |
| **Vignette**| Kotak centang + kekuatan 0–100 | Nonaktif / 0 | Semua | Penghilangan vignette sisa secara manual (mencerahkan sudut), ditumpuk di atas perkiraan Smart Vignette dari array.**Radiometrik — memengaruhi tampilan langsung DAN ekspor**, tidak seperti Denoise/Sharpness. |
| **Profil Warna** | Raw / Linear / Natural / Enhanced / Suhu Kustom | Natural | Hanya RGB | Lihat di bawah. |
| **Suhu Warna** | 2000–10000 K, langkah 100 | 5500 K | Hanya RGB, profil Suhu Kustom | Menetapkan keseimbangan putih ke suhu warna terkorelasi tetap (masukan DLS diabaikan). Nilai Kelvin yang terakhir dipilih akan disimpan saat beralih antar profil. |
| **Saturasi** | 0–200 (100 = netral) | 100 | Hanya RGB | Saturasi HSV pada pratinjau langsung. |
| **Kontras** | 0–200 (100 = netral) | 100 | Hanya RGB | Kontras linier di sekitar abu-abu tengah pada pratinjau langsung. |
| **Cerminkan H / Cerminkan V** | Kotak centang | Nonaktif | Semua | Membalik pratinjau secara horizontal / vertikal. |
| **Rotasi**| 0° / 90° / 180° / 270° | 0° | Semua | Memutar pratinjau. Orientasi diterapkan pada akhir rantai pratinjau backend —**tangkapan yang disimpan tetap dalam orientasi asli kamera**, dan tampilan komposit array mengabaikannya. |**Semantik profil warna** (kamera RGB):

* **Raw** — melewati seluruh rantai pemrosesan.
* **Linear** — sinyal gelap + flat-field + keseimbangan putih; tanpa matriks warna, tanpa gamma.
* **Natural** *(default)* — linear ditambah matriks koreksi warna yang diukur dan kurva nada yang beradaptasi dengan adegan.
* **Enhanced**— Natural ditambah vibrance dan kontras lokal CLAHE. Biaya tambahan hanya berlaku untuk**pratinjau langsung** — tangkapan yang disimpan selalu mendapatkan hasil akhir penuh terlepas dari profil yang digunakan.
* **Suhu Kustom** — Natural dengan keseimbangan putih dikunci pada nilai Kelvin pilihan Anda.

{% hint style="warning" %}
Untuk Natural, Enhanced, dan Suhu Kustom, panel menampilkan catatan nada: bingkai dicerahkan sesuai adegan masing-masing, sehingga gambar *tampilan* yang disimpan tidak dapat dibandingkan antar-bingkai. **Ekspor radian atau reflektansi untuk pengukuran.**
{% endhint %}

### Overlay tampilan (digambar di atas siaran langsung)

Fitur ini hanya ada di antarmuka pengguna — digambar di atas video, tidak pernah menyentuh siaran atau tangkapan layar.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Overlay | Kontrol | Default | Fungsinya |
| --- | --- | --- | --- |
| **Zebra** | Kotak centang + ambang batas 200–255 | Nonaktif / 250 | Garis-garis diagonal berwarna magenta pada piksel yang terpotong. |
| **Crosshair** | Kotak centang | Nonaktif | Tanda pusat bingkai. |
| **Grid** | Nonaktif / 3 × 3 / 9 × 9 | Nonaktif | Kisi komposisi. |
| **Histogram** | Kotak centang + lebar 0,10–0,90 bingkai | Nonaktif / 0,25 | Strip histogram yang ditampilkan secara real-time. |
| **Focus Peak** | Kotak centang + ambang batas 20–200 + contoh warna | Nonaktif / 80 / `#ff5722` | Penyorotan tepi Sobel untuk pemfokusan. |
| **Pembagian Saluran** | &quot;Tampilkan Pembagian (Red / Green / NIR)&quot; / Tombol &quot;Sembunyikan Pemisahan&quot; | Tersembunyi | Menambahkan tiga ubin skala abu-abu independen per kanal di samping gambar gabungan (label tombol mengikuti kanal filter kamera). Setiap ubin pemisahan dapat diseret dan memiliki warna tepi yang sama dengan kamera. Tidak tersedia pada kamera monokrom. Disimpan bersama proyek. |

### Pengukur Titik

* Kotak centang **Klik untuk Mengambil Sampel**: klik tampilan langsung untuk mengambil sampel satu piksel (ditandai dengan retikel berbentuk silang), atau klik-seret suatu wilayah untuk mendapatkan rata-rata piksel.**Hapus**menghapus sampel dan retikel. Fitur ini saling eksklusif dengan mode**Aim** AE-ROI.
* Menu tarik-turun **Tampilkan**:**Raw (kedalaman bit)**— angka digital asli pada kedalaman bit sensor (mis. 12-bit → 0..4095) — atau**Tampilan (8-bit)** (default). Saat indeks langsung aktif, Tampilan akan menampilkan nilai indeks yang dihitung (mis. NDVI) sebagai gantinya.
* Panel pembacaan mencantumkan koordinat piksel, ukuran bingkai, format piksel, kedalaman bit, dan tabel saluran (Chan / Nilai / %) dengan label pita dan panjang gelombang; pasangan hijau Bayer dirata-ratakan; sampel wilayah menampilkan &quot;N px avg&quot;.

Status pengukur titik hanya berlaku selama sesi.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Pengeksposuran Otomatis Prediktif (Didorong DLS)

Bagian ini hanya muncul jika **setidaknya satu sensor cahaya DAQ terhubung** — pemecah masalah memerlukan spektrum downwelling langsung untuk menggerakkannya.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Kontrol | Rentang | Default | Fungsinya |
| --- | --- | --- | --- |
| **Aktifkan** | Kotak centang | Aktif (kamera mandiri) | Solver bentuk tertutup menggunakan spektrum DLS ditambah skalar paket kalibrasi kamera untuk memastikan pita tercerah mendekati saturasi sambil menjaga pita tergelap di atas batas bawah SNR — satu penulisan eksposur per penyelesaian, tanpa loop penyetelan. Dirancang untuktimelapse bertenaga surya di mana setiap bidikan harus memiliki eksposur yang tepat. Backend secara diam-diam akan kembali ke AE reaktif setiap kali pembacaan DLS kedaluwarsa/hilang atau paket kalibrasi tidak dimuat. |
| **Pengehalusan (α)** | 0,05–1,0, langkah 0,05 | 0,3 | Pengehalusan solusi prediktif berturut-turut (semakin rendah = semakin halus). |
| **Reflektansi adegan**| Tombol**Kalibrasi Ulang ρ** | — | Menghitung ulang faktor reflektansi adegan yang digunakan oleh pemecah masalah. |

{% hint style="info" %}
**Penghubungan array menonaktifkan AE prediktif secara default** — untuk array, fitur &quot;smart AE&quot; dari Chloros ditambah pengaturan eksposur otomatis di sisi kamera menangani eksposur (dengan perlindungan saturasi), dan estimasi reflektansi pemandangan tunggal dari AE prediktif tidak aman untuk pemandangan campuran. Anda dapat-aktifkan kembali per kamera di sini jika Anda secara khusus menginginkan eksposur radiometrik yang digerakkan oleh DLS.
{% endhint %}

**Batas eksposur yang dikendalikan DAQ dan AE yang disesuaikan dengan insiden.**Terlepas dari kotak centang di atas, ketika sensor cahaya DAQ ditugaskan ke kamera RGB, Chloros menghitung — dari irradiansi turun absolut yang diukur — eksposur maksimum × gain di mana permukaan dengan reflektansi 100% tetap di bawah batas clipping, dan menerapkannya sebagai**batas atas**pada eksposur otomatis. Selama batas atas aktif, kamera berada dalam mode**terikat cahaya masuk**: kamera beroperasi dalam mode loop terbuka pada eksposur yang diukur dari cahaya masuk dengan gain pada 0 dB — eksposur mengikuti cahaya yang terukur, bukan konten pemandangan. Karena batas atas hanya dapat memperpendek eksposur, batas atas itu sendiri tidak dapat menyebabkan clipping. Batas atas ini akan dinonaktifkan secara otomatis — dan AE pemandangan normal akan dilanjutkan — setiap kali pembacaan DAQ hilang, kedaluwarsa (&gt;30 detik), atau gelap, atau jika ≥15% bingkai mengalami kliping pada eksposur yang terikat (artinya sensor dan kamera mendeteksi pencahayaan yang berbeda). Tidak ada tombol pengaturan di antarmuka pengguna; ini adalah perilaku standar setiap kali kamera RGB terhubung ke DAQ.

### Anggota Array Akuisisi &amp; Pemicu

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

juga menampilkan baris **Peran**(hanya baca: Master berwarna biru / Slave berwarna hijau),**Jalur Sinkronisasi**, dan**Rekan**.

| Kontrol | Pilihan | Default | Catatan |
| --- | --- | --- | --- |
| **Mode Pemicu** | Mati / Aktif | Aktif | Dinonaktifkan untuk anggota array (array mengelola pemicu). |
| **Sumber Pemicu** | Perangkat Lunak / Line0 (M8) / Line1 / Line2 | Line0 | Tersembunyi saat Mode Pemicu dalam keadaan Mati; dinonaktifkan untuk anggota array. Line0 adalah . |
| **Kecepatan Bingkai**| Otomatis / Manual + nilai | Otomatis |**Otomatis**: batas kecepatan bingkai kamera dinonaktifkan — eksposur menentukan fps, dan kotak tersebut menampilkan kecepatan aktual secara langsung.**Manual**: Anda membatasi fps dengan penggeser (1 hingga maksimum yang dibatasi bandwidth), yang didasarkan pada laju aktual saat ini. Anggota array melihat &quot;N fps (live)&quot; yang hanya dapat dibaca dengan keterangan &quot;Ditetapkan di pengaturan array&quot;. |

### Jaringan / Transportasi

| Baris | Perilaku |
| --- | --- |
| **Ukuran Paket**| 1500 (Standar) / 9000 (Jumbo) — default**Jumbo**. |
| **Throughput** | Batas throughput tautan yang hanya dapat dibaca dalam MB/s. Backend akan menyeimbangkan nilai ini di seluruh kamera yang terhubung setiap kali terjadi koneksi atau pemutusan koneksi. |
| **Penanganan Buffer** | Mode penanganan buffer yang hanya dapat dibaca. |

### Pengambilan Gambar

Panel ini diakhiri dengan tombol **&quot;Buka Pengaturan Pengambilan Gambar…&quot;** yang akan mengarahkan Anda ke [panel Pengaturan Pengambilan Gambar](capture.md#the-capture-settings-pane) (dinonaktifkan hingga proyek dibuka — &quot;Buat atau buka proyek untuk menyimpan hasil pengambilan gambar&quot;). Jika kamera disembunyikan atau dijeda, petunjuk akan mengingatkan Anda untuk menampilkan kembali/melanjutkan sebelum melakukan perekaman.

## Panel pengaturan array

Buka dengan mengklik ikon **roda gigi**pada baris ARRAY. Header: nama array dengan ikon pensil untuk mengganti nama, dan**×** untuk menutup. Bagian di bawah yang ditandai *hanya gabungan* hanya muncul untuk array yang terhubung dalam mode tampilan gabungan.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Sinkronisasi

Baris **Master**,**Slaves**, dan**Lini Sinkronisasi** hanya dapat dibaca.

### Sensor Cahaya Sekitar

Ditampilkan untuk array gabungan maupun terpisah:

* **Target Kalibrasi** — &quot;Mendeteksi target ArUco MAPIR dan memvalidasi NDVI terhadap LUT reflektansi panel&quot;; mengendalikan overlay target dan tabel validasi pada ubin gabungan.
* Menu tarik-turun **Sensor Cahaya** — mengikat satu DAQ ke seluruh array. Pilihan ini langsung berlaku, diteruskan ke menu tarik-turun Sensor Cahaya masing-masing kamera anggota (Anda masih dapat mengganti pengaturan per kamera), dan mulai meneruskan spektrum ke array.
* Baris **Status** langsung: Mati · &quot;Menunggu spektrum pertama…&quot; · &quot;Aktif — semua kamera dalam array telah dikoreksi pencahayaannya&quot; · &quot;Tidak ada spektrum baru dalam 3 detik terakhir — masih menggunakan pembacaan terakhir (tidak ada batas waktu data kadaluwarsa)…&quot;.
* Catatan di panel: &quot;Koreksi radiometrik untuk seluruh array. Pengaturan per kamera akan menggantikan pengaturan ini.&quot;

### Pengambilan Gambar — pengaturan sensor seragam *(hanya digabungkan)*

Pengaturan ini berlaku secara seragam untuk setiap anggota (perubahan per anggota akan merusak sinkronisasi). Pengeditan disimpan sementara dan diterapkan secara bersamaan.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Kontrol | Pilihan / rentang | Fungsinya |
| --- | --- | --- |
| **Format Piksel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Format sensor seragam untuk semua anggota. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning perangkat keras — mempertahankan bidang pandang penuh sekaligus meningkatkan SNR dan laju bingkai. Mengubahnya akan mengatur ulang bidang ROI ke bidang pandang penuh yang baru. |
| **Resolusi** prasetel | Penuh / Setengah / Seperempat | Relatif terhadap penggabungan piksel; mengisi bidang ROI dengan pemotongan yang terpusat. |
| **Pemotongan ROI (px)**| Bidang numerik W / H / X / Y | Pemotongan sensor. Lebar/tinggi disesuaikan ke kelipatan 16 (minimum 64); offset disesuaikan ke kelipatan 4. Petunjuk &quot;max WxH&quot; menunjukkan batas atas dan**Reset** akan kembali ke bidang pandang penuh. Selama pengeditan, kotak pratinjau pemotongan berwarna oranye ditampilkan secara langsung pada ubin array (termasuk skema sensor penuh saat memperluas pemotongan ke luar). |
| **Kecepatan Pemicu**| Tombol beralih Otomatis / Manual + fps 0,5–10, langkah 0,5 |**Otomatis**(default): sistem belakang menghitung kecepatan pemicu berdasarkan resolusi dan bandwidth — input dinonaktifkan dan menampilkan nilai yang dihitung.**Manual**: mengunci nilai Anda saat menekan**Apply**. |

Catatan di panel: &quot;Perubahan format/resolusi akan me-restart semua kamera sebentar. Laju pemicu berlaku secara langsung.&quot; Tombol **Apply / Cancel** terletak di bagian bawah panel.

### Penyelarasan (ko-registrasi) *(hanya gabungan)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Panel **Residual**: &quot;RMS x.xx px&quot; — hijau jika di bawah 1 px, kuning jika di bawah 3 px, merah jika melebihi itu atau jika ada kamera yang gagal; &quot;tidak ada profil&quot; sebelum penyelesaian pertama.
* Baris ringkasan: &quot;✓ Semua kamera telah diselaraskan (N)&quot; / &quot;⚠ p/N kamera telah diselaraskan —  <serial (filter)="">gagal&quot; / &quot;Pangkas aktif — Kalibrasi ulang untuk menyelaraskan (menggunakan sensor penuh)&quot; / &quot;Menunggu eksposur stabil…&quot;.
* Tabel per kamera: kamera (4 digit terakhir nomor seri + filter), kesalahan reproyeksi dalam px dengan jumlah kecocokan (&quot;ref&quot; untuk kamera utama), dan skor korelasi silang terstandarisasi tumpang tindih terhadap ambang batas lulus 0,35.
* Tombol **Kalibrasi Ulang Penyelarasan** (bertuliskan &quot;Kalibrasi penyelarasan&quot; sebelum profil pertama) — menjalankan kembali pendaftaran bersama pada bingkai baru.
* Kotak centang **&quot;Atur eksposur otomatis kamera untuk penyelarasan&quot;** (dicentang secara default) — sementara mencerahkan kamera yang gelap atau datar (eksposur terlebih dahulu, lalu gain) agar memiliki tekstur yang dapat dicocokkan, lalu mengembalikan AE.

Pratinjau gabungan akan secara otomatis disejajarkan saat dibuka; kalibrasi ulang jika fokus atau kedalaman pemandangan berubah. Penjajaran ini **secara desain hanya berlaku untuk sesi tersebut** — tidak pernah disimpan ke profil, karena bergantung pada jarak pemandangan pada saat itu. Hasil tangkapan masih dapat diekspor dengan registrasi piksel (lihat [Ekspor yang disejajarkan](capture.md#per-array-controls)).

### Vignette Cerdas

* Kotak centang **Aktifkan koreksi**— menerapkan perkiraan vignette per kamera ke rantai radiometrik (baik saat langsung**maupun** saat diekspor).
* **Kalibrasi dari tampilan saat ini**— arahkan susunan kamera ke target seragam (panel datar, dinding, atau langit) terlebih dahulu; setiap kamera diratakan secara individual dan status melaporkan peningkatan kerataan &quot;n/N kamera · −x,x %&quot;.**Hapus** menghapus perkiraan tersebut.
* Sesuaikan secara detail per kamera dengan penggeser **Vignette** per kamera di [Pratinjau Langsung](#live-preview).

### Pratinjau Langsung *(hanya gabungan)** **Indeks**: centang kotak centang + ikon roda gigi — membuka [Kalkulator Indeks](#index-calculator-pane) bersama dengan pita yang digambar dari**semua** kamera anggota. Baris pratinjau ekspresi di bawahnya menampilkan ekspresi saat ini (&quot;Belum ada ekspresi yang ditetapkan — buka kalkulator untuk membuatnya&quot;), yang diperbarui setiap detik.
* Menu tarik-turun **Resolusi Render**(preset yang sama seperti per kamera, default 720p): tinggi aliran tampilan langsung**dan** ukuran ekspor komposit yang disimpan. Catatan di panel: &quot;Ukuran pratinjau + komposit yang disimpan. Gambar per kamera selalu diekspor dalam resolusi penuh.&quot;

### Tampilkan Lapisan *(hanya gabungan)** Kotak centang **Aktifkan** (default nonaktif — kamera utama ditampilkan langsung; aktif = komposit berlapis).
* Menu tarik-turun **Latar Depan**/**Latar Belakang**: setiap kamera anggota (berdasarkan nama) atau**Indeks**. Saat Latar Depan diatur ke Indeks, piksel di luar batas LUT Min/Max akan menampilkan lapisan Latar Belakang.

### Tampilan Terpisah *(hanya gabungan)*

**&quot;Tampilkan kamera anggota&quot;**— tombol**Pisahkan / Sembunyikan kamera anggota** yang menambahkan umpan langsung masing-masing kamera anggota sebagai ubin kisi terpisah di samping komposisi. Ubin-ubin tersebut membaca buffer bingkai yang sudah ada dari array (tanpa koneksi kamera tambahan). Hanya tampilan kisi; disimpan per array bersama proyek.

### Kemampuan

Panel baca-saja yang diperbarui setiap 5 detik:

* **Label tingkatan**: &quot;Perekaman simultan&quot; (hijau) · &quot;Perekaman simultan (emisi FTD-staggered)&quot; (hijau) · &quot;Perekaman staggered (drift 100 ms)&quot; (kuning) · &quot;Konfigurasi terlalu besar&quot; (merah).
* **Kondisi bingkai**: &quot;x,xx % belum lengkap&quot; — hijau di bawah 1 %, kuning di bawah 5 %, merah pada 5 % atau lebih.
* **Garis koneksi**: &quot;NIC {mbps} Mbps - berkelanjutan {MB/s} MB/s&quot;.

Ini adalah anggaran bandwidth real-time dari array tersebut. Untuk model fps dan jaringan yang mendasarinya — serta apa yang perlu diubah ketika tingkatan berubah menjadi kuning atau merah — lihat [Susunan Multi-Kamera](arrays.md) dan [Referensi CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Panel Kalkulator Indeks

Halaman bilah samping ketiga, yang digunakan bersama oleh gear Indeks per-kamera dan gear Indeks array gabungan (satu per satu — judulnya bertuliskan &quot;Kalkulator Indeks — <camera name="">&quot; atau &quot;Kalkulator Indeks —<array name="">

&quot;). Panel ini menerima daftar pita (pita alami filter kamera, atau semua pita di seluruh anggota array), ekspresi saat ini, dan konfigurasi LUT (aktif/nonaktif, level — default 3, min — default 0,2, max — default 1), ditambah histogram indeks langsung. **Terapkan** akan menerapkan ekspresi; perubahan LUT langsung diterapkan pada pratinjau.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Pengaturan per kamera vs pengaturan yang dikelola array

Referensi cepat mengenai letak pengaturan saat sebuah kamera merupakan anggota array:

| Dikelola oleh array (hanya baca di panel kamera) | Masih per kamera di dalam array |
| --- | --- |
| Format piksel, resolusi, binning | Eksposur otomatis (eksposur, gain, target, perataan, ROI) |
| Mode/sumber pemicu, laju bingkai | Pengurangan noise, ketajaman, vignette |
| | Orientasi (cermin/rotasi), overlay tampilan, pengukur titik |
| | Indeks (array tampilan terpisah), pengikatan sensor cahaya |

Perilaku lintas-aspek lainnya:

* **Tampilan gabungan vs terpisah** dipilih saat menghubungkan array: gabungan = satu ubin komposit yang sejajar (anggota hanya mengirimkan umpan melalui Split View); terpisah = setiap anggota merender ubinnya sendiri yang disinkronkan. Sebuah kamera tidak pernah menampilkan umpan mandiri dan ubin susunan secara bersamaan.
* **Sambungan Otomatis**: membuka proyek yang disimpan akan memulihkan kamera dan susunannya serta menerapkan kembali setiap pengaturan yang disimpan ke backend sebelum aliran dilanjutkan.
* **Pembatasan perekaman**: kamera yang disembunyikan atau dijeda dikecualikan dari Perekaman Semua; sebuah array hanya diblokir sepenuhnya jika SEMUA anggota disembunyikan/dijeda. Lihat [Pengaturan &amp; Mode Perekaman](capture.md).

## Bagaimana pengaturan disimpan

Status tab kamera disimpan **bersama proyek**, bukan di browser:

* Setiap perubahan reaktif akan mengambil snapshot kamera dan array ke dalam berkas `cameras.json` proyek (dibounce 500 ms). Ini mencakup nama dan warna kamera, pengaturan eksposur/gain/AE, format piksel/resolusi/binning, laju pemicu, pengaturan pratinjau (resolusi render, penghilang noise, ketajaman, vignette, profil warna, saturasi/kontras), orientasi, overlay, pemisahan saluran, konfigurasi indeks, pengaturan AE prediktif, ROI AE, nama array, mode tampilan, pengaturan perekaman array (termasuk posisi pemotongan ROI), dan blok kisi (zoom umpan, mode tampilan, kunci kisi, urutan ubin manual, kamera tersembunyi, ubin tertutup, kamera aktif).
* Pengikatan sensor cahaya disimpan dalam berkas `sensors.json` proyek.
* Membuka kembali proyek akan menghubungkan kembali perangkat keras dan menerapkan kembali semua pengaturan tersebut.
* **Tidak ada proyek yang terbuka = sesisaja**: tanpa proyek, tidak ada yang tersimpan setelah menutup Chloros.
* Hanya sesi terlepas dari proyek: status jeda, sampel pengukur titik, kotak centang Target Kalibrasi per kamera (selalu terbuka dalam keadaan mati), dan profil penyelarasan susunan (dihitung ulang per sesi sesuai desain).
* Satu pengecualian: pilihan ekspor **Pengaturan Pengambilan** dan mode pengambilan tetap tersimpan per proyek di penyimpanan aplikasi lokal, bukan di `cameras.json` — lihat [Pengaturan &amp; Mode Pengambilan](capture.md).</array></camera></serial>

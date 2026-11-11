---
metaLinks: {}
---

# Memulai

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>

Chloros adalah aplikasi perangkat lunak dari [MAPIR](https://www.mapir.camera) untuk memproses citra multispektral, mengendalikan perangkat keras MAPIR secara langsung, dan merekam data sensor. Chloros 1.3.0 mendukung seluruh rangkaian produk MAPIR:

* **Kamera Survey3** — memproses tangkapan RAW+JPG menjadi peta reflektansi dan indeks vegetasi yang telah dikalibrasi. Lihat [Kamera yang Didukung](supported-cameras.md).
* **Kamera LATTICE** — menghubungkan modul kamera multispektral GigE secara langsung, baik secara individual maupun sebagai susunan multi-kamera yang tersinkronisasi: pratinjau, ambil gambar, dan olah menjadi produk radian dan reflektansi yang dikalibrasi. Lihat [bagian LATTICE](lattice/README.md).
* **Sensor cahaya DAQ** — sensor spektral DAQ-U (USB), DAQ-M (Bluetooth), serta DAQ-E dan DAQ-E-S (Ethernet): perekaman spektrum yang dikalibrasi secara langsung, `.daq` perekaman, serta iluminasi ke bawah untuk pemrosesan reflektansi. Lihat [bagian DAQ](daq/README.md).

{% hint style="success" %}
**Apa yang Baru di Chloros 1.3.0**: Asisten AI dapat mengoperasikan Chloros melalui `chloros-mcp`, dan CLI, SDK, serta `chloros-mcp` menawarkan fitur yang sama. Produk indeks LATTICE telah ditingkatkan, susunan sensor lengkap bekerja lebih cepat dan dapat beroperasi di jaringan apa pun, serta backend tetap berjalan di antara perintah. Lihat daftar "Apa yang Baru" di bawah ini, dan [Unduh](download.md) untuk penginstal.
{% endhint %}

{% hint style="info" %}
**Menggunakan Chloros dengan asisten AI?** Panduan ini dirancang khusus untuk itu. Arahkan asisten Anda ke:

* `https://mapir.gitbook.io/chloros/llms.txt` — indeks yang dapat dibaca mesin untuk setiap halaman.
* Halaman apa pun dalam format Markdown mentah — tambahkan `.md` ke URL-nya (misalnya `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referensi CLI](reference/cli-reference.md) dan [Referensi SDK](reference/sdk-reference.md) — halaman referensi lengkap dengan nilai yang tepat, yang ditulis untuk digunakan oleh LLM.

Contoh prompt: *"Baca https://mapir.gitbook.io/chloros/reference/cli-reference.md, lalu tulis skrip yang masuk ke sistem dan memproses folder ~/flights/flight_001 menjadi file GeoTIFF dengan nilai reflektansi + NDVI."*

Atau biarkan menjalankan Chloros secara langsung: `chloros-mcp`, sebuah server MCP yang diinstal bersama SDK, kompatibel dengan Claude Desktop, Claude Code, Cursor, dan klien MCP lainnya.

Panduan lengkap: [Menggunakan Chloros dengan Asisten AI](ai-assistants.md).
{% endhint %}

***

## Apa yang Baru di Chloros 1.3.0

* **Asisten AI dapat mengendalikan Chloros.** `chloros-mcp`, sebuah server MCP yang diinstal bersama Python SDK, memungkinkan Claude Desktop, Claude Code, Cursor, dan klien MCP lainnya untuk memproses citra serta mengoperasikan kamera LATTICE, array, dan sensor cahaya DAQ. Lihat [Menggunakan Chloros dengan Asisten AI](ai-assistants.md).
* **CLI, SDK, dan `chloros-mcp` menawarkan fitur yang sama.** Hanya proses masuk, penyediaan DAQ-E, dan server DAQ langsung yang tetap berada di CLI, demi keamanan. Beberapa skrip versi 1.2.0 memerlukan perubahan: lihat **Perubahan pada 1.3.0** di bagian atas [Referensi CLI](reference/cli-reference.md) dan [Referensi SDK](reference/sdk-reference.md).
* **Produk indeks LATTICE yang ditingkatkan.** NDVI dan produk indeks lainnya yang diekspor dari tangkapan LATTICE telah ditingkatkan, di setiap mode pemrosesan.
* **Opsi penginstal Windows.**Di bawah**Pengaturan sistem**, penginstal menawarkan opsi**Aktifkan jalur panjang Windows**dan**Kecualikan backend Chloros dari pemindaian Microsoft Defender**, keduanya dicentang secara default. Opsi pertama memungkinkan folder proyek memiliki nama yang lebih panjang dari 260 karakter; opsi kedua mencegah pemindaian real-time memperlambat pengambilan gambar kamera (sekitar 30% dari laju pengambilan pada kecepatan tinggi). Penginstal kini juga menambahkan folder CLI ke `PATH` hanya sekali, serta menghapus entri duplikat yang ditinggalkan oleh penginstal versi sebelumnya. Lihat [Unduh](download.md).
* **Folder proyek yang terlalu dalam ditolak sejak awal.** Dengan jalur panjang Windows dinonaktifkan (pengaturan default Windows), folder proyek yang terlalu dalam untuk batas jalur 260 karakter akan ditolak sebelum proses apa pun dijalankan, disertai informasi mengenai folder proyek terpanjang yang masih sesuai serta cara mengaktifkan jalur panjang. Sebelumnya, proses semacam itu tidak menghasilkan produk apa pun. Dengan jalur panjang diaktifkan, produk di folder yang sangat dalam kini mempertahankan metadata-nya (tag GPS dan XMP).
* **Masker pemotongan.** Ketika pemisahan band pada produk radian atau reflektansi LATTICE menghasilkan nilai band negatif, nilai tersebut dipotong menjadi nol, dan masker per-piksel di dalam folder `clamp_masks/` yang terletak di samping folder produk mencatat piksel dan band mana saja yang terpengaruh. Produk menamai maskernya dalam tag `Chloros:UnmixClampMask`. Tampilan langsung kini melakukan pemotongan persis seperti saat ekspor. Lihat [Format Gambar Keluaran](output-image-formats.md).
* **Produk mendeskripsikan dirinya sendiri.** Produk indeks memiliki tag pemrosesan sendiri (`Chloros:ProcessingLevel=index`, ekspresi indeks, dan apakah dihitung dari radian atau reflektansi) alih-alih menggunakan tag dari data mentah sumber. Produk Survey3 memiliki tag pemrosesan Chloros di setiap mode pemrosesan. File mentah dan file yang telah di-debayered LATTICE tetap dalam orientasi asli sensor dan mencatat orientasinya dalam XMP; Penampil Gambar hanya memutarnya untuk keperluan tampilan.
* **Paket Linux dikirim dalam bentuk yang belum diekstrak.** Backend dan CLI diinstal sebagai folder di bawah `/usr/lib/chloros/`, sehingga tidak lagi mengekstrak dirinya sendiri setiap kali diluncurkan. Pada arsitektur x86_64, backend tidak lagi membutuhkan waktu sekitar 100 detik untuk memulai, dan ukuran `.deb` kini sekitar setengah dari ukuran sebelumnya. ExifTool yang disertakan kini dapat berjalan pada instalasi Ubuntu yang bersih. `.deb` kini juga menginstal Python SDK untuk sistem `python3` (pasang dengan `sudo apt install ./chloros_1.3.0_<arch>.deb`), meningkatkan ukuran buffer penerimaan soket kernel yang dibutuhkan oleh aliran data kamera, serta menambahkan layanan backend yang berjalan sebagai pengguna Anda. Paket Jetson, `chloros_1.3.0_arm64_jp6.deb`, dikembangkan untuk JetPack 6 dan memproses data di GPU Jetson menggunakan build CUDA PyTorch dari JetPack. Di Linux, setiap gambar yang diekspor kini hanya menyentuh disk sekali (ditulis dan diberi tag di memori, lalu disalin ke lokasi tujuan): di Jetson, di mana kecepatan disk menjadi penentu, eksekusi LATTICE dengan 288 gambar memakan waktu 14 menit, bukan 24 menit. Lihat [Instalasi Linux](linux/linux-installation.md).
* **Kamera LATTICE OCN membaca data dengan lebih akurat di bawah sinar matahari.** Radiance, reflektansi, dan indeks mendapatkan koreksi cepat setelah pemisahan band, sehingga target abu-abu terbaca mendekati nilai NDVI netral di bawah sinar matahari. Kamera RGN tidak mengalami perubahan.
* **Laporan bug dalam aplikasi.** Ikon bug di bagian atas tab Debug Log akan mengirimkan deskripsi, hingga 10 gambar, log, dan detail sistem ke MAPIR. Lihat [GUI : Navigasi](navigation.md).
* **Array LATTICE.**Array mempertahankan sensor penuh secara default, dan pengiriman data 12-bit terkompresi (`--packed`, default `auto`) meningkatkan laju bingkai sensor penuh sekitar sepertiga; sebuah array berfungsi di jaringan apa pun pada kecepatan bingkai yang diizinkan oleh jaringan tersebut (koneksi lambat akan menurunkan kecepatan bingkai alih-alih menolak array atau memperkecil ukuran bingkainya); array yang kameranya mengalami restart daya akan pulih dengan sendirinya, dengan PTP kembali aktif; pengaturan eksposur otomatis tidak lagi menurunkan kecepatan tersebut dalam kondisi cahaya redup (sistem mengisi periode terlebih dahulu, lalu meningkatkan gain); Perekaman berkelanjutan tetap dapat mengikuti kecepatan array yang cepat; kontrol**Highlights** untuk kamera RGB; dan `ArraySession` dalam SDK mendapatkan `grab()`, `frame_stream()`, dan `continuous()`. Lihat [bagian LATTICE](lattice/README.md).
* **Eksposur otomatis yang lebih stabil.** Paparan tetap stabil pada cahaya konstan dan stabil dalam beberapa detik setelah terjadi perubahan; sorotan kecil tidak lagi menggelapkan pemandangan; paparan manual tetap mempertahankan penguatan pada mode Otomatis; dan kamera yang berhenti merespons di bawah sinar matahari terik akan dipertahankan pada tingkat cahaya yang sesuai, yang dibaca oleh sensor cahaya atau, jika tidak terhubung, oleh bingkai kamera itu sendiri. Lihat [Pengaturan Kamera](lattice/camera-settings.md).
* **Sensor cahaya.** **Dukungan DAQ-E-S:** difuser tetapnya tidak memerlukan koreksi penutup, dan setiap pembacaan mencakup data kemiringan, rotasi, dan sudut kemiringan unit. Sebuah rekaman menunjukkan apakah penutupnya dipilih atau diasumsikan, dan pemrosesan akan memberikan peringatan jika penutup diasumsikan: DAQ-U, DAQ-M, dan DAQ-E dikirim tanpa penutup, jadi sebutkan penutup yang terpasang. Perekaman langsung `.daq` tetap bertahan saat terjadi pemadaman listrik (simpan berkas `.daq-journal` beserta `.daq`-nya), dan DAQ-E yang berhenti mengirim data akan terhubung kembali dengan jeda yang ditandai dalam rekamannya. Sebuah proyek yang hanya menyimpan rekaman sensor cahaya akan menampilkan daftar dan memprosesnya melalui aplikasi maupun CLI, dan rekaman yang dibuat dengan `chloros_scripts` akan terdaftar sebagai DAQ yang membuatnya, dengan batas yang dapat Anda atur. Setiap penangkapan menyimpan pembacaan DAQ yang digunakannya. Lihat [bagian DAQ](daq/README.md).
* **Pemrosesan.**Frame LATTICE mulai diproses selama deteksi target. Referensi target dan DAQ ditentukan per band. Proyek yang dipindahkan akan menghubungkan ulang berkas-berkasnya. Ekspor dari penangkapan 8-bit berformat 8-bit, dengan format baru**TIFF (8-bit)**. Proses Survey3 tanpa target kalibrasi disimpan sebagai produk cadangannya (`Vignette_Corrected_Images` atau `Sensor_Response_Images`), bukan sebagai citra reflektansi. Tag DLS per-band pada citra reflektansi selalu berasal dari bingkai aslinya. Fitur Texture Aware ditolak untuk pengambilan gambar LATTICE, yang selalu menggunakan proses debayer standar.
* **CLI & SDK.** Jalur keluaran relatif kini berada di folder tempat Anda menjalankan perintah atau skrip (sebelumnya, backend yang terinstal akan mencoba menemukannya di folder instalasinya sendiri dan gagal). Backend tetap berjalan di antara perintah dan skrip, dan akan berhenti sendiri setelah 30 menit tidak digunakan; `chloros-cli backend status` dan `backend stop` mengelolanya. Perintah penangkapan mencantumkan nama setiap berkas yang tidak dapat ditulis dan keluar dengan kode 1, serta penangkapan berulang akan berhenti jika drive penuh. Eksekusi `process` yang berhasil melaporkan `Image products written: N (plus M clamp masks)`. Di Windows, `ChlorosLocal` / `process_folder` dari SDK akan menemukan backend yang terinstal (dan menggunakan yang sudah berjalan), sedangkan `read_image_audit_tags` berfungsi dari `pip install` biasa.

***

## Apa yang Baru di Chloros 1.2.0

* **Kontrol kamera langsung — tab Kamera baru.** Hubungkan kamera LATTICE satu per satu atau sebagai susunan multi-kamera yang tersinkronisasi (sinkronisasi waktu PTP, pengambilan gambar yang dipicu perangkat keras), dengan overlay pratinjau langsung, histogram per-pita, eksposur otomatis cerdas, kalkulator indeks langsung, dan pembaruan firmware kamera dalam aplikasi.
* **Sensor cahaya — tab Sensor Cahaya baru.** Hubungkan sensor DAQ-U (USB), DAQ-M (Bluetooth), dan DAQ-E (Ethernet); lihat spektrum yang telah dikalibrasi secara langsung (W/m²/nm), rekam berkas `.daq` ke dalam proyek Anda, pilih profil koreksi cap, dan perbarui firmware DAQ-E melalui jaringan.
* **Mode perekaman dan perekam.** Perekaman Tunggal / Berkelanjutan / Interval ditambah mode Perekaman Tercepat yang hanya menghasilkan data mentah; pemilihan per proyek mengenai kamera dan jenis ekspor yang dihasilkan oleh "Capture All"; perekam array untuk video indeks tingkat pemantauan dan burst data mentah tingkat analisis dengan pembuatan video offline.
* **Pipa pemrosesan LATTICE.** Impor folder perekaman LATTICE dan pisahkan setiap bingkai mentah menjadi produk debayered, pratinjau, radian float32 (W/m²/sr/nm), dan reflektansi dengan tombol pengaktifan per produk. Reflektansi dapat berasal dari target kalibrasi dalam bingkai atau sinar turun DAQ; penyelarasan array diterapkan pada ekspor; kalibrasi pabrik yang hilang diunduh secara otomatis berdasarkan nomor seri kamera.
* **Proyek mengingat konfigurasi perangkat keras.** Kamera dan sensor cahaya yang terhubung disimpan bersama proyek (`cameras.json` / `sensors.json`) dan terhubung kembali dengan pengaturan yang disimpan saat Anda membuka kembali proyek tersebut. Lihat [GUI: Proyek](projects.md).
* **Peningkatan penampil gambar.** Pembacaan piksel/indeks kursor dengan penskalaan reflektansi per-file yang benar, histogram lapisan, penggeser pengelompokan GSD, mode kisi Per Trigger / Per Kamera, tampilan produk LATTICE, serta ekspor sandbox Indeks/LUT ke disk.
* **CLI & SDK, diperluas secara signifikan.** Keluarga perintah baru `lattice`, `daq pool-*`, `project`, dan `time-sync`; opsi baru `process` (`--input-level`, tombol pengalih per produk, `--reflectance-source`, bendera penyelarasan array); penanganan koneksi cerdas SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`) yang secara otomatis memulai backend; Otomatisasi `open_project()`; roda SDK disertakan dalam penginstal dan dipublikasikan ke PyPI sebagai `chloros-sdk`.
* **Semantik kegagalan yang jujur.** Eksekusi `chloros-cli process` yang meminta produk tetapi tidak menulis satupun kini gagal dengan jelas dan keluar dengan nilai non-nol; eksekusi yang berhasil melaporkan berapa banyak produk gambar yang telah ditulis.
* **Tata letak keluaran baru.** Produk disimpan dalam folder `<project>/<camera>/<format>/<Product>_Images/` dan mempertahankan nama file sumber — folder, bukan sufiks nama file, yang mengidentifikasi produk. Lihat [Format Gambar Keluaran](output-image-formats.md).
* **Lebih banyak input, rencana, dan bahasa.** Dukungan input `.dng`; semua 38 bahasa antarmuka telah tersedia sepenuhnya; batasan perangkat keras per rencana dengan penggunaan gratis (tanpa login) hingga 4 kamera dan 2 sensor cahaya.
* **Keandalan.** Fitur “Stop Processing” berhenti dengan rapi disertai ringkasan eksekusi yang jelas, proyek multi-kamera mengekspor data dari setiap kamera, dan pembaruan melalui penginstal tidak lagi membuat Anda keluar dari akun.***

Chloros tersedia dalam 3 antarmuka aplikasi:

## Chloros: Aplikasi GUI desktop

Jendela terpisah mandiri dengan semua fitur, termasuk tab Kamera Langsung dan Sensor Cahaya. _Hanya untuk Windows._

## [Chloros CLI: Antarmuka baris perintah](CLI.md)

Pemrosesan batch baris perintah ditambah perintah langsung `lattice`, `daq pool-*`, `project`, dan `time-sync`. Sangat cocok untuk otomatisasi, pembuatan skrip, dan operasi tanpa antarmuka grafis. Tersedia di **Windows, Linux amd64, dan Linux arm64 (NVIDIA Jetson)**. _CLI
memerlukan langganan Chloros+ berbayar untuk mengaksesnya._

## [Chloros API: Python SDK](api-python-sdk.md)

Antarmuka Python berbasis pemrograman untuk otomatisasi dan alur kerja khusus: pemrosesan pipa lengkap, sesi kamera/array langsung, sesi sensor DAQ, dan otomatisasi proyek yang disimpan. Diinstal bersama paket desktop/CLI dan juga diterbitkan sebagai `pip install chloros-sdk`. _API memerlukan langganan Chloros+ berbayar untuk mengaksesnya._

***

## Platform yang Didukung

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ya | Ya | Ya |
| **Linux amd64 (x86_64)** | Tidak | Ya | Ya |
| **Linux arm64 (NVIDIA Jetson)** | Tidak | Ya | Ya |

Untuk petunjuk instalasi Linux, lihat bagian [Linux & Edge Computing](linux/linux-overview.md).

***

## Memulai dalam Tiga Langkah

1. **Instal** — unduh dan jalankan penginstal untuk platform Anda. Lihat [Unduh](download.md).
2. **Masuk (opsional untuk GUI)** — GUI memproses gambar secara gratis tanpa perlu akun. [Masuk ke Chloros+](chloros+-login.md) membuka akses ke pemrosesan paralel, akselerasi GPU, batas perangkat yang lebih tinggi, serta akses CLI/SDK.
3. **Buat proyek pertama Anda** — buka Chloros, buat [Proyek Baru](projects.md), [tambahkan gambar Anda](processing-images-gui/adding-files-to-a-project.md), dan [mulai pemrosesan](processing-images-gui/starting-the-processing.md). Untuk mengendalikan perangkat keras secara langsung, buka tab Kamera atau Sensor Cahaya — lihat [GUI: Navigasi](navigation.md).

***

## Chloros


Meskipun Chloros dapat digunakan secara gratis untuk sebagian besar tugas, Anda mungkin merasa membutuhkan lebih banyak fitur. Di sinilah lisensi berbayar untuk Chloros+ dapat bermanfaat bagi Anda. Dengan lisensi Chloros+, Anda dapat mengakses fitur-fitur baru seperti:

* **Pemrosesan Multi-threaded**: mempercepat pemrosesan gambar secara signifikan untuk proyek-proyek besar dengan memproses gambar secara bersamaan melalui alur kerja.
* **Akselerasi GPU (CUDA)**: manfaatkan opsi memori GPU yang lebih besar saat ini untuk semakin mempercepat alur kerja pemrosesan gambar. Kami merekomendasikan VRAM sebesar 4GB atau lebih untuk hasil terbaik.
* **Chloros+**[**CLI**](CLI.md) **Akses**: jalankan Chloros+ dari baris perintah untuk mengotomatisasi dan mengintegrasikannya ke dalam perangkat lunak Anda sendiri. Tersedia di semua tingkatan berlangganan berbayar; diberlakukan di sisi server.
* **Chloros+**[**API**](api-python-sdk.md) **Akses:** jalankan Chloros+ dari Python untuk kontrol terprogram, sehingga memungkinkan integrasi yang mulus dengan alur kerja penelitian, alur kerja analisis data, dan aplikasi khusus Anda. Tersedia di semua tingkatan berbayar; diberlakukan di sisi server.
* **Batas Perangkat Keras yang Lebih Tinggi**: hubungkan lebih banyak kamera dan sensor cahaya sekaligus. Tanpa login, antarmuka pengguna grafis (GUI) dapat menghubungkan hingga 4 kamera dan 2 sensor cahaya DAQ; tingkatan berbayar meningkatkan kedua batas tersebut:

| Paket | Kamera | Sensor cahaya DAQ |
| --- | --- | --- |
| Iron (gratis, tanpa login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Penggunaan Beberapa Perangkat**: setiap lisensi Chloros+ memungkinkan pendaftaran 2 perangkat atau lebih. Gunakan akun MAPIR Cloud Anda untuk mengelola perangkat yang terdaftar. Tambahkan dukungan untuk lebih banyak perangkat dengan meningkatkan lisensi Chloros+ Anda.
* **Metode Debayer Canggih yang Memperhatikan Tekstur:** debayer berkualitas tinggi yang memperhatikan tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir semua noise debayering.
* **Rumus Indeks Multispektral Kustom:** masukkan indeks multispektral kustom ke dalam kalkulator raster Chloros, baik untuk pemrosesan maupun kotak pasir (sandbox) penayangan gambar.
* **Linux & Komputasi Tepi:** jalankan Chloros pada platform Linux x86\_64 dan ARM64, termasuk NVIDIA Jetson, untuk pemrosesan lapangan dan tepi. Lihat [Ikhtisar Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Harga & Pendaftaran Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->

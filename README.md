---
metaLinks: {}
---

# Memulai

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros adalah aplikasi perangkat lunak dari [MAPIR](https://www.mapir.camera) untuk memproses citra multispektral, mengendalikan perangkat keras MAPIR secara langsung, dan merekam data sensor. Chloros 1.2.0 mendukung seluruh rangkaian produk MAPIR:

* **Kamera Survey3** — memproses tangkapan RAW+JPG menjadi peta reflektansi dan indeks vegetasi yang telah dikalibrasi. Lihat [Kamera yang Didukung](supported-cameras.md).
* **Kamera LATTICE** — menghubungkan modul kamera multispektral GigE secara langsung, baik secara individual maupun sebagai susunan multi-kamera yang tersinkronisasi: pratinjau, ambil gambar, dan olah menjadi produk radian dan reflektansi yang telah dikalibrasi. Lihat [bagian LATTICE](lattice/README.md).
* **Sensor cahaya DAQ** — sensor spektral DAQ-U (USB), DAQ-M (Bluetooth), dan DAQ-E (Ethernet): spektrum terkalibrasi secara langsung, perekaman `.daq`, dan iluminasi ke bawah untuk pemrosesan reflektansi. Lihat [bagian DAQ](daq/README.md).

{% hint style="success" %}
**Apa yang Baru di Chloros 1.2.0**: kontrol kamera dan array LATTICE secara langsung, integrasi sensor cahaya DAQ, mode perekaman dan perekam, alur pemrosesan radiometrik LATTICE yang lengkap, otomatisasi proyek dari CLI/SDK, dan masih banyak lagi. Lihat daftar &quot;Apa yang Baru&quot; di bawah ini, dan [Unduh](download.md) untuk catatan perubahan.
{% endhint %}

{% hint style="info" %}
**Menggunakan Chloros dengan asisten AI?** Panduan ini dirancang khusus untuk itu. Arahkan asisten Anda ke:

* `https://mapir.gitbook.io/chloros/llms.txt` — indeks yang dapat dibaca mesin untuk setiap halaman.
* Halaman apa pun dalam format Markdown mentah — tambahkan `.md` ke URL-nya (misalnya `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referensi CLI](reference/cli-reference.md) dan [Referensi SDK](reference/sdk-reference.md) — halaman referensi lengkap dengan nilai yang tepat, yang ditulis untuk digunakan oleh LLM.

Contoh prompt: *&quot;Baca https://mapir.gitbook.io/chloros/reference/cli-reference.md,, lalu tulis skrip yang masuk ke sistem dan memproses folder ~/flights/flight_001 menjadi berkas GeoTIFF dengan nilai reflektansi + NDVI.&quot;*

Panduan lengkap: [Menggunakan Chloros dengan Asisten AI](ai-assistants.md).
{% endhint %}

***

## Apa yang Baru di Chloros 1.2.0

* **Kontrol kamera langsung — tab Kamera baru.** Hubungkan kamera LATTICE satu per satu atau sebagai susunan multi-kamera yang tersinkronisasi (sinkronisasi waktu PTP, pengambilan gambar yang dipicu perangkat keras), dengan overlay pratinjau langsung, histogram per-pita, eksposur otomatis cerdas, kalkulator indeks langsung, dan pembaruan firmware kamera dalam aplikasi.
* **Sensor cahaya — tab Sensor Cahaya yang baru.** Hubungkan sensor DAQ-U (USB), DAQ-M (Bluetooth), dan DAQ-E (Ethernet); lihat spektrum yang telah dikalibrasi secara langsung (W/m²/nm), rekam berkas `.daq` ke dalam proyek Anda, pilih profil koreksi cap, dan perbarui firmware DAQ-E melalui jaringan.
* **Mode perekaman dan perekam.** Perekaman Tunggal / Berkelanjutan / Interval ditambah mode Perekaman Tercepat yang hanya menghasilkan data mentah; pemilihan per-proyek mengenai kamera dan jenis ekspor mana yang dihasilkan oleh &quot;Capture All&quot;; perekam array untuk video indeks tingkat pemantauan dan burst data mentah tingkat analisis dengan pembuatan video offline.
* **Pipa pemrosesan LATTICE.** Impor folder perekaman LATTICE dan pisahkan setiap frame mentah menjadi produk debayered, pratinjau, radian float32 (W/m²/sr/nm), dan reflektansi dengan tombol pengaktifan per produk. Reflektansi dapat berasal dari target kalibrasi dalam bingkai atau data DAQ downwelling; penyelarasan array diterapkan pada ekspor; kalibrasi pabrik yang hilang diunduh secara otomatis berdasarkan nomor seri kamera.
* **Proyek mengingat konfigurasi perangkat keras.** Kamera dan sensor cahaya yang terhubung disimpan bersama proyek (`cameras.json` / `sensors.json`) dan terhubung kembali dengan pengaturan yang disimpan saat Anda membuka kembali proyek tersebut. Lihat [GUI: Proyek](projects.md).
* **Peningkatan penampil gambar.** Pembacaan piksel/indeks kursor dengan penskalaan reflektansi per file yang benar, histogram lapisan, penggeser pengelompokan GSD, mode kisi Per Trigger / Per Camera, tampilan produk LATTICE, dan ekspor sandbox Indeks/LUT ke disk.
* **CLI &amp; SDK, diperluas secara signifikan.** Keluarga perintah baru `lattice`, `daq pool-*`, `project`, dan `time-sync`; opsi `process` baru (`--input-level`, pengaturan per produk, `--reflectance-source`, bendera penyelarasan array); penanganan smart-connect SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`) yang secara otomatis memulai backend; Otomatisasi `open_project()`; roda SDK disertakan dalam penginstal dan dipublikasikan ke PyPI sebagai `chloros-sdk`.
* **Semantik kegagalan yang jelas.** Eksekusi `chloros-cli process` yang meminta produk tetapi tidak menghasilkan apa pun kini gagal secara jelas dan keluar dengan nilai non-nol; eksekusi yang berhasil melaporkan berapa banyak produk gambar yang dihasilkan.
* **Tata letak keluaran baru.** Produk disimpan dalam folder `<project>/<camera>/<format>/<Product>_Images/` dan mempertahankan nama file sumber — folder, bukan sufiks nama file, yang mengidentifikasi produk. Lihat [Format Gambar Keluaran](output-image-formats.md).
* **Lebih banyak input, rencana, dan bahasa.** Dukungan input `.dng`; semua 38 bahasa antarmuka telah tersedia sepenuhnya; batasan perangkat keras per rencana dengan penggunaan gratis (tanpa login) hingga 4 kamera dan 2 sensor cahaya.
* **Keandalan.** Fitur “Stop Processing” berhenti dengan rapi disertai ringkasan proses yang jelas, proyek multi-kamera mengekspor data dari setiap kamera, dan pembaruan melalui penginstal tidak lagi membuat Anda keluar dari sistem.***

Chloros tersedia dalam 3 antarmuka aplikasi:

## Chloros: Aplikasi GUI desktop

Jendela terpisah mandiri dengan semua fitur, termasuk tab Kamera Langsung dan Sensor Cahaya. _Hanya untuk Windows._

## [Chloros CLI: Antarmuka baris perintah](CLI.md)

Pemrosesan batch baris perintah ditambah perintah langsung `lattice`, `daq pool-*`, `project`, dan `time-sync`. Sangat cocok untuk otomatisasi, pembuatan skrip, dan operasi tanpa antarmuka grafis. Tersedia di **Windows, Linux amd64, dan Linux arm64 (NVIDIA Jetson)**. _CLI memerlukan tingkatan berbayar Chloros+ untuk mengaksesnya._

## [Chloros API: Python SDK](api-python-sdk.md)

Antarmuka Python berbasis pemrograman untuk otomatisasi dan alur kerja khusus: pemrosesan jalur lengkap, sesi kamera/array langsung, sesi sensor DAQ, dan otomatisasi proyek yang disimpan. Diinstal bersama paket desktop/CLI dan juga diterbitkan sebagai `pip install chloros-sdk`. _API ini memerlukan tingkatan berbayar Chloros+ untuk diakses._

***

## Platform yang Didukung

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Ya | Ya | Ya |
| **Linux amd64 (x86_64)** | Tidak | Ya | Ya |
| **Linux arm64 (NVIDIA Jetson)** | Tidak | Ya | Ya |

Untuk petunjuk pemasangan Linux, lihat bagian [Linux &amp; Komputasi Tepi](linux/linux-overview.md).

***

## Memulai dalam Tiga Langkah

1. **Instal** — unduh dan jalankan penginstal untuk platform Anda. Lihat [Unduh](download.md).
2. **Masuk (opsional untuk GUI)** — GUI memproses gambar secara gratis tanpa perlu akun. [Masuk dengan Chloros+](chloros+-login.md) membuka akses ke pemrosesan paralel, akselerasi GPU, batas perangkat yang lebih tinggi, serta akses ke CLI/SDK.
3. **Buat proyek pertama Anda** — buka Chloros, buat [Proyek Baru](projects.md), [tambahkan gambar Anda](processing-images-gui/adding-files-to-a-project.md), dan [mulai pemrosesan](processing-images-gui/starting-the-processing.md). Untuk mengendalikan perangkat keras secara langsung, buka tab Kamera atau Sensor Cahaya — lihat [GUI: Navigasi](navigation.md).***

## Chloros+

Meskipun Chloros dapat digunakan secara gratis untuk sebagian besar tugas, Anda mungkin merasa memerlukan fitur yang lebih banyak. Di sinilah lisensi berbayar untuk Chloros+ dapat bermanfaat bagi Anda. Dengan lisensi Chloros+, Anda dapat membuka fitur-fitur baru seperti:

* **Pemrosesan Multi-threaded**: mempercepat pemrosesan gambar secara signifikan untuk proyek-proyek berskala besar dengan memproses gambar secara bersamaan melalui pipeline.
* **Akselerasi GPU (CUDA)**: manfaatkan opsi memori GPU yang lebih besar saat ini untuk semakin mempercepat alur kerja pemrosesan gambar. Kami merekomendasikan VRAM sebesar 4GB atau lebih untuk hasil terbaik.
* **Chloros+**[**CLI**](CLI.md)**Akses**: jalankan Chloros+ dari baris perintah untuk mengotomatiskan dan mengintegrasikannya ke dalam perangkat lunak Anda sendiri. Tersedia di semua tingkatan berbayar; diterapkan di sisi server.
* **Chloros+**[**API**](api-python-sdk.md)**Akses:** jalankan Chloros+ dari Python untuk kontrol terprogram, sehingga memungkinkan integrasi yang mulus dengan alur kerja penelitian, alur kerja analisis data, dan aplikasi khusus Anda. Tersedia di semua tingkatan berbayar; diterapkan di sisi server.
* **Batas Perangkat Keras yang Lebih Tinggi**: hubungkan lebih banyak kamera dan sensor cahaya sekaligus. Tanpa login, antarmuka pengguna (GUI) dapat menghubungkan hingga 4 kamera dan 2 sensor cahaya DAQ; tingkatan berbayar menaikkan kedua batas tersebut:

| Paket | Kamera | Sensor cahaya DAQ |
| --- | --- | --- |
| Iron (gratis, tanpa login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Penggunaan Beberapa Perangkat**: setiap lisensi Chloros+ memungkinkan pendaftaran 2 perangkat atau lebih. Gunakan akun MAPIR Cloud Anda untuk mengelola perangkat yang terdaftar. Tambahkan dukungan untuk lebih banyak perangkat dengan meningkatkan lisensi Chloros+ Anda.
* **Metode Debayer Canggih yang Memperhatikan Tekstur:** metode debayer berkualitas tinggi yang memperhatikan tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir seluruh noise debayer.
* **Rumus Indeks Multispektral Kustom:** masukkan indeks multispektral kustom ke dalam kalkulator raster Chloros, baik untuk pemrosesan maupun lingkungan uji coba tampilan gambar.
* **Linux &amp; Komputasi Tepi:** jalankan Chloros pada platform Linux x86_64 dan ARM64, termasuk NVIDIA Jetson, untuk pemrosesan di lapangan dan di tepi. Lihat [Gambaran Umum Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Harga &amp; Pendaftaran</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->

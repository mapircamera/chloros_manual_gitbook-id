---
metaLinks: {}
---

# Memulai

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros adalah aplikasi perangkat lunak dari [MAPIR](https://www.mapir.camera) untuk memproses gambar dan data sensor lainnya.

***{% hint style="success" %}**Apa yang Baru di Chloros 1.1.0**: Dukungan Linux asli (amd64 dan arm64), komputasi tepi NVIDIA Jetson, Adaptasi Komputasi Dinamis, pipa pemrosesan 4-thread, perintah dan opsi CLI baru. Lihat [Unduh](download.md) untuk catatan perubahan lengkap.
{% endhint %}

Chloros tersedia dalam 3 mode aplikasi:

## Chloros: Aplikasi GUI desktop

Jendela terpisah mandiri dengan semua fitur. _Hanya untuk Windows._

## [Chloros CLI: Antarmuka baris perintah](CLI.md)

Pemrosesan batch baris perintah. Sempurna untuk otomatisasi, penulisan skrip, dan operasi tanpa antarmuka grafis. Tersedia di **Windows, Linux amd64, dan Linux arm64 (NVIDIA Jetson)**. _CLI memerlukan lisensi Chloros+ untuk mengaksesnya._

## [Chloros API: Python SDK](api-python-sdk.md)

Antarmuka Python berbasis pemrograman untuk otomatisasi dan alur kerja kustom. Cocok untuk alur kerja penelitian, integrasi dengan aplikasi Python yang sudah ada, dan pembuatan alat kustom. Tersedia di **semua platform** melalui `pip install chloros-sdk`. _API memerlukan lisensi Chloros+ untuk mengaksesnya._***

## Platform yang Didukung

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Ya | Ya | Ya |
| **Linux amd64 (x86_64)** | Tidak | Ya | Ya |
| **Linux arm64 (NVIDIA Jetson)** | Tidak | Ya | Ya |

Untuk petunjuk instalasi Linux, lihat bagian [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Meskipun Chloros dapat digunakan secara gratis untuk sebagian besar tugas, Anda mungkin merasa membutuhkan lebih banyak fitur. Di sinilah lisensi berbayar untuk Chloros+ dapat bermanfaat bagi Anda. Dengan lisensi Chloros+, Anda dapat mengakses fitur-fitur baru seperti:

* **Pemrosesan Multi-threaded**: mempercepat pemrosesan gambar secara signifikan untuk proyek-proyek besar dengan memproses gambar secara bersamaan melalui pipeline.
* **Akselerasi GPU (CUDA)**: manfaatkan opsi memori GPU yang lebih tinggi saat ini untuk semakin mempercepat alur kerja pemrosesan gambar. Kami merekomendasikan VRAM 4GB atau lebih untuk hasil terbaik.
* **Chloros+**[**CLI**](CLI.md)**Akses**: jalankan Chloros+ dari baris perintah untuk mengotomatisasi dan mengintegrasikannya ke dalam perangkat lunak Anda sendiri.
* **Chloros+**[**API**](api-python-sdk.md)**Akses:** jalankan Chloros+ dari Python untuk kontrol terprogram, yang memungkinkan integrasi mulus dengan alur kerja penelitian, alur kerja analisis data, dan aplikasi khusus Anda.
* **Penggunaan Multi-Perangkat**: setiap lisensi Chloros+ memungkinkan pendaftaran 2+ perangkat. Gunakan akun MAPIR Cloud Anda untuk mengelola perangkat yang terdaftar. Tambahkan dukungan untuk lebih banyak perangkat dengan meng-upgrade lisensi Chloros+ Anda.
* **Metode Debayer Berbasis Tekstur Lanjutan:** metode debayer berkualitas tinggi yang peka terhadap tepi, dikombinasikan dengan model denoising AI/ML yang menghilangkan hampir seluruh noise debayering. 
* **Rumus Indeks Multispektral Kustom:** masukkan rumus indeks multispektral kustom di kalkulator raster Chloros, baik untuk pemrosesan maupun ruang uji tampilan gambar.
* **Linux &amp; Komputasi Tepi:** jalankan Chloros pada platform Linux x86_64 dan ARM64, termasuk NVIDIA Jetson, untuk pemrosesan lapangan dan tepi. Lihat [Ikhtisar Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Harga &amp; Pendaftaran</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

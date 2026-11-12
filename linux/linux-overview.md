# Gambaran Umum Linux

Chloros 1.3.0 menyediakan dukungan Linux bawaan untuk **CLI**dan**Python SDK** — pemrosesan citra multispektral tanpa antarmuka grafis, serta kontrol langsung terhadap kamera LATTICE dan sensor cahaya DAQ — pada workstation Linux, server, dan perangkat tepi NVIDIA Jetson.

{% hint style="info" %}
**Tidak ada antarmuka GUI desktop di Linux.**Antarmuka GUI Desktop Chloros hanya tersedia untuk Windows. Pengguna Linux berinteraksi dengan Chloros melalui [CLI](../CLI.md) dan [Python SDK](../api-python-sdk.md). `.deb` memang menambahkan entri**Chloros CLI** ke menu aplikasi Anda — entri ini hanya membuka emulator terminal yang menjalankan `chloros-cli`.
{% endhint %}

***

## Matriks Dukungan Platform

| Fitur | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI Desktop** | Ya | N/A | Tidak | Tidak |
| **Antarmuka Baris Perintah (CLI)** (`chloros-cli`) | Ya | Ya | Ya | Ya |
| **Python SDK** (`chloros-sdk`) | Ya | Ya | Ya | Ya |
| **Pipa pemrosesan gambar** | Ya | Ya | Ya | Ya |
| **Kontrol kamera LATTICE (langsung)** | Ya (tab Kamera) | Ya (`chloros-cli lattice`, SDK) | Ya | Ya |
| **Sensor cahaya DAQ (langsung)** | Ya (tab Sensor Cahaya) | Ya (`chloros-cli daq pool-*`, SDK) | Ya | Ya |
| **Sinkronisasi waktu PTP (host sebagai grandmaster)** | Ya | Ya (`chloros-cli time-sync`) | Ya | Ya |
| **Akselerasi GPU (CUDA)** | Ya | Ya | Ya | Ya (JetPack 6) |
| **Debayer yang mendukung tekstur** (membutuhkan GPU CUDA) | Ya (Chloros+) | Ya (Chloros+) | Ya (Chloros+) | Ya (Chloros+) |
| **Adaptasi Komputasi Dinamis** | Ya | Ya | Ya | Ya |
| **Backend sebagai layanan sistem** (`chloros-backend@<user>.service`) | Tidak | Tidak | Ya (pilihan) | Ya (pilihan) |
| **Pembaruan di tempat** (`chloros-cli update`) | Tidak (jalankan penginstal) | Tidak (jalankan penginstal) | Ya | Ya |

***

## Arsitektur yang Didukung

| Arsitektur | Deskripsi | Paket |
| --- | --- | --- |
| **amd64 (x86_64)** | Prosesor desktop/server standar (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Prosesor ARM — Keluarga NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Distribusi Linux yang Didukung

* **Ubuntu 22.04 LTS atau yang lebih baru** (amd64)
* **Debian 12 atau yang lebih baru** (amd64)
* **NVIDIA JetPack 6** (arm64 — platform Jetson Orin)

***

## Apa yang Didapatkan Pengguna Linux

* **Chloros CLI** — antarmuka baris perintah lengkap untuk pemrosesan batch, otomatisasi, dan pembuatan skrip
* **Chloros Python SDK** — antarmuka Python berbasis pemrograman untuk alur kerja penelitian dan alat khusus, diinstal oleh `.deb` untuk sistem `python3` (wheel yang sesuai versi untuk virtualenvs terdapat di `/usr/lib/chloros/sdk/`; PyPI diperuntukkan bagi mesin yang tidak memiliki paket Chloros)
* **Kontrol kamera LATTICE** — temukan, hubungkan, konfigurasikan, dan ambil gambar dari kamera LATTICE serta susunan multi-kamera yang disinkronkan melalui `chloros-cli lattice` dan SDK; `.deb` menyertakan runtime Arena SDK yang dibutuhkan oleh kamera-kamera tersebut
* **Kontrol sensor cahaya DAQ** — hubungkan sensor DAQ-U/M/E, alirkan spektrum yang telah dikalibrasi, dan rekam berkas `.daq` melalui `chloros-cli daq pool-*` dan SDK
* **Sinkronisasi waktu PTP** — backend Chloros menjalankan grandmaster PTP yang menjadi acuan bagi kamera LATTICE dan sensor DAQ-E; periksa dengan `chloros-cli time-sync`, dan jaga agar tetap berjalan tanpa antarmuka pengguna (headless) menggunakan unit systemd `chloros-backend@<user>.service` (lihat [Instalasi Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Otomatisasi proyek** — jalankan proyek yang disimpan secara headless dengan `chloros-cli project` dan `open_project` dari SDK
* **Akselerasi GPU** — pemrosesan yang dipercepat dengan CUDA pada GPU NVIDIA (GPU desktop pada amd64, dan GPU Jetson dengan JetPack 6)
* **Adaptasi Komputasi Dinamis** — deteksi perangkat keras otomatis dan pemilihan strategi pemrosesan, dengan opsi penggantian `CHLOROS_STRATEGY` sebagai jalan keluar bagi pengguna ahli
* **Semua fitur pemrosesan** — alur kerja yang sama seperti di Windows: kalibrasi, koreksi vignette, indeks vegetasi, dan semua format ekspor
* **Fitur Chloros+** — pemrosesan multi-threaded (pipelined), Texture Aware debayer (pada GPU CUDA), dan indeks khusus, dengan paket Chloros+ berbayar

## Apa yang Tidak Didapatkan Pengguna Linux

* **Antarmuka Grafis Desktop** — tidak ada antarmuka grafis; semua interaksi dilakukan melalui CLI atau Python SDK
* **Penampil Gambar** — tidak ada penampil gambar interaktif, tampilan kisi, atau penanda peta
* **Manajemen proyek visual** — proyek dibuat dan dijalankan melalui perintah CLI dan panggilan SDK (perangkat keras itu sendiri — kamera, sensor, pengambilan data — tetap dapat dikendalikan sepenuhnya dari terminal)

***

## Persyaratan Lisensi

Akses CLI dan SDK memerlukan **tingkatan Chloros+ berbayar — Copper atau lebih tinggi**(Copper, Bronze, Silver, Gold). Tingkatan gratis**Iron** tidak memiliki akses CLI/SDK. Batasan ini diterapkan oleh backend, bukan hanya oleh CLI:

| Situasi | Respons backend |
| --- | --- |
| Belum masuk | `401` dengan `error_code: AUTH_REQUIRED` |
| Masuk pada tingkatan Iron gratis | `403` dengan `error_code: PLAN_UPGRADE_REQUIRED` |

`chloros-cli status` berfungsi di semua tingkatan — ini adalah satu-satunya rute yang dikecualikan dari pembatasan — sehingga alasan penolakan selalu terlihat.

***

## Memulai di Linux

1. **Instal Chloros** — lihat [Instalasi Linux](linux-installation.md) untuk proses instalasi `.deb`
2. **Verifikasi** — `chloros-cli --version` menampilkan `Chloros CLI 1.3.0`; `chloros-cli selftest` menjalankan diagnostik 7 langkah
3. **Periksa Python SDK** — `.deb` menginstalnya: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (di dalam virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Masuk** — `chloros-cli login your@email.com 'your-password'` (sekali per mesin; pembaruan paket mempertahankan data login, menghapus paket akan menghapusnya)
5. **Proses dataset pertama Anda** — `chloros-cli process ~/datasets/flight001`

Untuk NVIDIA Jetson, lihat [Panduan NVIDIA Jetson](nvidia-jetson-guide.md) khusus untuk pengaturan platform, perilaku termal, dan penerapan di lapangan.

***

## Langkah Selanjutnya

* [Instalasi Linux](linux-installation.md) — panduan instalasi terperinci, lokasi berkas, dan pemecahan masalah untuk amd64 dan arm64
* [Panduan NVIDIA Jetson](nvidia-jetson-guide.md) — pengaturan khusus Jetson, perilaku memori dan termal, serta penerapan di lapangan
* [CLI : Baris Perintah](../CLI.md) — panduan CLI
* [API : Python SDK](../api-python-sdk.md) — panduan SDK
* [Referensi CLI](../reference/cli-reference.md) dan [Referensi SDK](../reference/sdk-reference.md) — daftar lengkap perintah/API untuk versi 1.3.0
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — cara Chloros beradaptasi dengan perangkat keras Anda

{% hint style="info" %}
**Membaca panduan ini secara terprogram.** Setiap halaman juga disajikan sebagai Markdown mentah di URL-nya masing-masing ditambah `.md` (misalnya `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), dan indeks seluruh manual dipublikasikan di [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}

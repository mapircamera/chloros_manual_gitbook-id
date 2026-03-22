# Gambaran Umum Linux

Chloros 1.1.0 menghadirkan dukungan asli Linux untuk **CLI**dan**Python SDK**, sehingga memungkinkan pemrosesan gambar multispektral tanpa antarmuka pengguna (headless) pada workstation, server, dan perangkat tepi NVIDIA Jetson.

{% hint style="info" %}
**Tidak ada GUI pada Linux.** GUI Desktop Chloros hanya tersedia pada Windows. Pengguna Linux berinteraksi dengan Chloros melalui [CLI](../CLI.md) dan [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matriks Dukungan Platform

| Fitur | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Antarmuka Pengguna Desktop** | Ya | N/A | Tidak | Tidak |
| **CLI** | Ya | Ya | Ya | Ya |
| **Python SDK** | Ya | Ya | Ya | Ya |
| **Akselerasi GPU (CUDA)** | Ya | Ya | Ya | Ya (JetPack 6) |
| **Debayer yang Mendukung Tekstur** | Ya (Chloros+) | Ya (Chloros+) | Ya (Chloros+) | Ya (Chloros+) |
| **Adaptasi Komputasi Dinamis** | Ya | Ya | Ya | Ya |***

## Arsitektur yang Didukung

| Arsitektur | Deskripsi | Metode Instalasi |
| --- | --- | --- |
| **amd64 (x86_64)** | Prosesor desktop/server standar (Intel, AMD) | Paket `.deb` |
| **arm64 (aarch64)** | Prosesor berbasis ARM, terutama NVIDIA Jetson | Paket `.deb` (JetPack 6) |

## Distribusi Linux yang Didukung

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — platform Jetson)***

## Apa yang Didapatkan Pengguna Linux

* **Chloros CLI** — Antarmuka baris perintah lengkap untuk pemrosesan batch, otomatisasi, dan penulisan skrip
* **Chloros Python SDK** — Antarmuka Python berbasis pemrograman (`pip install chloros-sdk`) untuk integrasi ke dalam alur kerja penelitian dan alat kustom
* **Akselerasi GPU** — Pemrosesan yang dipercepat oleh CUDA pada GPU NVIDIA (desktop dan Jetson)
* **Adaptasi Komputasi Dinamis** — Deteksi perangkat keras otomatis dan optimasi strategi pemrosesan
* **Semua Fitur Pemrosesan** — Jalur pemrosesan multispektral yang sama seperti Windows (kalibrasi, koreksi vignette, indeks vegetasi, semua format ekspor)
* **Fitur Chloros+** — Pemrosesan multi-threaded, debayer yang mendukung tekstur, indeks kustom (dengan lisensi Chloros+)

## Apa yang Tidak Didapatkan Pengguna Linux

* **Antarmuka Pengguna Desktop** — Tidak ada antarmuka grafis; semua interaksi dilakukan melalui CLI atau Python SDK
* **Penerjemah Gambar** — Tidak ada penerjemah gambar interaktif, tampilan grid, atau penanda peta
* **Manajemen Proyek Visual** — Proyek dikelola melalui perintah CLI dan panggilan SDK***

## Memulai dengan Linux

1. **Instal Chloros** — Lihat [Instalasi Linux](linux-installation.md) untuk instalasi paket `.deb`
2. **Instal Python dan SDK** (opsional) — `pip install chloros-sdk`
3. **Aktifkan lisensi Anda** — `chloros-cli login your@email.com 'password'`
4. **Proses dataset pertama Anda** — `chloros-cli process ~/datasets/flight001`

Bagi pengguna NVIDIA Jetson, lihat [Panduan NVIDIA Jetson](nvidia-jetson-guide.md) khusus untuk pengaturan dan optimasi platform.

***

## Langkah Selanjutnya

* [Instalasi Linux](linux-installation.md) — Petunjuk instalasi terperinci untuk amd64 dan arm64
* [Panduan NVIDIA Jetson](nvidia-jetson-guide.md) — Pengaturan khusus Jetson, manajemen termal, dan penerapan di lapangan
* [CLI : Baris Perintah](../CLI.md) — Referensi lengkap CLI
* [API : Python SDK](../api-python-sdk.md) — Referensi lengkap SDK
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — Bagaimana Chloros menyesuaikan diri dengan perangkat keras Anda

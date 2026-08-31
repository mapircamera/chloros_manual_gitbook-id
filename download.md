---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Unduh

Unduh versi terbaru Chloros untuk memulai pemrosesan citra multispektral.

### Persyaratan Sistem

#### Windows

| Persyaratan          | Minimum                                              | Direkomendasikan                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistem Operasi** | Windows 10 (64-bit)                                  | Windows 11 (64-bit)                                  |
| **Prosesor**        | Intel Core i5 atau yang setara                          | Intel Core i7 atau yang lebih baik                              |
| **Memori (RAM)**     | 8GB                                                  | 16GB atau lebih                                         |
| **Kartu Grafis**    | Kompatibel dengan DirectX 11                                | GPU NVIDIA dengan VRAM 4GB+                            |
| **Penyimpanan**          | Ruang kosong 6GB                                       | SSD dengan ruang kosong 10GB+                            |
| **Layar**          | 1920x1080                                            | 2560x1440 atau lebih tinggi                                  |
| **Internet**         | Diperlukan untuk aktivasi lisensi \[opsional\] Chloros+ | Diperlukan untuk aktivasi lisensi \[opsional\] Chloros+ |

#### Linux amd64 (x86\_64)

| Persyaratan       | Minimum                    | Direkomendasikan               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribusi**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prosesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 atau yang lebih baik   |
| **Memori (RAM)**  | 8GB                        | 16GB atau lebih              |
| **Kartu Grafis** | Tidak ada (diproses oleh CPU)      | GPU NVIDIA dengan VRAM 4GB+ |
| **Penyimpanan**       | Ruang kosong 2GB             | SSD dengan ruang kosong 10GB+       |
| **Python**        | Python 3.7+ (untuk SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Persyaratan      | Minimum                      | Direkomendasikan                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson dengan JetPack 6 | Jetson Orin NX 16GB atau AGX Orin |
| **Memori (RAM)** | 8GB (bersama GPU/CPU)         | 16GB+ bersama                    |
| **Penyimpanan**      | Ruang kosong 2GB               | SSD NVMe dengan ruang kosong 10GB+        |
| **Python**       | Python 3.7+ (untuk SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Akselerasi GPU**: Pengguna Chloros+ dengan GPU NVIDIA dapat menggunakan akselerasi CUDA untuk pemrosesan yang jauh lebih cepat. Fitur ini berfungsi baik pada Windows (GPU desktop) maupun Linux (GPU desktop dan NVIDIA Jetson). Pengguna Chloros+ juga mendapatkan pemrosesan multi-thread untuk kecepatan maksimum.
{% endhint %}

***

## Unduh Chloros

### Rilis Stabil Terbaru: Versi 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Unduh Chloros untuk Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Unduh Chloros untuk Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Unduh Chloros untuk Linux arm64 / Jetson (.deb)</a>

#### Penginstal Windows (GUI + CLI + Backend)

* **Tipe File**: .exe (Penginstal Windows)**Langkah-langkah Instalasi:**

1. Unduh berkas .exe di atas
2. Klik dua kali penginstal untuk memulai instalasi
3. Ikuti petunjuk wizard instalasi
4. Pilih direktori instalasi (default: `C:\Program Files\MAPIR\Chloros\`)
5. Selesaikan instalasi dan jalankan Chloros atau Chloros CLI
6. Masuk dengan [akun MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (atau lanjutkan dengan versi gratis)

{% hint style="success" %}
Penginstal secara otomatis menambahkan `chloros-cli` ke PATH sistem Anda untuk akses baris perintah.
{% endhint %}

#### Linux amd64 (Paket .deb — CLI + Backend)

* **Jenis File**: .deb (paket Debian/Ubuntu)
* **Arsitektur**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (Paket .deb — CLI + Backend)

* **Jenis File**: .deb (JetPack 6)
* **Arsitektur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Lihat [Instalasi Linux](linux/linux-installation.md) untuk petunjuk pengaturan terperinci dan [Panduan NVIDIA Jetson](linux/nvidia-jetson-guide.md) untuk panduan khusus Jetson.

#### Python SDK (Semua Platform)

Setiap penginstal menyertakan roda `chloros_sdk` yang sesuai, sehingga versi SDK selalu sesuai dengan GUI/CLI/backend yang terinstal. Pada Windows, penginstal akan menginstalnya ke dalam sistem Anda Python secara otomatis; pada Linux, `.deb` menempatkan wheel di `/usr/lib/chloros/sdk/` dan menampilkan perintah instalasi:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Untuk host yang hanya menggunakan pip (tidak ada paket Chloros yang terinstal), SDK juga tersedia di PyPI:

```bash
pip install chloros-sdk
```

Lihat [API : Python SDK](api-python-sdk.md) dan [Referensi SDK](reference/sdk-reference.md) untuk dokumentasi.

{% hint style="info" %}
**Pengguna Linux**: Paket `.deb` menginstal CLI dan backend. Tidak ada antarmuka grafis (GUI) untuk Linux — semua interaksi dilakukan melalui CLI atau SDK.
{% endhint %}

***

## Sumber Daya Tambahan

### Python SDK

Untuk pengembang dan alur kerja otomatisasi, instal Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentasi**: [API: Python SDK](api-python-sdk.md)**Persyaratan**: Chloros harus sudah terinstal (penginstal Windows atau paket Linux `.deb`), diperlukan login lisensi Chloros+***

## Yang Termasuk

### Penginstal Windows

* ✅ **Chloros GUI** - Antarmuka grafis dengan fitur lengkap
* ✅ **Chloros CLI** - Antarmuka baris perintah (memerlukan lisensi Chloros+)
* ✅ **Chloros Backend** - Mesin pemrosesan
* ✅ **Profil Kamera** - Template kamera MAPIR yang telah dikonfigurasi sebelumnya

### Paket .deb Linux

* ✅ **Chloros CLI** - Antarmuka baris perintah (memerlukan lisensi Chloros+)
* ✅ **Chloros Backend** - Mesin pemrosesan
* ✅ **Profil Kamera** - Template kamera MAPIR yang telah dikonfigurasi sebelumnya
* ❌ Tanpa GUI — Linux hanya tersedia dalam versi headless CLI/SDK

### Python SDK (pip, semua platform)

* ✅ **Chloros SDK** - Python API (memerlukan lisensi Chloros+)***

## Tingkatkan ke Chloros+

Buka fitur-fitur lanjutan dengan berlangganan Chloros+:

* 🚀 **Pemrosesan Multi-threaded** - Memproses gambar secara paralel
* ⚡ **Akselerasi GPU (CUDA)** - Memanfaatkan kekuatan GPU NVIDIA
* 💻 **Akses CLI** - Otomatisasi dengan alat baris perintah
* 🐍 **Python SDK** - Akses API secara terprogram
* 📱 **Beberapa Perangkat** - Dapat digunakan pada 2–10+ perangkat (tergantung paket langganan)
* **🐻 Metode Debayer Canggih yang Memperhatikan Tekstur** - debayer berkualitas tinggi yang memperhatikan tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir semua noise debayering.
* 🧮 **Rumus Kustom** - Buat indeks multispektral kustom

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Lihat Paket &amp; Harga Chloros+</a></p>***

## Bantuan Instalasi

### Pemecahan Masalah

**Instalasi gagal dengan pesan kesalahan:**

* Pastikan Anda memiliki hak administrator
* Nonaktifkan sementara perangkat lunak antivirus
* Pastikan sistem Anda memenuhi persyaratan sistem minimum

**Aplikasi tidak dapat dijalankan (Windows):**

* Pastikan Windows 10/11 (64-bit) telah terinstal
* Perbarui driver grafis
* Periksa Penampil Peristiwa Windows untuk detail kesalahan
* Hubungi dukungan dengan log kesalahan

**CLI tidak dapat dijalankan (Linux):**

* Pastikan paket `.deb` telah terinstal dengan benar: `dpkg -l | grep chloros`
* Periksa izin: `sudo chmod +x /usr/bin/chloros-cli`
* Jalankan diagnostik: `chloros-cli selftest`
* Periksa apakah ada pustaka yang hilang: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Masalah aktivasi lisensi:**

* Pastikan koneksi internet aktif
* Verifikasi kredensial di [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Periksa apakah firewall tidak memblokir Chloros
* Lihat [Chloros+ Login](chloros+-login.md) untuk petunjuk terperinci

### Mendapatkan Dukungan

Butuh bantuan untuk instalasi atau pengaturan?

* 📧 **Email**: info@mapir.camera
* 🌐 **Situs Web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentasi**: [Memulai](./)
* ❓ **FAQ**: [Pertanyaan yang Sering Diajukan](faq.md)***

## Pembaruan Perangkat Lunak

Chloros memeriksa ketersediaan pembaruan, memberi tahu saat versi baru tersedia, dan menautkan ke halaman unduhan ini — Anda dapat memperbarui dengan menjalankan penginstal baru yang telah ditandatangani. Pengaturan dan proyek Anda tetap utuh setelah pembaruan. Pada Linux dan Jetson, `chloros-cli update` memeriksa versi yang lebih baru dan menawarkan untuk mengunduh serta menginstal `.deb` yang sesuai (perintah ini hanya tersedia di Linux).

***

## Catatan Perubahan**Versi 1.2.0 (Terbaru)**— lihat**Apa yang Baru di Chloros 1.2.0** di halaman [Memulai](./) untuk daftar fitur lengkap.

<details>

<summary>Versi 1.0.5</summary>

**Tanggal Rilis: 10 Februari 2026**

**Fitur Baru*** **Metode Debayer yang Memperhatikan Tekstur \[Hanya Chloros+] -** Metode yang Memperhatikan Tekstur menggunakan debayer berkualitas tinggi yang peka terhadap tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir seluruh noise debayering.
* **Dukungan untuk Target Kalibrasi T4P*** **Pemrosesan GPU Chloros+ yang lebih cepat, manajemen memori yang lebih baik**

**Perbaikan Bug*** Antarmuka pengguna (GUI) yang sepenuhnya baru, kini seharusnya dapat berjalan di semua komputer Windows.

</details>

<details>

<summary>Versi 1.0.4</summary>

**Tanggal Rilis: 5 Januari 2026**

**Fitur Baru*** **Toggle Gambar/Metadata**: Ditambahkan tombol toggle di Penjelajah File untuk menampilkan metadata gambar yang dipilih dalam bentuk tabel, bukan dalam kisi gambar
* **Penggeser Zoom Kisi Gambar**: Penggeser UI baru untuk menyesuaikan ukuran thumbnail (juga mendukung CTRL + roda mouse)
* **Tombol Ekspor Kisi Gambar**: Tombol di baris atas untuk mengganti thumbnail dari format JPG ke ekspor yang telah diproses (Target, Reflektansi, Indeks, LUT)
* **Tab Peta**: Peta 2D interaktif baru yang menampilkan penanda lokasi GPS pada gambar
  * Mendukung Google Maps dan ubin peta ESRI (secara otomatis memilih layanan ubin terbaik berdasarkan ketersediaan tingkat zoom)
  * Pratinjau thumbnail saat mengarahkan kursor ke penanda peta

**Perbaikan Bug*** Dukungan yang ditingkatkan untuk menginstal Chloros pada komputer dengan bahasa selain bahasa Inggris

</details>

<details>

<summary>Versi 1.0.3</summary>

**Tanggal Rilis: 20 Desember 2025**

**Fitur Baru*** Peluncuran Awal

**Peningkatan*** Peluncuran Awal

**Perbaikan Bug*** Peluncuran Awal

**Masalah yang Diketahui*** Peluncuran Awal

</details>***

## Perjanjian Lisensi**Perangkat Lunak Hak Milik** - Hak Cipta (c) 2026 MAPIR Inc.

Dilarang menggunakan, mendistribusikan, atau memodifikasi tanpa izin.

**Versi Gratis**: Tersedia untuk penggunaan pribadi dan komersial dengan batasan fitur**Chloros+**: Lisensi berbasis langganan untuk fitur lanjutan dan penerapan komersial

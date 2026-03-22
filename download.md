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
| **Prosesor**        | Intel Core i5 atau setara                          | Intel Core i7 atau lebih baik                              |
| **Memori (RAM)**     | 8GB                                                  | 16GB atau lebih                                         |
| **Kartu Grafis**    | Kompatibel dengan DirectX 11                                | GPU NVIDIA dengan VRAM 4GB+                            |
| **Penyimpanan**          | Ruang kosong 6GB                                       | SSD dengan ruang kosong 10GB+                            |
| **Layar**          | 1920x1080                                            | 2560x1440 atau lebih tinggi                                  |
| **Internet**         | Diperlukan untuk aktivasi lisensi \[opsional\] Chloros+ | Diperlukan untuk aktivasi lisensi \[opsional\] Chloros+ |

#### Linux amd64 (x86_64)

| Persyaratan       | Minimum                    | Direkomendasikan               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribusi**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Prosesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 atau lebih baik   |
| **Memori (RAM)**  | 8GB                        | 16GB atau lebih              |
| **Kartu Grafis** | Tidak diperlukan (pemrosesan CPU)      | GPU NVIDIA dengan VRAM 4GB+ |
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
**Akselerasi GPU**: Pengguna Chloros+ dengan GPU NVIDIA dapat menggunakan akselerasi CUDA untuk pemrosesan yang jauh lebih cepat. Fitur ini berfungsi pada Windows (GPU desktop) dan Linux (GPU desktop dan NVIDIA Jetson). Pengguna Chloros+ juga mendapatkan pemrosesan multi-threaded untuk kecepatan maksimum.
{% endhint %}

***

## Unduh Chloros

### Rilis Stabil Terbaru (23 Maret 2026): Versi 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Unduh Chloros untuk Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Unduh Chloros untuk Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Unduh Chloros untuk Linux arm64 / Jetson (.deb)</a>

#### Penginstal Windows (GUI + CLI + Backend)

* **Tipe File**: .exe (Penginstal Windows)**Langkah-langkah Instalasi:**

1. Unduh file .exe di atas
2. Klik dua kali penginstal untuk memulai instalasi
3. Ikuti petunjuk wizard instalasi
4. Pilih direktori instalasi (default: `C:\Program Files\[USER]\Chloros\`)
5. Selesaikan instalasi dan jalankan Chloros atau Chloros CLI
6. Masuk dengan akun [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (atau lanjutkan dengan versi gratis)

{% hint style="success" %}
Penginstal secara otomatis menambahkan `chloros-cli` ke PATH sistem Anda untuk akses baris perintah.
{% endhint %}

#### Linux amd64 (Paket .deb — CLI + Backend)

* **Tipe File**: .deb (paket Debian/Ubuntu)
* **Arsitektur**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (Paket .deb — CLI + Backend)

* **Tipe File**: .deb (JetPack 6)
* **Arsitektur**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Lihat [Instalasi Linux](linux/linux-installation.md) untuk petunjuk pengaturan terperinci dan [Panduan NVIDIA Jetson](linux/nvidia-jetson-guide.md) untuk panduan khusus Jetson.

#### Python SDK (Semua Platform)

```bash
pip install chloros-sdk
```

Lihat [API : Python SDK](api-python-sdk.md) untuk dokumentasi.

{% hint style="info" %}
**Pengguna Linux**: Paket `.deb` menginstal CLI dan backend. Python dan SDK diinstal secara terpisah melalui pip. Tidak ada antarmuka grafis (GUI) untuk Linux — semua interaksi dilakukan melalui CLI atau SDK.
{% endhint %}

***

## Sumber Daya Tambahan

### Python SDK

Untuk pengembang dan alur kerja otomatisasi, instal Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentasi**: [API: Python SDK](api-python-sdk.md)**Persyaratan**: Chloros harus terinstal (penginstal Windows atau paket Linux `.deb`), diperlukan login lisensi Chloros+***

## Yang Termasuk

### Penginstal Windows

* ✅ **Chloros GUI** - Antarmuka grafis berfitur lengkap
* ✅ **Chloros CLI** - Antarmuka baris perintah (membutuhkan lisensi Chloros+)
* ✅ **Chloros Backend** - Mesin pemrosesan
* ✅ **Profil Kamera** - Template kamera MAPIR yang telah dikonfigurasi sebelumnya

### Paket .deb Linux

* ✅ **Chloros CLI** - Antarmuka baris perintah (memerlukan lisensi Chloros+)
* ✅ **Chloros Backend** - Mesin pemrosesan
* ✅ **Profil Kamera** - Template kamera MAPIR yang telah dikonfigurasi sebelumnya
* ❌ Tanpa GUI — Linux hanya tersedia dalam mode headless CLI/SDK

### Python SDK (pip, semua platform)

* ✅ **Chloros SDK** - Python API (membutuhkan lisensi Chloros+)***

## Upgrade ke Chloros+

Buka fitur-fitur lanjutan dengan langganan Chloros+:

* 🚀 **Pemrosesan Multi-threaded** - Memproses gambar secara paralel
* ⚡ **Akselerasi GPU (CUDA)** - Manfaatkan kekuatan GPU NVIDIA
* 💻 **Akses CLI** - Otomatisasi dengan alat baris perintah
* 🐍 **Python SDK** - Akses programatik ke API
* 📱 **Perangkat Ganda** - Gunakan pada 2-10+ perangkat (tergantung paket)
* **🐻 Metode Debayer Canggih yang Sensitif terhadap Tekstur** - debayer berkualitas tinggi yang sensitif terhadap tepi, dikombinasikan dengan model penghilang noise AI/ML yang menghilangkan hampir semua noise debayering.
* 🧮 **Rumus Kustom** - Buat indeks multispektral kustom

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Lihat Paket &amp; Harga Chloros+</a></p>***

## Bantuan Instalasi

### Pemecahan Masalah

**Instalasi gagal dengan pesan kesalahan:**

* Pastikan Anda memiliki hak administrator
* Nonaktifkan sementara perangkat lunak antivirus
* Periksa apakah Anda memenuhi persyaratan sistem minimum

**Aplikasi tidak dapat dijalankan (Windows):**

* Pastikan Windows 10/11 (64-bit) terinstal
* Perbarui driver grafis
* Periksa Windows Event Viewer untuk detail kesalahan
* Hubungi dukungan dengan log kesalahan

**CLI tidak dapat dijalankan (Linux):**

* Pastikan paket `.deb` terinstal dengan benar: `dpkg -l | grep chloros`
* Periksa izin: `sudo chmod +x /usr/bin/chloros-cli`
* Jalankan diagnostik: `chloros-cli selftest`
* Periksa apakah ada pustaka yang hilang: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Masalah aktivasi lisensi:**

* Pastikan koneksi internet aktif
* Verifikasi kredensial di [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Pastikan firewall tidak memblokir Chloros
* Lihat [Chloros+ Login](chloros+-login.md) untuk petunjuk terperinci

### Mendapatkan Dukungan

Butuh bantuan untuk instalasi atau pengaturan?

* 📧 **Email**: info@mapir.camera
* 🌐 **Situs Web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentasi**: [Memulai](./)
* ❓ **FAQ**: [Pertanyaan yang Sering Diajukan](faq.md)***

## Catatan Perubahan

<details>

<summary>Versi 1.1.0 (Terbaru)</summary>

**Tanggal Rilis: Maret 2026**

**Fitur Baru*** **Dukungan Linux** — Dukungan asli untuk CLI dan SDK pada Linux amd64 (x86\_64) dan arm64 (NVIDIA Jetson JetPack 6). Instal melalui paket `.deb`.
* **Dukungan NVIDIA Jetson** — Pemrosesan yang dioptimalkan untuk perangkat tepi Jetson Nano, Orin Nano, Orin NX, dan AGX Orin.
* **Adaptasi Komputasi Dinamis** — Deteksi perangkat keras otomatis dan optimasi strategi pemrosesan. Chloros menyesuaikan diri dengan perangkat keras Anda, mulai dari Jetson Nano hingga workstation multi-GPU.
* **Pipa Pemrosesan 4-Thread** — Thread deteksi, kalibrasi, pemrosesan, dan ekspor yang berjalan secara bersamaan dengan alokasi memori GPU dinamis.
* **Perintah CLI Baru** — `selftest` (diagnostik sistem) dan `update` (pengelolaan pembaruan Linux).
* **Bendera Proses CLI Baru** — `--debayer` (standar/sensitif tekstur), `--indices` (tentukan indeks), `--target` (cari subfolder target terlebih dahulu untuk deteksi lebih cepat).
* **Item Menu GUI Baru** — Tambahkan File, Tambahkan Folder, dan Mulai/Hentikan Pemrosesan kini dapat diakses dari menu tarik-turun utama.**Peningkatan**

* Deteksi otomatis backend lintas platform (jalur Windows dan Linux)
* Peningkatan SDK dan `get_status()` dengan pelacakan kemajuan per-thread
* Kecualian baru SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Manajemen termal dan pembatasan adaptif untuk NVIDIA Jetson
* Manajemen memori otomatis dengan fallback OOM ke pemrosesan GPU berlapis

</details>

<details>

<summary>Versi 1.0.5</summary>

**Tanggal Rilis: 10 Februari 2026**

**Fitur Baru*** **Metode Debayer yang Mempertimbangkan Tekstur \[Hanya Chloros+] -** Metode ini menggunakan debayer berkualitas tinggi yang peka tepi, dikombinasikan dengan model denoising AI/ML yang menghilangkan hampir seluruh noise debayering.
* **Dukungan untuk Target Kalibrasi T4P*** **Pemrosesan GPU Chloros+ yang lebih cepat, manajemen memori yang lebih baik**

**Perbaikan Bug*** Antarmuka pengguna (GUI) yang sepenuhnya baru, seharusnya sudah berfungsi di semua komputer Windows sekarang.

</details>

<details>

<summary>Versi 1.0.4</summary>

**Tanggal Rilis: 5 Januari 2026**

**Fitur Baru*** **Toggle Gambar/Metadata**: Ditambahkan tombol toggle di File Browser untuk menampilkan metadata gambar yang dipilih dalam bentuk tabel, bukan grid gambar
* **Slider Zoom Grid Gambar**: Slider UI baru untuk menyesuaikan ukuran thumbnail (juga mendukung CTRL + roda mouse)
* **Tombol Ekspor Kisi Gambar**: Tombol di baris atas untuk mengganti thumbnail dari JPG ke ekspor yang telah diproses (Target, Reflektansi, Indeks, LUT)
* **Tab Peta**: Peta 2D interaktif baru yang menampilkan penanda lokasi GPS gambar
  * Mendukung Google Maps dan ubin peta ESRI (secara otomatis memilih layanan ubin terbaik berdasarkan ketersediaan tingkat zoom)
  * Pratinjau thumbnail saat mengarahkan kursor mouse ke penanda peta

**Perbaikan Bug*** Dukungan yang ditingkatkan untuk menginstal Chloros pada komputer dengan bahasa selain Inggris

</details>

<details>

<summary>Versi 1.0.3</summary>

**Tanggal Rilis: 20 Desember 2025**

**Fitur Baru*** Peluncuran Awal

**Peningkatan*** Peluncuran Awal

**Perbaikan Bug*** Peluncuran Awal

**Masalah yang Diketahui*** Peluncuran Awal

</details>***

## Perjanjian Lisensi**Perangkat Lunak Proprietary** - Hak Cipta (c) 2026 MAPIR Inc.

Penggunaan, distribusi, atau modifikasi tanpa izin dilarang.

**Versi Gratis**: Tersedia untuk penggunaan pribadi dan komersial dengan batasan fitur**Chloros+**: Lisensi berbasis langganan untuk fitur lanjutan dan penerapan komersial

---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Unduh

Unduh versi terbaru Chloros untuk memulai pemrosesan gambar multispektral.

### Persyaratan Sistem

| Persyaratan          | Minimum                                              | Direkomendasikan                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistem Operasi** | Windows 10 (64-bit)                                  | Windows 11 (64-bit)                                  |
| **Prosesor**        | Intel Core i5 atau setara                          | Intel Core i7 atau lebih baik                              |
| **Memori (RAM)**     | 8GB                                                  | 16GB atau lebih                                         |
| **Kartu Grafis**    | Kompatibel dengan DirectX 11                                | GPU NVIDIA dengan VRAM 4GB+                            |
| **Penyimpanan**          | Ruang kosong 6GB                                       | SSD dengan ruang kosong 10GB+                            |
| **Layar**          | 1920x1080                                            | 2560x1440 atau lebih tinggi                                  |
| **Internet**         | Diperlukan untuk aktivasi lisensi Chloros+ (opsional) | Diperlukan untuk aktivasi lisensi Chloros+ (opsional) |

{% hint style="info" %}
**Peningkatan GPU**: Pengguna Chloros+ dengan GPU NVIDIA dapat menggunakan akselerasi CUDA untuk pemrosesan yang jauh lebih cepat. Pengguna Chloros+ juga mendapatkan pemrosesan multi-threaded untuk kecepatan maksimum.
{% endhint %}

***

## Unduh Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Unduh Chloros di</a>

Sini

### Rilis Stabil Terbaru

**Pemasang Chloros untuk Windows*** **Versi**: 1.0.5
* **Tanggal Rilis**: 10 Februari 2026
* **Ukuran File (Unduh)**: 1,6 GB
* **Ukuran File (Terpasang)**: 5,7 GB
* **Tipe File**: .exe (Penginstal Windows)

#### **Langkah-langkah Instalasi:**

1. Unduh file `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Klik ganda penginstal untuk memulai instalasi
3. Ikuti petunjuk wizard instalasi
4. Pilih direktori instalasi (default: `C:\Program Files\[USER]\Chloros\`)
5. Selesaikan instalasi dan jalankan Chloros atau Chloros CLI
6. Masuk dengan akun [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (atau lanjutkan dengan versi gratis)

{% hint style="success" %}
Pemasang secara otomatis menambahkan `chloros-cli` ke jalur sistem PATH Anda untuk akses baris perintah.
{% endhint %}

***

## Sumber Daya Tambahan

### Python SDK

Bagi pengembang dan alur kerja otomatisasi, instal Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Dokumentasi**: [API: Python SDK](api-python-sdk.md)**Persyaratan**: Chloros Desktop harus diinstal, login lisensi Chloros+ diperlukan***

## Apa yang Termasuk

Pemasangan Chloros mencakup:

* ✅ **Chloros** - Antarmuka grafis (GUI) penuh fitur
* ✅ **Chloros CLI** - Antarmuka baris perintah (membutuhkan lisensi Chloros+)
* ✅ **Chloros SDK** - Python API (membutuhkan lisensi Chloros+)
* ✅ **Profil Kamera** - Template kamera MAPIR yang telah dikonfigurasi sebelumnya***

## Upgrade ke Chloros+

Buka fitur lanjutan dengan langganan Chloros+:

* 🚀 **Pemrosesan Multi-threaded** - Memproses gambar secara paralel
* ⚡ **Percepatan GPU (CUDA)** - Manfaatkan kekuatan GPU NVIDIA
* 💻 **Akses CLI** - Otomatisasi dengan alat baris perintah
* 🐍 **Python SDK** - Akses programatik API
* 📱 **Perangkat Multi** - Gunakan pada 2-10+ perangkat (tergantung pada paket)
* **🐻 Metode Debayer Berbasis Tekstur Canggih** - metode debayer berkualitas tinggi yang sadar tepi, dikombinasikan dengan model denoising AI/ML yang menghilangkan hampir semua noise debayering. 
* 🧮 **Rumus Kustom** - Buat indeks multispektral kustom

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Lihat Rencana &amp; Harga Chloros+</a></p>***

## Bantuan Instalasi

### Pemecahan Masalah

**Instalasi gagal dengan pesan kesalahan:**

* Pastikan Anda memiliki hak administrator
* Nonaktifkan sementara perangkat lunak antivirus
* Pastikan Anda memenuhi persyaratan sistem minimum

**Aplikasi tidak dapat dijalankan:**

* Pastikan Windows 10/11 (64-bit) terinstal
* Perbarui driver grafis
* Periksa Windows Event Viewer untuk detail kesalahan
* Hubungi dukungan dengan log kesalahan

**Masalah aktivasi lisensi:**

* Pastikan koneksi internet aktif
* Verifikasi kredensial di [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Periksa apakah firewall memblokir Chloros
* Lihat [Chloros+ Login](chloros+-login.md) untuk instruksi detail

### Mendapatkan Dukungan

Butuh bantuan dengan instalasi atau pengaturan?

* 📧 **Email**: info@mapir.camera
* 🌐 **Situs Web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentasi**: [Getting Started](./)
* ❓ **FAQ**: [Frequently Asked Questions](faq.md)***

## Catatan Perubahan

<details>

<summary>Versi 1.0.5</summary>

#### **Tanggal Rilis**: 10 Februari 2026**Fitur Baru*** **Metode Debayer Berbasis Tekstur \[Chloros+ Saja] -** Metode Debayer Berbasis Tekstur menggunakan teknik debayer berkualitas tinggi yang sensitif terhadap tepi, dikombinasikan dengan model denoising AI/ML yang menghilangkan hampir semua noise debayering.
* **Dukungan untuk Target Kalibrasi T4P*** **Pemrosesan GPU Chloros+ yang lebih cepat, manajemen memori yang lebih baik**

**Perbaikan Bug*** Antarmuka pengguna (GUI) yang sepenuhnya baru, sekarang seharusnya berfungsi di semua komputer Windows.

</details>

<details>

<summary>Versi 1.0.4</summary>

#### **Tanggal Rilis**: 5 Januari 2026**Fitur Baru*** **Toggle Gambar/Metadata**: Ditambahkan toggle di File Browser untuk menampilkan metadata gambar yang dipilih dalam tabel alih-alih grid gambar
* **Slider Zoom Grid Gambar**: Slider UI baru untuk menyesuaikan ukuran thumbnail (juga mendukung CTRL + roda mouse)
* **Tombol Ekspor Grid Gambar**: Tombol di baris atas untuk beralih antara thumbnail JPG dan ekspor yang diproses (Targets, Reflectance, Index, LUT)
* **Tab Peta**: Peta interaktif 2D baru yang menampilkan penanda lokasi GPS gambar
  * Mendukung Google Maps dan ubin peta ESRI (otomatis memilih layanan ubin terbaik berdasarkan ketersediaan tingkat zoom)
  * Pratinjau thumbnail saat mengarahkan kursor mouse ke penanda peta

**Perbaikan Bug*** Peningkatan dukungan untuk menginstal Chloros pada komputer dengan bahasa non-Inggris

</details>

<details>

<summary>Versi 1.0.3</summary>

#### **Tanggal Rilis**: 20 Desember 2025**Fitur Baru*** Peluncuran Awal

**Peningkatan*** Peluncuran Awal

**Perbaikan Bug*** Peluncuran Awal

**Masalah yang Diketahui*** Peluncuran Awal

</details>***

## Perjanjian Lisensi**Perangkat Lunak Proprietary** - Hak Cipta (c) 2026 MAPIR Inc.

Penggunaan, distribusi, atau modifikasi tanpa izin dilarang.

**Versi Gratis**: Tersedia untuk penggunaan pribadi dan komersial dengan batasan fitur**Chloros+**: Lisensi berlangganan untuk fitur lanjutan dan implementasi komersial

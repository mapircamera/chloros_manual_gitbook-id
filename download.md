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
| **Internet**         | Diperlukan untuk aktivasi lisensi \[opsional] Chloros+ | Diperlukan untuk aktivasi lisensi \[opsional] Chloros+ |

#### Linux amd64 (x86\_64)

| Persyaratan       | Minimum                    | Direkomendasikan               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribusi**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Prosesor**     | x86\_64 (Intel/AMD)        | Intel Core i7 atau yang lebih baik   |
| **Memori (RAM)**  | 8GB                        | 16GB atau lebih              |
| **Kartu Grafis** | Tidak diperlukan (diproses oleh CPU)      | GPU NVIDIA dengan VRAM 4GB+ |
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
**Akselerasi GPU**: Pengguna Chloros+ dengan GPU NVIDIA dapat menggunakan akselerasi CUDA untuk pemrosesan yang jauh lebih cepat. Fitur ini berfungsi dengan GPU desktop NVIDIA di Windows dan Linux amd64, serta pada NVIDIA Jetson dengan JetPack 6. Pengguna Chloros+ juga mendapatkan pemrosesan multi-threaded untuk kecepatan maksimal.
{% endhint %}

***

## Unduh Chloros


### Rilis Stabil Terbaru: Versi 1.3.0 (11 November 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Unduh Chloros untuk Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Unduh Chloros untuk Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Unduh Chloros untuk Linux arm64 / Jetson (.deb)</a>


#### Penginstal Windows (GUI + CLI + Backend)

* **Jenis File**: .exe (Penginstal Windows)

**Langkah-langkah Instalasi:**

1. Unduh berkas .exe di atas
2. Klik dua kali penginstal untuk memulai instalasi
3. Ikuti petunjuk wizard instalasi
4. Pilih direktori instalasi (default: `C:\Program Files\Chloros\`)
5. Pada halaman **Pilih Tugas Tambahan**, biarkan kedua opsi**Pengaturan sistem** tetap dicentang (disarankan — lihat di bawah)
6. Selesaikan instalasi dan jalankan Chloros atau Chloros CLI
7. Masuk dengan [akun MAPIR Cloud Chloros+ Anda](https://cloud.mapir.camera/pricing) (atau lanjutkan dengan versi gratis)

**Opsi pengaturan sistem** (keduanya dicentang secara default):

| Opsi, sebagaimana tertulis di penginstal | Fungsinya | Menonaktifkan instalasi diam-diam |
| --- | --- | --- |
| **Aktifkan jalur panjang Windows** (memungkinkan folder proyek berada di tingkat yang lebih dalam dari 260 karakter; disarankan) | Menetapkan pengaturan sistem Windows `LongPathsEnabled` secara global ke nilai 1. Windows secara default menonaktifkan pengaturan ini, sehingga jalur file tidak boleh melebihi 260 karakter; produk Chloros ditempatkan hingga sekitar 130 karakter di bawah folder proyek, sehingga jika opsi jalur panjang dinonaktifkan, folder proyek yang lebih dalam akan ditolak. Berlaku untuk program yang dijalankan setelah instalasi. Pengaturan ini tetap aktif setelah penghapusan instalasi, karena program lain mungkin bergantung padanya. | `/MERGETASKS="!longpaths"` |
| **Kecualikan backend Chloros dari pemindaian Microsoft Defender** (perekaman kamera jauh lebih cepat; disarankan) | Menambahkan pengecualian proses Microsoft Defender untuk `chloros-backend.exe` di folder instalasi, sehingga perlindungan real-time tidak memindai setiap file yang ditulis oleh backend, di mana pun file tersebut ditulis. Pada laju pengambilan gambar yang tinggi, pemindaian memakan waktu sekitar 30% dari laju pengambilan gambar (77,7 vs 52,7 pengambilan per detik dalam pengambilan gambar berkelanjutan dengan 9 kamera). Tidak ada program lain yang dikecualikan, dan pengecualian ini dihapus saat pencabutan instalasi. Jika Defender tidak ada atau diblokir oleh kebijakan, langkah ini gagal tanpa pemberitahuan dan proses instalasi berlanjut. | `/MERGETASKS="!defenderexclusion"` |

Instalasi diam-diam (`/SILENT` atau `/VERYSILENT`) menerapkan kedua opsi tersebut kecuali Anda memilih untuk tidak ikut serta dengan nilai `/MERGETASKS` yang ditampilkan.

{% hint style="success" %}
Penginstal menambahkan folder CLI (`C:\Program Files\Chloros\cli\`) ke sistem Anda `PATH` hanya sekali — penginstalan ulang atau peningkatan versi tidak lagi menambahkan entri duplikat, dan menghapus duplikat yang ditambahkan oleh penginstal sebelumnya. Buka terminal baru setelah menginstal untuk menggunakan `chloros-cli`. Penginstal juga menambahkan dua aturan Windows Firewall untuk backend, pada setiap profil jaringan: **Chloros Backend (UDP-In)**, untuk aliran kamera, PTP, mDNS, dan aliran data DAQ-E, serta**Chloros Backend (TCP-In)**, yang terbatas pada subnet lokal, untuk pembaruan firmware DAQ-E, di mana sensor terhubung kembali ke komputer ini untuk mengumpulkan gambar. Keduanya dihapus saat pencopotan instalasi.
{% endhint %}

#### Linux amd64 (Paket .deb — CLI + Backend)

* **Jenis File**: .deb (paket Debian/Ubuntu)
* **Arsitektur**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (Paket .deb — CLI + Backend)

* **Jenis File**: .deb (JetPack 6)
* **Arsitektur**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Gunakan nama file yang sebenarnya Anda unduh, dan pertahankan `./` agar `apt` menginstal file tersebut alih-alih mencari paket berdasarkan namanya. Lakukan instalasi dengan `apt install`, bukan `dpkg -i`: apt juga menginstal persyaratan Python SDK, yang direkomendasikan oleh paket tersebut alih-alih diwajibkan, dan `dpkg -i` melewatkannya. Sejak versi 1.3.0, backend dan CLI diinstal dalam bentuk yang sudah diekstrak di bawah `/usr/lib/chloros/`, sehingga keduanya dapat dijalankan tanpa perlu mengekstrak file terlebih dahulu.

Paket Jetson menggunakan build CUDA PyTorch dari JetPack 6, sehingga pemrosesan, termasuk Texture Aware debayer, berjalan di GPU Jetson. Paket ini menggunakan pustaka CUDA dan cuDNN yang diinstal oleh JetPack 6 (`sudo apt install nvidia-jetpack` pada gambar yang tidak memilikinya).

Lihat [Instalasi Linux](linux/linux-installation.md) untuk petunjuk pengaturan terperinci dan [Panduan NVIDIA Jetson](linux/nvidia-jetson-guide.md) untuk panduan khusus Jetson.

#### Python SDK (Semua Platform)

Setiap penginstal menyertakan wheel `chloros_sdk` yang sesuai, sehingga versi SDK selalu sesuai dengan GUI/CLI/backend yang terinstal. Di Windows, penginstal secara otomatis menginstalnya ke dalam Python sistem Anda. Di Linux, `.deb` menginstal SDK untuk sistem `python3` itu sendiri (ke dalam `/usr/lib/python3/dist-packages`), sehingga `import chloros_sdk` langsung berfungsi setelah instalasi, tanpa perlu pip maupun koneksi jaringan. Hanya virtualenv—yang tidak dapat mengakses paket sistem—yang memerlukan wheel yang disertakan untuk diinstal di dalamnya:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Untuk host yang hanya menggunakan pip (tidak ada paket Chloros yang terinstal; SDK kemudian menggerakkan backend di mesin lain), SDK juga tersedia di PyPI:

```bash
pip install chloros-sdk
```

Lihat [API: Python SDK](api-python-sdk.md) dan [Referensi SDK](reference/sdk-reference.md) untuk dokumentasi.

{% hint style="info" %}
**Pengguna Linux**: Paket `.deb` menginstal CLI, backend, dan Python SDK. Tidak ada antarmuka grafis (GUI) untuk Linux — semua interaksi dilakukan melalui CLI atau SDK.
{% endhint %}

***

## Sumber Daya Tambahan

### SDK Python


Untuk pengembang dan alur kerja otomatisasi, Chloros Python SDK disertakan dalam setiap penginstal: penginstal Windows menginstalnya ke dalam Python Anda, sedangkan `.deb` untuk Linux menginstalnya untuk sistem `python3`. Pada mesin yang tidak memiliki paket Chloros (mesin yang mengendalikan backend di mesin lain), instal dari PyPI:

```bash
pip install chloros-sdk
```

**Dokumentasi**: [API: Python SDK](api-python-sdk.md)

**Persyaratan**: backend Chloros yang akan dijalankan (penginstal Windows atau paket Linux `.deb` akan menginstalnya), serta akun Chloros+ pada tingkatan berbayar

***

## Yang Termasuk

### Penginstal Windows


* ✅ **GUI Chloros** - Antarmuka grafis dengan fitur lengkap
* ✅ **Chloros CLI** - Antarmuka baris perintah (memerlukan lisensi Chloros+)
* ✅ **Backend Chloros** - Mesin pemrosesan
* ✅ **Python SDK** - diinstal ke Python Anda jika tersedia (memerlukan lisensi Chloros+)
* ✅ **Profil Kamera** - Template kamera MAPIR yang telah dikonfigurasi sebelumnya

### Paket .deb Linux


* ✅ **Chloros CLI** - Antarmuka baris perintah (memerlukan lisensi Chloros+)
* ✅ **Chloros Backend** - Mesin pemrosesan
* ✅ **Python SDK** - diinstal untuk sistem `python3` (memerlukan lisensi Chloros+)
* ✅ **Profil Kamera** - Template kamera MAPIR yang telah dikonfigurasi sebelumnya
* ❌ Tanpa GUI — Linux hanya mendukung CLI/SDK tanpa antarmuka grafis

### Python SDK (PyPI, untuk mesin tanpa paket Chloros)

* ✅ **Chloros SDK** - Python API (memerlukan lisensi Chloros+)

***

## Tingkatkan ke Chloros+

Buka fitur-fitur lanjutan dengan berlangganan Chloros+:

* 🚀 **Pemrosesan Multi-threaded** - Memproses gambar secara paralel
* ⚡ **Akselerasi GPU (CUDA)** - Manfaatkan kekuatan GPU NVIDIA
* 💻 **Akses CLI** - Otomatisasi dengan alat baris perintah
* 🐍 **Python SDK** - Akses API secara terprogram
* 📱 **Beberapa Perangkat** - Gunakan pada 2–10+ perangkat (tergantung paket)
* **🐻 Metode Debayer Canggih yang Memperhatikan Tekstur** - metode debayer berkualitas tinggi yang memperhatikan tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir semua noise debayer.
* 🧮 **Rumus Kustom** - Buat indeks multispektral kustom

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Lihat Paket & Harga Chloros+</a></p>

***

## Bantuan Instalasi

### Pemecahan Masalah

**Instalasi gagal dengan pesan kesalahan:**

* Pastikan Anda memiliki hak administrator
* Nonaktifkan sementara perangkat lunak antivirus
* Periksa apakah Anda memenuhi persyaratan sistem minimum

**Aplikasi tidak mau berjalan (WindowsWindows):**

* Pastikan Windows 10/11 (64-bit) sudah terinstal
* Perbarui driver grafis
* Periksa Windows Event Viewer untuk detail kesalahannya
* Hubungi dukungan dengan log kesalahannya

**CLI tidak mau berjalan (Linux):**

* Pastikan paket `.deb` terinstal dengan benar: `dpkg -l | grep chloros`
* Periksa izin: `sudo chmod +x /usr/bin/chloros-cli`
* Jalankan diagnostik: `chloros-cli selftest`
* Periksa apakah ada pustaka yang hilang: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (sejak versi 1.3.0, `/usr/lib/chloros/chloros-backend` adalah skrip peluncur kecil; programnya sendiri ada di `/usr/lib/chloros/backend/`)

**Masalah aktivasi lisensi:**

* Pastikan koneksi internet aktif
* Verifikasi kredensial di [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Pastikan firewall tidak memblokir Chloros

* Lihat [Chloros+ Login](chloros+-login.md) untuk petunjuk terperinci

### Mendapatkan Bantuan

Butuh bantuan untuk instalasi atau pengaturan?

* 📧 **Email**: info@mapir.camera
* 🌐 **Situs Web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentasi**: [Memulai](./)
* ❓ **FAQ**: [Pertanyaan yang Sering Diajukan](faq.md)

***

## Pembaruan Perangkat Lunak

Chloros memeriksa ketersediaan pembaruan, mengumumkan saat versi baru tersedia, dan menautkan ke halaman unduhan ini — Anda dapat memperbarui dengan menjalankan penginstal baru yang telah ditandatangani. Pengaturan, proyek, dan data login Anda tetap utuh setelah pembaruan. Di Linux dan Jetson, `chloros-cli update` memeriksa apakah ada versi yang lebih baru dan menawarkan untuk mengunduh serta menginstal `.deb` yang sesuai (perintah ini hanya tersedia di Linux).

***

## Catatan Perubahan**Versi 1.3.0 (Terbaru, 11 November 2026)**— lihat**Apa yang Baru di Chloros

1.3.0** di halaman [Memulai](./) untuk daftar lengkapnya.**Versi 1.2.0**— lihat**Apa yang Baru di Chloros 1.2.0** di halaman [Memulai](./).

<details>

<summary>Versi 1.0.5</summary>

**Tanggal Rilis: 10 Februari 2026**

**Fitur Baru**

* **Metode Debayer yang Memperhatikan Tekstur \[Hanya Chloros+] -** Metode ini menggunakan debayer berkualitas tinggi yang peka terhadap tepi, dikombinasikan dengan model penghilang noise berbasis AI/ML yang menghilangkan hampir seluruh noise debayering.
* **Dukungan untuk Target Kalibrasi T4P**
* **Pemrosesan GPU Chloros+ yang lebih cepat, pengelolaan memori yang lebih baik**

**Perbaikan Bug**

* Antarmuka pengguna (GUI) yang sepenuhnya baru, kini seharusnya berfungsi di semua komputer Windows.

</details>

<details>

<summary>Versi 1.0.4</summary>

**Tanggal Rilis: 5 Januari 2026**

**Fitur Baru**

* **Toggle Gambar/Metadata**: Ditambahkan tombol toggle di File Browser untuk menampilkan metadata gambar yang dipilih dalam bentuk tabel, bukan dalam kisi gambar
* **Penggeser Zoom Kisi Gambar**: Penggeser UI baru untuk menyesuaikan ukuran thumbnail (juga mendukung CTRL + roda mouse)
* **Tombol Ekspor Kisi Gambar**: Tombol di baris atas untuk mengganti thumbnail dari JPG ke ekspor yang telah diproses (Target, Reflektansi, Indeks, LUT)
* **Tab Peta**: Peta 2D interaktif baru yang menampilkan penanda lokasi GPS gambar
  * Mendukung Google Maps dan ubin peta ESRI (secara otomatis memilih layanan ubin terbaik berdasarkan ketersediaan tingkat zoom)
  * Pratinjau thumbnail saat mengarahkan kursor ke penanda peta

**Perbaikan Bug**

* Peningkatan dukungan untuk menginstal Chloros pada komputer dengan bahasa selain bahasa Inggris

</details>

<details>

<summary>Versi 1.0.3</summary>

**Tanggal Rilis: 20 Desember 2025**

**Fitur Baru**

* Peluncuran Awal

**Peningkatan**

* Peluncuran Awal

**Perbaikan Bug**

* Peluncuran Awal

**Masalah yang Diketahui**

* Peluncuran Awal

</details>

***

## Perjanjian Lisensi**Perangkat Lunak Berhak Cipta** - Hak Cipta (c) 2026 MAPIR Inc.

Dilarang menggunakan, mendistribusikan, atau memodifikasi tanpa izin.

**Versi Gratis**: Tersedia untuk penggunaan pribadi dan komersial dengan batasan fitur**Chloros+**: Lisensi berbasis langganan untuk fitur lanjutan dan penerapan komersial

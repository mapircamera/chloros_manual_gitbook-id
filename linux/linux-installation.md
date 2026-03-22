# Instalasi Linux

Chloros didistribusikan untuk Linux sebagai paket `.deb` yang menginstal CLI dan backend. Python dan SDK diinstal secara terpisah melalui pip.

***

## Linux amd64 (x86_64)

### Persyaratan Sistem

| Persyaratan | Minimum | Direkomendasikan |
| --- | --- | --- |
| **Distribusi** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Prosesor** | x86_64 (Intel/AMD) | Intel Core i7 atau yang lebih baik |
| **Memori (RAM)** | 8GB | 16GB atau lebih |
| **Kartu Grafis** | Tidak diperlukan (pemrosesan CPU) | GPU NVIDIA dengan VRAM 4GB+ |
| **Penyimpanan** | Ruang kosong 2GB | SSD dengan ruang kosong 10GB+ |
| **Python** | Python 3.7+ (untuk SDK) | Python 3.10+ |

### Instalasi

Unduh paket `.deb` dan instal:

```bash
sudo dpkg -i chloros-amd64.deb
```

Verifikasi instalasi:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Persyaratan Sistem

| Persyaratan | Minimum | Direkomendasikan |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson dengan JetPack 6 | Jetson Orin NX 16GB atau AGX Orin |
| **JetPack** | JetPack 6.x | JetPack 6 terbaru |
| **Memori (RAM)** | 8GB (bersama GPU/CPU) | 16GB+ bersama |
| **Penyimpanan** | Ruang kosong 2GB | SSD NVMe dengan ruang kosong 10GB+ |
| **Python** | Python 3.7+ (untuk SDK) | Python 3.10+ |

### Instalasi

Unduh paket JetPack 6 `.deb` dan instal:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Verifikasi instalasi:

```bash
chloros-cli --version
```

Untuk pengaturan Jetson yang lebih rinci termasuk manajemen termal dan penerapan di lapangan, lihat [Panduan NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Instalasi (Semua Linux)

Python SDK diinstal secara terpisah melalui pip dan berfungsi pada amd64 dan arm64:

```bash
pip install chloros-sdk
```

Untuk menyertakan dukungan streaming progresif opsional:

```bash
pip install chloros-sdk[progress]
```

Verifikasi SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Paket `.deb` menginstal Chloros, CLI, dan backend. Python SDK adalah paket pip terpisah yang berkomunikasi dengan backend melalui HTTP API lokal.
{% endhint %}

***

## Direktori Konfigurasi

Chloros pada Linux mengikuti [Spesifikasi Direktori Dasar XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Tujuan | Linux Jalur | Windows Setara |
| --- | --- | --- |
| **Konfigurasi** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Data / Proyek** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Kredensial** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Lokasi Eksekusi Backend

Paket `.deb` menginstal backend ke lokasi standar. CLI dan SDK mendeteksi jalur backend secara otomatis:

| Metode Instalasi | Jalur Backend |
| --- | --- |
| Paket `.deb` | `/usr/lib/chloros/chloros-backend` |
| Manual / kustom | `/opt/mapir/chloros/backend/chloros-backend` |

Anda dapat mengganti jalur backend menggunakan bendera `--backend-exe` CLI atau parameter konstruktor `backend_exe` SDK.

***

## Pengaturan Awal

### 1. Aktifkan Lisensi Anda

Lisensi Chloros+ diperlukan untuk mengakses CLI dan SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Periksa Status Lisensi Anda

```bash
chloros-cli status
```

### 3. Proses Dataset Pertama Anda

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Jalankan Diagnostik Sistem

Pastikan sistem Anda dikonfigurasi dengan benar:

```bash
chloros-cli selftest
```

Ini menjalankan 7 pemeriksaan diagnostik termasuk versi, startup backend, konektivitas API, dan ketersediaan CUDA/GPU.

***

## Contoh Skrip Bash

### Memproses Beberapa Set Data

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Memproses dengan Pengaturan Kustom

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Pemrosesan Otomatis dengan Cron

Tambahkan ke crontab Anda (`crontab -e`) untuk memproses dataset baru secara otomatis:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Contoh Python SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Pemecahan Masalah

### CLI Tidak Ditemukan Setelah Instalasi

Jika `chloros-cli` tidak ditemukan setelah menginstal paket `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Izin Ditolak

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Backend Gagal Dimulai

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA Tidak Terdeteksi

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Perpustakaan Bersama Tidak Ditemukan

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Memperbarui Chloros pada Linux

Gunakan perintah pembaruan bawaan untuk memeriksa dan menginstal pembaruan:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Langkah Selanjutnya

* [Panduan NVIDIA Jetson](nvidia-jetson-guide.md) — Optimasi dan penerapan khusus Jetson
* [CLI : Baris Perintah](../CLI.md) — Referensi perintah CLI lengkap
* [API : Python SDK](../api-python-sdk.md) — Referensi lengkap SDK
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — Bagaimana Chloros beradaptasi dengan perangkat keras Anda

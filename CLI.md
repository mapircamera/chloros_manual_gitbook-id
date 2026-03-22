# CLI : Baris Perintah

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** menyediakan akses baris perintah yang kuat ke mesin pemrosesan gambar Chloros, yang memungkinkan otomatisasi, pembuatan skrip, dan operasi tanpa antarmuka pengguna untuk alur kerja pencitraan Anda.

### Fitur Utama

* 🚀 **Otomatisasi** - Pemrosesan batch skrip untuk beberapa dataset
* 🔗 **Integrasi** - Dapat diintegrasikan ke dalam alur kerja dan pipeline yang sudah ada
* 💻 **Operasi Tanpa Antarmuka Grafis** - Dapat dijalankan tanpa antarmuka grafis
* 🌍 **Multi-Bahasa** - Mendukung 38 bahasa
* ⚡ **Pemrosesan Paralel** - [Adaptasi Komputasi Dinamis](processing-architecture/dynamic-compute-adaptation.md) secara otomatis mengoptimalkan sesuai dengan perangkat keras Anda

### Persyaratan

| Persyaratan          | Detail                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistem Operasi** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Lisensi**          | Chloros+ ([diperlukan paket berbayar](https://cloud.mapir.camera/pricing)) |
| **Memori**           | Minimal 8GB RAM (disarankan 16GB)                                  |
| **Internet**         | Diperlukan untuk aktivasi lisensi                                     |
| **Ruang Disk**       | Bervariasi sesuai ukuran proyek                                              |

{% hint style="warning" %}
**Persyaratan Lisensi**: CLI memerlukan langganan berbayar Chloros+. Paket Standar (gratis) tidak memiliki akses ke CLI. Kunjungi [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) untuk meningkatkan paket.
{% endhint %}

## Panduan Cepat

### Instalasi

#### Windows

CLI secara otomatis disertakan dalam penginstal Chloros:

1. Unduh dan jalankan **Chloros Installer.exe**

2. Selesaikan wizard pemasangan
3. CLI terpasang di: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Pemasang secara otomatis menambahkan `chloros-cli` ke jalur sistem (PATH) Anda. Mulai ulang terminal Anda setelah pemasangan.
{% endhint %}

#### Linux

Pasang paket `.deb` untuk arsitektur Anda:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Untuk pengaturan Linux yang lebih rinci, lihat [Linux Instalasi](linux/linux-installation.md).

### Pengaturan Awal

Sebelum menggunakan CLI, aktifkan lisensi Chloros+ Anda:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Penggunaan Dasar

Proses folder dengan pengaturan default:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referensi Perintah

### Sintaks Umum

```
chloros-cli [global-options] <command> [command-options]
```

***

## Perintah

### `process` - Memproses Gambar

Memproses gambar dalam folder dengan kalibrasi.

**Sintaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Contoh:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Opsi Perintah Pemrosesan

| Opsi                | Tipe    | Default        | Deskripsi                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Jalur    | _Diperlukan_     | Folder yang berisi gambar multispektral RAW/JPG                                         |
| `-o, --output`        | Jalur    | Sama dengan input  | Folder keluaran untuk gambar yang telah diproses                                                     |
| `-n, --project-name`  | String  | Dibuat otomatis | Nama proyek khusus                                                                    |
| `--vignette`          | Bendera    | Diaktifkan        | Aktifkan koreksi vignette                                                             |
| `--no-vignette`       | Bendera    | -              | Nonaktifkan koreksi vignette                                                            |
| `--reflectance`       | Bendera    | Diaktifkan        | Aktifkan kalibrasi reflektansi                                                         |
| `--no-reflectance`    | Bendera    | -              | Nonaktifkan kalibrasi reflektansi                                                        |
| `--ppk`               | Bendera    | Dinonaktifkan       | Terapkan koreksi PPK dari data sensor cahaya .daq                                      |
| `--format`            | Pilihan  | TIFF (16-bit)  | Format keluaran: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Bilangan bulat | Otomatis           | Ukuran target minimum dalam piksel untuk deteksi panel kalibrasi                          |
| `--target-clustering` | Bilangan bulat | Otomatis           | Ambang batas pengelompokan target (0-100)                                                    |
| `--debayer`           | Pilihan  | `standard`     | Metode debayer: `standard` atau `texture-aware` (hanya Chloros+)                          |
| `--target`, `--targets` | Bendera  | Dinonaktifkan       | Hanya cari target kalibrasi di subfolder &quot;target&quot; atau &quot;targets&quot; (mempercepat pemrosesan) |
| `--indices`           | Daftar    | Tidak ada           | Indeks vegetasi yang akan dihitung (misalnya, `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | String  | Tidak ada           | Mengunci eksposur untuk model kamera (Pin 1)                                                 |
| `--exposure-pin-2`    | String  | Tidak ada           | Mengunci eksposur untuk model kamera (Pin 2)                                                 |
| `--recal-interval`    | Bilangan bulat | Otomatis      | Interval kalibrasi ulang dalam detik                                                      |
| `--timezone-offset`   | Bilangan bulat | 0              | Perbedaan zona waktu dalam jam                                                               |

***

### `login` - Otentikasi Akun

Masuk menggunakan kredensial Chloros+ Anda untuk mengaktifkan pemrosesan CLI.

**Sintaks:**

```bash
chloros-cli login <email> <password>
```

**Contoh:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Karakter Khusus**: Gunakan tanda kutip tunggal di sekitar kata sandi yang mengandung karakter seperti `$`, `!`, atau spasi.
{% endhint %}

**Output:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Hapus Kredensial

Hapus kredensial yang tersimpan dan keluar dari akun Anda.

**Sintaks:**

```bash
chloros-cli logout
```

**Contoh:**

```bash
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Pengguna SDK**: Python SDK juga menyediakan metode programatik `logout()` untuk menghapus kredensial dalam skrip Python. Lihat [dokumentasi Python SDK](api-python-sdk.md#logout) untuk detailnya.
{% endhint %}

***

### `status` - Periksa Status Lisensi

Menampilkan status lisensi dan otentikasi saat ini.

**Sintaks:**

```bash
chloros-cli status
```

**Contoh:**

```bash
chloros-cli status
```

**Output:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Periksa Kemajuan Ekspor

Memantau kemajuan ekspor Thread 4 selama atau setelah pemrosesan.

**Sintaks:**

```bash
chloros-cli export-status
```

**Contoh:**

```bash
chloros-cli export-status
```

**Kasus Penggunaan:** Jalankan perintah ini saat pemrosesan sedang berjalan untuk memeriksa kemajuan ekspor.***

### `language` - Kelola Bahasa Antarmuka

Lihat atau ubah bahasa antarmuka CLI.

**Sintaks:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Contoh:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Bahasa yang Didukung (Total 38)

| Kode    | Bahasa              | Nama Asli      |
| ------- | --------------------- | ---------------- |
| `en`    | Inggris               | English          |
| `es`    | Spanyol               | Español          |
| `pt`    | Portugis            | Português        |
| `fr`    | Prancis                | Français         |
| `de`    | Jerman                | Deutsch          |
| `it`    | Italia               | Italiano         |
| `ja`    | Jepang              | 日本語              |
| `ko`    | Korea                | 한국어              |
| `zh`    | Tionghoa (Sederhana)  | 简体中文             |
| `zh-TW` | Tionghoa (Tradisional) | 繁體中文             |
| `ru`    | Rusia               | Русский          |
| `nl`    | Belanda                | Nederlands       |
| `ar`    | Arab                | العربية          |
| `pl`    | Polandia                | Polski           |
| `tr`    | Turki               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesia            | Bahasa Indonesia |
| `vi`    | Vietnam            | Tiếng Việt       |
| `th`    | Thailand                  | ไทย              |
| `sv`    | Swedia               | Svenska          |
| `da`    | Denmark                | Dansk            |
| `no`    | Norwegia             | Norsk            |
| `fi`    | Finlandia               | Suomi            |
| `el`    | Yunani                 | Ελληνικά         |
| `cs`    | Ceko                 | Čeština          |
| `hu`    | Hongaria             | Magyar           |
| `ro`    | Rumania              | Română           |
| `uk`    | Ukraina             | Українська       |
| `pt-BR` | Portugis Brasil  | Português Brasileiro |
| `zh-HK` | Kanton             | 粵語             |
| `ms`    | Melayu                 | Bahasa Melayu    |
| `sk`    | Bahasa Slovakia                | Slovenčina       |
| `bg`    | Bahasa Bulgaria             | Български        |
| `hr`    | Bahasa Kroasia              | Hrvatski         |
| `lt`    | Lituania            | Lietuvių         |
| `lv`    | Latvia               | Latviešu         |
| `et`    | Estonia              | Eesti            |
| `sl`    | Slovenia             | Slovenščina      |

{% hint style="success" %}
**Persistensi Otomatis**: Preferensi bahasa Anda disimpan ke `~/.chloros/cli_language.json` dan tetap berlaku di semua sesi.
{% endhint %}

***

### `set-project-folder` - Atur Folder Proyek Default

Ubah lokasi folder proyek default (dibagikan dengan antarmuka pengguna pada Windows).

**Sintaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Contoh:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Tampilkan Folder Proyek

Tampilkan lokasi folder proyek default saat ini.

**Sintaks:**

```bash
chloros-cli get-project-folder
```

**Contoh:**

```bash
chloros-cli get-project-folder
```

**Hasil:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Atur Ulang ke Default

Atur ulang folder proyek ke lokasi default.

**Sintaks:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Jalankan Diagnostik Sistem

Jalankan 7 pemeriksaan diagnostik untuk memverifikasi konfigurasi sistem Anda.

**Sintaks:**

```bash
chloros-cli selftest
```

**Pemeriksaan yang dilakukan:**

1. Pemeriksaan versi
2. Ketersediaan port (5000)
3. Pengaktifan backend
4. Uji konektivitas API
5. Informasi sistem dan deteksi GPU
6. Verifikasi model denoiser
7. Pemeriksaan ketersediaan CUDA

{% hint style="info" %}
**Berguna untuk pemecahan masalah**: Jalankan `selftest` setelah instalasi untuk memverifikasi bahwa sistem Anda telah dikonfigurasi dengan benar, terutama pada Linux/Jetson di mana pengaturan GPU dan CUDA mungkin memerlukan verifikasi.
{% endhint %}

***

### `update` - Periksa Pembaruan (Hanya Linux)

Periksa dan instal pembaruan CLI pada sistem Linux.

**Sintaks:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opsi    | Deskripsi                        |
| --------- | ---------------------------------- |
| `--check` | Hanya periksa pembaruan, jangan instal |

{% hint style="info" %}
Perintah ini hanya tersedia pada Linux. Pada Windows, pembaruan disediakan melalui penginstal.
{% endhint %}

***

## Opsi Global

Opsi-opsi ini berlaku untuk semua perintah:

| Opsi            | Tipe    | Default       | Deskripsi                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Jalur    | Terdeteksi otomatis | Jalur ke file eksekusi backend                       |
| `--port`          | Bilangan bulat | 5000          | Nomor port backend API                          |
| `--restart`       | Bendera    | -             | Memaksa restart backend (menghentikan proses yang ada) |
| `--version`       | Bendera    | -             | Menampilkan informasi versi dan keluar                |
| `--help`          | Bendera    | -             | Menampilkan informasi bantuan dan keluar                   |

{% hint style="info" %}
**Deteksi otomatis backend**: Jalur `--backend-exe` dideteksi secara otomatis per platform:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manual)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Contoh dengan Opsi Global:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Panduan Pengaturan Pemrosesan

### Pemrosesan Paralel &amp; Adaptasi Komputasi Dinamis

Chloros 1.1.0 mencakup [Adaptasi Komputasi Dinamis](processing-architecture/dynamic-compute-adaptation.md) — mesin pemrosesan **secara otomatis mendeteksi perangkat keras Anda** dan memilih strategi optimal:

| Platform | Strategi | Pekerja | Jalur | Catatan |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Efisien memori, serial |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Pemrosesan GPU bersamaan |
| **Desktop dengan GPU 8GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Kinerja desktop yang baik |
| **Desktop dengan GPU 12GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Kinerja desktop optimal |
| **Sistem hanya CPU** | `CPU_PARALLEL` | inti - 1 | `cpu_fallback` | Tidak memerlukan GPU |

{% hint style="success" %}
**Tidak perlu konfigurasi manual!** Chloros mendeteksi secara otomatis CPU, GPU, RAM, dan (pada Jetson) sensor suhu Anda, lalu mengonfigurasi alur pemrosesan optimal secara otomatis.
{% endhint %}

### Metode Debayer

| Metode | Bendera CLI | Kualitas | Kecepatan | Lisensi |
| --- | --- | --- | --- | --- |
| **Standar (Cepat, Kualitas Sedang)** | `--debayer standard` | Baik | Cepat | Gratis / Chloros+ |
| **Texture Aware (Lambat, Kualitas Tertinggi)** | `--debayer texture-aware` | Tertinggi | Lambat | Hanya Chloros+ |

Metode debayer default adalah **Standar**. Metode**Texture Aware** menggunakan model denoising AI/ML untuk hasil berkualitas tertinggi, tetapi memerlukan lisensi Chloros+ dan GPU NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Koreksi Vignette

**Fungsinya:** Mengoreksi penurunan kecerahan di tepi gambar (sudut gelap yang umum pada gambar kamera).

* **Diaktifkan secara default** - Sebagian besar pengguna sebaiknya tetap mengaktifkan ini
* Gunakan `--no-vignette` untuk menonaktifkan

{% hint style="success" %}
**Rekomendasi**: Selalu aktifkan koreksi vignette untuk memastikan kecerahan yang merata di seluruh bingkai.
{% endhint %}

### Kalibrasi Reflektansi

Mengonversi nilai sensor mentah menjadi persentase reflektansi standar menggunakan panel kalibrasi.

* **Diaktifkan secara default** - Penting untuk analisis vegetasi
* Membutuhkan panel target kalibrasi dalam gambar
* Gunakan `--no-reflectance` untuk menonaktifkan

{% hint style="info" %}
**Persyaratan**: Pastikan panel kalibrasi terpapar dengan baik dan terlihat jelas dalam gambar Anda untuk konversi reflektansi yang akurat.
{% endhint %}

### Koreksi PPK

**Fungsinya:** Menerapkan koreksi Kinematik Pasca-Pemrosesan menggunakan data log DAQ-A-SD untuk meningkatkan akurasi GPS.

* **Dinonaktifkan secara default**
* Gunakan `--ppk` untuk mengaktifkan
* Membutuhkan file .daq di folder proyek dari sensor cahaya DAQ-A-SD MAPIR.

### Format Keluaran

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Kedalaman Bit</th><th width="116.5999755859375">Ukuran File</th><th>Cocok Untuk</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>Bilangan bulat 16-bit</td><td>Besar</td><td>Analisis GIS, fotogrametri (disarankan)</td></tr><tr><td><strong>TIFF (32-bit, Persen)</strong></td><td>Bilangan desimal 32-bit</td><td>Sangat Besar</td><td>Analisis ilmiah, penelitian</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>Bilangan bulat 8-bit</td><td>Sedang</td><td>Pemeriksaan visual, berbagi web</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>Bilangan bulat 8-bit</td><td>Kecil</td><td>Pratinjau cepat, keluaran terkompresi</td></tr></tbody></table>***

## Otomatisasi &amp; Skrip

### Pemrosesan Batch PowerShell (Windows)

Proses beberapa folder dataset secara otomatis di Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Skrip Batch Windows (Windows)

Perulangan sederhana untuk pemrosesan batch pada Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Pemrosesan Batch Bash (Linux)

Memproses beberapa folder dataset pada Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Skrip Otomatisasi Python (Lintas Platform)

Otomatisasi lanjutan dengan penanganan kesalahan (berfungsi pada Windows dan Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Alur Kerja Pemrosesan

### Alur Kerja Standar

1. **Masukan**: Folder yang berisi pasangan gambar RAW/JPG
2. **Deteksi**: CLI secara otomatis memindai file gambar yang didukung
3. **Pemrosesan**: Mode paralel menyesuaikan dengan jumlah inti CPU Anda (Chloros+)
4. **Output**: Membuat subfolder berdasarkan model kamera dengan gambar yang telah diproses

### Contoh Struktur Output

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Perkiraan Waktu Pemrosesan

Waktu pemrosesan tipikal untuk 100 gambar (masing-masing 12MP):

| Platform | Mode | Waktu Perkiraan | Catatan |
| --- | --- | --- | --- |
| **Desktop 12GB+ GPU** | `GPU_PARALLEL` | 5-10 menit | Opsi tercepat |
| **Desktop 8GB GPU** | `GPU_SINGLE` | 10-15 menit | Kinerja baik |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 15-25 menit | Komputasi tepi |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 30-60 menit | Terbatas memori |
| **Hanya CPU** | `CPU_PARALLEL` | 20-40 menit | Tidak memerlukan GPU |

{% hint style="info" %}
**Tips Kinerja**: Waktu pemrosesan bervariasi tergantung pada jumlah gambar, resolusi, metode debayer, dan perangkat keras. Metode debayer Texture Aware memakan waktu jauh lebih lama daripada metode Standar. Lihat [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) untuk detailnya.
{% endhint %}

***

## Pemecahan Masalah

### CLI Tidak Ditemukan

**Windows Kesalahan:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Solusi:**

1. Periksa lokasi instalasi:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Gunakan jalur lengkap jika tidak ada di PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Tambahkan ke PATH secara manual:
   * Buka Properti Sistem → Variabel Lingkungan
   * Edit variabel PATH
   * Tambahkan: `C:\Program Files\Chloros\resources\cli`
   * Mulai ulang terminal

**Kesalahan Linux:**

```
chloros-cli: command not found
```

**Solusi Linux:**

1. Periksa instalasi:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Muat ulang shell Anda:

```bash
source ~/.bashrc
```

3. Periksa izin:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Backend Gagal Dimulai**Kesalahan:**

```

Backend failed to start within 30 seconds
```

**Solusi:**

1. Periksa apakah backend sudah berjalan (tutup terlebih dahulu)
2. Periksa apakah firewall tidak memblokir (Windows) atau periksa ketersediaan port (Linux: `lsof -i :5000`)
3. Coba port yang berbeda:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Paksa restart backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Pada Linux, periksa apakah file executable backend ada:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Masalah Lisensi / Otentikasi**Kesalahan:**

```

Chloros+ license required for CLI access
```

**Solusi:**

1. Pastikan Anda memiliki langganan Chloros+ yang aktif
2. Masuk dengan kredensial Anda:

```bash
chloros-cli login user@example.com 'password'
```

3. Periksa status lisensi:

```bash
chloros-cli status
```

4. Hubungi dukungan: info@mapir.camera

***

### Tidak Ditemukan Gambar**Kesalahan:**

```

No images found in the specified folder
```

**Solusi:**

1. Pastikan folder berisi format yang didukung (.RAW, .TIF, .JPG)
2. Periksa jalur folder sudah benar (gunakan tanda kutip untuk jalur yang mengandung spasi)
3. Pastikan Anda memiliki izin baca untuk folder tersebut
4. Periksa ekstensi file sudah benar

***

### Pemrosesan Terhenti atau Macet**Solusi:**

1. Periksa ruang disk yang tersedia (pastikan cukup untuk output)
2. Tutup aplikasi lain untuk membebaskan memori
3. Kurangi jumlah gambar (proses secara bertahap)

***

### Port Sudah Digunakan**Kesalahan:**

```

Port 5000 is already in use
```

**Solusi:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## FAQ

### Q: Apakah saya memerlukan lisensi untuk CLI?

**A:**Ya! CLI memerlukan lisensi berbayar**Chloros+**.

* ❌ Paket Standar (gratis): CLI dinonaktifkan
* ✅ Paket Chloros+ (berbayar): CLI sepenuhnya diaktifkan

Berlangganan di: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q: Bisakah saya menggunakan CLI pada server tanpa antarmuka grafis (GUI)?**A:** Ya! CLI berjalan sepenuhnya tanpa antarmuka grafis (headless). Ini adalah kasus penggunaan utama pada Linux.**Server Windows:**
* Server Windows 2016 atau yang lebih baru
* Visual C++ Redistributable terinstal

**Server Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) atau JetPack 6 (arm64)
* Instal melalui paket `.deb`

**Kedua platform:**
* RAM minimal 8GB (disarankan 16GB)
* Aktivasi lisensi satu kali: `chloros-cli login user@example.com 'password'`

***

### Q: Di mana gambar yang diproses disimpan?**A:**Secara default, gambar yang diproses disimpan di**folder yang sama dengan input** dalam subfolder model kamera (misalnya, `Survey3N_RGN/`).

Gunakan opsi `-o` untuk menentukan folder output yang berbeda:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### Q: Bisakah saya memproses beberapa folder sekaligus?**A:** Tidak secara langsung dalam satu perintah, tetapi Anda dapat menggunakan skrip untuk memproses folder secara berurutan. Lihat bagian [Otomatisasi &amp; Skrip](CLI.md#automation--scripting).***

### Q: Bagaimana cara menyimpan output CLI ke berkas log?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### Q: Apa yang terjadi jika saya menekan Ctrl+C selama pemrosesan?**A:** CLI akan:

1. Menghentikan pemrosesan dengan lancar
2. Mematikan backend
3. Keluar dengan kode 130

Gambar yang diproses sebagian mungkin tetap ada di folder output.

***

### Q: Bisakah saya mengotomatiskan pemrosesan CLI?**A:** Tentu saja! CLI dirancang untuk otomatisasi. Lihat [Otomatisasi &amp; Skrip](CLI.md#automation--scripting) untuk PowerShell (Windows), Batch (Windows), Bash (Linux), dan contoh Python (multi-platform).***

### Q: Bagaimana cara memeriksa versi CLI?**A:**

```bash
chloros-cli --version
```

**Output:**

```

Chloros CLI 1.1.0
```

***

## Mendapatkan Bantuan

### Bantuan Baris Perintah

Lihat informasi bantuan langsung di CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Saluran Dukungan

* **Email**: info@mapir.camera
* **Situs Web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Harga**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Contoh Lengkap

### Contoh 1: Pemrosesan Dasar

Proses dengan pengaturan default (vignette, reflektansi):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Contoh 2: Keluaran Ilmiah Berkualitas Tinggi

32-bit float TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Contoh 3: Pemrosesan Pratinjau Cepat

8-bit PNG tanpa kalibrasi untuk tinjauan cepat:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Contoh 4: Pemrosesan yang Dikoreksi PPK

Terapkan koreksi PPK dengan reflektansi:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Contoh 5: Lokasi Keluaran Kustom

Proses ke lokasi yang berbeda dengan format tertentu:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Contoh 6: Alur Kerja Otentikasi

Alur otentikasi lengkap (sama di semua platform):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Contoh 7: Penggunaan Multi-Bahasa

Ubah bahasa antarmuka (sama di semua platform):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```

# CLI : Baris Perintah

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** menyediakan akses baris perintah yang kuat ke mesin pemrosesan gambar Chloros, memungkinkan otomatisasi, pemrograman skrip, dan operasi tanpa antarmuka pengguna untuk alur kerja pemrosesan gambar Anda.

### Fitur Utama

* 🚀 **Otomatisasi** - Pengolahan batch skrip untuk beberapa dataset
* 🔗 **Integrasi** - Terintegrasi dalam alur kerja dan pipa kerja yang sudah ada
* 💻 **Operasi Tanpa Antarmuka Pengguna** - Berjalan tanpa antarmuka pengguna
* 🌍 **Multi-Bahasa** - Dukungan untuk 38 bahasa
* ⚡ **Pemrosesan Paralel** - Skalabilitas dinamis sesuai CPU (hingga 16 pekerja paralel)

### Persyaratan

| Persyaratan          | Rincian                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistem Operasi** | Windows 10/11 (64-bit)                                              |
| **Lisensi**          | Chloros+ ([paket berbayar diperlukan](https://cloud.mapir.camera/pricing)) |
| **Memori**           | Minimal 8GB RAM (disarankan 16GB)                                  |
| **Internet**         | Diperlukan untuk aktivasi lisensi                                     |
| **Ruang Disk**       | Bervariasi tergantung ukuran proyek                                              |

{% hint style=&quot;warning&quot; %}
**Persyaratan Lisensi**: CLI memerlukan langganan berbayar Chloros+. Rencana standar (gratis) tidak memiliki akses ke CLI. Kunjungi [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) untuk melakukan upgrade.
{% endhint %}

## Panduan Cepat

### Instalasi

CLI secara otomatis disertakan dalam penginstal Chloros:

1. Unduh dan jalankan **Chloros Installer.exe**
2. Selesaikan wizard instalasi
3. CLI diinstal ke: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
Penginstal secara otomatis menambahkan `chloros-cli` ke jalur sistem PATH Anda. Restart terminal Anda setelah instalasi.
{% endhint %}

### Pengaturan Awal

Sebelum menggunakan CLI, aktifkan lisensi Chloros+ Anda:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Penggunaan Dasar

Proses folder dengan pengaturan default:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Referensi Perintah

### Sintaks Umum

```
chloros-cli [global-options] <command> [command-options]
```

***

## Perintah

### `process` - Proses Gambar

Memproses gambar dalam folder dengan kalibrasi.

**Sintaks:**

```bash
chloros-cli process <input-folder> [options]
```

**Contoh:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opsi Perintah Pemrosesan

| Opsi                | Tipe    | Default        | Deskripsi                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Jalan    | _Diperlukan_     | Folder yang berisi gambar multispektral RAW/JPG                                         |
| `-o, --output`        | Jalan    | Sama dengan input  | Folder output untuk gambar yang diproses                                                     |
| `-n, --project-name`  | String  | Dibuat otomatis | Nama proyek kustom                                                                    |
| `--vignette`          | Bendera    | Diaktifkan        | Aktifkan koreksi vignette                                                             |
| `--no-vignette`       | Bendera    | -              | Nonaktifkan koreksi vignette                                                            |
| `--reflectance`       | Bendera    | Diaktifkan        | Aktifkan kalibrasi reflektansi                                                         |
| `--no-reflectance`    | Bendera    | -              | Nonaktifkan kalibrasi reflektansi                                                        |
| `--ppk`               | Bendera    | Dinonaktifkan       | Terapkan koreksi PPK dari data sensor cahaya .daq                                      |
| `--format`            | Pilihan  | TIFF (16-bit)  | Format output: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Bilangan bulat | Otomatis           | Ukuran target minimum dalam piksel untuk deteksi panel kalibrasi                          |
| `--target-clustering` | Bilangan bulat | Otomatis           | Ambang batas pengelompokan target (0-100)                                                    |
| `--exposure-pin-1`    | String  | None           | Kunci eksposur untuk model kamera (Pin 1)                                                 |
| `--exposure-pin-2`    | String  | None           | Kunci eksposur untuk model kamera (Pin 2)                                                 |
| `--recal-interval`    | Bilangan bulat | Otomatis           | Interval kalibrasi ulang dalam detik                                                      |
| `--timezone-offset`   | Bilangan bulat | 0              | Pergeseran zona waktu dalam jam                                                               |

***

### `login` - Otentikasi Akun

Masuk dengan kredensial Chloros+ Anda untuk mengaktifkan pemrosesan CLI.

**Sintaks:**

```bash
chloros-cli login <email> <password>
```

**Contoh:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Karakter Khusus**: Gunakan tanda kutip tunggal di sekitar kata sandi yang mengandung karakter seperti `$`, `!`, atau spasi.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Hapus Kredensial

Hapus kredensial yang disimpan dan keluar dari akun Anda.

**Sintaks:**

```bash
chloros-cli logout
```

**Contoh:**

```powershell
chloros-cli logout
```

**Output:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Periksa Status Lisensi

Tampilkan status lisensi dan otentikasi saat ini.

**Sintaks:**

```bash
chloros-cli status
```

**Contoh:**

```powershell
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

```powershell
chloros-cli export-status
```

**Kasus Penggunaan:** Panggil perintah ini saat pemrosesan sedang berjalan untuk memeriksa kemajuan ekspor.

***

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

```powershell
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Bahasa yang Didukung (38 Total)

| Kode    | Bahasa               | Nama Asli      |
| ------- | --------------------- | ---------------- |
| `en`    | Inggris               | English          |
| `es`    | Spanyol               | Español          |
| `pt`    | Portugis            | Português        |
| `fr`    | Prancis                | Français         |
| `de`    | Jerman                | Deutsch          |
| `it`    | Italia               | Italiano         |
| `ja`    | Jepang              | 日本語              |
| `ko`    | Korea                | 한국어              |
| `zh`    | Mandarin (Sederhana)  | 简体中文             |
| `zh-TW` | Mandarin (Tradisional) | 繁體中文             |
| `ru`    | Rusia               | Русский          |
| `nl`    | Belanda                 | Nederlands       |
| `ar`    | Arab                 | العربية          |
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
| `pt-BR` | Portugis Brasil       | Português Brasileiro |
| `zh-HK` | Kanton             | 粵語             |
| `ms`    | Melayu                 | Bahasa Melayu    |
| `sk`    | Slovak                | Slovenčina       |
| `bg`    | Bulgaria             | Български        |
| `hr`    | Kroasia              | Hrvatski         |
| `lt`    | Lithuania            | Lietuvių         |
| `lv`    | Latvia               | Latviešu         |
| `et`    | Estonia              | Eesti            |
| `sl`    | Slovenia             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Persisten Otomatis**: Preferensi bahasa Anda disimpan ke `~/.chloros/cli_language.json` dan tetap berlaku di semua sesi.
{% endhint %}

***

### `set-project-folder` - Atur Folder Proyek Default

Ubah lokasi folder proyek default (berbagi dengan antarmuka pengguna).

**Sintaks:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Contoh:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Tampilkan Lokasi Folder Proyek

Tampilkan lokasi folder proyek default saat ini.

**Sintaks:**

```bash
chloros-cli get-project-folder
```

**Contoh:**

```powershell
chloros-cli get-project-folder
```

**Output:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Kembalikan ke Default

Kembalikan folder proyek ke lokasi default.

**Sintaks:**

```bash
chloros-cli reset-project-folder
```

***

## Opsi Global

Opsi ini berlaku untuk semua perintah:

| Opsi          | Tipe    | Default       | Deskripsi                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Jalur    | Terdeteksi otomatis | Jalur ke executable backend                       |
| `--port`        | Bilangan bulat | 5000          | Nomor port backend API                          |
| `--restart`     | Bendera    | -             | Memaksa restart backend (membunuh proses yang ada) |
| `--version`     | Bendera    | -             | Menampilkan informasi versi dan keluar                |
| `--help`        | Bendera    | -             | Menampilkan informasi bantuan dan keluar                   |

**Contoh dengan Opsi Global:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Panduan Pengaturan Pemrosesan

### Pemrosesan Paralel

Chloros+ CLI **otomatis menyesuaikan** pemrosesan paralel sesuai dengan kemampuan komputer Anda:

**Cara Kerjanya:**

* Mendeteksi inti CPU dan RAM Anda
* Mengalokasikan pekerja: **2× inti CPU** (menggunakan hyperthreading)
* **Maksimum: 16 pekerja paralel** (untuk stabilitas)

**Tingkat Sistem:**

| Tipe Sistem   | CPU        | RAM      | Pekerja  | Kinerja     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ inti  | 32+ GB   | Hingga 16 | Kecepatan maksimum   |
| **Menengah** | 8-15 inti | 16-31 GB | 8-16     | Kecepatan excellent |
| **Bawah**   | 4-7 inti  | 8-15 GB  | 4-8      | Kecepatan baik      |

{% hint style=&quot;success&quot; %}
**Optimasi Otomatis**: CLI secara otomatis mendeteksi spesifikasi sistem Anda dan mengonfigurasi pemrosesan paralel optimal. Tidak perlu konfigurasi manual!
{% endhint %}

### Metode Debayer

CLI menggunakan **Kualitas Tinggi (Lebih Cepat)** sebagai algoritma debayer default dan direkomendasikan:

| Metode                      | Kualitas | Kecepatan | Deskripsi                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Kualitas Tinggi (Lebih Cepat)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritma yang memperhatikan tepi (default, direkomendasikan) |

### Koreksi Vignette

**Fungsinya:** Mengoreksi penurunan cahaya di tepi gambar (sudut gelap yang umum pada gambar kamera).

* **Diaktifkan secara default** - Sebagian besar pengguna disarankan untuk tetap mengaktifkannya
* Gunakan `--no-vignette` untuk menonaktifkan

{% hint style=&quot;success&quot; %}
**Rekomendasi**: Selalu aktifkan koreksi vignette untuk memastikan kecerahan yang merata di seluruh bingkai.
{% endhint %}

### Kalibrasi Reflektansi

Mengonversi nilai sensor mentah menjadi persentase reflektansi standar menggunakan panel kalibrasi.

* **Diaktifkan secara default** - Esensial untuk analisis vegetasi
* Membutuhkan panel target kalibrasi dalam gambar
* Gunakan `--no-reflectance` untuk menonaktifkan

{% hint style=&quot;info&quot; %}
**Persyaratan**: Pastikan panel kalibrasi terpapar dengan baik dan terlihat dalam gambar Anda untuk konversi reflektansi yang akurat.
{% endhint %}

### Koreksi PPK

**Fungsinya:** Menerapkan koreksi kinematik pasca-pemrosesan menggunakan data log DAQ-A-SD untuk meningkatkan akurasi GPS.

* **Dinonaktifkan secara default**
* Gunakan `--ppk` untuk mengaktifkan
* Membutuhkan file .daq di folder proyek dari sensor cahaya DAQ-A-SD MAPIR.

### Format Output

<table><thead><tr><th width="197">Format</th><th width="130.20001220703125">Kedalaman Bit</th><th width="116.5999755859375">Ukuran File</th><th>Terbaik untuk</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>Bilangan bulat 16-bit</td><td>Besar</td><td>Analisis GIS, fotogrametri (disarankan)</td></tr><tr><td><strong>TIFF (32-bit, Persen)</strong></td><td>Bilangan floating-point 32-bit</td><td>Sangat besar</td><td>Analisis ilmiah, penelitian</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>Bilangan bulat 8-bit</td><td>Sedang</td><td>Pemeriksaan visual, berbagi di web</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>Bilangan bulat 8-bit</td><td>Kecil</td><td>Pratinjau cepat, output terkompresi</td></tr></tbody></table>***

## Otomatisasi &amp; Skrip

### Pemrosesan Batch PowerShell

Memproses folder dataset secara otomatis:

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

### Skrip Batch Windows

Loop sederhana untuk pemrosesan batch:

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

### Python Skrip Otomatisasi

Otomatisasi lanjutan dengan penanganan kesalahan:

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
2. **Penemuan**: CLI secara otomatis memindai file gambar yang didukung
3. **Pemrosesan**: Mode paralel menyesuaikan dengan jumlah inti CPU Anda (Chloros+)
4. **Output**: Membuat subfolder berdasarkan model kamera dengan gambar yang telah diproses

### Struktur Output Contoh

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

| Mode              | Waktu      | Perangkat Keras                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Mode Paralel** | 5-10 menit | i7/Ryzen 7, 16GB RAM, SSD (hingga 16 pekerja) |
| **Mode Paralel** | 10-15 menit | i5/Ryzen 5, 8GB RAM, HDD (hingga 8 pekerja)   |

{% hint style=&quot;info&quot; %}
**Tips Kinerja**: Waktu pemrosesan bervariasi tergantung pada jumlah gambar, resolusi, dan spesifikasi komputer.
{% endhint %}

***

## Pemecahan Masalah

### CLI Tidak Ditemukan

**Kesalahan:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Solusi:**

1. Verifikasi lokasi instalasi:

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
   * Restart terminal

***

### Backend Gagal Dimulai

**Kesalahan:**

```
Backend failed to start within 30 seconds
```

**Solusi:**

1. Periksa apakah backend sudah berjalan (tutup terlebih dahulu)
2. Periksa apakah Firewall Windows tidak memblokir
3. Coba port yang berbeda:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Paksa restart backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Masalah Lisensi / Otentikasi

**Kesalahan:**

```
Chloros+ license required for CLI access
```

**Solusi:**

1. Pastikan Anda memiliki langganan Chloros+ yang aktif
2. Masuk dengan kredensial Anda:

```powershell
chloros-cli login user@example.com 'password'
```

3. Periksa status lisensi:

```powershell
chloros-cli status
```

4. Hubungi dukungan: info@mapir.camera

***

### Tidak Ditemukan Gambar

**Kesalahan:**

```
No images found in the specified folder
```

**Solusi:**

1. Pastikan folder berisi format yang didukung (.RAW, .TIF, .JPG)
2. Periksa jalur folder benar (gunakan tanda kutip untuk jalur dengan spasi)
3. Pastikan Anda memiliki izin baca untuk folder
4. Periksa ekstensi file benar

***

### Proses Terhenti atau Macet

**Solusi:**

1. Periksa ruang disk yang tersedia (pastikan cukup untuk output)
2. Tutup aplikasi lain untuk membebaskan memori
3. Kurangi jumlah gambar (proses secara bertahap)

***

### Port Sudah Digunakan

**Kesalahan:**

```
Port 5000 is already in use
```

**Solusi:**

Tentukan port yang berbeda:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## FAQ

### Q: Apakah saya memerlukan lisensi untuk CLI?

**A:** Ya! CLI memerlukan lisensi berbayar **Chloros+**.

* ❌ Rencana Standar (gratis): CLI dinonaktifkan
* ✅ Paket Chloros+ (berbayar): CLI sepenuhnya diaktifkan

Berlangganan di: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### Q: Apakah saya dapat menggunakan CLI pada server tanpa antarmuka grafis (GUI)?

**A:** Ya! CLI dapat berjalan sepenuhnya tanpa antarmuka grafis. Persyaratan:

* Windows Server 2016 atau versi terbaru
* Visual C++ Redistributable terinstal
* RAM yang cukup (minimal 8GB, disarankan 16GB)
* Aktivasi lisensi GUI sekali saja pada mesin mana pun

***

### Q: Di mana gambar yang diproses disimpan?

**A:** Secara default, gambar yang diproses disimpan di folder yang sama dengan input, dalam subfolder model kamera (misalnya, `Survey3N_RGN/`).

Gunakan opsi `-o` untuk menentukan folder output yang berbeda:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### Q: Bisakah saya memproses beberapa folder sekaligus?

**A:** Tidak secara langsung dalam satu perintah, tetapi Anda dapat menggunakan skrip untuk memproses folder secara berurutan. Lihat bagian [Automation &amp; Scripting](CLI.md#automation--scripting).

***

### Q: Bagaimana cara menyimpan output CLI ke file log?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### Q: Apa yang terjadi jika saya menekan Ctrl+C selama pemrosesan?

**A:** CLI akan:

1. Menghentikan pemrosesan dengan lancar
2. Mematikan backend
3. Keluar dengan kode 130

Gambar yang diproses sebagian mungkin tetap ada di folder output.

***

### Q: Apakah saya dapat mengotomatisasi pemrosesan CLI?

**A:** Tentu saja! CLI dirancang untuk otomatisasi. Lihat [Automation &amp; Scripting](CLI.md#automation--scripting) untuk contoh PowerShell, Batch, dan Python.

***

### Q: Bagaimana cara memeriksa versi CLI?

**A:**

```powershell
chloros-cli --version
```

**Output:**

```
Chloros CLI 1.0.2
```

***

## Mendapatkan Bantuan

### Bantuan Baris Perintah

Lihat informasi bantuan langsung di CLI:

```powershell
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
* **Harga**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Contoh Lengkap

### Contoh 1: Pengolahan Dasar

Pengolahan dengan pengaturan default (vignette, reflektansi):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Contoh 2: Output Ilmiah Berkualitas Tinggi

32-bit float TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Contoh 3: Pemrosesan Pratinjau Cepat

8-bit PNG tanpa kalibrasi untuk tinjauan cepat:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Contoh 4: Pemrosesan yang Dikoreksi PPK

Terapkan koreksi PPK dengan reflektansi:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Contoh 5: Lokasi Keluaran Kustom

Proses ke drive berbeda dengan format spesifik:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Contoh 6: Alur Kerja Otentikasi

Lengkapi alur kerja otentikasi:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Contoh 7: Penggunaan Multi-Bahasa

Ubah bahasa antarmuka:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```

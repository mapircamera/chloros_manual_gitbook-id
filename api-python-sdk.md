# API : Python SDK

**Chloros Python SDK** memberikan akses programatik ke mesin pemrosesan gambar Chloros, memungkinkan otomatisasi, alur kerja kustom, dan integrasi yang mulus dengan aplikasi dan alur kerja penelitian Python Anda.

### Fitur Utama

* 🐍 **Python Asli** - Kode bersih dan Pythonic untuk pemrosesan gambar
* 🔧 **Akses Penuh** - Kontrol penuh atas pemrosesan gambar
* 🚀 **Otomatisasi** - Bangun alur kerja pemrosesan batch kustom
* 🔗 **Integrasi** - Embed Chloros dalam aplikasi Python yang sudah ada
* 📊 **Siap untuk Penelitian** - Sempurna untuk alur kerja analisis ilmiah
* ⚡ **Pemrosesan Paralel** - Skalabel hingga inti CPU Anda (Chloros+)

### Persyaratan

| Persyaratan          | Rincian                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Harus diinstal secara lokal                                           |
| **Lisensi**          | Chloros+ ([paket berbayar diperlukan](https://cloud.mapir.camera/pricing)) |
| **Sistem Operasi** | Windows 10/11 (64-bit)                                              |
| **Python**           | Python 3.7 atau lebih tinggi                                                |
| **Memori**           | Minimal 8GB RAM (disarankan 16GB)                                  |
| **Internet**         | Diperlukan untuk aktivasi lisensi                                     |

{% hint style=&quot;warning&quot; %}
**Persyaratan Lisensi**: Python SDK memerlukan langganan berbayar Chloros+ untuk akses API. Rencana standar (gratis) tidak memiliki akses ke API/SDK. Kunjungi [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) untuk melakukan upgrade.
{% endhint %}

## Panduan Cepat

### Instalasi

Instal melalui pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Pengaturan Awal**: Sebelum menggunakan SDK, aktifkan lisensi Chloros+ Anda dengan membuka Chloros, Chloros (Browser) atau Chloros CLI dan masuk dengan kredensial Anda. Ini hanya perlu dilakukan sekali.
{% endhint %}

### Penggunaan Dasar

Proses folder dengan beberapa baris saja:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Kontrol Penuh

Untuk alur kerja lanjutan:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Panduan Instalasi

### Persyaratan

Sebelum menginstal SDK, pastikan Anda memiliki:

1. **Chloros Desktop** terinstal ([download](download.md))
2. **Python 3.7+** terinstal ([python.org](https://www.python.org))
3. **Lisensi Chloros+ yang aktif** ([upgrade](https://cloud.mapir.camera/pricing))

### Instalasi melalui pip

**Instalasi standar:**

```bash
pip install chloros-sdk
```

**Dengan dukungan pemantauan progres:**

```bash
pip install chloros-sdk[progress]
```

**Instalasi pengembangan:**

```bash
pip install chloros-sdk[dev]
```

### Verifikasi Instalasi

Uji apakah SDK terinstal dengan benar:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Pengaturan Pertama Kali

### Aktivasi Lisensi

SDK menggunakan lisensi yang sama dengan Chloros, Chloros (Browser), dan Chloros CLI. Aktivasi sekali melalui antarmuka pengguna (GUI) atau CLI:

1. Buka **Chloros atau Chloros (Browser)** dan masuk ke tab Pengguna <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Atau, buka **CLI**.  
2. Masukkan kredensial Chloros+ Anda dan masuk  
3. Lisensi disimpan secara lokal (tetap ada setelah reboot)  

{% hint style=&quot;success&quot; %}
**Pengaturan Satu Kali**: Setelah masuk melalui antarmuka pengguna (GUI) atau CLI, SDK secara otomatis menggunakan lisensi yang disimpan. Tidak diperlukan otentikasi tambahan!
{% endhint %}

### Uji Koneksi

Pastikan SDK dapat terhubung ke Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Referensi API

### Kelas ChlorosLocal

Kelas utama untuk pemrosesan gambar lokal Chloros.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parameter:**

| Parameter                 | Tipe | Default                   | Deskripsi                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL dari backend lokal Chloros          |
| `auto_start_backend`      | bool | `True`                    | Mulai backend secara otomatis jika diperlukan |
| `backend_exe`             | str  | `None` (deteksi otomatis)      | Jalan ke executable backend            |
| `timeout`                 | int  | `30`                      | Batas waktu permintaan dalam detik            |
| `backend_startup_timeout` | int  | `60`                      | Batas waktu untuk memulai backend (detik) |

**Contoh:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Metode

#### `create_project(project_name, camera=None)`

Buat proyek Chloros baru.

**Parameter:**

| Parameter      | Tipe | Diperlukan | Deskripsi                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Ya      | Nama proyek                                     |
| `camera`       | str  | Tidak       | Template kamera (misalnya, &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Mengembalikan:** `dict` - Respons pembuatan proyek

**Contoh:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Impor gambar dari folder.

**Parameter:**

| Parameter     | Tipe     | Diperlukan | Deskripsi                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Ya      | Jalan ke folder dengan gambar         |
| `recursive`   | bool     | Tidak      | Cari subfolder (default: False) |

**Hasil:** `dict` - Hasil impor dengan jumlah file

**Contoh:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Konfigurasi pengaturan pemrosesan.

**Parameter:**

| Parameter                 | Tipe | Default                 | Deskripsi                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Kualitas Tinggi (Lebih Cepat)&quot; | Metode Debayer                   |
| `vignette_correction`     | bool | `True`                  | Aktifkan koreksi vignette      |
| `reflectance_calibration` | bool | `True`                  | Aktifkan kalibrasi reflektansi  |
| `indices`                 | daftar | `None`                  | Indeks vegetasi yang akan dihitung |
| `export_format`           | string  | &quot;TIFF (16-bit)&quot;         | Format output                   |
| `ppk`                     | bool | `False`                 | Aktifkan koreksi PPK          |
| `custom_settings`         | dict | `None`                  | Pengaturan kustom lanjutan        |

**Format Ekspor:**

* `"TIFF (16-bit)"` - Direkomendasikan untuk GIS/fotogrametri
* `"TIFF (32-bit, Percent)"` - Analisis ilmiah
* `"PNG (8-bit)"` - Inspeksi visual
* `"JPG (8-bit)"` - Output terkompresi

**Indeks Tersedia:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2, dan lainnya.

**Contoh:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Proses gambar proyek.

**Parameter:**

| Parameter           | Tipe     | Default      | Deskripsi                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Mode pemrosesan: &quot;parallel&quot; atau &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Tunggu hingga selesai                       |
| `progress_callback` | callable | `None`       | Fungsi panggilan balik kemajuan (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Interval polling untuk kemajuan (detik)   |

**Mengembalikan:** `dict` - Hasil pemrosesan

{% hint style=&quot;warning&quot; %}
**Mode Paralel**: Membutuhkan lisensi Chloros+. Secara otomatis menyesuaikan dengan inti CPU Anda (hingga 16 pekerja).
{% endhint %}

**Contoh:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Mendapatkan konfigurasi proyek saat ini.

**Mengembalikan:** `dict` - Konfigurasi proyek saat ini

**Contoh:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Mendapatkan informasi status backend.

**Mengembalikan:** `dict` - Status backend

**Contoh:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Matikan backend (jika dimulai oleh SDK).

**Contoh:**

```python
chloros.shutdown_backend()
```

***

### Fungsi Kemudahan

#### `process_folder(folder_path, **options)`

Fungsi kemudahan satu baris untuk memproses folder.

**Parameter:**

| Parameter                 | Tipe     | Default         | Deskripsi                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Diperlukan        | Jalan ke folder dengan gambar     |
| `project_name`            | str      | Dibuat otomatis  | Nama proyek                   |
| `camera`                  | str      | `None`          | Template kamera                |
| `indices`                 | list     | `["NDVI"]`      | Indeks untuk perhitungan           |
| `vignette_correction`     | bool     | `True`          | Aktifkan koreksi vignette     |
| `reflectance_calibration` | bool     | `True`          | Aktifkan kalibrasi reflektansi |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Format output                  |
| `mode`                    | str      | `"parallel"`    | Mode pemrosesan                |
| `progress_callback`       | dapat dipanggil | `None`          | Panggilan balik kemajuan              |

**Mengembalikan:** `dict` - Hasil pemrosesan

**Contoh:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Dukungan Pengelola Konteks

SDK mendukung pengelola konteks untuk pembersihan otomatis:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Contoh Lengkap

### Contoh 1: Pemrosesan Dasar

Memproses folder dengan pengaturan default:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Contoh 2: Alur Kerja Kustom

Kontrol penuh atas alur pemrosesan:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Contoh 3: Pemrosesan Berkelompok Beberapa Folder

Memproses beberapa dataset penerbangan:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Contoh 4: Integrasi Alur Kerja Penelitian

Integrasikan Chloros dengan analisis data:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Contoh 5: Pemantauan Kemajuan Kustom

Pemantauan kemajuan lanjutan dengan pencatatan:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Contoh 6: Penanganan Kesalahan

Penanganan kesalahan yang andal untuk penggunaan produksi:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Contoh 7: Alat Baris Perintah

Bangun alat CLI kustom dengan SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    
    args = parser.parse_args()
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Penggunaan:**

```bash
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
```

***

## Penanganan Kecuali

SDK menyediakan kelas kecuali khusus untuk jenis kesalahan yang berbeda:

### Hierarki Kecuali

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Contoh Kecelakaan

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Topik Lanjutan

### Konfigurasi Backend Kustom

Gunakan lokasi atau konfigurasi backend kustom:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Pemrosesan Non-Blok

Mulai pemrosesan dan lanjutkan dengan tugas lain:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Manajemen Memori

Untuk dataset besar, proses dalam batch:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Pemecahan Masalah

### Backend Tidak Berjalan

**Masalah:** SDK gagal memulai backend

**Solusi:**

1. Verifikasi Chloros Desktop terinstal:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Periksa apakah Windows Firewall tidak memblokir
3. Coba jalur backend manual:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Lisensi Tidak Terdeteksi

**Masalah:** SDK menampilkan peringatan tentang lisensi yang hilang

**Solusi:**

1. Buka Chloros, Chloros (Browser) atau Chloros CLI dan login.
2. Verifikasi lisensi tersimpan di cache:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Hubungi dukungan: info@mapir.camera

***

### Kesalahan Impor

**Masalah:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Solusi:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Waktu Pengolahan Habis

**Masalah:** Waktu pengolahan habis

**Solusi:**

1. Tingkatkan batas waktu:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Proses dalam batch yang lebih kecil
3. Periksa ruang disk yang tersedia
4. Pantau sumber daya sistem

***

### Port Sudah Digunakan

**Masalah:** Port belakang 5000 terpakai

**Solusi:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Atau temukan dan tutup proses yang bertabrakan:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Tips Kinerja

### Optimalkan Kecepatan Pemrosesan

1. **Gunakan Mode Paralel** (membutuhkan Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Kurangi Resolusi Output** (jika dapat diterima)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Nonaktifkan Indeks yang Tidak Diperlukan**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Proses di SSD** (bukan HDD)

***

### Optimasi Memori

Untuk dataset besar:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Pemrosesan Latar Belakang

Bebaskan Python untuk tugas lain:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Contoh Integrasi

### Integrasi Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## FAQ

### Q: Apakah SDK memerlukan koneksi internet?

**A:** Hanya untuk aktivasi lisensi awal. Setelah masuk melalui Chloros, Chloros (Browser) atau Chloros CLI, lisensi disimpan secara lokal dan dapat digunakan offline selama 30 hari.

***

### Q: Apakah saya dapat menggunakan SDK pada server tanpa antarmuka grafis (GUI)?  

**A:** Ya! Persyaratan:  

* Windows Server 2016 atau versi terbaru  
* Chloros terinstal (sekali saja)  
* Lisensi diaktifkan pada mesin mana pun (lisensi yang disimpan di cache disalin ke server)

***

### Q: Apa perbedaan antara Desktop, CLI, dan SDK?

| Fitur         | Antarmuka Desktop | Baris Perintah CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Antarmuka**   | Klik dan pilih | Perintah          | Python API  |
| **Terbaik untuk**    | Pekerjaan visual | Pemrograman        | Integrasi |
| **Otomatisasi**  | Terbatas     | Baik             | Sangat baik   |
| **Fleksibilitas** | Dasar       | Baik             | Maksimal     |
| **Lisensi**     | Chloros+    | Chloros+         | Chloros+    |

***

### Q: Bisakah saya mendistribusikan aplikasi yang dibangun dengan SDK?

**A:** Kode SDK dapat diintegrasikan ke dalam aplikasi Anda, tetapi:

* Pengguna akhir memerlukan Chloros yang terinstal
* Pengguna akhir memerlukan lisensi Chloros+ yang aktif
* Distribusi komersial memerlukan lisensi OEM

Hubungi info@mapir.camera untuk pertanyaan terkait OEM.

***

### Q: Bagaimana cara memperbarui SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### Q: Di mana gambar yang diproses disimpan?

Secara default, di jalur proyek:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### Q: Apakah saya dapat memproses gambar dari skrip Python yang berjalan secara terjadwal?

**A:** Ya! Gunakan Windows Task Scheduler dengan skrip Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Jadwalkan melalui Task Scheduler untuk dijalankan setiap hari.

***

### Q: Apakah SDK mendukung async/await?

**A:** Versi saat ini bersifat sinkron. Untuk perilaku asinkron, gunakan `wait=False` atau jalankan di thread terpisah:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Mendapatkan Bantuan

### Dokumentasi

* **Referensi API**: Halaman ini

### Saluran Dukungan

* **Email**: info@mapir.camera
* **Situs Web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Harga**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Contoh Kode

Semua contoh yang tercantum di sini telah diuji dan siap digunakan. Salin dan sesuaikan untuk kasus penggunaan Anda.

***

## Lisensi

**Perangkat Lunak Proprietary** - Hak Cipta (c) 2025 MAPIR Inc.

SDK memerlukan langganan Chloros+ yang aktif. Penggunaan, distribusi, atau modifikasi tanpa izin dilarang.

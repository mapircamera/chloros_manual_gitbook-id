# Panduan NVIDIA Jetson

Chloros pada NVIDIA Jetson memungkinkan pemrosesan gambar multispektral di tepi jaringan — di lapangan, pada UAV, dan di instalasi terpencil. Chloros secara otomatis mendeteksi model Jetson Anda dan mengoptimalkan strategi pemrosesannya untuk perangkat keras Anda.

***

## Model Jetson yang Didukung

| Model                | RAM            | Strategi Pemrosesan                                   | Penggunaan yang Direkomendasikan                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB bersama | `GPU_PARALLEL` (4 pekerja)                            | Kinerja maksimum, dataset besar                      |
| **Jetson Orin NX**   | 8-16GB bersama  | `GPU_PARALLEL` (3 pekerja, 16GB) / `GPU_SINGLE` (8GB) | Rekomendasi utama untuk penerapan di udara dan lapangan |
| **Jetson Orin Nano** | 8 GB bersama     | `GPU_SINGLE` (1 pekerja)                               | Komputasi tepi tingkat pemula                                 |
| **Jetson Nano**      | 4-8GB bersama   | `GPU_SINGLE` (1 pekerja)                               | Tingkat pemula, dengan keterbatasan memori                          |

{% hint style="info" %}
**Model Jetson lama** (TX2, TX1, Xavier NX) mungkin tidak didukung. Kinerja akan bervariasi tergantung pada memori GPU yang tersedia dan kemampuan CUDA.
{% endhint %}

***

## Persyaratan

* **JetPack 6.x** (disarankan versi terbaru)
* **NVIDIA CUDA** (termasuk dalam JetPack)
* **Lisensi Chloros+** (diperlukan untuk akses ke CLI/SDK)

## Instalasi

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Untuk detail umum pemasangan Linux, lihat [Pemasangan Linux](linux-installation.md).

***

## Adaptasi Komputasi Dinamis pada Jetson

Chloros secara otomatis mendeteksi model Jetson Anda dan memilih strategi pemrosesan yang optimal. **Tidak diperlukan penyesuaian manual.**

### Cara Kerjanya

Saat startup, Chloros melakukan profil sistem Anda:

1. **Mendeteksi model Jetson** melalui `/proc/device-tree/model`
2. **Membaca memori GPU/bersama yang tersedia**

3.**Memilih strategi pemrosesan** (`GPU_PARALLEL`, `GPU_SINGLE`, atau `CPU_PARALLEL`)
4. **Menetapkan jumlah pekerja, jenis pipa, dan alokasi memori** secara otomatis

### Perilaku Per Model

| Model Jetson                | Strategi       | Pekerja | Jalur                       | Konkuren |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (efisien memori) | Serialized  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serial  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Diserialisasi  |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (jalur GPU penuh)    | Bersamaan  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Bersamaan  |

{% hint style="success" %}
**Jetson Orin NX 16GB** adalah pilihan ideal untuk penerapan di tepi jaringan — perangkat ini menerapkan strategi `GPU_PARALLEL` dengan 3 pekerja bersamaan, menghadirkan pemrosesan GPU paralel yang sesungguhnya dalam bentuk yang ringkas.
{% endhint %}

Perbedaan utama antar platform terletak pada **memori**. Sebuah Jetson Nano dengan memori bersama 8GB harus memproses gambar satu per satu menggunakan pendekatan tiled yang efisien memori, sementara Orin NX dengan 16GB dapat menjalankan 3 gambar melalui GPU secara bersamaan menggunakan pipa terpadu (fused pipeline) berkapasitas throughput lebih tinggi.

Untuk referensi adaptasi komputasi lengkap, lihat [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md).

***

## Manajemen Termal

Perangkat Jetson memiliki ruang termal yang terbatas, terutama dalam deployment tertutup atau di udara. Chloros mencakup pemantauan termal otomatis dan pembatasan kecepatan:

| Suhu                | Tindakan                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Operasi normal — kecepatan pemrosesan penuh          |
| **70°C** (Peringatan)  | Mengurangi ukuran batch secara otomatis                   |
| **80°C** (Kritikal) | Pembatasan agresif — kurangi koncurrency         |
| **90°C** (Matikan) | Hentikan pemrosesan GPU sepenuhnya — pendinginan diperlukan |

{% hint style="warning" %}
**Pastikan ventilasi dan pendinginan yang memadai** untuk pemrosesan berkelanjutan, terutama di dalam wadah lapangan tertutup atau sistem udara. Pembatasan termal akan mengurangi throughput pemrosesan untuk melindungi perangkat keras.
{% endhint %}

***

## Manajemen Memori

Perangkat Jetson menggunakan **memori terpadu** — GPU dan CPU berbagi RAM fisik yang sama. Ini berarti VRAM yang dilaporkan (misalnya, 15,3 GB pada Orin NX 16 GB) bukanlah memori GPU khusus; melainkan dibagi dengan sistem operasi dan proses lainnya.

### Rekomendasi Ruang Tukar

Untuk dataset besar atau pemrosesan debayer Texture Aware, Chloros mungkin merekomendasikan pembuatan ruang tukar:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Perkiraan memori per gambar:**

* Debayer standar: ~10 MB per gambar
* Debayer Texture Aware: ~15 MB per gambar

Chloros secara otomatis menghitung memori yang dibutuhkan berdasarkan ukuran dataset Anda dan memberi peringatan jika swap direkomendasikan.

### OOM (Kehabisan Memori) Cadangan

Jika kondisi kehabisan memori terdeteksi selama pemrosesan:

1. Chloros secara otomatis mengurangi jumlah pekerja GPU
2. Beralih dari pipa `fused_gpu` ke `tiled_gpu` (lebih efisien memori)
3. Memproses terus dengan throughput yang berkurang daripada mengalami crash

***

## Penerapan di Lapangan

### Pertimbangan Daya

| Model Jetson     | Konsumsi Daya Tipikal | Catatan                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10W              | USB-C atau jack barrel    |
| Jetson Orin Nano | 7-15W              | Jack barrel DC          |
| Jetson Orin NX   | 10-25W             | Jack barrel DC          |
| Jetson AGX Orin  | 15-60W             | USB-C PD atau jack barrel |

Rencanakan anggaran daya Anda untuk pemrosesan berkelanjutan — konsumsi daya puncak terjadi selama Thread 3 (Pemrosesan) yang intensif GPU.

### Rekomendasi Penyimpanan

* **NVMe SSD** sangat direkomendasikan untuk implementasi arm64
* Kartu SD terlalu lambat untuk pemrosesan — gunakan hanya sebagai media boot
* Rencanakan ruang 2-3 kali ukuran data gambar mentah Anda untuk output yang telah diproses

### Operasi Tanpa Layar Melalui SSH

Chloros CLI ideal untuk deployment Jetson tanpa layar:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Pemrosesan Otomatis dengan systemd

Buat layanan systemd untuk pemrosesan otomatis:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Pasangkan dengan timer systemd untuk pemrosesan terjadwal:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Contoh Alur Kerja

### Pemrosesan Jetson Dasar

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK pada Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Pemrosesan Batch Beberapa Penerbangan

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistem Jetson yang Direkomendasikan untuk Penggunaan Lapangan

Untuk penerapan di lapangan dan udara, pertimbangkan opsi papan penghubung Jetson Orin NX 16GB berikut:

* **Udara/drone**: Sistem dengan peringkat getaran (MIL-STD), ringan (di bawah 300g), pendinginan pasif
* **Lapangan yang keras**: Casing tahan air IP67/IP69K dengan konektivitas kamera PoE GigE
* **Minimal/anggaran terbatas**: Kit pengembang dengan casing tambahan

Hubungi [Dukungan MAPIR](https://www.mapir.camera/community/contact) untuk rekomendasi perangkat keras spesifik sesuai skenario penerapan Anda.

***

## Langkah Selanjutnya

* [Linux Instalasi](linux-installation.md) — Rincian umum instalasi Linux
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — Referensi lengkap strategi komputasi
* [Pipa Pemrosesan](../processing-architecture/processing-pipeline.md) — Memahami pipa 4-thread
* [CLI : Baris Perintah](../CLI.md) — Referensi lengkap CLI
* [API : Python SDK](../api-python-sdk.md) — Referensi lengkap SDK

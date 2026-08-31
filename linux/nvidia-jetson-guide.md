# Panduan NVIDIA Jetson

Chloros pada NVIDIA Jetson memungkinkan pemrosesan gambar multispektral di tepi jaringan — di lapangan, pada UAV, dan di instalasi terpencil. Chloros 1.2.0 mendeteksi model Jetson Anda saat startup dan mengoptimalkan strategi pemrosesannya sesuai dengan perangkat keras yang ditemukan. **Tidak diperlukan penyesuaian manual.**

***

## Model Jetson yang Didukung

| Model                | RAM            | Strategi Pemrosesan                                     | Penggunaan yang Direkomendasikan                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB bersama | `GPU_PARALLEL` (2 pekerja)                              | Kinerja maksimum, kumpulan data besar                      |
| **Jetson Orin NX**   | 8-16GB bersama  | `GPU_PARALLEL` (2 pekerja, 16GB) / `GPU_SINGLE` (8GB)   | Rekomendasi utama untuk penerapan di udara dan lapangan |
| **Jetson Orin Nano** | 8 GB bersama     | `GPU_SINGLE` (1 pekerja, sekuensial)                     | Komputasi tepi tingkat pemula                                 |

{% hint style="info" %}
Paket arm64 Linux memerlukan **JetPack 6**, yang tersedia pada keluarga Jetson Orin. Model yang lebih lama (Nano, TX2, Xavier NX) tidak dapat menjalankan JetPack 6 dan tidak didukung oleh paket saat ini.
{% endhint %}

***

## Persyaratan

* **JetPack 6.x** (disarankan versi terbaru)
* **NVIDIA CUDA** (termasuk dalam JetPack)
* **Paket berbayar Chloros+** — Tingkat Copper atau lebih tinggi (diperlukan untuk semua akses ke CLI/SDK; diberlakukan di sisi server)

## Instalasi

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Untuk detail umum pemasangan Linux, lokasi berkas, dan pemecahan masalah, lihat [Pemasangan Linux](linux-installation.md).

{% hint style="info" %}
**Letakkan direktori ekstraksi pada penyimpanan yang cepat.** Biner yang telah dikompilasi akan mengekstrak dirinya sendiri ke direktori sementara pada setiap peluncuran — proses ini sangat lambat jika dilakukan dari kartu SD. Chloros secara otomatis menggunakan `/mnt/ssd/tmp` jika file tersebut ada; jika tidak, atur `TMPDIR` ke jalur di NVMe Anda (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptasi Komputasi Dinamis pada Jetson

### Cara Kerjanya

Saat startup, Chloros melakukan profil sistem Anda:

1. **Mendeteksi model Jetson** melalui `/proc/device-tree/model`
2. **Membaca memori GPU/CPU bersama yang tersedia** (Jetson menggunakan memori terpadu)
3. **Memilih strategi pemrosesan** (`GPU_PARALLEL`, `GPU_SINGLE`, atau `CPU_PARALLEL`)
4. **Menetapkan jumlah worker, jenis pipeline, dan alokasi memori** secara otomatis

Keputusan ini didasarkan pada **total RAM bersama**, bukan pada nama model:

* **Total RAM di bawah 12 GB**(semua Jetson 8 GB): `GPU_SINGLE` dengan**1 pekerja — pemrosesan sekuensial yang disengaja**. Memori terlalu terbatas untuk pekerja yang berjalan secara bersamaan, sehingga gambar diproses satu per satu. Pada Jetson dengan**8 GB atau kurang**, Thread 3 melewati kumpulan pekerja sepenuhnya dan menjalankan pekerjaannya per gambar secara in-process.
* **12GB atau lebih**(Orin NX 16GB, AGX Orin): memori terpadu memenuhi syarat untuk `GPU_PARALLEL`, namun jumlah pekerja**dibatasi hingga 2 pada Jetson** — GPU, RAM proses pekerja, dan konteks CUDA per pekerja semuanya menggunakan kolam bersama yang sama, sehingga penambahan pekerja berisiko menyebabkan kegagalan kehabisan memori.

Anda dapat mengganti pilihan otomatis tersebut dengan variabel lingkungan `CHLOROS_STRATEGY` — lihat [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Perilaku Per-Model

| Model Jetson                | Strategi       | Pekerja | Eksekusi                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | Loop sekuensial dalam proses (`tiled_gpu` saat terjadi tekanan memori) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | Loop berurutan dalam proses                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | Proses pekerja bersamaan, jalur `fused_gpu`  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | Proses pekerja bersamaan, jalur `fused_gpu`  |

Perbedaan utama antar platform adalah **memori**. Jetson 8GB harus memproses gambar satu per satu menggunakan pendekatan tiled yang efisien dalam penggunaan memori saat beban kerja tinggi, sedangkan Orin 16GB+ dapat memproses 2 gambar secara bersamaan melalui GPU menggunakan pipa terpadu (fused pipeline) dengan throughput yang lebih tinggi.

### Alokasi GPU per Model

Setiap model Jetson juga memiliki profil perangkat keras yang membatasi seberapa banyak alokasi memori dari pool bersama yang dapat digunakan untuk pemrosesan, serta menentukan skala ukuran batch:

| Model | Batas atas alokasi GPU | Pengali ukuran batch | Disediakan untuk sistem/layar |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

RAM yang terdeteksi menyesuaikan profil: Jetson yang melaporkan **16 GB atau lebih** akan mengalami peningkatan faktor pengali batch sebesar ×1,2. Ukuran batch dasar sebelum diterapkan faktor pengali adalah 8 gambar.

Untuk referensi adaptasi komputasi lengkap, lihat [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md).

***

## Batas Frekuensi GPU untuk Texture Aware pada Nano dan Orin Nano

Proses debayer Texture Aware menjalankan inferensi jaringan saraf GPU, yang dapat memicu **peringatan arus berlebih**pada model Jetson berdaya rendah (kelas 10–15 W) saat kecepatan clock GPU berada pada level maksimum. Sebelum pemrosesan Texture Aware pada**Jetson Nano atau Orin Nano**, Chloros memeriksa frekuensi maksimum GPU dan membatasinya hingga**510 MHz** (510000000) jika saat ini lebih tinggi:

* Jika CLI dapat menulis node sysfs frekuensi GPU, pembatasan tersebut **diterapkan secara otomatis** dan konfirmasi akan ditampilkan.
* Jika tidak (membutuhkan akses root), CLI menampilkan perintah `sudo` yang tepat untuk menerapkan pembatasan secara manual, menunggu sejenak agar Anda dapat membacanya, lalu melanjutkan — pemrosesan tetap berjalan tetapi mungkin menampilkan peringatan arus berlebih.

Untuk menerapkan batas daya secara manual sebelum pemrosesan:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Model dengan daya lebih tinggi (Orin NX 25W, AGX Orin 60W) berjalan pada kecepatan GPU penuh; tidak ada batas daya yang diterapkan. Debayer Standar tidak pernah memicu batas daya pada model mana pun.

{% hint style="info" %}
**Texture Aware pada Jetson selalu memproses satu gambar pada satu waktu.** Setiap pekerja memerlukan konteks CUDA-nya sendiri (~1GB) ditambah salinan model denoiser-nya sendiri, yang tidak dapat ditangani oleh memori terpadu — sehingga pada Jetson, jalur Texture Aware diikat ke satu pekerja dengan akses GPU yang diserialisasikan. Diharapkan Texture Aware akan jauh lebih lambat daripada Standard pada semua Jetson.
{% endhint %}

***

## Manajemen Termal

Perangkat Jetson memiliki ruang gerak termal yang terbatas, terutama dalam pemasangan tertutup atau di udara. Chloros memantau suhu SoC dan secara otomatis membatasi ukuran batch:

| Suhu         | Tindakan                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Operasi normal — kecepatan pemrosesan penuh          |
| **70°C** (Peringatan)  | Ukuran batch dikurangi secara bertahap (100% → 50% antara 70°C dan 80°C) |
| **80°C** (Kritikal) | Pembatasan agresif (50% → 0% antara 80°C dan 90°C) |
| **90°C** (Mati) | Hentikan pemrosesan GPU sepenuhnya — diperlukan pendinginan |

{% hint style="warning" %}
**Pastikan ventilasi dan pendinginan yang memadai** untuk pemrosesan berkelanjutan, terutama di dalam wadah lapangan tertutup atau sistem udara. Pembatasan termal akan mengurangi throughput pemrosesan untuk melindungi perangkat keras.
{% endhint %}

***

## Manajemen Memori

Perangkat Jetson menggunakan **memori terpadu** — GPU dan CPU berbagi RAM fisik yang sama. VRAM yang dilaporkan (misalnya ~15,3 GB pada Orin NX 16 GB) bukanlah memori GPU khusus; melainkan RAM yang sama yang digunakan oleh sistem operasi dan setiap proses lainnya.

### Peringatan dan Rekomendasi Terkait Swap

Sebelum pemrosesan di Jetson, CLI menghitung jumlah gambar RAW di folder input Anda (`.tif`, `.tiff`, `.raw`, `.dng` — pratinjau JPG tidak dihitung), memperkirakan kebutuhan memori puncak yang diperlukan untuk proses tersebut, dan **memberikan peringatan sebelum memulai** jika RAM + swap kemungkinan tidak mencukupi. Peringatan tersebut diberi judul `LOW MEMORY WARNING - Jetson Detected`, mencantumkan jumlah gambar Anda, RAM, swap saat ini, dan perkiraan puncak, lalu memberikan perintah `fallocate` / `chmod` / `mkswap` / `swapon` yang disesuaikan dengan ukuran proyek Anda (tidak pernah kurang dari 8GB). Program akan berhenti sejenak selama beberapa detik agar pesan tersebut tidak hilang saat layar bergulir, kemudian pemrosesan dilanjutkan.**Perkiraan memori yang digunakan oleh peringatan:**

| Mode Debayer | Dasar | Per gambar |
| --- | --- | --- |
| Standar | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (model + waktu eksekusi Python) | ~15 MB |

Peringatan ini muncul ketika perkiraan puncak melebihi RAM + swap dikurangi margin keamanan 1 GB, dan perhitungan hanya mencakup swap **yang didukung file** — pengaturan yang hanya menggunakan zram tetap akan ditandai.

Untuk menambahkan swap secara manual (contoh: 8 GB):



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```### Penanganan OOM (Out of Memory)

Selama pemrosesan, Chloros memantau memori GPU dan mengurangi kinerja secara bertahap alih-alih mengalami crash:

1. Saat pemanfaatan memori GPU melebihi **85%**, ukuran batch dikurangi secara preventif
2. Jika kejadian kehabisan memori (OOM) tetap terjadi, ukuran batch **dipotong setengah**, dan dipotong setengah lagi pada setiap kejadian OOM berikutnya; setiap batch yang berhasil selanjutnya mengurangi penalti tersebut satu langkah
3. Di bawah tekanan yang berkelanjutan, jalur pemrosesan beralih dari `fused_gpu` ke jalur `tiled_gpu` yang lebih efisien dalam penggunaan memori, dan ke pemrosesan CPU sebagai upaya terakhir

***

## Penerapan di Lapangan

### Pertimbangan Daya

| Model Jetson     | Konsumsi Daya Umum | Catatan                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | Soket DC barrel          |
| Jetson Orin NX   | 10–25 W             | Soket DC barrel          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD atau soket barrel |

Rencanakan anggaran daya Anda untuk pemrosesan berkelanjutan — konsumsi daya puncak terjadi selama Thread 3 (Pemrosesan) yang intensif GPU.

### Rekomendasi Penyimpanan

* **SSD NVMe** sangat direkomendasikan untuk penerapan arm64
* Kartu SD terlalu lambat untuk pemrosesan — gunakan hanya sebagai media boot
* Rencanakan ruang penyimpanan sebesar 2–3 kali ukuran data gambar mentah Anda untuk hasil yang telah diproses

### Operasi Tanpa Monitor melalui SSH

Chloros CLI ideal untuk penerapan Jetson tanpa antarmuka grafis:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend Selalu Aktif untuk Sinkronisasi Waktu LATTICE / DAQ-E

Jika Jetson Anda mengontrol kamera LATTICE atau sensor cahaya DAQ-E tanpa antarmuka grafis, aktifkan layanan systemd backend agar grandmaster PTP berjalan terus-menerus (unit ini terpasang tetapi tidak diaktifkan secara default):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Lihat [Instalasi Linux](linux-installation.md#always-on-ptp-for-headless-hosts) untuk detailnya, termasuk cara paket ini membuat port PTP 319/320 dapat diikat tanpa hak akses root.

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

`chloros-cli process` akan keluar dengan nilai non-nol jika proses yang meminta produk tidak menulis gambar apa pun, sehingga status kegagalan systemd berguna untuk pemantauan.

Gabungkan dengan timer systemd untuk pemrosesan terjadwal:

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

### Pemrosesan Dasar Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

### Python dan SDK pada Jetson

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
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistem Jetson yang Direkomendasikan untuk Penggunaan di Lapangan

Untuk penerapan di lapangan dan di udara, pertimbangkan opsi papan induk Jetson Orin NX 16GB berikut ini:

* **Di udara/drone**: Sistem dengan peringkat ketahanan getaran (MIL-STD), ringan (di bawah 300g), dan pendinginan pasif
* **Lapangan yang berat**: Casing tahan air IP67/IP69K dengan konektivitas kamera GigE PoE
* **Minimal/hemat biaya**: Kit pengembang dengan casing tambahan

Hubungi [Dukungan MAPIR](https://www.mapir.camera/community/contact) untuk rekomendasi perangkat keras khusus sesuai skenario penerapan Anda.

***

## Langkah Selanjutnya

* [Pemasangan Linux](linux-installation.md) — Detail umum pemasangan Linux
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — Referensi lengkap strategi komputasi
* [Pipa Pemrosesan](../processing-architecture/processing-pipeline.md) — Memahami pipa 4-thread
* [CLI : Baris Perintah](../CLI.md) — Panduan CLI
* [API : Python SDK](../api-python-sdk.md) — Panduan SDK
* [Referensi CLI](../reference/cli-reference.md) dan [Referensi SDK](../reference/sdk-reference.md) — Daftar lengkap perintah/API untuk versi 1.2.0

# Panduan NVIDIA Jetson

Chloros pada NVIDIA Jetson memungkinkan pemrosesan gambar multispektral di tepi jaringan — di lapangan, pada UAV, dan di instalasi terpencil. Chloros 1.3.0 mendeteksi model Jetson Anda saat startup dan mengoptimalkan strategi pemrosesannya sesuai dengan perangkat keras yang terdeteksi. **Tidak diperlukan penyesuaian manual.**
{% hint style="info" %}
**Paket Jetson 1.3.0 diproses pada GPU Jetson.** `chloros_1.3.0_arm64_jp6.deb` dikembangkan untuk JetPack 6 dan menggunakan build CUDA PyTorch dari JetPack 6, sehingga pemrosesan — termasuk Texture Aware debayer — berjalan di GPU Orin. Paket ini tidak menyertakan CUDA itu sendiri: paket ini menggunakan pustaka CUDA dan cuDNN yang diinstal oleh JetPack 6 (`sudo apt install nvidia-jetpack` pada gambar yang tidak memiliki pustaka tersebut). Jika tidak ada GPU CUDA yang dapat digunakan, pemrosesan berjalan di CPU dan Texture Aware menggunakan debayer Standar.
{% endhint %}

***

## Model Jetson yang Didukung

| Model                | RAM            | Strategi Pemrosesan                                     | Penggunaan yang Direkomendasikan                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64GB bersama | `GPU_PARALLEL` (2 pekerja)                              | Kinerja maksimum, kumpulan data besar                      |
| **Jetson Orin NX**   | 8–16 GB bersama  | `GPU_PARALLEL` (2 pekerja, 16 GB) / `GPU_SINGLE` (8 GB)   | Rekomendasi utama untuk penerapan di lapangan              |
| **Jetson Orin Nano** | 8 GB bersama     | `GPU_SINGLE` (1 pekerja, sekuensial)                     | Komputasi tepi tingkat pemula                                 |

{% hint style="info" %}
Paket Linux arm64 memerlukan **JetPack 6**, yang tersedia pada keluarga Jetson Orin. Model yang lebih lama (Nano, TX2, Xavier NX) tidak dapat menjalankan JetPack 6 dan tidak didukung oleh paket saat ini.
{% endhint %}

***

## Persyaratan

* **JetPack 6.x** (disarankan versi terbaru)
* **NVIDIA CUDA dan cuDNN** dari JetPack 6 (`sudo apt install nvidia-jetpack` jika gambar sistem Anda tidak menyertakannya)
* **Paket Chloros+ berbayar** — tingkatan Copper atau lebih tinggi (diperlukan untuk semua akses CLI/SDK; diberlakukan di sisi server)

## Instalasi

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

`.deb` menginstal SDK Python Python untuk sistem `python3`, yang sesuai dengan build ini. Hanya virtualenv yang memerlukan wheel yang disertakan untuk diinstal di dalamnya: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (sepenuhnya offline). Untuk detail umum instalasi Linux, lokasi berkas, dan pemecahan masalah, lihat [Instalasi Linux](linux-installation.md).

{% hint style="info" %}
**Tidak perlu lagi mengekstrak saat diluncurkan.** Mulai versi 1.3.0, backend dan CLI diinstal dalam bentuk yang sudah diekstrak di bawah `/usr/lib/chloros/`, sehingga keduanya tidak lagi mengekstrak diri ke direktori sementara setiap kali dijalankan. File sementara tetap disimpan ke `/mnt/ssd/tmp` secara otomatis jika folder tersebut ada; jika tidak, Anda dapat mengatur `TMPDIR` ke jalur di NVMe Anda (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptasi Komputasi Dinamis pada Jetson

### Cara Kerjanya

Saat startup, Chloros akan membuat profil sistem Anda:

1. **Mendeteksi model Jetson** melalui `/proc/device-tree/model`
2. **Membaca memori GPU/CPU bersama yang tersedia** (Jetson menggunakan memori terpadu)
3. **Menanyakan kepada PyTorch apakah dapat menggunakan CUDA** — dengan CUDA dan cuDNN dari JetPack 6 yang telah terinstal, hal ini dimungkinkan
4. **Memilih strategi pemrosesan** (`GPU_PARALLEL`, `GPU_SINGLE`, atau `CPU_PARALLEL`) dan jumlah pekerja secara otomatis

Keputusan ini ditentukan oleh **total RAM bersama**, bukan oleh nama model:

* **Jika total RAM di bawah 12GB**(semua Jetson 8GB):**1 pekerja — pemrosesan sekuensial yang disengaja**. Memori terlalu terbatas untuk pekerja bersamaan, sehingga gambar diproses satu per satu. Pada Jetson dengan**8GB atau kurang**, Thread 3 melewati kumpulan pekerja sepenuhnya dan menjalankan tugas per gambar secara in-process. Log backend menamai strategi ini `GPU_SINGLE`.
* **12GB atau lebih**(Orin NX 16GB, AGX Orin): memori terpadu memenuhi syarat untuk `GPU_PARALLEL`, tetapi jumlah pekerja**dibatasi hingga 2 pada Jetson** — GPU, RAM proses pekerja, dan konteks CUDA per pekerja semuanya menggunakan pool bersama yang sama, sehingga semakin banyak pekerja berisiko mengalami kegagalan karena kehabisan memori.
* **Tidak ada CUDA yang dapat digunakan** (pustaka CUDA JetPack tidak ada): `CPU_PARALLEL` pada 12GB atau lebih, satu gambar pada satu waktu di bawah ini.

Anda dapat mengganti pilihan otomatis tersebut dengan variabel lingkungan `CHLOROS_STRATEGY` — lihat [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Perilaku Per Model

| Model Jetson                | Strategi       | Pekerja | Eksekusi                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | Loop berurutan dalam proses (`tiled_gpu` saat tekanan memori) |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 1       | Loop berurutan dalam proses                     |
| **Jetson Orin NX 16GB**     | `GPU_PARALLEL` | 2       | Proses pekerja bersamaan, jalur `fused_gpu`  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 2       | Proses pekerja bersamaan, jalur `fused_gpu`  |

Perbedaan utama antar platform adalah **memori**. Jetson 8GB harus memproses gambar satu per satu menggunakan pendekatan tiled yang efisien dalam penggunaan memori saat beban tinggi, sedangkan Orin 16GB+ dapat memproses 2 gambar secara bersamaan melalui GPU menggunakan pipa terpadu (fused pipeline) dengan throughput yang lebih tinggi.

### Anggaran GPU per Model

Setiap model Jetson juga memiliki profil perangkat keras yang membatasi seberapa banyak pemrosesan yang dapat diambil dari pool bersama, serta menentukan skala ukuran batch:

| Model | Batas atas anggaran GPU | Pengali ukuran batch | Disediakan untuk sistem/layar |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

RAM yang terdeteksi menyesuaikan profil: Jetson yang melaporkan **16 GB atau lebih** akan mengalami peningkatan faktor pengali batch sebesar ×1,2. Ukuran batch dasar sebelum diterapkan faktor pengali adalah 8 gambar.

Untuk referensi adaptasi komputasi lengkap, lihat [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md).

***

## Batas Frekuensi GPU untuk Texture Aware pada Nano dan Orin Nano

Proses debayer Texture Aware menjalankan inferensi jaringan saraf GPU, yang dapat memicu **peringatan arus berlebih**pada model Jetson berdaya rendah (kelas 10–15 W) saat kecepatan clock GPU berada pada level maksimum. Saat Anda menjalankan perintah `--debayer texture-aware` pada**Jetson Nano atau Orin Nano**, CLI akan memeriksa frekuensi maksimum GPU dan membatasinya hingga **510 MHz** (510000000) jika frekuensi saat ini lebih tinggi:

* Jika CLI dapat menulis node sysfs frekuensi GPU, pembatasan tersebut **diterapkan secara otomatis** dan konfirmasi akan ditampilkan.
* Jika tidak (membutuhkan akses root), CLI akan menampilkan perintah `sudo` yang tepat untuk menerapkan pembatasan secara manual, menunggu sejenak agar Anda dapat membacanya, lalu melanjutkan — pemrosesan tetap berjalan tetapi mungkin menampilkan peringatan arus berlebih.

Untuk menerapkan batasan ini sendiri sebelum pemrosesan:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Model berdaya tinggi (Orin NX 25W, AGX Orin 60W) berjalan pada kecepatan GPU penuh; tidak ada batasan yang diterapkan. Debayer Standar tidak pernah memicu batasan pada model mana pun.

{% hint style="info" %}
**Texture Aware pada Jetson selalu memproses satu gambar pada satu waktu.** Setiap worker memerlukan konteks CUDA tersendiri (~1GB) ditambah salinan model denoiser tersendiri, yang tidak dapat ditangani oleh memori terpadu — sehingga pada Jetson, jalur Texture Aware dibatasi pada satu worker dengan akses GPU yang diserialisasikan. Diharapkan Texture Aware akan jauh lebih lambat daripada Standard pada semua perangkat Jetson. Texture Aware berlaku untuk gambar Survey3: hasil tangkapan LATTICE selalu menggunakan debayer Standard. Tanpa GPU CUDA yang dapat digunakan, proses akan menggunakan mode Standard, dan backend mencatat satu baris `[DEBAYER] … needs a CUDA GPU …` yang menjelaskan alasannya (ditampilkan bersama `chloros-cli --verbose`).
{% endhint %}

***

## Manajemen Termal

Perangkat Jetson memiliki ruang termal yang terbatas, terutama dalam penerapan tertutup atau di udara. Chloros memantau suhu SoC dan secara otomatis membatasi ukuran batch:

| Suhu                | Tindakan                                            |
| ------------------- | ------------------------------------------------- |
| **< 70°C**          | Operasi normal — kecepatan pemrosesan penuh          |
| **70°C** (Peringatan) | Ukuran batch berkurang secara bertahap (100% → 50% antara 70°C dan 80°C) |
| **80°C** (Kritis) | Pembatasan agresif (50% → 0% antara 80°C dan 90°C) |
| **90°C** (Mati) | Hentikan pemrosesan GPU sepenuhnya — diperlukan pendinginan |

{% hint style="warning" %}
**Pastikan ventilasi dan pendinginan yang memadai** untuk pemrosesan berkelanjutan, terutama di dalam wadah lapangan tertutup atau sistem udara. Pembatasan termal akan mengurangi throughput pemrosesan untuk melindungi perangkat keras.
{% endhint %}

***

## Manajemen Memori

Perangkat Jetson menggunakan **memori terpadu** — GPU dan CPU berbagi RAM fisik yang sama. VRAM yang dilaporkan (misalnya ~15,3 GB pada Orin NX 16 GB) bukanlah memori GPU khusus; melainkan RAM yang sama yang digunakan oleh sistem operasi dan setiap proses lainnya.

### Peringatan dan Rekomendasi Terkait Swap

Sebelum melakukan pemrosesan di Jetson, CLI akan menghitung jumlah gambar RAW di folder input Anda (`.tif`, `.tiff`, `.raw`, `.dng` — pratinjau JPG tidak dihitung), memperkirakan kebutuhan memori puncak yang diperlukan untuk proses tersebut, dan **memberikan peringatan sebelum memulai** jika RAM + swap kemungkinan tidak mencukupi. Peringatan tersebut berjudul `LOW MEMORY WARNING - Jetson Detected`, menampilkan jumlah gambar Anda, RAM, swap saat ini, dan perkiraan puncak, lalu memberikan perintah tepat `fallocate` / `chmod` / `mkswap` / `swapon` yang disesuaikan dengan ukuran proyek Anda (tidak pernah kurang dari 8 GB). Sistem akan berhenti sejenak selama beberapa detik agar pesan tersebut tidak hilang saat layar bergulir ke atas, kemudian pemrosesan dilanjutkan.**Perkiraan memori yang digunakan oleh peringatan ini:**

| Mode Debayer | Dasar | Per gambar |
| --- | --- | --- |
| Standar | ~1,5 GB | ~10 MB |
| Texture Aware | ~2,5 GB (model + runtime Python) | ~15 MB |

Peringatan ini muncul ketika perkiraan puncak melebihi RAM + swap dikurangi margin keamanan 1 GB, dan perhitungan hanya mencakup swap **yang didukung file** — konfigurasi yang hanya menggunakan zram tetap akan ditandai.

Untuk menambahkan swap secara manual (contoh: 8 GB):

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
```

<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->


### Penanganan OOM (Out of Memory)

Selama pemrosesan, Chloros memantau memori GPU dan mengurangi beban secara bertahap alih-alih mengalami crash:

1. Ketika pemanfaatan memori GPU melebihi **85%**, ukuran batch dikurangi secara preventif
2. Jika kejadian kehabisan memori (OOM) tetap terjadi, ukuran batch **dipotong setengah**, dan dipotong setengah lagi pada setiap kejadian OOM berikutnya; setiap batch yang berhasil selanjutnya akan mengurangi penalti tersebut satu langkah
3. Di bawah tekanan yang berkelanjutan, pipa pemrosesan akan beralih dari jalur `fused_gpu` ke jalur `tiled_gpu` yang lebih efisien dalam penggunaan memori, dan ke pemrosesan CPU sebagai upaya terakhir

Karena GPU berbagi RAM Jetson, RAM ditambah swap masih membatasi durasi eksekusi — peringatan swap di atas adalah hal yang perlu diperhatikan.

***

## Penerapan di Lapangan

### Pertimbangan Daya

| Model Jetson     | Konsumsi Daya Tipikal | Catatan                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | Soket DC barrel          |
| Jetson Orin NX   | 10–25 W             | Soket DC barrel          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD atau soket DC barrel |

Rencanakan anggaran daya Anda untuk pemrosesan berkelanjutan — konsumsi daya puncak terjadi selama Thread 3 (Pemrosesan) yang intensif GPU.

### Rekomendasi Penyimpanan

* **SSD NVMe** sangat direkomendasikan untuk penerapan arm64
* Kartu SD terlalu lambat untuk pemrosesan — gunakan hanya sebagai media boot
* Rencanakan ruang penyimpanan sebesar 2-3 kali ukuran data gambar mentah Anda untuk hasil pemrosesan

### Operasi Tanpa Antarmuka Grafis (Headless) melalui SSH


Chloros CLI sangat ideal untuk implementasi Jetson tanpa antarmuka grafis:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend Selalu Aktif untuk Sinkronisasi Waktu LATTICE / DAQ-E

Jika Jetson Anda mengontrol kamera LATTICE atau sensor cahaya DAQ-E tanpa antarmuka grafis, aktifkan layanan sistemd backend **sebagai pengguna Anda** agar grandmaster PTP berjalan terus-menerus dan skrip SDK dapat menemukan backend yang sedang berjalan (unit ini terpasang tetapi tidak diaktifkan secara default):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

Backend tersebut kemudian berjalan sebagai Anda: data tangkapan yang ditulisnya adalah milik Anda, dan backend tersebut menggunakan `chloros-cli login` Anda. Lihat [Instalasi Linux](linux-installation.md#always-on-ptp-for-headless-hosts) untuk detailnya, termasuk akun root `chloros-backend.service` dan cara paket ini membuat port PTP 319/320 dapat diikat tanpa hak akses root.

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

`chloros-cli process` akan keluar dengan nilai non-nol jika eksekusi yang meminta produk tidak menulis gambar apa pun, sehingga status kegagalan systemd bermakna untuk pemantauan.

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
    --indices NDVI GNDVI OSAVI
```

### SDK Python di Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Sistem Jetson yang Direkomendasikan untuk Penggunaan di Lapangan

Untuk penerapan di lapangan dan di udara, pertimbangkan opsi papan induk Jetson Orin NX 16GB berikut ini:

* **Di udara/drone**: Sistem dengan peringkat ketahanan getaran (MIL-STD), ringan (di bawah 300 g), dan pendinginan pasif
* **Lapangan yang berat**: Casing tahan air IP67/IP69K dengan konektivitas kamera GigE PoE
* **Minimal/hemat biaya**: Kit pengembang dengan casing tambahan

Hubungi [Dukungan MAPIR](https://www.mapir.camera/community/contact) untuk rekomendasi perangkat keras khusus sesuai skenario penerapan Anda.

***

## Langkah Selanjutnya

* [Instalasi Linux](linux-installation.md) — Detail umum instalasi Linux

* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — Referensi lengkap strategi komputasi
* [Pipa Pemrosesan](../processing-architecture/processing-pipeline.md) — Memahami pipa 4-thread
* [CLI : Baris Perintah](../CLI.md) — Panduan CLI

* [API: Python SDK](../api-python-sdk.md) — Panduan SDK

* [Referensi CLI](../reference/cli-reference.md) dan [Referensi SDK](../reference/sdk-reference.md) — Daftar lengkap perintah/API untuk versi 1.3.0

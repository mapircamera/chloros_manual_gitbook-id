# Instalasi Linux

Chloros didistribusikan untuk Linux sebagai paket `.deb` yang menginstal CLI dan server backend. Python SDK adalah paket pip terpisah (juga disertakan di dalam `.deb` sebagai wheel yang sesuai versi).

Nama file paket mencantumkan versi dan arsitektur: `chloros_1.2.0_amd64.deb` untuk x86_64, dan `chloros_1.2.0_arm64_jp6.deb` untuk build JetPack 6 Jetson. Gantilah dengan file yang sebenarnya Anda unduh pada perintah di bawah ini.

***

## Linux amd64 (x86_64)

### Persyaratan Sistem

| Persyaratan | Minimum | Direkomendasikan |
| --- | --- | --- |
| **Distribusi** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Prosesor** | x86_64 (Intel/AMD) | Intel Core i7 atau yang lebih baik |
| **Memori (RAM)** | 8GB | 16GB atau lebih |
| **Kartu grafis** | Tidak diperlukan (diproses oleh CPU) | GPU NVIDIA dengan VRAM 4GB+ (12GB+ membuka fitur `GPU_PARALLEL`, 7GB+ menjaga Texture Aware tetap nonaktif pada jalur gambar tunggal) |
| **Penyimpanan** | Ruang kosong 2GB | SSD dengan ruang kosong 10GB+ |
| **Python** | Python 3.7+ (untuk SDK) | Python 3.10+ |

> **Ubuntu 20.04 dan Debian 11 tidak didukung.** Daftar ketergantungan `.deb`
> diturunkan dari apa yang sebenarnya ditautkan oleh backend Chloros, dan itu mencakup
> `libc6 (>= 2.34)`. Baik Focal maupun bullseye sama-sama menyertakan glibc 2.31, sehingga `apt` langsung menolak
> pemasangan tersebut daripada membiarkannya gagal nanti saat runtime.

### Instalasi

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` tidak dapat menyelesaikan ketergantungan. Jika dilaporkan ada paket yang hilang, `sudo apt-get install -f` (atau `sudo apt --fix-broken install`) akan menyelesaikan proses instalasi — ini adalah jalur normal, bukan kesalahan.
{% endhint %}

Verifikasi instalasi:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Persyaratan Sistem

| Persyaratan | Minimum | Direkomendasikan |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson dengan JetPack 6 | Jetson Orin NX 16GB atau AGX Orin |
| **JetPack** | JetPack 6.x | JetPack 6 versi terbaru |
| **Memori (RAM)** | 8GB (bersama GPU/CPU) | 16GB+ bersama (12GB+ adalah batas minimum untuk pekerja GPU paralel) |
| **Penyimpanan** | Ruang kosong 2GB | SSD NVMe dengan ruang kosong 10GB+ |
| **Python** | Python 3.7+ (untuk SDK) | Python 3.10+ |

### Instalasi

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Tata letak yang sama dengan amd64 `.deb`, dengan build CUDA yang dioptimalkan untuk Jetson Orin / Orin NX / Orin Nano. Untuk perilaku memori, termal, dan penerapan di lapangan Jetson, lihat [Panduan NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Pemasangan Python dan SDK (Semua Linux)

SDK adalah klien murni Python HTTP untuk backend, sehingga paket yang sama dapat berjalan di amd64 dan arm64. Dua sumber:**Dari PyPI** — rilis stabil yang telah dipublikasikan:

```bash
pip install chloros-sdk
```

**Dari file wheel yang disertakan** — dijamin cocok dengan CLI/backend yang baru saja Anda instal (gunakan ini jika versi `.deb` Anda lebih baru daripada yang ada di PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Distribusi PEP 668** (Ubuntu 23.10+, Debian 12+) menolak instalasi pip secara sistem. Gunakan `pip install --user …`, lingkungan virtual, atau `sudo pip install --break-system-packages …`. Penginstal paket tidak pernah secara otomatis menginstal SDK ke dalam sistem Python Anda — pilihan tersebut diserahkan kepada Anda.
{% endhint %}

Fitur tambahan opsional:

| Ekstra | Perintah | Menambahkan |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` untuk streaming kemajuan secara langsung |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` untuk transportasi BLE (DAQ-M) |

Verifikasi SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` menginstal Chloros, CLI, dan backend. Python dan SDK berkomunikasi dengan backend tersebut melalui jaringan lokal HTTP dan API (`http://127.0.0.1:5000`) dan memulainya secara otomatis saat diperlukan. Selalu gunakan alamat IPv4 literal daripada `localhost` — karena `localhost` dapat diterjemahkan menjadi `::1` dan memakan waktu sekitar dua detik per permintaan.
{% endhint %}

***

## Pengaturan Awal

### 1. Masuk

Akses ke CLI dan SDK memerlukan tingkatan berbayar Chloros+ (**Copper** atau lebih tinggi), yang diberlakukan di sisi server: pemanggil yang tidak masuk akan mendapatkan `401 AUTH_REQUIRED`, sedangkan pemanggil dengan paket gratis (Iron) akan mendapatkan `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Kredensial disimpan dalam cache di `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Anda harus masuk kembali setelah setiap instalasi atau pembaruan.** Skrip `prerm` dari paket ini secara sengaja menghapus `~/.chloros/user_session.json` dan lisensi yang disimpan dalam cache untuk setiap pengguna di mesin tersebut, sehingga build baru selalu memvalidasi ulang lisensi alih-alih mengandalkan cache yang sudah kadaluwarsa.
{% endhint %}

### 2. Periksa Status Lisensi Anda

```bash
chloros-cli status
```

`chloros-cli status` berfungsi di semua tingkatan (termasuk yang gratis), sehingga Anda selalu dapat mengetahui mengapa akses tersedia atau tidak.

### 3. Jalankan Diagnostik Sistem

```bash
chloros-cli selftest
```

Tujuh pemeriksaan dijalankan secara berurutan, dan perintah akan keluar dengan nilai non-nol jika salah satunya gagal:

| # | Pemeriksaan | Apa yang dibuktikan |
| --- | --- | --- |
| 1 | **Versi** | CLI melaporkan versinya (`v1.2.0`). |
| 2 | **Port tersedia** | Port 5000 kosong, *atau* sudah dibalas oleh backend Chloros yang berfungsi dengan baik (yang dianggap lulus). |
| 3 | **Pengaktifan backend** | Biner backend berhasil dijalankan. |
| 4 | **Uji API (`/api/test`)** | Backend merespons `status: ok`. |
| 5 | **Info sistem** | Menampilkan `GPU: <name>, CUDA: <bool>, PyTorch: <version>` dari `/api/system-info`. |
| 6 | **Model denoiser** | Menemukan model `*.pth.enc` (pada Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| Texture Aware sebenarnya dapat digunakan — memerlukan CUDA**dan** setidaknya satu berkas model. |

Proses berakhir dengan `N/7 checks passed`, mencantumkan setiap kegagalan berdasarkan namanya.

### 4. Memproses Kumpulan Data Pertama Anda

```bash
chloros-cli process ~/datasets/flight001
```

***

## Berkas dan Direktori

### Per Pengguna

Chloros menyimpan kredensial dan konfigurasi CLI dalam satu direktori lintas platform, **`~/.chloros/`** (pada Windows, `%USERPROFILE%\.chloros\`). Dua cache khusus Linux justru mengikuti konvensi XDG — cache ini akan mengikuti pengaturan `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` jika telah ditetapkan.

| Jalur | Tujuan |
| --- | --- |
| `~/.chloros/user_session.json` | Cache sesi login yang ditulis oleh `chloros-cli login` (dihapus setiap kali menginstal/memperbarui paket) |
| `~/.chloros/working_directory.txt` | Penggantian folder proyek default (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferensi bahasa CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Pengaturan bahasa yang dibagikan dengan antarmuka pengguna (GUI) Windows — pengaturan di sini (`language`) memiliki prioritas lebih tinggi daripada `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache selama satu jam untuk pemeriksaan pembaruan saat startup Linux/Jetson |
| `~/.chloros/backend.log` | Log backend saat backend diluncurkan oleh CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Paket kalibrasi LATTICE per kamera yang disimpan dalam cache, diindeks berdasarkan nomor seri dan hash bundel |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Pengaturan pengguna opsional untuk profil koreksi batas DAQ |
| `~/.config/chloros/system_config.json` | Profil perangkat keras yang disimpan dalam cache dari Dynamic Compute Adaptation — hapus untuk memaksa deteksi perangkat keras baru |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Log server backend, satu berkas per peluncuran |
| `~/Chloros Projects/` | Folder proyek default jika tidak ada pengaturan pengganti |

### Seluruh Sistem

| Jalur | Tujuan |
| --- | --- |
| `/usr/bin/chloros-cli` | Skrip pembungkus — mengatur `LD_LIBRARY_PATH` untuk pustaka asli yang disertakan, lalu menjalankan biner yang sebenarnya |
| `/usr/bin/chloros-backend` | Skrip pembungkus — sama, ditambah `CHLOROS_PRODUCTION=1` sehingga gerbang otentikasi backend tidak pernah dapat menonaktifkan dirinya sendiri secara diam-diam |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Biner yang telah dikompilasi |
| `/usr/lib/chloros/arena_runtime/` | Runtime Arena SDK yang diperlukan oleh kamera LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Model denoiser terenkripsi yang digunakan oleh Texture Aware debayer |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK wheel yang sesuai dengan build ini persis |
| `/usr/lib/chloros/exiftool` | exiftool yang disertakan (ditautkan secara simbolis ke `/usr/local/bin/exiftool` hanya jika tidak ada exiftool sistem) |
| `/etc/chloros/update.conf` | Konfigurasi saluran pembaruan yang dibaca oleh `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Mengatur `net.ipv4.ip_unprivileged_port_start = 319` agar backend dapat mengikat port PTP tanpa hak akses root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Mengarahkan pemuat dinamis ke `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Memberikan akses kepada pengguna yang masuk ke jembatan serial USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Layanan backend yang selalu aktif (diinstal, **belum diaktifkan**) |
| `/usr/share/applications/chloros-cli.desktop` | Entri menu aplikasi &quot;Chloros CLI&quot; yang membuka terminal |

## Lokasi Eksekusi Backend

CLI dan SDK mendeteksi backend secara otomatis:

| Komponen | Jalur |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Ganti jalur backend dengan bendera `--backend-exe` CLI atau parameter konstruktor `backend_exe` SDK, dan port dengan `--port` (default `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` mengarahkan keluarga perintah **`lattice`**,**`project`**, dan**`daq pool-*`** ke backend jarak jauh. Perintah inti (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) sengaja mengabaikannya dan selalu menargetkan `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamera LATTICE dan Sensor Cahaya DAQ pada Linux

Semua rangkaian perintah live-hardware berfungsi pada Linux (amd64 dan Jetson):

* **`chloros-cli lattice`** — mendeteksi, menghubungkan, mengonfigurasi, dan merekam data dari kamera LATTICE serta susunan yang disinkronkan. `.deb` menyertakan runtime Arena SDK yang diperlukan dan mendaftarkannya ke pemuat dinamis.
* **`chloros-cli daq pool-*`** — menghubungkan sensor cahaya DAQ-U/M/E melalui pool backend, mengalirkan spektrum yang telah dikalibrasi, dan merekam berkas `.daq`. CLI yang telah dikompilasi hanya menyertakan keluarga `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — menjalankan proyek yang telah disimpan (termasuk kamera, sensor, dan pengaturan pemrosesannya) tanpa antarmuka pengguna.
* **`chloros-cli time-sync`** — memeriksa grandmaster PTP yang dijalankan oleh backend Chloros untuk kamera LATTICE dan sensor DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` diperlukan oleh `pool-latest`, `pool-stream`, `pool-record`, dan `pool-set-cap`; `pool-list` menampilkan ID yang saat ini ada di pool.

{% hint style="info" %}
**Pilihlah `--eth-host` untuk koneksi DAQ-E pertama pada mesin multi-homed.** Penemuan otomatis (auto-discovery) menelusuri mDNS dan dapat melewatkan antarmuka sensor akibat cache ARP yang kosong, sehingga koneksi `pool-connect --eth` pertama setelah boot mungkin gagal meskipun sensor dalam kondisi sempurna. Dengan memasukkan alamat IP atau nama host sensor, proses penemuan otomatis dapat dilewati sepenuhnya.
{% endhint %}

**Izin serial DAQ-U** ditangani oleh aturan udev yang terpasang (`uaccess` + grup `dialout`). Jika sensor yang sudah terpasang tetap tidak dapat diakses, muat ulang aturan tersebut atau pasang ulang sensor:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Lihat [referensi CLI](../CLI.md) untuk daftar perintah lengkap.

### PTP Selalu Aktif untuk Host Tanpa Antarmuka Grafis

Pada instalasi pertama, unit systemd `chloros-backend.service` dihasilkan tetapi **tidak diaktifkan**. Pada Jetson tanpa layar atau server yang harus menjaga sinkronisasi waktu PTP tetap berjalan terus-menerus untuk sensor DAQ-E dan kamera LATTICE, aktifkan unit tersebut:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Tanpa unit ini, PTP hanya berjalan selama backend Chloros aktif — yaitu, selama sesi CLI/SDK sedang berlangsung.

Unit ini mengikat backend ke pengaturan lingkungan `127.0.0.1:5000` (pengaturan lingkungan `CHLOROS_HOST` / `CHLOROS_PORT` di dalam unit; ganti dengan `sudo systemctl edit chloros-backend.service`) dan memulainya kembali jika terjadi kegagalan setelah 5 detik.

**Bagaimana PTP mendapatkan portnya.** PTP menggunakan UDP 319/320, keduanya di bawah batas bawah port istimewa normal 1024. Paket `postinst` menulis `/etc/sysctl.d/60-chloros-ptp.conf` dengan `net.ipv4.ip_unprivileged_port_start = 319`, yang memungkinkan backend mengikat port-port tersebut saat berjalan sebagai pengguna Anda. Paket ini juga menerapkan `setcap cap_net_bind_service,cap_net_raw=+ep` ke biner backend sebagai langkah pengamanan ganda — itulah mengapa `libcap2-bin` dinyatakan sebagai dependensi paket ini.***

## Contoh Skrip Bash

{% hint style="info" %}
**Kode keluar yang ramah skrip.**`chloros-cli process` mengembalikan kode keluar `0` jika berhasil dan**kode keluar bukan nol jika gagal — termasuk saat menjalankan perintah yang meminta produk citra tetapi tidak menghasilkan apa pun** (skrip ini menampilkan `Processing finished but wrote no image products.` serta menyebutkan nama folder proyek dan penyebab umum kegagalan). Eksekusi yang berhasil melaporkan jumlah produk gambar yang dihasilkan (`Image products written: N`). Kode keluar: `0` berhasil, `1` gagal, `2` kesalahan argumen, `130` terputus.
{% endhint %}

### Memproses Beberapa Kumpulan Data

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Memproses dengan Pengaturan Khusus

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Nilai `--format` yang valid berjumlah tepat empat, dan mengandung spasi — selalu masukkan dalam tanda kutip:

| Nilai `--format` | Folder keluaran |
| --- | --- |
| `TIFF (16-bit)` *(default)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` menerima `standard` (default) atau `texture-aware` (Chloros+).

### Pemrosesan Otomatis dengan Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Contoh Python dan SDK

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

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Izin Ditolak

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### &quot;setcap gagal&quot; Selama Instalasi

`.deb` menerapkan `cap_net_bind_service` ke `/usr/lib/chloros/chloros-backend` agar dapat mengikat port PTP 319/320 tanpa hak akses root. Jika `libcap2-bin` tidak ada saat instalasi, panggilan tersebut akan dilewati. Instal paket tersebut dan instal ulang paketnya:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP Tidak Dapat Dimulai / Tidak Dapat Mengikat Port 319

Pastikan batas port tanpa hak istimewa telah diturunkan, dan terapkan kembali untuk boot saat ini jika belum:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Kemudian periksa grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### &quot;Driver kamera LATTICE tidak ditemukan&quot;

Runtime Arena SDK tidak dapat diselesaikan. Pastikan konfigurasi loader yang ditulis oleh paket tersebut ada dan telah diperbarui:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
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

Log backend untuk kegagalan peluncuran terdapat di `~/.cache/chloros/logs/`.

### CUDA Tidak Terdeteksi

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` melaporkan hal yang sama dalam satu baris: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Perpustakaan Bersama Hilang

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Pengaktifan Lambat pada Sistem Kartu SD

Biner yang telah dikompilasi akan mengekstrak dirinya sendiri ke direktori sementara pada setiap kali diluncurkan. Jika `/mnt/ssd/tmp` ada, Chloros akan menggunakannya secara otomatis; jika tidak, atur `TMPDIR` ke sistem berkas yang cepat:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Memperbarui Chloros di Linux

Perintah `update` hanya tersedia di Linux/Jetson. Perintah ini memeriksa versi yang dipublikasikan di saluran pembaruan yang dikonfigurasi pada `/etc/chloros/update.conf` dan menawarkan untuk mengunduh serta menginstal `.deb` yang sesuai:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Pada Linux/Jetson, CLI juga menjalankan pemeriksaan pembaruan non-blocking pada setiap kali startup (hasilnya disimpan dalam cache selama satu jam di `~/.chloros/update_cache.json`) dan menampilkan pesan `Update available: vX.Y.Z` jika terdapat versi yang lebih baru. Pengaturan dan proyek Anda tetap utuh setelah pembaruan; Anda perlu masuk kembali setelahnya.

## Menghapus Instalasi

```bash
sudo apt remove chloros
```

Penghapusan akan menghentikan `chloros-backend.service`, mengembalikan batas bawah port tanpa hak istimewa ke nilai default (1024), menghapus tautan simbolis exiftool yang disertakan dan konfigurasi pemuat Arena, serta menghapus kredensial yang disimpan dalam cache. Proyek dan berkas data `~/.chloros/` Anda tetap utuh.

***

## Langkah Selanjutnya

* [Panduan NVIDIA Jetson](nvidia-jetson-guide.md) — Optimasi dan penerapan khusus untuk Jetson
* [CLI : Baris Perintah](../CLI.md) — panduan CLI
* [API : Python SDK](../api-python-sdk.md) — panduan SDK
* [Referensi CLI](../reference/cli-reference.md) dan [Referensi SDK](../reference/sdk-reference.md) — daftar perintah/API yang lengkap untuk versi 1.2.0
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — cara Chloros beradaptasi dengan perangkat keras Anda

# Instalasi Linux

Chloros didistribusikan untuk Linux dalam bentuk paket `.deb` yang menginstal antarmuka baris perintah (CLI), server backend, dan SDK Python Python (untuk sistem `python3`, ditambah file wheel yang sesuai versinya untuk virtualenvs).

Nama file paket mencantumkan versi dan arsitektur: `chloros_1.3.0_amd64.deb` untuk x86_64, dan `chloros_1.3.0_arm64_jp6.deb` untuk arm64 (NVIDIA Jetson, JetPack 6). Ganti dengan berkas yang sebenarnya Anda unduh pada perintah di bawah ini.

{% hint style="info" %}
**Baru di versi 1.3.0: program-program ini dikirimkan dalam bentuk yang sudah diekstrak.** Backend dan CLI diinstal sebagai folder, yaitu `/usr/lib/chloros/backend/` dan `/usr/lib/chloros/cli/`, bukan sebagai file self-extracting tunggal. Keduanya tidak lagi mengekstrak dirinya sendiri ke folder sementara setiap kali dijalankan, sehingga prosesnya jauh lebih cepat (pada x86_64, backend biasanya membutuhkan waktu sekitar 100 detik untuk mengekstrak setiap kali dimulai) dan tidak lagi memerlukan ruang kosong beberapa GB di `/tmp`. Ukuran `.deb` untuk amd64 juga sekitar setengah dari ukuran sebelumnya.
{% endhint %}

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

> **Ubuntu 20.04 dan Debian 11 tidak didukung.** Daftar dependensi `.deb`
> berasal dari apa yang sebenarnya ditautkan oleh backend Chloros, dan itu mencakup
> `libc6` yang lebih baru daripada glibc 2.31 yang disertakan dalam rilis focal dan bullseye, sehingga `apt` langsung menolak
> proses instalasi daripada membiarkannya gagal nanti saat runtime.

### Instalasi

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Gunakan `apt install ./…`, bukan `dpkg -i`.** `apt` menyelesaikan ketergantungan paket tersebut dan juga menginstal apa yang direkomendasikannya: persyaratan Python SDK, `python3-numpy` dan `python3-requests`. `dpkg -i` tidak menginstal keduanya. CLI, dan backend bersifat mandiri dan dapat berjalan tanpa komponen tersebut, tetapi `import chloros_sdk` membutuhkannya; hasil instalasi akan mencantumkan nama komponen yang hilang. Pertahankan `./`, agar `apt` menginstal berkas tersebut alih-alih mencari paket dengan nama tersebut. Paket ini juga mendeklarasikan `libgl1`, yang dimuat oleh OpenCV yang disertakan dan mungkin tidak tersedia pada gambar server minimal.
{% endhint %}

Verifikasi instalasi:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Persyaratan Sistem

| Persyaratan | Minimum | Direkomendasikan |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson dengan JetPack 6 | Jetson Orin NX 16GB atau AGX Orin |
| **JetPack** | JetPack 6.x | JetPack 6 versi terbaru |
| **Memori (RAM)** | 8GB (bersama GPU/CPU) | 16GB+ bersama (12GB+ adalah batas minimum untuk pemrosesan paralel) |
| **Penyimpanan** | Ruang kosong 2GB | SSD NVMe dengan ruang kosong 10GB+ |
| **Python** | Python 3.7+ (untuk SDK) | Python 3.10+ |

### Instalasi

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Tata letak yang sama dengan amd64 `.deb` (Python SDK, termasuk buffer penerima dan layanan backend), untuk Jetson Orin / Orin NX / Orin Nano pada **JetPack 6** (L4T R36). PyTorch-nya adalah build CUDA JetPack 6 untuk Orin, sehingga pemrosesan GPU, termasuk Texture Aware debayer, berjalan di GPU Jetson. Paket ini tidak menyertakan CUDA itu sendiri: paket ini menggunakan pustaka CUDA dan cuDNN yang diinstal oleh JetPack 6 (`sudo apt install nvidia-jetpack` pada gambar yang tidak menyertakannya). Jika tidak ada GPU CUDA yang dapat digunakan, pemrosesan akan berjalan di CPU dan `--debayer texture-aware` akan menggunakan debayer Standar (backend akan mencatat satu baris `[DEBAYER] … needs a CUDA GPU …` yang menjelaskan alasannya). Untuk informasi mengenai memori, suhu, dan perilaku penerapan di lapangan pada Jetson, lihat [Panduan NVIDIA Jetson](nvidia-jetson-guide.md).***

## Instalasi Python SDK (Semua Linux)**`.deb` akan menginstal SDK untuk Anda.** Kedua paket (amd64 dan Jetson) menginstalnya untuk sistem `python3`, sehingga `import chloros_sdk` langsung berfungsi setelah instalasi, tanpa perlu pip maupun koneksi jaringan. Paket-paketnya (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api`, dan pemuat proyek) dimasukkan ke dalam `/usr/lib/python3/dist-packages`, yang dimiliki oleh paket tersebut: pembaruan akan menggantikannya, dan menghapus paket tersebut akan menghapusnya. Versi selalu sesuai dengan CLI dan backend yang Anda instal.

Persyaratan SDK itu sendiri, yaitu `python3-numpy` dan `python3-requests`, **direkomendasikan, bukan wajib** oleh paket: `sudo apt install ./chloros_<version>_<arch>.deb` menginstalnya, sedangkan `dpkg -i` tidak. Output instalasi melaporkan versi SDK yang diimpor oleh `python3`, persyaratan apa pun yang hilang, serta adanya `chloros_sdk` di tempat lain pada jalur `python3` yang menimpa salinan paket tersebut (seperti yang diinstal sebelumnya dengan `pip install --user`).**Di dalam virtualenv**, yang tidak dapat mengakses paket sistem, instal wheel yang disertakan ke dalamnya (sepenuhnya offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Dari PyPI** — hanya untuk mesin yang tidak memiliki paket Chloros terinstal, di mana SDK-nya menggerakkan backend di mesin lain (lihat [Mode Backend Jarak Jauh](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Fitur tambahan opsional, untuk instalasi melalui pip (di lingkungan virtualenv atau PyPI):

| Fitur Tambahan | Perintah | Menambahkan |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` untuk streaming kemajuan secara langsung |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` untuk transportasi BLE (DAQ-M) |

Verifikasi SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python SDK berkomunikasi dengan backend yang diinstal oleh `.deb` melalui HTTP API lokal (`http://127.0.0.1:5000`). `ChlorosLocal` dan helper koneksi akan memulainya secara otomatis saat diperlukan; `connect_all()` dari proyek yang disimpan tidak pernah melakukannya, jadi pada mesin tanpa antarmuka grafis, jalankan backend sebagai [layanan](#always-on-ptp-for-headless-hosts). Selalu gunakan alamat IPv4 secara literal daripada `localhost` — `localhost` dapat diterjemahkan menjadi `::1` dan memakan waktu sekitar dua detik per permintaan.
{% endhint %}

***

## Pengaturan Awal

### 1. Masuk

Akses CLI dan SDK memerlukan tingkatan Chloros+ berbayar (**Copper** atau lebih tinggi), yang diterapkan di sisi server: pemanggil yang tidak masuk akan mendapatkan `401 AUTH_REQUIRED`, sedangkan pemanggil dengan tingkatan gratis (Iron) akan mendapatkan `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Kredensial disimpan dalam cache di `~/.chloros/user_session.json`.

{% hint style="info" %}
**Peningkatan versi mempertahankan login Anda; penghapusan akan menghapusnya.** Menginstal versi yang lebih baru (`.deb`) di atas instalasi yang sudah ada (termasuk melalui `chloros-cli update`) akan mempertahankan `~/.chloros/user_session.json` dan lisensi yang disimpan dalam cache, sehingga tugas CLI/SDK tanpa antarmuka pengguna tetap berfungsi. Menghapus paket (`sudo apt remove chloros`) akan menghapus data login yang tersimpan dan cache lisensi untuk setiap pengguna di mesin tersebut, sehingga instalasi baru memerlukan `chloros-cli login` lagi.
{% endhint %}

### 2. Periksa Status Lisensi Anda

```bash
chloros-cli status
```

`chloros-cli status` berfungsi di semua tingkatan (termasuk gratis), sehingga Anda selalu dapat mengetahui mengapa akses tersedia atau tidak.

### 3. Jalankan Diagnostik Sistem

```bash
chloros-cli selftest
```

Tujuh pemeriksaan dijalankan secara berurutan, dan perintah akan keluar dengan nilai non-nol jika salah satunya gagal:

| # | Pemeriksaan | Apa yang dibuktikan |
| --- | --- | --- |
| 1 | **Versi** | CLI melaporkan versinya (`v1.3.0`). |
| 2 | **Port tersedia** | Port 5000 kosong, *atau* sudah direspon oleh backend Chloros yang berfungsi dengan baik (hal ini dianggap lulus). |
| 3 | **Pengaktifan backend** | Biner backend berhasil dijalankan. |
| 4 | **Uji API (`/api/test`)** | Backend merespons dengan `status: ok`. |
| 5 | **Info sistem** | Menampilkan `GPU: <name>, CUDA: <bool>, PyTorch: <version>` dari `/api/system-info`. |
| 6 | **Model denoiser** | Menemukan model `*.pth.enc` (di Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Denoiser**| Texture Aware sebenarnya dapat digunakan — memerlukan CUDA**dan** setidaknya satu berkas model. |

Proses berakhir dengan `N/7 checks passed`, mencantumkan setiap kegagalan berdasarkan namanya.

### 4. Memproses Dataset Pertama Anda

```bash
chloros-cli process ~/datasets/flight001
```

***

## Berkas dan Direktori

### Per Pengguna

Chloros menyimpan kredensial dan konfigurasi CLI-nya dalam satu direktori lintas platform, **`~/.chloros/`** (di Windows, `%USERPROFILE%\.chloros\`). Dua cache khusus Linux mengikuti konvensi XDG — cache ini akan menggunakan `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` jika telah ditetapkan.

| Jalur | Tujuan |
| --- | --- |
| `~/.chloros/user_session.json` | Cache sesi login yang ditulis oleh `chloros-cli login` (tetap ada saat pembaruan paket; dihapus saat paket dihapus) |
| `~/.chloros/working_directory.txt` | Penggantian folder proyek default (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferensi bahasa antarmuka baris perintah (CLI) (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Pengaturan bahasa yang dibagikan dengan antarmuka pengguna Windows — nilai `language` di sini memiliki prioritas lebih tinggi daripada `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache selama satu jam untuk pemeriksaan pembaruan saat startup Linux/Jetson |
| `~/.chloros/backend.log` | Log backend saat backend diluncurkan oleh CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Paket kalibrasi LATTICE per kamera yang disimpan dalam cache, diindeks berdasarkan nomor seri dan hash bundel |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Penggantian profil koreksi cap DAQ oleh pengguna (opsional) |
| `~/.config/chloros/system_config.json` | Profil perangkat keras yang disimpan dalam cache dari Dynamic Compute Adaptation — hapus untuk memaksa deteksi perangkat keras baru |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Log server backend, satu berkas per peluncuran |
| `~/Documents/MAPIR/Chloros Projects/` | Folder proyek default jika tidak ada pengaturan pengganti |

### Seluruh Sistem

| Jalur | Tujuan |
| --- | --- |
| `/usr/bin/chloros-cli` | Skrip pembungkus — mengatur `LD_LIBRARY_PATH` untuk pustaka asli yang disertakan (dan `TMPDIR` menjadi `/mnt/ssd/tmp` jika folder tersebut ada), kemudian menjalankan biner yang sebenarnya |
| `/usr/bin/chloros-backend` | Skrip pembungkus — sama untuk backend, yang ditandai sebagai build produksi yang terinstal |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | CLI dan backend yang telah dikompilasi, diinstal dalam bentuk yang belum dikompres (fitur baru di versi 1.3.0); program-program tersebut adalah `cli/chloros-cli` dan `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Skrip peluncur dua baris yang menjalankan program-program sebenarnya di atas — disimpan di jalur ini karena SDK, CLI, dan skrip lama mencarinya di sini |
| `/usr/lib/chloros/arena_runtime/` | Runtime Arena SDK yang diperlukan oleh kamera LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Model denoiser terenkripsi yang digunakan oleh Texture Aware debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (dan `lattice_sdk`, `lattice_cam`, `arena_api`, pemuat proyek) | Python SDK, diinstal untuk sistem `python3` dan dimiliki oleh paket |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Roda Python SDK yang sesuai dengan build ini persis, untuk virtualenv |
| `/usr/lib/chloros/exiftool` | Peluncur ExifTool yang disertakan (ditautkan secara simbolis ke `/usr/local/bin/exiftool` hanya jika tidak ada exiftool sistem). Mulai versi 1.3.0, skrip ExifTool dilengkapi dengan pustaka Perl-nya sendiri di `/usr/lib/chloros/exiftool-dist/`, sehingga dapat berjalan tanpa paket sistem `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Konfigurasi saluran pembaruan yang dibaca oleh `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Menyetel `net.ipv4.ip_unprivileged_port_start = 319` agar backend dapat mengikat port PTP tanpa hak akses root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Meningkatkan ukuran buffer penerimaan soket kernel (`net.core.rmem_max`, `net.core.rmem_default`) menjadi 32 MB untuk aliran kamera LATTICE, dan menerapkannya saat instalasi (nilai yang sudah diatur lebih tinggi dibiarkan apa adanya). Pada pengaturan default kernel, paket kamera akan dibuang di kernel dan sebuah array akan kehilangan frame serta laju bingkai; `chloros-cli lattice network` melaporkan status buffer saat kapasitasnya rendah. Lihat [Host Linux — buffer penerimaan soket](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Mengarahkan pemuat dinamis ke `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Memberikan akses kepada pengguna yang masuk ke jembatan serial USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Layanan backend selalu aktif (opt-in) yang menjalankan backend sebagai pengguna yang Anda tentukan (terpasang, **belum diaktifkan**) — layanan yang perlu diaktifkan, lihat [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Layanan yang sama yang menjalankan backend sebagai **root**(terpasang,**belum diaktifkan**), dipertahankan agar mesin yang mengaktifkannya tetap berjalan |
| `/var/log/chloros/` | Log terperinci layanan root, dapat dibaca oleh grup `adm` (dihapus saat paket dihapus sepenuhnya) |
| `/usr/share/applications/chloros-cli.desktop` | Entri menu aplikasi "Chloros CLI" yang membuka terminal |

**Firewall (baru di versi 1.3.0).** Jika `ufw` atau `firewalld` aktif saat instalasi, paket ini akan membuka port UDP `5002` (data DAQ-E), `5353` (penemuan mDNS), `319`, dan `320` (PTP), serta menghapus aturan-aturan tersebut saat paket dihapus. Jika firewall terpasang tetapi tidak aktif, firewall tersebut tidak menambahkan apa pun dan mencantumkan port-port yang harus diizinkan jika Anda mengaktifkannya nanti. Firewall tersebut tidak membuka port TCP apa pun: pembaruan firmware DAQ-E, di mana sensor terhubung kembali ke komputer ini melalui TCP untuk mengumpulkan gambar, akan gagal dengan kode kesalahan `… device never connected back on TCP <port> …` jika firewall yang aktif memblokir koneksi tersebut.

## Lokasi Eksekutabel Backend

CLI dan SDK mendeteksi backend secara otomatis:

| Komponen | Jalur |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (peluncur untuk `/usr/lib/chloros/backend/chloros-backend`) |

Ganti jalur backend dengan bendera CLI `--backend-exe` atau parameter konstruktor SDK `backend_exe`, dan port dengan `--port` (default `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` mengarahkan keluarga perintah **`lattice`**, keluarga perintah**`project`**, dan**`daq pool-*`** ke backend jarak jauh. Perintah inti (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) secara sengaja mengabaikannya dan selalu menargetkan `http://127.0.0.1:<port>`.
{% endhint %}

***

## Kamera LATTICE dan Sensor Cahaya DAQ di Linux

Semua keluarga perintah live-hardware berfungsi di Linux (amd64 dan Jetson):

* **`chloros-cli lattice`** — mendeteksi, menghubungkan, mengonfigurasi, dan merekam dari kamera LATTICE serta array yang disinkronkan. `.deb` menggabungkan runtime Arena SDK yang diperlukan dan mendaftarkannya ke pemuat dinamis.
* **`chloros-cli daq pool-*`** — menghubungkan sensor cahaya DAQ-U/M/E melalui kumpulan backend, mengalirkan spektrum yang telah dikalibrasi, merekam berkas `.daq` dan menukar profil cap, serta untuk DAQ-E, mengubah pengaturan jaringannya, me-restart-nya, dan memperbarui firmware-nya. `chloros-cli daq discover` memindai sensor yang dapat Anda hubungkan, dan `chloros-cli daq --help` mencantumkan subperintah `pool-*`.
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
**`--eth-host` melewati proses pencarian.** `pool-connect --eth` menemukan DAQ-E melalui mDNS, dengan cadangan ARP yang berfungsi dari cache ARP yang kosong; dengan memasukkan alamat IP atau nama host sensor menggunakan `--eth-host`, koneksi langsung terjalin.
{% endhint %}

**Izin serial DAQ-U** ditangani oleh aturan udev yang terpasang (`uaccess` + grup `dialout`). Jika sensor yang sudah terpasang tetap tidak dapat diakses, muat ulang aturan tersebut atau pasang ulang sensor:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Lihat [referensi CLI](../CLI.md) untuk daftar perintah lengkap.

### PTP Selalu Aktif untuk Host Tanpa Antarmuka Grafis

Paket ini menginstal dua layanan systemd untuk backend, yang keduanya tidak diaktifkan. Pada Jetson tanpa antarmuka grafis atau server yang harus menjaga sinkronisasi waktu PTP tetap berjalan terus-menerus untuk sensor DAQ-E dan kamera LATTICE, atau di mana skrip SDK menghubungkan proyek yang disimpan (`connect_all()` tidak pernah memulai backend), aktifkan layanan tersebut **sebagai pengguna Anda**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` menjalankan backend sebagai pengguna tersebut: data tangkapan yang ditulisnya adalah milik pengguna tersebut, log terperincinya ada di `~/.cache/chloros/logs` miliknya, dan layanan ini menggunakan `chloros-cli login` (`~/.chloros`), jadi masuklah sekali sebagai pengguna tersebut terlebih dahulu. Proses pemasangan paket akan menampilkan perintah ini dengan nama pengguna Anda terisi.

Tanpa layanan, backend (dan bersamaan dengan itu grandmaster PTP) hanya berjalan selama ada yang menjalankannya — perintah `chloros-cli` atau skrip SDK.

Unit ini mengikat backend ke pengaturan lingkungan `127.0.0.1:5000` (pengaturan lingkungan `CHLOROS_HOST` / `CHLOROS_PORT` di dalam unit; ganti dengan `sudo systemctl edit chloros-backend@.service`) dan memulainya kembali jika terjadi kegagalan setelah 5 detik.

**Layanan root.**`chloros-backend.service` menjalankan backend sebagai**root**. Layanan ini tetap terpasang sehingga mesin yang mengaktifkannya tetap berjalan, tetapi semua yang ditulis oleh backend root menjadi milik root, sehingga pengguna tidak dapat menghapus tangkapan mereka sendiri; log-nya berada di `/var/log/chloros`, yang dapat dibaca oleh grup `adm`. Untuk beralih ke pengguna Anda:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Backend layanan bersifat bersama, tidak pernah dihentikan oleh klien.** `chloros-cli` dan skrip SDK menggunakannya apa adanya: `process` berjalan di atasnya, memuat proyeknya sendiri menggantikan proyek terakhir. Di mana CLI biasanya akan me-restart backend (`--restart`, atau `process` yang impornya tidak menemukan apa pun), CLI tersebut menyebutkan nama unit, menampilkan pesan "`sudo systemctl restart <unit>`", dan keluar dari "`1`", karena backend yang dihentikan dengan cara tersebut akan tetap mati. Mulai ulang sendiri dengan perintah "`sudo systemctl restart chloros-backend@$USER.service`". Backend layanan tidak pernah menghentikan dirinya sendiri saat idle, dan `chloros-cli backend stop` menolaknya, dengan menyebutkan perintah `systemctl stop`.**Bagaimana PTP mendapatkan port-nya.** PTP menggunakan UDP 319/320, keduanya berada di bawah batas port istimewa normal 1024. Paket `postinst` menulis `/etc/sysctl.d/60-chloros-ptp.conf` dengan `net.ipv4.ip_unprivileged_port_start = 319`, yang memungkinkan backend mengikat port tersebut saat berjalan sebagai pengguna Anda. Sejak versi 1.3.0, paket ini tidak menetapkan kemampuan berkas (`setcap`) pada backend, dan `libcap2-bin` tidak lagi menjadi dependensi: pada backend yang telah diekstrak, kemampuan berkas akan menghentikan prosesnya agar tidak dapat dimulai, jadi jangan tambahkan sendiri.***

## Contoh Skrip Bash

{% hint style="info" %}
**Kode keluar yang ramah skrip.**`chloros-cli process` akan keluar dengan kode `0` jika berhasil dan**kode non-nol jika gagal — termasuk eksekusi yang meminta produk gambar tetapi tidak menulis satupun** (skrip ini menampilkan `Processing finished but wrote no image products.` serta menyebutkan nama folder proyek dan penyebab umum kegagalan). Eksekusi yang berhasil melaporkan jumlah produk gambar yang dihasilkan (`Image products written: N`, ditambah `(plus M clamp masks)` jika masker dihasilkan). Kode keluar: `0` (berhasil), `1` (gagal), `2` (kesalahan argumen), `130` (terputus).
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Nilai `--format` yang valid berjumlah tepat empat, dan mengandung spasi — selalu masukkan dalam tanda kutip:

| Nilai `--format` | Folder keluaran |
| --- | --- |
| Nilai `TIFF (16-bit)` *(default)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` menerima `standard` (default) atau `texture-aware` (Chloros+).

### Pemrosesan Otomatis dengan Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Contoh SDK Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Backend Tidak Dapat Dimulai Setelah Menambahkan Kemampuan Berkas

Jangan jalankan `setcap` di backend. Sejak versi 1.3.0, paket ini tidak menetapkan kemampuan berkas apa pun, dan port PTP 319/320 berasal dari pengaturan `sysctl` yang dijelaskan di [Bagaimana PTP Mendapatkan Portnya](#always-on-ptp-for-headless-hosts). Kemampuan berkas akan menempatkan backend yang tidak dikompresi dalam mode eksekusi aman di mana backend tersebut tidak dapat menemukan pustaka-pustakanya sendiri, sehingga tidak dapat dimulai. Jika Anda telah menambahkan kemampuan berkas tersebut, hapuslah:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP Tidak Dapat Dimulai / Tidak Dapat Mengikat Port 319

Pastikan batas bawah port tanpa hak istimewa telah diturunkan, dan terapkan kembali untuk boot saat ini jika belum:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Kemudian periksa grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Perintah `lattice` Gagal dengan Pesan "Driver kamera LATTICE tidak ditemukan"

Backend tidak dapat memuat runtime kamera LATTICE yang terpasang bersamanya. Setiap perintah `lattice` dijalankan di backend; CLI itu sendiri tidak memuat driver kamera. Instal ulang Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): perintah `.deb` menginstal runtime di samping backend, di dalam `/usr/lib/chloros/arena_runtime`, dan mendaftarkannya ke pemuat dinamis. Untuk memeriksa pendaftaran tersebut:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Log backend untuk kegagalan peluncuran terdapat di `~/.cache/chloros/logs/`. Jika port 5000 milik layanan backend (`systemctl status chloros-backend@$USER.service`), mulailah ulang layanan tersebut daripada menghentikan prosesnya.

### CUDA Tidak Terdeteksi

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` melaporkan hal yang sama dalam satu baris: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Pustaka Bersama Hilang

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Pengaktifan Lambat pada Sistem Kartu SD

Mulai versi 1.3.0, backend dan CLI diinstal dalam bentuk yang belum di-unpack, sehingga keduanya tidak lagi mengekstrak dirinya sendiri ke direktori sementara setiap kali diluncurkan. Berkas sementara akan disimpan ke `/mnt/ssd/tmp` secara otomatis jika folder tersebut ada; jika tidak, Anda dapat mengarahkan berkas-berkas tersebut ke sistem berkas yang cepat:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Memperbarui Chloros di Linux

Perintah `update` hanya tersedia di Linux/Jetson. Perintah ini memeriksa versi yang dipublikasikan di saluran pembaruan yang dikonfigurasi di `/etc/chloros/update.conf` dan menawarkan untuk mengunduh serta menginstal versi yang sesuai di `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Di Linux/Jetson, CLI juga menjalankan pemeriksaan pembaruan non-blocking pada setiap kali startup (hasilnya disimpan dalam cache selama satu jam di `~/.chloros/update_cache.json`) dan menampilkan pesan `Update available: vX.Y.Z` jika terdapat versi yang lebih baru. Pengaturan, proyek, dan sesi login Anda tetap utuh setelah pembaruan.

Peningkatan paket akan menghentikan semua layanan backend selama proses instalasi, sehingga skrip yang sedang berjalan akan kehilangan akses ke kamera dan sensornya, lalu memulai kembali setiap layanan yang diaktifkan (layanan yang dimulai secara manual tetapi tidak pernah diaktifkan akan tetap berhenti, seperti setelah reboot).

## Menghapus Instalasi

```bash
sudo apt remove chloros
```

Penghapusan akan menghentikan dan menonaktifkan semua layanan backend, mengembalikan batas bawah port tanpa hak istimewa ke nilai default (1024), menghapus berkas pengaturan buffer penerimaan (nilai yang sedang berjalan tetap berlaku hingga reboot), tautan simbolis exiftool yang disertakan, konfigurasi Arena loader, aturan firewall yang ditambahkan oleh paket ini, serta Python SDK yang diinstalnya, dan menghapus kredensial yang tersimpan dalam cache. Proyek dan berkas data `~/.chloros/` Anda tetap utuh. `sudo apt purge chloros` juga menghapus `/var/log/chloros`.

***

## Langkah Selanjutnya

* [Panduan NVIDIA Jetson](nvidia-jetson-guide.md) — Optimasi dan penerapan khusus Jetson
* [CLI : Baris Perintah](../CLI.md) — panduan CLI
* [API : Python SDK](../api-python-sdk.md) — panduan SDK
* [Referensi CLI](../reference/cli-reference.md) dan [Referensi SDK](../reference/sdk-reference.md) — daftar lengkap perintah/API untuk versi 1.3.0
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) — cara Chloros menyesuaikan diri dengan perangkat keras Anda

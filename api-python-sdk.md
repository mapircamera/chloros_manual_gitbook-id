# API : Python SDK

{% hint style="info" %}
**Mencari API yang lengkap?** Halaman ini adalah tutorial praktis. Setiap kelas publik, metode, tanda tangan yang tepat, dan contoh yang dapat disalin dan ditempelkan terdapat di [Referensi SDK](reference/sdk-reference.md), yang dioptimalkan untuk asisten AI.**Sedang menggunakan asisten AI?** Tempelkan URL ini ke dalam obrolan agar asisten AI memiliki Chloros 1.2.0 API yang lengkap dan terbaru:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Setiap halaman panduan ini tersedia dalam format markdown mentah di slug huruf kecilnya + `.md`, dan seluruh panduan diindeks di `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` di PyPI) mengendalikan semua fungsi yang dapat dilakukan oleh aplikasi desktop dari Python: pemrosesan gambar secara batch, kontrol kamera dan array LATTICE secara langsung, sesi sensor cahaya DAQ, serta otomatisasi proyek yang disimpan. Ini merupakan lapisan tipis di atas backend lokal yang sama yang digunakan oleh antarmuka pengguna grafis (GUI) dan CLI (HTTP di `127.0.0.1:5000`), sehingga perilakunya identik di ketiga antarmuka tersebut.

## Instalasi

Proses instalasi terdiri dari dua langkah: pertama, instal paket desktop Chloros (yang menyediakan backend pemrosesan dan runtime perangkat keras), kemudian instal paket Python.

**Langkah 1 — Instal Chloros.** Windows: jalankan penginstal desktop (jalur default `C:\Program Files\MAPIR\Chloros\`) dari halaman [Unduh](download.md). Linux: instal paket `.deb` ([Pemasangan Linux](linux/linux-installation.md)).**Langkah 2 — Instal SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Anda mungkin bahkan tidak memerlukan pip: setiap penginstal menyertakan wheel SDK yang sesuai. Penginstal Windows secara otomatis menginstalnya ke dalam sistem Python Anda; penginstal Linux `.deb` menempatkannya di `/usr/lib/chloros/sdk/` dan menampilkan perintah yang tepat `pip install --user`. PyPI diperbarui pada rilis build, sehingga `pip install chloros-sdk` sesuai dengan rilis stabil terbaru.

**Langkah 3 — Masuk sekali per mesin:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kredensial disimpan dalam cache di `~/.chloros/` (kedua platform). Di Windows, Anda dapat masuk dengan cara yang setara melalui tab User <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> pada aplikasi desktop. SDK memerlukan paket berbayar Chloros+ — lihat [Persyaratan lisensi](#license-requirement) di bawah ini.

| Persyaratan | Detail |
| --- | --- |
| **Chloros terinstal** | Windows: penginstal desktop; Linux: paket `.deb` (menyediakan biner backend) |
| **Python** | 3.7 atau lebih tinggi (dikembangkan/diuji pada 3.10) |
| **Sistem operasi** | Windows 10/11 64-bit, Ubuntu 22.04 LTS atau yang lebih baru, atau NVIDIA Jetson (JetPack 6) |
| **Lisensi** | Akun Chloros+ yang aktif, paket berbayar apa pun (Copper atau lebih tinggi) |

## Keunggulan dalam 60 detik

Satu panggilan cukup untuk membuat proyek, mengimpor folder, mengonfigurasi pemrosesan, dan menjalankan pipeline — serta memulai backend secara otomatis jika belum berjalan:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Pada Linux, gunakan jalur Linux: `/home/user/drone_images/flight001`. SDK berfungsi sama di kedua platform.)

Memproses folder tangkapan LATTICE? Gunakan wrapper yang kompatibel dengan LATTICE — wrapper ini menerapkan pengaturan default yang tepat (tanpa deteksi target panel, debayer standar):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — kontrol alur kerja penuh

Untuk apa pun yang lebih rumit dari perintah satu baris, gunakan `ChlorosLocal`. Perintah ini akan menjalankan backend saat pertama kali digunakan (`auto_start_backend=True`), membuat dan mengonfigurasi proyek, memantau kemajuan, serta menampilkan ringkasan setelah proses selesai.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Pertahankan `http://127.0.0.1:5000` default daripada menggantinya dengan `localhost` — pada Windows, `localhost` akan diterjemahkan menjadi `::1` terlebih dahulu dan memakan waktu ~2 detik per permintaan terhadap backend yang hanya mendukung IPv4.
{% endhint %}

Gunakan sebagai pengelola konteks untuk memastikan pembersihan:

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` menerima kata kunci berikut: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation`, dan `custom_settings`. Nilai-nilai utama:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Tombol-tombol khusus LATTICE (`input_level`, `radiometric_output`, serta keluarga `array_alignment*`) didokumentasikan beserta tabel nilai lengkapnya dalam [Referensi SDK](reference/sdk-reference.md#supported-values).

### Memantau kemajuan

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Membaca ringkasan pasca-eksekusi — dan mendeteksi eksekusi kosong

Setelah selesai, `process()` melampirkan ringkasan pemrosesan backend sebagai `result["summary"]`. Setiap entri dalam `summary["hints"]` berupa kalimat lengkap yang menjelaskan hal-hal penting — misalnya, mengapa suatu eksekusi tidak menghasilkan keluaran apa pun — dan setiap petunjuk juga dikeluarkan kembali sebagai Python `UserWarning`, sehingga eksekusi kosong dapat didiagnosis sendiri meskipun Anda tidak pernah memeriksa kamus tersebut:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` tidak memicu kesalahan ketika suatu eksekusi tidak menghasilkan gambar apa pun.** Inilah satu-satunya perbedaan yang disengaja antara SDK dan CLI: `chloros-cli process` menganggap &quot;produk diminta, tetapi tidak ada yang ditulis&quot; sebagai kegagalan dan keluar dengan nilai non-nol, sedangkan SDK kembali secara normal dan melaporkan kondisi tersebut melalui `summary` / petunjuk. Jika pipeline Anda seharusnya berhenti saat eksekusi kosong, periksalah sendiri — periksa `summary` (atau hitung file di bawah folder proyek) daripada mengandalkan pengecualian.
{% endhint %}

## Smart Connect — perangkat keras aktif

Tiga helper membuka sesi persisten di kumpulan perangkat keras backend — kumpulan yang sama yang digunakan oleh GUI, sehingga skrip SDK dapat berjalan berdampingan dengan aplikasi desktop tanpa berebut port serial atau bandwidth jaringan. Ketiganya secara otomatis memulai backend lokal jika tidak ada yang sedang berjalan.

### Kamera LATTICE tunggal — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Susunan yang disinkronkan — `connect_array`

`connect_array` adalah titik masuk yang direkomendasikan untuk rig multi-kamera. Skrip ini menjalankan alur persiapan cerdas yang sama seperti GUI: analisis jaringan, pemilihan otomatis tingkat sinkronisasi, sinkronisasi waktu PTP, pemilihan format piksel per kamera, pengaturan awal AE, dan pengaktifan pemicu GPIO. **Serial pertama adalah master** (yang memicu pulsa pemicu perangkat keras); sisanya adalah slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Tambahkan `smart=True` ke pengambilan gambar array apa pun untuk menunggu hingga pengaturan eksposur otomatis stabil di semua kamera sebelum memicu pemicu. Untuk mode pengambilan gambar (Tunggal / Berkelanjutan / Interval / Tercepat), perekam, burst-to-video, dan penyelarasan array, lihat [Referensi SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor cahaya DAQ — `connect_daq_sensor`

Tanpa argumen, `connect_daq_sensor()` mendeteksi secara cerdas jenis transportasi (urutan prioritas: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Setiap frame membawa nilai 135-point `spectrum` (W/m²/nm setelah dikalibrasi), bendera `is_saturated`, dan CIE `x`, `y`, serta `z`. Untuk menentukan sensor atau protokol transmisi tertentu — pilihan yang andal pada host dengan beberapa antarmuka jaringan, di mana penemuan otomatis Ethernet mungkin tidak mendeteksi DAQ-E yang berfungsi dengan baik pada upaya pertama — berikan satu petunjuk eksplisit:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Perhatikan bahwa profil koreksi kapitalisasi (`cap_id`) **bukanlah** pengaturan SDK — pilihlah melalui `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Proyek yang disimpan — `open_project`

Proyek Chloros yang disimpan akan mempertahankan perangkat keras yang terhubung (`cameras.json` + `sensors.json` bersama dengan `project.json`), dan `chloros_sdk.open_project(path)` dapat menghubungkan kembali semuanya sekaligus serta menjalankan penangkapan berdasarkan nama perangkat. Lihat [Otomatisasi Proyek](reference/sdk-reference.md#project-automation--chlorosproject) dalam referensi.

## Apa yang didapatkan dari instalasi hanya melalui pip

Periksa bendera ketersediaan tingkat modul sebelum menggunakan permukaan perangkat keras:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Pada host yang **hanya** memiliki `pip install chloros-sdk` dan tidak memiliki paket desktop Chloros:

* `ChlorosLocal`, `process_folder`, dan `process_lattice_capture` **tidak** berfungsi — modul-modul tersebut memerlukan biner backend yang disertakan dalam penginstal desktop.
* Pembantu smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) adalah klien murni HTTP, sehingga dapat beroperasi dengan backend di mesin lain — namun backend yang disertakan hanya terikat pada loopback, sehingga Anda harus meneruskan port tersebut sendiri (misalnya `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) dan meneruskan `backend_url="http://127.0.0.1:5000"` bersama `auto_start_backend=False`. Lihat [Mode Backend Jarak Jauh](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Kelas LATTICE yang terhubung langsung ke perangkat keras (`LatticeCamera`, `CameraPool`, …) dapat diimpor, tetapi memerlukan runtime Arena SDK dari paket desktop — tanpa itu, `CAMERA_AVAILABLE` akan menjadi `False`.
* `daq_sdk` (kelas DAQ langsung) disertakan dalam instalasi desktop, bukan dalam paket PyPI, sehingga `DAQ_AVAILABLE` menjadi `False` pada host yang hanya menggunakan pip — kendalikan sensor DAQ melalui `connect_daq_sensor()` terhadap backend (yang ditunnel) sebagai gantinya.

## Persyaratan lisensi

Akses ke SDK memerlukan login Chloros+ yang aktif pada tingkatan berbayar apa pun — **Copper atau lebih tinggi**(Copper / Bronze / Silver / Gold); tingkatan gratis Iron tidak memiliki akses ke SDK/CLI. Penegakan dilakukan**di sisi server**: setiap permintaan SDK harus disertai sesi aktif dan paket berbayar, atau backend akan mengembalikan `403` / `PLAN_UPGRADE_REQUIRED` (dinaikkan menjadi `ChlorosLicenseError` oleh `ChlorosLocal`, dan menjadi `ChlorosConnectError` oleh helper `connect_*`). Pemanggil yang telah keluar dari sistem akan mendapatkan `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — menjalankan ulang `chloros-cli login` memperbaiki kasus pertama tetapi tidak yang kedua.

Penggunaan offline berfungsi dalam masa tenggang paket: tingkatan dibaca dari cache validasi server (5 menit) atau cache lisensi yang ditandatangani dan terikat pada perangkat (30 hari untuk paket bulanan; hingga masa berlangganan berakhir untuk paket tahunan). Saat masa tenggang berakhir, paket akan berubah menjadi gratis dan akses SDK akan terhenti hingga perangkat terhubung ke server sekali. `chloros-cli status` tetap dapat diakses pada tingkatan gratis sehingga alasannya selalu terlihat. Lihat [Chloros+ Masuk](chloros+-login.md).

## Pengecualian

Tangkap kelas dasar untuk menangani &quot;segala hal yang salah dengan Chloros&quot;:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Semua pengecualian pipa (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) berasal dari `ChlorosError`. Satu hal yang perlu diperhatikan: `ChlorosConnectError` — hanya ditimbulkan oleh `connect_camera` / `connect_array` / `connect_daq_sensor` — berasal dari `Exception` biasa, **bukan** dari `ChlorosError`, sehingga `except ChlorosError` tidak akan mendeteksinya. Hierarki lengkapnya terdapat di [Referensi SDK](reference/sdk-reference.md#exceptions).

## Lihat juga

* [Referensi SDK](reference/sdk-reference.md) — permukaan API yang lengkap, dioptimalkan untuk asisten AI.
* [Referensi CLI](reference/cli-reference.md) — setiap subperintah CLI mencerminkan panggilan SDK.
* [Unduh](download.md) — penginstal untuk Windows dan Linux.

# API: Python SDK

{% hint style="info" %}
**Mencari API lengkap?** Halaman ini adalah panduan praktis. Setiap kelas publik, metode, tanda tangan yang tepat, dan contoh yang dapat disalin-tempel terdapat di [Referensi SDK](reference/sdk-reference.md), yang dioptimalkan untuk asisten AI.**Sedang bekerja dengan asisten AI?** Tempelkan URL ini ke dalam obrolan agar Anda memiliki Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Chloros API Setiap halaman panduan ini tersedia dalam format markdown mentah di slug huruf kecilnya + `.md`, dan seluruh panduan diindeks di `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` di PyPI) mengendalikan semua fungsi yang dapat dilakukan aplikasi desktop dari Python: pemrosesan gambar batch, kontrol kamera dan array LATTICE secara langsung, sesi pengumpulan data (DAQ) sensor cahaya, serta otomatisasi proyek yang disimpan. Ini merupakan lapisan tipis di atas backend lokal yang sama yang digunakan oleh GUI dan CLI (HTTP di `127.0.0.1:5000`), sehingga perilakunya identik di ketiga antarmuka tersebut.

## Instalasi

SDK ini disertakan dalam paket desktop Chloros, yang juga menyediakan backend pemrosesan dan runtime perangkat keras, serta selalu memiliki versi yang sesuai.

**Langkah 1 — Instal Chloros.** Windows: jalankan penginstal desktop (jalur default `C:\Program Files\Chloros\`) dari halaman [Unduh](download.md). Linux: instal paket `.deb` dengan `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Instalasi Linux](linux/linux-installation.md)).**Langkah 2 — Periksa SDK** (Python 3.7+). Biasanya tidak perlu menginstal apa pun:

* **Windows:** penginstal akan memasang roda SDK yang disertakan ke dalam Python sistem Anda (jika Python belum terpasang, langkah ini akan dilewati).
* **Linux:** `.deb` menginstal SDK untuk sistem `python3` ke dalam `/usr/lib/python3/dist-packages`. Persyaratannya, `python3-numpy` dan `python3-requests`, direkomendasikan oleh paket ini: `apt install` menyertakannya, sedangkan `dpkg -i` tidak.
* **Virtualenv** tidak dapat mendeteksi paket sistem; instal wheel yang disertakan ke dalamnya: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` di Linux, atau `chloros_sdk-*.whl` di dalam folder `sdk` pada folder instalasi Windows (secara default adalah `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Pada mesin yang tidak memiliki paket Chloros, di mana SDK-nya menggerakkan backend di mesin lain (lihat [Apa yang diperoleh dari instalasi hanya dengan pip](#what-a-pip-only-install-gets)), instal dari PyPI: `pip install chloros-sdk`. PyPI diperbarui pada setiap rilis, sehingga selalu sesuai dengan rilis stabil terbaru.

**Langkah 3 — Masuk sekali per mesin:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kredensial disimpan dalam cache di `~/.chloros/` (kedua platform). Di Windows, Anda juga dapat masuk melalui tab Pengguna <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> pada aplikasi desktop. SDK ini memerlukan paket Chloros+ berbayar — lihat [Persyaratan lisensi](#license-requirement) di bawah ini.

| Persyaratan | Detail |
| --- | --- |
| **Chloros terinstal** | Windows: penginstal desktop; Linux: paket `.deb` (menyediakan biner backend dan SDK) |
| **Python** | 3.7 atau lebih tinggi (dikembangkan/diuji pada 3.10) |
| **Sistem operasi** | Windows 10/11 64-bit, Ubuntu 22.04 LTS atau yang lebih baru, atau NVIDIA Jetson (JetPack 6) |
| **Lisensi** | Akun Chloros+ yang aktif, paket berbayar apa pun (Copper atau lebih tinggi) |

## Selesai dalam 60 detik

Satu perintah cukup untuk membuat proyek, mengimpor folder, mengonfigurasi pemrosesan, dan menjalankan pipeline — serta memulai backend secara otomatis jika belum berjalan:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Di Linux, gunakan jalur Linux: `/home/user/drone_images/flight001`. SDK-nya berfungsi sama di kedua platform.)

Memproses folder tangkapan LATTICE? Gunakan wrapper yang kompatibel dengan LATTICE — ini menerapkan pengaturan default yang tepat (tanpa deteksi target panel, debayer standar):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — kontrol pipeline penuh

Untuk apa pun yang lebih dari satu baris perintah, gunakan `ChlorosLocal`. Perintah ini menggunakan backend yang sudah berjalan (dari aplikasi desktop), atau memulai backend yang terinstal saat pertama kali digunakan (`auto_start_backend=True`; di Windows dari folder `backend` dalam folder instalasi Chloros, secara default menggunakan `C:\Program Files\Chloros\backend\`), membuat dan mengonfigurasi proyek, memantau kemajuan, serta menampilkan ringkasan setelah proses selesai.

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
Pertahankan `http://127.0.0.1:5000` default daripada menggantinya dengan `localhost` — pada Windows, `localhost` akan diterjemahkan menjadi `::1` terlebih dahulu dan memakan waktu sekitar 2 detik per permintaan terhadap backend yang hanya mendukung IPv4.
{% endhint %}

Gunakan sebagai pengelola konteks. Keluar dari blok ini akan menghentikan penggunaan backend oleh instance ini, bukan backend itu sendiri: backend tetap berjalan untuk skrip berikutnya dan akan berhenti sendiri setelah 30 menit tidak digunakan. `shutdown_backend()` menghentikan instance ini setelah dimulai.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` menerima kata kunci berikut: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation`, dan `custom_settings`. Nilai-nilai utama:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
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

Setelah selesai, `process()` melampirkan ringkasan pemrosesan backend sebagai `result["summary"]`. Setiap entri dalam `summary["hints"]` berupa kalimat lengkap yang menjelaskan hal-hal penting — misalnya, mengapa suatu eksekusi tidak menghasilkan output apa pun — dan setiap petunjuk juga diterbitkan ulang sebagai Python `UserWarning`, sehingga eksekusi kosong dapat mendiagnosis dirinya sendiri meskipun Anda tidak pernah memeriksa kamus tersebut:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` tidak memicu pengecualian ketika suatu eksekusi tidak menghasilkan gambar apa pun.** Inilah satu-satunya perbedaan yang disengaja antara SDK dan CLI: `chloros-cli process` menganggap "produk diminta, tetapi tidak ada yang ditulis" sebagai kegagalan dan keluar dengan nilai non-nol, sedangkan SDK kembali secara normal dan melaporkan kondisi tersebut melalui `summary` / petunjuk. Jika pipeline Anda harus berhenti pada eksekusi yang kosong, periksa sendiri — periksa `summary` (atau hitung file di bawah folder proyek) daripada mengandalkan pengecualian.
{% endhint %}

## Smart Connect — perangkat keras langsung

Tiga helper membuka sesi persisten di kolam perangkat keras backend — kolam yang sama yang digunakan oleh CLI, skrip lain, dan tab kamera di GUI, sehingga skrip SDK dapat berjalan bersamaan tanpa berebut port serial atau bandwidth jaringan. Ketiga helper tersebut secara otomatis memulai backend lokal jika tidak ada yang sedang berjalan. Panel DAQ pada GUI bukan bagian dari kumpulan tersebut: tutup sensor cahaya di GUI sebelum membukanya dari skrip.

### Kamera LATTICE tunggal — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Jalur relatif mengacu pada lokasi di mana skrip Anda dijalankan (berubah pada versi 1.3.0).** Dengan backend lokal, jalur relatif `output_dir` (termasuk `"output"` default) — dan jalur lain yang dibuka oleh backend (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — disesuaikan dengan direktori kerja skrip Anda, sehingga `cam.capture("output/")` menulis ke `<current folder>/output/`. Pada versi 1.2.0, backend yang terinstal menerjemahkannya ke dalam folder instalasinya sendiri dan mengalami kegagalan dengan pesan "Akses ditolak". Terhadap backend jarak jauh, jalur dikirimkan sesuai yang diketik.
{% endhint %}

### Array tersinkronisasi — `connect_array`

`connect_array` adalah titik masuk yang direkomendasikan untuk rig multi-kamera. Modul ini menjalankan alur persiapan cerdas yang sama seperti antarmuka pengguna grafis (GUI): analisis jaringan, pemilihan otomatis tingkat sinkronisasi, sinkronisasi waktu PTP, pemilihan format piksel per kamera, penyemaian AE, dan pengaktifan pemicu GPIO. **Serial pertama adalah master** (yang memicu pulsa pemicu perangkat keras); sisanya adalah slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Tambahkan `smart=True` ke penangkapan array apa pun untuk menunggu hingga eksposur otomatis stabil di semua kamera sebelum memicu pemicu.

Fitur baru di versi 1.3.0, `ArraySession` juga memberikan bingkai yang disinkronkan secara langsung, tanpa menulis file:

* `arr.grab()` mengembalikan satu bingkai per kamera, semuanya dari satu pemicu, sebagai array numpy (`{serial: array}`); `.group_id` adalah pemicu bersama dan `.errors` menamai kamera mana pun yang tidak termasuk.
* `arr.frame_stream(fps=5.0)` menghasilkan satu set seperti itu per tick, untuk pemrosesan langsung dalam pipeline Anda sendiri.
* `arr.continuous("output/")` menjalankan perekaman berkelanjutan bawaan backend (mode Continuous pada GUI) dan mengembalikan `ContinuousHandle` bersama dengan `status()`, `wait()`, dan `stop()` — gunakan ini sebagai pengganti loop perekaman saat Anda perlu menyimpan setiap pemicu dengan laju tinggi.

Untuk mode penangkapan (Tunggal / Berkelanjutan / Interval / Tercepat), perekam, burst-to-video, dan penyelarasan array, lihat [Referensi SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor cahaya DAQ — `connect_daq_sensor`

Tanpa argumen, `connect_daq_sensor()` mendeteksi transportasi secara cerdas (urutan prioritas: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Setiap frame membawa nilai 135-titik `spectrum` (W/m²/nm setelah dikalibrasi), bendera `is_saturated`, dan CIE `x`, `y`, serta `z`. Untuk menentukan sensor atau protokol transmisi tertentu, berikan satu petunjuk eksplisit:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Perhatikan bahwa profil koreksi cap (`cap_id`) **bukan** pengaturan di SDK — pilih profil tersebut melalui `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Proyek yang disimpan — `open_project`

Proyek Chloros yang disimpan akan mempertahankan perangkat keras yang terhubung (`cameras.json` + `sensors.json` bersama dengan `project.json`), dan `chloros_sdk.open_project(path)` dapat menghubungkan kembali kamera, array, dan sensor cahayanya sekaligus serta mengontrol pengambilan gambar berdasarkan nama perangkat. Sensor cahaya terhubung kembali melalui backend Chloros yang sedang berjalan, dengan pengaturan dan batas (cap) yang tersimpan. Lihat [Otomatisasi Proyek](reference/sdk-reference.md#project-automation--chlorosproject) dalam referensi.

## Apa yang diperoleh dari instalasi pip-only

Periksa bendera ketersediaan tingkat modul sebelum menggunakan permukaan perangkat keras:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Pada host yang **hanya** memiliki `pip install chloros-sdk` dan tidak memiliki paket desktop Chloros:

* `ChlorosLocal`, `process_folder`, dan `process_lattice_capture` **tidak** berfungsi — modul-modul tersebut memerlukan biner backend yang disertakan dalam penginstal desktop.
* Pembantu smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) adalah klien HTTP murni, sehingga dapat berfungsi dengan backend di mesin lain — namun backend yang disertakan hanya terikat pada loopback, sehingga Anda harus melakukan port forwarding sendiri (misalnya `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) dan menjalankan `backend_url="http://127.0.0.1:5000"` bersama dengan `auto_start_backend=False`. Lihat [Mode Backend Jarak Jauh](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Kelas LATTICE yang terhubung langsung ke perangkat keras (`LatticeCamera`, `CameraPool`, …) dapat diimpor, tetapi memerlukan runtime Arena SDK dari paket desktop — tanpa itu, `CAMERA_AVAILABLE` akan menjadi `False`.
* Sensor DAQ hanya digerakkan melalui backend (yang ditunnel) — `connect_daq_sensor()`, atau `proj.sensors` dari suatu proyek; backend tersebut membuka sensor pada mesinnya sendiri.

## Asisten AI — `chloros-mcp`

SDK ini mencakup `chloros-mcp`, sebuah server MCP yang memungkinkan Claude Desktop, Claude Code, Cursor, dan klien MCP lainnya mengendalikan Chloros: memproses citra, serta menjalankan kamera LATTICE, array, dan sensor cahaya DAQ. Alat-alatnya mencakup fungsi yang dilakukan oleh SDK ini. Alat ini terinstal bersama SDK, sehingga pengaturan untuk Claude Code hanya memerlukan satu baris perintah:

```bash
claude mcp add chloros -- chloros-mcp
```

Klien lainnya: [Menggunakan Chloros dengan Asisten AI](ai-assistants.md). Opsi dan semua alat: [Server MCP (Asisten AI)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Persyaratan Lisensi

Akses ke SDK memerlukan login Chloros+ yang aktif pada tingkatan berbayar apa pun — **Copper atau lebih tinggi**(Copper / Bronze / Silver / Gold); tingkatan gratis Iron tidak memiliki akses SDK/CLI. Penegakan dilakukan**di sisi server**: setiap permintaan SDK harus disertai sesi aktif dan paket berbayar, atau backend akan mengembalikan `403` / `PLAN_UPGRADE_REQUIRED` (dinaikkan menjadi `ChlorosLicenseError` oleh `ChlorosLocal`, dan menjadi `ChlorosConnectError` oleh helper `connect_*`). Pemanggil yang telah keluar dari sistem mendapatkan `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — menjalankan ulang `chloros-cli login` memperbaiki kasus pertama tetapi tidak yang kedua.

Penggunaan offline berfungsi selama masa tenggang paket: tingkatan dibaca dari cache validasi server (5 menit) atau cache lisensi yang ditandatangani dan terikat pada perangkat (30 hari untuk paket bulanan; hingga masa berlangganan berakhir untuk paket tahunan). Saat masa tenggang berakhir, paket akan beralih ke versi gratis dan akses SDK akan terhenti hingga perangkat terhubung ke server sekali. `chloros-cli status` tetap dapat diakses pada tingkatan gratis sehingga alasannya selalu terlihat. Lihat [Chloros+ Login](chloros+-login.md).

## Pengecualian

Tangani kelas dasar untuk menangani "segala kesalahan Chloros":

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

Semua pengecualian pipa (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) diturunkan dari `ChlorosError`. Satu hal yang perlu diperhatikan: `ChlorosConnectError` — hanya dipicu oleh `connect_camera` / `connect_array` / `connect_daq_sensor` — berasal dari `Exception` biasa, **bukan** dari `ChlorosError`, sehingga `except ChlorosError` tidak akan mendeteksinya. Hierarki lengkapnya terdapat di [Referensi SDK](reference/sdk-reference.md#exceptions).

## Lihat juga

* [Referensi SDK](reference/sdk-reference.md) — permukaan API lengkap, yang dioptimalkan untuk asisten AI.
* [Referensi CLI](reference/cli-reference.md) — setiap subperintah CLI mencerminkan panggilan SDK.
* [Menggunakan Chloros dengan Asisten AI](ai-assistants.md) — panduan pengaturan dan resep prompt `chloros-mcp`.
* [Unduh](download.md) — penginstal untuk Windows dan Linux.

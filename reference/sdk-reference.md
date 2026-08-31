# Chloros Python SDK Referensi

**Versi:**

1.2.0**Dibuat:**29 Juli 2026 19:19 ·**Direvisi:** 30 Agustus 2026**Paket:** `chloros-sdk` (PyPI)**Target:** Dioptimalkan untuk penggunaan LLM; mudah dibaca manusia.**Cakupan:** Setiap kelas publik, fungsi, dan helper yang diekspos oleh `import chloros_sdk`, dengan contoh yang dapat disalin dan ditempelkan yang mencakup pemrosesan gambar, kontrol kamera tunggal, array tersinkronisasi, sensor DAQ, dan otomatisasi proyek.

Jika Anda hanya membutuhkan poin-poin penting, langsung ke:
- [Instalasi &amp; Panduan Cepat](#installation)
- [Smart-Connect untuk Array LATTICE](#smart-connect-for-lattice-cameras)
- [Sesi Sensor DAQ](#daq-sensor-sessions)
- [Otomatisasi Proyek](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arsitektur dalam 60 Detik

SDK adalah lapisan tipis Python di atas backend Chloros (server Flask yang sama yang digunakan oleh antarmuka pengguna desktop dan CLI). Untuk otomatisasi, Anda mengimpor `chloros_sdk` dan memanggil metode tingkat tinggi; di balik layar, setiap panggilan menjadi permintaan HTTP ke backend lokal di port 5000 — `http://127.0.0.1:5000/api/...` (secara sengaja bukan `localhost`, yang akan diarahkan ke `::1` terlebih dahulu di Windows dan memakan waktu ~2 detik per permintaan terhadap backend yang hanya mendukung IPv4). Backend mengelola kumpulan perangkat keras — kamera, sensor DAQ, profil penyelarasan, buffer bingkai — sehingga skrip SDK dapat berjalan bersamaan dengan GUI tanpa berebut port serial atau bandwidth NIC.

Ada tiga antarmuka yang akan Anda gunakan:

1. **`ChlorosLocal` + fungsi bebas** (`process_folder`, `process_lattice_capture`) — Jalur pemrosesan gambar. Jalankan seluruh folder melalui kalibrasi / debayer / ekspor indeks dari satu panggilan `Python`.
2. **Penanganan koneksi cerdas** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Buka sesi backend persisten untuk perangkat keras langsung. Alur &quot;smart-prep&quot; yang sama seperti pada GUI: pemeriksaan jaringan, pemilihan tier otomatis, PTP, penyemaian AE, konfigurasi pemicu GPIO.
3. **`ChlorosProject` / `open_project`** — Memuat proyek yang telah disimpan (folder berisi `cameras.json` + `sensors.json` + `project.json`), menghubungkan semuanya sekaligus, dan menjalankan penangkapan dengan handle yang diberi nama.

Antarmuka 1 dan 2 **secara otomatis memulai backend lokal** jika belum ada yang dalam keadaan mendengarkan (biner yang disertakan yang sama yang dijalankan oleh GUI/ CLI) — sehingga skrip sederhana dapat berjalan dari shell baru tanpa Anda perlu menjalankan backend terlebih dahulu. Berikan argumen `auto_start_backend=False` untuk menonaktifkan fitur ini (misalnya, saat mengarah ke backend jarak jauh, yang tidak pernah dijalankan). Lihat [Backend Auto-Start](#backend-auto-start). Surface 3 berperilaku berbeda: `open_project()` tidak menerima parameter `auto_start_backend`, dan `connect_all()` tidak pernah menjalankan backend — ia melakukan pemeriksaan terhadap `http://127.0.0.1:5000` sekali dan, jika tidak ada respons, secara diam-diam beralih ke kontrol perangkat langsung (tanpa backend) `lattice_sdk`. Hanya `proj.process()` dan `stream(..., overlays=True)` yang secara malas membuat `ChlorosLocal()` (yang melakukan auto-start).

Ketiganya memerlukan otentikasi: jalankan `chloros-cli login` sekali di mesin tersebut, atau masuk melalui antarmuka desktop. Panggilan SDK tanpa sesi yang valid akan memicu `ChlorosAuthenticationError`.

Persyaratan:
- Python 3.7+ (sesuai yang dinyatakan oleh paket; dikembangkan/diuji pada versi 3.10)
- Chloros Desktop terinstal secara lokal (biner backend disertakan dalam penginstal)
- Akun Chloros+ yang aktif. Tingkat akses SDK / CLI adalah **Copper**atau lebih tinggi (Copper / Bronze / Silver / Gold); tingkat gratis**Iron**tidak memiliki akses ke SDK / CLI. Hal ini diberlakukan**di sisi server**sisi server**: setiap permintaan yang diberi bendera SDK / CLI harus disertai dengan sesi aktif dan paket berbayar, atau backend akan mengembalikan `403` dengan `error_code: PLAN_UPGRADE_REQUIRED` (ditampilkan sebagai `ChlorosLicenseError` oleh `ChlorosLocal`, dan sebagai `ChlorosConnectError` oleh helper `connect_*`). Pemanggil yang sudah keluar dari sistem akan menerima `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — kedua kode tersebut berbeda karena menjalankan ulang `chloros-cli login` akan memperbaiki yang pertama tetapi tidak dapat memperbaiki yang kedua.
- Penggunaan offline didukung selama masa tenggang paket: tingkatan langganan dibaca dari cache validasi server (5 menit) atau cache lisensi yang ditandatangani dan terikat pada perangkat (30 hari untuk paket bulanan, hingga masa berlakunya langganan berakhir untuk paket tahunan). Ketika masa tenggang tersebut berakhir, paket akan beralih ke versi gratis dan akses ke SDK / CLI akan terhenti hingga perangkat dapat terhubung ke server sekali. `chloros-cli status` (`GET /api/license-status`) tetap dapat diakses pada tingkatan gratis sehingga alasannya terlihat — ini adalah satu-satunya rute SDK / CLI yang dikecualikan dari pembatasan tingkatan.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS atau yang lebih baru**, atau Jetson (JetPack 6). Ubuntu 20.04**tidak** didukung: ketergantungan `.deb` berasal dari apa yang dihubungkan oleh backend, termasuk `libc6 (>= 2.34)`, dan Focal menyediakan glibc 2.31.

---

## Instalasi

Python SDK adalah lapisan tipis Python di atas backend Chloros. Untuk semua hal di luar beberapa alur kerja yang hanya melibatkan DAQ, Anda perlu **menginstal paket desktop Chloros secara lokal** (penginstal Windows atau Linux `.deb`) — itulah yang menyediakan biner backend, runtime Arena SDK untuk kamera LATTICE, dan paket kalibrasi.

Unduhan terbaru: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Langkah 1 — Instal paket platform Chloros

#### Windows (.exe)

1. Unduh `Chloros-Setup-x.y.z.exe` dari halaman unduhan.
2. Jalankan penginstal dan ikuti panduan wizard. Jalur instalasi default adalah `C:\Program Files\MAPIR\Chloros\`.
3. Jalankan Chloros setidaknya sekali dan masuk menggunakan akun Chloros+ Anda.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Langkah 2 — Instal Python SDK

**Penginstal Chloros menyertakan wheel SDK yang sesuai.** Setiap penginstal Windows dan berkas .deb Linux akan menempatkan berkas `chloros_sdk-X.Y.Z-py3-none-any.whl` di disk yang persis sesuai dengan versi GUI / CLI / backend. Anda tidak perlu terus-menerus memeriksa PyPI untuk tetap sinkron.

#### Windows

Penginstal secara otomatis menjalankan `pip install` terhadap wheel yang disertakan menggunakan Python sistem Anda (pembuka `py.exe` lebih disukai, jika tidak tersedia akan menggunakan `python -m pip`). Tidak diperlukan tindakan apa pun — `import chloros_sdk` berfungsi di lingkungan Python Anda setelah instalasi berhasil. Jika tidak ada Python di sistem, penginstal akan melewati langkah ini tanpa pemberitahuan, dan antarmuka grafis (GUI) serta CLI tetap berfungsi.

#### Linux (.deb)

Berkas .deb menempatkan berkas wheel di `/usr/lib/chloros/sdk/`. `postinst` mencetak perintah yang tepat — distribusi PEP 668 menolak penulisan global melalui pip secara default, sehingga kami tidak melakukan pemasangan otomatis:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Untuk penerapan Jetson yang terisolasi (air-gapped), proses ini sepenuhnya offline — berkas wheel sudah ada di disk.

#### PyPI Publik

Untuk host yang hanya menggunakan pip (tidak ada paket desktop Chloros yang terinstal; alur kerja remote-backend atau hanya DAQ):

```bash
pip install chloros-sdk
```

PyPI diperbarui pada build penginstal versi rilis, sehingga wheel yang dipublikasikan sesuai dengan rilis stabil terbaru. Build pengembangan (misalnya `1.1.4.dev1`) hanya didistribusikan melalui wheel penginstal yang disertakan.

#### Verifikasi

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Diperlukan langganan Chloros+.** Semua panggilan ke SDK memerlukan login Chloros+ yang aktif. Jalankan `chloros-cli login user@example.com 'YourPassword'` sekali per mesin; kredensial disimpan dalam cache di `~/.chloros/`.

### Apakah Saya Membutuhkan Paket Desktop?

Paket pip saja **tidak** cukup untuk sebagian besar alur kerja. Berikut adalah kebutuhan masing-masing permukaan SDK:

| Permukaan SDK | Membutuhkan Paket Desktop? | Mengapa |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ya** | Secara otomatis menjalankan biner backend di `/usr/lib/chloros/chloros-backend` (Linux) atau `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ya**(lokal)**/ Tidak**(jarak jauh) | Klien murni HTTP melalui backend. Backend lokal → memerlukan paket desktop. Backend jarak jauh → `backend_url=`**melalui terowongan** (lihat Mode Backend Jarak Jauh — backend bawaan hanya mengikat loopback). |
| `ChlorosProject` / `open_project` | **Ya** | Mengakses proyek yang disimpan melalui backend. |
| Kelas LATTICE langsung (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ya** | Membutuhkan runtime asli Arena SDK yang disertakan dalam paket desktop. Jika tidak, `CAMERA_AVAILABLE` sama dengan `False` saat diimpor. |
| Kelas DAQ langsung (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Tidak** | Penggunaan pyserial/bleak/zeroconf murni (Python). Lingkungan yang hanya menggunakan pip dapat mengendalikan DAQ secara end-to-end. |

### Mode Remote-Backend (host hanya menggunakan pip, melalui terowongan)

> **Backend yang disertakan tidak dapat dijangkau melalui LAN.** Build produksi
> hanya mengikat loopback (kedua keluarga loopback) dan secara tegas menolak
> satu-satunya mode non-loopback (`CHLOROS_CLOUD_MODE`), sehingga
> `backend_url="http://<lan-ip>:5000"` **tidak dapat beroperasi terhadap
> Chloros yang terinstal** — pola tersebut hanya pernah berfungsi dengan backend source/dev
> . Untuk mengendalikan backend di mesin lain, alihkan port loopback-nya
> sendiri dan arahkan SDK ke terowongan:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Host headless / CI / robotics dapat mempertahankan satu mesin dengan instalasi desktop lengkap sebagai &quot;server Chloros&quot; dan `pip install chloros-sdk` di tempat lain — tetapi transportasi di antara keduanya adalah terowongan yang diatur pengguna di atas, bukan URL LAN langsung.

> **Keterbatasan yang diketahui — `ChlorosLocal` tidak hanya mendukung pip-saja.** `ChlorosLocal(backend_url=BACKEND)` saat ini menyelesaikan biner backend lokal di konstruktornya *sebelum* memeriksa URL, dan memunculkan `ChlorosBackendError` (“backend Chloros tidak ditemukan…”) ketika tidak ada paket desktop yang terinstal — bahkan dengan backend jarak jauh yang dapat dijangkau. Hanya antarmuka smart-connect di atas (`connect_camera` / `connect_array` / `connect_daq_sensor`, ditambah `analyze_array_network` dan helper `list_*` / `discover_*`) yang berfungsi dari host yang hanya menggunakan pip.

### Alur Kerja Khusus DAQ (host yang hanya menggunakan pip)

Jika Anda hanya memerlukan sensor DAQ dan tidak menggunakan kamera LATTICE atau pemrosesan gambar, paket pip sudah lengkap:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Tanpa backend, tanpa .deb, dan tidak perlu login ke Chloros+ untuk pekerjaan DAQ langsung ke perangkat keras.

---

## Panduan Cepat

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Indeks Tingkat Atas API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Pemrosesan Gambar — `ChlorosLocal`

Kelas pipa utama. Menjalankan backend saat pertama kali digunakan, membuat/mengonfigurasi proyek, memantau kemajuan, dan mengembalikan ringkasan setelah proses selesai.

### Konstruktor

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

### Metode

| Metode | Deskripsi |
| --- | --- |
| `create_project(project_name, camera=None)` | Membuat proyek baru (secara opsional dengan templat kamera seperti `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Mengimpor gambar RAW/TIF/JPG/DNG **dan rekaman sensor cahaya `.daq`**. Mengembalikan `count` (gambar) dan `scan_count` (rekaman). Memberikan peringatan hanya jika folder tersebut tidak berisi keduanya. |
| `export_light_sensor(daq=True, csv=True)` | Menulis data yang telah dikalibrasi `.daq` + `.csv` untuk setiap rekaman sensor cahaya pada proyek, ke dalam `<project>/Light Sensor/`. Lihat [Rekaman Sensor Cahaya](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Atur parameter pemrosesan. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Jalankan alur pemrosesan. Mengembalikan `{"status": "complete", "async": False}`, ditambah kunci `summary` saat backend menyediakannya — lihat [Ringkasan Pasca-Eksekusi &amp; Petunjuk](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Periksa status backend. |
| `logout()` | Hapus kredensial yang tersimpan dalam cache. |
| `shutdown_backend()` | Hentikan backend (jika dimulai dengan perintah `SDK` -started). |
| `discover_cameras()` | Temukan kamera LATTICE **melalui backend instance ini** (`/api/camera/discover`). Mengembalikan daftar kamus (`serial`, `model`, `ip`, …) — bentuknya sama seperti yang terlihat di GUI/CLI. Daftar kosong jika tidak ditemukan atau backend tidak dapat dijangkau. |
| `camera_capture(output_dir, format="tiff", **settings)` | Merekam satu frame**melalui backend**(dimulai secara otomatis oleh handle ini) sehingga mendapatkan persiapan yang sama seperti yang ditampilkan di GUI/ CLI (default 12-bit, penggunaan ulang pool, metadata kalibrasi tertanam). Tentukan target dengan `serial=` atau `device_index=`; lewati `exposure`/`gain`/`pixel_format`/`preset` sebagai `**settings`. Mengembalikan kamus metadata warisan (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Menghasilkan bingkai pratinjau yang digabungkan dengan overlay dari kamera yang digabungkan — klien MJPEG ringan di atas rute backend`/api/camera/<serial>/stream-annotated` (zebra / grid / crosshair / histogram / peaking / spot yang digambar di sisi server). `decode=True` menghasilkan array BGR; `False` menghasilkan byte mentah JPEG. Juga dapat diakses per-proyek sebagai `ChlorosProject.stream(overlays=True)`. |

Gunakan sebagai pengelola konteks untuk pembersihan yang terjamin:

```python
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

### Perekaman Sensor Cahaya — dikalibrasi `.daq` + `.csv`

Sebuah DAQ-U / DAQ-M / DAQ-E dapat direkam **tanpa** paket kalibrasinya. Itulah
yang dilakukan perekam publik [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) secara default: mereka menulis data hitungan sensor mentah dan menandai
file tersebut sehingga Chloros mengambil kalibrasi pabrik sensor tersebut **berdasarkan nomor seri** — cache lokal
terlebih dahulu, kemudian MAPIR Cloud — dan menerapkannya saat impor.

Chloros menulis hasilnya kembali sebagai dua produk per rekaman, di bawah
`<project>/Light Sensor/`:

| Produk | Apa itu |
| --- | --- |
| `<name>_calibrated.daq` | Arsip yang dapat diproses ulang — skema yang sama dengan perekaman langsung, kini menyatakan bundel yang menghasilkannya. Mengimpornya kembali **tidak** akan mengkalibrasinya untuk kedua kalinya. |
| `<name>_calibrated.csv` | Irradiance spektral dalam W/m²/nm pada grid panjang gelombang sensor itu sendiri, satu baris per pembacaan, ditambah kolom fotometrik (daya total, lux fotopik/skotopik, PPFD, dan/hijau/merah, panjang gelombang puncak). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Hanya untuk sensor tanpa bundel (DAQ-A).** Hitungan spektral mentah sensor — *bukan* irradiansi. Lihat di bawah. |

`process()` melakukan ekspor ini sebagai salah satu tahapnya. Proses ini **tidak** memerlukan citra:
sensor cahaya yang diterbangkan sendiri merupakan alur kerja utama, dan proyek semacam itu secara
konstruksi tidak memiliki citra sama sekali.

**Rekaman DAQ-A diekspor sebagai hitungan mentah.** Keluarga DAQ-A sudah ada sebelum sistem
bundel per-serial dan tidak memiliki bundel yang perlu diambil — melainkan dikalibrasi di lapangan menggunakan
target reflektansi, itulah sebabnya ia tidak pernah membutuhkannya. Rekaman-rekaman tersebut diekspor
di bawah akar `_raw`, bukan `_calibrated`: nama file yang berbeda, bukan bendera
di dalam berkas, karena klaim tersebut harus tetap utuh saat dikirim melalui email sebagai nama berkas mentah. Header
`.csv` menyatakan `raw spectral sensor counts (NOT irradiance)` dan memperingatkan bahwa
nilai-nilai tersebut dapat dibandingkan **di dalam** berkas — persis seperti yang digunakan oleh kalibrasi berbasis target
— dan bukan antar sensor. Kolom fotometrik yang bergantung pada daya (daya total,
lux fotopik/skotopik, PPFD) menghasilkan nilai **NULL** alih-alih diintegrasikan dari hitungan.

DAQ-U / DAQ-M / DAQ-E yang bundelnya tidak dapat diambil tetap **dilewati**,
bukan ditulis mentah: di situ bundel tersebut ada dan &quot;sambungkan kembali dan proses ulang&quot; adalah saran yang tepat.

Rekaman **v1.01 / v1.02** lama (ditulis oleh DAQ-A-SD) tidak menyertakan epoch per pembacaan,
hanya waktu penulisan berkas. Pencocok gambar↔downwelling masih menolaknya — mencocokkan
bingkai dengan waktu penulisan akan menghasilkan kesalahan yang tidak terlihat — tetapi eksportir membacanya, dan
CSV mencetak `clock=daq_created_on` sehingga produk menyatakan jam mana yang digunakannya.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Rekaman yang bundel kalibrasinya tidak dapat diambil (offline, atau sensor tanpa
kalibrasi dalam file) dilaporkan di bawah `skipped` **bersama alasannya**. Rekaman tersebut tidak pernah
ditulis sebagai file &quot;telah dikalibrasi&quot; yang berisi hitungan mentah — sambungkan ke internet dan
jalankan kembali, maka ekspor akan selesai.

### Panggilan Kembali Kemajuan

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Ringkasan-Ringkasan &amp; Petunjuk

Setelah selesai, `process()` mengambil `GET /api/processing-summary` dan melampirkan isinya sebagai `result["summary"]`. Pengambilan ini dilakukan dengan upaya terbaik dan tidak pernah menghalangi pengembalian yang berhasil — jika ringkasan tidak tersedia, `process()` akan kembali ke bentuk dasar `{"status": "complete", "async": False}`. Setiap entri dalam `summary["hints"]` — kalimat lengkap dengan saran perbaikan, misalnya mengapa suatu eksekusi menghasilkan output nol — juga dikirim ulang sebagai Python `UserWarning`, sehingga eksekusi dengan output nol dapat mendiagnosis dirinya sendiri meskipun Anda tidak pernah memeriksa kamus:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` adalah bagian yang dapat dibaca mesin:

| Kunci | Apa yang dihitung |
| --- | --- |
| `models` | Kelompok kamera dalam proses tersebut. |
| `images_in_groups` | Gambar sumber di seluruh kelompok tersebut. |
| `targets_found` | Target reflektansi yang terdeteksi. |
| `images_calibrated` | Gambar yang dikalibrasi oleh proses tersebut. |
| `exported_files` | **File hasil gambar yang dihasilkan oleh proses tersebut.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Rekaman sensor cahaya, dihitung terpisah secara sengaja — data ini berasal dari tahap yang berbeda dan ada pada proses yang sama sekali tidak menghasilkan gambar, sehingga menggabungkannya akan membuat proses DAQ-saja akan terlihat seolah-olah telah mengekspor gambar. |

Di sampingnya: `summary["output_dirs"]` (setiap direktori yang ditulis),
`summary["light_sensor_export"]`, `summary["stopped"]` (berlaku jika pengguna menghentikan
proses, sehingga hitungan parsial tidak dianggap sebagai proses yang selesai namun hasilnya kurang), dan
`summary["groups"]` (rincian per-kelompok).

`exported_files` dicatat oleh pipa **saat menulis**, bukan dipindai dari
objek gambar proyek setelahnya. Strategi paralel dan GPU membangun objek gambarnya sendiri
(dalam subproses pekerja untuk jalur GPU), sehingga pemindaian lama melaporkan
`0 file(s) written` untuk setiap proses tersebut dan kemudian mengeluarkan petunjuk ekspor nol — pada eksekusi
di mana semuanya telah berjalan dengan baik. Jika Anda membuat skrip berdasarkan angka ini, eksekusi paralel yang berjalan dengan baik kini
akan melaporkan jumlah yang tidak nol.

Laporan &quot;Light-sensor skips&quot; mencantumkan alasan yang sebenarnya ditetapkan oleh pembaca untuk setiap berkas —
skema yang tidak dapat dibaca, bundel yang hilang, kesalahan penulisan — **yang telah dideduplikasi**, sehingga dua puluh file
yang dilewati karena satu penyebab akan dianggap sebagai satu penyebab, bukan dua puluh pengulangan dari penyebab tersebut.

> **`process()` tidak memicu kesalahan ketika suatu eksekusi tidak menghasilkan gambar apa pun.** Inilah satu-satunya hal di mana SDK dan
> CLI secara sengaja berbeda: `chloros-cli process` menganggap &quot;produk diminta, tetapi tidak ada yang
> ditulis&quot; sebagai kegagalan dan keluar dengan nilai bukan nol, sedangkan SDK kembali secara normal dan melaporkan
> kondisi tersebut melalui `summary` / hints. Jika pipa Anda harus berhenti pada proses kosong, periksalah
> sendiri — periksa `summary` (atau hitung file di bawah folder proyek) daripada mengandalkan
> tidak adanya pengecualian. Penyebab umumnya adalah folder input yang tidak dikenali sebagai
> penangkapan dan produk yang dilewati karena tidak berlaku untuk kamera yang ada (misalnya, radian dari kamera RGB-only
>).

### Fungsi Praktis

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Nilai yang Didukung

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Keluaran Radiometrik (pipeline multispektral LATTICE)

Tingkat ekspor multispektral LATTICE (M3C/M3M) dari pipa `process` — `reflectance` (default), `radiance`, `sensor-response`, atau `all` (setiap mode yang berlaku per gambar) — sesuai dengan pengaturan pemrosesan **&quot;Keluaran radiometrik&quot;** proyek. `configure()` memiliki kata kunci khusus untuknya:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Jalan keluar lanjutan — menulis kunci `"Radiometric output"` proyek melalui `custom_settings` — masih berfungsi, tetapi ingatlah bahwa ini akan menggantikan seluruh blok pengaturan (lihat peringatan di bawah):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (default) membagi radian kamera dengan **DAQ downwelling yang disesuaikan dengan timestamp**, yang diselesaikan secara otomatis dari `.daq` (DAQ-U/M/E)**atau produk `.csv`**asli DAQ-M yang ditemukan bersama citra; setiap paket kalibrasi per kamera atau DAQ yang tidak tersedia secara lokal akan**diunduh secara otomatis dari AWS** pada penggunaan pertama. Layanan *CLI* menampilkan ini sesuai dengantombol pengalih jenis produk pada `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **menggantikan** seluruh blok pengaturan yang dihitung (secara desain, ini melewati kata kunci lain dan validasi pada `configure()`). Saat menggunakannya, sertakan setiap kunci `Project Settings` yang Anda perlukan, seperti pada contoh di atas.

---

## Smart-Connect untuk Kamera LATTICE

Sesi backend yang persisten untuk perangkat keras langsung. Titik akhir yang sama dengan yang digunakan GUI, sehingga perilakunya identik di SDK / CLI / GUI.

### Kamera Tunggal — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Tanda Tangan `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Metode

| Metode | Deskripsi |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Membaca node GenICam; mengembalikan `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Menulis node berdasarkan nama ramah (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Mengambil **satu** bingkai. Mengembalikan daftar berelemen tunggal yang berisi kamus metadata bingkai. (Fungsi pengambilan bingkai burst/multi-frame telah dihapus — panggil `capture()` dalam loop jika Anda memerlukan serangkaian frame.) |
| `disconnect()` | Melepaskan dari pool. Tidak melakukan apa-apa jika kita terhubung ke sesi yang sudah terbuka. |

Kontrol ekspor `capture()` (model yang sama dengan array + GUI):

- `processing` / `levels` — `processing="all"` menyimpan setiap tipe ekspor yang berlaku; `levels=["raw","radiance"]` hanya menyimpan yang tersebut (menggantikan `processing`). Abaikan keduanya untuk menggunakan pengaturan default backend.
- `force_daq=True` — menyimpan pembacaan DAQ/DLS yang ditetapkan sebagai file tambahan `.daq` bahkan pada pengambilan data mentah saja, sehingga bingkai tersebut dapat diproses ulang menjadi reflektansi/indeks nanti. Tidak beroperasi jika tidak ada DAQ yang terhubung.

### Array Sinkron — `ArraySession` (Smart-Prep)

`connect_array` adalah **titik masuk yang direkomendasikan** untuk pengaturan multi-kamera. Fitur ini menjalankan alur kerja smart-prep GUI lengkap di balik layar:

1. **Analisis jaringan** (`/api/camera/array/recommend`) — menemukan ukuran frame terbesar yang sesuai dengan tingkatan sim-emit tanpa kehilangan frame.
2. **Pemilihan tingkatan otomatis** — `sim-capture-sim-emit` jika kabel mampu menanganinya; jika tidak, `sim-capture-ftd-stagger` atau `slip-emit-and-capture`.
3. **Pengecilan otomatis**— secara diam-diam memperkecil ukuran frame / meningkatkan penggabungan (binning) saat saluran tidak dapat mempertahankan resolusi yang diminta.**Jaring pengaman ini tidak mencakup kelebihan langganan agregat**: terlalu banyak kamera untuk kabel tidak dapat diatasi dengan memperkecil bingkai — lihat [Over-Subscription](#over-subscription-the-per-cam-floor).
4. **PTP diaktifkan**secara default — cap waktu antar kamera disinkronkan ke satu jam bersama dengan akurasi**~1 ms**. Eksposur simultan berasal dari pemicu perangkat keras M8 (**&lt; 100 µs** antar modul), bukan dari PTP: PTP menyelaraskan *cap waktu*, bukan eksposur.
5. **Pemilihan otomatis format piksel per kamera** — kamera RGB → `BayerRG8`, multispektral → `BayerRG12`.
6. **Penetapan awal AE** — mengambil snapshot status AE terkini setiap kamera agar koneksi tidak mereset eksposur di tengah proses.
7. **Konfigurasi pemicu GPIO** — `connect_array` mengaktifkan setiap kamera (`TriggerMode=On`, `TriggerSource=Line2`) sehingga pulsa dari master menggerakkan slave melalui kabel M8. Ini adalah langkah khusus untuk pengaturan array: jika hanya satu kamera yang diaktifkan dengan `LatticeCamera`, kamera tersebut akan beroperasi secara bebas.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Tanda Tangan

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Nilai-nilai `force_tier`:
- `"sim-capture-sim-emit"` — simultan sejati (semua kamera memancarkan pada tepi clock yang sama).
- `"sim-capture-ftd-stagger"` — penggeseran waktu yang fleksibel (kamera memancarkan pada waktu yang sedikit bergeser sehingga paket diserialisasikan di jalur).
- `"slip-emit-and-capture"` — penangkapan berurutan per kamera (tanpa sinkronisasi waktu; satu-satunya opsi saat tidak ada ukuran frame yang sesuai dengan mode simultan).

`wire_ceiling_mbps` menggantikan **anggaran kabel berkelanjutan host** dalam MB/s — angka tunggal
yang menjadi acuan alokasi seluruh array. Biarkan tetap pada `None` untuk menggunakan nilai yang terdeteksi secara otomatis
. Kurangi nilainya ketika array melaporkan frame yang rusak akibat GVSP: nilai otomatis tersebut diperoleh
dari laju tautan yang diiklankan oleh NIC, yang cenderung melebih-lebihkan adaptor USB, jalur PCIe yang tipis, dan
fabrik bersama yang sibuk — dan perkiraan yang terlalu tinggi tersebut muncul sebagai frame yang rusak, bukan sebagai
koneksi yang terlihat lambat. Nilai ini disimpan secara permanen di blok penangkapan array proyek, sehingga
pembukaan ulang atau pengaturan `connect_array` di kemudian hari akan mengembalikannya seperti pengaturan array lainnya.
Lihat [Kesehatan Array](#array-health--which-subsystem-is-losing-frames).

#### Over-Subscription (batas bawah per kamera)

Pacing Sim-emit mengalokasikan porsi anggaran kabel yang aman dari tabrakan kepada setiap kamera, dengan batas bawah **8 MB/s per kamera**(`per_cam_floor_bps`). Begitu `N × floor` melampaui batas atas yang aman dari tabrakan, array**melebihi kapasitas kabel**— mode kegagalannya adalah kehilangan paket GVSP, bukan penurunan laju bingkai — dan tidak ada solusi untuk ukuran bingkai:**penggabungan dan ROI mengurangi byte per bingkai, bukan byte per detik yang diatur**yang dibandingkan oleh pemeriksaan agregat. Batas praktis resolusi penuh pada host 1 GbE:**6 kamera @ 1500 MTU, 9 dengan bingkai jumbo** (`max_cams_collision_safe` dalam laporan respons analisis menunjukkan batas maksimum untuk saluran Anda). Solusi: kurangi jumlah kamera, gunakan frame jumbo dari ujung ke ujung, atau gunakan NIC yang lebih cepat.

- Respons `analyze_array_network()` dan `/api/camera/array/connect` memuat `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe`, dan `per_cam_floor_bps`. Ketika `oversubscribed` bernilai true, proyeksi **mengosongkan bidang fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) alih-alih melaporkan laju yang lambat namun masih berfungsi, yang dapat menyesatkan.
- `POST /api/camera/array/connect` menerima parameter body `pin_resolution` (**hanya HTTP — bukan argumen kata kunci SDK**; `connect_array` tidak mengeksposnya). Penetapan (pinning) menghapus jaring pengaman penurunan binning, sehingga koneksi yang kelebihan langganan dengan pengaturan `pin_resolution` akan**ditolak secara tegas** dengan pesan kesalahan yang mencantumkan setiap solusi. Tanpa penguncian, proses koneksi dilanjutkan dengan walk-down tetapi memberikan peringatan bahwa penyusutan tidak dapat membersihkan agregat.
- Jalan keluar untuk pengujian: atur `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` di lingkungan backend untuk menurunkan tingkat penolakan menjadi peringatan keras — Anda tetap dapat terhubung dan menerima kehilangan paket yang hilang.

#### Kesehatan Array — subsistem mana yang mengalami kehilangan frame

`GET /api/camera/array/<array_id>/capability` menampilkan blok `health` yang aktif pada
array yang terhubung, dievaluasi ulang dalam jendela bergulir **10 detik**. Indikator ini membagi kehilangan frame
menjadi dua penyebab yang memerlukan perbaikan berbeda, alih-alih satu tingkat &quot;tidak lengkap&quot; yang
tidak menyebutkan keduanya:

| Kolom | Artinya | Subsistem mana |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per serial) | Frame **telah tiba namun secara struktural rusak**— kehilangan paket GVSP. |**Jaringan**: alokasi bandwidth, pacing, cincin penerima NIC, MTU |
| `never_arrived_rate_pct` (per serial) | Frame **sama sekali tidak pernah tiba**— kamera tidak memicu, atau tidak ada data yang dikirimkan. |**Pemicu / sinkronisasi**: kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Tingkat terburuk kamera untuk masing-masing. | — |
| `per_cam_rate_pct` | Tingkat ketidaklengkapan gabungan per kamera (kedua penyebab digabungkan). | — |
| `stable_for_seconds` | Berapa lama setiap kamera berada di bawah 0,01 %. | — |

Selain `health`, catatan yang sama melaporkan angka alokasi keseluruhan yang tersisa:

| Kolom | Artinya |
| --- | --- |
| `wire_ceiling_mbps` | Anggaran bandwidth yang berlaku secara berkelanjutan pada host, MB/s. |
| `wire_ceiling_source` | Asal usul angka tersebut, dalam bentuk teks — misalnya `USB-capped 200 MB/s (was theoretical 1062; …)` atau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` ketika `wire_ceiling_mbps=` menetapkannya. |
| `nic_is_usb` | `true` untuk adaptor Ethernet USB. |

Tidak ada SDK wrapper untuk titik akhir ini — bacalah secara langsung:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Membacanya:** nilai `gvsp_corrupt_rate_pct` yang tidak nol dengan `never_arrived_rate_pct` bernilai 0 berarti
pemicu dan sinkronisasi kabel sempurna, serta 100% kehilangan terjadi pada jalur jaringan — turunkan
nilai `wire_ceiling_mbps` dan sambungkan ulang. Pola sebaliknya mengindikasikan masalah pada kabel sinkronisasi atau
jalur pemicu.

> **`target_fps` bukanlah faktor penentu untuk frame yang rusak.** Pengaturan kecepatan GevSCPD ditulis sekali saat
> terhubung, sehingga menurunkan laju pemicu mengubah siklus kerja, bukan
> laju burst emisi simultan. Pengurangan permintaan sebesar 5× yang diukur tidak menghasilkan perbaikan, sementara
> menurunkan batas maksimum kabel dari 240 menjadi 200 MB/s mengubah tingkat kerusakan pada rig yang sama dari 10,4% menjadi
> 0,00%.

> **Fitur penyusutan otomatis di tengah aliran tidak tersedia pada firmware TRI032S.** Sebuah array yang sedang berjalan tidak dapat
> memperbaiki masalah ini sendiri; putuskan sambungan dan sambungkan kembali agar pengatur waktu koneksi merencanakan ulang berdasarkan
> batas atas yang baru.

Sebuah **adaptor Ethernet USB dibatasi hingga 200 MB/s** oleh probe terlepas dari
spesifikasi teknisnya: tabel efisiensi yang mengubah kecepatan tautan menjadi angka berkelanjutan
berasal dari PCIe, dan adaptor jaringan USB (NIC) mengiklankan kecepatan tautan Ethernet-nya sambil dibatasi oleh
bus USB dan drivernya. Batasan ini bersifat absolut, bukan fraksi — adaptor USB 1 GbE
menghasilkan ~80 MB/s dan tidak terpengaruh.

#### `ArraySession` Metode

| Metode | Deskripsi |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Satu grup penangkapan yang disinkronkan. Mengembalikan `CaptureResult` (daftar kamus bingkai + `.skipped`). Kontrol ekspor di bawah ini. |
| `capture(..., smart=True)` | **Perekaman cerdas** — menunggu hingga AE stabil di semua kamera, lalu memicu perekaman. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Pengambilan tercepat: hanya data mentah + pembacaan DAQ yang ditetapkan (+ indeks gabungan bebas). Mencerminkan tombol &quot;Perekaman Tercepat&quot;. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Tunggal / Berkelanjutan / Interval dalam satu loop terbatas. Mengembalikan `list[CaptureResult]`.**Membutuhkan `count` dan/atau `duration_s`** agar dapat dihentikan (SDK tidak memiliki Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Mulai merekam tampilan indeks gabungan langsung ke video/GIF → `RecorderHandle`. Satu perekam gabungan per array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Mulai pengambilan gambar beruntun raw-Bayer dengan fps tinggi → `RecorderHandle`. Proses ulang secara offline dengan `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Memproses ulang secara offline rentetan gambar mentah yang tersimpan menjadi video yang telah dikalibrasi. Menunggu hingga selesai (`wait=True`) dan mengembalikan `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Memeriksa status pekerjaan pembangunan offline: `{running, result, error, burst_dir}`. |
| `disconnect()` | Melepaskan seluruh array. |

Kontrol ekspor `capture()` (menggunakan titik akhir yang sama dengan yang digunakan oleh GUI/CLI):

- `processing` / `levels` — `processing="all"` (atau `levels=["raw","radiance",…]`) menyimpan setiap jenis ekspor yang berlaku per kamera; satu nilai `processing` hanya menyimpan tingkat tersebut.
- `aligned=True` — menyesuaikan ekspor non-raw setiap anggota ke [profil penyelarasan](#array-alignment) array (terdaftar bersama); raw tetap tidak disesuaikan tetapi membawa transformasi dalam metadata. Akan kembali ke mode tidak teralignasi (dengan peringatan yang ditampilkan di `alignment` hasil) jika array tidak memiliki profil.
- `render_index=False` — lewati overlay indeks vegetasi per kamera; secara default menampilkannya di tempat yang telah dikonfigurasi.
- `force_daq=True` — menyimpan pembacaan DAQ/DLS yang ditetapkan sebagai file sidecar `.daq` meskipun tidak ada tingkat yang dipilih yang membutuhkannya.

**Kompresi TIFF (hanya opsi HTTP):**`ArraySession.capture()` tidak mengirimkan kunci `compression`, sehingga pengaturan default backend berlaku — `POST /api/camera/array/capture` membaca parameter tubuh `compression`parameter tubuh , `"deflate"` secara default (zlib L1 tanpa kehilangan data + prediktor horizontal, ~4,1 MB per bingkai resolusi penuh). `"none"` menulis tanpa kompresi (~6,3 MB/bingkai) dengan**kecepatan penulisan ~5× lebih cepat** — keduanya lossless dan dibaca secara identik saat diimpor. SDK tidak menyediakan argumen kata kunci (kwarg) untuk ini; jalan keluarnya adalah `chloros-cli lattice array-capture --compression none` atau HTTP mentah. DEFLATE juga memegang GIL (Python), sehingga penulisan terkompresi tidak dapat diparalelkan di antara benang penulisan per kamera — perekaman resolusi penuh berkelanjutan dengan 8 kamera pada laju sensor memerlukan `compression: "none"`. Detail: [CLI Referensi → array-capture](cli-reference.md).**Penggantian ekspor per-anggota (hanya HTTP):**titik akhir yang sama juga menerima `exclude_serials` (daftar — menghapus anggota dari kumpulan yang disimpan; array tersebut tetap dipicu sebagai satu grup yang disinkronkan dan anggota yang dikecualikan dikembalikan dalam `excluded`), `serial_levels` (penggantian tingkat per-kamera `{serial: [level tokens]}`), dan `serial_index` (penggantian `{serial: bool}` perindeks per-kamera). Ini adalah parameter tubuh yang setara dengan GUI dan**belum merupakan argumen kata kunci SDK**; anggota yang tidak ada dalam peta akan dialihkan ke pengaturan tingkat array `levels` / `render_index`.

##### Memeriksa Kamera yang Dilewati — `CaptureResult.skipped`

`ArraySession.capture()` mengembalikan sebuah `CaptureResult`, yang merupakan subkelas dari `list`: lakukan iterasi, indeks, `len()` — setiap pola yang ada tetap berfungsi. Kode baru dapat memeriksa atribut `.skipped` untuk melihat cam mana saja yang dikecualikan dan alasannya. Kasus yang paling umum adalah kamera &quot;RGB&quot; dalam susunan filter campuran saat Anda meminta `processing="radiance"` atau `"reflectance"` — radian per piksel Bayer tidak bermakna bagi sensor broadband, sehingga bagian belakang sistem melewatkan kamera-kamera tersebut daripada menghasilkan data yang tidak bermakna.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Token alasan mengikuti pola `<level>-not-applicable-to-rgb-cam` (satu entri per tingkat yang dilewati, masing-masing berisi `level`). Pengabaian yangadalah `reflectance-skipped-no-fresh-dls` (tidak ada pembacaan downwelling baru yang tersedia), `reflectance-skipped-bound-daq-unavailable (…)` (DAQ yang terikat tidak dapat dijangkau), dan `dls-uncalibrated-band-<nm>` — pita tersebut sebagian besar berada di luar rentang yang dikalibrasi secara radiometrik oleh sensor cahaya DAQ (~374–974 nm), sehingga pembagian reflektansi absolut berbasis DAQ ditolak dan bingkai tersebut diturunkan ke respons sensor. Di antara SKU yang tersedia, hanya F988 yang memicunya; jalur yang didukung kamera tersebut adalah alur kerja panel reflektansi.

Tingkat `processing`:

| Tingkat | Keluaran |
| --- | --- |
| `"raw"` | Bayer saluran tunggal (kamera monokrom: pita tunggal) langsung dari sensor. |
| `"debayered"` *(default SDK)* | BGR 3 saluran melalui demosaik bilinear (kamera monokrom: skala abu-abu 1-saluran). |
| `"radiance"` | float32 W/m²/sr/nm melalui rantai radiometrik lengkap. Hanya multispektral — kameraRGB dilewati. |
| `"reflectance"` | uint16 0..32768 (siap untuk Pix4D); memerlukan pemasangan DAQ langsung untuk referensi absolut. Hanya multispektral. |
| `"display"` | Rantai lengkap yang sesuai dengan pratinjau GUI (CCM + WB + gamma sesuai profil kamera). |
| `"all"` | **Satu berkas per tingkat yang berlaku** untuk setiap kamera (sesuai dengan pengaturan default &quot;Capture All&quot; / CLI di antarmuka pengguna). Berkas `CaptureResult` yang dikembalikan kemudian berisi satu kamus bingkai per `(cam, level)`, dengan tingkat yang tercantum dalam setiap kamus; tingkat yang tidak berlaku muncul di `.skipped`. Pembacaan DAQ yang digunakan untuk setiap frame reflektansi disimpan sebagai sidecar `.daq`. |

> **Catatan — pengaturan default berbeda dari CLI.** `ArraySession.capture()` secara default diatur ke `processing="debayered"`; perintah `chloros-cli lattice array-capture` secara default diatur ke `processing="all"`. Berikan `processing="all"` secara eksplisit dari SDK untuk mencerminkan penyimpanan multi-level di CLI /GUI.

### Mode Pengambilan &amp; Perekam

Permukaan array mencerminkan panel pengambilan di GUI: mode rana Tunggal / Berkelanjutan / Interval / Tercepat, ditambah dua perekam (video komposit langsung dan raw burst → pemrosesan ulang offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**adalah loop Kontinu/Interval dari SDK. Karenatidak ada `Ctrl+C` untuk menghentikannya dari skrip, Anda**harus** meneruskan `count` dan/atau `duration_s` (proses akan berhenti saat salah satunya tercapai). `interval_s` diukur dari awal setiap putaran (sesuai dengan GUI). Argumen kata kunci (kwargs) lainnya diteruskan langsung ke `capture()`.
- **`record`** berkategori *monitoring*: nilai ini menangkap komposit indeks gabungan secara langsung sebagaimana ditampilkan, sehingga aliran gabungan harus terbuka agar frame dapat diterima. Satu perekam komposit per array (akan memicu pengecualian jika sudah ada yang berjalan).
- **`burst` → `build_video`** adalah *tingkat analisis*: `burst` menulis frame mentah + manifes per frame + satu `.daq` per pembacaan DLS yang berbeda di bawah `<output>/bursts/<base>/` pada kecepatan penuh loop pengambilan (tanpa rantai, tanpa exiftool, tanpa tampilan langsung). `build_video` mencocokkan waktu setiap bingkai dengan `.daq` terdekat dan menjalankan kembali rantai radian/reflektansi/indeks dari pipeline impor. `products` adalah daftar `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (default: indeks gabungan). `burst().stop()` juga secara otomatis memicu pembuatan indeks gabungan dengan upaya terbaik, yang dikembalikan sebagai `build_job` dalam hasil penghentian.

#### `RecorderHandle`

Dikembalikan oleh `ArraySession.record()` dan `ArraySession.burst()`. Gunakan sebagai pengelola konteks untuk menghentikan secara otomatis saat keluar dari lingkup, atau jalankan secara manual.

| Anggota | Deskripsi |
| --- | --- |
| `job_id` | ID pekerjaan backend (str). |
| `kind` | `"composite"` (dari `record`) atau `"raw"` (dari `burst`). |
| `start_stats` | Kamus yang dikembalikan oleh panggilan `start`. |
| `result` | `None` saat berjalan; kamus hasil akhir setelah dihentikan. |
| `stats(timeout=10.0)` | Statistik pekerjaan secara real-time (frame yang ditulis, fps yang tercapai, waktu yang berlalu). |
| `stop(timeout=60.0)` | Menghentikan perekam; mengembalikan dan menyimpan hasil akhir dalam cache. Idempoten (panggilan kedua akan mengembalikan hasil yang disimpan dalam cache). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Menyambungkan ke Array yang Sudah Terhubung — `attach_array`

Jika array sudah aktif (dibuka oleh GUI, atau sesi SDK sebelumnya memanggil `connect_array`), gunakan `attach_array` untuk mendapatkan handle ke array tersebut alih-alih menghubungkannya kembali. `connect_array` selalu menampilkan pesan kesalahan &quot;Kamera  <sn>sudah </sn>ada <sn>dalam array<id>&quot; dalam situasi tersebut, karena mengirimkan POST ke `/array/connect` untuk anggota dalam pool tidak bersifat idempoten; `attach_array` membaca `/api/camera/array/list` dan mencocokkan baik berdasarkan array_id maupun serials.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Pola: SDK skrip yang berbagi tenant dengan GUI desktop harus mencoba `attach_array` terlebih dahulu dan beralih ke `connect_array` jika belum ada array dalam pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Penting — saat context-manager keluar, koneksi AKAN terputus.**`ArraySession.disconnect()` selalu mengirim POST ke `/array/disconnect`; tidak ada penjaga &quot;terhubung tetapi tidak dimiliki&quot; seperti yang ada pada `CameraSession` / `DAQSensorSession`. Jika Anda berbagi ruang dengan antarmuka pengguna grafis (GUI) dan tidak ingin menghapus array saat keluar dari ruang lingkup,**jangan gunakan blok `with`** — simpan handle tersebut dalam variabel biasa dan lewati perintah eksplisit `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Alat Bantu Analisis Jaringan

Berguna sebelum membuka array — memprediksi apakah pengaturan yang Anda usulkan akan sesuai:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` adalah salah satu dari `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (jika tidak, maka `error`). `auto_capped_fps` berarti resolusi yang diminta hanya sesuai dengan cincin RX pada laju pemicu yang dibatasi — pertahankan resolusi tersebut dan alihkan `target_fps=result["recommended"]["recommended_target_fps"]` ke `connect_array` (lihat [Contoh 6](#6-pengujian-kemampuan-sebelum-menghubungkan-susunan-4-kamera)).

**Cara membaca proyeksi** (model yang sama dengan panel Pengaturan Susunan di GUI):

- **Burst (`frame_bytes_total`) dijumlahkan per kamera pada format piksel asli masing-masing kamera.**Kamera mono**M3M**mengalirkan data Mono12 (2 B/px) terlepas dari nilai `pixel_format` yang Anda berikan, sehingga satu bingkai resolusi penuh dari 4 kamera berukuran**~25 MB** jika menggunakan tiga kamera mono, bukan ~12,6 MB seperti yang diasumsikan pada konfigurasi 8. Backend menentukan format masing-masing kamera berdasarkan modelnya.
- **Admittance (`burst_fits_nic_ring`) peka terhadap drain**, bukan perbandingan antara burst utuh dan ring: sim-emit berlaku ketika host mengosongkan ring RX lebih cepat daripada kamera mengisinya. Host 10G + kamera 1 GbE**menerima** resolusi penuh bahkan ketika burst melebihi kapasitas ring; host 1 GbE akan memblokir (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` merupakan batas atas konservatif untuk pengambilan serial** — `max(readout+emit, N×emit)` dengan emisi per kamera dibatasi oleh tautan kamera 1 GbE, tidak bergantung pada eksposur. Misalnya, ~2,8 fps untuk susunan 4 kamera resolusi penuh 12-bit (sesuai dengan hasil pengukuran runtime ~2,7–3,0). Model lengkap: [CLI Referensi → Model fps &amp; burst susunan](cli-reference.md#array-fps--burst-model).
- **Over-subscription (`oversubscribed: true`) berarti batas bawah N × per kamera melebihi batas aman tabrakan-**batas atas yang aman** — bidang fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) menunjukkan nilai 0, dan penyusutan otomatis/pengelompokan tidak dapat memperbaikinya (metode tersebut mengurangi byte per frame, bukan byte per detik yang diatur). Solusinya adalah mengurangi jumlah kamera, menggunakan jumbo frame, atau menggunakan NIC yang lebih cepat; `max_cams_collision_safe` melaporkan batas atas (6 kamera resolusi penuh pada 1 GbE @ MTU 1500, 9 kamera dengan jumbo). Respons tersebut juga menyertakan `aggregate_demand_bps`, `collision_safe_ceiling_bps`, dan `per_cam_floor_bps` (8 MB/s). Lihat [Over-Subscription](#over-subscription-the-per-cam-floor).

### Penemuan &amp; Daftar

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Array LATTICE menjalankan AE berkelanjutan di latar belakang segera setelah terhubung, tetapi pemandangan yang baru diarahkan memerlukan waktu sejenak untuk konvergen. **Smart-capture** adalah fitur praktis yang terintegrasi: fitur ini memeriksa eksposur setiap kamera, menunggu hingga array stabil di seluruh jendela, lalu memicu proses penangkapan gambar. Fitur ini setara dengan antarmuka pengguna grafis (GUI): tombol &quot;smart&quot; pada aplikasi desktop memanggil titik akhir backend yang sama.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Saat mengoperasikan melalui `ChlorosProject` (bagian berikutnya), Anda akan mendapatkan lebih banyak pengaturan:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Kebijakan smart-AE secara default bersifat konservatif. Perketat pengaturan `exposure_tolerance_pct` untuk pekerjaan radiometri yang menuntut ketelitian tinggi; longgarkan untuk adegan yang berubah cepat di mana Anda hanya menginginkan hasil yang &quot;cukup mendekati&quot;.

---

## Sesi Sensor DAQ

Kumpulan backend persisten untuk sensor spektral (DAQ-U melalui USB, DAQ-M melalui BLE, DAQ-E melalui Ethernet). Mencerminkan antarmuka kamera: smart-detect, penggunaan ulang kumpulan, pengikatan idempoten.

### Deteksi Cerdas (Tanpa Konfigurasi)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioritas: Ethernet → BLE → USB. Berikan petunjuk eksplisit apa pun untuk mengunci transportasi.

### Transportasi yang Dikunci

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Metode `DAQSensorSession`

| Metode | Deskripsi |
| --- | --- |
| `status(timeout=10.0)` | Ringkasan entri pool (status streaming/perekaman, rentang panjang gelombang, SHA kalibrasi, waktu integrasi, frame_avg, status AE). |
| `latest(n=1, timeout=10.0)` | Mengembalikan hingga N frame spektrum terbaru. |
| `stream_start()` / `stream_stop()` | Melanjutkan / menjeda streaming (handle tetap terbuka). |
| `record_start(output_dir=None, device_name=None)` | Memulai perekaman berkas .daq. Mengembalikan jalur berkas. Ditolak untuk DAQ-U/M tanpa paket kalibrasi AWS (DAQ-E dikecualikan). |
| `record_stop()` | Hentikan perekaman. Mengembalikan `{path, rows}`. |
| `disconnect()` | Melepaskan dari pool. Tidak beroperasi untuk handle yang terhubung tetapi bukan milik sendiri. |

> **Profil koreksi Cap (`cap_id`) bukanlah pengaturan yang dapat diSDK.** `connect_daq_sensor()` / `DAQSensorSession` tidak mengekspos parameter `cap_id` atau metode `set_cap`. Pilih profil koreksi batas armada melalui CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) atau rute &quot;HTTP&quot; pada backend (`/api/daq/connect` dan `/api/daq/<id>/cap-id` menerima `cap_id`).

### Penemuan — mencari alamat untuk terhubung

`discover_daq_sensors()` memindai USB / BLE / ETH untuk mencari sensor yang *mungkin* dapat Anda buka. Ini adalah mitra DAQ dari `discover_lattice_cameras()`, dan satu-satunya cara untuk mendapatkan **MAC BLE DAQ-M** — DAQ-E memiliki nama host dan DAQ-U memiliki port COM, tetapi MAC tidak tercetak pada perangkat maupun tercantum oleh sistem operasi.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Kolom | Deskripsi |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / MAC BLE / nama host — diteruskan ke `connect_daq_sensor` sebagai `port=` / `mac=` / `eth_host=`. |
| `display` | Label yang mudah dibaca manusia. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, atau `None` untuk port yang tidak dapat diidentifikasi oleh pemindaian (adaptor serial USB tidakdapat dibedakan tanpa probe, sehingga yang tidak dikenal ditampilkan daripada disembunyikan). |
| `extra` | Rincian per-transport (nama yang diiklankan BLE, produsen USB, IP/fw DAQ-E/…). Nilai kosong diabaikan. |

| Parameter | Default | Deskripsi |
| --- | --- | --- |
| `transports` | ketiga-tiganya | Urutan (atau string CSV) yang membatasi pemindaian. Sebaiknya disertakan jika Anda tahu apa yang diinginkan — BLE adalah bagian yang lambat. |
| `scan_timeout` | 5 | Jendela pemindaian per-transport dalam detik; backend membatasi nilainya antara 1–20. |
| `timeout` | 60,0 | Batas maksimum `HTTP` untuk seluruh panggilan (seperti di bagian lain dalam `SDK`). |
| `auto_start_backend` | `True` | Menjalankan backend lokal jika tidak ada yang sedang berjalan. Tidak pernah dijalankan untuk `backend_url` jarak jauh. |

> **Sensor yang sudah terbuka di pool tidak akan muncul.** Periferal BLE yang sudah terhubung akan berhenti mengiklankan diri, dan port COM yang terbuka tidak dapat diuji, sehingga daftar penemuan menampilkan apa yang *tersedia untuk dihubungkan*. Hasil kosong segera setelah Anda menghubungkan sesuatu adalah hal yang diharapkan — gunakan `list_daq_sensors()` untuk perangkat yang sudah Anda miliki. Transport yang pemindaiannya tidak dapat dijalankan (tidak ada bleak / zeroconf yang terpasang) akan dilewati alih-alih memicu kesalahan, sehingga mesin tanpa Bluetooth tetap mendapatkan respons USB dan ETH-nya.

### Daftar

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Penggunaan Bersama dengan GUI / CLI

Jika GUI sudah memiliki sensor yang terbuka, memanggil `connect_daq_sensor(port="COM3")` dari Python akan mengembalikan handle yang ditandai sebagai `already_connected=True`. `disconnect()` sesi tersebut kemudian menjadi no-op sehingga skrip SDK Anda tidak mencabut sensor dari GUI saat keluar dari scope.

### Kelas Perangkat Keras Langsung (Tanpa Backend)

`daq_sdk` diekspor ulang oleh `chloros_sdk` sehingga Anda juga dapat mengendalikan sensor secara end-to-end dalam proses tanpa backend:

> **Ketersediaan:**`daq_sdk` disertakan dalam instalasi desktop Chloros,**bukan** dalam paket PyPI — `pip install chloros-sdk` menyediakan `lattice_sdk` tetapi tidak menyertakan `chloros_sdk.DAQ_AVAILABLE == False`. Periksa bendera tersebut sebelum menggunakan kelas-kelas ini; pada host yang hanya menggunakan pip, kendalikan sensor melalui [`connect_daq_sensor()`](#daq-sensor-sessions) sebagai gantinya, yang tidak memerlukan pustaka transportasi lokal.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Pilihlah jalur smart-connect (`connect_daq_sensor`) jika Anda ingin kepemilikan bersama dengan GUI; gunakan kelas langsung untuk skrip headless yang memiliki sensor secara eksklusif.

---

## Otomatisasi Proyek — `ChlorosProject`

Proyek &quot;Chloros&quot; yang disimpan adalah sebuah folder yang berisi `cameras.json` + `sensors.json` + `project.json`. `open_project` memuat manifes, dan `connect_all` mengaktifkan setiap perangkat yang disimpan secara daring dengan pengaturan yang disimpan — keadaan perangkat keras yang sama seperti yang akan dihasilkan oleh antarmuka pengguna grafis (GUI).

### Contoh Minimal

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Atau sebagai pengelola konteks:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Metode `ChlorosProject`

| Metode | Deskripsi |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Menemukan + menghubungkan setiap perangkat yang disimpan. Mengembalikan laporan koneksi per kelas. Menggunakan backend yang sedang berjalan jika ada yang mendengarkan di `127.0.0.1:5000`; jika tidak, secara diam-diam beralih ke kontrol perangkat langsung (tanpa backend) `lattice_sdk` — tidak pernah membuat backend. |
| `disconnect_all()` | Memutuskan semua koneksi. |
| `capture_all(output_dir=".")` | Satu bingkai dari setiap kamera + array + spektrum dari setiap sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Generator yang menghasilkan frame BGR `numpy` dari kamera (atau array) yang ditentukan. `overlays=False` adalah loop pengambilan langsung `lattice_sdk` (array menghasilkan kamus `{serial: frame}`). `overlays=True` diteruskan melalui `ChlorosLocal.camera_stream()` → umpan MJPEG `/api/camera/<serial>/stream-annotated` dari backend, dengan blok `ui.overlay` diteruskan sebagai parameter kueri. Membutuhkan mode backend dan **kamera mandiri**: kamera dalam mode langsung memicu `RuntimeError` (backend tidak dapat mengambil kamera yang dimiliki oleh proses ini) dan sebuah array memicu `NotImplementedError` (menumpuk komposit per kamera — mengalirkan anggota berdasarkan nama). Setara dengan satu kali eksekusi: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Jalankan penyelarasan pada setiap array yang saat ini terhubung. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Jalankan pipa kalibrasi/indeks pada gambar-gambar proyek (mencakup `ChlorosLocal.process`; keempat ini adalah **satu-satunya** argumen kata kunci yang diterima — `indices=` dst. memicu pengecualian `TypeError`; atur indeks melalui `ChlorosLocal.configure()`). Membangun `ChlorosLocal()` secara malas, yang secara otomatis memulai backend. |

Atribut:
- `proj.cameras` — `Dict[str, CameraHandle]` yang diindeks berdasarkan nama DAN nomor seri.
- `proj.arrays` — `Dict[str, ArrayHandle]` yang diindeks berdasarkan nama DAN array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` yang diindeks berdasarkan nama DAN slot_id.
- `proj.config` — `project.json["config"]` kamus.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Tingkat pemrosesan.** `capture()`, `grab()`, dan `frame_stream()` semuanya menggunakan token `processing`
, dan rantai ini bersifat kumulatif — setiap tingkat menjalankan semua yang ada di atasnya:

| Tingkat | Keluaran | Catatan |
| --- | --- | --- |
| `raw` | Bayer 1-saluran, asli sensor | Tanpa demosaik. Overlay tidak tersedia pada tingkat ini. |
| `debayered` | BGR 3-saluran (**default**) | Demosaik bilinear. Satu-satunya tingkat yang berfungsi tanpa mode backend. |
| `radiance` | float32, W/m²/sr/nm | Rantai radiometrik lengkap: demosaik + pemisahan warna 3×3 (multispec) + DSNU + flat-field + skala NIST, dengan eksposur × gain dipisahkan sehingga nilainya absolut. |
| `reflectance` | uint16, 32768 = 1,0 | Radian dibagi dengan irradiansi turun (ρ = π·L/E). Membutuhkan pembacaan DLS/DAQ — lihat catatan di bawah. |
| `display` | 8-bit mirip sRGB | Hasil render setara antarmuka pengguna: CCM + white balance + gamma melalui profil warna aktif kamera. |

Apa pun selain `debayered` memerlukan mode backend; kamera mode langsung akan memicu
`NotImplementedError`. `reflectance` memerlukan pembacaan irradiansi turun yang dapat digunakan — titik akhir bingkai secara otomatis menarik
DAQ yang digabungkan ke slot DLS kamera secara otomatis, tetapi tanpa DAQ yang terikat, rantai menolak
keluaran reflektansi dan secara jujur menandai penurunan kualitas dalam metadata yang dikembalikan daripada diam-diam
mengembalikan hasil yang lebih rendah.

> **Skala DN reflektansi — jangan dikodekan secara permanen.** Reflektansi LATTICE menggunakan `32768` = ρ 1,0 dan menandai
> XMP `Chloros:PixelScale=32768`; reflektansi Survey3 menggunakan `65535` = ρ 1,0 dan tidak memiliki
> tag `Chloros:*`. Baca tag tersebut dan bagi dengan nilainya. Nilai ini didefinisikan dalam domain uint16, sehingga tetap
> `32768` untuk setiap format yang melakukan penskalaan ulang (TIFF 16-bit, 8-bit PNG /JPG, 32-bit persentase) — normalisasikan
> tipe data yang disimpan kembali ke uint16 terlebih dahulu (×257 dari 8-bit, ×65535 dari float). Satu-satunya pengecualian:
> tangkapan sumber 8-bit yang ditulis sebagai 8-bit TIFF akan *dipotong*, bukan diskalakan ulang, sehingga tidak ada skala yang menggambarkannya
> — Chloros mengabaikan `PixelScale` dan tuple MicaSense sepenuhnya dalam kasus tersebut. Perlakuan tag yang hilang
> pada berkas reflektansi LATTICE sebagai &quot;tidak ada skala yang valid&quot;, bukan sebagai nilai default.

> **EXIF diteruskan ke ekspor.** `process()` menyalin blok GPS dari pengambilan sumber
> **dan ExifIFD-nya** ke setiap produk, sehingga ekspor membawa `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal`, dan `CameraSerialNumber` serta
> georeferensi. `FocalLength` adalah data yang digunakan Pix4D untuk menghitung jarak sampel tanah — tanpa data ini,
> rekonstruksi akan kembali ke skala yang sangat salah (dalam sebuah kasus yang diukur, lokasi seluas 411 m
> berubah menjadi 47,8 km). Salinan ini sengaja bukan `-all:all`: tag struktural IFD0 merusak
> keluaran LATTICE, dan `ExifImageWidth`/`Height` dikecualikan karena keduanya mendeskripsikan
> pengambilan sumber, bukan raster yang diekspor.

Sub-flag tahap penangkapan (berlaku untuk tingkat radiometrik — `radiance`, `reflectance`, `display`):

| Flag | Default | Arti |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + penyetaraan-field + pemisahan 3x3 + skala radiometrik NIST. |
| `apply_white_balance` | `True` | WB LUT. Mendukung DLS saat DAQ terikat pada kamera. |
| `apply_index` | `False` | Evaluasi indeks vegetasi. |
| `index_expression` | `None` | Rumus pengganti. Tidak kosong → mengaktifkan indeks secara otomatis. |
| `annotated` | `False` | Overlay dekorasi GUI (zebra/grid/peaking). Tidak tersedia untuk `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Tipe kembaliannya adalah `CapturePathMap`, bukan `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` adalah `Dict[str, Union[str, List[str]]]`: tingkat tunggal
> `processing` memberikan satu jalur untuk setiap nomor seri, sedangkan yang bertingkat (`"all"`, atau
> daftar `levels` yang eksplisit) memberikan **daftar terurut** dari setiap produk yang disimpan untuk
> kamera. Komposit gabungan langsung, jika ada yang sedang streaming, tiba di bawah kunci tambahan
> `"combined"` alih-alih di bawah nomor seri. Kode yang mengasumsikan `str` akan gagal pada
> bentuk daftar tanpa ada pemeriksa tipe yang keberatan — anotasi tersebut menyebutkan `Dict[str, str]`
> selama beberapa waktu setelah bentuk daftar dirilis, itulah sebabnya alias tersebut ada. Normalisasikan
> jika Anda menginginkan bentuk datar:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Penyelarasan Array

`ArrayHandle` menampilkan seluruh permukaan penyelarasan. Profil bersifat hanya untuk sesi secara default — panggil `export_alignment()` secara eksplisit untuk menyimpannya.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Penyelarasan Saat Koneksi

`connect_all(align=...)` dapat secara otomatis menyelaraskan setiap array saat koneksi:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Akan menggunakan `project.json["config"]["auto_align_on_connect"]` sebagai cadangan jika tidak ditentukan.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Perangkat Keras Langsung (Tanpa Backend)

Jika Anda ingin tidak bergantung sama sekali pada backend (CI, robot tanpa antarmuka pengguna, tertanam), impor `lattice_sdk` dan `daq_sdk` secara langsung — keduanya diekspor ulang oleh `chloros_sdk`. Perhatikan `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` terdapat dalam paket PyPI (namun memerlukan runtime Arena SDK), sedangkan `daq_sdk` hanya disertakan dalam instalasi desktop.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Preset dan pemicu

Tiga dari empat preset **free-run**: kamera melakukan eksposur secara terus-menerus dan
`capture()` mengembalikan frame berikutnya. `triggered` merupakan pengecualian — preset ini mengaktifkan
kamera untuk mendeteksi tepi perangkat keras pada Jalur 2, sehingga tidak menangkap apa pun hingga tepi tersebut terdeteksi.

| Pengaturan Awal | Pemicu | Gunakan saat |
| --- | --- | --- |
| `default` | free-run | penggunaan umum |
| `high_speed` | free-run | 8-bit, batas 60 fps, eksposur singkat |
| `high_quality` | free-run | 12-bit, tanpa batasan fps — pilihan umum untuk foto diam |
| `triggered` | **diaktifkan, Jalur 2** | kamera terhubung ke kabel sinkronisasi M8 dan ada perangkat lain yang memicunya |

Jika Anda memilih `triggered` (atau mengatur `trigger_mode="On"` sendiri) tanpa ada yang
mengendalikan Line 2, setiap `capture()` akan mengalami timeout — hal ini wajar, karena Anda meminta
kamera untuk menunggu. Berkas SDK menjelaskan hal ini saat terjadi; lihat
[SC_ERR_TIMEOUT selama pengambilan gambar](#direct-hardware-backend-free).

> **Catatan — Pesan &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011` saat terhubung bukanlah kesalahan.**&gt; Saat terhubung, SDK mencoba menegosiasikan**jumbo frames** (paket GVSP berukuran 9000 byte) untuk throughput yang lebih tinggi. Pada koneksi NIC point-to-point langsung (misalnya alamat `169.254.x.x` link-local), jaringan biasanya tidak dapat menangani jumbo frames, sehingga probe ini mengalami timeout dan mencatat baris seperti:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Inilah **solusi cadangan yang dirancang**: SDK secara otomatis kembali ke paket standar 1500 byte dan kamera tetap terhubung seperti biasa (baris-baris `[chunk-enable …]` yang mengikuti merupakan bagian dari urutan koneksi normal). Perekaman tetap berfungsi.
>
> Anda dapat melewati probe ini, tetapi **ini bukan sekadar penekan log — probe ini menonaktifkan jumbo frame.** Kamera hanya merespons ping Don&#x27;t-Fragment hingga 1500 byte, terlepas seberapa baik jaringan Anda, sehingga uji ping saja tidak pernah dapat mendeteksi jumbo; probe ini adalah satu-satunya yang bisa melakukannya. Nonaktifkan fitur ini dan kamera akan terus mengirimkan paket standar 1500 byte selamanya, di jaringan apa pun:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Hanya berguna di jaringan yang Anda *tahu* tidak dapat menangani jumbo, di mana hal ini menghemat sekitar satu detik waktu koneksi per kamera. Karena ini merupakan pertukaran yang nyata, bukan sekadar perubahan kosmetik, &quot;SDK&quot; kini menampilkan pesan tersebut saat Anda menggunakannya:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Biarkan saja kecuali Anda memiliki alasan.** Jika dibiarkan aktif, setiap koneksi akan mengukur ulang jaringan yang Anda miliki: sambungkan ke switch yang mendukung paket jumbo, dan koneksi berikutnya akan secara otomatis mendeteksi paket jumbo tanpa perlu konfigurasi apa pun dan tanpa perlu me-restart.
>
> Jika Anda *ingin* throughput paket jumbo, aktifkan paket jumbo dari ujung ke ujung (MTU NIC 9000 + switch yang mendukungnya), atau tetapkan ukurannya dengan `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` saat Anda yakin tautan tersebut mendukungnya — meskipun lebih disarankan menggunakan `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` per perintah daripada mengaturnya secara permanen, karena ukuran yang ditetapkan secara tetap akan melewati proses pengujian dan menghentikan adaptasi terhadap jaringan di depannya. **Setiap** perangkat di jalur tersebut harus dapat meneruskan paket jumbo — termasuk splitter atau injektor PoE, yang biasanya menjadi alasan mengapa konfigurasi yang seharusnya mendukung paket jumbo tidaktidak dapat meneruskannya.

> **`SC_ERR_TIMEOUT -1011` selama `capture()` / `grab*()` merupakan masalah yang berbeda — yang satu itu adalah kesalahan yang sebenarnya.**&gt; Catatan di atas hanya mengenai `-1011` yang tercatat oleh**probe connect-time**. Kesalahan yang sama yang muncul dari**penangkapan** berarti kamera terhubung dengan baik tetapi tidak mengirimkan gambar apa pun:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Petunjuk utamanya adalah kamera yang saluran *kontrol*nya dalam kondisi baik — proses penemuan berjalan lancar, pengaturan dan penulisan `[chunk-enable …]` semuanya berhasil — sementara *setiap* bingkai mengalami timeout.
>
> **Penyebab umumnya adalah kamera diaktifkan untuk pemicu perangkat keras.** Dengan `trigger_mode="On"` dan `trigger_source="Line2"`, kamera tidak mengirimkan apa pun sama sekali hingga sinyal listrik tiba di kabel sinkronisasi M8. Jika tidak ada kabel yang menggerakkan jalur tersebut, setiap upaya pengambilan gambar akan menunggu selamanya. Kamera tidak rusak dan jaringan berfungsi dengan baik — jaringan tersebut hanya menjalankan perintah yang diberikan.
>
> `CameraSettings()` serta preset `default` / `high_speed` / `high_quality` beroperasi secara bebas, dan pengambilan gambar yang mengalami timeout saat dalam mode aktif akan menampilkan penjelasan alih-alih hanya menampilkan `-1011`. `PRESETS["triggered"]` mengaktifkan Line2, sesuai desain.
>
> Untuk memaksa kamera apa pun beroperasi dalam mode free-run:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Jika masih mengalami timeout dengan `trigger_mode="Off"`, berarti kamera tersebut benar-benar tidakmengirimkan data — kirimkan log dan `ip link show` kepada kami.

#### Profil Warna (Pratinjau Langsung RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` menentukan profil warna tampilan untuk **pratinjau langsung** pada kamera RGB (kamera multispec mengabaikan pengaturan ini):

| Profil | Arti |
| --- | --- |
| `raw` | Mengabaikan rantai radiometrik sepenuhnya. |
| `linear` | DSNU + flat + WB, tanpa CCM, tanpa gamma. |
| `natural` | Linear + CCM terukur + gamma sRGB, hanya dengan penyelesaian standar (penghalusan kroma + desaturasi sorotan) — pengaturan default yang realistis. |
| `enhanced` | `natural` ditambah penyempurnaan hub-parity lengkap (penghilangan fringes, vibrance, kontras lokal CLAHE). Tampilan yang lebih kaya dengan biaya penyempurnaan per frame sekitar **dua kali lipat**, sehingga framerate LIVE menjadi lebih rendah. |
| `custom_temp` | `natural` tetapi WB dikunci pada `custom_cct_k` Kelvin (DLS diabaikan; dibatasi pada 2000–10000 K). |

Profil ini adalah pengatur kecepatan/tampilan **hanya untuk pratinjau langsung**: tangkapan yang disimpan selalu mendapatkan hasil akhir yang kaya dan lengkap terlepas dari profil yang dipilih, sehingga memilih `natural` untuk menghemat waktu bingkai tidak menurunkan kualitas hasil yang tersimpan di disk. Profil yang tidak dikenal akan menaikkan nilai `ValueError`; ketika backend chloros dapat dijangkau, perubahan tersebut juga dikirimkan (POST) ke backend tersebut sehingga bingkai pratinjau berikutnya mencerminkannya (pengguna direct-SDK tanpa backend tetap mendapatkan perubahan pengaturan).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Kamera Mono (M3M) dan `Calibration`

Kamera mono **M3M** (`M3M-<lens>-F<wavelength>`) bersifat single-band: satu bidang skala abu-abu, tanpa mosaik Bayer, tanpa matriks crosstalk spektral 3×3. `Calibration` mengenali hal ini dan menampilkan bendera `is_mono`. Reflektansi tetap berlaku sebagai peta radiometrik per-pita (proses pemisahan adalah matriks identitas), namun perhitungan multi-pita pada satu kamera menghasilkan nilai yang valid alih-alih nilai yang tidak bermakna:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Untuk membuat indeks vegetasi dari perangkat keras mono, gabungkan beberapa kamera M3M pada panjang gelombang yang berbeda menjadi tumpukan multi-pita yang selaras (lihat [Penjajaran Array](#array-alignment)) dan hitung indeks tersebut pada tumpukan tersebut, bukan pada satu kamera.

Mode langsung DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Kunci yang diterima `apply_sensor_settings`**— tepatnya `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; tidak disarankan lagi karena digantikan oleh `cap_id`), `filter_model` (DAQ-M), dan `cap_id` (semua jenis DAQ; `None`/`""`/`"none"` = sensor murni, tanpa koreksi batas). Kunci yang tidak dikenal akan**diabaikan tanpa pemberitahuan** — misalnya, `{"integration_time": 64}` tidak melakukan apa-apa (harus `integration_time_ms`). Mengembalikan `{"applied": [...], "errors": {...}}` dan tidak pernah memicu pengecualian.

`chloros_sdk` hanya mengekspor ulang permukaan inti yang digunakan di atas. API publik `daq_sdk` yang lengkap (22 nama) menambahkan yang berikut ini — impor langsung dari `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Pengecualian

Tangkap kelas dasar untuk menangani &quot;segala hal yang salah saChloros&quot;:

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

> `ChlorosAuthenticationError` dan `ChlorosConfigurationError` diekspor di tingkat atas bersama yang lainnya; keduanya juga dapat diimpor dari `chloros_sdk.exceptions` seperti yang ditunjukkan.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Contoh End-to-End

### 1. Memproses Folder dengan Bilah Kemajuan Kustom

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Array LATTICE Langsung → Reflektansi + Referensi DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Kampanye Pengambilan Data Berbasis Proyek

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Aliran Frame Multi-Kamera → Pipelne NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Skrip Pengambilan Data Langsung ke Perangkat Keras (Tanpa Backend) Tanpa Antarmuka Pengguna

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Pengujian Kemampuan Sebelum Menghubungkan Susunan 4 Kamera

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Setara Resep Perekaman (Python Murni)

DSL resep dari CLI memiliki setara langsung di Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Pengaktifan Otomatis Backend

Titik masuk smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, dan `discover_lattice_cameras` — adalah klien tipis HTTP yang mengasumsikan adanya backend yang mendengarkan pada `127.0.0.1:5000` (URL default permukaan smart-connect). Jika antarmuka grafis (GUI) atau CLI sudah berjalan, salah satunya pasti aktif. Dari skrip dasar, mungkin tidak ada — sehingga fungsi-fungsi ini **secara otomatis menjalankan biner backend yang disertakan** (tanpa jendela, sama seperti yang dilakukan oleh `ChlorosLocal`) sebelum dipanggil pertama kali, kemudian menunggu hingga `backend_startup_timeout` hingga proses tersebut siap.

Aturan:

- **Hanya URL lokal yang akan dijalankan.** Sebuah `backend_url` yang mengarah ke `localhost` / `127.0.0.1` / `[::1]` memenuhi syarat; host lain dianggap sebagai mesin milik orang lain dan tidak akan pernah diaktifkan.
- **Backend dibiarkan tetap berjalan untuk digunakan kembali** (sama seperti CLI) — tidak ada penutupan otomatis saat skrip Anda berakhir.-menjalankan skrip tersebut akan menggunakan kembali backend yang masih aktif.
- **Nonaktifkan dengan `auto_start_backend=False`** pada salah satu panggilan tersebut (misalnya, saat Anda mengarah ke backend jarak jauh, atau Anda mengelola siklus hidup backend sendiri).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Jika biner yang disertakan tidak dapat ditemukan atau dijalankan, panggilan HTTP berikutnya akan memicu kesalahan `ChlorosConnectError` yang dapat ditindaklanjuti dan **sesuai platform**, bukan sekadar jejak penolakan koneksi — di Windows, pesan ini mengarahkan Anda ke aplikasi desktop atau perintah `chloros-cli`; di Linux (tanpa GUI), pesan ini mengarahkan Anda ke perintah `chloros-cli` atau `.deb`.

---

## Lingkungan &amp; Header

SDK menandai setiap panggilan backend HTTP dengan `X-Chloros-Client: sdk`. Backend menerapkan aturan lisensi SDK / CLI (diperlukan login **dan** paket berbayar Chloros+) alih-alih jalur gratis antarmuka grafis. Pengaturan ini dilakukan secara otomatis saat impor — Anda tidak perlu melakukan apa pun.

`http://localhost` dan `http://127.0.0.1` terdeteksi sebagai backend lokal. Panggilan ke host lain (misalnya, layanan analitik Anda sendiri) tidak akan terpengaruh.

Ganti backend URL dengan meneruskan `backend_url=` (atau `api_url=` pada `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url` non-loopback hanya menjangkau backend source/dev — backend bawaan hanya terikat ke loopback; lihat Mode Backend Jarak Jauh untuk pola terowongan.)

---

## Versi &amp; Kompatibilitas

- Versi SDK ditampilkan sebagai `chloros_sdk.__version__`.
- SDK mengikat perilaku ke versi backend yang disertakan. Menggabungkan SDK yang lebih lama dengan backend yang lebih baru biasanya berfungsi (titik akhir yang kompatibel ke depan), tetapi menggabungkan SDK yang lebih baru dengan backend yang lebih lama dapat memunculkan kesalahan `404` pada titik akhir baru — perbarui aplikasi desktop agar sesuai.
- Permukaan smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) dan titik akhir analisis jaringan mengembalikan skema JSON yang stabil; bidang baru bersifat tambahan.

---

## Petunjuk Pemecahan Masalah

- **`ChlorosAuthenticationError: Login required`** → Jalankan `chloros-cli login EMAIL PASSWORD` sekali di mesin ini, atau masuk melalui aplikasi desktop Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Panggilan smart-connect secara otomatis memulai backend lokal, sehingga pesan ini hanya muncul jika biner yang disertakan tidak dapat ditemukan/dijalankan (mis.misalnya, host yang hanya menggunakan pip tanpa paket desktop). Pesan ini disesuaikan dengan platform: di Windows, buka aplikasi desktop atau jalankan perintah `chloros-cli` apa pun; di Linux, jalankan perintah `chloros-cli` (tidak ada antarmuka grafis) atau instal `.deb`. Untuk backend jarak jauh, berikan `backend_url=` (dan `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** saat impor → `lattice_sdk` gagal dimuat (biasanya DLL runtime Arena SDK belum terinstal). Permukaan non-kamera tetap berfungsi.
- **Array connect mengembalikan resolusi di bawah resolusi asli**→ Fitur smart-prep backend secara otomatis memperkecil ukuran bingkai agar sesuai dengan saluran. Gunakan `analyze_array_network()` untuk mengetahui alasannya, kemudian tingkatkan koneksi, terima pengecilan tersebut, atau gunakan `force_tier="slip-emit-and-capture"` untuk pengambilan gambar berurutan. Jaring pengaman pengecilan ini**tidak** mencakup kelebihan langganan agregat (`oversubscribed: true`, bidang fps 0): terlalu banyak kamera untuk saluran tidak dapat diatasi dengan binning/ROI — kurangi jumlah kamera, aktifkan bingkai jumbo, atau pindah ke NIC yang lebih cepat (lihat [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` melaporkan ring penerimaan (RX ring) NIC sangat kecil (~0,26 MB) / gerbang koneksi menampilkan pesan &quot;FRAMES WILL DROP&quot;** → Ring penerimaan NIC host berada pada nilai defaultnya (sering kali diatur ulang menjadi 32 setelah pembaruan driver NIC). Pada set adaptor Realtek USB 10GbE, atur `ReceiveBufferLen=256` dan `PendingReceives=64` (tingkat tinggi), lalu mulai ulang backend agar dapatmembaca ulang ring tersebut. Prosedur lengkap: [CLI Referensi → Pengaturan &amp; Penyesuaian NIC Host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host macet saat restart/shutdown, kemudian muncul kesalahan WMI `Invalid class` / NIC tidak dapat diaktifkan** → Driver USB 10GbE yang menyebabkan `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Perbarui driver adaptor ke versi terbaru (≥ 2026) dan terapkan kembali pengaturan receive-ring. Lihat [Referensi CLI → Pengaturan &amp; Penyesuaian NIC Host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflektansi ditolak** → DAQ yang aktif harus diikat ke kamera (atau array) untuk. Hubungkan melalui GUI atau gunakan `processing="radiance"` (W/m²/sr/nm) yang tidaktidak memerlukan sensor yang dipasangkan.
- **Perekaman `smart=True` memakan waktu lebih lama dari yang diharapkan** → Konvergensi AE bergantung pada dinamika adegan; perketat pengaturan `exposure_tolerance_pct` atau persingkat `stability_window_s` jika Anda menginginkan (kurang stabil).

---

## Lihat Juga

- [Referensi CLI](cli-reference.md) — setiap subperintah CLI mencerminkan panggilan SDK.
- [Panduan Sensor DAQ](../daq/README.md) — aturan pemasangan kabel, kalibrasi, dan perekaman khusus sensor.
- Dokumen daring: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>

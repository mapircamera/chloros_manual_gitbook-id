# Referensi Chloros CLI

**Versi:**

1.2.0**Dibuat:**29 Juli 2026 19:19 ·**Direvisi:** 30 Agustus 2026**Sasaran:** Dioptimalkan untuk penggunaan LLM; mudah dibaca oleh manusia.**Cakupan:** Setiap subperintah `chloros-cli` yang ditujukan untuk pengguna, beserta opsi dan contoh yang dapat disalin dan ditempelkan.

Dokumen ini merupakan referensi lengkap untuk perintah `chloros-cli`yang disertakan dalam MAPIR Chloros. Dokumen ini sengaja dibuat sedetail mungkin agar LLM (atau manusia) dapat menyusun alur kerja apa pun yang didukung dari daftar di bawah ini tanpa perlu memeriksa kode sumber.

Jika Anda hanya membutuhkan poin-poin penting, langsung ke:
- [Panduan Cepat Lima Menit](#five-minute-quickstart)
- [Alur Kerja Koneksi Pertama LATTICE Camera](#lattice-camera-first-connect-workflow)
- [Alur Kerja Koneksi Pertama DAQ Sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Mode Pengambilan, Perekam, &amp; Pemrosesan Ulang Offline](#capture-modes-recorders--offline-reprocess)

---

## Konvensi

- Semua perintah diawali dengan awalan `chloros-cli`. Pada Windows, file biner-nya adalah `chloros-cli.exe`; pada Linux /Jetson, file biner-nya adalah `chloros-cli`.
- Argumen opsional ditampilkan sebagai `--flag`. Argumen posisional yang wajib ditampilkan tanpa tanda kurung.
- Jika nilai default telah ditentukan, mengabaikan bendera (flag) akan menggunakan nilai tersebut.
- CLI adalah klien HTTP ringan yang berjalan di atas backend Chloros (server Flask di `127.0.0.1:5000`). Backend ini dimulai secara otomatis oleh sebagian besar perintah. `CHLOROS_BACKEND_URL=<url>` mengarah ke **`lattice`**,**`project`**, dan**`daq pool-*`** ke backend jarak jauh — perintah inti (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) secara sengaja menandai `http://127.0.0.1:<port>` dan mengabaikannya (literal IPv4 menghindari penalti &quot;Windows&quot; pada `localhost`→`::1` ~2 detikper permintaan). Lihat [Variabel Lingkungan](#environment-variables).
- Diperlukan login akun Chloros+ untuk semua panggilan SDK / CLI (jalankan `chloros-cli login` sekali per mesin; disimpan dalam cache di `~/.chloros/`).
- Contoh menggunakan Linux ; pada Windows, ganti `/home/user/...` dengan `C:/Users/.../...`.

---

## Ringkasan Tingkat Atas

```
chloros-cli [global options] COMMAND [command options]
```

### Opsi Global

| Bendera | Deskripsi |
| --- | --- |
| `--backend-exe PATH` | Mengganti eksekutabel backend yang terdeteksi secara otomatis. |
| `--port N` | Port backend HTTP (default: `5000`). |
| `-v, --verbose` | Mengaktifkan keluaran terperinci. |
| `--restart` | Memaksa backend untuk memulai ulang (menghentikan semua proses `backend_server.py` yang sedang berjalan). |
| `--version` | Menampilkan versi (`Chloros CLI 1.2.0`). |
| `--help` | Menampilkan bantuan tingkat atas. |

### Indeks Perintah

| Perintah | Tujuan |
| --- | --- |
| [`process`](#chloros-cli-process) | Memproses folder yang berisi tangkapan Survey3 atau LATTICE secara end-to-end. |
| [`login`](#chloros-cli-login) | Melakukan otentikasi mesin ini dengan akun Chloros+. |
| [`logout`](#chloros-cli-logout) | Hapus kredensial yang tersimpan dalam cache. |
| [`status`](#chloros-cli-status) | Tampilkan status lisensi / otentikasi saat ini. |
| [`export-status`](#chloros-cli-export-status) | Tampilkan kemajuan ekspor Thread-4 secara langsung selama proses `process` berjalan. |
| [`language`](#chloros-cli-language) | Mengatur atau menampilkan daftar bahasa tampilan CLI (didukung 38 bahasa). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Folder proyek default (digunakan bersama dengan GUI). |
| [`update`](#chloros-cli-update) | Memeriksa dan menginstal pembaruan CLI (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostik sistem + uji fungsional. |
| [`time-sync`](#chloros-cli-time-sync) | Status / kontrol grandmaster PTP. |
| [`lattice`](#chloros-cli-lattice) | Kontrol &amp; pengambilan gambar kamera LATTICE (45+ subperintah). |
| [`daq`](#chloros-cli-daq) | Kontrol sensor spektral DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Membuka dan menjalankan proyek yang disimpan di Chloros (kamera + DAQ). |

---

## Instalasi

`chloros-cli` disertakan dalam penginstal desktop Chloros di setiap platform yang didukung — tidak ada unduhan terpisah CLI. Menginstal paket platform akan menambahkan `chloros-cli` ke `PATH` Anda bersama dengan aplikasi desktop dan yang digunakannya.

Unduhan terbaru: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Penginstal ini juga menyertakan skrip peluncur praktis (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) yang membuka shell CLI yang siap digunakan; skrip-skrip ini dijelaskan dalam [CLI Panduan Pengguna](../CLI.md) dan tidak diulangi di sini.

### Windows (.exe)

1. Unduh penginstal Windows dari halaman unduhan.
2. Jalankan `Chloros-Setup-x.y.z.exe` dan ikuti panduan penginstalan. Jalur instalasi default adalah `C:\Program Files\Chloros\` (CLI berada di `C:\Program Files\Chloros\cli\`, yang ditambahkan oleh penginstal ke PATH).
3. Buka terminal baru (`cmd.exe`, PowerShell, atau Terminal Windows) agar `PATH` yang telah diperbarui dapat terdeteksi.

```powershell
chloros-cli --version
```

Penginstal secara otomatis menambahkan `chloros-cli.exe` ke PATH sistem Anda dan menyertakan runtime Arena SDK yang diperlukan untuk kamera LATTICE.

### Linux amd64 (.deb)

Untuk Ubuntu 22.04 LTS atau yang lebih baru / workstation berbasis Debian x86_64.

> **Ubuntu 20.04 tidak didukung.** Daftar dependensi paket ini berasal dari
> apa yang sebenarnya ditautkan oleh backend, dan itu mencakup `libc6 (>= 2.34)`;
> Focal menyertakan glibc 2.31. `apt` menolak instalasi daripada membiarkannya gagal saat
> runtime.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Paket .deb ini menginstal:
- `chloros-cli` hingga `/usr/bin/chloros-cli`
- Backend yang telah dikompilasi menjadi `/usr/lib/chloros/chloros-backend`
- Runtime Arena SDK (untuk kamera LATTICE)
- Model denoiser, paket kalibrasi, dan konfigurasi saluran pembaruan

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Tata letak yang sama dengan berkas .deb amd64, dengan build CUDA yang disesuaikan untuk Jetson Orin / Orin NX / Orin Nano.

### Otentikasi Sekali Per Perangkat

Setiap platform memerlukan login satu kali di Chloros+ sebelum panggilan SDK / CLI dapat berfungsi:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kredensial disimpan dalam cache di `~/.chloros/user_session.json`.

### Verifikasi Instalasi

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Diperlukan langganan Chloros+.**CLI memerlukan paket Chloros+ yang aktif.**Copper**adalah tingkatan awal Chloros+ — setiap tingkatan berbayar Chloros+ memiliki akses ke CLI / SDK; hanya tingkatan gratis**Iron** yang tidak memilikinya. (Peta ID paket: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Lakukan peningkatan di [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Batas ini diberlakukan oleh backend, bukan hanya oleh CLI: permintaan yang diberi bendera SDK / CLI tanpa paket berbayar akan ditolak dengan kode `403 PLAN_UPGRADE_REQUIRED`, baik berasal dari `chloros-cli`, Python SDK, maupun klien HTTP yang dibuat sendiri. Panggilanakan menerima kode kesalahan `401 AUTH_REQUIRED`. Akses tetap berfungsi secara offline selama masa tenggang paket (30 hari untuk paket bulanan, hingga masa berlaku habis untuk paket tahunan) dan berhenti saat masa tenggang berakhir; kode kesalahan `chloros-cli status` tetap muncul sehingga alasannya terlihat (ini adalah rute SDK / CLI yang dikecualikan dari pembatasan tingkatan — `GET /api/license-status`).

---

## Panduan Cepat 5 Menit

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Memproses folder gambar melalui alur kerja lengkap Chloros (deteksi target → kalibrasi → vignette → reflektansi → ekspor indeks).

### Ringkasan

```
chloros-cli process INPUT [OPTIONS]
```

### Argumen Posisi

| Argumen | Deskripsi |
| --- | --- |
| `INPUT` | Jalur ke folder input yang berisi berkas `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE), atau `.dng`. |

### Opsi Umum

| Opsi | Default | Deskripsi |
| --- | --- | --- |
| `-o, --output PATH` | folder baru dengan cap waktu di bawah jalur proyek default Anda (`~/Chloros Projects` kecuali jika dikonfigurasi) | Folder proyek yang akan dibuat atau digunakan kembali. Jika folder tersebut sudah berisi berkas `project.json`, maka berkas `_1`/`_2` yang baru akan dibuat sebagai folder saudara, bukan menimpa yang sudah ada. |
| `-n, --project-name NAME` | otomatis (cap waktu) | Nama proyek. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` menggunakan Chloros+ neural debayer; lebih lambat tetapi kualitasnya lebih tinggi. |
| `--vignette / --no-vignette` | `--vignette` | Koreksi vignette. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibrasi reflektansi (menggunakan target panel jika ditemukan, NIST per-kalibrasi serial untuk LATTICE). Untuk LATTICE multispektral, ini juga berfungsi sebagai tombol pengalih **produk** reflektansi — lihat [Tombol Pengalih Ekspor Per-Produk](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | mati | Menerapkan koreksi PPK GNSS dari berkas sidecar. |
| `--exposure-pin-1 MODEL` | mati | Menetapkan &quot;pin-1&quot; pada rig kamera ganda Survey3 . |
| `--exposure-pin-2 MODEL` | off | Menetapkan model &quot;pin-2&quot;. |
| `--recal-interval SECONDS` | 0 | Memaksa perhitungan kalibrasi dijalankan ulang setiap N detik selama waktu perekaman. |
| `--timezone-offset HOURS` | lokal | Mengganti selisih zona waktu yang tertanam dalam metadata keluaran. |
| `--format FORMAT` | `TIFF (16-bit)` | Salah satu dari `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | tidak ada | Indeks vegetasi (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Memaksa titik masuk pipa untuk berkas LATTICE TIFF (Survey3 .raw tidak terpengaruh). Juga jalan keluar yang memungkinkan penangkapan **tanpa raw** diproses sama sekali — lihat [Bagaimana tampilan folder penangkapan](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | aktif | Menghasilkan produk debayering linier (`Debayered_Images`). Lihat [Tombol Pengaturan Ekspor Per Produk](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | aktif | Menghasilkan pratinjau tampilan (`Preview_Images`): RGB = white balance (sumber cahaya DAQ jika tersedia, jika tidak maka gray-world) + gamma; multispec = peregangan warna palsu. |
| `--radiance / --no-radiance` | aktif | Mengeluarkan radian float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referensi untuk produk reflektansi LATTICE: `auto` = target dalam bingkai yang lulus QA adalah referensi absolut, DAQ-downwelling (ρ = π·L/E) sebagai cadangan; `target` = ketat (tanpa substitusi DAQ); `daq` = otoritatif DAQ. Lihat [Tombol Pengaturan Ekspor Per Produk](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | tidak ada | Direktori pemindaian reflektansi target **yang diukur** per unit (`<serial>.csv`); jika tidak ditemukan, akan menggunakan spektrum T3/T4P nominal sebagai pengganti. |
| `--array-alignment / --no-array-alignment` | aktif | Susunan LATTICE: menerapkan penyelarasan antar-modul yang tercantum dalam file XMP setiap pengambilan gambar (`Chloros:Alignment*`) ke setiap produk yang diproses (debayered / pratinjau / radiance / reflektansi / indeks). Tidak melakukan apa-apa untuk gambar tanpa tag tersebut. |
| `--array-alignment-crop / --no-array-alignment-crop` | crop | Memotong ekspor yang telah diselaraskan ke wilayah tumpang tindih bersama array sehingga semua modul berbagi satu jejak; `--no-…` mempertahankan kanvas sensor penuh (diisi hitam di luar sumber). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Resampling untuk distorsi penyelarasan. `nearest` mempertahankan nilai DN sumber yang tepat (tanpa pencampuran nilai radiometrik antar-piksel). |

### Opsi Deteksi Target

| Bendera | Deskripsi |
| --- | --- |
| `--min-target-size PIXELS` | Ukuran panel-target minimum (px) untuk detektor. |
| `--target-clustering 0-100` | Sensitivitas pengelompokan. |
| `--target / --targets` | Perlakukan folder masukan sebagai panel-target saja (lewati deteksi survei). |

### Contoh

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Tombol Pengaturan Ekspor Per Produk (LATTICE multispektral)

Pemrosesan LATTICE menghasilkan **setiap produk yang berlaku dalam satu kali proses**. Keempat tombol pengaturan per-tipe — `--debayered`, `--preview`, `--radiance`, `--reflectance` — semuanya**diaktifkan secara default**; gunakan formulir `--no-<type>` untuk menonaktifkan salah satunya. Kamera utama RGB hanya pernah menghasilkan data yang telah di-debayering + pratinjau (tidak ada radian/reflektansi per-pita), sehingga `--radiance`/`--reflectance` tidak berfungsi untuk kamera tersebut. Tombol pengalih diabaikan untuk Survey3 `.raw` (yang mengikuti jalur reflektansi/target standar). *(Bendera `--radiometric-output {reflectance,radiance,sensor-response}` yang lama telah **dihapus** dan digantikan oleh tombol-tombol ini; tidak ada lagi tingkat `sensor-response`.)*

| Produk | Keluaran | Perlu DAQ downwelling? |
| --- | --- | --- |
| `--debayered` | Demosaik linier (`Debayered_Images`). | Tidak. |
| `--preview` | Pratinjau tampilan (`Preview_Images`): RGB = WB + gamma; multispec = peregangan warna palsu. | Tidak. |
| `--radiance` | float32 W/m²/sr/nm dari rantai radiometrik lengkap (`Radiance_Images`). | No. |
| `--reflectance` | uint16 reflektansi ρ (`32768` = 1,0), siap untuk Pix4D. | **Ya**, kecuali jika ditambatkan oleh target dalam bingkai yang lulus QA (lihat di bawah). |

`--reflectance-source` memilih referensi reflektansi:**`auto`**(default) menjadikan target dalam bingkai yang lulus QA sebagai**referensi absolut**— rantai garis empiris yang diikat target dievaluasi silang pada panel yang disisihkan dan pemenang yang terukur diterapkan — menggunakan pembagian downwelling DAQ (ρ = π·L/E) sebagai cadangan jika tidak ada target atau QA gagal;**`target`**bersifat ketat (tanpa substitusi DAQ);**`daq`**memilih perilaku yang mengutamakan DAQ. Geometri target (ArUco / ROI tetap / strip) berasal dari konfigurasi target proyek; `--target-reflectance-dir DIR` menyimpan per-unit pemindaian**yang diukur** (`<serial>.csv`) yang dicari berdasarkan nomor seri/QR unit target, dengan spektrum T3/T4P nominal sebagai cadangan.

Jalur reflektansi DAQ secara otomatis menentukan ****pancaran ke bawah yang sesuai dengan cap waktu**secara otomatis dari**`.daq`**(DAQ-U/M/E) yang terekam**atau `.csv` asli DAQ-M**yang ditemukan bersama citra. Jikakamera atau paket kalibrasi DAQ tidak disimpan dalam cache lokal, alur kerja ini**secara otomatis mengambilnya dari AWS** pada penggunaan pertama (memerlukan koneksi internet sekali; disimpan dalam cache di bawah `~/.chloros/`).

#### Membaca piksel reflektansi (Pix4D / Metashape / skrip Anda sendiri)

Reflektansi disimpan sebagai DN bilangan bulat, dan **nilai DN yang berarti ρ = 1,0 bergantung pada kamera sumber**:

| Sumber | ρ = 1,0 adalah | Cara mengetahuinya |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ruang kosong hingga ρ 2,0) | Tanda XMP `Chloros:PixelScale=32768` tercantum pada file. |
| Survey3 | `65535` (dipotong pada ρ 1,0) | Tidak ada tag XMP `Chloros:*` — ketiadaan itu *adalah* sinyalnya. |

**Baca nilai `Chloros:PixelScale` dan bagi dengan nilainya** daripada mengasumsikan nilai konstan. Tag tersebut didefinisikan dalam domain uint16, sehingga nilainya tetap `32768` di seluruh format keluaran yang melakukan penskalaan ulang — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)`, dan `TIFF (32-bit, Percent)` semuanya bersifat self-describing (menormalisasi tipe data yang disimpan kembali ke uint16 terlebih dahulu: ×257 dari 8-bit, ×65535 dari float).

> **Satu kasus tidak memiliki skala, sesuai desain.** Ketika tangkapan sumber 8-bit (BayerRG8) ditulis sebagai TIFF 8-bit, pipa pemrosesan akan *memotong* nilainya menjadi 0..255 alih-alih melakukan penskalaan ulang, sehingga setiap nilai di atas ρ≈0,008 akan diratakan menjadi 255 dan tidak ada skala yang menggambarkan file tersebut. Chloros dengan sengaja mengabaikan pasangan `Chloros:PixelScale` dan `MicaSense:RadiometricCalibration` di sana, serta mencatat alasannya. **Jika tag tersebut tidak ada pada berkas reflektansi LATTICE, jangan mengasumsikan adanya skala — ekspor ulang pada 16-bit atau 32-bit** daripada membagi piksel yang sebenarnya tidak dapat dibagi.

#### Data EXIF diteruskan ke hasil ekspor

`process` menyalin **blok GPS dan ExifIFD** dari tangkapan sumber ke setiap produk, sehingga
ekspor tersebut membawa `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal`, dan
`CameraSerialNumber` bersama dengan georeferensi.

**`FocalLength` bukanlah hal opsional untuk fotogrametri.** Pix4D menghitung jarak sampel tanah berdasarkan
panjang fokus ditambah ketinggian; jika tag tersebut tidak ada, program akan menggunakan skala yang sangat salah. Pada satu
penerbangan di kebun jeruk dengan 49 pengambilan gambar, tag yang hilang mengubah lokasi berukuran 411 m × 160 m menjadi
lokasi yang direkonstruksi berukuran 47,8 km × 13 km — ortofoto 455 MP yang sebagian besar berisi data kosong, yang kemudian dianggap sebagai masalah penataan ubin dan
masalah BigTIFF sebelum ada yang memeriksa GSD. Jika ortofoto Anda menghasilkan skala yang tidak masuk akal,
jalankan `exiftool -FocalLength` pada produk yang diekspor terlebih dahulu.

Salinan ini sengaja **tidak** `-all:all`: Tag struktural IFD0 merusak keluaran LATTICE saat
disalin, dan `ExifImageWidth` / `ExifImageHeight` dikecualikan karena menggambarkan
*sumber* penangkapan — ekspor yang pernah diubah ukurannya akan membawa dimensi
yang bertentangan dengan rasternya sendiri. XMP ditulis secara langsung, bukan disalin, karena ExifTool
membuang tag XMP dari pemanggilan yang sama saat blok XMP disalin (yang akan menghilangkan tag kalibrasi MAPIR
).

### Lokasi hasil keluaran

Produk ditulis **di bawah folder proyek, dikelompokkan berdasarkan kamera dan kemudian berdasarkan format file**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folder kamera adalah `LATT-<sensor>-<lens>-F<filter>` untuk LATTICE (sesuai dengan EXIF
`Model` pada pengambilan gambar) dan `<model>_<filter>` untuk Survey3 — dua kamera yang berbagi sensor dan filter tetapi berbeda
dalam lensa tetap memiliki pohon folder terpisah, karena vignette, bidang pandang, dan distorsi berbeda. Format
folder mengikuti `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, atau `tiff32` untuk
`TIFF (32-bit, Percent)`X.

> **Setiap produk yang diekspor tetap menggunakan nama berkas SUMBER.** Ekspor radiance dari
> `capture_…_raw.tif` tetap bernama `capture_…_raw.tif` — hanya saja berada di dalam
> `tiff32/Radiance_Images/`. **Folderlah yang mengidentifikasi produk, bukan nama file**, jadi pencarian
> dengan pola `*radiance*.tif` tidak akan menemukan apa pun; gunakan pencocokan berdasarkan direktori sebagai gantinya.

### Rekaman sensor cahaya — yang telah dikalibrasi `.daq` + `.csv`

`process` juga menangani rekaman `.daq` di folder input Anda, dan hal ini **tidak**
memerlukan gambar apa pun untuk melakukannya: DAQ-U / DAQ-M / DAQ-E yang diterbangkan sendiri sudah merupakan
perekaman yang lengkap, dan folder yang hanya berisi berkas `.daq` merupakan masukan yang valid.

Sebuah DAQ dapat direkam **tanpa** kalibrasinya — itulah yang dilakukan oleh perekam umum
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) secara default: mereka menulis hitungan sensor mentah dan menandai berkas tersebut sehingga
Chloros mengambil kalibrasi pabrik sensor tersebut **melalui nomor seri** (dari cache lokal terlebih dahulu,
kemudian MAPIR Cloud) dan menerapkannya. `process` menulis hasilnya kembali:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` memuat satu baris per pembacaan: stempel waktu UTC, waktu integrasi, daya total,
lux fotopik/skotopik, PPFD (dan pembagian biru/hijau/merah), panjang gelombang puncak, lalu
spektrum penuh pada kisi panjang gelombang sensor itu sendiri. `.daq` mengimpor ulang tanpa perlu
dikalibrasi untuk kedua kalinya.

Jika berhasil, proses tersebut melaporkan `Light-sensor products written: N (calibrated .daq + .csv)`.
Teks dalam tanda kurung menjelaskan apa yang sebenarnya ditulis, sehingga berbunyi
`(RAW COUNTS — this sensor has no calibration bundle)` untuk sensor tanpa bundel dan
`(N calibrated, M raw counts)` untuk folder yang berisi keduanya. Judul
`[DAQ-EXPORT]` dan `[RUN-SUMMARY]` yang dimiliki backend tersebut juga menggunakan susunan kata yang sama — tidak ada satu pun
dari ketiganya yang dapat menyebut ekspor mentah sebagai data yang telah dikalibrasi.

Perekaman DAQ-U / DAQ-M / DAQ-E yang bundel kalibrasinya tidak dapat diambil — Anda
sedang offline, atau sensor tersebut tidak memiliki file kalibrasi — akan **dilewati dengan alasan** pada
baris `[DAQ-EXPORT]`, dan tidak pernah disimpan sebagai berkas “telah dikalibrasi” yang berisi hitungan mentah.
Hubungkan ke internet dan jalankan kembali. Alasan yang dimaksud adalah alasan yang sebenarnya
ditentukan oleh pembaca untuk berkas tersebut (skema tidak terbaca, tidak ada paket, kesalahan penulisan), dan ringkasan
proses mencantumkan alasan-alasan **yang berbeda** — dua puluh berkas dilewati karena satu penyebab yang dibaca sebagai satu
penyebab, bukan dua puluh pengulangan dari penyebab tersebut.

#### Rekaman DAQ-A diekspor sebagai hitungan mentah

Keluarga **DAQ-A** muncul sebelum sistem bundel per-serial dan tidak memiliki bundel kalibrasi
yang perlu diambil — perangkat ini justru dikalibrasi di lapangan menggunakan target reflektansi, itulah
sebabnya perangkat ini tidak pernah membutuhkannya. Penolakan terhadap rekaman-rekaman tersebut membuat mereka tidak memiliki cara untuk mendapatkan
angka-angka tersebut sama sekali, sehingga mereka mengekspornya dengan **nama yang berbeda**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Nama file yang berbeda, bukan bendera di dalam file, karena klaim tersebut harus tetap utuh
saat dikirim melalui email sebagai nama mentah. Header `.csv` menyatakan
`raw spectral sensor counts (NOT irradiance)` dan memperingatkan bahwa nilai-nilai tersebut dapat dibandingkan
**di dalam** file — yang persis seperti tujuan penggunaannya dalam kalibrasi berbasis target — dan
bukan antar sensor. Kolom fotometrik yang bergantung pada daya (daya total, lux fotopik dan
skotopik, PPFD) ditulis **NULL** alih-alih diintegrasikan dari hitungan, dan ringkasan
menyebutkan `RAW COUNTS` sehingga data yang &quot;diekspor&quot; dalam log tidak dapat dibaca sebagai irradiansi.

Perekaman **v1.01 / v1.02** versi lama (ditulis oleh DAQ-A-SD) tidak memiliki epoch per pembacaan,
hanya waktu penulisan berkas. Alat pencocok gambar↔sinar turun masih menolaknya — mencocokkan
bingkai dengan waktu penulisan akan menghasilkan kesalahan yang tidak terlihat — tetapi eksportir membacanya, dan
CSV mencetak `clock=daq_created_on` sehingga hasilnya menunjukkan jam mana yang digunakan.

### Catatan

- `process` secara otomatis mendeteksi apakah folder Anda berformat Survey3, LATTICE, atau campuran.
- Kemajuan ditampilkan melalui Server-Sent Events; CLI menampilkan kemajuan real-time per-thread (Mendeteksi, Menganalisis, Memproses, Mengekspor).
- Untuk Linux /Jetson, CLI memeriksa swap dan mungkin memberikan peringatan sebelum memproses folder berukuran besar. Debayer yang peka tekstur juga secara otomatis menerapkan batasan frekuensi GPU pada Jetson berdaya rendah (Nano, Orin Nano).
- Jika berhasil, proses pelaporan akan menunjukkan berapa banyak produk gambar yang ditulis (`Image products written: N`).

#### Proses yang tidak menulis gambar akan gagal

Jika Anda meminta produk dan proses tersebut tidak menulis **sama sekali** — hanya `project.json` dan
`calibration_data.json` — `process` menganggapnya sebagai kegagalan: ia mencetak
`Processing finished but wrote no image products.` dan **keluar dengan status non-nol**, sehingga skrip dapat
mendeteksinya. Pesan tersebut menyebutkan nama folder proyek dan penyebab yang umum:

- folder input tidak dikenali sebagai hasil pengambilan (periksa tata letak dan `--input-level`), atau
- setiap produk yang diminta dilewati karena tidak berlaku untuk kamera-kamera tersebut (misalnya, meminta
  radian/reflektansi dari kamera yang hanya mendukung mode &quot;RGB&quot;).

Jalankan kembali dengan `--verbose` dan periksa log backend untuk baris `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
yang menjelaskan alasan pengabaian per kamera yang tidak tercantum dalam keluaran CLI.

Eksekusi yang disengaja hanya menggunakan metadata — semua produk dinonaktifkan dan tanpa `--indices` — tetap menghasilkan
**berhasil**, karena keluaran gambar kosong adalah hasil yang benar dalam kasus ini.

Demikian pula dengan **proses hanya sensor cahaya**:** sebuah folder rekaman `.daq` tidak memiliki citra yang dapat diekspor
secara definisi, dan proses tersebut dinilai berdasarkan data kalibrasi `.daq` / `.csv` yang dihasilkan sebagai gantinya.

---

## `chloros-cli login`

Otentikasi mesin ini dengan akun cloud Chloros+. Kredensial disimpan dengan aman di `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Contoh

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (menghapus atau menduplikasi bagian dari kata sandi). Saat terjadi kesalahan 401, CLI secara otomatis mencoba ulang dengan menambahkan kembali `$$`, lalu dengan setengah bagian kata sandi yang telah dihilangkan duplikatnya; jika percobaan ulang berhasil, sistem akan masuk dan menampilkan sintaks tanda kutip tunggal yang benar untuk digunakan berikutnya.

> **Penggunaan tanpa antarmuka pengguna/penggunaan skrip: tidak adanya sesi yang disimpan dalam cache berarti munculnya prompt interaktif, bukan kegagalan instan.** Setiap perintah yang memunculkan backend (`process`, `status`, `export-status`, `time-sync`, …) yang dijalankan tanpa lisensi/sesi yang disimpan dalam cache akan masuk ke prompt interaktif `Email:` / `Password:` pada stdin sebelum melanjutkan. Oleh karena itu, pekerjaan tanpa pengawasan tanpa sesi yang disimpan dalam cache akan macet menunggu masukan — jalankan `chloros-cli login EMAIL PASSWORD` sekali per mesin sebelum menjadwalkan pekerjaan tanpa antarmuka pengguna.

---

## `chloros-cli logout`

Menghapus sesi yang tersimpan dalam cache dan memaksa login baru pada panggilan berikutnya.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Menampilkan tingkatan lisensi saat ini (Iron/Copper/Bronze/Silver/Gold), pengguna yang terotentikasi, dan jumlah pengikatan perangkat.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Periksa kemajuan ekspor Thread-4 secara real-time. Aman untuk dipanggil **selama** proses `process` berjalan dari shell lain.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Atur bahasa tampilan &quot;CLI&quot; (didukung 38 bahasa, termasuk CJK, RTL, dan Indic). Akan beralih secara mulus ke bahasa Inggris pada konsol lama yang tidak dapat menampilkan skrip tersebut.

```
chloros-cli language [LANG_CODE] [--list]
```

### Contoh

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Perintah Folder Proyek

Perintah ini mengelola lokasi folder proyek default (digunakan bersama dengan antarmuka pengguna grafis).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Hanya untuk Jetson. Memeriksa `version_url` dari `/etc/chloros/update.conf` dan menawarkan untuk mengunduh + menginstal `.deb` yang sesuai.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Pada Linux /Jetson, CLI juga menjalankan **pemeriksaan pembaruan otomatis pada setiap kali startup** (tidak memblokir, tidak pernah menunda perintah): skrip ini membaca `/etc/chloros/update.conf`, menyimpan hasilnya dalam cache selama 1 jam di `~/.chloros/update_cache.json`, dan menampilkan `Update available: vX.Y.Z / Run: chloros-cli update` jika terdapat versi yang lebih baru. Dilewati tanpa pemberitahuan jika terjadi kesalahan apa pun dan saat menemuWindows.

---

## `chloros-cli selftest`

Menjalankan uji coba 7 langkah: versi, ketersediaan port, pengaktifan backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), keberadaan model denoiser, kesiapan CUDA+denoiser.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Status dan kontrol grandmaster PTP. Host Chloros menjalankan grandmaster PTP; unit LATTICE cams dan DAQ-E bertindak sebagai slave untuk cap waktu lintas perangkat.

| Subperintah | Deskripsi |
| --- | --- |
| `status` | Tampilkan status grandmaster, prioritas BMCA, identitas jam. |
| `peers` | Menampilkan daftar slave yang terdeteksi melalui Delay_Req (kamera + sensor DAQ-E). |
| `cameras` | Perkesehatan PTP per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Mulai ulang proses grandmaster. |
| `set-priority --priority1 N --priority2 N` | Mengabaikan prioritas BMCA. |

### Contoh

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Kontrol kamera LATTICE. Setiap subperintah dialihkan melalui backend Chloros; backend tersebut mengelola kumpulan kamera (cam pool) sehingga panggilan CLI berikutnya akan menggunakan kembali handle yang sama.

### Opsi Umum (digunakan oleh sebagian besar subperintah)

| Bendera | Deskripsi |
| --- | --- |
| `-d, --device N` | Indeks kamera (default: 0). |
| `-s, --serial SN` | Nomor seri spesifik; menggantikan `--device`. |
| `--serials SN1,SN2,…` | Nomor seri yang dipisahkan dengan koma untuk operasi multi-kamera. |
| `--all` | Beroperasi pada setiap kamera yang terdeteksi. |
| `--exposure US` | Waktu pencahayaan dalam mikrodetik. |
| `--gain DB` | Penguatan dalam dB. |
| `--pixel-format FMT` | misalnya: `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimensi gambar. |
| `--preset {default,high_quality,high_speed,triggered}` | Menerapkan preset pengaturan. Semua dalam mode bebas kecuali `triggered`, yang mengaktifkan kamera untuk mendeteksi tepi perangkat keras pada Baris 2 — tanpa adanya sinyal yang mengaktifkan jalur tersebut, kamera akan menunggu selamanya daripada merekam. |
| `-o, --output DIR` | Direktori keluaran (default: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Ukuran paket GVSP. `auto` menjalankan probe ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Alur Kerja Koneksi Pertama Kamera LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referensi Subperintah

#### Penemuan &amp; Informasi

| Subperintah | Tujuan |
| --- | --- |
| `lattice info` | Menampilkan daftar kamera yang terhubung (pabrikan, model, nomor seri, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Menganalisis sistem host untuk konfigurasi kamera yang optimal. `--no-discover` melewati proses penemuan kamera (lebih cepat, analisis hanya menggunakan NIC). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Memeriksa/perbaiki pengaturan NIC; perkirakan bandwidth/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Kemampuan jaringan backend skema stabil + rekomendasi array (mengembalikan `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` mempertahankan resolusi yang diminta tetapi membatasi fps target — baca `recommended.recommended_target_fps` dan gunakan sebagai target koneksi; anggap sebagai keberhasilan, bukan kesalahan. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Analisis skenario tanpa membuka kamera. **`--n-active` adalah jumlah total kamera yang terhubung, bukan hanya yang ada di array ini**— lemparkan pengecualian ini ketika kamera mandiri melakukan streaming secara bersamaan, atau anggaran jaringan dihitung berdasarkan permintaan yang tidak memperhitungkan jumlah kamera secara penuh (default: `len(--models)`). Selalu mencetak nilai agregat `Wire budget:` (MB/s yang diminta vs batas atas yang aman dari tabrakan) dan baris `Max cameras:`, serta menandai `** OVER-SUBSCRIBED**` ketika array menggunakan bandwidth jaringan melebihi kapasitas — lihat [Model fps &amp; burst array](#array-fps--burst-model). |
| `lattice gpu` | Tampilkan status GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Periksa atau perbarui firmware kamera. Pilihan lokal `.fwa` terkunci: file di `firmware/<MODEL_PREFIX>/` yang sesuai dengan build&#x27;`MIN_FIRMWARE_VERSION` di-flash jika ada (hanya versi tertinggi sebagai cadangan), sehingga gambar vendor yang lebih baru yang disimpan di disk tidak aktif sampai pin tersebut di-bump — rilis yang lebih baru secara sengaja mencapai unit melalui manifest AWS yang ditandatangani, yang lebih disukai jika lebih baru. |
| `lattice presets [--apply NAME]` | Menampilkan daftar atau menerapkan preset kamera. |
| `lattice status` | Status kamera langsung. |

#### Pengambilan Gambar

| Subperintah | Tujuan |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Satu bingkai. **Menyimpan semua jenis ekspor secara default** (`--processing all`); lihat [Tingkat Ekspor Perekaman](#capture-export-levels-the-all-default). `--levels` menyimpan subset eksplisit (menggantikan `--processing`); `--force-daq` menulis pembacaan DAQ yang ditugaskan sebagai file sidecar `.daq` sidecar bahkan pada pengambilan data mentah saja. `--jpeg-quality` = JPEG kualitas 1–100 (default 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Mengalirkan data ke disk hingga Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Pratinjau MJPEG langsung berbasis browser. `--ae-damping` mengatur peredaman eksposur otomatis (0,4–100). |

#### Penyesuaian Sensor

| Subperintah | Tujuan |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Membaca/menulis node GenICam apa pun. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Eksposur &amp; AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Gain &amp; auto-g. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | ROI sensor &amp; binning. |
| `lattice format [--set FMT] [--list]` | Format piksel. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Pemicu perangkat keras/perangkat lunak. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (tanpa bendera = WB sekali jepret) | Operasi WBRGB /Hanya untuk kamera Bayer; tidak beroperasi (dilewati) pada M3M mono. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | Jalur warna tampilan &quot;RGB&quot;. `natural` (default) adalah penyempurnaan langsung yang hemat; `enhanced` menambahkan penghilang fringing + vibrance + kontras lokal CLAHE untuk tampilan hub-parity penuh dengan biaya penyempurnaan per-bingkai sekitar 2× lipat, sehingga menghasilkan laju bingkai **langsung** yang lebih rendah — tangkapan yang disimpan selalu mendapatkan penyempurnaan penuh dalam kedua kasus. RGB /Hanya untuk kamera Bayer; dilewati pada M3M mono. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Menampilkan saturasi/kontras (kamera dengan filter RGB). Dilewati pada M3M mono. |
| `lattice filter [--set NAME] [--list]` | Atur model filter kamera (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Periksa node daya/termal; aktifkan/nonaktifkan mode idle berdaya rendah. |

#### Kalibrasi &amp; Sensor

| Subperintah | Tujuan |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrasi menggunakan target reflektansi. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Perintah sensor cahaya turun bawaan. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Menerapkan koreksi vignette pada gambar yang ada. |

#### Multi-Kamera (Sesi Transien)

| Subperintah | Tujuan |
| --- | --- |
| `lattice multi-info` | Menampilkan daftar semua kamera dengan peran sinkronisasi. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Satu bingkai yang disinkronkan dari setiap kamera. Menyimpan **semua jenis ekspor secara default**saat array persisten terhubung; fallback sementara tanpa array**hanya di-debayered** (jalankan `array-connect` terlebih dahulu untuk sisanya). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Streaming bingkai yang disinkronkan (sementara). |
| `lattice multi-test [--count N]` | Uji waktu sinkronisasi GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Deteksi otomatis pengkabelan master/slave GPIO. |

#### Penyelarasan

| Subperintah | Tujuan |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — ditambah pengaturan detektor/penyandingan `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, pengaturan RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinasi multi-frame `[--averaging mean\|median\|inlier_weighted]`, batasan geometris `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, pembatasan spasial `[--roi X0,Y0,X1,Y1] [--mask PATH]`, dan penggantian per-slave `[--per-cam-override SN:KEY=VALUE]` (dapat diulang) | Hitung profil penyelarasan dari kamera langsung. `--prefilter` secara default diatur ke `gradient` (peta tepi; sesuai dengan penyelarasan GUI/array — tepi tetap ada di seluruh pita spektral). `--matcher flann` memberikan hasil yang baik di atas ~5000 fitur; `--averaging median` tahan terhadap satu pengambilan gambar yang buruk, `inlier_weighted` memberi bobot berdasarkan jumlah kecocokan; `--lock-scale` memproyeksikan ke rotasi terdekat (tanpa skala), `--lock-axis` men-nol-kan satu komponen translasi; `--mask` berlaku untuk setiap kamera (gunakan `--per-cam-override` untuk pengaturan per kamera, misalnya `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` menolak menyimpan kalibrasi yang RMS reprojeksinya melebihi batas. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Mengambil satu bingkai multi-band yang telah diselaraskan. `--bit-depth` secara default menyesuaikan dengan kamera; `--no-crop` mempertahankan bingkai penuh (diisi dengan warna hitam); `--interpolation` (default `linear`) dan `--border-mode`/`--border-value` (default `constant`/0) mengontrol warp CPU — jalur GPU tetap bilinear terlepas dari pengaturan tersebut. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Bingkai multi-band yang selaras dengan aliran (pengaturan warp yang sama dengan `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Tampilkan detail profil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Ubah urutan lapisan. |

#### Indeks / Perhitungan Vegetasi

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Kumpulan bendera lengkap: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (dapat diulang), `--capture-level raw|debayered|radiance|reflectance|unknown` (mengganti tingkat perekaman yang tercatat di sumber TIFF; default: dibaca dari metadata TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Dengan `--live`, kenop distorsi penyelarasan juga berlaku: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **`--channel` peka terhadap huruf besar-kecil.** Sisi simbol harus sama persis dengan nama saluran preset (preset menggunakan huruf kecil, misalnya NDVI = `red`,`nir` — periksa `--list-presets`), dan bagian band harus sesuai dengan nama band dalam tumpukan yang disejajarkan (atau berupa indeks band berbasis 0 dalam mode offline). `--channel red=Red_660 --channel nir=NIR_850` berfungsi; `--channel RED=660` gagal dengan kesalahan `channel_map missing entries`.

#### Koneksi Persisten (Smart-Prep, Alur Setara GUI)

Perintah-perintah ini menjaga kamera tetap terbuka di kolam belakang (backend pool) di seluruh pemanggilan CLI.

| Subperintah | Tujuan |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Menambahkan satu kamera ke kolam (kamera tunggal, tanpa array). |
| `lattice cam-disconnect [--serial SN] [--all]` | Melepaskan. |
| `lattice cam-list` | Menampilkan daftar kamera di pool. |
| **`lattice array-connect`**|**Menghubungkan array sinkronisasi persisten (titik masuk yang direkomendasikan).** Menjalankan alur persiapan cerdas (smart-prep) GUI secara lengkap. |
| `lattice array-disconnect [--array-id ID] [--all]` | Lepaskan sebuah array. |
| `lattice array-list` | Menampilkan daftar array yang terhubung. |
| `lattice array-status [--array-id ID]` | Data langsung f, PTP, kesalahan terakhir. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Satu penangkapan yang disinkronkan dari array langsung — Tunggal / Berkelanjutan / Interval / Tercepat. **Secara default menggunakan `all`** (satu file per jenis ekspor yang berlaku per kamera). Kamera yang dilewati (misalnya, RGB dikecualikan dari radian/reflektansi) dilaporkan dengan `Skipped: SN:<serial> (<reason>)`; pembacaan DAQ yang digunakan untuk reflektansi disimpan bersamanya dan dilaporkan dengan `DAQ: <path>`. Lihat [Mode Pengambilan, Perekam, &amp; Pemrosesan Ulang Offline](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Merekam tampilan indeks gabungan langsung ke video/GIF (kualitas pemantauan; memerlukan aliran gabungan yang terbuka). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Burst Bayer mentah dengan fps tinggi (tingkat analisis; diproses ulang secara offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Memproses ulang rentetan gambar mentah yang telah disimpan menjadi video yang telah dikalibrasi. |

##### Opsi `array-connect`

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--serials SN1,SN2,…` | temukan secara otomatis semua kamera LATTICE (diperlukan ≥2) | Kamera dengan nomor seri pertama adalah MASTER. Jika diabaikan, proses penemuan akan menyaring ke model-model LATTICE (`TRI032*`) dan menghubungkan semuanya. |
| `--line {Line0,Line2,Line3}` | `Line2` | Jalur sinkronisasi GPIO. |
| `--target-fps F` | otomatis | Kecepatan pemicu MASTER. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | otomatis | Mengganti pengatur tingkatan. |
| `--wire-ceiling-mbps MB_PER_S` | terdeteksi otomatis | **Anggaran bandwidth berkelanjutan host, dalam MB/s — angka yang menjadi acuan alokasi seluruh array.** Kurangi nilainya saat array melaporkan frame yang rusak akibat GVSP: nilai otomatis ini diperoleh dari kecepatan koneksi yang diiklankan oleh NIC, yang cenderung melebih-lebihkan kemampuan adaptor USB, jalur PCIe yang sempit, dan jaringan bersama yang sibuk. Disimpan dalam blok penangkapan array proyek, sehingga pembukaan ulang / CLI / SDK akan memulihkannya. Lihat [Kesehatan Array](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Pengelompokan perangkat keras. |
| `--no-recommend` | off | Lewati. |
| `--no-ptp` | off | Nonaktifkan PTP (stempel waktu antar-kamera kemudian **tidak** dapat dibandingkan). |

### Smart-AE / Smart-Capture

Array LATTICE menjalankan AE secara terus-menerus di latar belakang segera setelah terhubung, tetapi adegan yang baru diatur memerlukan waktu sejenak untuk konvergensi. `array-capture --smart` adalah **solusi praktis**: fitur ini menunggu hingga AE stabil di semua kamera dalam array, lalu memicu proses pengambilan gambar. Gunakan fitur ini saat Anda mengganti pemandangan di tengah sesi.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Kebijakan stabilisasi ini bersifat konservatif dengan : batas waktu 5 detik, jendela stabilitas 1,5 detik, toleransi penyebaran eksposur ±5%. Sesuaikan melalui Pengaturan Ekspor Pengambilan Gambar (SDK, `ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) jika Anda membutuhkan perilaku yang berbeda dari otomatisasi.

### Tingkat Ekspor Pengambilan Gambar (pengaturan default `all`)

Mulai rilis ini, `lattice capture`, `lattice multi-capture`, dan `lattice array-capture` **secara default menggunakan `--processing all`** — satu berkas tersimpan per jenis ekspor yang berlaku untuk setiap kamera, sesuai dengan perilaku &quot;Capture All&quot; pada antarmuka pengguna (GUI). Tingkat-tingkatnya adalah:

| Tingkat | Keluaran | Berlaku untuk |
| --- | --- | --- |
| `raw` | Bayer saluran tunggal (kamera monokrom: pita tunggal) langsung dari sensor. | Semua kamera. |
| `debayered` | Demosaik BGR 3 saluran (kamera monokrom:-saluran skala abu-abu). | Semua kamera. |
| `radiance` | float32 W/m²/sr/nm melalui rantai radiometrik lengkap. | Hanya multispektral (M3C/M3M) — **dilewati untuk kamera dengan filRGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), siap untuk Pix4D. | Hanya multispektral, dan **hanya jika DAQ terikat + kamera telah dikalibrasi**; jika tidak, dilewati. |
| `preview` / `display` | Rantai pratinjau GUI lengkap (CCM + WB + gamma sesuai profil kamera). `lattice capture` menamai ini `preview`; `array-capture`/`multi-capture` menggunakan `display`. | Semua kamera. |

Tentukan satu tingkat saja untuk menyimpan hanya tingkat tersebut (`--processing debayered`). Saat Anda meminta `all`, tingkat yang tidak berlaku untuk kamera tertentu akan dilewati (dan dilaporkan), bukan dianggap error — kamera yang tidak terhubung atau belum dikalibrasi tetap mendapatkan `raw` / `debayered` / `preview`.

Untuk setiap bingkai reflektansi, pembacaan DAQ downwelling yang sebenarnya digunakan ditulis ke file sidecar **`.daq`** di samping citra (sehingga penangkapan tersebut dapat diproses ulang nanti) dan dilaporkan pada baris `DAQ:`.

### Tampilan folder penangkapan

Setiap jenis ekspor disimpan di **subfolder tersendiri** di bawah `-o`, sehingga penangkapan bertingkat tidak pernah mencampurkan jenis-jenis:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` adalah cap waktu pengambilan gambar dan `<serial>` adalah nomor seri kamera, sehingga satu grup yang disinkronkan memiliki
cap waktu yang sama di seluruh kamera. **Perhatikan satu ketidaksimetrisan:** tingkat `display` disimpan dalam folder
bernama `preview/`, sedangkan file-file tersebut tetap menggunakan `_display` dalam namanya — nama folder dan sufiks berbeda
hanya tingkat tersebut saja. Tingkat yang tidak dikenal akan dialihkan ke folder dengan nama mereka sendiri, dan jika subfolder
tidak dapat dibuat, file akan ditulis ke akar output daripada hilang.

**Memproses ulang folder captures:**arahkan `chloros-cli process` ke**akar captures**
(`output/`). `process` biasanya hanya mengimpor folder yang Anda tentukan, tetapi jika folder tersebut tidak berisi
gambar dan memiliki subfolder, maka sistem akan menelusuri ke bawah secara otomatis — sehingga subfolder tingkat akar dan
folder induk `.daq` akan diambil sekaligus. Setiap tingkat dalam tangkapan akan diimpor sebagai satu gambar dengan
tingkat-tingkat lainnya tersedia sebagai mode, bukan sebagai satu gambar per tingkat.

Menamai **subfolder tingkat** secara langsung (misalnya `output/raw/`) juga bisa dilakukan. Melakukan hal tersebut akan meninggalkan
`.daq` tertinggal, jadi salin atau arahkan pembacaan DAQ tersebut bersamaan saat Anda menghitung ulang produk radiometrik
dari `raw/` — jika tidak, pencocokan cap waktu tidak memiliki acuan untuk diselesaikan.

**Pemrosesan selalu dimulai dari `raw`.** Dalam setiap penangkapan, bingkai mentah merupakan sumber pipa pemrosesan;
`debayered`, `radiance`, `reflectance`, dan `preview` muncul sebagai mode tampilan tetapi tidak pernah dimasukkan
kembali melalui pipa. Memproses ulang produk turunan akan menerapkan kembali vignette, CCM, dan
perhitungan radiance yang sudah tertanam dalam pikselnya, sehingga Chloros menolaknya daripada
melakukan pemrosesan ganda. Dua konsekuensi yang perlu diketahui:

- Render `index/` dan `composite/` **tidak pernah** diproses. Keduanya merupakan keluaran, bukan hasil tangkapan —
  render LUT NDVI tidak memiliki interpretasi radiance yang bermakna.
- Folder tangkapan yang diekspor **tanpa** `raw` (misalnya `array-capture --processing reflectance`) tidak memiliki
  sumber pipeline yang sah. Capture tersebut dapat diimpor dan ditampilkan secara normal, tetapi `process` melewatkannya
  dan menampilkannya sebagai berikut:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Jika Anda benar-benar perlu meneruskan produk turunan — sesi hub yang direkam dengan
  `demosaic` diaktifkan, atau folder warisan — `--input-level {raw,debayered,processed}` memaksa titik masuk
  dan mengabaikan pengabaian tersebut. Bendera tersebut merupakan jalan keluar yang disengaja; `auto` (pengaturan default)
  tidak pernah memproses penangkapan yang tidak memiliki data mentah.

### Penangkapan yang Dilewati dalam Susunan Filter Campuran

Saat Anda mencampur kamerRGB, dan multispektral dalam satu susunan, `array-capture --processing radiance` (atau `reflectance`) menyimpan bingkai multispektral dan **melewati** kamera RGB — radian per-Bayer tidak bermakna untuk sensor broadband. CLI mencetak setiap file yang disimpan (bersama tingkat ekspornya), setiap `.daq` yang ditulis, dan setiap pengabaian secara eksplisit, sehingga jumlah file tidakmengejutkan:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Token alasan melewati mengikuti pola `<level>-not-applicable-to-rgb-cam`. Reflektansi juga dapat dilewati dengan `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, dan dengan `dls-uncalibrated-band-<nm>` jika pita tersebut sebagian besar berada di luar rentang yang dikalibrasi secara radiometrik dari sensor cahaya DAQ (~374–974 nm) — di antara SKU yang tersedia, hanya F988 yang mendukung alur kerja panel reflektansi.

Gunakan `--processing debayered` (atau `display`) untuk menyertakan semua kamera terlepas dari jenis filternya, atau `all` (default) untuk mendapatkan semua level yang berlaku per kamera dalam satu kali eksekusi.

---

## Mode Pengambilan, Perekam, &amp; Pemrosesan Ulang Offline

Semua ini beroperasi pada **array persisten** (jalankan `array-connect` terlebih dahulu). Fitur-fitur ini mencerminkan panel pengambilan pada antarmuka pengguna grafis (GUI).

### Mode `array-capture`

`array-capture` adalah perintah tunggal dengan empat mode rana ditambah serangkaian tombol pengaturan ekspor:

| Mode | Bendera | Perilaku |
| --- | --- | --- |
| **Tunggal** *(default)* | (tidak ada) | Satu grup penangkapan yang disinkronkan, lalu keluar. |
| **Berkelanjutan** | `--continuous` |-ke-belakang hingga `Ctrl+C`, `--count N`, atau `--duration S`. |
| **Interval** | `--interval S` | Satu putaran setiap `S` detik (diukur dari awal setiap putaran), batas yang sama. |
| **Tercepat** | `--fastest` | Hanya data mentah + pembacaan DAQ yang ditetapkan + komposit indeks gabungan; melewati perhitungan radian/reflektansi/tampilan sehingga bingkai ditampilkan dengan cepat. Menyiratkan `--processing raw --force-daq`. Memproses ulang `.daq` yang disimpan menjadi produk yang dikalibrasi nanti. |

Tombol pengaturan ekspor (dapat digabungkan dengan mode apa pun; semuanya menggunakan antarmuka pengguna/titik akhir SDK):

| Bendera | Efek |
| --- | --- |
| `--processing LEVEL` | Tingkat ekspor tunggal, atau `all` (default). |
| `--levels L1,L2,…` | Subset eksplisit dari (misalnya `raw,radiance,reflectance`); **menggantikan `--processing`**. |
| `--aligned` / `--no-aligned` | Melakukan warp ekspor non-raw setiap anggota ke [profil penyelarasan](#alignment) (terdaftar bersama). Data mentah tetap tidak diwarp tetapi membawa transformasi dalam metadata. Kembali ke tidak selaras (dengan peringatan) jika array tidak memiliki profil. |
| `--index` / `--no-index` | Simpan / lewati overlay indeks vegetasi per-kamera jika telah dikonfigurasi. Default: tampilkan. |
| `--force-daq` | Simpan pembacaan DAQ/DLS yang ditetapkan sebagai berkas sidecar `.daq` meskipun tidak ada tingkat yang dipilih yang membutuhkannya (misalnya, pengambilan data mentah saja), sehingga bingkai-bingkai tersebut dapat diproses ulang menjadi reflektansi/indeks secara offline. |
| `--smart` | Tunggu hingga AE stabil di semua kamera sebelum memicu pemotretan (lihat [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresi pikselTIFF. `deflate` (default) = zlib L1 tanpa kehilangan data + prediktor horizontal, ~4,1 MB per bingkai resolusi penuh; `none` = tidak terkompresi, penulisan ~5× lebih cepat pada ~6,3 MB per bingkai — gunakan untuk kecepatan berkelanjutan maksimum jika ruang disk mencukupi. Keduanya tanpa kehilangan data dan dibaca secara identik saat diimpor. |

> **Penulisan tunggal TIFF + model kecepatan berkelanjutan.**Hasil tangkapan ditulis dalam**satu**tifffile yang membawa piksel + XMP + IFD0 Make/Model (diukur pada Mono12 resolusi penuh: 36 ms terkompresi / 6,5 ms tidak terkompresi, dibandingkan dengan ~148 ms untuk metode lama “tulis lalu tulis ulang dengan ExifTool”); satu-satunya pekerjaan ExifTool yang tersisa (pembenahan sub-penyempurnaan IFD) berjalan pada pekerja latar belakang asinkron, dan sebuah bingkai sudah selesai serta siap diimpor meskipun proses tersebut tidak pernah berjalan. Perhatikan bahwa kompresi DEFLATE mengunci GIL Python, sehingga penulisan terkompresi**tidak**dapat dijalankan secara paralel di seluruh utas penulis per kamera — perekaman resolusi penuh berkelanjutan dengan 8 kamera pada kecepatan sensor (~10,4 fps) memerlukan `--compression none`**dan** disk kelas NVMe (kecepatan penulisan berkelanjutan ~500 MB/s). Opsi yang sama tersedia sebagai `compression` pada `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — video/GIF indeks gabungan (kelas pemantauan)

Merekam apa pun yang ditampilkan oleh **tampilan indeks gabungan langsung** ke `.avi` (dan opsional ke `.gif`). Karena mengambil data dari komposit langsung, aliran gabungan harus dalam keadaan terbuka (misalnya, array sedang dipandang pratinjau di antarmuka pengguna) agar bingkai-bingkai dapat terekam. Proses ini memeriksa kemajuan setiap 2 detik dan berhenti pada `--duration`, `Ctrl+C`, atau ketika perekam berhenti secara otomatis.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--array-id ID` | hanya array | Array target (dihilangkan jika hanya satu yang terhubung). |
| `-o, --output DIR` | `output` | Direktori keluaran (lokal backend). |
| `--fps F` | `10` | Kecepatan bingkai perekaman. |
| `--duration S` | hingga Ctrl+C | Otomatis-berhenti setelah `S` detik. |
| `--gif` | nonaktif | Juga menulis GIF animasi. |
| `--gif-only` | nonaktif | Hanya menulis GIF (tanpa `.avi`). |

### `array-burst` — burst raw-Bayer dengan fps tinggi (tingkat analisis)

Membaca buffer grup sinkronisasi loop pengambilan gambar secara langsung — **tidak memerlukan rantai kalibrasi, exiftool, maupun tampilan langsung** — sehingga berjalan pada kecepatan pengambilan gambar penuh kamera. Menulis bingkai mentah + manifes per bingkai + satu `.daq` per pembacaan DLS yang berbeda di bawah `<output>/bursts/<base>/`. Proses ulang secara offline (perintah berikutnya), atau berikan `--build` untuk melakukannya segera setelah berhenti.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--array-id ID` | hanya array | Array target. |
| `-o, --output DIR` | `output` | Direktori keluaran (hasil burst disimpan di `<DIR>/bursts/<base>/`). |
| `--duration S` | hingga Ctrl+C | Berhenti otomatis setelah `S` detik. |
| `--max-frames N` | tak terbatas | Berhenti otomatis setelah `N` bingkai mentah. |
| `--build` | mati | Setelah berhenti, segera proses ulang rentetan (sama seperti `array-build-video`). |
| `--products …` | `combined:index` | Dengan `--build`: video mana saja yang akan dibuat (lihat di bawah). |
| `--fps F` | `10` | Dengan `--build`: fps video keluaran. |
| `--save-tiffs` | nonaktif | Dengan `--build`: juga simpan perTIFF yang dikalibrasi per bingkai. |
| `--gif` | mati | Dengan `--build`: juga menulis GIF animasi(s). |

### `array-build-video` — memproses ulang burst yang disimpan secara offline

Menyesuaikan waktu setiap frame mentah dengan pembacaan `.daq` terdekat yang disimpan dan meneruskannya melalui **rantai indeks radian / reflektansi / indeks yang sama dengan pipa impor**, menghasilkan satu atau lebih video.

`--products` adalah daftar yang dipisahkan koma dari item `kind:level`, di mana `kind` ∈ `per_cam` | `combined` dan `level` ∈ `radiance` | `reflectance` | `index`. Sebuah `level` (tanpa `kind:`) secara default akan menjadi `per_cam`. Standarnya adalah `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--burst-dir DIR` | (diperlukan) | Jalur ke folder burst (`…/bursts/<base>/`). |
| `--products …` | Daftar `combined:index` | Daftar `kind:level`, seperti di atas. |
| `--fps F` | `10` | FPS video keluaran. |
| `--save-tiffs` | nonaktif | Juga menyimpan file TIFF yang telah dikalibrasi per-frame bersamaan dengan video(s). |
| `--gif` | nonaktif | Juga menulis GIF animasi(s). |

> **Pilih perekam yang tepat.** `array-record` adalah *kelas pemantauan* — merekam komposit langsung sebagaimana ditampilkan dan memerlukan aliran yang terbuka. `array-burst` → `array-build-video` adalah *kelas analisis* — menyimpan data sensor mentah pada kecepatan penuh dan merekonstruksi video radian/refleksiindeks yang telah dikalibrasi, tanpa memerlukan tampilan langsung.

### Kamera Pita Tunggal Mono (M3M)

Seri **M3M**adalah versi mono dari seri Bayer**M3C**: satu filter interferensi pita sempit per kamera (`M3M-<lens>-F<wavelength>`, misalnya `M3M-L87-F685`), sehingga sensor menghasilkan**satu pita skala abu-abu** tanpa mosaik Bayer. Tidak ada yang perlu didemosaik, tidak ada crosstalk antar-saluran yang perlu dipisahkan, dan tidak ada white balance yang perlu diatur — seluruh alur kerja warna RGB -display sama sekali tidak berlaku.

Apa artinya hal ini pada CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**mendeteksi kamera monokrom dan**melewatinya dengan pesan satu baris** alih-alih menerapkan pengaturan yang tidak berguna. Mereka tetap berjalan normal saat diuji pada kamera RGB /Bayer M3C dalam sesi yang sama.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** masih berfungsi — radian dan reflektansi adalah peta radiometrik *per-pita* dan didefinisikan dengan sempurna untuk satu pita. Frame mono membawa matriks respons sensor **identitas** (tanpa pemisahan 3×3), sehingga bidang tersebut melewati perhitungan kalibrasi tanpa perubahan.
- **Satu kamera mono saja tidak dapat menghasilkan indeks vegetasi.**NDVI / NDRE /dll. memerlukan setidaknya dua band (misalnya Red + NIR). Untuk mendapatkan indeks dari perangkat mono, arahkan**beberapa** kamera M3M pada panjang gelombang yang berbeda, gabungkan menjadi satu tumpukan multi-band, dan hitung indeks *dari tumpukan tersebut*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` simbol-simbolnya harus **persis** sama dengan nama saluran pada preset (peka huruf besar/kecil; NDVI menggunakan huruf kecil `red`,`nir` — lihat `--list-presets`), dan nama sisi band harus merujuk pada band dalam tumpukan yang disejajarkan (mode offline juga menerima indeks band berbasis 0, misalnya `--channel red=0 --channel nir=1`).

Pembeda di seluruh tumpukan adalah token `M3M` dalam string model (token ini tidak pernah muncul dalam string `M3C`), yang ditampilkan ke GUI/SDKsebagai `is_mono`.

---

## Pengaturan NIC Host &amp; Penyesuaian (array LATTICE)

Kamera LATTICE mengalirkan GVSP melalui adaptor Ethernet host, sehingga untuk array multikamera, **driver**adaptor dan**ukuran receive-ring** sama pentingnya dengan kecepatan tautan. Pengaturan yang salah akan muncul sebagai gerbang `FRAMES WILL DROP` / `Reduce ROI to enable` di panel Pengaturan Array (dan pada `lattice network-analysis` / `analyze_array_network()` pada &quot;SDK&quot;), meskipun kamera-kamera tersebut sendiri dalam kondisi baik.

### Adaptor USB 10GbE — Realtek RTL8157 (&quot;Realtek USB 10GbE Family Controller&quot;)

| Item | Nilai yang diperlukan | Mengapa hal ini penting |
| --- | --- | --- |
| **Versi driver**|**≥ v10.67 (Jan 2026)**, INF `rtump64x64sta.inf` | Driver**2016**yang sudah usang (v10.65, `rtump64x64.inf`) tidak menangani proses pematian daya dan bugcheck dengan benar saat menggunakan**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**pada saat shutdown/restart/ssleep. Proses transisi macet (timeout ~5 menit), pengguna mematikan paksa, dan shutdown yang tidak bersih yang berulang-ulang**merusak repositori WMI**(PowerShell/alat mulai mengalami kegagalan dengan kode `Invalid class`) dan**menyebabkan tumpukan USB macet** pada booting berikutnya (NIC tidak dapat diaktifkan; drive USB berhenti terdeteksi). Perbarui dari realtek.com (atau vendor dongle) sebelum mengandalkan restart yang bersih. |
| **Buffer Penerimaan**— kata kunci `ReceiveBufferLen` |**256**(maksimum driver) | Cincin RX NIC. Nilai default driver sebesar**32**hanya menyisakan ~0,26 MB ruang cincin yang dapat digunakan — jauh terlalu kecil untuk burst multi-kamera — sehingga panel array melaporkan `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` dan koneksi terblokir. Pada**256**, ukuran cincin menjadi besar (**~13,5 MB yang diukur pada host 10GbE di lab**), sehingga memberikan ruang kosong yang cukup pada pipa RX untuk burst GVSP multi-kamera. (Apakah konfigurasi tertentu benar-benar *terhubung* ditentukan oleh dua pemeriksaan — pemeriksaan **kesadaran drainase**dan pemeriksaan**over-subscription agregat** — bukan perbandingan mentah antara burst dan ring; lihat [Model fps &amp; burst array](#array-fps--burst-model).) |
| **URB Penerimaan**— kata kunci `PendingReceives` |**64** (maks) | Blok permintaan USB yang sedang dikirim; tingkatkan bersamaan dengan Buffer Penerimaan untuk penyerapan burst. |
| **Jumbo Frame** — kata kunci `*JumboPacket` | **9014** | Diperlukan untuk paket GVSP 9000 byte (6 kali lebih sedikit paket per frame dibandingkan 1500). |

> ⚠️ **Pembaruan driver NIC AKAN MENGEMBALIKAN properti lanjutan ini ke pengaturan default.**Setelah memperbarui atau mengganti driver adaptor,**terapkan kembali** `ReceiveBufferLen=256` dan `PendingReceives=64`, atau panel array akan memblokir koneksi lagi meskipun &quot;tidak ada perubahan pada perangkat keras.&quot; Ini adalah penyebab utama mengapa perangkat yang sebelumnya berfungsi tiba-tiba menolak terhubung.

Terapkan dari PowerShell dengan **hak akses administrator** (ganti dengan nama adaptor Anda, misalnya `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` mencakup adaptor USB 10GbE.** Sekarang sistem mendeteksi jenis adaptor dan menyesuaikan kata kunci receive-ring yang tepat: `*ReceiveBuffers`→2048 untuk NIC PCIe (Intel I219, dll.), atau `ReceiveBufferLen`→256 + `PendingReceives`→64 untuk pengontrol **USB** 10GbE Realtek (yang tidak mengekspos `*ReceiveBuffers`). Target dibatasi hingga nilai maksimum yang dilaporkan oleh masing-masing driver (`NumericParameterMaxValue`), sehingga tidak akan pernah menulis nilai di luar rentang. Jalankan perintah ini dari terminal **dengan hak akses tinggi** ; seperti halnya penyesuaian berbasis registri lainnya, perubahan ini berlaku setelah adaptor di-restart atau sistem di-reboot. Perintah manual `Set-NetAdapterAdvancedProperty` di atas tetap menjadi alternatif yang baik — perintah tersebut berlaku secara langsung (mengikat ulang adaptor) tanpa perlu restart.

### Dasar-dasar jaringan (semua tautan LATTICE)

- **Pengalamatan:** link-local `169.254.0.0/16` (GigE Vision LLA). Host menggunakan alamat statis `169.254.x.x/16`; kamera + DAQ-E menetapkan alamat sendiri dalam rentang yang sama. Tidak diperlukan DHCP/gateway.
- **Ukuranet:**lebih disarankan jumbo (9000), tetapi biarkan auto-probe menemukannya — auto-probe akan mengukur ulang pada setiap koneksi dan sudah melewati batas ICMP 1500 byte kamera melalui probe GVSP, sehingga akan menggunakan jumbo di mana pun kabel benar-benar mendukungnya. Tetapkan nilai tetap dengan `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` hanya jika Anda lebih tahu daripada probe, dan utamakan pengaturan per-perintah daripada permanen: nilai tetap akan melewati proses probe, sehingga jika jalur sebenarnya tidak dapat mentransmisikan 9000**setiap** penangkapan akan mengalami timeout dengan `SC_ERR_TIMEOUT -1011` (lihat [Variabel Lingkungan](#environment-variables)).
- **Cincin RX berskala dengan `ReceiveBufferLen`:**pada nilai default `32`, cincin yang dapat digunakan adalah ~0,26 MB (terlalu kecil untuk burst multi-kamera apa pun); pada nilai maksimum `256`, ukurannya besar (~13,5 MB yang diukur pada host 10GbE di lab), sehingga memberikan ruang cadangan yang nyata. Apakah suatu konfigurasi dapat terhubung kemudian ditentukan oleh pemeriksaan penerimaan yang menyadari beban**dan** pemeriksaan kelebihan langganan agregat di bawah ini — bukan perbandingan mentah antara burst dan ring.

### Model fps &amp; burst Array

Cara membaca panel Pengaturan Array (dan `lattice analyze-array` / `analyze_array_network` pada SDK):

- **Burst dijumlahkan per kamera pada format piksel asli masing-masing kamera.**Kamera Mono**M3M**kamera mengalirkan**Mono12 (2 B/px)**; kamera**M3C**Bayer mengalirkan 8- atau 12-bit (TRI032S secara diam-diam mengeluarkan BayerRG12 bahkan ketika BayerRG8 diminta). Jadi, satu bingkai resolusi penuh dari 4 kamera berukuran**~12.6 MB jika semuanya 8-bit, tetapi ~25 MB dengan tiga kamera mono 12-bit**. Proyeksi ini menentukan format setiap kamera berdasarkan modelnya (cache identitas), sehingga data burst sesuai dengan apa yang sebenarnya dikirimkan melalui kabel — bukan asumsi BayerRG8 yang seragam.
- **Adaptor Ethernet USB dibatasi hingga 200 MB/s terlepas dari spesifikasi yang tertera.** Tabel efisiensi yang mengubah kecepatan tautan menjadi angka yang berkelanjutan berasal dari PCIe; sebuah NIC USB mengiklankan kecepatan tautan *Ethernet*-nya tetapi dibatasi oleh bus USB dan drivernya. Sebuah dongle USB 10GbE pernah mencapai ~1063 MB/s &quot;berkelanjutan&quot; — angka yang tidak pernah diuji — dan pengaturan kecepatan yang dihasilkan merusak 6–18 % frame, meskipun tetap melaporkan target fps yang normal. NIC yang terhubung melalui USB kini dibatasi pada **200 MB/s** sebagai batas absolut (batasnya adalah bus, sehingga tidak berskala sesuai spesifikasi; adaptor USB 1 GbE menghasilkan ~80 MB/s dan tidak terpengaruh). `wire_ceiling_source` pada catatan kemampuan menyatakan hal tersebut secara tertulis, dan `nic_is_usb` menandainya. Ganti pengaturan tersebut dengan `--wire-ceiling-mbps`.
- **Kapasitas masuk (admittance) mempertimbangkan beban keluar (drain), bukan membandingkan burst keseluruhan dengan ring.** Sebuah burst simultan hanya perlu muat dalam *backlog sementara* = `max(0, Σ per-cam arrival − host drain) × emit_window`, bukan seluruh burst. Pada-host / fabric cam yang lambat (sebuah host **PCIe**10G + 4× cam 1 GbE: kedatangan ≈ 320 MB/s, pengosongan ≈ 1063 MB/s), host mengosongkan lebih cepat daripada cam mengisi, backlog ≈ 0, sehingga**sim-emit**resolusi penuh**diterima**meskipun burst 25 MB melebihi kapasitas ring 13,5 MB. Letakkan keempat kamera yang sama di belakang adaptor**USB**10GbE dan laju pengosongan menjadi 200 MB/s, bukan 1.063 — laju kedatangan melampaui laju pengosongan, dan kerugiannya muncul sebagai frame yang rusak, bukan sebagai laju frame yang lebih rendah. Pada host 1 GbE, batas bawah DLThr kamera sebesar 31,25 MB/s DLThr membuat kedatangan melebihi laju pengosongan → sistem dengan benar**memblokir** (untuk *kelas* pemblokiran ini, kurangi ROI atau gunakan binning ≥ 2). Admittance adalah salah satu dari **dua** gerbang koneksi — yang lainnya adalah agregat ataspemeriksaan subskripsi di bawah ini.
- **FPS yang diproyeksikan adalah batas atas pengambilan serial yang konservatif.**Loop pengambilan host saat ini menarik buffer setiap kamera**secara serial**(~satu jendela pengiriman per kamera masing-masing), sehingga siklus dibatasi oleh `max(readout+emit, N × emit)` dengan pengiriman per kamera dibatasi pada**link akses**kamera (1 GbE ≈ 80 MB/s), bukan oleh uplink host. Untuk susunan 4 kamera resolusi penuh 12-bit, nilainya**~2,8 fps**, sesuai dengan hasil pengukuran ~2,7–3,0. fps ini sengaja dibuat**tidak bergantung pada eksposur**, sehingga dalam adegan redup, nilai aktual dapat turun sedikit di bawah batas atas seiring bertambahnya durasi eksposur. Pengambilan data secara serial adalah pembatas fps yang sesungguhnya; melakukan paralelisasi akan menaikkan batas atas mendekati laju emisi tunggal.
- **Kelebihan alokasi agregat merupakan penghambat koneksi yang kritis.**Alokasi bandwidth per kamera memiliki batas bawah sebesar**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), sehingga begitu batas bawah tersebut tercapai, permintaan agregat (`per_cam × N`) dapat melebihi**batas maksimum kabel yang aman dari tabrakan**(`sustained × sim_emit_factor`). Batas maksimum resolusi penuh yang praktis pada 1 GbE:**6 kamera pada 1500 MTU, 9 kamera dengan jumbo**. Batas atas ini merupakan sifat dari kabel dan batas bawah saja — hal ini**tidak bergantung pada ukuran frame**, sehingga**pengelompokan (binning) dan ROI yang lebih kecil TIDAK membantu** (keduanya menurunkan jumlah byte per *frame*, bukan jumlah byte per *detik* yang diatur oleh GevSCPD); satu-satunya solusi adalah mengurangi jumlah kamera, menghentikan penggunaan frame jumbo, atau NIC yang lebih cepat. Gejalanya adalah kehilangan paket GVSP, bukan penurunan fps yang bertahap, sehingga `analyze-array` mengosongkan angka fps yang dapat dicapai dan menampilkan `**OVER-SUBSCRIBED**`, dan `array-connect` dengan resolusi yang dikunci **menolak untuk terhubung** (proses walk-down biasanya mengelompokkan frame ke bawah, yang juga tidak mengatasi masalah blok ini). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` menurunkan status penolakan menjadi peringatan keras untuk pekerjaan pengujian — lihat [Variabel Lingkungan](#environment-variables).

### Kesehatan array — subsistem mana yang kehilangan frame

`GET /api/camera/array/<array_id>/capability` dari array yang terhubung menampilkan
`health`, yang dievaluasi ulang dalam jendela bergulir **10 detik**. Blok ini membagi kehilangan frame
menjadi dua penyebab yang memerlukan perbaikan berlawanan, alih-alih melaporkan satu tingkat &quot;tidak lengkap&quot;
yang tidak menyebutkan keduanya:

| Kolom | Artinya | Subsistem mana |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per serial) | Frame **telah tiba tetapi secara struktural rusak**— kehilangan paket GVSP. |**Jaringan**: alokasi bandwidth, kecepatan pengiriman, cincin penerima NIC, MTU |
| `never_arrived_rate_pct` (per nomor seri) | Frame **sama sekali tidak pernah tiba**— kamera tidak memicu, atau tidak ada data yang dikirimkan. |**Pemicu / sinkronisasi**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Tingkat terburuk dari setiap kamera. | — |
| `per_cam_rate_pct` | Tingkat ketidaklengkapan gabungan per kamera (kedua penyebab tersebut digabungkan). | — |
| `stable_for_seconds` | Berapa lama setiap kamera tetap berada di bawah 0,01 %. | — |

Di atas 5%, backend mencatat baris `[array-health <id>] WARN` yang menyebutkan pemisahan — pada
pelanggaran pertama, saat terjadi perubahan tingkat keparahan, sekali setiap menit selama kondisi tersebut berlanjut, dan sekali saat
kondisi tersebut teratasi. Bagian yang rusak mencetak `[gvsp-corrupt <SN>]` pada pelanggaran pertama per kamera dan
alasannya, lalu ringkasan setiap 60 detik. Setiap evaluasi tetap terekam dalam berkas log backend;
penghitung terus berjalan pada setiap buffer terlepas dari apa yang dicatat.

Catatan yang sama melaporkan angka yang menjadi dasar alokasi keseluruhan:

| Kolom | Artinya |
| --- | --- |
| `wire_ceiling_mbps` | Anggaran bandwidth yang berlaku secara berkelanjutan pada host, MB/s. |
| `wire_ceiling_source` | Asal usul angka tersebut, dalam bentuk kata — mis.misalnya `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` atau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` ketika `--wire-ceiling-mbps` (atau bidang **Wire Budget** pada GUI) mengaturnya. |
| `nic_is_usb` | `true` untuk adaptor Ethernet USB — lihat batas 200 MB/s di atas. |

**Membacanya:** nilai `gvsp_corrupt_rate_pct` dengan `never_arrived_rate_pct` bernilai 0
berarti pemicu dan sinkronisasi kabel sempurna, dan 100% kerugian terjadi pada jalur jaringan
— kurangi nilai `--wire-ceiling-mbps` dan sambungkan kembali. Pola sebaliknya mengindikasikan adanya masalah pada
kabel sinkronisasi atau jalur pemicu.

> **`--target-fps` bukanlah pengatur untuk frame yang rusak.** Pengaturan kecepatan GevSCPD ditulis
> sekali saat koneksi, sehingga menurunkan laju pemicu akan mengubah siklus kerja dan bukan
> laju burst emisi simultan. Pengurangan permintaan sebesar 5× yang diukur tidak menghasilkan perbaikan;
> menurunkan batas maksimum kabel dari 240 menjadi 200 MB/s menurunkan tingkat kerusakan pada rig yang sama dari 10,4 %
> menjadi 0,00 %.

> **Fitur penyusutan otomatis di tengah aliran tidak tersedia pada firmware TRI032S.** Sebuah array yang sedang berjalan
> tidak dapat memperbaiki masalah ini sendiri; putuskan sambungan dan sambungkan kembali agar pemilih waktu koneksi dapat
> merencanakan ulang dengan batas atas yang baru.

### Gejala → solusi

| Gejala (Pengaturan Array / koneksi / `analyze_array_network`) | Penyebab | Solusi |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` diatur ulang ke 32 (biasanya setelah pembaruan driver) | Atur `ReceiveBufferLen`→256, `PendingReceives`→64; buka kembali panel (mulai ulang backend jika ukuran ring lama masih tersimpan dalam cache) |
| Proses restart/shutdown macet; kemudian `Invalid class` kesalahan WMI, NIC tidak dapat diaktifkan, drive USB hilang | Driver Realtek USB 10GbE versi lama tahun 2016 → BSOD `0x9F` → pematian paksa | Perbarui driver adaptor ke v10.67 atau lebih tinggi (2026), lalu terapkan kembali pengaturan receive-ring di atas |
| Koneksi berhasil tetapi menghasilkan resolusi di bawah standar asli | Smart-prep secara otomatis memperkecil bingkai agar sesuai dengan kabel | Tingkatkan tautan / terima pengecilan tersebut / `--force-tier slip-emit-and-capture` |
| Array melaporkan fps target yang baik tetapi hanya menghasilkan sebagian kecil darinya; `health.gvsp_corrupt_rate_pct` tidak nol, `never_arrived_rate_pct` 0 | Anggaran kabel yang disimpulkan oleh host melebihi kapasitas yang sebenarnya dapat dipertahankan (umum terjadi pada adaptor Ethernet USB, jalur PCIe yang sempit, atau jaringan bersama) | Sambungkan kembali dengan nilai `--wire-ceiling-mbps` yang lebih rendah dan periksa kembali blok kesehatan. **Bukan** `--target-fps` — Pacing GevSCPD ditetapkan saat koneksi |
| Kamera tidak terdeteksi dalam grup yang dipublikasikan; `health.never_arrived_rate_pct` bukan nol, `gvsp_corrupt_rate_pct` 0 | Jalur pemicu / jalur sinkronisasi — kamera tidak aktif, bukan masalah jaringan | Periksa kabel sinkronisasi M8 dan `--line`; pastikan setiap anggota dalam status siaga (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` terlampaui di `analyze-array`, atau penolakan koneksi dengan resolusi yang ditetapkan (`array over-subscribes the wire`) | Total permintaan per kamera (batas bawah 8 MB/s × N kamera) melebihi batas atas koneksi yang aman dari tabrakan — 6 kamera resolusi penuh pada 1 GbE @1500 MTU, 9 dengan jumbo | Jumlah kamera yang lebih sedikit, penggunaan frame jumbo dari ujung ke ujung, atau NIC yang lebih cepat. **ROI/binning TIDAK akan membantu** (batas tersebut tidak bergantung pada ukuran frame). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` mengesampingkan pengaturan pada pengujian (menerima kehilangan paket) |

---

## `chloros-cli daq`

Perintah sensor spektral. Dua kelas:
- **`pool-*`**— klien tipis HTTP yang mengendalikan sensor melalui pool persisten backend.**Ini adalah jalur yang didukung, dan satu-satunya yang tersedia di CLI.** Backend mengelola transportasi, sehingga GUI, skrip CLI, dan SDK semuanya berbagi satu handle aktif alih-alih berebut port serial.
- **Semua yang lain**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — akses langsung ke perangkat keras, didokumentasikan di bawah ini demi kelengkapan. Komponen-komponen ini memerlukan paket `daq` Python, yang**tidak disertakan dalam artefak yang dikirimkan mana pun**: CLI yang telah dikompilasi tidak menyertakannya (`scripts/Build-CLI.ps1` menetapkan `--nofollow-import-to=daq`, dan transportasi `pyserial` / `bleak` / `zeroconf` menyertakannya), dan paket PyPI SDK juga tidak memuatnya. Keduanya hanya dapat dijalankan dari checkout sumber, jadi anggaplah ini sebagai jalur pengembangan internal MAPIR daripada sesuatu yang dapat diakses secara umum.
- **`discover` / `list`** berada di antara keduanya: keduanya merupakan perintah langsung ke perangkat keras dari checkout sumber, tetapi pada build yang telah dirilis, perintah-perintah ini akan beralih ke `pool-discover` dan backend-lah yang melakukan pemindaian. Jadi, pemindaian berfungsi di mana saja — hal ini penting karena ini adalah satu-satunya cara untuk mengetahui BLE MAC dari DAQ-M.

> **`chloros-cli daq --help`** (dan `-h` / `help`) mencantumkan subperintah `pool-*` — bantuan sengaja diarahkan ke klien pool sehingga mencerminkan perintah yang benar-benar dijalankan. Jika Anda menjalankan subperintah direct-hardware pada build yang sudah dirilis, program akan keluar dengan pesan kesalahan eksplisit yang menyebutkan paket yang hilang dan mengarahkan Anda kembali ke `pool-*`; tidak ada yang gagal secara diam-diam. (`discover` / `list` merupakan pengecualian — keduanya dialihkan ke `pool-discover` dan langsung berfungsi.)
>
> **Semua yang dibutuhkan pelanggan dapat diakses melalui `pool-*`** — terhubung, melakukan streaming, merekam berkas `.daq` yang telah dikalibrasi, dan mengganti profil kapasitor. DAQ juga dapat dikendalikan dari Python dengan `chloros_sdk.connect_daq_sensor()`, yang menggunakan jalur terpusat yang sama.

### Alur Kerja Koneksi Pertama Sensor DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referensi `pool-*`

| Subperintah | Tujuan |
| --- | --- |
| `daq pool-connect` (smart-detect) | Membuka sensor di kumpulan backend. |
| `daq pool-connect --port PORT` | DAQ-U pada port serial tertentu. |
| `daq pool-connect --ble` | DAQ-M melalui BLE, pemindaian MAC otomatis. |
| `daq pool-connect --mac MAC` | DAQ-M melalui BLE pada alamat MAC yang diketahui (mengimplikasikan `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E melalui Ethernet pada host yang diketahui. |
| `daq pool-connect --eth` | DAQ-E melalui Ethernet, host terdeteksi otomatis (mDNS + cadangan ARP; berfungsi meskipun cache ARP kosong di Windows dan Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Sesuaikan jendela integrasi / status AE. |
| `daq pool-connect --no-stream` | Terhubung tetapi belum mulai streaming (lanjutkan dengan `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil koreksi batas. Nilai default di backend adalah `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Pindai setiap transport untuk mencari sensor yang dapat Anda hubungkan, tanpa benar-benar terhubung. **Inilah cara menemukan alamat MAC BLE DAQ-M.** `daq discover` / `daq list` secara otomatis diarahkan ke sini pada build yang telah dirilis. Sensor yang sudah terbuka di pool tidak tercantum — DAQ-M yang terhubung akan berhenti mengiklankan alamatnya — jadi gunakan `pool-list` untuk sensor-sensor tersebut. |
| `daq pool-list` | Menampilkan semua sensor di pool backend. |
| `daq pool-disconnect --sensor-id ID [--all]` | Melepaskan. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Frame spektrum N terbaru. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Lanjutkan / jeda streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Mulai / hentikan perekaman .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Tukar profil koreksi cap saat runtime. |

### Subperintah Perangkat Keras Langsung (hanya untuk checkout sumber — tidak ada dalam build yang dirilis)

> Dicantumkan demi kelengkapan. Subperintah ini memerlukan paket `daq` Python ditambah `pyserial` / `bleak` / `zeroconf`, yang tidak ada satupun yang disertakan dalam rilis terkompilasi CLI atau PyPI SDK — perintah-perintah ini hanya dapat dijalankan dari checkout sumber MAPIR. **Jika Anda menggunakan rilis Chloros, gunakan perintah `pool-*` di atas sebagai gantinya**; perintah tersebut mencakup koneksi, streaming, perekaman, dan pemilihan cap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Buka, sambungkan ke, dan jalankan proyek Chloros yang telah disimpan (folder yang berisi `cameras.json` + `sensors.json` + `project.json`). Semua proses diarahkan melalui backend sehingga antarmuka pengguna (GUI) dan CLI menampilkan status perangkat keras yang identik.

### Referensi Subperintah

| Subperintah | Tujuan |
| --- | --- |
| `project open PATH` | Menampilkan daftar perangkat proyek (kamera, array, sensor). |
| `project devices PATH [--reconnect]` | Menampilkan daftar atau menjalankan ulang proses pencarian. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Menghubungkan setiap kamera / array / sensor yang tersimpan. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Pengambilan gambar tunggal dari kamera atau array yang ditentukan. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Pengambilan beruntun N-frame dari kamera atau array yang ditentukan (`-n/--count` default 5; `-i/--interval` detik antar-bingkai, nilai default 0). Burst array menghilangkan duplikat grup yang disinkronkan berulang (pengawas kedaluwarsa) sehingga array dengan siklus parsial tidak dapat mengembalikan N salinan dari satu bingkai; mencetak hasil per iterasi. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Streaming ke disk melalui tugas backend. `--poll-interval` = selang waktu (dalam detik) antara polling `/stats` (default 2,0). |
| `project sensor read PATH NAME [--json]` | Frame spektrum terbaru. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Rekam .daq. |
| `project run PATH RECIPE.yaml` | Jalankan resep perekaman YAML/JSON. `--dry-run` melakukan validasi tanpa dijalankan. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Hitung penyelarasan untuk sebuah array — lihat [tabel opsi di bawah ini](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Cetak profil penyelarasan saat ini. |
| `project align clear PATH NAME` | Hapus profil yang tersimpan dalam cache. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Sesuaikan transformasi satu slave. |
| `project align export PATH NAME --to FILE` | Simpan profil ke JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Muat profil yang telah disimpan. |

#### Opsi `project align calibrate`

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metode penyelarasan. **Ejaan ini berbeda dari `lattice align-calibrate`**, yang menggunakan bentuk singkat `orb` / `akaze` / `phase`; kedua perintah tersebut tidak dapat digunakan secara bergantian pada bendera ini. |
| `--model {translation, rigid, affine, homography}` | `affine` | Mengubah model agar sesuai. |
| `--frames N` | `1` | Menyinkronkan snapshot bingkai ke nilai rata-rata. |
| `--reference SN` | master | Nomor seri kamera referensi; setiap anggota lainnya diwarp ke kamera tersebut. |
| `--max-features N` | `5000` | Batas jumlah fitur ORB. |
| `--ratio-threshold F` | `0.75` | Uji rasio Lowe&#x27;. |
| `--ransac-threshold-px F` | `3.0` | Ambang batas inlier RANSAC. |
| `--min-matches N` | `15` | **Ambang batas kualitas** — tolak hasil pemecahan jika jumlah kecocokan inlier di bawah batas ini. |
| `--max-reproj-err-px F` | `4.0` | **Batasan kualitas** — tolak hasil pemecahan jika melebihi nilai kesalahan reproyeksi RMS ini. |
| `--checkerboard RxC` | — | Geometri papan untuk `--method checkerboard`, mis.misalnya `9x6`. |
| `--name PROFILE` | kosong | Nama profil tertanam dalam file JSON yang disimpan. **Bukan nama array** — itu adalah posisi `NAME`. |

Kedua dua gerbang kualitas adalah alasan mengapa kalibrasi dapat berhasil dalam pemecahan namun tetap
menolak untuk disimpan: profil yang gagal pada salah satu gerbang tersebut akan secara diam-diam menyebabkan ketidaksesuaian pada setiap
penangkapan berikutnya, sehingga profil tersebut ditolak daripada disimpan.

### Contoh

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL Resep

`project run RECIPE.yaml` menerima berkas YAML atau JSON yang mendeskripsikan urutan tindakan:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Tindakan yang didukung: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Tindakan `burst` memerlukan `name` (wajib), `count` (default 5), `interval` (detik, default 0), `output`, `format`, dan `settings` (bentuk pengaturan per-kamera yang sama dengan `apply`); burst array menggunakan watchdog grup yang baru disinkronkan yang sama seperti `project burst`.

Jalankan:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Variabel Lingkungan

| Variabel | Efek |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Mengganti backend URL (default `http://127.0.0.1:5000`) — **hanya diakui oleh keluarga perintah `lattice`, `project`, dan keluarga perintah `daq pool-*`.** Perintah inti (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) mengabaikan `http://127.0.0.1:<port>` dan mengabaikan variabel ini (literal IPv4 melewati Windows `localhost`→`::1` ~2 s per permintaan), sehingga mereka selalu menargetkan mesin lokal. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` menurunkan tingkat penolakan koneksi akibat kelebihan langganan pada array (permintaan agregat per-cam &gt; batas aman tabrakan pada saluran dengan `pin_resolution`) menjadi &quot;peringatan keras dan lanjutkan&quot;, dengan menerima kehilangan paket GVSP. Hanya untuk penggunaan benchmark — lihat [Model fps &amp; burst array](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Ditetapkan oleh CLI itu sendiri; memerintahkan backend untuk mengaktifkan pemrosesan paralel. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` melewatkan probe fallback GVSP (hanya hasil ICMP). **Ini menonaktifkan jumbo, bukan sekadar meredam log** — kamera hanya merespons ping DF hingga 1500 pada setiap jalur, sehingga probe ini adalah satu-satunya cara untuk mendeteksi jumbo. Menghemat ~1 detik per kamera per koneksi; menghabiskan ~1,45× batas bandwidth kabel jika jaringan *sebenarnya* dapat mengangkut paket jumbo. SDK akan memberikan peringatan saat Anda mengaturnya. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Menetapkan ukuran paket GVSP ke N byte; melewatkan proses pengujian sepenuhnya. Lebih disarankan menggunakan pengaturan per perintah (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) daripada mengaturnya secara permanen: ukuran yang dibatasi akan menghentikan adaptasi terhadap jaringan di depannya, dan membatasi ukuran menjadi 9000 pada jalur yang tidak dapat mentransmisikan paket jumbo akan menyebabkan **setiap** penangkapan mengalami timeout dengan kode kesalahan `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Mengganti direktori ekstraksi onefile Nuitka. Opsi CLI secara otomatis menggunakan `/mnt/ssd/tmp` jika ada. |

---

## Kode Keluar

| Kode | Arti |
| --- | --- |
| `0` | Berhasil. |
| `1` | Kegagalan umum (kebanyakan kesalahan subperintah). |
| `2` | Kesalahan argumen. |
| `130` | Dihentikan oleh Ctrl+C. |

---

## Petunjuk Pemecahan Masalah

- **&quot;Diperlukan login&quot;** → Jalankan `chloros-cli login EMAIL PASSWORD` sekali pada mesin ini.
- **&quot;backend tidak dapat dijangkau&quot;** → Jalankan aplikasi desktop Chloros, atau jalankan biner backend secara langsung (`chloros-backend`), atau periksa `CHLOROS_BACKEND_URL` jika menggunakan koneksi jarak jauh.
- **Perintah `lattice` gagal dengan pesan &quot;Driver kamera LATTICE tidak ditemukan&quot;** → Runtime Arena SDK belum terpasang; CLI dilengkapi dengan `win32api` yang disertakan di Windows, tetapi runtime C merupakan bagian dari penginstal GUI.
- **Array connect / Array Settings menampilkan &quot;FRAMES WILL DROP&quot; atau &quot;Reduce ROI to enable&quot;** → Ukuran ring penerimaan NIC host terlalu kecil (biasanya diatur ulang menjadi 32 setelah pembaruan driver NIC). Lihat [Pengaturan &amp; Penyesuaian NIC Host](#host-nic-setup--tuning-lattice-arrays) — atur `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Mesin macet saat restart/shutdown, kemudian WMI menampilkan `Invalid class` / NIC tidak dapat diaktifkan / drive USB hilang** → Driver adaptor USB 10GbE yang sudah usang menyebabkan `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Perbarui driver adaptor — lihat [Pengaturan &amp; Penyesuaian NIC Host](#host-nic-setup--tuning-lattice-arrays).
- **Peringatan swap Jetson** → Tambahkan swap berbasis file; perintah `CLI` akan menampilkan perintah `fallocate` / `swapon` yang tepat.
- **Perintah langsung DAQ tidak ada** → Harusnya: paket `chloros-cli` yang disertakan sengaja tidak menyertakan paket `daq`, sehingga hanya `pool-*` yang tersedia (SDK PyPI juga tidak menyediakannya). Gunakan `pool-*`, yang mengendalikan sensor yang sama melalui backend, atau `chloros_sdk.connect_daq_sensor()` dari Python.

---

## Lihat Juga

- [Python Referensi SDK](sdk-reference.md) — setara secara pemrograman untuk setiap perintah CLI.
- [Panduan Sensor DAQ](../daq/README.md) — pengkabelan + kalibrasi khusus sensor.
- Dokumen daring: `https://mapir.gitbook.io/chloros/cli`

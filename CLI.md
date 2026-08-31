# CLI : Baris Perintah

> **Referensi lengkap:**[Referensi CLI](reference/cli-reference.md) mendokumentasikan**setiap opsi dari setiap subperintah** dan dioptimalkan untuk asisten AI — tempelkan URL ke asisten Anda dan minta perintah yang berfungsi: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tips untuk alat AI:** setiap halaman panduan ini tersedia dalam format Markdown mentah dengan menambahkan `.md` ke URL-nya (misalnya `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), dan `https://mapir.gitbook.io/chloros/llms.txt` mengindeks seluruh manual untuk digunakan oleh LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Apa ItuCLI


`chloros-cli` adalah antarmuka baris perintah (command-line front end) untuk mesin pemrosesan yang sama yang digunakan oleh aplikasi desktopChloros
. Ini adalah klien tipisHTTP
yang terhubung ke backendChloros
(server lokal di `127.0.0.1:5000`) — sebagian besar perintah akan memulai backend secara otomatis, sehingga satu panggilan `chloros-cli process …` saja sudah cukup untuk sebuah skrip.

Aplikasi ini berjalan di **Windows
10/11 (x64)**dan**Linux
(x86_64, serta NVIDIA Jetson arm64 pada JetPack 6)**, di terminal mana pun, tanpa memerlukan antarmuka grafis (GUI). Verifikasi instalasi Anda dengan:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Ringkasan kelompok perintah:

* **Pemrosesan &amp; akun** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 bahasa — lihat [Bahasa yang Didukung](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (hanya untukLinux
/Jetson)
* **Perangkat keras langsung** — `lattice` (kontrol kamera LATTICE, 45+ subperintah), `daq pool-*` (sensor cahaya DAQ), `time-sync` (PTP)
* **Otomatisasi** — `project` (menjalankan proyekChloros
yang disimpan dalam mode headless, termasuk resep perekaman YAML)

Opsi global yang perlu diketahui: `--port N` (port backend, default `5000`), `-v/--verbose`, `--restart` (memaksa backend untuk memulai ulang), `--backend-exe PATH`. Lihat [ReferensiCLI
](reference/cli-reference.md) untuk daftar lengkapnya.

***

## Instalasi

CLI
**termasuk dalam penginstalChloros** di semua platform — tidak ada unduhan terpisahCLI
. Dapatkan penginstal dari halaman [Unduh](download.md).

###Windows


Penginstal akan menempatkanCLI
di:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

dan menambahkan folder tersebut ke sistem Anda `PATH` — **buka terminal baru**setelah menginstal agar `PATH` yang telah diperbarui dapat terdeteksi. Pemasang juga menempatkan skrip peluncur (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) di direktori akar instalasi serta**Chloros
CLI
** pintasan di menu Start, yang masing-masing akan membuka terminal dengan `chloros-cli` yang siap digunakan.

###Linux


Instal `.deb` untuk arsitektur Anda:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Ini akan menginstal `chloros-cli` hingga `/usr/bin/chloros-cli` (sudah berada pada versi `PATH`) dan backend ke versi `/usr/lib/chloros/chloros-backend`, beserta runtime ArenaSDK
yang diperlukan untuk kamera LATTICE. Lihat [Linux
Instalasi](linux/linux-installation.md) untuk detailnya.

### Verifikasi

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Masuk &amp; Lisensi

CLI
(dan akses kePython
SDK
) memerlukan **paket berbayarChloros
+**— semua tingkatan berbayar memilikinya; tingkatan gratis tidak. Batasan ini diterapkan**di sisi server** oleh backend, bukan oleh binerCLI
: panggilan saat tidak masuk akan ditolak dengan kode `401 AUTH_REQUIRED`, dan panggilan yang dilakukan saat login pada paket gratis akan ditolak dengan kode `403 PLAN_UPGRADE_REQUIRED`, baik yang berasal dari `chloros-cli`,SDK
, maupun klienHTTP
yang dibuat sendiri. Lakukan upgrade di [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Masuk **sekali per mesin**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Kata sandi dengan karakter khusus**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` akan rusak oleh shell (CLI
mendeteksi hal ini pada status 401 dan melakukan percobaan ulang secara otomatis, tetapi tanda kutip tunggal dapat menghindari masalah ini sepenuhnya).
{% endhint %}

Sesi disimpan dalam cache di `~/.chloros/user_session.json` dan tetap berfungsi secara offline selama masa tenggang paket (30 hari untuk paket bulanan, hingga masa berlaku habis untuk paket tahunan). `chloros-cli status` tetap berfungsi bahkan tanpa paket berbayar, sehingga alasan penolakan selalu terlihat.

{% hint style="danger" %}
**Sedang menjadwalkan pekerjaan headless? Silakan masuk terlebih dahulu.**Perintah yang memunculkan backend (`process`, `status`, `export-status`, …) yang dijalankan**tanpa sesi yang disimpan dalam cache**tidak akan segera gagal — perintah tersebut akan beralih ke prompt interaktif `Email:` / `Password:` pada stdin. Oleh karena itu, tugas cron atau langkah CI yang dijalankan tanpa pengawasan akan**terhenti sambil menunggu masukan**. Jalankan `chloros-cli login EMAIL 'PASSWORD'` sekali pada mesin tersebut sebelum menjadwalkan apa pun.
{% endhint %}

***

## Proses Pengolahan Pertama Anda

Arahkan `process` ke folder berisi data tangkapan — program ini akan mendeteksi otomatisSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng`, atau campuran:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Alur kemajuan ditampilkan secara langsung per utas pipa (Deteksi, Analisis, Pemrosesan, Ekspor), dan proses yang berhasil diakhiri dengan laporan jumlah produk gambar yang telah ditulis (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Tempat hasil dijalankan

`process` menyimpan ke dalam **folder proyek**, bukan ke folder masukan Anda:

* Tanpa `-o`: proyek dibuat di bawah folder proyek default Anda (dibagikan dengan GUI; kelola dengan `get-project-folder` / `set-project-folder`, cadangan `~/Chloros Projects`), dinamai berdasarkan `-n/--project-name` atau cap waktu (`YYYYMMDD_HHMMSS`) jika diabaikan.
* Dengan `-o PATH`: folder tersebut **adalah** folder proyek. Jika folder tersebut sudah berisi `project.json`, maka folder saudara dengan akhiran `_1`/`_2`… akan dibuat alih-alih menimpa yang sudah ada.

Di dalam proyek, produk dikelompokkan **berdasarkan kamera, kemudian berdasarkan format file**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folder kamera adalah `LATT-<sensor>-<lens>-F<filter>` untuk LATTICE (sesuai dengan EXIF pengambilan gambar `Model`) dan `<model>_<filter>` (misalnya `Survey3N_RGN`) untukSurvey3
. Folder format mengikuti `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, atau `tiff32` untuk `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Setiap produk yang diekspor tetap menggunakan nama berkas ASAL.**Ekspor radiance dari `capture_..._raw.tif` tetap bernama `capture_..._raw.tif` — hanya saja berkas tersebut berada di dalam `tiff32/Radiance_Images/`.**Folderlah yang mengidentifikasi produk, bukan nama berkas**, jadi gunakan pola glob untuk direktori, bukan untuk sufiks `*radiance*`.
{% endhint %}

### Opsi yang akan Anda gunakan

| Bendera | Default | Fungsinya |
| --- | --- | --- |
| `-o, --output PATH` | folder proyek default | Lokasi folder proyek (lihat di atas). |
| `-n, --project-name NAME` | cap waktu | Nama proyek. |
| `--format FMT` | `TIFF (16-bit)` | Salah satu dari `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, atau `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | tidak ada | Indeks vegetasi yang akan diekspor (lihat [Indeks Vegetasi](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neural debayer, lebih lambat, kualitas tertinggi (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | aktif | Koreksi vignette. |
| `--reflectance / --no-reflectance` | aktif | Kalibrasi reflektansi; untuk LATTICE, ini juga berfungsi sebagai tombol pengatur produk reflektansi. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Memaksa titik masuk pipeline untuk file TIFF LATTICE. |

Untuk hal-hal lainnya — penyetelan deteksi target, PPK, pin eksposur, bendera penyelarasan array — lihat bagian [`process` dari ReferensiCLI
](reference/cli-reference.md).

***

## Memilih Apa yang Akan Diekspor (Produk LATTICE)

Pemrosesan LATTICE menyebar ke **setiap produk yang berlaku dalam satu kali proses**. Keempat tombol pengaturan per produk semuanya**diaktifkan secara default**; gunakan formulir `--no-` untuk menonaktifkan salah satunya:

| Tombol | Produk |
| --- | --- |
| `--debayered` | Demosaik linier → `Debayered_Images/` |
| `--preview` | Tampilkan pratinjau (keseimbangan putih + gamma; peregangan warna palsu untuk multispektral) → `Preview_Images/` |
| `--radiance` | radian float32, W/m²/sr/nm → `Radiance_Images/` (selalu `tiff32/`) |
| `--reflectance` | uint16 reflektansi, siap untuk Pix4D → `Reflectance_Calibrated_Images/` |

RGB
kamera utama hanya pernah memancarkan data yang telah di-debayering + pratinjau — radian/reflektansi per-band tidak bermakna bagi sensor broadband, sehingga pengaturan tersebut tidak berpengaruh bagi mereka.Survey3
`.raw` mengabaikan pengaturan tersebut dan mengikuti jalur reflektansi/target standar.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (default `auto`) memilih referensi reflektansi: `auto` membuat [target kalibrasi](calibration-targets.md) dalam bingkai yang lolos QA sebagai referensi absolut dan beralih ke pembagian cahaya turun sensor DAQ (ρ = π·L/E) jika tidak ada target; `target` bersifat ketat (tanpa penggantian DAQ); `daq` mengutamakan data DAQ. Pemindaian target yang diukur per satuan dapat disediakan dengan `--target-reflectance-dir`.

{% hint style="info" %}
**Membaca piksel reflektansi:**nilai DN yang berarti ρ = 1,0 bersifat**per-sumber** — File LATTICE menandai `Chloros:PixelScale=32768` dalam XMP; fileSurvey3
menggunakan 65535 (dan tidak menyertakan tag `Chloros:*`). Baca tag tersebut dan bagi dengan nilainya, bukan dengan mengasumsikan nilai konstan. Detail serta satu kasus tepi tanpa skala yang disengaja terdapat di [ReferensiCLI
](reference/cli-reference.md).
{% endhint %}

**Pemrosesan selalu dimulai dari `raw`.** Produk turunan (ekspor debayered/radiance/reflectance) tidak pernah dimasukkan kembali melalui pipa pemrosesan — mengimpor ulang dan memprosesnya akan menerapkan perhitungan kalibrasi dua kali, sehinggaChloros
melewatkannya dan memberi tahu hal tersebut. `--input-level` adalah jalan keluar yang disengaja ketika Anda benar-benar perlu memaksa titik masuk.

***

## Saat Proses Gagal

Mulai versi 1.2.0, `process` akan gagal dengan jelas alih-alih &quot;berhasil&quot; tanpa menampilkan hasil apa pun:

* Eksekusi yang **meminta produk tetapi tidak menulis apa pun**— hanya `project.json` dan `calibration_data.json` — akan menampilkan pesan `Processing finished but wrote no image products.` dan**keluar dengan nilai non-nol**, sehingga skrip dapat mendeteksinya. Penyebab umumnya: folder input tidak dikenali sebagai penangkapan (periksa tata letak dan `--input-level`), atau setiap produk yang diminta tidak dapat diterapkan untuk kamera-kamera tersebut (misalnya, meminta radian/reflektansi dari kamera yang hanya mendukungRGB
).
* **Eksekusi khusus metadata saja** (semua produk dinonaktifkan, tanpa `--indices`) tetap dianggap berhasil — keluaran gambar kosong adalah hasil yang benar dalam kasus ini.
* Jalankan kembali dengan `--verbose` dan periksa log backend untuk baris `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, yang menjelaskan alasan pengabaian per kamera.

Kode keluar: `0` berhasil · `1` kegagalan umum · `2` kesalahan argumen · `130` terputus oleh Ctrl+C.

***

## Indeks Vegetasi

Jalankan `--indices` dengan satu atau lebih nama preset; setiap indeks akan disimpan dalam folder `<INDEX>_Index_Images/` masing-masing:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 nama preset yang diterima oleh `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Terdapat tiga daftar indeks — jangan sampai tertukar.**Menu tarik-turun Pengaturan Proyek pada GUI memiliki 27 rumus (ditambah `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — kelima rumus tersebut hanya tersedia di antarmuka pengguna dan**tidak** berlaku untuk `--indices`). Perintah live/offline `lattice index --preset` menggunakan daftar 22 preset terpisah miliknya sendiri. Rumus dan perhitungan band didokumentasikan dalam [Rumus Indeks Multispektral](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensor Cahaya DAQ: Gambaran Singkat

Keluarga `daq pool-*` mengendalikan sensor spektrMAPIR
(DAQ-U melalui USB, DAQ-M melalui BLE, DAQ-E melalui Ethernet) melalui kolam persisten backend — antarmuka pengguna (GUI),CLI
, danSDK
semuanya berbagi satu handle aktif. **`pool-*` adalah jalur DAQ yang didukung dalam paket yang dikirimkanCLI
**; subperintah `daq` lain yang mungkin Anda temui hanyalah permukaan internal khusus sumber (MAPIR
) dan akan keluar dengan pesan kesalahan eksplisit yang mengarahkan Anda ke `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` tanpa `--duration` berjalan hingga `pool-record --stop`; direktori keluaran default adalah `~/Documents/DAQ Live View/` **di mesin backend**. Profil koreksi batas dipilih pada saat koneksi (`--cap-id`, default backend `sunshine_cosine`) dan dapat diganti secara langsung dengan profil `pool-set-cap` — profil batas atas dan rentang kalibrasi sensor dijelaskan dalam bab DAQ pada manual ini.

{% hint style="warning" %}
**DAQ-E pada host multi-NIC:** proses penemuan otomatis `pool-connect --eth` pertama setelah boot dapat gagal meskipun sensor dalam kondisi baik. `--eth-host <ip-or-hostname>` adalah bentuk yang andal — gunakan opsi ini setiap kali proses penemuan tidak berhasil.
{% endhint %}

***

## Kamera LATTICE, PTP &amp; Otomatisasi Proyek

Keluarga perintah `lattice` (lebih dari 45 subperintah) mencakup seluruh proses kerja kamera LATTICE: penemuan, pengambilan gambar tunggal, susunan sinkronisasi berkelanjutan dengan alur koneksi smart-prep antarmuka pengguna (GUI), pratinjau browser langsung, penyelarasan, perhitungan indeks, dan diagnostik host-NIC. Contoh:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Selain itu: `chloros-cli time-sync` melaporkan tentang grandmaster PTP yang dijalankan oleh hostChloros
(kamera LATTICE dan sensor DAQ-E bertindak sebagai slave untuk cap waktu lintas perangkat), dan `chloros-cli project` membuka proyekChloros
yang telah disimpan dan mengendalikan kamera, array, serta sensornya secara headless — termasuk resep perekaman YAML yang diotomatisasi.

Ketiga keluarga ini (`lattice`, `project`, `daq pool-*`) juga merupakan satu-satunya yang mendukung `CHLOROS_BACKEND_URL` untuk mengendalikan backend **jarak jauh**; perintah inti selalu ditujukan ke mesin lokal.

Panduan lengkap tersedia di bab-bab LATTICE dalam manual ini; setiap opsi tercantum dalam [ReferensiCLI
](reference/cli-reference.md).

***

## Pemecahan Masalah: 5 Teratas

| Gejala | Solusi |
| --- | --- |
| `Login required`, atau pekerjaan terjadwal macet di prompt `Email:` | Jalankan `chloros-cli login EMAIL 'PASSWORD'` sekali pada mesin ini — perintah tanpa sesi yang disimpan dalam cache akan dijalankan secara interaktif alih-alih langsung gagal. |
| `backend unreachable` | Jalankan aplikasi desktopChloros
, atau jalankan biner backend secara langsung (`chloros-backend`). Jika Anda mengarahkan `lattice`/`project`/`daq pool-*` ke backend jarak jauh, periksa `CHLOROS_BACKEND_URL`. |
| Koneksi array diblokir: `FRAMES WILL DROP` / `Reduce ROI to enable` | Cincin penerimaan NIC host diatur ulang ke pengaturan default — penyebab utama rig yang sebelumnya berfungsi dengan baik tiba-tiba menolak terhubung, biasanya setelah pembaruan driver NIC. Jalankan perintah `chloros-cli lattice network --fix` dari terminal **dengan hak akses tinggi** (atau atur `ReceiveBufferLen=256`, `PendingReceives=64`); lihat bagian *Pengaturan &amp; Penyempurnaan NIC Host* dalam referensi. |
| Subperintah `daq` keluar: &quot;memerlukan paket daq lengkap…&quot; | Hal ini diharapkan pada build yang telah dikirim —CLI
yang telah dikompilasi hanya menyediakan keluarga perintah `daq pool-*`, yang mencakup perintah connect, stream, record, dan pemilihan cap. Gunakan `pool-*` (atau `chloros_sdk.connect_daq_sensor()` dariPython
). |
| Jetson menampilkan peringatan swap sebelum folder berukuran besar | Tambahkan swap berbasis file —CLI
menampilkan perintah `fallocate`/`swapon` yang tepat untuk dijalankan. |

***

## Mendapatkan Bantuan

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Setiap opsi, setiap subperintah:** [CLI
Referensi](reference/cli-reference.md)
* **Setara denganPython
:** [Python
SDK
](api-python-sdk.md) dan [SDK
Referensi](reference/sdk-reference.md)
* **Dukungan:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

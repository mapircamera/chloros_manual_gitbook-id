# CLI: Baris Perintah

> **Referensi lengkap:**[Referensi CLI](reference/cli-reference.md) mendokumentasikan**setiap opsi dari setiap subperintah** dan dioptimalkan untuk asisten AI — tempelkan URL-nya ke asisten Anda dan minta perintah yang berfungsi: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tips untuk alat AI:** setiap halaman dari manual ini tersedia dalam format Markdown mentah dengan menambahkan `.md` ke URL-nya (misalnya `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), dan `https://mapir.gitbook.io/chloros/llms.txt` mengindeks seluruh panduan ini untuk digunakan oleh LLM.

> **Ingin asisten Anda menjalankan Chloros sendiri?** `chloros-mcp`, sebuah server MCP yang diinstal bersama Python SDK, kompatibel dengan Claude Desktop, Claude Code, Cursor, dan klien MCP lainnya: lihat [Menggunakan Chloros dengan Asisten AI](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Apa Itu CLI


`chloros-cli` adalah antarmuka baris perintah (CLI
) untuk mesin pemrosesan yang sama yang digunakan oleh aplikasi desktop Chloros
. Ini adalah klien HTTP
tipis di atas backend Chloros
(server lokal di `127.0.0.1:5000`) — sebagian besar perintah memulai backend secara otomatis, sehingga satu panggilan `chloros-cli process …` sudah cukup untuk sebuah skrip. Backend kemudian tetap berjalan untuk perintah berikutnya: backend akan berhenti pada `chloros-cli logout` (kecuali jika aplikasi sedang terbuka atau proses eksekusi atau kompilasi sedang berlangsung), pada `chloros-cli backend stop`, atau secara otomatis setelah 30 menit tanpa adanya klien, proses eksekusi, atau perangkat yang terhubung. `chloros-cli backend status` menampilkannya.

Aplikasi ini berjalan di **Windows
10/11 (x64)**dan**Linux
(x86_64, serta NVIDIA Jetson arm64 pada JetPack 6)**, di terminal mana pun, tanpa memerlukan antarmuka grafis (GUI). Verifikasi instalasi Anda dengan:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

Ringkasan keluarga perintah:

* **Pemrosesan &amp; akun** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 bahasa — lihat [Bahasa yang Didukung](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (hanya Linux
/Jetson)
* **Perangkat keras langsung** — `lattice` (kontrol kamera LATTICE, 45+ subperintah), `daq pool-*` (sensor cahaya DAQ), `time-sync` (PTP)
* **Otomatisasi** — `project` (menjalankan proyek Chloros
yang disimpan tanpa antarmuka grafis, termasuk resep perekaman YAML)

Opsi global yang perlu diketahui: `--port N` (port backend, default `5000`), `-v/--verbose`, `--restart` (memaksa backend untuk memulai ulang), `--backend-exe PATH`. Lihat [Referensi CLI
](reference/cli-reference.md) untuk daftar lengkapnya.

***

## Instalasi

CLI
**termasuk dalam penginstal Chloros** di setiap platform — tidak ada unduhan CLI
terpisah. Unduh penginstal dari halaman [Unduh](download.md).

### Windows


Penginstal menempatkan CLI
di:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

dan menambahkan folder tersebut ke sistem Anda `PATH` sekali saja (pembaruan juga akan menghapus entri duplikat yang ditinggalkan oleh penginstal sebelumnya) — **buka terminal baru**setelah menginstal agar `PATH` yang diperbarui terdeteksi. Penginstal juga menempatkan skrip peluncur (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) di direktori akar instalasi serta**Chloros
CLI
** di menu Start, yang masing-masing akan membuka terminal dengan `chloros-cli` siap digunakan.

### Linux


Instal `.deb` untuk arsitektur Anda menggunakan `apt` (bukan `dpkg -i`, yang melewatkan persyaratan yang direkomendasikan oleh Python
SDK
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Ini akan menginstal `chloros-cli` hingga `/usr/bin/chloros-cli` (sudah ada di `PATH`), program backend dan CLI
yang diekstrak di `/usr/lib/chloros/backend/` dan `/usr/lib/chloros/cli/` (`/usr/lib/chloros/chloros-backend` adalah peluncur yang menjalankan backend), runtime Arena SDK
yang dibutuhkan backend untuk kamera LATTICE, serta Python
SDK
untuk sistem `python3`. Lihat [Instalasi Linux
](linux/linux-installation.md) untuk detailnya.

### Verifikasi

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Masuk &amp; Lisensi

Akses CLI (dan Python
SDK
) memerlukan **paket Chloros
+ berbayar**— semua tingkatan berbayar memilikinya; tingkatan gratis tidak. Batasan ini diterapkan**di sisi server** oleh backend, bukan oleh biner CLI
: panggilan saat tidak masuk akan ditolak dengan kode `401 AUTH_REQUIRED`, dan panggilan yang dilakukan saat login pada tingkatan gratis akan ditolak dengan kode `403 PLAN_UPGRADE_REQUIRED`, baik berasal dari `chloros-cli`, SDK
, maupun klien HTTP
yang dibuat sendiri. Lakukan upgrade di [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Masuk **sekali per mesin**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Kata sandi dengan karakter khusus**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` akan rusak oleh shell (CLI
mendeteksi hal ini pada kode status 401 dan melakukan percobaan ulang otomatis, tetapi tanda kutip tunggal sepenuhnya menghindari masalah ini).
{% endhint %}

Sesi disimpan dalam cache di `~/.chloros/user_session.json` dan tetap berfungsi secara offline selama masa tenggang paket (30 hari untuk paket bulanan, hingga masa berlaku habis untuk paket tahunan). `chloros-cli status` tetap berfungsi bahkan tanpa paket berbayar, sehingga alasan penolakan selalu terlihat.

{% hint style="danger" %}
**Ingin menjadwalkan pekerjaan headless? Silakan masuk terlebih dahulu.**Perintah yang memunculkan backend (`process`, `status`, `export-status`, …) yang dijalankan**tanpa sesi yang disimpan dalam cache**tidak akan segera gagal — perintah tersebut akan beralih ke prompt interaktif `Email:` / `Password:` pada stdin. Oleh karena itu, tugas cron atau langkah CI yang dijalankan tanpa pengawasan akan**terhenti sambil menunggu masukan**. Jalankan `chloros-cli login EMAIL 'PASSWORD'` sekali pada mesin tersebut sebelum menjadwalkan apa pun.
{% endhint %}

***

## Proses Pengolahan Pertama Anda

Arahkan `process` ke folder hasil penangkapan — program ini akan mendeteksi secara otomatis Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng`, atau kombinasi keduanya:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Aliran kemajuan ditampilkan secara langsung per utas pipa (Deteksi, Analisis, Pemrosesan, Ekspor), dan proses yang berhasil berakhir dengan melaporkan berapa banyak produk gambar yang telah ditulis: `Image products written: N`, diikuti oleh `(plus M clamp masks)` jika proses tersebut juga menulis [masker clamp](output-image-formats.md#clamp-masks) (masker ini tidak dihitung sebagai produk).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Jika backend Chloros
sudah berjalan (misalnya milik aplikasi desktop), `process` akan menggunakannya dan membuka proyeknya di sana. Proses ini akan ditolak selama aplikasi tersebut masih terbuka pada proyek yang dibuka oleh aplikasi: jalankan dari aplikasi, atau tutup proyek tersebut. Jika proses sebelumnya masih dalam tahap penyelesaian, sistem akan menunggu hingga 15 detik sebelum menolak. `--restart` memaksa penggunaan backend baru. Di Linux
, backend yang dijalankan sebagai layanan systemd tidak pernah di-restart: `process` menggunakannya apa adanya, dan di mana seharusnya backend tersebut di-restart (`--restart`, atau saat impor tidak menemukan apa pun), maka `sudo systemctl restart <unit>` akan menampilkan pesan dan keluar dari `1`.

### Tempat keluaran disimpan

`process` menulis ke dalam **folder proyek**, bukan ke folder input Anda:

* Tanpa `-o`: proyek dibuat di bawah folder proyek default Anda (dibagikan dengan antarmuka pengguna; kelola dengan `get-project-folder` / `set-project-folder`; jika tidak ada yang ditetapkan, `Documents/MAPIR/Chloros Projects` di folder pengguna Anda), diberi nama oleh `-n/--project-name` atau cap waktu (`YYYYMMDD_HHMMSS`) jika diabaikan.
* Dengan `-o PATH`: folder tersebut **adalah** folder proyek. Jika folder tersebut sudah berisi `project.json`, maka akan dibuat file saudara dengan akhiran `_1`/`_2`… yang diberi akhiran, alih-alih menimpa yang sudah ada. Jalur relatif `PATH` diambil relatif terhadap folder tempat Anda menjalankan `chloros-cli`.

{% hint style="info" %}
**Jalur relatif (diubah pada versi 1.3.0).** Setiap jalur yang dibuka oleh backend — `-o`/`--output` pada perintah apa pun (termasuk perintah penangkapan default `output`), `--input`, `--burst-dir`, `--profile`, `--save`, dan `--mask` — akan diselesaikan berdasarkan folder tempat Anda menjalankan `chloros-cli`, sehingga `-o captures/` menulis ke `<current folder>/captures/`. Pada versi 1.2.0, backend menyelesaikan jalur tersebut di folder instalasinya sendiri, yang gagal dengan pesan &quot;Akses ditolak&quot;. Terhadap backend jarak jauh (`CHLOROS_BACKEND_URL`), jalur dikirim sesuai yang diketik dan diselesaikan di mesin tersebut.
{% endhint %}

{% hint style="warning" %}
**Windows
: jaga agar folder proyek tetap pendek kecuali jika jalur panjang diaktifkan.** Dengan jalur panjang Windows
dinonaktifkan (pengaturan default Windows
), tidak ada jalur file yang boleh lebih panjang dari 260 karakter, dan produk berada hingga sekitar 130 karakter di bawah folder proyek. `process` memeriksa hal ini sebelum dimulai: folder proyek yang terlalu dalam akan ditolak dengan pesan **&quot;Folder proyek terlalu dalam untuk komputer ini&quot;**, folder proyek terpanjang yang masih muat, dan cara mengaktifkan jalur panjang — tidak ada yang diproses. Gunakan `-o` (atau `set-project-folder`) yang lebih pendek, atau aktifkan jalur panjang Windows
(opsi **Aktifkan jalur panjang Windows** pada penginstal, yang dicentang secara default) dan mulai ulang Chloros
.
{% endhint %}

Di dalam proyek, produk dikelompokkan **berdasarkan kamera, lalu berdasarkan format file**:

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

Folder kamera adalah `LATT-<sensor>-<lens>-F<filter>` untuk LATTICE (sesuai dengan EXIF pengambilan gambar `Model`) dan `<model>_<filter>` (misalnya `Survey3N_RGN`) untuk Survey3
. Folder format mengikuti `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, atau `tiff32` untuk `TIFF (32-bit, Percent)`.

Untuk kamera multispektral M3C, folder `clamp_masks/` juga dapat muncul di dalam folder format, di samping folder-folder produk (misalnya `tiff16/clamp_masks/Reflectance_Calibrated_Images/` dan `tiff32/clamp_masks/Radiance_Images/`). Folder ini berisi masker per-piksel dari produk radian dan reflektansi di mana pemisahan pita membatasi nilai negatif menjadi nol — lihat [Masker pembatas](output-image-formats.md#clamp-masks). Tidak ada folder produk yang berisi masker.

{% hint style="info" %}
**Setiap produk yang diekspor tetap menggunakan nama berkas SUMBER.**Ekspor radian dari `capture_..._raw.tif` tetap bernama `capture_..._raw.tif` — hanya saja berkas tersebut berada di dalam `tiff32/Radiance_Images/`.**Folderlah yang mengidentifikasi produk, bukan nama berkas**, jadi gunakan glob untuk direktori, bukan untuk sufiks `*radiance*`.
{% endhint %}

### Opsi yang akan Anda gunakan

| Opsi | Default | Fungsinya |
| --- | --- | --- |
| `-o, --output PATH` | folder proyek default | Lokasi folder proyek (lihat di atas). |
| `-n, --project-name NAME` | cap waktu | Nama proyek. |
| `--format FMT` | `TIFF (16-bit)` | Salah satu dari `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, atau `JPG (8-bit)`. Chloros
tidak melakukan peningkatan resolusi: pada semua gambar 8-bit, format yang lebih dalam ditulis sebagai `TIFF (8-bit)`, dan hal ini disebutkan sekali. |
| `--indices NAME [NAME ...]` | tidak ada | Indeks vegetasi yang akan diekspor (lihat [Indeks Vegetasi](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neural, lebih lambat, kualitas tertinggi (Chloros
+, memerlukan GPU CUDA: tanpa GPU CUDA, proses akan menggunakan `standard`; ditolak untuk tangkapan LATTICE, yang selalu menggunakan `standard`). |
| `--vignette / --no-vignette` | aktif | Koreksi vignette. |
| `--reflectance / --no-reflectance` | aktif | Kalibrasi reflektansi; untuk LATTICE, ini juga berfungsi sebagai tombol pengaktif/penonaktif produk reflektansi. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Memaksa titik masuk pipeline untuk berkas TIFF LATTICE. |

Untuk hal-hal lainnya — penyetelan deteksi target, PPK, pin eksposur, bendera penyelarasan array — lihat [bagian `process` dari Referensi CLI
](reference/cli-reference.md).

***

## Memilih Apa yang Akan Diekspor (Produk LATTICE)

Pemrosesan LATTICE menyebar ke **setiap produk yang berlaku dalam satu kali proses**. Empat tombol pengaturan per produk semuanya**diaktifkan secara default**; gunakan formulir `--no-` untuk menonaktifkan salah satunya:

| Tombol | Produk |
| --- | --- |
| `--debayered` | Demosaik linier → `Debayered_Images/`: offset gelap dari paket kalibrasi dihilangkan, kemudian dilakukan demosaik yang memperhatikan tepi, tanpa koreksi linearitas, flat field, atau unmix — file yang sama dengan yang disimpan oleh pengambilan gambar pada tingkat `debayered`. Tetap dalam orientasi asli sensor (tersimpan dalam XMP). |
| `--preview` | Pratinjau tampilan (keseimbangan putih + gamma; peregangan warna palsu untuk multispektral) → `Preview_Images/` |
| `--radiance` | radian float32, W/m²/sr/nm → `Radiance_Images/` (selalu `tiff32/`) |
| `--reflectance` | uint16 reflektansi, siap untuk Pix4D → `Reflectance_Calibrated_Images/` |

Kamera utama RGB
hanya memancarkan data yang telah di-debayering + pratinjau — radian/reflektansi per-band tidak bermakna untuk sensor broadband, sehingga tombol-tombol tersebut tidak berfungsi untuk kamera tersebut. Survey3
`.raw` mengabaikan tombol-tombol tersebut dan mengikuti jalur reflektansi/target standar.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (default `auto`) memilih referensi reflektansi: `auto` menjadikan [target kalibrasi](calibration-targets.md) dalam bingkai yang lulus QA sebagai referensi absolut dan beralih ke pembagian cahaya turun sensor cahaya DAQ (ρ = π·L/E) jika tidak ada target (target yang lulus QA dengan bendera akan bersaing dengan DAQ dalam skor pengukuran); `target` bersifat ketat (tidak ada penggantian oleh DAQ); `daq` mengutamakan DAQ. Pemindaian target terukur per unit dapat disediakan dengan `--target-reflectance-dir`.

{% hint style="info" %}
**Membaca piksel reflektansi:**nilai DN yang berarti ρ = 1,0 bersifat**per-sumber** — berkas LATTICE menyertakan tag `Chloros:PixelScale=32768` dalam XMP; Berkas Survey3
menggunakan nilai 65535 dan tidak menyertakan tag `Chloros:PixelScale` (namun menyertakan tag pemrosesan `Chloros:*` lainnya). Baca tag tersebut dan bagi dengan nilainya, bukan dengan mengasumsikan nilai konstan. Rincian dan satu kasus tepi tanpa skala yang disengaja terdapat dalam [Referensi CLI
](reference/cli-reference.md).
{% endhint %}

**Pemrosesan selalu dimulai dari `raw`.** Produk turunan (ekspor debayered/radiance/reflectance) tidak pernah dimasukkan kembali ke dalam alur kerja — mengimpor ulang dan memprosesnya akan menerapkan perhitungan kalibrasi dua kali, sehingga Chloros
melewatkannya dan memberi tahu hal tersebut. `--input-level` adalah jalan keluar yang disengaja ketika Anda benar-benar perlu memaksa titik masuk.

***

## Saat Proses Gagal

Mulai versi 1.2.0, `process` akan gagal secara jelas alih-alih &quot;berhasil&quot; tanpa menampilkan hasil apa pun:

* Eksekusi yang **meminta produk tetapi tidak menulis apa pun**— hanya `project.json` dan `calibration_data.json` — akan menampilkan pesan `Processing finished but wrote no image products.` dan**keluar dengan nilai non-nol**, sehingga skrip dapat mendeteksinya. Penyebab umumnya: folder input tidak dikenali sebagai hasil pengambilan (periksa tata letak dan `--input-level`), atau setiap produk yang diminta tidak dapat diterapkan untuk kamera-kamera tersebut (misalnya, meminta radiance/reflectance dari kamera yang hanya mendukung RGB
).
* **Eksekusi khusus metadata saja** (semua produk dinonaktifkan, tanpa `--indices`) tetap dianggap berhasil — keluaran gambar kosong adalah hasil yang benar dalam kasus ini.
* Jalankan kembali dengan `--verbose` dan periksa log backend untuk baris `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, yang menjelaskan alasan pengabaian per kamera.
* Pada Windows
dengan opsi jalur panjang dinonaktifkan, folder proyek yang terlalu dalam untuk batas jalur 260 karakter akan ditolak **sebelum** pemrosesan dimulai, dengan folder proyek terpanjang yang masih muat (lihat [Tempat penyimpanan hasil](#where-the-outputs-land)).
* Perintah perekaman (`lattice capture`, `lattice array-capture`, dan keluarga perintah perekaman lainnya) mencantumkan setiap file yang tidak dapat ditulis dan keluar dengan kode `1`; perekaman berulang akan berhenti ketika drive penuh.

Kode keluar: `0` berhasil · `1` kegagalan umum · `2` kesalahan argumen · `130` terputus oleh Ctrl+C.

***

## Indeks Vegetasi

Jalankan `--indices` dengan satu atau lebih nama preset; setiap indeks akan disimpan dalam folder `<INDEX>_Index_Images/` tersendiri:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

22 nama preset yang diterima oleh `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Nama yang belum tercantum dalam proyek ditambahkan ke dalamnya. Nama yang bukan preset, atau yang tidak dapat dibuat oleh rentang kamera, dilewati dengan peringatan yang menyebutkan namanya. Jika CLI
tidak dapat membaca daftar indeks proyek saat ini, daftar tersebut akan dibiarkan apa adanya dan muncul peringatan, bukan diganti.

{% hint style="warning" %}
**Terdapat tiga daftar indeks — jangan sampai tertukar.**Menu tarik-turun Pengaturan Proyek di GUI memiliki 27 rumus (menambahkan `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — kelima rumus tersebut hanya tersedia di antarmuka pengguna dan**tidak** berlaku untuk `--indices`). Perintah live/offline `lattice index --preset` menggunakan daftar 22 preset tersendiri. Rumus dan perhitungan band didokumentasikan dalam [Rumus Indeks Multispektral](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensor Cahaya DAQ: Pengenalan Singkat

Keluarga `daq pool-*` mengendalikan sensor spektral DAQ MAPIR
(DAQ-U melalui USB, DAQ-M melalui BLE, DAQ-E melalui Ethernet) melalui pool persisten backend — GUI, CLI
, dan SDK
semuanya berbagi satu handle aktif. **`pool-*` adalah jalur DAQ yang didukung dalam CLI
yang disertakan**; subperintah `daq` lain yang mungkin Anda temui hanyalah permukaan internal MAPIR
yang hanya berfungsi sebagai sumber dan akan keluar dengan pesan kesalahan eksplisit yang mengarahkan Anda ke `pool-*`.

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

`pool-record` tanpa `--duration` berjalan hingga `pool-record --stop`; direktori keluaran default adalah `~/Documents/DAQ Live View/` **di mesin backend**. Profil koreksi batas dipilih pada saat koneksi (`--cap-id`, default backend `sunshine_cosine`) dan dapat diganti secara langsung dengan `pool-set-cap` — profil batas dan rentang kalibrasi sensor dibahas dalam bab DAQ pada manual ini.

{% hint style="warning" %}
**DAQ-E pada host multi-NIC:** proses penemuan otomatis `pool-connect --eth` pertama setelah boot dapat gagal meskipun sensor dalam kondisi baik. `--eth-host <ip-or-hostname>` adalah opsi yang andal — gunakan opsi ini setiap kali proses penemuan tidak berhasil.
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

Selain itu: `chloros-cli time-sync` melaporkan status grandmaster PTP yang dijalankan oleh host Chloros
(kamera LATTICE dan sensor DAQ-E bertindak sebagai slave untuk cap waktu lintas perangkat), dan `chloros-cli project` membuka proyek Chloros
yang telah disimpan serta mengendalikan kamera, array, dan sensornya secara headless — termasuk resep perekaman YAML yang diotomatisasi.

Ketiga keluarga ini (`lattice`, `project`, `daq pool-*`) juga merupakan satu-satunya yang mendukung `CHLOROS_BACKEND_URL` untuk mengendalikan backend **jarak jauh**; perintah inti selalu ditujukan ke mesin lokal.

Panduan lengkap tersedia di bab-bab LATTICE dalam manual ini; setiap opsi tercantum dalam [Referensi CLI
](reference/cli-reference.md).

***

## Pemecahan Masalah: 5 Teratas

| Gejala | Solusi |
| --- | --- |
| `Login required`, atau tugas terjadwal macet pada prompt `Email:` | Jalankan `chloros-cli login EMAIL 'PASSWORD'` sekali pada mesin ini — perintah tanpa prompt sesi yang disimpan dalam cache akan dijalankan secara interaktif alih-alih langsung gagal. |
| `backend unreachable` | Jalankan aplikasi desktop Chloros
, atau jalankan backend secara manual: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` di Windows
(folder `backend` di dalam folder instalasi Anda; folder tersebut tidak ada di `PATH`), `chloros-backend` di Linux
. `chloros-cli backend status` menunjukkan apakah salah satunya sedang berjalan; `lattice`/`project`/`daq pool-*` tidak pernah memulai layanan tersebut. Jika Anda mengarahkan layanan tersebut ke backend jarak jauh, periksa `CHLOROS_BACKEND_URL`. |
| Koneksi array terblokir: `FRAMES WILL DROP` / `Reduce ROI to enable` | Cincin penerimaan NIC host diatur ulang ke pengaturan default — penyebab utama sistem yang sebelumnya berfungsi menolak terhubung, biasanya setelah pembaruan driver NIC. Jalankan `chloros-cli lattice network --fix` dari terminal **dengan hak akses tinggi** (atau atur `ReceiveBufferLen=256`, `PendingReceives=64`); lihat bagian *Pengaturan &amp; Penyesuaian NIC Host* dalam referensi. |
| Subperintah `daq` keluar: &quot;memerlukan paket DAQ lengkap…&quot; | Gunakan perintah `daq pool-*`, yang mengendalikan sensor yang sama melalui backend: connect, stream, record, pemilihan cap, serta pengaturan jaringan untuk DAQ-E, restart, dan pembaruan firmware (atau `chloros_sdk.connect_daq_sensor()` dari Python
). |
| Jetson menampilkan peringatan swap sebelum folder berukuran besar | Tambahkan swap berbasis file — antarmuka baris perintah (CLI
) menampilkan perintah `fallocate`/`swapon` yang tepat untuk dijalankan. |

***

## Mendapatkan Bantuan

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Setiap opsi, setiap subperintah:** [Referensi CLI
](reference/cli-reference.md)
* **Setara Python
:** [Python
SDK
](api-python-sdk.md) dan [Referensi SDK
](reference/sdk-reference.md)
* **Dukungan:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)

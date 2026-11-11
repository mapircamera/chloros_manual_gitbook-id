# Pengaturan & Mode Pengambilan Gambar

Proses pengambilan gambar di tab "Kamera" dikendalikan oleh satu tombol merah **Capture All**dan satu panel**Capture Settings** yang menentukan hasil dari tombol tersebut: kamera mana saja yang terlibat, jenis ekspor apa yang disimpan oleh masing-masing kamera, serta apakah rana akan dipicu sekali, secara terus-menerus, atau dengan interval tertentu. Halaman ini mendokumentasikan seluruh alur kerja — konfigurasi, proses perekaman itu sendiri, lokasi penyimpanan file di disk, dan cara memproses ulang file tersebut menjadi produk yang telah dikalibrasi di kemudian hari. Kontrol kamera dan susunan kamera itu sendiri terdapat di [Pengaturan Kamera](camera-settings.md).

{% hint style="info" %}
**Proses pengambilan gambar memerlukan proyek yang terbuka.** Opsi "Capture All" dan ikon roda gigi "Capture Settings" dinonaktifkan hingga proyek dibuka ("Buat atau buka proyek untuk menyimpan hasil pengambilan gambar"). Setiap hasil pengambilan gambar disimpan di bawah folder proyek di `captures/`.
{% endhint %}

## Panel Pengaturan Pengambilan Gambar

Buka panel ini dengan ikon **roda gigi di sebelah "Capture All"**pada daftar kamera di bilah samping, atau dengan tombol**"Open Capture Settings…"** di bagian bawah panel pengaturan per-kamera mana pun. Judul panel bertuliskan "Capture Settings" dengan tombol ← untuk kembali.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Pilihan Anda di sini — kamera yang disertakan, kotak centang per jenis, dan mode perekaman — disimpan **per proyek** dan dipulihkan saat Anda membukanya kembali.

### Mode perekaman

Tiga tombol mode di bagian atas panel:

| Mode | Fungsinya | Pengaturan tambahan (default) |
| --- | --- | --- |
| **Tunggal** *(default)* | Satu pengambilan gambar di semua kamera yang dipilih. | — |
| **Berkelanjutan**| Pengambilan gambar berturut-turut hingga kondisi penghentian terpenuhi. | Berhenti berdasarkan**Jumlah pengambilan gambar** (default 1) *atau* **Durasi pengambilan gambar** (default 10 detik; satuan: detik / menit / jam / hari). |
| **Interval**(timelapse) | Pengambilan gambar beruntun berdasarkan timer. |**Jumlah pengambilan gambar per interval**(default 1) ·**Setiap**N satuan (default 5 detik) ·**Selama** N satuan (default 1 menit). |

Dalam mode Berkelanjutan atau Interval, tombol Capture All berubah menjadi tombol **Stop (N)** saat proses berjalan, menghitung jumlah pengambilan gambar seiring berjalannya waktu. Drive yang penuh akan menghentikan proses (lihat [Alur Capture All](#the-capture-all-flow)).**Mode Continuous pada array berjalan di dalam backend (baru di versi 1.3.0).** Ketika setiap kamera dalam daftar termasuk dalam sebuah array (kamera mandiri disembunyikan atau dikecualikan), mode Continuous tidak lagi mengirim satu permintaan per penangkapan: backend Chloros menjalankan loop penangkapan itu sendiri, menulis satu grup yang disinkronkan sambil memilih yang berikutnya, dan tombol tersebut memeriksa jumlahnya. Jika ada kamera mandiri dalam daftar, Chloros melakukan penangkapan per pass seperti sebelumnya.

{% hint style="warning" %}
**Perekaman cepat di Windows: pastikan backend Chloros dikecualikan dari pemindaian Microsoft Defender.** Pemindaian real-time terhadap setiap file yang ditulis oleh perekaman berkecepatan tinggi menghabiskan sekitar 30% dari kecepatan perekaman dalam pengujian MAPIR (ratusan file per detik pada ROI kecil). Penginstal Windows menawarkan opsi **"Kecualikan backend Chloros dari pemindaian Microsoft Defender (perekaman kamera jauh lebih cepat; disarankan)"**, yang dicentang secara default, dan menghapus pengecualian tersebut saat penghapusan instalasi. Jika Anda mencabut centangnya, jalankan kembali penginstal dengan opsi tersebut dicentang, atau tambahkan pengecualian proses Defender untuk `chloros-backend.exe` (di dalam folder `backend` pada folder instalasi Chloros). Saat Defender menggunakan satu inti CPU atau lebih selama perekaman berkelanjutan (Continuous capture) pada suatu array, backend Chloros mencatatnya sekali dalam log-nya, dengan dua solusi yang sama ini.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Memilih kamera dan jenis ekspor

Teks bantuan pada panel ini merangkumnya: pilih kamera dan jenis ekspor mana yang akan dihasilkan oleh "Capture All" — semuanya aktif secara default, dan pilihan-pilihan tersebut disimpan bersama proyek ini.

* Tombol **Pilih semua / Jangan pilih apa pun** akan mengubah status kotak centang "Sertakan" pada setiap kamera sekaligus.
* **Tombol pengalih jenis ekspor massal**(dua baris tombol):**Semua Raw / Semua Debayered / Semua Pratinjau / Semua Radiance / Semua Reflektansi / Semua Indeks**. Masing-masing memiliki tiga status warna: hijau ✓ = aktif untuk setiap kamera yang mendukungnya, kuning – = aktif untuk beberapa kamera, abu-abu = tidak ada yang mendukung. Tombol pengalih dinonaktifkan jika tidak ada kamera yang terhubung yang mendukung jenis tersebut. Semuanya menjadi abu-abu saat Fastest Capture aktif.
* **Baris per kamera**: kotak centang "Sertakan", ditambah daftar yang dapat diperluas (▸/▾) berisi jenis ekspor yang berlaku untuk kamera tersebut dengan kotak centang individual. Baris tersebut menampilkan jumlah yang aktif seperti "4/6".

### Jenis ekspor dan kamera mana saja yang mendukungnya

Terdapat enam jenis ekspor: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Hanya jenis yang berlaku yang muncul pada baris masing-masing kamera:

| Jenis ekspor | Konten | RGB (FRGB) | Multispektral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaik Bayer (mono: pita tunggal) langsung dari sensor | ✓ | ✓ | ✓ |
| **Debayered** | Demosaik linier: offset gelap (DSNU) dari paket kalibrasi dihilangkan dari data mentah, kemudian dilakukan demosaik yang memperhatikan tepi — tanpa koreksi linearitas, flat field, atau unmix. Disimpan sebagai RGB 3-saluran TIFF (mono: skala abu-abu 1-saluran) | ✓ | ✓ | ✓ |
| **Pratinjau** | Rantai tampilan lengkap (keseimbangan putih + gamma sesuai profil kamera, ditambah pengaturan Highlights kamera RGB; multispektral: peregangan warna palsu) | ✓ | ✓ | ✓ |
| **Radian** | float32 W/m²/sr/nm melalui rantai radiometrik lengkap | — (tidak tersedia) | ✓ | ✓ |
| **Reflektansi** | uint16 ρ (32768 = 1,0) | — (tidak tersedia) | ✓ — hanya ditampilkan jika kamera memiliki sensor cahaya DAQ (miliknya sendiri atau diwarisi dari array-nya) | sama seperti multispektral |
| **Indeks** | Render indeks vegetasi (LUT) | — | ✓ — memerlukan ekspresi indeks yang diaktifkan dan tidak kosong pada kamera, serta tidak tersedia untuk anggota array gabungan (array memiliki satu indeks bersama) | — (indeks memerlukan ≥2 band; lihat [Kamera Mono & Indeks Vegetasi](mono-indices.md)) |

Radian dan reflektansi tidak pernah ditawarkan untuk kamera RGB — radian per-Bayer tidak bermakna untuk sensor fotometrik pita lebar.

### Pengambilan Tercepat

Tombol pengalih **⚡ Pengambilan Tercepat — hanya raw**(berwarna oranye saat aktif) menggantikan semua pilihan ekspor menjadi**hanya raw** — ditambah komposit indeks gabungan gratis untuk array — sehingga bingkai tersimpan secepat mungkin: perhitungan radian/reflektansi/tampilan dilewati sepenuhnya pada saat pengambilan.

{% hint style="info" %}
**Sebuah `.daq` tetap disimpan.** Saat sensor cahaya ditetapkan, Pengambilan Tercepat tetap mencatat pembacaan DAQ downwelling (ke dalam folder `daq/` dari hasil pengambilan) — sehingga produk radian, reflektansi, dan indeks semuanya dapat dibuat kemudian melalui pemrosesan ulang (lihat [Memproses ulang penangkapan](#re-processing-captures-into-calibrated-products)). Fastest Capture juga tidak merusak pilihan kotak centang Anda: matikan fitur ini dan pilihan tersebut akan kembali.
{% endhint %}

### Kontrol per-array

Setiap array yang terhubung memiliki kartu grup tersendiri di panel:

* **Kotak centang Include** (tiga status di seluruh anggota) dan nama array beserta mode tampilannya: "(digabung | terpisah)".
* Kotak centang **Aligned**(secara default**aktif**): menyesuaikan ekspor anggota dengan profil penyelarasan array sehingga ekspor terdaftar secara piksel di seluruh kamera. Data mentah tetap tidak di-warp tetapi membawa transformasi dalam metadata-nya. (Profil itu sendiri dihitung di [panel pengaturan array](camera-settings.md#alignment-co-registration).) File anggota array disimpan dalam orientasi asli masing-masing kamera, dengan pengaturan Cermin/Rotasi dicatat dalam file alih-alih diterapkan.
* Baris kamera anggota disusun bersarang di dalam kartu.

Kartu array juga menampung dua perekam. Anggap saja keduanya sebagai **pemantauan vs analisis**:

| Perekam | Tingkat | Apa yang direkam |
| --- | --- | --- |
| **● Merekam video indeks gabungan / ■ Hentikan perekaman** *(hanya untuk array gabungan)* | **Pemantauan** | Komposisi indeks gabungan langsung ke video pada 10 fps — 8-bit, resolusi pratinjau, LUT sudah terintegrasi. Membutuhkan proyek yang terbuka dan tampilan streaming langsung. Menampilkan bingkai + waktu yang telah berlalu selama perekaman. |
| **⦿ Rekam burst mentah / ■ Hentikan burst mentah** *(array apa pun)* | **Analisis**| Frame Bayer mentah pada laju pengambilan langsung (tanpa pemrosesan) ditambah manifes per-frame dan pembacaan `.daq`, ke dalam `captures/bursts/`. Setelah burst, tombol**Buat video** akan muncul: tombol ini memproses ulang burst secara offline menjadi video yang telah dikalibrasi — indeks gabungan dan/atau radian / reflektansi / indeks per kamera — ditambah TIFF opsional. Pembuatan indeks gabungan dimulai secara otomatis saat Anda menghentikan burst. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Alur

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Kerja "Capture All"

Tekan **Capture All** di daftar kamera pada bilah samping:

1. Setiap kamera yang disertakan, terlihat, dan tidak dalam keadaan jeda akan merekam dengan jenis ekspor yang dipilih. **Array memicu perekaman sebagai satu pemicu yang tersinkronisasi** (satu grup yang disinkronkan di seluruh anggotanya — lihat [Array Multi-Kamera](arrays.md)); kamera mandiri merekam secara individual.
2. Kamera yang tersembunyi (ikon mata) atau dalam keadaan dijeda akan dilewati. Sebuah array hanya akan sepenuhnya diblokir jika *semua* anggotanya tersembunyi atau dalam keadaan dijeda.
3. Setiap kali sensor cahaya ditetapkan, pembacaan DAQ downwelling yang sesuai disimpan sebagai berkas `.daq` di dalam folder `daq/` rekaman — bahkan untuk rekaman yang hanya berupa data mentah — sehingga produk radiometrik selalu dapat dihitung kemudian.
4. Tombol tersebut menampilkan kemajuan secara langsung — "Sedang merekam… selesai/total" — dan dalam mode Kontinu/Interval berubah menjadi **Berhenti (N)**. Setiap item perekaman memiliki batas waktu 300 detik.
5. Saat proses pengambilan selesai, pesan hasil singkat akan menampilkan **"Saved N files"**atau**"Saved N, F failed"**, ditambah "(S hidden/paused skipped)" jika ada kamera yang dilewati. Jika drive penuh, proses akan berhenti dan melaporkan**"Berhenti: drive penuh. Tersimpan N, F gagal"** — setiap file yang tidak dapat ditulis dihitung sebagai gagal.

## Lokasi penyimpanan hasil penangkapan

Hasil penangkapan disimpan di bawah proyek yang terbuka pada `<project>/captures/`. Setiap jenis ekspor disimpan di **subfolder tersendiri**, sehingga penangkapan multi-level tidak pernah mencampurkan jenis:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` adalah cap waktu penangkapan dan `<serial>` adalah nomor seri kamera. Pengambilan gambar mandiri diberi nama `capture_<ts>_SN<serial>_<level>`; pengambilan gambar array dari satu pemicu yang disinkronkan diberi nama `sync_<ts>_SN<serial>_<level>` dan **berbagi satu cap waktu untuk semua kamera dalam grup** (sufiks level dihilangkan jika sebuah kamera hanya menyimpan satu level).
* **Satu hal yang perlu diperhatikan:** level tampilan disimpan dalam folder bernama `preview/`, sedangkan nama file tetap `_display` — perbedaan folder dan sufiks hanya terjadi pada level tersebut.
* Level yang tidak dikenal akan disimpan di folder dengan nama level tersebut; jika subfolder tidak dapat dibuat, berkas akan ditulis ke direktori akar penangkapan alih-alih hilang.
* Berkas TIFF hasil penangkapan dikompresi tanpa kehilangan data (DEFLATE) secara default dan menyimpan metadata kalibrasi serta pemrosesan lengkapnya **di dalam XMP berkas tersebut** — tangkapan bersifat self-describing, tanpa berkas tambahan selain berkas `.daq` (dan `.daq.cameras.json`-nya).
* **Masker pembatas (fitur baru di versi 1.3.0).**Jika pemisahan spektral 3×3 harus membatasi suatu band menjadi 0 pada setidaknya satu piksel, produk radian, reflektansi, dan indeks akan mendapatkan masker per piksel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = pita pada kanal R produk, bit 1 = G, bit 2 = B); tag XMP produk `Chloros:UnmixClampMask` menunjuk ke sana. Masker-masker tersebut berada**di samping** folder produk, bukan di dalamnya, sehingga folder produk seperti `reflectance/` dapat diserahkan ke perangkat lunak fotogrametri apa adanya. Fitur impor bawaan Chloros melewati masker-masker tersebut.
* **Orientasi (1.3.0).** Produk Preview, Radiance, Reflectance, dan Index dari kamera mandiri diputar sesuai pengaturan Mirror/Rotation-nya; sedangkan Raw dan Debayered tetap seperti yang dibaca oleh sensor. Semua file anggota array tetap dalam format aslinya. Setiap berkas mencatat pengaturan tersebut dalam XMP-nya, sehingga pemrosesan ulang akan menerapkannya dengan cara yang sama.
* **Kamera mana yang menjadi sumber berkas `.daq` (1.3.0).** `.daq` dari suatu pengambilan gambar hanya dicocokkan, pada saat pemrosesan, dengan gambar dari kamera yang disebutkan dalam `.daq.cameras.json`-nya. Oleh karena itu, kamera mandiri yang tidak memiliki sensor cahaya sendiri tidak lagi meminjam pembacaan dari kamera lain — hubungkan sensor ke kamera tersebut, atau catat sensor di tab Sensor Cahaya. Perekaman sensor cahaya (dan setiap `.daq` tanpa file tersebut, termasuk tangkapan yang lebih lama) masih berfungsi untuk setiap kamera.

Ini adalah tata letak yang sama yang ditulis oleh `chloros-cli lattice capture` / `array-capture` ke direktori `-o` mereka — didokumentasikan dalam [Referensi CLI § Tampilan folder tangkapan](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Memproses ulang tangkapan menjadi produk yang telah dikalibrasi

Frame mentah yang ditangkap ditambah berkas `.daq` yang disimpan adalah semua yang dibutuhkan oleh alur pemrosesan — inilah mengapa Fastest Capture aman digunakan untuk pekerjaan nyata.

* **GUI**: tambahkan folder tangkapan ke proyek ([Menambahkan File ke Proyek](../processing-images-gui/adding-files-to-a-project.md)) dan olah seperti biasa.
* **CLI**: arahkan `process` ke **akar folder tangkapan**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` biasanya hanya mengimpor folder yang Anda tentukan, tetapi jika folder tersebut tidak berisi gambar dan memiliki subfolder, maka subfolder tersebut akan diproses secara otomatis — sehingga subfolder tingkat dan folder `daq/` akan diambil sekaligus. Setiap penangkapan diimpor sebagai **satu gambar** dengan tingkat lainnya dilampirkan sebagai mode tampilan, bukan sebagai satu gambar per tingkat.

Menamai subfolder tingkat secara langsung (misalnya `…/captures/raw/`) juga berfungsi, tetapi meninggalkan folder `daq/` — salin folder tersebut bersamanya saat Anda-mengolah produk radiometrik dari `raw/`, jika tidak, pencocokan cap waktu tidak akan memiliki acuan untuk diselesaikan.

{% hint style="warning" %}
**Pemrosesan selalu dimulai dari `raw`.** Dalam setiap pengambilan data, bingkai mentah (raw frame) merupakan sumber pipa pengolahan; `debayered`, `radiance`, `reflectance`, dan `preview` muncul sebagai mode tampilan tetapi tidak pernah dikembalikan melalui pipa — memproses ulang produk turunan akan menerapkan kembali perhitungan vignette, warna, dan radian yang sudah tertanam dalam pikselnya, sehingga Chloros menolak daripada melakukan pemrosesan ganda. Render `index/` dan `composite/` sama sekali tidak diproses (keduanya merupakan output, bukan hasil tangkapan). Folder rekaman yang disimpan **tanpa** impor data mentah akan ditampilkan secara normal, tetapi `process` akan melewatinya dan memberi tahu hal tersebut; `--input-level {raw,debayered,processed}` adalah jalan keluar yang disengaja untuk memaksa titik masuk. Lihat [Referensi CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) untuk pesan "skip" yang tepat.
{% endhint %}

Dua perilaku lain yang perlu diketahui saat membuat skrip pemrosesan ulang:

* Eksekusi `chloros-cli process` yang meminta produk tetapi **tidak menghasilkan produk gambar**akan gagal dengan pesan kesalahan yang jelas dan keluar dengan nilai non-nol** — Anda tidak akan pernah mendapatkan eksekusi kosong yang diam-diam. Eksekusi yang berhasil melaporkan jumlah produknya. (Eksekusi yang sengaja hanya menghasilkan metadata tetap dianggap berhasil.)
* Ekspor yang telah diproses dan diimpor ulang tidak pernah mengambil slot data mentah dari tangkapan — data mentah asli selalu tetap menjadi sumber pipa.

## Setara CLI



Semua yang ada di halaman ini dapat dijalankan tanpa antarmuka grafis. Mode tangkapan GUI secara langsung sesuai dengan `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Tunggal | `chloros-cli lattice array-capture` |
| Berkelanjutan | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Perekaman Tercepat | `array-capture --fastest` |
| Kotak centang "Aligned" | `--aligned / --no-aligned` |
| Kotak centang jenis ekspor | `--processing LEVEL` atau `--levels L1,L2,…` (default `all`) |
| Rekam video indeks | `chloros-cli lattice array-record` |
| Rekam burst mentah / Buat video | `chloros-cli lattice array-burst` / `array-build-video` |

Tabel flag lengkap, opsi perekaman smart-AE yang stabil (`--smart`), dan model laju berkelanjutan dijelaskan dalam [Referensi CLI § Mode Perekaman, Perekam, & Pemrosesan Ulang Offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

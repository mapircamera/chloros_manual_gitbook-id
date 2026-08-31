# Pengaturan &amp; Mode Pengambilan Gambar

Proses pengambilan gambar di tab Kamera dikendalikan oleh satu tombol merah **Capture All**dan satu panel**Capture Settings** yang menentukan hasil dari tombol tersebut: kamera mana saja yang terlibat, jenis ekspor apa yang disimpan oleh masing-masing kamera, serta apakah rana akan dipicu sekali, secara terus-menerus, atau dengan interval tertentu. Halaman ini mendokumentasikan seluruh alur kerja — konfigurasi, proses pengambilan gambar itu sendiri, lokasi penyimpanan file di disk, dan cara memproses ulang file-file tersebut menjadi produk yang telah dikalibrasi di kemudian hari. Kontrol kamera dan susunan kamera itu sendiri terdapat di [Pengaturan Kamera](camera-settings.md).

{% hint style="info" %}
**Pengambilan gambar memerlukan proyek yang terbuka.** Opsi &quot;Capture All&quot; dan ikon roda gigi &quot;Capture Settings&quot; dinonaktifkan hingga proyek dibuka (&quot;Buat atau buka proyek untuk menyimpan hasil pengambilan gambar&quot;). Setiap hasil pengambilan gambar disimpan di bawah folder proyek di `captures/`.
{% endhint %}

## Panel Pengaturan Perekaman

Buka panel ini dengan ikon **roda gigi di sebelah &quot;Perekaman Semua&quot;**pada daftar kamera di bilah samping, atau dengan tombol**&quot;Buka Pengaturan Perekaman…&quot;** di bagian bawah panel pengaturan per-kamera mana pun. Bagian atas panel bertuliskan &quot;Pengaturan Pengambilan&quot; dengan tombol ← kembali.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Pilihan Anda di sini — kamera yang disertakan, kotak centang per jenis, dan mode pengambilan — disimpan **per proyek** dan dipulihkan saat Anda membukanya kembali.

### Mode perekaman

Tiga tombol mode di bagian atas panel:

| Mode | Fungsinya | Pengaturan tambahan (default) |
| --- | --- | --- |
| **Tunggal** *(default)* | Satu kali perekaman di semua kamera yang dipilih. | — |
| **Berkelanjutan**| Pengambilan gambar berturut-turut hingga kondisi penghentian terpenuhi. | Dihentikan berdasarkan**Jumlah pengambilan gambar** (default 1) *atau* **Durasi pengambilan gambar** (default 10 detik; satuan: detik / menit / jam / hari). |
| **Interval**(timelapse) | Pengambilan gambar berturut-turut berdasarkan timer. |**Jumlah pengambilan gambar per interval**(default 1) ·**Setiap**N satuan (default 5 detik) ·**Selama** N satuan (default 1 menit). |

Dalam mode Kontinu atau Interval, tombol Capture All akan berubah menjadi tombol **Stop (N)** saat sedang berjalan, menghitung jumlah pengambilan gambar seiring berjalannya proses.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Memilih kamera dan jenis ekspor

Teks bantuan pada panel ini merangkumnya: pilih kamera dan jenis ekspor mana yang dihasilkan oleh &quot;Capture All&quot; — semuanya aktif secara default, dan pilihan-pilihan tersebut disimpan bersama proyek ini.

* Tombol **Pilih semua / Hapus semua** akan mengubah status kotak centang &quot;Sertakan&quot; pada setiap kamera sekaligus.
* **Tombol pengalih jenis ekspor massal**(dua baris tombol):**Semua Raw / Semua Debayered / Semua Pratinjau / Semua Radiance / Semua Reflektansi / Semua Indeks**. Masing-masing memiliki tiga status warna: hijau ✓ = aktif untuk setiap kamera yang mendukungnya, kuning – = aktif untuk beberapa kamera, abu-abu = tidak ada. Tombol pengalih dinonaktifkan jika tidak ada kamera yang terhubung yang mendukung jenis tersebut. Semuanya menjadi abu-abu saat &quot;Fastest Capture&quot; aktif.
* **Baris per kamera**: kotak centang &quot;Sertakan&quot;, ditambah daftar yang dapat diperluas (▸/▾) berisi jenis ekspor yang berlaku untuk kamera tersebut dengan kotak centang individual. Baris tersebut menampilkan jumlah yang aktif seperti &quot;4/6&quot;.

### Jenis ekspor dan kamera mana saja yang mendukungnya

Terdapat enam jenis ekspor: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Hanya jenis yang berlaku yang muncul pada baris masing-masing kamera:

| Jenis ekspor | Konten | RGB (FRGB) | Multispektral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaik Bayer (mono: pita tunggal) langsung dari sensor | ✓ | ✓ | ✓ |
| **Debayered** | Demosaik linier (mono: skala abu-abu 1 saluran) | ✓ | ✓ | ✓ |
| **Pratinjau** | Rantai tampilan lengkap (keseimbangan putih + gamma sesuai profil kamera; multispektral: peregangan warna palsu) | ✓ | ✓ | ✓ |
| **Radiance** | float32 W/m²/sr/nm melalui rantai radiometrik lengkap | — (tidak tersedia) | ✓ | ✓ |
| **Reflektansi** | uint16 ρ (32768 = 1,0) | — (tidak tersedia) | ✓ — hanya ditampilkan jika kamera memiliki sensor cahaya DAQ (miliknya sendiri atau diwarisi dari array-nya) | sama seperti multispektral |
| **Indeks** | Render indeks vegetasi (LUT) | — | ✓ — memerlukan ekspresi indeks yang diaktifkan dan tidak kosong pada kamera, serta tidak tersedia untuk anggota array gabungan (array memiliki satu indeks bersama) | — (indeks memerlukan ≥2 band; lihat [Kamera Mono &amp; Indeks Vegetasi](mono-indices.md)) |

Radian dan reflektansi tidak pernah ditawarkan untuk kamera RGB — radian per-Bayer tidak bermakna untuk sensor fotometrik pita lebar.

### Pengambilan Tercepat

Tombol pengalih **⚡ Pengambilan Tercepat — hanya raw**(berwarna oranye saat aktif) menggantikan semua pilihan ekspor menjadi**hanya raw** — ditambah komposit indeks gabungan gratis untuk array — sehingga bingkai tersimpan secepat mungkin: perhitungan radian/reflektansi/tampilan dilewati sepenuhnya pada saat pengambilan gambar.

{% hint style="info" %}
**File `.daq` tetap disimpan.** Saat sensor cahaya ditetapkan, Pengambilan Tercepat tetap mencatat pembacaan DAQ downwelling di samping bingkai mentah — sehingga produk radian, reflektansi, dan indeks dapat dibuat nanti melalui pemrosesan ulang (lihat [Pemrosesan ulang pengambilan](#re-processing-captures-into-calibrated-products)). Fastest Capture juga tidak merusak pilihan kotak centang Anda: matikan fitur ini dan pilihan tersebut akan kembali.
{% endhint %}

### Kontrol per-array

Setiap array yang terhubung memiliki kartu grupnya sendiri di panel:

* **Kotak centang &quot;Sertakan&quot;** (tiga status di seluruh anggota) dan nama array beserta mode tampilannya: &quot;(digabung | terpisah)&quot;.
* Kotak centang **Aligned**(default**aktif**): menyesuaikan ekspor anggota dengan profil penyelarasan array sehingga ekspor terdaftar secara piksel di seluruh kamera. Data mentah tetap tidak disesuaikan tetapi membawa transformasi dalam metadata-nya. (Profil itu sendiri dihitung di [panel pengaturan array](camera-settings.md#alignment-co-registration-combined-only).)
* Baris kamera anggota disusun secara bersarang di dalam kartu.

Kartu array juga menampung dua perekam. Anggap saja keduanya sebagai **pemantauan vs analisis**:

| Perekam | Tingkat | Apa yang direkam |
| --- | --- | --- |
| **● Rekam video indeks gabungan / ■ Hentikan perekaman** *(hanya array gabungan)* | **Pemantauan** | Komposisi indeks gabungan langsung ke video pada 10 fps — 8-bit, resolusi pratinjau, LUT terintegrasi. Membutuhkan proyek yang terbuka dan tampilan langsung streaming. Menampilkan bingkai + waktu yang telah berlalu selama perekaman. |
| **⦿ Rekam burst mentah / ■ Hentikan burst mentah** *(array apa pun)* | **Analisis**| Frame Bayer mentah pada laju pengambilan langsung (tanpa pemrosesan) ditambah manifes per-frame dan pembacaan `.daq`, ke dalam `captures/bursts/`. Setelah burst, tombol**Buat Video** muncul: tombol ini memproses ulang burst secara offline menjadi video yang dikalibrasi — indeks gabungan dan/atau radian / reflektansi / indeks per kamera — ditambah TIFF opsional. Pembuatan indeks gabungan dimulai secara otomatis saat Anda menghentikan burst.

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

|## Alur

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Kerja “Capture All” Tekan **Capture All** di daftar kamera pada bilah samping:

1. Setiap kamera yang disertakan, terlihat, dan tidak dalam keadaan dijeda akan merekam dengan jenis ekspor yang dipilih. **Array memotret sebagai satu pemicu yang tersinkronisasi** (satu grup yang tersinkronisasi di seluruh anggota — lihat [Array Multi-Kamera](arrays.md)); kamera mandiri merekam secara individual.
2. Kamera yang tersembunyi (ikon mata) atau dalam keadaan dijeda akan dilewati. Sebuah array hanya akan sepenuhnya diblokir jika *semua* anggotanya tersembunyi atau dalam keadaan dijeda.
3. Setiap kali sensor cahaya ditetapkan, pembacaan DAQ downwelling yang sesuai disimpan sebagai berkas `.daq` bersama citra — bahkan untuk perekaman hanya dalam format mentah — sehingga produk radiometrik selalu dapat diperoleh kemudian.
4. Tombol ini menampilkan kemajuan secara langsung — &quot;Sedang merekam… selesai/total&quot; — dan dalam mode Kontinu/Interval berubah menjadi **Berhenti (N)**. Setiap item perekaman memiliki batas waktu 300 detik.
5. Saat proses pengambilan selesai, pesan singkat hasil akan menampilkan **&quot;Saved N files&quot;**atau**&quot;Saved N, F failed&quot;**, ditambah &quot;(S hidden/paused skipped)&quot; jika ada kamera yang dilewati.

## Lokasi Penyimpanan Hasil Pengambilan

Hasil pengambilan disimpan di bawah proyek yang terbuka pada `<project>/captures/`. Setiap jenis ekspor disimpan dalam **subfolder tersendiri**, sehingga pengambilan multi-level tidak pernah mencampurkan jenis:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` adalah cap waktu rekaman dan `<serial>` adalah nomor seri kamera. Rekaman mandiri diberi nama `capture_<ts>_SN<serial>_<level>`; perekaman array dari satu pemicu yang disinkronkan diberi nama `sync_<ts>_SN<serial>_<level>` dan **berbagi satu cap waktu untuk semua kamera dalam grup** (sufiks level dihilangkan ketika sebuah kamera hanya menyimpan satu level).
* **Satu hal yang perlu diperhatikan:** level tampilan disimpan dalam folder bernama `preview/`, sedangkan nama file tetap `_display` — folder dan sufiks hanya berbeda untuk level tersebut.
* Level yang tidak dikenal akan disimpan di folder dengan nama level tersebut; jika subfolder tidak dapat dibuat, berkas akan ditulis ke akar direktori captures daripada hilang.
* File TIFF hasil tangkapan dikompresi tanpa kehilangan data (DEFLATE) secara default dan menyimpan metadata kalibrasi serta pemrosesan lengkapnya **di dalam file XMP** — hasil tangkapan bersifat self-describing, tanpa file sidecar selain file `.daq`.

Tata letak ini sama dengan yang ditulis oleh `chloros-cli lattice capture` / `array-capture` ke direktori `-o` mereka — didokumentasikan dalam [Referensi CLI § Tampilan folder rekaman](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Memproses ulang rekaman menjadi produk yang telah dikalibrasi

Frame mentah yang ditangkap ditambah berkas `.daq` yang telah disimpan adalah semua yang dibutuhkan oleh alur pemrosesan — inilah mengapa &quot;Fastest Capture&quot; aman untuk pekerjaan nyata.

* **GUI**: tambahkan folder tangkapan ke proyek ([Menambahkan File ke Proyek](../processing-images-gui/adding-files-to-a-project.md)) dan proses seperti biasa.
* **CLI**: arahkan `process` ke**folder akar tangkapan**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` biasanya hanya mengimpor folder yang Anda tentukan, tetapi jika folder tersebut tidak berisi gambar dan memiliki subfolder, maka sistem akan menjelajah ke subfolder secara otomatis — sehingga subfolder tingkat dan berkas-berkas di direktori akar `.daq` akan diimpor sekaligus. Setiap penangkapan diimpor sebagai **satu gambar** dengan tingkat lainnya dilampirkan sebagai mode tampilan, bukan sebagai satu gambar per tingkat.

Menamai subfolder tingkat secara langsung (misalnya `…/captures/raw/`) juga berfungsi, tetapi akan meninggalkan berkas akar `.daq` — salin berkas-berkas tersebut bersamaan saat Anda membuat ulang produk radiometrik dari `raw/`, jika tidak, pencocokan cap waktu tidak akan memiliki referensi untuk diselesaikan.

{% hint style="warning" %}
**Pemrosesan selalu dimulai dari `raw`.**Dalam setiap pengambilan data, bingkai mentah merupakan sumber pipa pemrosesan; `debayered`, `radiance`, `reflectance`, dan `preview` disertakan sebagai mode tampilan tetapi tidak pernah dimasukkan kembali ke dalam pipeline — memproses ulang produk turunan akan menerapkan kembali perhitungan vignette, warna, dan radiance yang sudah tertanam dalam pikselnya, sehingga Chloros ditolak daripada diproses dua kali. Render `index/` dan `composite/` tidak pernah diproses sama sekali (keduanya adalah keluaran, bukan hasil tangkapan). Folder tangkapan yang disimpan**tanpa** impor data mentah akan ditampilkan secara normal, tetapi `process` akan melewatinya dan memberi tahu hal tersebut; `--input-level {raw,debayered,processed}` adalah jalan keluar yang disengaja yang memaksa adanya titik masuk. Lihat [Referensi CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) untuk pesan pengabaian yang tepat.
{% endhint %}

Dua perilaku lain yang perlu diketahui saat membuat skrip pemrosesan ulang:

* Eksekusi `chloros-cli process` yang meminta produk tetapi **tidak menghasilkan produk citra**akan gagal dengan jelas dan keluar dengan nilai non-nol** — Anda tidak akan pernah mendapatkan eksekusi kosong yang diam-diam. Eksekusi yang berhasil melaporkan jumlah produknya. (Eksekusi yang sengaja hanya menghasilkan metadata tetap dianggap berhasil.)
* Ekspor yang telah diproses dan diimpor ulang tidak pernah mengambil slot data mentah dari tangkapan — data mentah asli selalu tetap menjadi sumber pipa.

## Setara dengan CLI

Semua yang ada di halaman ini dapat dijalankan tanpa antarmuka grafis (headless). Mode penangkapan GUI secara langsung sesuai dengan `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Tunggal | `chloros-cli lattice array-capture` |
| Berkelanjutan | `array-capture --continuous [--count N] [--duration S]` |
| Interval | `array-capture --interval S [--duration S]` |
| Perekaman Tercepat | `array-capture --fastest` |
| Kotak centang yang sejajar | `--aligned / --no-aligned` |
| Kotak centang jenis ekspor | `--processing LEVEL` atau `--levels L1,L2,…` (default `all`) |
| Rekam video indeks | `chloros-cli lattice array-record` |
| Rekam burst mentah / Buat video | `chloros-cli lattice array-burst` / `array-build-video` |

Tabel bendera lengkap, opsi pengambilan gambar dengan smart-AE (`--smart`), dan model kecepatan konstan terdapat di [CLI Referensi § Mode Pengambilan Gambar, Perekam &amp; Pemrosesan Ulang Offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

# Referensi Chloros CLI

**Versi:**

1.3.0**Dibuat:**29 Juli 2026 19:19 ·**Direvisi:** 6 November 2026**Target Pembaca:** Dioptimalkan untuk penggunaan oleh LLM; mudah dibaca oleh manusia.**Cakupan:** Setiap subperintah yang berinteraksi dengan pengguna dari `chloros-cli`, beserta opsi dan contoh yang dapat disalin dan ditempelkan.

Dokumen ini merupakan referensi lengkap untuk alat baris perintah `chloros-cli` yang disertakan dalam MAPIR Chloros. Dokumen ini sengaja dibuat sedetail mungkin agar LLM (atau manusia) dapat menyusun alur kerja apa pun yang didukung dari daftar di bawah ini tanpa perlu memeriksa kode sumber.

Jika Anda hanya membutuhkan poin-poin penting, langsung ke:
- [Panduan Cepat Lima Menit](#five-minute-quickstart)
- [Alur Kerja Koneksi Pertama LATTICE Camera](#lattice-camera-first-connect-workflow)
- [Alur Kerja Koneksi Pertama DAQ Sensor](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Mode Pengambilan, Perekam, & Pemrosesan Ulang Offline](#capture-modes-recorders--offline-reprocess)

> **Mengendalikan Chloros dari asisten AI?** Claude Desktop, Claude Code, Cursor, dan klien MCP lainnya dapat menjalankan pemrosesan, kamera dan array LATTICE, serta sensor cahaya DAQ melalui `chloros-mcp`, server MCP yang terinstal bersama Python SDK (dan dengan paket .deb Linux, di `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. Pengaturan, opsi, dan alat tersedia di referensi SDK pada bagian [Server MCP (Asisten AI)](sdk-reference.md#mcp-server-ai-assistants). Asisten dengan shell juga dapat menjalankan setiap perintah di halaman ini secara langsung.

> **Perubahan pada versi 1.3.0** (yang akan terdeteksi oleh skrip atau alur kerja versi 1.2.0 yang sudah ada).
>
> **Perubahan yang mungkin perlu ditangani oleh skrip versi 1.2.0:**
>
> - **`--preset` berlaku.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply`, dan `align-stream` dihapus pada versi 1.2.0 dan dijalankan menggunakan pengaturan bawaan kamera. Kini preset ini mengatur format, eksposur, gain, dan laju bingkai: `--preset high_speed` merekam 8-bit pada 5 ms, 6 dB, 60 fps. `lattice cam-connect --preset` juga mempertahankan format preset tersebut (pada versi 1.2.0, kamera diatur ke 12-bit setelah preset diterapkan). Lihat [Opsi Umum](#common-options-shared-by-most-subcommands).
> - **Format piksel Bayer pada kamera M3M (mono) adalah kedalaman pikselnya.** Preset-preset tersebut menyebutkan format Bayer, yang tidak dapat diambil oleh sensor mono: pada versi 1.2.0, kamera menolak format tersebut dan mempertahankan format yang sudah ada, sehingga `--preset high_speed` tidak berformat 8-bit di sana. Kini format tersebut menjadi format Mono dengan kedalaman yang sama: `high_speed` dan `--pixel-format BayerRG8` menghasilkan `Mono8`, `high_quality`, `triggered` dan `BayerRG12` menghasilkan `Mono12`. Dalam susunan campuran M3C + M3M, setiap kamera menggunakan format bersama dalam kelompoknya masing-masing, dan proyek `network-analysis` / `analyze-array --pixel-format` memproyeksikan setiap kamera pada kedalaman tersebut. Hal yang sama berlaku sebaliknya: sebuah M3C yang diminta menggunakan format Mono akan menggunakan format Bayer pada kedalaman tersebut, sehingga susunan yang master-nya adalah M3M tidak lagi membuka kembali kamera-kamera M3C-nya dalam mode Mono. Lihat [Kamera Pita Tunggal Mono (M3M)](#mono-m3m-single-band-cameras).
> - **Ekspor `lattice array-capture` secara default sudah disejajarkan, sama seperti ekspor dari aplikasi.** Setiap file `debayered`, `display`, `radiance`, `reflectance`, dan berkas indeks disesuaikan ke bingkai kamera utama dan dipotong sesuai tumpang tindih antar anggota (lebih kecil dari sensor sebesar offset kamera, `XMP-mapir:AlignmentJSON` `"applied": true`), sehingga hasil pengambilan gambar melalui CLI adalah berkas yang sama dengan hasil pengambilan gambar aplikasi Chloros pada adegan yang sama; `raw` tetap dalam format aslinya. Sebelumnya, pengambilan gambar tanpa `--aligned` menyimpan setiap berkas dalam ukuran aslinya, tanpa distorsi. `--no-aligned` mempertahankan hal tersebut. Pengaturan default yang sama berlaku untuk `--smart`, `--continuous` dan `--interval`, serta pada SDK dan `chloros-mcp`. Lihat [mode `array-capture`](#array-capture-modes).
> - **`lattice array-capture --compression` secara default menggunakan `none`.** File ditulis tanpa kompresi (sekitar 6,3 MB per bingkai resolusi penuh) untuk kecepatan berkelanjutan tercepat; versi 1.2.0 menulis dalam mode `deflate` (sekitar 4,1 MB). Gunakan `--compression deflate` untuk ukuran berkas yang lebih kecil. Keduanya bebas kehilangan data.
> - **Format PNG dan JPEG hanya mendukung `display` / `preview` serta produk indeks.** Pengambilan data dengan `--format png` atau `jpg` yang memiliki tingkat yang tidak dapat ditampung oleh berkas 8-bit akan ditolak sebelum proses pengambilan dimulai (kode status HTTP400, keluar dari `1`), demikian pula dengan pengambilan data default `--processing all` yang menggunakan `--format png`/`jpg`. Lihat [Tingkat Ekspor Pengambilan Data](#capture-export-levels-the-all-default).
> - **`lattice dls` memerlukan `--mac`.**Tidak ada lagi MAC default (sebelumnya merupakan alamat satu unit); `daq pool-discover --only ble` menemukan alamat sensor.**`lattice dls --camera SERIAL`** adalah yang baru.
> - **Kegagalan lainnya keluar dari `1`.** `process` ketika ekspor suatu proses tidak dapat diverifikasi atau kehilangan gambar, meskipun produk lainnya telah ditulis; perintah penangkapan dengan file apa pun yang tidak dapat ditulis (file lainnya tetap tersimpan); `lattice continuous` dan `multi-stream` terjadi pada drive yang penuh; `array-burst` / `array-record` ketika perekam melaporkan kesalahan; `array-build-video` ketika produk yang diminta dilewati atau tidak dibuat sesuai permintaan; `project burst` / `stream` ketika proses penangkapan gagal di tengah jalan; `project sensor log` ketika rekaman tidak mengandung pembacaan apa pun; `lattice exposure --set`, `gain --set`, dan `configure --set` dengan nilai yang ditolak oleh kamera. `lattice network-analysis --json` keluar karena ringkasan juga keluar (`1` untuk `needs_force_slip`). Sebaliknya: `daq pool-disconnect` dari sensor yang tidak lagi berada dalam kumpulan akan keluar dengan kode `0`. Lihat [Kode Keluar](#exit-codes).
> - **`lattice array-connect` yang lambat tidak pernah gagal saat array terhubung.** Proses ini menunggu hingga 600 detik (180 detik pada versi 1.2.0; `project connect` 90 detik), mencetak langkah backend seiring berjalannya proses, dan koneksi yang belum selesai pada saat itu akan dibatalkan oleh backend — setiap kamera dikembalikan ke kondisi semula — sehingga kode kesalahan `Error:` (keluar dengan kode `1`) berarti tidak ada array yang terhubung. Pada versi 1.2.0, koneksi yang dilakukan segera setelah kamera dinyalakan dapat berlangsung lebih lama dari waktu tunggu dan menghubungkan array setelah perintah gagal. Sekarang, Ctrl+C membatalkan koneksi tersebut. Lihat [Koneksi Persisten](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` mempertahankan sensor secara penuh.** Tanpa `--width` / `--height` / `--binning`, setiap kamera diatur ke sensor penuh dan laju bingkai turun sesuai dengan kemampuan komputer dan jaringan ini; output koneksi mencantumkan opsi yang lebih cepat (binning atau ROI yang lebih kecil) beserta laju yang diproyeksikan. Versi 1.2.0 memperkecil atau melakukan binning pada frame untuk menjaga laju tetap tinggi. Lihat [Opsi `array-connect`](#array-connect-options).
> - **Jaringan yang lambat tidak lagi menghalangi pengoperasian array.** Setiap kamera disesuaikan kecepatannya sesuai dengan porsi bandwidth yang tersedia, dan laju bingkai pun menurun seiring dengan itu. Jika kamera tidak dapat disesuaikan secepat itu, array akan berjalan dalam mode emisi serial (`slip-emit-and-capture`): setiap bingkai tiba, tetapi kamera-kamera tidak diekspos secara bersamaan. `--force-tier sim-capture-sim-emit` mengharuskan eksposur simultan. Lihat [FPS Array & Model Burst](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` menunjukkan pengaturan koneksi yang berlaku**: tanpa `--pixel-format`, format yang ditetapkan oleh connect (Mono12 / BayerRG12, sebelumnya BayerRG8), dan keputusan [transportasi 12-bit terkompresi](#packed-12-bit-transport---packed). Oleh karena itu, perintah yang sama dapat menampilkan laju bingkai yang berbeda dari versi 1.2.0.
> - **Jalur relatif mengacu pada lokasi tempat Anda menjalankan perintah.** `-o` dan jalur lain yang dibuka oleh backend (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), langkah resep `output`, serta nilai default `output/` dari `lattice index --live` disesuaikan dengan folder tempat Anda menjalankan `chloros-cli`. Pada versi 1.2.0, backend menafsirkannya di folder instalasinya sendiri, yang pada build yang sudah terinstal gagal dengan pesan "Akses ditolak". Lihat [Konvensi](#conventions).
> - **`lattice index` berjalan di backend dalam bentuk apa pun**, seperti perintah `lattice` lainnya, jadi jalankan backend terlebih dahulu. Indeks offline (`--input`) dihitung oleh backend, yang membaca berkas TIFF dan menulis hasilnya di sampingnya atau di `-o`, dan `--list-presets` / `--list-gradients` mencetak tabel backend. Di Windows, fitur-fitur ini kini berfungsi: sebelumnya, Windows CLI merespons setiap bentuk perintah `lattice index`, termasuk `--live`, dengan `Error: lattice_sdk not available`. Di Linux, indeks offline sebelumnya dijalankan di dalam CLI tanpa backend dan tanpa login; kini keduanya diperlukan, seperti perintah CLI lainnya. Lihat [Indeks / Perhitungan Vegetasi](#index--vegetation-maths).
> - **Berkas baru di samping produk.** Setiap produk radiance, reflektansi, dan indeks di mana pemisahan band membatasi (clamp) suatu piksel akan mendapatkan masker di dalam folder `clamp_masks/` di samping folder produk tersebut, serta mencantumkan baris `clamp mask:` pada hasil cetak. Lihat [Masker pembatasan pemisahan band](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientasi kamera.** Perekaman dengan satu kamera menyimpan file `raw` dan `debayered` sesuai dengan pembacaan sensor, serta mencatat pembalikan/rotasi kamera dalam file XMP; `process` menerapkan informasi tersebut pada produk yang dihasilkan. File anggota array tetap dalam format aslinya kecuali Anda meneruskan `--orient` yang baru (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Lihat [Orientasi kamera](#bagaimana-tampilan-folder-hasil-perekaman).
> - **Nama tingkat ekspor yang bukan salah satu dari daftar tersebut akan ditolak.** `lattice capture --processing` dan setiap daftar `--levels` mencakup `raw`, `debayered`, `radiance`, `reflectance`, atau `display` (atau alias seperti `preview`; `--processing` serta `all`). Nama lain apa pun akan ditolak sebelum kamera disentuh (HTTP 400 `UNKNOWN_LEVEL`, keluar `1`); 1.2.0 menyimpan `raw` untuk itu dan keluar dari `0`.
> - **`daq pool-set-cap` memerlukan `--cap-id`.** Tanpa itu, perintah akan keluar dari `2`; 1.2.0 menghapus koreksi penutup, sehingga sensor tetap telanjang. `--cap-id none` meminta sensor mentah.
> - **`lattice resolution --binning-mode` telah dihapus.** Binning selalu menggunakan metode Rata-rata; bendera tersebut masih diterima, menampilkan peringatan, dan tidak melakukan apa-apa. Lihat [Penyesuaian Sensor](#sensor-tuning).
> - **`process --debayer texture-aware` ditolak untuk tangkapan LATTICE.** Untuk folder yang berisi tangkapan LATTICE, `process` keluar dari `1` setelah impor, menyebutkan jumlahnya, sebelum memproses apa pun. Tangkapan LATTICE selalu mendapatkan demosaic standar, sehingga versi 1.2.0 menerima opsi tersebut dan mengabaikannya; aplikasi Chloros juga tidak menawarkannya untuk tangkapan tersebut. Lihat [`chloros-cli process`](#chloros-cli-process).
> - **Kamera akan dikembalikan ke pengaturan semula — kecuali `--keep-settings`.** `lattice array-disconnect`, setiap perintah sementara (`multi-capture` tanpa array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) dan `lattice continuous` mengembalikan setiap kamera ke kondisi semula: kabel pemicu dan jalur sinkronisasi, PTP-nya, serta setiap pengaturan kamera yang diubah selama sesi (paparan dan penguatan, target AE, batas dan redaman, keseimbangan putih, tingkat hitam, format piksel, binning dan ROI, pembalikan, laju bingkai). `--keep-settings` justru menyimpan setiap pengaturan kamerake dalam memori kamera itu sendiri, sehingga kamera menyala dengan pengaturan tersebut; `--clear-settings` mengatur ulang memori tersebut ke pengaturan pabrik. Dalam kedua kasus tersebut, kamera dibiarkan berjalan bebas. Sebuah proyek mengatur kameranya dari pengaturan yang telah disimpan saat menghubungkannya. Lihat [Memori kamera](#camera-memory----keep-settings----clear-settings).
> - **Setiap mode menyimpan pengaturan proyek dengan cara yang sama.** Setiap perubahan pada kamera, array, atau sensor cahaya yang terhubung ke suatu proyek — baik dari aplikasi, CLI, SDK, maupun server MCP — disimpan ke dalam file `cameras.json` / `sensors.json`, dan membuka proyek tersebut dalam mode apa pun akan mengatur perangkat berdasarkan pengaturan tersebut. Sebelumnya, hanya aplikasi yang menyimpan proyek, dan penyimpanan berikutnya akan menimpa semua yang ada sebelumnya. Lihat [Satu catatan proyek, semua mode](#one-project-record-every-mode).
> - **File yang dibuat dari beberapa kamera mencantumkan nama masing-masing kamera.** Tumpukan multiband (`align-apply`, `align-stream`, `index --live --save-multiband`), indeks langsung yang dibuat dari band beberapa kamera, dan gambar `composite/` dari pengambilan array masing-masing menyimpan nomor seri, model, dan paket kalibrasi kamera sumber sebagai daftar (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), serta daftar EXIF `SerialNumber` yang mencantumkan nomor seri yang dipisahkan dengan koma. Dulu, file-file tersebut sama sekali tidak memuat informasi kamera, atau hanya memuat nomor seri kamera pertama saja. Lihat [File yang dibuat dari beberapa kamera](#files-made-from-several-cameras).
> - **Hanya file `raw` yang memuat tag kalibrasi kamera.** Set yang kompatibel dengan MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration` per kanal, set `MicaSenseVignetting*`) serta muatan `CalibrationJSON` menggambarkan nilai mentah sensor, sehingga hanya terdapat pada berkas penangkapan `raw`. Setiap berkas lainnya — berkas penangkapan `debayered`, `display`, `radiance`, `reflectance`, serta berkas indeks, bingkai `lattice continuous` dan `multi-stream`, dan setiap produk `process` — memuat identitas kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) dan, pada saat pengambilan gambar, catatan bingkai itu sendiri (`XMP-mapir:FrameJSON`: eksposur, penguatan, suhu, format piksel, waktu pengambilan gambar). Pada versi 1.2.0, file-file tersebut juga membawa set data mentah, yang diterapkan oleh alat (Pix4D, Metashape) diterapkan pada piksel yang bukan nilai mentah. `process` menemukan kalibrasi file tersebut berdasarkan nomor serinya dan `CalBundleSha256`.
>
> **Perubahan lainnya:**
>
> - **Kecepatan bingkai otomatis suatu array disesuaikan dengan apa yang dihasilkan oleh array tersebut.** Kecepatan yang kehilangan lebih dari 1% dari pemicunya akan diturunkan, dinilai berdasarkan beberapa pengamatan jangka panjang daripada hanya satu kali, dan kecepatan yang sebelumnya diturunkan akan dicoba kembali sesekali. Kecepatan yang Anda atur sendiri tidak akan diubah.
> - **Pencahayaan otomatis kini lebih stabil.** Fitur ini mempertahankan pencahayaan konstan, menyesuaikan diri dalam hitungan detik setelah terjadi perubahan, mengabaikan sorotan kecil, dan mengaktifkan kembali kamera yang berhenti merespons di bawah sinar matahari yang terik. Lihat [Bagaimana perilaku eksposur otomatis](#how-auto-exposure-behaves).
> - **`process` hanya menghitung gambar dan rekaman Anda.** Jika dijalankan kembali pada folder yang berisi berkas kalibrasi proyek `camera_cal/`, berkas-berkas tersebut tidak lagi dihitung ("Ditemukan 96 gambar" pada kumpulan 90 gambar), dan proyek yang sudah memasukkannya akan menghapusnya.
> - **`lattice multi-test` dimulai dari status kamera yang baru dinyalakan.** Skrip ini mengaktifkan PTP kamera selama proses berjalan, menunggu jam internal kamera terkunci dan stabil sebelum pemicu pertama — setiap kamera terkunci selama 120 detik, sekitar 2 menit sejak dinyalakan (`--ptp-settle SECONDS` mengaturnya), karena jam yang baru saja terkunci masih memiliki selisih hingga ~200 us selama sekitar 2 menit — dan mengembalikan pengaturan PTP setiap kamera ke kondisi semula. Kamera yang masih terkunci dari pengujian sebelumnya tidak ditunggu lagi. Sebelumnya, PTP dinonaktifkan pada jaringan dengan satu switch di mana tidak ada DAQ-E yang merespons, dan jam bebas (free-running clock) setiap kamera dibaca sebagai jam PTP yang tidak akan terkunci (`PTP is not locked`, setiap kali dijalankan). Lihat [Multi-Kamera (Sesi Transien)](#multi-camera-transient-sessions).
> - **Perekaman `.daq` menunjukkan apakah penutupnya dipilih atau diasumsikan.** DAQ-U, DAQ-M, dan DAQ-E dikirim tanpa penutup, dan Chloros tidak dapat mendeteksi apakah penutup terpasang, sehingga sensor yang terhubung tanpa penutup yang disebutkan secara default diberi kode `sunshine_cosine`, sebuah *asumsi*. Setiap rekaman Chloros (skema 1.30) kini menyimpan `cap_id_source` di samping `cap_id`: `operator` jika Anda memberi nama penutup (termasuk `none`), dan `auto_default` jika menggunakan nama default. Saat memproses rekaman yang penutupnya diasumsikan, dengan pengaturan penutup DAQ proyek dibiarkan pada "Auto", akan menampilkan pesan `[daq-dls] WARNING` yang menyebutkan nama file: DAQ-U tanpa pengaturan di bawah koreksi sinar matahari menunjukkan nilai sekitar 30 kali lebih tinggi. **Beri nama cap yang Anda pasang** saat menghubungkan, atau atur cap DAQ proyek, dan peringatan tersebut akan hilang.
> - **Tidak ada lagi jendela konsol pip di Windows.** Pada PC yang telah menginstal Python, backend dan setiap image worker-nya menjalankan `pip` sekali saat startup, dan masing-masing membuka jendela konsol. Kini hal tersebut tidak terjadi lagi.
> - **Perekaman langsung `.daq` tetap utuh meski terjadi pemadaman listrik.**Perekaman ditulis dengan jurnal dan sinkronisasi disk penuh pada setiap commit, di belakang sensor, sehingga disk yang lambat hanya menunda file, bukan pembacaan data. Rekaman yang terhenti di tengah proses commit (pemadaman listrik, penghentian paksa) dapat meninggalkan berkas `<name>.daq-journal` di sampingnya:**salin kedua berkas tersebut bersama-sama.** Chloros mengembalikan jurnal saat pertama kali membaca berkas, sehingga rekaman dibuka dengan semua data yang telah dikomit sebelum penghentian (sekitar 3 detik terakhir hilang); pada versi 1.2.0, pemadaman listrik pada waktu yang tidak tepat dapat membuat seluruh berkas tidak dapat dibaca ("gambar disk basis data rusak").
> - **Perekaman pada disk yang sibuk tidak lagi membuat sensor macet.** `daq pool-record` mengantrekan pembacaan dan menuliskannya pada thread tersendiri; pembacaan yang tidak dapat ditulis akan dihitung (`daq pool-list` menampilkan `N write error(s)` dengan kesalahan terakhir) alih-alih membekukan sensor (satu penulisan memakan waktu 8,4 detik di samping empat kamera yang menyimpan file TIFF).
> - **`lattice multi-capture` tanpa array tidak lagi kehilangan frame yang telah disimpan saat pemicu (trigger) mengalami timeout.** Pemicu kamera yang melebihi batas waktu (`SC_ERR_TIMEOUT -1011`, terlihat pada empat kamera di satu switch 1 GbE) menghentikan proses pada kelompok frame tersebut: kelompok yang sudah disimpan tercantum dengan `Saved:`, kemudian muncul pesan kesalahan, keluar `1`. Pada versi 1.2.0, yang muncul adalah kode kesalahan HTTP500 tanpa daftar, meskipun file-file tersebut ada di disk.
> - **Kamera yang dipicu oleh perangkat lunak tidak lagi melewatkan setiap pemicu kedua.** Kamera dalam mode pemicu perangkat lunak kini mengantrekan pemicu yang tiba selama proses pembacaan (atau Chloros menunggu hingga kamera siap), seperti yang sudah dilakukan oleh kamera yang disinkronkan melalui kabel.
> - **`daq pool-discover` menamai DAQ-E-S `DAQ-E-S`** di kolom `MODEL`-nya (dan kolom `--json` dari `model`). Nama tersebut tertulis sebagai `DAQ-E` untuk setiap sensor Ethernet; sensor yang tidak merespons pemeriksaan identitas dalam waktu sekitar 2 detik tetap terdaftar, sebagai `DAQ-E`.
> - **Produk indeks LATTICE yang ditingkatkan** (NDVI dan indeks lainnya) dari `process`, di setiap mode pemrosesan.
> - **Kamera OCN: produk radian, reflektansi, dan indeks yang ditingkatkan di bawah sinar matahari.** Koreksi armada diterapkan setelah pemisahan pita; kamera RGN tidak berubah, dan nilai OCN berbeda dari versi 1.2.0. Hal ini juga berlaku untuk tangkapan OCN yang dibuat dengan versi sebelumnya, saat diproses pada komputer yang menyimpan kalibrasi yang digunakan saat pengambilan gambar (Chloros menyimpan kalibrasi kamera setelah kamera terhubung).
> - **`lattice continuous` merekam bingkai mentah.** Proses `tiff` menghasilkan bingkai 3-plane yang telah melalui proses demosaic, yang tidak dapat dijadikan titik awal oleh `process`. Frame-frame tersebut kini menjadi berkas `raw` yang dihasilkan oleh perekaman array: satu bidang mosaik dengan nilai asli sensor, tidak terkompresi, beserta paket kalibrasi. Skrip yang membaca tiga bidang dari berkas-berkas ini hanya membaca satu bidang.
>
> - **Setiap penangkapan menyimpan pembacaan DAQ yang digunakannya, seperti yang dilakukan oleh GUI.** `lattice capture`, `lattice continuous` (satu `.daq` per bingkai), `array-capture`, dan `multi-capture` dengan array yang terhubung (dan SDK) menulis `.daq` dan `.match.json`-nya setiap kali DAQ diikat atau dimasukkan ke dalam pool, termasuk penangkapan mentah saja; pada versi 1.2.0, penangkapan mentah atau yang telah di-debayered hanya disimpan satu kali dengan `--force-daq`. `--no-daq` menonaktifkannya, `--daq-tolerance-ms MS` menetapkan toleransi pencocokan untuk penangkapan tersebut, dan `daq pool-bind` mengikat kamera atau array ke DAQ dari CLI. Dengan `.daq`, penangkapan yang disimpan hanya dicocokkan dalam `process` dengan kamera yang disebutkan namanya. Sebuah penangkapan kini menunggu pembacaan berikutnya dari DAQ agar dapat menggabungkan pembacaan di kedua sisi eksposur. Lihat [Pembacaan DAQ yang disimpan oleh proses pengambilan gambar](#the-daq-reading-a-capture-saves) dan [Pembacaan DAQ yang Jauh dari Waktu Paparan](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** menambahkan indeks ke proyek untuk setiap band kamera, termasuk proyek baru. Pada versi 1.2.0, nama yang belum tercantum dalam proyek diabaikan dengan `Unknown index name(s) ignored`.
> - **`process` di samping array langsung:**tidak lagi me-restart backend yang menampung kamera atau DAQ, dan menunggu hingga 15 detik untuk proses yang sedang berakhir.**Ctrl+C** menghentikan proses di backend, bukan hanya di CLI. Lihat [Catatan `process`](#notes).
> - **Folder proyek yang terlalu dalam untuk batas jalur 260 karakter di Windows** akan ditolak sejak awal dengan menggunakan folder terpanjang yang masih sesuai, alih-alih menjalankan proses yang tidak menghasilkan apa-apa. Penginstal Windows kini menawarkan opsi "Aktifkan jalur panjang Windows" dan pengecualian Microsoft Defender untuk backend, keduanya dicentang secara default. Lihat [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** menjalankan penangkapan berkelanjutan milik backend: baris kemajuan setiap 2 detik, ringkasan alih-alih daftar per file, dan kode keluar bukan nol jika ada file yang tidak tertulis atau drive penuh. Lihat [mode `array-capture`](#array-capture-modes).
> - **`array-capture` pertama setelah `array-connect`** menunggu hingga 20 detik untuk kelompok frame pertama dari array, alih-alih gagal dengan kode HTTP 503 "Array masih dalam proses pemanasan".
> - **Pencahayaan array di bawah ROI:** ROI kecil tidak lagi membatasi pencahayaan anggota array pada periode free-run sensor (4,9 ms pada resolusi 768 × 304). Lihat [Pencahayaan pada anggota array](#exposure-on-an-array-member).
> - **Eksposur otomatis tidak lagi menurunkan laju bingkai otomatis array.** Array yang disinkronkan melalui kabel beroperasi pada laju yang diizinkan oleh jaringan dan pembacaan kamera; eksposur otomatis mengisi periode yang diberikan oleh laju tersebut, lalu menaikkan gain. Paparan manual yang lebih lama dari periode tersebut (`lattice exposure --set`, `configure --set`) akan memperlambat pemicu agar sesuai dengan durasi tersebut alih-alih ditolak, dan kecepatan akan kembali normal saat anggota array kembali ke mode otomatis. Lihat [Pencahayaan pada anggota array](#exposure-on-an-array-member).
> - **Array menghasilkan lebih sedikit kelompok bingkai yang tidak lengkap**, dan lebih sedikit kode kesalahan 503 ("Tidak ada kelompok bingkai sinkronisasi baru"): input pemicu setiap kamera slave kini telah dibersihkan dari gangguan.
> - **Transmisi 12-bit terkompresi** (`--packed`, default `auto`) meningkatkan kecepatan bingkai array sensor penuh sekitar sepertiga. `array-connect --packet-size` kini diterapkan (versi 1.2.0 menerimanya namun mengabaikannya). Lihat [Transportasi 12-bit terkompresi](#packed-12-bit-transport---packed).
> - **Sebuah array yang kameranya pulih secara otomatis setelah siklus daya.** Setelah siklus daya sakelar, array `array-connect`-membuka kembali kamera-kamera tersebut saat mereka merespons dan kembali melakukan streaming dengan PTP yang diaktifkan; pada versi 1.2.0, proses ini berhenti secara permanen dan memerlukan `array-connect` baru. `array-status` melaporkan `ptp_enabled: False` (dan `ptp_off:`) saat kamera membaca PTP dalam keadaan mati, padahal sebelumnya melaporkan pengaturan yang digunakan saat array terhubung. Lihat [Koneksi Persisten](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` merekam tanpa GUI yang terbuka.** Saat tampilan gabungan GUI ditutup, backend menampilkan tampilan itu sendiri:NDVI array, atau indeks yang Anda berikan dengan `--expr` baru. Perekaman yang tidak menulis bingkai apa pun kini akan gagal (`no-frames: …`, keluar dari `1`). Sebelumnya, perekaman hanya dilakukan saat tampilan gabungan GUI terbuka, dan perekaman yang tidak menulis apa pun tetap keluar dari `0`. Lihat [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Kamera yang sudah dipegang oleh Chloros tidak akan dibuka untuk kedua kalinya.** Perintah yang mencoba membukanya sendiri akan menerima respons HTTP 409 `CAMERA_HELD`, yang menyebutkan kamera yang sedang dipegang dan perintah yang melepaskannya. Hal ini termasuk kamera yang sedang digunakan oleh perintah lain saat ini (seperti `multi-test` atau `multi-capture` yang membuka kameranya, atau `lattice capture`): perintah kedua akan ditolak hingga perintah pertama selesai. Membukanya kembali dapat menghentikan aliran pemegang atau menyebabkan backend crash. Lihat [Koneksi Persisten](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Perintah multi-kamera sementara akan mengirimkan semua kamera atau gagal.** `lattice multi-capture` tanpa array yang terhubung, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, dan `index --live` keluar dari `1` dengan menyebutkan nama kamera yang tidak mengirimkan bingkai apa pun. Aliran data melewati kelompok frame yang hilang dari sebuah kamera dan berhenti setelah tiga kali berturut-turut; `multi-test` mencetak satu baris per pemicu, `(no frame)` di mana sebuah kamera melewatkannya; `align-stream` juga keluar dari `1` karena kesalahan pekerjaan. Setiap proses kemudian mengembalikan setiap kamera ke kondisi semula: pemicu dan kabel sinkronisasi, PTP (`PtpEnable`), serta setiap pengaturan kamera yang diubah selama proses — eksposur dan gain, target AE, batas dan redaman, keseimbangan putih, tingkat hitam, format piksel, binning dan ROI, pembalikan, laju bingkai — kecuali `--keep-settings` / `--clear-settings` (lihat [Memori kamera](#camera-memory----keep-settings----clear-settings)). Sebelumnya, perintah-perintah ini menyimpan semua frame yang masuk dan keluar dari `0` (`align-stream` menampilkan pesan kesalahan pekerjaan sebagai `Note:`), dan membiarkan setiap kamera tetap berada di `TriggerMode` `On`. Perintah-perintah tersebut juga mengatur eksposur setiap kamera berdasarkan adegan sebelum menyimpan bingkai (paling lama beberapa detik), pada target di mana array yang terhubung memulai setiap kamera pada: 40 multispektral, 60 RGB (`--brightness N` yang mengaturnya), dan mempertahankannya selama proses; demikian pula halnya dengan satu unit `lattice capture` pada kamera yang tidak dikendalikan oleh Chloros. Sebelumnya, bingkai pertama setiap kamera diambil pada batas eksposur kamera itu sendiri, sekitar 100 ms, terlepas dari kondisi cahaya, dan sering kali terpotong. Lihat [Multi-Kamera (Sesi Transien)](#multi-camera-transient-sessions).
> - **Perintah multi-kamera transien berbagi jaringan seperti yang dilakukan oleh `array-connect`, dan hanya membuka `--serials`.** Setiap kamera disesuaikan dengan porsinya pada tautan host, sehingga empat kamera berjalan pada tautan 1 Gbps; sebelumnya, keempat kamera tersebut meminta total 125 MB/s di antara mereka, yang mencakup seluruh kapasitas 1 GbE, dan setiap frame kehilangan paket. Susunan kamera yang tidak dapat ditangani oleh jaringan akan ditolak sebelum kamera mana pun mulai melakukan streaming. `--serials` kini memilih kamera-kamera tersebut, dengan kamera pertama yang memicu pemicu; sebelumnya, setiap perintah membuka semua kamera di jaringan terlepas dari apa yang dikatakan oleh `--serials`. Kegagalan yang disebabkan oleh kehilangan paket GVSP kini ditampilkan secara eksplisit, bukan lagi mengarah ke kabel sinkronisasi M8. Lihat [Multi-Kamera (Sesi Sementara)](#multi-camera-transient-sessions).
> - **`lattice array-disconnect` membiarkan kamera berjalan bebas, sesuai kondisi yang ditemukan.** `TriggerMode` `Off`, dan semua hal lainnya sebagaimana ditemukan saat koneksi — pengaturan pemicu dan jalur sinkronisasi (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP, serta pengaturan kamera yang diubah selama sesi — sehingga satuperintah kamera pada anggota sebelumnya akan langsung mengambil bingkai, dan master tidak lagi tetap berada di `TriggerSource` `Software`. `--keep-settings` menyimpan pengaturan sesi ke dalam memori masing-masing kamera, `--clear-settings` mereset memori tersebut (lihat [Memori kamera](#camera-memory----keep-settings----clear-settings)). `lattice continuous` juga mengatur kamera yang ditemukan dalam keadaan aktif menjadi mode bebas selama sesi (dan mengembalikannya setelahnya), dan sesi tanpa bingkai akan keluar dari `1`.
> - **`lattice configure --get` / `lattice exposure`** mencetak `note:` yang berisipaparan atau penguatan sebenarnya saat mode otomatisnya berjalan. Perintah yang ditolak `lattice exposure --set` / `configure --set` menunjukkan rentang yang diterima kamera dan mengembalikan mode otomatis alih-alih mematikan paparan otomatis. Lihat [Penyesuaian Sensor](#sensor-tuning).
> - **Kontrol kamera baru:**`lattice highlights` (menurunkan kecerahan permukaan putih terang pada tampilan RGB) dan `lattice exposure --brightness-auto`.**`lattice multi-detect`** mendeteksi kamera yang terhubung melalui kabel dalam satu putaran pemindaian dan secara default menetapkan kamera RGB sebagai kamera utama.
> - **Perintah DAQ baru:** `daq pool-bind`; `pool-enable-ptp`; dan untuk pengaturan DAQ-E itu sendiri: `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` dan `pool-update-firmware`. Lihat [Pengaturan Perangkat DAQ-E](#daq-e-device-settings).
> - **DAQ-E yang berhenti mengirim data akan dihubungkan kembali oleh backend.** Ketika aliran data dari DAQ-E yang tergabung dalam pool menjadi diam (misalnya akibat siklus daya pada sakelar yang menyalakannya, terputusnya koneksi kamera), backend akan menghubungkannya kembali dengan pengaturan sebelumnya, dan mencoba kembali dengan interval yang semakin lama selama perangkat tersebut tidak terhubung. Perekaman pada perangkat tersebut berlanjut di `.daq` yang sama dengan jeda yang ditandai dalam file, dan `pool-record --stop` mencantumkan setiap jeda tersebut. Pada versi 1.2.0, sensor tetap tidak mengirim data hingga berkas `pool-disconnect` + `pool-connect`, yang juga mengakhiri perekaman. Lihat [A DAQ-E That Stops Delivering](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** melaporkan apakah paket dari PTP grandmaster mencapai jaringan (`health`), dengan pesan `WARNING:` di stderr jika tidak. Lihat [`time-sync`](#chloros-cli-time-sync).
> - **Paket .deb Linux:** paket ini meningkatkan ukuran buffer penerimaan soket kernel yang dibutuhkan oleh aliran kamera (pada pengaturan default kernel, array akan kehilangan frame dan laju frame), menambahkan layanan backend yang berjalan sebagai pengguna Anda, dan `lattice network` memeriksa buffer tersebut. CLI tidak pernah menghentikan backend yang dijalankan oleh layanan, dan pembaruan akan memulai kembali layanan backend yang diaktifkan. Lihat [Menjalankan backend sebagai layanan](#running-the-backend-as-a-service-headless) dan [Host Linux — buffer penerimaan soket](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` menunjukkan saat tidak dapat melakukan penyelarasan:**setiap alasan penangkapan disimpan tanpa penyelarasan (tidak ada profil penyelarasan, anggota dengan resolusi berbeda, array yang tidak disinkronkan melalui kabel, bingkai yang tidak dapat dipetakan ke profil) akan dicetak sebagai baris `Alignment:`.**`chloros-cli daq help`** adalah `daq --help` (prosesnya telah berakhir dengan kode `2`), dan di Linux, daftar bantuan mencantumkan perintah langsung perangkat keras `daq` yang disertakan dalam build tersebut.
> - **Jetson:** paketnya adalah `chloros_1.3.0_arm64_jp6.deb`, untuk JetPack 6, dan pemrosesan GPU-nya berjalan di Orin menggunakan CUDA PyTorch dari JetPack. Lihat [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Konvensi

- Semua perintah diawali dengan awalan `chloros-cli`. Di Windows, binari tersebut adalah `chloros-cli.exe`; di Linux/Jetson, nama binernya adalah `chloros-cli`.
- Argumen opsional ditampilkan sebagai `--flag`. Argumen posisional yang wajib ditampilkan tanpa tanda kurung.
- Jika nilai default telah ditentukan, mengabaikan bendera tersebut akan menggunakan nilai tersebut.
- CLI adalah klien HTTP ringan yang berjalan di atas backend Chloros (server di `127.0.0.1:5000` yang juga digunakan oleh aplikasi desktop). **Perintah apa saja yang menjalankannya:**
  - Perintah inti — `process`, `login`, `logout`, `status`, `export-status`, `time-sync`, dan `selftest` — menggunakan backend yang sedang berjalan, atau memulainya jika tidak ada yang berjalan, dan menghentikannya kembali setelah selesai.
  - **Perintah `lattice`, `project`, dan `daq` tidak pernah memulai layanan backend.** Perintah-perintah tersebut memerlukan backend yang sudah berjalan: backend dari aplikasi desktop Windows, [layanan backend](#running-the-backend-as-a-service-headless) Linux, atau backend yang Anda jalankan sendiri (lihat [Mulai Backend](#start-the-backend)). Tanpa backend, perintah-perintah tersebut akan menampilkan pesan `Error: Chloros backend not reachable at http://127.0.0.1:5000`, disertai perintah yang digunakan untuk menjalankannya, lalu keluar dengan pesan `1`.
- `CHLOROS_BACKEND_URL=<url>` mengarahkan keluarga perintah **`lattice`**,**`project`**, dan**`daq pool-*`** ke backend lain (`lattice network-analysis` mengarahkan ke `--backend-url`). Perintah inti secara sengaja mengunci `http://127.0.0.1:<port>` dan mengabaikannya (literal IPv4 menghindari penalti waktu sekitar 2 detik per permintaan yang terjadi pada Windows saat mengubah `localhost` menjadi `::1`). Lihat [Variabel Lingkungan](#environment-variables).
- **Jalur relatif diselesaikan berdasarkan folder tempat Anda menjalankan `chloros-cli`.** Backend adalah proses yang membuka folder keluaran dan masukan, jadi sebelum mengirimkan jalur, CLI mengubahnya menjadi absolut: `-o` / `--output` (termasuk nilai defaultnya, `output`), `--input`, `--burst-dir`, `--profile`, `--save`, dan `--mask`, sebuah langkah [resep](#resep-dsl) `output` (default `output`), serta folder default `output` dari `lattice index --live`. Oleh karena itu, `-o captures/` menulis ke `<current folder>/captures/`, dan jalur yang dicetak bersifat absolut. `~` diperluas. Terhadap backend jarak jauh (`CHLOROS_BACKEND_URL` yang menamai mesin lain), jalur dikirim sesuai yang diketik dan diselesaikan di mesin tersebut.
- Login akun Chloros+ diperlukan untuk semua SDK/CLI (jalankan `chloros-cli login` sekali per mesin; disimpan dalam cache di `~/.chloros/`).
- Contoh menggunakan jalur Linux; pada Windows, ganti `/home/user/...` dengan `C:/Users/.../...`. Port serial DAQ-U'berupa `COM3` di Windows dan `/dev/ttyUSB0` / `/dev/ttyACM0` di Linux.
- Ringkasan menulis bendera yang saling eksklusif sebagai `{--a | --b}` dan bendera opsional dalam `[...]`.

---

## Ringkasan Tingkat Atas

```
chloros-cli [global options] COMMAND [command options]
```

### Opsi Global

| Bendera | Deskripsi |
| --- | --- |
| `--backend-exe PATH` | Mengganti eksekutabel backend yang terdeteksi secara otomatis. |
| `--port N` | Port HTTP backend (default: `5000`). Hanya digunakan oleh perintah inti (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project`, dan `daq` menggunakan `CHLOROS_BACKEND_URL` (`http://127.0.0.1:5000` secara default). |
| `-v, --verbose` | Aktifkan keluaran terperinci. |
| `--restart` | Memaksa memulai ulang backend (menghentikan semua proses `backend_server.py` yang sedang berjalan). Backend yang dijalankan oleh layanan systemd (Linux .deb) tidak akan pernah dihentikan: perintah ini akan menampilkan pesan `sudo systemctl restart <unit>` sebagai gantinya dan keluar dari `1`. |
| `--version` | Menampilkan versi (`Chloros CLI 1.3.0`). |
| `--help` | Menampilkan bantuan tingkat atas. |

### Indeks Perintah

| Perintah | Tujuan |
| --- | --- |
| [`process`](#chloros-cli-process) | Memproses folder hasil penangkapan data Survey3 atau LATTICE secara menyeluruh. |
| [`login`](#chloros-cli-login) | Melakukan otentikasi mesin ini dengan akun Chloros+. |
| [`logout`](#chloros-cli-logout) | Menghapus kredensial yang tersimpan dalam cache. |
| [`status`](#chloros-cli-status) | Menampilkan status lisensi / otentikasi saat ini. |
| [`export-status`](#chloros-cli-export-status) | Tampilkan kemajuan ekspor Thread-4 secara langsung selama proses `process` berjalan. |
| [`language`](#chloros-cli-language) | Mengatur atau menampilkan daftar bahasa tampilan CLI (38 bahasa didukung). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#perintah-folder-proyek) | Folder proyek default (digunakan bersama dengan GUI). |
| [`update`](#chloros-cli-update) | Memeriksa dan menginstal pembaruan Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnostik sistem + uji fungsional dasar. |
| [`time-sync`](#chloros-cli-time-sync) | Status / kontrol grandmaster PTP. |
| [`lattice`](#chloros-cli-lattice) | Kontrol & perekaman kamera LATTICE (45+ subperintah). |
| [`daq`](#chloros-cli-daq) | Kontrol sensor spektral DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Membuka dan menjalankan proyek Chloros yang telah disimpan (kamera + DAQ). |

---

## Instalasi

`chloros-cli` disertakan dalam penginstal desktop Chloros pada setiap platform yang didukung — tidak ada unduhan CLI terpisah. Menginstal paket platform akan menambahkan `chloros-cli` ke `PATH` Anda bersama dengan aplikasi desktop dan biner backend yang digunakannya.

Unduhan terbaru: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Penginstal ini juga menyertakan skrip peluncur praktis (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) yang membuka shell CLI yang siap digunakan.

### Windows (.exe)

1. Unduh penginstal Windows dari halaman unduhan.
2. Jalankan `Chloros Installer 1.3.0.exe` dan ikuti panduan penginstalan. Jalur instalasi default adalah `C:\Program Files\Chloros\` (CLI-nya berada di `C:\Program Files\Chloros\cli\`, yang ditambahkan oleh penginstal ke PATH).
3. Buka terminal baru (`cmd.exe`, PowerShell, atau Terminal Windows) agar `PATH` yang telah diperbarui dapat dikenali.

```powershell
chloros-cli --version
```

Penginstal secara otomatis menambahkan `chloros-cli.exe` ke sistem Anda `PATH` dan menyertakan driver kamera LATTICE. Backend-nya adalah `C:\Program Files\Chloros\backend\chloros-backend.exe` (folder `backend` di dalam folder instalasi Anda), yang tidak terdapat di `PATH`.

**Pengaturan sistem.** Halaman *Pilih Tugas Tambahan* pada penginstal memiliki dua opsi di bawah **Pengaturan sistem**, keduanya**dicentang secara default**:

| Opsi (sebagaimana tertulis di penginstal) | Fungsinya | Menonaktifkan instalasi diam-diam |
| --- | --- | --- |
| **Aktifkan jalur panjang Windows** (memungkinkan folder proyek berada di tingkat yang lebih dalam dari 260 karakter; disarankan) | Mengatur nilai `LongPathsEnabled` di seluruh sistem Windows menjadi 1. Windows secara default menonaktifkan pengaturan ini, sehingga tidak ada jalur yang boleh melebihi 260 karakter; produk Chloros berada sekitar 130 karakter di bawah folder proyek, sehingga folder proyek yang lebih dalam akan ditolak (lihat [Pemecahan Masalah](#troubleshooting-pointers)). Berlaku untuk program yang dijalankan setelah instalasi. Tetap diaktifkan saat uninstall, karena program lain mungkin bergantung padanya. | `/MERGETASKS="!longpaths"` |
| **Kecualikan backend Chloros dari pemindaian Microsoft Defender**(perekaman kamera jauh lebih cepat; disarankan) | Menambahkan pengecualian**proses** Defender untuk `chloros-backend.exe` di folder instalasi, sehingga perlindungan waktu nyata tidak memindai setiap file yang ditulis oleh backend, di mana pun file tersebut ditulis. Tidak ada program lain yang dikecualikan; pengecualian ini dihapus saat uninstall. Jika Defender tidak ada atau diblokir oleh kebijakan, langkah ini gagal tanpa pemberitahuan dan proses instalasi berlanjut. | `/MERGETASKS="!defenderexclusion"` |

**Mengapa opsi Defender penting:** pada laju penangkapan yang tinggi, pemindaian real-time terhadap file penangkapan menghabiskan sekitar 30% dari laju penangkapan (diukur pada penangkapan berkelanjutan dengan 9 kamera: 77,7 penangkapan/detik tidak disertakan, 52,7 dipindai). `lattice array-capture --continuous` menampilkan catatan saat mendeteksi Defender menggunakan satu inti CPU atau lebih selama penangkapan; jalankan kembali penginstal dengan opsi tersebut dicentang.

Instalasi diam-diam (`/SILENT` atau `/VERYSILENT`) menerapkan keduanya kecuali jika dinonaktifkan, misalnya `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Firewall Windows.** Penginstal selalu menambahkan dua aturan masuk untuk `chloros-backend.exe`, pada setiap profil jaringan, dan penghapus instalasi menghapusnya: `Chloros Backend (UDP-In)`, untuk aliran kamera, PTP, mDNS, dan aliran data DAQ-E, serta `Chloros Backend (TCP-In)`, yang dibatasi pada subnet lokal, untuk pembaruan firmware DAQ-E, di mana sensor terhubung kembali ke komputer ini untuk mengumpulkan gambar (lihat [Pengaturan Perangkat DAQ-E](#daq-e-device-settings)). API HTTP HTTP backend tetap terikat pada `127.0.0.1`.

### Linux amd64 (.deb)

Untuk Ubuntu 22.04 LTS atau yang lebih baru / workstation berbasis Debian dengan arsitektur x86_64.

> **Ubuntu 20.04 tidak didukung.** Daftar dependensi paket ini berasal dari
> apa yang sebenarnya ditautkan oleh backend, dan itu mencakup `libc6 (>= 2.35)`;
> focal menyertakan glibc 2.31. `apt` menolak proses instalasi daripada membiarkannya gagal saat
> runtime. Daftar tersebut juga mencantumkan `libgl1`, yang dimuat oleh OpenCV yang disertakan dan mungkin tidak ada pada
> gambar server minimal; `apt install` menyertakannya.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

Paket .deb menginstal:
- `chloros-cli` hingga `/usr/bin/chloros-cli`, dan `chloros-backend` (memulai backend) ke `/usr/bin/chloros-backend`
- Backend dan CLI yang telah dikompilasi, setelah diekstrak, berada di `/usr/lib/chloros/backend/` dan `/usr/lib/chloros/cli/`
- Driver kamera LATTICE
- Model denoiser, paket kalibrasi, dan konfigurasi saluran pembaruan
- SDK Python untuk sistem `python3` (`import chloros_sdk`). `apt install` juga membawa persyaratannya sendiri, yaitu `python3-numpy` dan `python3-requests` (direkomendasikan, bukan wajib: program-program di atas bersifat mandiri; `dpkg -i` melewatkannya, dan hasil instalasi akan menyebutkan apa yang kurang). Di dalam virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, yang meningkatkan ukuran buffer penerimaan soket kernel untuk aliran kamera hingga 32 MB, dan menerapkannya sekaligus (nilai yang sudah diatur lebih tinggi akan dibiarkan apa adanya). Menghapus paket ini akan menghapus berkas tersebut. Lihat [Host Linux — buffer penerimaan soket](#linux-hosts--socket-receive-buffers).
- Dua layanan systemd opsional untuk backend, keduanya tidak diaktifkan (bagian berikutnya)

#### Menjalankan backend sebagai layanan (headless)

Tanpa layanan, backend (dan bersama itu PTP grandmaster) hanya berjalan selama ada yang menjalankannya: `chloros-backend` di terminal, skrip SDK, atau perintah inti `chloros-cli` selama masa kerjanya. Untuk backend yang selalu aktif pada mesin headless, aktifkan layanan **sebagai pengguna Anda**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` menjalankan backend sebagai pengguna tersebut. Data yang ditulisnya adalah milik pengguna tersebut, log terperincinya terdapat di file `~/.cache/chloros/logs` miliknya, dan menggunakan file `chloros-cli login` (`~/.chloros`) milik pengguna tersebut, jadi masuklah terlebih dahulu sebagai pengguna tersebut. PTP tidak memerlukan akses root. Seperti setiap backend yang terinstal, backend ini hanya mendengarkan pada port `127.0.0.1:5000`.

`chloros-backend.service` menjalankan backend sebagai **root**. Backend ini tetap terinstal agar mesin yang mengaktifkannya tetap berjalan, tetapi semua yang ditulis oleh backend root menjadi milik root, sehingga pengguna tidak dapat menghapus rekaman mereka sendiri. Untuk beralih ke akun pengguna Anda:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

Log terperinci layanan root terdapat di `/var/log/chloros`, yang dapat dibaca oleh anggota grup `adm` (layanan ini menetapkan `CHLOROS_LOG_DIR`; lihat [Variabel Lingkungan](#environment-variables)). Pemasangan paket akan menampilkan perintah `enable` dengan nama pengguna Anda terisi, serta opsi peralihan tersebut saat layanan root diaktifkan.

**Aktifkan satu unit backend.** Hanya satu backend yang berjalan pada satu waktu. Unit kedua yang diaktifkan (`chloros-backend.service` selain `chloros-backend@<user>.service`, atau dua unit `@` milik pengguna) tidak mau dimulai dan tetap berada pada status `failed` dengan status keluar 98; systemd tidak akan memulainya kembali. `systemctl status`-nya menjelaskan alasannya: `Another instance is already running. Exiting.`, lalu apa yang menahan backend (port 5000, atau berkas kunci dan pengguna yang memilikinya), kemudian `systemd will not restart <unit> (exit status 98)`. Setelah reboot, salah satu unit tersebut mungkin yang akan berjalan. Untuk menggantinya, nonaktifkan unit yang tidak Anda gunakan, seperti dijelaskan di atas.**CLI** tidak pernah menghentikan backend suatu layanan. `chloros-cli` menggunakannya apa adanya, seperti yang dilakukan GUI dan SDK: `process` berjalan di atasnya, memuat proyeknya sendiri menggantikan proyek terakhir. Di mana CLI biasanya akan me-restart backend (`--restart`, atau `process` yang impornya tidak menemukan apa pun), ia menyebutkan nama unit tersebut, menampilkan `sudo systemctl restart <unit>`, dan keluar dari `1`, karena backend yang dihentikan dengan cara tersebut akan tetap mati. `GET /api/status` menamai unit tersebut sebagai `service` (`null` untuk backend yang tidak menjalankan layanan apa pun).**Peningkatan versi dan penghapusan.** Pembaruan paket akan menghentikan semua layanan backend, kemudian memulai kembali setiap layanan yang diaktifkan, dimulai dengan unit `@<user>` (satu unit yang dimulai secara manual tetapi tidak pernah diaktifkan akan tetap berhenti, seperti setelah reboot). Jika lebih dari satu layanan diaktifkan, hanya satu yang berjalan dan yang lainnya tetap gagal seperti di atas; proses pembaruan menampilkan pesan `Chloros: N backend units are enabled: …` dan `Keep one enabled: sudo systemctl disable --now <unit>`. Penghapusan paket ini akan menghentikan dan menonaktifkan layanan-layanan tersebut. Menghapus paket tersebut juga akan menghapus `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Tata letak yang sama dengan berkas .deb amd64, termasuk buffer penerimaan dan [layanan backend](#running-the-backend-as-a-service-headless), untuk Jetson Orin / Orin NX / Orin Nano pada **JetPack 6** (L4T R36). PyTorch-nya adalah build CUDA JetPack 6 untuk Orin, sehingga pemrosesan GPU, termasuk debayer yang mendukung tekstur, berjalan di GPU Jetson. Paket ini tidak menyertakan CUDA itu sendiri: paket ini menggunakan pustaka CUDA dan cuDNN yang diinstal oleh JetPack 6 (`sudo apt install nvidia-jetpack` pada gambar yang tidak menyertakannya). Jika tidak ada GPU CUDA yang dapat digunakan, pemrosesan berjalan di CPU dan pemrosesan yang mendukung tekstur menggunakan debayer standar (lihat [`process`](#chloros-cli-process)).

### Otentikasi Sekali Per Mesin

Setiap platform memerlukan login Chloros+ satu kali sebelum panggilan SDK/CLI dapat berfungsi:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Kredensial disimpan dalam cache di `~/.chloros/user_session.json`.

### Memulai Backend

Perintah `lattice`, `project`, dan `daq` berkomunikasi dengan backend yang sudah berjalan; perintah-perintah tersebut tidak pernah memulai backend (lihat [Konvensi](#konvensi)). Pastikan backend tetap berjalan saat Anda menggunakannya:

| Platform | Cara |
| --- | --- |
| Windows | Buka aplikasi desktop Chloros (backend-nya juga melayani CLI), atau jalankan `"C:\Program Files\Chloros\backend\chloros-backend.exe"` di terminal tersendiri (folder `backend` di folder instalasi Anda) dan biarkan tetap terbuka. |
| Linux/Jetson | Jalankan `chloros-backend` di terminal tersendiri, atau biarkan satu instance selalu berjalan dengan [layanan backend](#running-the-backend-as-a-service-headless). |

Hanya satu backend yang berjalan pada satu waktu; setiap klien (GUI, CLI, skrip SDK) berbagi backend tersebut. Perintah inti (`process`, `login`, `status`, …) juga menggunakannya, dan akan memulainya jika tidak ada yang sedang berjalan.

**Sebuah backend tetap berjalan di antara perintah-perintah**, sehingga perintah berikutnya tidak perlu melakukan proses startup ulang. Proses ini berhenti saat Anda keluar (`chloros-cli logout`, kecuali jika aplikasi Chloros terbuka atau proses eksekusi atau pembangunan offline sedang berlangsung), saat Anda menjalankan `chloros-cli backend stop`, atau secara otomatis setelah 30 menit tanpa ada yang menggunakannya: tidak ada perintah, skrip, atau aplikasi, tidak ada proses eksekusi atau pembuatan offline, tidak ada kamera atau sensor cahaya yang terhubung. `backend stop` menolak saat proses eksekusi atau pembuatan offline sedang berlangsung, dengan menyebutkan namanya; `--force` menghentikannya saat itu juga dan proses eksekusi atau pembuatan tersebut hilang (serangkaian data tetap tersimpan, dan `array-build-video` membangunnya kembali). Atur `CHLOROS_BACKEND_IDLE_MIN` sebelum backend mulai untuk mengubah batas waktu 30 menit (`0`: tidak pernah); [layanan backend](#running-the-backend-as-a-service-headless) tidak akan pernah berhenti dengan sendirinya. `chloros-cli backend status` menunjukkan apakah ada yang sedang berjalan, apa yang menggunakannya, dan apakah ada proses pembuatan offline yang sedang berjalan. `process` membuka proyeknya di backend yang sedang berjalan, kecuali jika aplikasi Chloros sudah terbuka dengan proyek yang dibuka: maka proses tersebut akan ditolak, dan Anda harus memprosesnya dari aplikasi atau menutup proyek tersebut terlebih dahulu. Sebuah proyek yang dibiarkan terbuka oleh `process` sebelumnya atau skrip SDK akan digantikan begitu saja.

### Verifikasi Instalasi

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

`selftest` akan keluar (`0`) hanya jika ketujuh pemeriksaan berhasil. Pemeriksaan terakhir memerlukan GPU CUDA (seperti GPU NVIDIA, atau Jetson Orin) serta model denoiser; sehingga pada komputer tanpa GPU tersebut, sistem akan menampilkan `Failed: CUDA + Denoiser` dan keluar dari `1`; semua fitur kecuali debayer berbasis tekstur yang hanya berjalan di GPU tetap berfungsi di sana.

> **Diperlukan langganan Chloros+.**CLI memerlukan paket Chloros+ yang aktif.**Copper**adalah tingkatan Chloros+ pemula — setiap tingkatan Chloros+ berbayar memiliki akses ke CLI/SDK; hanya tingkatan gratis**Iron** yang tidak memilikinya. (Peta ID paket: `0`=Iron/gratis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Lakukan peningkatan di [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Batas minimum ini diberlakukan oleh backend, bukan hanya oleh CLI: permintaan yang menggunakan SDK/CLI tanpa paket berbayar akan ditolak dengan kode kesalahan `403 PLAN_UPGRADE_REQUIRED`, baik berasal dari `chloros-cli`, Python SDK, maupun klien HTTP yang dibuat sendiri. Pemanggil yang sudah logout akan menerima kode kesalahan `401 AUTH_REQUIRED`. Akses tetap berfungsi secara offline selama masa tenggang paket (30 hari per bulan, hingga masa berlaku habis untuk paket tahunan) dan berhenti saat masa tenggang tersebut berakhir; `chloros-cli status` tetap berfungsi sehingga alasannya terlihat (ini adalah satu-satunya rute SDK/CLI yang dikecualikan dari batasan tingkatan — `GET /api/license-status`).

---

## Panduan Cepat Lima Menit

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

`process` (langkah 2) memulai backend untuk eksekusinya sendiri jika belum ada yang berjalan dan menghentikannya setelah selesai, jadi mulailah satu backend untuk langkah 4 ke depan. Lihat [Memulai Backend](#start-the-backend).

---

## `chloros-cli process`

Proses folder gambar melalui seluruh alur kerja Chloros (deteksi target → kalibrasi → vignette → reflektansi → ekspor indeks).

### Ringkasan

```
chloros-cli process INPUT [OPTIONS]
```

### Argumen Posisional

| Argumen | Deskripsi |
| --- | --- |
| `INPUT` | Jalur ke folder masukan yang berisi berkas `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE), atau `.dng`. |

### Opsi Umum

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `-o, --output PATH` | folder baru yang dinamai sesuai dengan `-n` di dalam folder proyek default | Folder proyek. Proyek dinamai sesuai dengan nama folder tersebut, dan folder yang sudah berisi berkas `project.json` akan diberi nama `_1` / `_2` sebagai folder saudara, bukan ditimpa. Folder proyek default adalah yang dipilih oleh `set-project-folder` (atau antarmuka pengguna), jika tidak maka `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | otomatis (cap waktu) | Nama proyek, hanya digunakan **tanpa** `-o` (dengan `-o`, proyek akan menggunakan nama folder). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` menggunakan debayer neural Chloros+ untuk citra Survey3; lebih lambat tetapi kualitasnya lebih tinggi. Ini hanya untuk Survey3: Pengambilan gambar LATTICE selalu menggunakan demosaic standar, sehingga untuk folder yang berisi pengambilan gambar LATTICE, `process` akan menolak `texture-aware` setelah impor, dengan pesan kesalahan yang menyebutkan jumlahnya, dan keluar dari `1` sebelum memproses apa pun (aplikasi Chloros juga tidak menawarkan Texture Aware untuk gambar-gambar tersebut). Dibutuhkan GPU CUDA (GPU NVIDIA pada Windows/Linux x64, atau GPU Orin pada Jetson): tanpa GPU tersebut, proses akan menggunakan `standard`, dan backend mencatat satu baris log `[DEBAYER] … needs a CUDA GPU …` yang menjelaskan alasannya (ditampilkan dengan `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Koreksi vignette. |
| `--reflectance / --no-reflectance` | `--reflectance` | Kalibrasi reflektansi (menggunakan target panel jika ditemukan, kalibrasi NIST per-serial untuk LATTICE). Untuk LATTICE multispektral, ini juga berfungsi sebagai **produk** reflektansi — lihat [Toggle Ekspor Per-Produk](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | mati | Terapkan koreksi PPK GNSS dari berkas sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | tidak ada | Model kamera yang dihubungkan ke pin eksposur 1, untuk sinkronisasi waktu PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | none | Model kamera yang terhubung ke pin eksposur 2, untuk sinkronisasi waktu PPK. |
| `--recal-interval SECONDS` | `0` | Interval kalibrasi ulang minimum, dalam detik (*Interval kalibrasi ulang minimum* proyek). |
| `--timezone-offset HOURS` | `0` | Selisih zona waktu sensor cahaya dari UTC, dalam jam (nilai *Selisih zona waktu sensor cahaya* proyek). |
| `--format FORMAT` | `TIFF (16-bit)` | Salah satu dari `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. Chloros tidak pernah meningkatkan kedalaman bit gambar: pada semua gambar 8-bit, format yang lebih dalam ditulis sebagai `TIFF (8-bit)` dan proses tersebut menyatakan hal itu sekali. Dalam proyek yang menggabungkan tangkapan 8-bit dan 12/16-bit, setiap tangkapan diekspor dengan kedalaman aslinya atau yang lebih rendah. Radiance selalu berupa float 32-bit. |
| `--indices NAME [NAME ...]` | tidak ada | Indeks vegetasi berdasarkan nama preset (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), tidak membedakan huruf besar-kecil. Masing-masing ditambahkan ke proyek sesuai dengan band kamera yang bersangkutan, termasuk proyek baru. Nama yang bukan preset, atau yang tidak dapat dihasilkan oleh band kamera (misalnya, `NDRE` memerlukan RE dan NIR, yang tidak disediakan oleh satu kamera pun), akan ditampilkan dalam peringatan dan tidak mendapatkan gambar indeks. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Memaksa titik masuk alur kerja untuk berkas TIFF LATTICE (berkas .raw Survey3 tidak terpengaruh). Juga jalan keluar yang memungkinkan pengambilan gambar **tanpa file raw** diproses sama sekali — lihat [Tampilan folder captures](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | aktif | Menghasilkan produk debayering linier (`Debayered_Images`). Lihat [Toggle Ekspor Per Produk](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | aktif | Menghasilkan pratinjau tampilan (`Preview_Images`): RGB = keseimbangan putih (sumber cahaya DAQ jika tersedia, jika tidak maka gray-world) + gamma; multispektral = peregangan warna palsu. |
| `--radiance / --no-radiance` | aktif | Mengirimkan radian float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referensi untuk produk reflektansi LATTICE: `auto` = target dalam bingkai yang lulus QA adalah referensi absolut (target dengan bendera QA bersaing dengan DAQ berdasarkan skor pengukuran), fallback DAQ-downwelling (ρ = π·L/E); `target` = ketat (tanpa penggantian DAQ); `daq` = DAQ-otoritatif. Lihat [Tombol Pengaturan Ekspor Per-Produk](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | tidak ada | Direktori pemindaian reflektansi target **yang diukur** per unit (`<serial>.csv`); akan kembali ke spektrum T3/T4P nominal jika tidak ditemukan. |
| `--array-alignment / --no-array-alignment` | aktif | Array LATTICE: menerapkan penyelarasan antar-modul yang tercantum dalam file XMP `Chloros:Alignment*` setiap pengambilan data ke setiap produk yang diproses (debayering / pratinjau / radian / reflektansi / indeks). Tidak melakukan apa-apa untuk gambar tanpa tag tersebut. |
| `--array-alignment-crop / --no-array-alignment-crop` | crop | Potong ekspor yang telah diselaraskan ke wilayah tumpang tindih bersama array sehingga semua modul berbagi satu area; `--no-…` mempertahankan kanvas sensor penuh (diisi hitam di luar sumber). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Resampling untuk distorsi penyelarasan. `nearest` mempertahankan nilai DN sumber yang tepat (tanpa pencampuran nilai radiometrik antar-piksel). |

### Opsi Deteksi Target

| Bendera | Deskripsi |
| --- | --- |
| `--min-target-size PIXELS` | Ukuran minimum target kalibrasi dalam piksel untuk detektor. |
| `--target-clustering 0-100` | Persentase pengelompokan target (default `60`). |
| `--target / --targets` | Mencari target kalibrasi hanya di subfolder `target` atau `targets` dari input (lebih cepat). |

### Contoh

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

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

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Tombol Pengaturan Ekspor Per-Produk (LATTICE multispektral)

Pemrosesan LATTICE menyebar ke **setiap produk yang berlaku dalam satu kali proses**. Keempat tombol pengaturan per-tipe — `--debayered`, `--preview`, `--radiance`, `--reflectance` — semuanya**diaktifkan secara default**; gunakan formulir `--no-<type>` untuk menonaktifkan salah satunya. Kamera utama RGB hanya memancarkan data yang telah di-debayering + pratinjau (tidak ada radian/reflektansi per-band), sehingga `--radiance`/`--reflectance` tidak berpengaruh bagi kamera tersebut. Tombol pengatur ini diabaikan untuk `.raw` pada Survey3 (yang mengikuti jalur reflektansi/target standar). *(Bendera `--radiometric-output {reflectance,radiance,sensor-response}` **dihapus** dan digantikan oleh opsi-opsi ini; tidak ada lagi level `sensor-response`.)*

| Produk | Output | Perlu DAQ downwelling? |
| --- | --- | --- |
| `--debayered` | Demosaik linier (`Debayered_Images`): file yang sama yang disimpan oleh penangkapan pada tingkat `debayered`-nya — offset gelap (DSNU) dari paket kalibrasi pada data mentah, kemudian demosaik yang peka tepi dari OpenCV; tanpa LUT linearitas, flat field, atau unmix. | Tidak. |
| `--preview` | Pratinjau tampilan (`Preview_Images`): RGB = WB + gamma; multispektral = peregangan warna palsu. | Tidak. |
| `--radiance` | float32 W/m²/sr/nm dari rantai radiometrik lengkap (`Radiance_Images`). | No. |
| `--reflectance` | uint16 reflektansi ρ (`32768` = 1,0), Siap untuk Pix4D. | **Ya**, kecuali jika diikat oleh target dalam bingkai yang lulus QA (lihat di bawah). |

`--reflectance-source` memilih referensi reflektansi:**`auto`**(default) menjadikan target dalam bingkai yang lulus QA sebagai**referensi absolut**— rantai garis empiris yang diikat oleh target tersebut dievaluasi silang pada panel yang disisihkan, dan pemenang yang diukur diterapkan, band demi band: band yang tidak dapat dikalibrasi oleh DAQ, atau yang penyesuaian kemiringan DAQ-nya gagal, akan menggunakan garis dari target itu sendiri (`ReflectanceElBandMethods` menentukan yang mana) — target yang lulus QA dengan bendera (panel terlewat, band tidak berurutan) juga bersaing dengan pembagian DAQ berdasarkan skor yang diukur tersebut, dan bingkai yang mengalahkannya diberi cap `ReflectanceChain=daq` — kembali ke batas bawah DAQ (ρ = π·L/E) ketika tidak ada target atau QA gagal;**`target`**bersifat ketat (tidak ada penggantian DAQ);**`daq`**memilih untuk mengikuti perilaku otoritatif DAQ. Geometri target (ArUco / ROI tetap / strip) berasal dari konfigurasi target proyek; `--target-reflectance-dir DIR` menyimpan pemindaian**terukur** pemindaian per unit (`<serial>.csv`) yang dicari berdasarkan nomor seri/QR unit target, dengan spektrum T3/T4P nominal sebagai cadangan.

Jalur reflektansi DAQ secara otomatis menentukan **sinar turun yang sesuai dengan cap waktu**dari**`.daq`**(DAQ-U/M/E) yang terekam**atau file `.csv` asli DAQ-M**yang ditemukan bersama citra tersebut. Jika paket kalibrasi per-kamera atau DAQ tidak disimpan dalam cache lokal, pipeline**akan mengambilnya secara otomatis dari AWS** saat pertama kali digunakan (membutuhkan koneksi internet sekali; disimpan dalam cache di bawah `~/.chloros/`).**Siapa pemilik pembacaan ini.**Sebuah `.daq` yang disimpan oleh penangkapan adalah pembacaan dari kamera yang dijelaskannya — dinamai dalam file sidecar `<file>.daq.cameras.json` di sampingnya (`lattice capture`, `multi-capture`, `array-capture` dan antarmuka pengguna; sebuah burst mentah mencantumkan pembacaan setiap kamera dalam manifesnya) — dan `process` mencocokkannya**hanya dengan gambar dari kamera-kamera tersebut**. Kamera mandiri yang tidak memiliki pembacaan sendiri oleh karena itu tidak lagi meminjam DAQ kamera lain; ikat DAQ ke kamera tersebut, atau rekam satu. Rekaman DAQ itu sendiri (`daq pool-record`, atau rekaman yang dimulai di GUI), atau `.daq` apa pun tanpa sidecar tersebut (termasuk tangkapan lama), tetap berfungsi untuk setiap kamera.

#### Membaca piksel reflektansi (Pix4D / Metashape / skrip Anda sendiri)

Reflektansi disimpan sebagai bilangan bulat DN, dan **nilai DN yang berarti ρ = 1,0 bergantung pada kamera sumber**:

| Sumber | ρ = 1,0 adalah | Cara mengetahuinya |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ruang kosong hingga ρ 2,0) | Tanda XMP `Chloros:PixelScale=32768` tercantum pada file. |
| Survey3 | `65535` (dipotong pada ρ 1,0) | Tidak ada tag XMP `Chloros:*` — ketiadaan tersebut *adalah* sinyalnya. |

**Baca `Chloros:PixelScale` dan bagi dengan nilainya** daripada mengasumsikan nilai konstan. Tag tersebut didefinisikan dalam domain uint16, sehingga nilainya tetap `32768` di seluruh format keluaran yang melakukan penskalaan ulang — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`, dan `TIFF (32-bit, Percent)` semuanya bersifat self-describing (normalisasikan tipe data yang disimpan kembali ke uint16 terlebih dahulu: ×257 dari 8-bit, ×65535 dari float).

> **File dari build yang lebih lama mungkin tidak memiliki skala.** Sebuah file TIFF reflektansi 8-bit diskalakan ulang dan diberi label `Chloros:PixelScale` seperti yang lainnya. Build sebelum versi 1.3.0 malah *memotong* nilainya menjadi 0..255, sehingga setiap nilai di atas ρ≈0,008 diratakan menjadi 255, dan mereka menghilangkan baik tuple `Chloros:PixelScale` maupun `MicaSense:RadiometricCalibration`. **Jika tag tersebut tidak ada pada berkas reflektansi LATTICE, jangan mengasumsikan adanya skala — olah ulang berkas tersebut** daripada membagi piksel yang sebenarnya tidak dapat dibagi.

#### Membaca piksel indeks (indeks NDVI, dan indeks vegetasi lainnya)

**Raster indeks bilangan bulat merupakan penyesuaian kontras per gambar, bukan peta dengan rentang −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, dan `JPG (8-bit)` — termasuk file TIFF 8-bit yang dihasilkan dari pengambilan data 8-bit — menyimpan

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

di mana `IndexMin` / `IndexMax` adalah **nilai indeks terendah dan tertinggi yang terbatas dari bingkai tersebut**. DN 0 bukanlah −1, dan skala penuh bukanlah +1.

Stempel Chloros menandai kedua titik ujung tersebut sehingga pemetaan tetap dapat dibalik:

| Tag | Arti |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvensi di bawah ini |
| `Chloros:IndexMin` | nilai indeks yang disimpan sebagai DN 0 |
| `Chloros:IndexMax` | nilai indeks yang disimpan sebagai skala penuh |

Baca nilai-nilai tersebut dan balikkan, lalu normalisasikan DN yang disimpan ke rentang 0…1 berdasarkan nilai maksimum tipe data (dtype) yang disimpan terlebih dahulu, sehingga satu ekspresi dapat mencakup semua format bilangan bulat:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

Pembacaan tidak memerlukan `-config pix4d.config` — exiftool membaca properti XMP ini tanpa bantuan. Konfigurasi hanya diperlukan untuk *menulis* nilai-nilai tersebut.

> **`IndexMin == IndexMax`** berarti setiap piksel pada bingkai tersebut memiliki nilai indeks yang sama. Raster diisi pada skala tengah dan tidak ada yang perlu dibagi — ambil nilai ujungnya.

> **Tidak adanya tag `Chloros:Index*` berarti tidak ada pemetaan yang dinyatakan.** Ekspor LUT adalah hasil render berwarna yang pikselnya berupa warna gradien yang disesuaikan dengan jendela ambang batas LUT Anda, dan bingkai tanpa piksel indeks terbatas tidak memiliki rentang yang dapat dinyatakan. Jangan mengasumsikan adanya pemetaan dalam kedua kasus tersebut — aturan yang sama berlaku seperti pada kasus `PixelScale` yang hilang.**Karena peregangan dilakukan per gambar, dua bingkai dari satu penerbangan tidak dapat dibandingkan sebagai DN**— masing-masing memiliki titik akhir sendiri, sehingga nilai NDVI yang sama akan berada pada DN yang berbeda di gambar yang berbeda. Untuk pekerjaan kuantitatif di seluruh penerbangan (statistik zonasi, ambang batas tetap, rangkaian waktu), ekspor**`TIFF (32-bit, Percent)`**: raster indeks ditulis di sana sebagai float32**nilai indeks itu sendiri**, tanpa perlu penskalaan ulang dan tanpa tag. Terlepas dari nama formatnya, produk indeks tidak dikalikan ×100 — penskalaan tersebut hanya berlaku untuk reflektansi bertipe data integer.

#### Masker pembatas pemisahan spektral (piksel mana saja yang pembatasannya diterapkan pada suatu band)

Pemisahan spektral 3×3 dapat membuat nilai suatu band menjadi negatif — baik ketika cahaya hampir tidak mengandung band tersebut, maupun ketika adegan berada di luar rentang yang dijelaskan oleh matriks pemisahan kamera. Itu bukanlah nilai yang sebenarnya, sehingga ekspor menetapkannya ke 0 — dan tampilan langsung menunjukkan nilai 0 yang sama. Sebuah pita dapat dibatasi pada beberapa piksel atau hampir semuanya; masker mencatat keduanya. `Chloros:Band{R,G,B}UnmixClampFraction` menunjukkan seberapa besar bagian frame yang mengalami hal tersebut; masker pembatasan menunjukkan **piksel mana saja**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— di dalam folder `clamp_masks/`**di samping** folder produk, bukan di dalamnya (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; ekspor dari `process` menghasilkan `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — untuk setiap produk radian, produk reflektansi, dan indeks di mana setidaknya satu piksel terbatasi: baik data yang diambil maupun ekspor `process` (serta berkas TIFF per-frame dari `array-build-video --save-tiffs`). Data satu-pita `uint8`, pada kisi piksel produk itu sendiri (termasuk orientasi, ROI, binning, dan penyelarasan array). Setiap topeng mencantumkan nama kamera produknya (`Make` / `Model` / serial), waktu pengambilan, dan tag lensa, serta mencantumkan pita yang diwakili oleh bit-bitnya (`XMP-Camera:BandName`, bit 0 terlebih dahulu); `ImageDescription` dan `Software` menandainya sebagai masker clamp, dan masker ini tidak mengandung kalibrasi.
- **Bit**: bit 0 = pita pada saluran R produk (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Bit yang aktif berarti nilai piksel pada band tersebut telah dibatasi menjadi 0 — perlakukan nilai tersebut, serta indeks apa pun yang dibuat darinya, sebagai data dengan tingkat kepercayaan rendah.
- **`Chloros:UnmixClampMask`** dicantumkan pada setiap produk yang telah melalui proses pemisahan: jalur masker relatif terhadap folder produk itu sendiri (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, selalu dipisahkan oleh `/` — gabungkan dengan folder gambar untuk membuka masker), atau `none` jika tidak ada piksel yang dibatasi. Tidak adanya tag berarti produk tersebut tidak pernah menjalani proses unmix (kamera RGB atau monokrom, level raw/debayered).
- `lattice capture` / `array-capture` mencantumkan jalur masker di bawah file yang disimpan (`clamp mask: …`), dan hasil pengambilan gambar mencantumkannya sebagai `clamp_mask` pada bingkai tersebut.
- `lattice index --live --save-multiband` menulis satu masker per kamera sumber untuk tumpukan multiband — lihat [`lattice index` — masker tumpukan multiband](#lattice-index--multiband-stack-masks). Video tidak memiliki masker: `array-build-video` melaporkan fraksi clamp setiap bingkai sebagai gantinya (lihat [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Karena tidak ada folder produk yang menyimpan masker, pada tingkat mana pun, sebuah folder produk (`radiance/`, `reflectance/`, folder `*_Images/` dari ekspor `process`) dapat diserahkan ke Pix4D, Metashape, atau perangkat lunak fotogrametri lainnya apa adanya — termasuk subfolder — dan perangkat lunak tersebut hanya menerima gambar. Berikan folder produk tersebut, bukan folder di atasnya (folder di atasnya juga berisi `clamp_masks/` dan semua jenis ekspor lainnya). Fitur impor bawaan Chloros mengabaikan masker. Membaca masker: `(mask & 1) != 0` berarti "saluran R dibatasi di sini", `& 2` G, `& 4` B.

#### Data EXIF yang disertakan dalam ekspor

`process` menyalin **blok GPS dan ExifIFD** dari pengambilan gambar sumber ke setiap produk, sehingga
ekspor mencakup `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal`, dan
`CameraSerialNumber` bersama dengan georeferensi.

**`FocalLength` bukanlah opsi dalam fotogrametri.** Pix4D menghitung jarak sampel permukaan (GSD) dari
panjang fokus ditambah ketinggian; jika tag tersebut tidak ada, program akan menggunakan skala yang sangat salah. Pada satu
penerbangan di kebun jeruk dengan 49 pengambilan gambar, tag yang hilang mengubah lokasi seluas 411 m × 160 m menjadi rekonstruksi
47,8 km × 13 km — ortofoto 455 MP yang sebagian besar berisi data kosong, yang kemudian dianggap sebagai masalah penataan ubin dan
masalah BigTIFF sebelum ada yang memeriksa GSD. Jika ortofoto Anda menghasilkan skala yang tidak masuk akal, jalankan `exiftool -FocalLength` pada produk yang diekspor terlebih dahulu.

Salinan ini sengaja **bukan** `-all:all`: tag struktural IFD0'tag strukturalnya merusak output LATTICE saat
disalin, dan `ExifImageWidth` / `ExifImageHeight` dikecualikan karena menggambarkan
perekaman *sumber* — ekspor yang pernah diubah ukurannya akan membawa dimensi
yang bertentangan dengan rasternya sendiri. XMP ditulis secara langsung, bukan disalin, karena ExifTool
membuang tag XMP dari pemanggilan yang sama saat blok XMP disalin (yang akan menghilangkan tag kalibrasi MAPIR).

### Lokasi penyimpanan hasil

Hasil ditulis **di bawah folder proyek, dikelompokkan berdasarkan kamera dan kemudian berdasarkan format file**:

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

Folder kamera untuk LATTICE adalah `LATT-<sensor>-<lens>-F<filter>` (sesuai dengan EXIF penangkapan
`Model`) dan `<model>_<filter>` untuk Survey3 — dua kamera yang berbagi sensor dan filter tetapi berbeda
dalam lensa tetap memiliki pohon folder terpisah, karena vignette, bidang pandang, dan distorsi berbeda. Format
folder mengikuti `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, atau `tiff32` untuk
`TIFF (32-bit, Percent)`.

> **Setiap produk yang diekspor tetap menggunakan nama file SUMBER.** Ekspor radiance dari
> `capture_…_raw.tif` tetap bernama `capture_…_raw.tif` — hanya saja berada di
> `tiff32/Radiance_Images/`. **Folderlah yang mengidentifikasi produk, bukan nama file**, jadi pencarian global
> untuk `*radiance*.tif` tidak akan menemukan apa pun; gunakan pencocokan berdasarkan direktori sebagai gantinya.

### Rekaman sensor cahaya — yang telah dikalibrasi `.daq` + `.csv`

`process` juga menangani rekaman `.daq` di folder input Anda, dan **tidak**
membutuhkan citra apa pun untuk melakukannya: DAQ-U / DAQ-M / DAQ-E yang diterbangkan sendiri sudah merupakan
rekaman yang lengkap, dan folder yang hanya berisi berkas `.daq` merupakan masukan yang valid.

Sebuah DAQ dapat direkam **tanpa** kalibrasinya — itulah yang dilakukan oleh perekam umum
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) lakukan secara default: mereka menulis hitungan sensor mentah dan menandai berkas tersebut sehingga
Chloros mengambil kalibrasi pabrik sensor tersebut **melalui serial** (cache lokal terlebih dahulu,
kemudian MAPIR Cloud) dan menerapkannya. `process` menulis hasilnya kembali:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` memuat satu baris per pembacaan: cap waktu UTC, waktu integrasi, daya total,
lux fotopik/skotopik, PPFD (dan pembagian biru/hijau/merahnya), panjang gelombang puncak, lalu
spektrum penuh pada kisi panjang gelombang sensor itu sendiri. `.daq` mengimpor ulang tanpa perlu
dikalibrasi untuk kedua kalinya.

`<name>` adalah nama file perekaman itu sendiri. Dua rekaman di folder yang berbeda dapat berbagi
satu — pembacaan per pengambilan (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) yang disimpan dalam
milidetik yang sama oleh dua kamera — dan masing-masing tetap mendapatkan hasilnya sendiri: yang pertama mempertahankan
`<name>`, sedangkan yang berikutnya menambahkan folder yang membedakannya **setelah** nama
(`<name>__<capture>_daq_calibrated.daq`), sehingga tanggal dan waktu rekaman tetap menjadi bagian awal.

Jika berhasil, proses ini akan melaporkan `Light-sensor products written: N (calibrated .daq + .csv)`.
Teks dalam tanda kurung menjelaskan apa yang sebenarnya ditulis, sehingga terbaca
`(RAW COUNTS — this sensor has no calibration bundle)` untuk sensor tanpa bundel dan
`(N calibrated, M raw counts)` untuk folder yang berisi keduanya. Judul dari backend itu sendiri,
`[DAQ-EXPORT]` dan `[RUN-SUMMARY]`, memperoleh kata-katanya dengan cara yang sama — tidak ada satu pun
dari ketiganya yang dapat menyebut ekspor mentah sebagai ekspor yang telah dikalibrasi.

Perekaman DAQ-U / DAQ-M / DAQ-E yang bundel kalibrasinya tidak dapat diambil — Anda
sedang offline, atau sensor tersebut tidak memiliki file kalibrasi — akan **dilewati dengan alasan** pada
baris `[DAQ-EXPORT]`, tidak pernah ditulis sebagai berkas "terkalibrasi" yang berisi hitungan mentah.
Hubungkan ke internet dan jalankan kembali. Alasannya adalah yang sebenarnya
ditentukan oleh pembaca untuk berkas tersebut (skema tidak dapat dibaca, tidak ada paket, kesalahan penulisan), dan ringkasan
jalankan mencantumkan alasan-alasan **yang berbeda** — dua puluh berkas yang dilewati karena satu penyebab dicatat sebagai satu
penyebab, bukan dua puluh pengulangan dari penyebab tersebut. CLI menyebutkan setiap rekaman yang dilewati di akhir
proses, beserta alasannya:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Folder rekaman yang tidak mengekspor **satu pun** rekaman akan gagal: lihat
[Eksekusi yang tidak menulis gambar apa pun akan gagal](#a-run-that-writes-no-images-fails).

Rekaman yang namanya berakhiran `_calibrated` atau `_raw` (`field_raw.daq`) **tidak
diimpor**: itu adalah nama yang diberikan `process` kepada hasilnya sendiri, dan hasil yang dibaca kembali
sebagai rekaman akan dihitung dua kali. CLI menyatakan hal tersebut dan menyebutkan nama berkas —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— alih-alih melaporkan folder kosong. Ubah nama rekaman tersebut sebelum memprosesnya.

#### Rekaman DAQ-A diekspor sebagai hitungan mentah

Keluarga **DAQ-A** muncul sebelum sistem bundel per-serial dan tidak memiliki bundel kalibrasi
yang perlu diambil — alih-alih, perangkat ini dikalibrasi di lapangan menggunakan target reflektansi, itulah
sebabnya perangkat ini tidak pernah membutuhkannya. Menolak rekaman-rekaman tersebut membuat mereka tidak memiliki cara sama sekali untuk
mengeluarkan datanya, sehingga rekaman tersebut diekspor dengan **nama yang berbeda**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Nama file yang berbeda, bukan bendera di dalam file, karena klaim tersebut harus tetap utuh
saat dikirim melalui email hanya sebagai nama file. Header `.csv` menyatakan
`raw spectral sensor counts (NOT irradiance)` dan memperingatkan bahwa nilai-nilai tersebut dapat dibandingkan
**di dalam** file — yang persis merupakan tujuan penggunaan kalibrasi berbasis target — dan
bukan antar sensor. Kolom fotometrik yang bergantung pada daya (daya total, lux fotopik dan
skotopik, PPFD) ditulis sebagai **NULL** alih-alih diintegrasikan dari hitungan, dan ringkasan
pengujian menunjukkan `RAW COUNTS` sehingga "diekspor" dalam log tidak dapat dibaca sebagai irradiansi.

Perekaman **v1.01 / v1.02** versi lama (ditulis oleh DAQ-A-SD) tidak memiliki epoch per pembacaan,
hanya waktu penulisan berkas. Pencocok gambar↔sinar turun masih menolaknya — mencocokkan
bingkai dengan waktu penulisan akan menghasilkan kesalahan yang tidak terlihat — tetapi eksportir membacanya, dan
CSV-nya mencetak `clock=daq_created_on` sehingga produk tersebut menyatakan jam mana yang digunakan.

### Catatan

- `process` secara otomatis mendeteksi apakah folder Anda berjenis Survey3, LATTICE, atau campuran.
- Kemajuan proses ditampilkan melalui Server-Sent Events; antarmuka CLI menampilkan kemajuan real-time per-thread (Detecting, Analyzing, Processing, Exporting).
- Untuk Linux/Jetson, CLI memeriksa ruang swap dan mungkin memberikan peringatan sebelum memproses folder besar. Debayer yang mendukung tekstur juga secara otomatis menerapkan batasan frekuensi GPU pada Jetson berdaya rendah (Nano, Orin Nano).
- Jika berhasil, proses tersebut melaporkan berapa banyak produk gambar yang telah ditulis (`Image products written: N`, dengan `(plus M clamp masks)` jika proses unmix membatasi nilai piksel mana pun).
- **`process` berbagi backend yang sedang berjalan dengan aman.** Perintah ini secara otomatis me-restart backend yang sudah berjalan hanya jika backend tersebut dalam keadaan idle dengan proyek sebelumnya yang masih dimuat dan tidak memegang kamera, array, atau DAQ apa pun. Backend yang memegang salah satu dari elemen tersebut (GUI yang melakukan streaming array, skrip SDK, perekaman DAQ) digunakan apa adanya, tidak pernah di-restart, sehingga `process` dapat berjalan bersamaan dengan array yang sedang aktif. Jika backend melaporkan adanya proses yang sedang berjalan, `process` akan menunggu hingga 15 detik hingga proses tersebut selesai (proses yang baru saja selesai membutuhkan waktu sekitar satu detik untuk dibersihkan), kemudian menolak dengan kode kesalahan `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. Bendera global `--restart` tetap memaksa penggunaan backend baru, dan mengakhiri sesi kamera, array, atau DAQ apa pun yang berjalan di sana. Sebuah backend yang dijalankan sebagai layanan systemd di Linux tidak pernah di-restart: `process` menggunakannya, dan `--restart` menampilkan pesan `sudo systemctl restart <unit>` sebagai gantinya (lihat [Menjalankan backend sebagai layanan](#running-the-backend-as-a-service-headless)). Begitu pula dengan backend milik aplikasi Chloros: saat aplikasi terbuka, `process` menggunakannya, memuat proyeknya sebagai pengganti proyek aplikasi (backend yang dimulai sebagai penggantinya akan mengunci aplikasi yang terbuka tersebut).
- **Proses yang sedang berjalan akan mempertahankan proyeknya.** Selama proses pemrosesan sedang berlangsung — yang dimulai oleh aplikasi, `process`, skrip SDK, atau `chloros-mcp` — backend menolak untuk membuat, membuka, mengimpor ke dalam, menambah atau menghapus file, memindahkan, menghapus, atau mengubah pengaturan pemrosesan proyek, dengan kode status HTTP 409 `PROCESSING_IN_PROGRESS` dan `A processing run is in progress on project '<name>', so ... now is refused`; sebuah `process` yang menerimanya akan keluar dari `1`. Pengaturan tampilan (ukuran thumbnail) tetap tersimpan. Mode pemrosesan dan pengaturan reflektansi target proyek juga ditolak dengan cara yang sama (suatu proses tetap menggunakan mode saat dimulai). Sebuah tangkapan yang disimpan ke dalam proyek di tengah proses (dari aplikasi, CLI, SDK, atau `chloros-mcp`) disimpan seperti biasa, tetapi baru ditambahkan ke daftar file proyek setelah proses selesai; hal ini disebutkan dalam balasannya (`project_add`), begitu pula dengan rekaman sensor cahaya yang dihentikan ke dalam proyek. Sebelumnya, klien kedua dapat beralih ke proyek yang sedang terbuka di tengah proses, dan proses yang sedang berjalan diekspor ke proyek lain.
- **Ctrl+C menghentikan eksekusi di backend, bukan hanya di CLI.** CLI meminta backend untuk menghentikan pemrosesan, menampilkan pesan `Waiting for the backend to stop (Ctrl+C again to stop waiting)...`, dan menunggu hingga 120 detik untuk `Processing stopped.`, sehingga `chloros-cli process … && next` tidak memulai `next` pada backend yang masih sibuk dengan proses tersebut. Tekan Ctrl+C untuk kedua kalinya akan mengakhiri penantian. `process` kemudian akan keluar dari `130`, seperti halnya perintah lain yang terputus.

#### Proses yang tidak menulis gambar akan gagal

Jika Anda meminta produk dan proses tersebut tidak menulis **sama sekali** — hanya `project.json` dan
`calibration_data.json` — `process` menganggapnya sebagai kegagalan: ia mencetak
`Processing finished but wrote no image products.` dan **keluar dengan nilai bukan nol**, sehingga skrip dapat
mendeteksinya. Pesan tersebut menyebutkan nama folder proyek dan penyebab yang umum:

- folder input tidak dikenali sebagai data tangkapan (periksa tata letak dan `--input-level`), atau
- setiap produk yang diminta dilewati karena tidak berlaku untuk kamera-kamera tersebut (misalnya, meminta
  radiance/reflectance dari kamera yang hanya mendukung RGB).

Jalankan kembali dengan opsi `--verbose` dan periksa log backend untuk baris `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
yang menjelaskan alasan pengabaian per kamera yang tidak muncul dalam keluaran CLI.

Proses yang sengaja hanya menggunakan metadata — semua produk dinonaktifkan dan tanpa `--indices` — tetap
**berhasil**, karena keluaran gambar kosong adalah hasil yang benar dalam kasus ini.

#### Proses yang ekspornya tidak dapat diverifikasi akan gagal

Setelah proses berjalan, `process` memeriksa tag file yang diekspor menggunakan ExifTool. Jika ExifTool tidak dapat dijalankan, maka
akan ditampilkan `Verifikasi ekspor GAGAL: ExifTool tidak dapat dijalankan, sehingga tag dari N dari M file yang diekspor
tidak diperiksa (lihat baris [EXIFTOOL] di log backend)` and `Hasil diproses dan tetap
ada di tempatnya; jalankan ulang untuk memverifikasinya.`, does not print `Pemrosesan selesai!`, dan **keluar dengan nilai bukan nol**. Hasil
tersebut tetap disimpan.

#### Tangkapan mana yang diambil selama proses

Tangkapan yang selesai sebelum `process` dijalankan akan disertakan. Tangkapan yang disimpan selama proses tidak disertakan; `process`
memberi peringatan tentang berapa banyak yang terlewatkan, dan menjalankannya kembali akan menyertakannya.

#### Proses yang kehilangan gambar akan gagal

Jika sebuah gambar gagal selama proses, `process` mencetak `N image(s) failed and were not exported` dengan
penyebab paling umum, tidak mencetak `Processing complete!`, dan **keluar dengan nilai non-nol**. Produk-produk lainnya
tetap ditulis dan tetap berada di tempatnya, dan log backend mencatat setiap kegagalan.

Demikian pula dengan **proses yang hanya menggunakan sensor cahaya**: folder rekaman `.daq` tidak memiliki citra untuk diekspor
secara definisi, dan eksekusi dinilai berdasarkan `.daq` / `.csv` yang telah dikalibrasi yang ditulisnya sebagai gantinya.
Jika tidak menulis **sama sekali** — setiap rekaman dilewati — program akan menampilkan
`Processing finished but wrote no light-sensor products.`, memberi nama setiap rekaman dengan
alasan mengapa rekaman tersebut dilewati, dan **keluar dengan nilai non-nol**.

---

## `chloros-cli login`

Otentikasi mesin ini dengan akun cloud Chloros+. Kredensial disimpan dengan aman dalam `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Contoh

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (menghapus atau menduplikasi bagian dari kata sandi). Jika terjadi kesalahan 401, CLI secara otomatis akan mencoba kembali dengan `$$` yang-ditambahkan, kemudian dengan setengah kata sandi yang telah dihilangkan duplikatnya; jika percobaan ulang berhasil, sistem akan masuk dan menampilkan sintaks tanda kutip tunggal yang benar untuk digunakan di lain waktu.

> **Penggunaan tanpa antarmuka grafis/melalui skrip: tidak adanya sesi yang disimpan berarti akan muncul prompt login.** Perintah inti (`process`, `status`, `export-status`, `time-sync`, …) yang dijalankan tanpa sesi yang disimpan dalam cache akan meminta `Email:` / `Password:` melalui stdin sebelum melanjutkan. Jika stdin tertutup (cron, unit systemd), proses akan langsung gagal dengan pesan `login cancelled`; jika stdin berupa terminal terbuka atau pipa, proses akan menunggu masukan. Jalankan `chloros-cli login EMAIL PASSWORD` sekali per mesin sebelum menjadwalkan pekerjaan tanpa antarmuka grafis.

---

## `chloros-cli logout`

Menghapus sesi yang tersimpan dalam cache dan memaksa pengguna untuk masuk kembali pada panggilan berikutnya.

Keluar saat proses sedang berjalan tidak akan menghentikannya: proses tersebut akan selesai dalam mode pemrosesan yang sama saat dimulai, dan proses berikutnya akan menggunakan pemrosesan standar. `logout` menyatakan hal tersebut (`The processing run in progress finishes in <mode> mode; the next run uses standard`), begitu pula aplikasi saat Anda keluar keluar di sana.

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

Atur bahasa tampilan CLI (38 bahasa didukung, termasuk CJK, RTL, dan Indic). Akan beralih secara mulus ke bahasa Inggris pada konsol lama yang tidak dapat menampilkan skrip tersebut.

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

Perintah-perintah ini mengelola lokasi folder proyek default (dibagikan dengan antarmuka grafis).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` menampilkan folder tujuan proyek baru. Jika folder yang dikonfigurasi tidak lagi ada (dihapus, diganti namanya, atau berada di drive yang tidak terhubung), CLI akan beralih ke folder default, seperti yang dilakukan antarmuka grafis: folder default ditampilkan di stdout, peringatan yang menyebutkan nama folder yang hilang dikirim ke stderr, dan kode keluarnya adalah `0`.

---

## `chloros-cli update`

Hanya untuk Linux/Jetson. Memeriksa `version_url` dari `/etc/chloros/update.conf` dan, jika terdapat versi yang lebih baru, mengunduh `.deb` yang sesuai dan langsung menginstalnya dengan `sudo dpkg -i` (tanpa permintaan konfirmasi; `sudo` mungkin meminta kata sandi Anda). Pada Jetson, hanya sistem JetPack 6 yang ditawari pembaruan ini. Di Windows, perintah tersebut tidak melakukan apa-apa dan melaporkan `up to date`: perbarui dengan penginstal baru.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Di Linux/Jetson, CLI juga menjalankan **pemeriksaan pembaruan otomatis pada setiap kali startup** (tidak memblokir, tidak pernah menunda perintah): ia membaca `/etc/chloros/update.conf`, menyimpan hasilnya dalam cache selama 1 jam di `~/.chloros/update_cache.json`, dan menampilkan `Update available: vX.Y.Z / Run: chloros-cli update` jika ada versi yang lebih baru. Dilewati tanpa pemberitahuan jika terjadi kesalahan apa pun dan di Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` menonaktifkannya. Karena cache 1 jam, rilis yang diterbitkan dalam waktu satu jam tersebut masih dapat terbaca sebagai versi terbaru.

---

## `chloros-cli selftest`

Menjalankan uji coba 7 langkah: versi, ketersediaan port, startup backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), keberadaan model denoiser, kesiapan CUDA+penyaring noise. Skrip ini menggunakan backend yang sedang berjalan, atau memulai satu backend untuk pengujian dan menghentikannya setelah selesai. Skrip ini menampilkan `N/7 checks passed` dan pemeriksaan yang gagal, serta keluar dari `0` hanya jika ketujuh pemeriksaan lulus; pemeriksaan CUDA+denoiser akan gagal pada komputer yang tidak memiliki GPU CUDA (lihat [Verifikasi Instalasi](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Status dan kontrol grandmaster PTP. Host Chloros menjalankan grandmaster PTP; kamera LATTICE dan unit DAQ-E bertindak sebagai slave terhadapnya untuk cap waktu lintas perangkat (sekitar 1 ms antar perangkat). PTP menyelaraskan **cap waktu**: kamera dalam sebuah array memotret secara bersamaan karena pemicu perangkat keras M8 yang mengaktifkannya (dengan selisih waktu kurang dari 100 µs antar modul), bukan karena PTP. `chloros-cli time-sync` tanpa subperintah akan menjalankan `status`.

| Subperintah | Deskripsi |
| --- | --- |
| `status` | Menampilkan status grandmaster, prioritas BMCA, identitas jam, dan apakah paketnya mencapai jaringan: `health` adalah `OK` (`STOPPED` saat grandmaster tidak berjalan), atau `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` dengan baris `health_detail`, dan kemudian baris `WARNING:` dicetak ke stderr (stdout tetap berupa JSON). `sync_sent` / `announce_sent` menghitung hanya pesan yang berhasil terkirim; `*_send_errors` dan `last_send_error` menghitung yang tidak berhasil. Setelah koneksi kamera terputus dan kembali, grandmaster membangun kembali soketnya sendiri (`rebinds`, `last_rebind`). |
| `peers` | Daftar slave yang terdeteksi melalui Delay_Req (kamera + sensor DAQ-E). |
| `cameras` | Status kesehatan PTP per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Mulai ulang proses grandmaster. |
| `set-priority --priority1 N --priority2 N` | Mengabaikan prioritas BMCA. |

Grandmaster mengumumkan melalui NIC kamera (setiap adaptor dengan alamat `169.254.x.x` yang tercantum). Jika tidak ada yang aktif, sistem akan beralih ke adaptor rute default hanya jika adaptor tersebut terhubung melalui kabel, bukan Wi-Fi atau adaptor virtual (Tailscale, WSL), karena tidak ada kamera atau DAQ-E yang berada di balik adaptor tersebut. Jika tidak ada yang perlu diumumkan, sistem berjalan tanpa soket (`health` `NO_SOCKETS`), dan kamera atau DAQ berikutnya yang terhubung akan membangunnya kembali pada NIC yang aktif saat itu; `time-sync restart` melakukannya sekaligus.

Grandmaster berjalan dalam proses tersendiri, terpisah dari backend (`own_process` / `process_pid` di `time-sync status`), dengan prioritas tinggi di Windows (`process_priority`), sehingga backend yang sibuk (proses pemrosesan, ekspor, beberapa perintah sekaligus) tidak dapat menunda waktu yang dicatatnya; sebelumnya, satu penundaan semacam itu dapat menggeser jam setiap kamera sekaligus. Proses tersebut berakhir di backend; jika tidak dapat dimulai, grandmaster berjalan di dalam backend seperti sebelumnya (`own_process` false, dan log backend mencatat hal tersebut).

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

Kontrol kamera LATTICE. Setiap subperintah dialihkan melalui backend Chloros; backend mengelola kumpulan kamera (cam pool) sehingga panggilan CLI berikutnya menggunakan kembali handle terbuka yang sama.

### Opsi Umum (digunakan oleh sebagian besar subperintah)

| Bendera | Deskripsi |
| --- | --- |
| `-d, --device N` | Indeks kamera (default: 0). |
| `-s, --serial SN` | Nomor seri spesifik; menggantikan `--device`. |
| `--serials SN1,SN2,…` | Nomor seri yang dipisahkan koma untuk operasi multi-kamera. [Perintah multi-kamera sementara](#multi-camera-transient-sessions) hanya membuka kamera-kamera ini, dengan kamera pertama yang memicu pemicu. |
| `--all` | Beroperasi pada setiap kamera yang terdeteksi. |
| `--exposure US` | Waktu pemaparan dalam mikrodetik. |
| `--gain DB` | Gain dalam dB. |
| `--pixel-format FMT` | misalnya `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` pada M3M. Nama tersebut menentukan kedalaman bitnya dalam keluarga kamera itu sendiri: pada M3M, nama Bayer menentukan format Mono dari kedalamannya (`BayerRG8` menentukan `Mono8`), sedangkan pada M3C, nama Mono menentukan nama Bayer (`Mono12` menetapkan `BayerRG12`). |
| `--width N` / `--height N` | Dimensi gambar. |
| `--preset {default,high_quality,high_speed,triggered}` | Menerapkan preset pengaturan: format piksel, eksposur, gain, dan laju bingkai (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` mencantumkannya). Pada M3M, formatnya adalah format Mono dengan kedalaman yang sama: `high_speed` merekam `Mono8`, `high_quality` dan `triggered` serta `Mono12`. Sebuah bendera yang disertakan bersamanya (`--exposure`, `--gain`, `--pixel-format`) akan mengesampingkan pengaturan bawaan, dan pengaturan yang tidak disebutkan dalam pengaturan bawaan akan mempertahankan pengaturan kamera itu sendiri. Semuakecuali `triggered`, yang mengaktifkan kamera untuk mendeteksi perubahan sinyal pada Line 2 — jika tidak ada sinyal yang menggerakkan jalur tersebut, kamera akan menunggu selamanya daripada melakukan perekaman. |
| `-o, --output DIR` | Direktori keluaran (default: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Ukuran paket GVSP. `auto` menjalankan probe ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Alur Kerja Koneksi Pertama Kamera LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

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
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
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

#### Penemuan & Informasi

| Subperintah | Tujuan |
| --- | --- |
| `lattice info` | Menampilkan daftar kamera yang terhubung (pabrikan, model, nomor seri, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Menganalisis sistem host untuk konfigurasi kamera yang optimal. `--no-discover` melewati proses penemuan kamera (lebih cepat, analisis hanya menggunakan NIC). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Memeriksa/memperbaiki pengaturan NIC (di Linux, termasuk buffer penerimaan soket kernel — lihat [host Linux](#linux-hosts--socket-receive-buffers)); memperkirakan bandwidth/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Rekomendasi kemampuan jaringan backend skema stabil+ rekomendasi array (mengembalikan `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` secara default menggunakan sensor penuh, `2048` × `1536`. Fungsi ini keluar pada `0` untuk `ok`, `auto_capped_fps` dan `auto_shrunk`, serta `1` untuk `needs_force_slip` dan `error`, dengan atau tanpa `--json`. Permintaan ini sampai ke backend di `--backend-url` (default `http://127.0.0.1:5000`), bukan `CHLOROS_BACKEND_URL`. `auto_capped_fps` mempertahankan resolusi yang diminta tetapi membatasi fps target — baca `recommended.recommended_target_fps` dan lewati sebagai target koneksi; anggap sebagai keberhasilan, bukan sebagai kesalahan. `auto_shrunk` hanya muncul ketika bingkai yang diminta tidak dapat dikirimkan pada laju bingkai apa pun; connect tidak pernah memperkecil bingkai secara otomatis. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Analisis skenario tanpa membuka kamera (`--width` / `--height` default `2048` × `1536`). Baik ini maupun `network-analysis` memproyeksikan apa yang akan disiarkan oleh connect: tanpa `--pixel-format`, format per-kamera yang diterapkan oleh connect (Mono12 / BayerRG12), dan dengan `--packed auto` (default), keputusan [transportasi 12-bit terkemas](#packed-12-bit-transport---packed) yang akan diambil oleh Connect pada komputer ini — sehingga fps yang disebutkan adalah laju operasi array tersebut. Keduanya menampilkan keputusan pengemasan. **`--n-active` adalah jumlah total kamera yang terhubung, bukan hanya array ini**— tingkatkan nilainya ketika kamera mandiri melakukan streaming secara bersamaan, atau alokasi bandwidth dihitung berdasarkan permintaan yang tidak memperhitungkan jumlah kamera tersebut secara penuh (default: `len(--models)`). Selalu menampilkan baris agregat `Wire budget:` (MB/s yang diminta vs batas aman tabrakan) dan `Max cameras:`, serta menandai `** OVER-SUBSCRIBED**` ketika array melebihi kapasitas kabel — lihat [Model fps & burst array](#array-fps--burst-model). |
| `lattice gpu` | Tampilkan status GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Periksa atau perbarui firmware kamera. Pilihan lokal `.fwa` dikunci: berkas di `firmware/<MODEL_PREFIX>/` yang sesuai dengan `MIN_FIRMWARE_VERSION` build akan di-flash jika ada (hanya versi tertinggi sebagai cadangan), sehingga gambar vendor yang lebih baru yang disiapkan di disk tidak aktif sampai pin tersebut diubah — rilis yang lebih baru secara sengaja mencapai unit melalui manifest AWS yang ditandatangani, yang lebih disukai jika versi tersebut lebih baru. |
| `lattice presets [--apply NAME]` | Menampilkan daftar atau menerapkan preset kamera. `--apply` menentukan nilai-nilai yang menjadi bagian dari nama preset (format piksel, eksposur, gain, laju bingkai, pemicu) dan menampilkan pengaturan tersebut sebagaimana dibaca oleh kamera setelah: pada M3M, format Mono dari kedalaman preset; laju bingkai yang digunakannya, yang bisa lebih rendah dari yang diminta preset (`high_speed` meminta 60 fps; kamera full-frame pada koneksi 1 Gbps berjalan sekitar 35). Setiap pengaturan lain yang ditulis oleh program ini dikembalikan sesuai dengan yang ditemukan. `--apply default` mengatur ulang kamera ke pengaturan pabriknya: pengaturan pabrik dengan eksposur otomatis bebas pada Line2, batas throughput koneksi dan koreksi cacat dimatikan, serta keseimbangan putih yang dikalibrasi pada kamera RGB dipertahankan (nilai 1 pada jenis kamera lainnya). Tidak ada data yang disimpan ke memori kamera, sehingga saat dinyalakan kembali, kamera akan memulai dari memori internalnya seperti sebelumnya. Saat melakukan perekaman atau koneksi dengan perintah `--preset`, `default` tidak memiliki pengaturan prasetel. |
| `lattice status` | Status kamera langsung. |

#### Perekaman

| Subperintah | Tujuan |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Satu bingkai. **Menyimpan setiap jenis ekspor secara default**(`--processing all`); lihat [Tingkat Ekspor Perekaman](#capture-export-levels-the-all-default). `--processing` menerima `all`, `raw`, `debayered`, `radiance`, `reflectance`, atau `display` (`preview` diterima untuk `display`); nama lain apa pun akan ditolak sebelum kamera disentuh (keluar dari `1`). `--levels` menyimpan subset eksplisit (menggantikan `--processing`). Sebuah kamera yang tidak dimiliki oleh Chloros dibuka untuk perekaman, dan eksposurnya diatur berdasarkan adegan terlebih dahulu (paling lama beberapa detik), seperti yang dilakukan oleh [perintah sementara](#multi-camera-transient-sessions); ketika perekaman berakhir, apa pun cara berakhirnya, kamera tersebut dikembalikan ke kondisi semula (format, eksposur, jalur warna, setiap pengaturan yang diubah selama perekaman — daftar di bawah [Multi-Kamera (Sesi Sementara)](#multi-camera-transient-sessions)). Data DAQ yang digunakan untuk pembacaan rekaman tersebut disimpan sebagai berkas sidecar `.daq`, yang mencakup tangkapan mentah; `--no-daq` / `--daq-tolerance-ms` mengonfigurasinya ([DAQ yang membaca rekaman akan menyimpan](#the-daq-reading-a-capture-saves)). `--force-daq` masih diterima. `--jpeg-quality` = kualitas JPEG 1–100 (default 95).**File yang disimpan ke dalam proyek yang terbuka akan ditambahkan ke proyek tersebut.** Seperti di aplikasi, hasil tangkapan yang file-filenya disimpan di dalam folder proyek yang terbuka di Chloros akan ditambahkan ke daftar file proyek tersebut; file yang disimpan di tempat lain dibiarkan apa adanya. Perintah ini menampilkan satu baris yang menyatakan hal tersebut (`Project: ...`), atau, saat proses pengolahan sedang berjalan, bahwa file-file tersebut ditahan dan akan ditambahkan setelah proses selesai. Demikian pula dengan `multi-capture`, `array-capture` (`--smart` juga) dan `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Mengalirkan data ke disk hingga Ctrl+C. Frames dari proses `tiff`adalah file `raw` yang berisi rangkaian penangkapan dari kamera tersebut: mosaik sensor, satu bidang dengan nilai-nilainya sendiri, tidak terkompresi, beserta paket kalibrasi (bingkai 2048 x 1536 12-bit berukuran 6,0 MiB ditambah paket tersebut, apa pun pemandangannya), sehingga `process` dimulai dari bingkai-bingkai tersebut. `png` dan `jpg` berisi 8 bit, sehingga rangkaian tersebut merupakan bingkai yang telah melalui proses demosaic untuk ditampilkan, yang tidak menjadi titik awal bagi `process`. Dengan pengikatan DAQ (`daq pool-bind`) atau digabungkan, setiap bingkai mendapatkan `.daq` dan `.match.json`-nya sendiri di folder `daq/` dari rangkaian tersebut, karena `lattice capture` menyimpan satu; `--no-daq` menonaktifkan fitur tersebut, dan perintah berhenti mencetak jumlahnya. Setiap frame ditulis sekali, beserta metadata lengkapnya; saat menekan Ctrl+C, frame yang sudah diambil ditulis sebelum kamera dilepaskan. Proses berhenti sendiri saat drive penuh, mencetak `Failed:` / `Disk full:` dan keluar sebagai `1`; setiap bingkai yang tidak dapat ditulis juga akan keluar sebagai `1`. Kamera yang dibiarkan dalam mode siaga untuk pemicu perangkat keras (`TriggerMode` `On`) akan diatur ke mode bebas (free-run) selama proses pengambilan gambar. Saat proses pengambilan gambar berakhir, kamera dikembalikan ke kondisi semula — termasuk pengaturan pemicunya dan setiap pengaturan yang diubah selama proses (pencahayaan, penguatan, format, …) — kecuali `--keep-settings`, yang menyimpan pengaturan sesi ke memori kamera itu sendiri (mode bebas), atau `--clear-settings`, yang mengatur ulang memori tersebut ke pengaturan pabrik. Proses yang tidak merekam bingkai apa pun dianggap gagal: jika tidak ada bingkai selama 10 detik, proses akan berhenti sendiri, mencetak `No frames:` beserta alasannya, dan keluar dengan kode `1`. Frame yang tidak dapat diproses oleh writer dihitung sebagai frame yang terlewat, dan saat berhenti, sistem mencetak pesan `Dropped:` beserta alasannya (kecepatan bingkai kamera, writer, dan antrian); kecepatan bingkai yang lebih rendah atau format piksel 8-bit akan menyimpan semua frame tersebut. Perintah ini melakukan check-in ke backend setiap 2 detik; jika perintah ini mati tanpa menghentikan perekaman (`kill -9`, kehabisan memori), backend akan menghentikannya 30 detik setelah check-in terakhir dan mengembalikan kamera (`stop_reason` `client_gone`, dan sebuah baris di log backend). `multi-stream` dan `align-stream` melakukan hal yang sama. Proses yang dimulai di mana kamera membutuhkan waktu lebih lama untuk dibuka daripada batas waktu perintah (60 detik; 180 detik untuk `multi-stream` / `align-stream`) — kamera masih dalam proses booting — dibatalkan oleh backend: kamera dikembalikan ke posisi semula dan ditutup, serta tidak ada perekaman yang tetap berjalan (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Pratinjau MJPEG langsung berbasis browser: menghubungkan kamera, menampilkan URL aliran, dan membukanya di browser Anda. `--ae-damping` mengatur peredaman eksposur otomatis (0,4–100). |

##### Aliran langsung (MJPEG; tidak ada RTSP)

Chloros menyajikan video langsung sebagai **MJPEG melalui HTTP**(`multipart/x-mixed-replace`, batas `frame`) dari backend.**RTSP tidak didukung.** Buka URL di peramban, atau baca menggunakan klien MJPEG apa pun (`cv2.VideoCapture(url)` dari OpenCV, ffmpeg, VLC). Kamera harus terhubung ke backend terlebih dahulu (`lattice cam-connect`, `lattice array-connect`, `lattice viewer`, atau melalui antarmuka pengguna grafis).

| URL (`http://127.0.0.1:5000` + …) | Aliran |
| --- | --- |
| `/api/camera/stream/<serial>` | Pratinjau tampilan, hingga sekitar 30 fps. Untuk anggota array, `?max_h=` (default `720`) dan `?jpeg_q=` (default `80`) mengatur tinggi dan kualitas JPEG. Ini adalah URL yang dibuka oleh `lattice viewer`. |
| `/api/camera/<serial>/stream-annotated` | Pratinjau yang sama dengan overlay antarmuka pengguna yang digambar di: `zebra=1`, `crosshair=1`, `grid=3x3` (atau `9x9`), `histogram=1`, `peaking=1`, `spot=1`; tanpa diminta, pengaturan overlay yang disimpan proyek untuk kamera tersebut. |
| `/api/camera/<serial>/stream-reflectance` | Reflektansi 8-bit per-band: ρ = piksel ÷ bagian`X-Chloros-Pixel-Scale` (255 kecuali jika `refl_max` diaktifkan). `fps` (default `10`), `jpeg_q` (default `95`), `max_h` (default `0` = native), `band=<name>`, `align=1`. JPEG, sehingga mengalami kehilangan data. Ditolak dengan kode status HTTP 409 jika kamera tidak memiliki nilai reflektansi (kamera RGB, tanpa paket kalibrasi, tanpa pembacaan DAQ). |

- **Pencocokan bingkai antar kamera.**Setiap bagian yang dikirim oleh**anggota array**, pada ketiga aliran, membawa header `X-Chloros-Group-Id` (penghitung grup sinkronisasi array), `X-Chloros-Frame-Id`, dan `X-Chloros-Timestamp-Ns`. Bagian dari kamera berbeda dengan `X-Chloros-Group-Id` yang sama berasal dari pemicu yang sama. Kamera mandiri tidak memiliki grup dan tidak mengirimkan ID grup. Browser mengabaikan header per-bagian; klien yang mengurai aliran multipart itu sendiri yang membacanya.
- **Dijeda saat tampilan array terbuka.** `/api/camera/stream/<serial>` dari anggota array mengirimkan bingkai statis "paused" saat aliran langsung gabungan array terbuka (misalnya, tampilan array pada GUI), dan dilanjutkan saat ditutup.

#### Penyesuaian Sensor

| Subperintah | Tujuan |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Membaca/menulis node GenICam apa pun. `--set ExposureTime=…` (`Gain=…`) tanpa opsi `ExposureAuto` (`GainAuto`) yang mematikan mode otomatis terlebih dahulu, seperti yang dilakukan oleh `lattice exposure --set`; nilai yang ditolak oleh kamera akan mencetak `Failed to set <node>: …` di stderr dan keluar dari `1`. `--get` dan tampilan berkelompok default mencetak `note:` di bawah `ExposureTime` / `Gain` saat mode otomatis tersebut berjalan — lihat [Membaca eksposur dan gain dalam mode otomatis](#reading-exposure-and-gain-under-auto-mode). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Eksposur & AE. Nilai eksposur `--set` atau target `--brightness` dianggap sebagai milik Anda (kontrol eksposur otomatis Chloros membiarkannya apa adanya); gain tetap seperti semula, sehingga dalam mode otomatis, sistem terus menyesuaikan ke arah target hingga Anda mengatur `lattice gain`. `--auto` mengembalikan pengaturan eksposur, `--brightness-auto` mengembalikan target ke target otomatis Chloros. Tanpa bendera, sistem membaca node eksposur, dengan nilai `note:` yang sama seperti `configure --get`. Jika kamera menolak nilai `--set`, maka akan mencetak `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` di stderr — mode otomatis yang sebelumnya dimatikan akan dipulihkan, bukan dibiarkan dalam keadaan Off — tidak mencetak baris `Exposure: … us (auto OFF)`, dan keluar dari `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Penguatan & penguatan otomatis. Nilai `--set` yang ditolak oleh kamera mencetak `Failed to set Gain: …` di stderr, tidak ada baris `Gain: … dB (auto OFF)`, dan keluar dari `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | ROI sensor & binning. Binning selalu **Rata-rata** — lihat di bawah. |
| `lattice format [--set FMT] [--list]` | Format piksel. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Pemicu perangkat keras/perangkat lunak. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (tanpa bendera = WB sekali jepret) | Operasi WB. Hanya untuk kamera RGB/Bayer; tidak beroperasi (dilewati) pada M3M mono. Mode satu kali mempertahankan white balance yang dihitungnya (`BalanceWhiteAuto` dan penguatan); setiap pengaturan lain yang ditulisnya dikembalikan seperti semula, dan semuanya dikembalikan jika mode satu kali tidak dapat dijalankan. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Jalur pemrosesan warna tampilan RGB. `natural` (default) adalah pengolahan langsung yang sederhana; `enhanced` menambahkan defringe + vibrance + kontras lokal CLAHE untuk tampilan paritas hub penuh dengan biaya pemrosesan per bingkai sekitar 2× lipat, sehingga menghasilkan laju bingkai **langsung** yang lebih rendah — tangkapan yang disimpan selalu mendapatkan hasil pemrosesan penuh dalam kedua kasus. Hanya untuk kamera RGB/Bayer; dilewati pada M3M mono. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Saturasi/kontras pratinjau langsung (kamera filter RGB); `100` netral, `--reset` mengatur keduanya menjadi 100. Tangkapan yang disimpan tidak terpengaruh. Dilewati pada M3M mono. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (tanpa bendera = baca) | Tampilkan **Highlights** (kamera filter RGB): kurangi kecerahan piksel paling terang dan paling putih — misalnya, permukaan putih cerah yang ditampilkan sebagai gumpalan putih datar. `0` = mati, `-100` = paling kuat. Pratinjau langsung dan tangkapan pada tingkat tampilan mengikuti pengaturan ini; radian/reflektansi tidak pernah mengikuti, dan fitur ini tidak dapat memulihkan data sensor yang terpotong (kurangi eksposur untuk itu). Dilewati pada M3M mono. |
| `lattice filter [--list]` | Menampilkan daftar model filter yang dikenali oleh Chloros (`RGN-IMX265`, `OCN`, `NGB`, …). Filter kamera dibaca dari string modelnya dan tidak dapat diatur: `--set NAME` hanya mencari model, lalu keluar dari `1` dengan `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Memeriksa node daya/termal; mengaktifkan/menonaktifkan mode idle berdaya rendah. `--sleep` membuat kamera tetap dalam keadaan tidur (batas throughput tautannya diaktifkan (`DeviceLinkThroughputLimitMode` `On`) pada batas bawahnya, mode hemat daya diaktifkan jika kamera memilikinya) dengan setiap pengaturan lainnya dikembalikan seperti semula; mode tidur yang gagal dikembalikan secara utuh. Proses bangun mengembalikan koneksi ke kondisi saat mode tidur menemukannya (batas, mode, dan penundaan paket `GevSCPD`, yang dipertahankan kamera pada 0 selama batas aktif; catatan disimpan di komputer ini, sehingga proses bangun setelah backend dimulai ulang juga menemukannya), atau, untuk kamera yang tidak dimasukkan ke mode tidur oleh komputer ini namun tampak dalam mode tidur (batasnya aktif pada batas bawah), dengan batas dinonaktifkan dan nilai `GevSCPD` dibiarkan 0 pada 80 (pengaturan pabrik); batas yang ditetapkan di atas batas bawah dibiarkan apa adanya. Probe dan proses bangun mengembalikan nilai yang ditulis saat dibuka. |

> **Binning selalu menggunakan metode Rata-rata.** `lattice resolution --binning N` menetapkan binning Rata-rata dan
> tidak menyediakan opsi pergantian mode. Binning Sum mengalikan setiap DN dengan N², yang menghabiskan sekitar dua stop
> ruang headroom highlight per langkah binning; Chloros membagi faktor tersebut kembali sebelum
> rantai radiometrik, tetapi piksel yang didorong oleh Sum hingga mencapai batas atas sensor akan terpotong dan tidak dapat
> dipulihkan. Hal ini tidak memberikan keuntungan apa pun yang tidak dimiliki oleh mode Rata-rata — indeks bersifat invarian skala dan skalar radian
> dikalibrasi pada skala Rata-rata. Jika kamera tiba dalam keadaan sudah disetel ke Sum (ditinggalkan
> dalam keadaan tersebut oleh alat lain, atau akibat reset pabrik), Chloros mendeteksinya dari pembacaan node dan
> melakukan kompensasi; menjalankan `lattice resolution --binning N` akan mengembalikannya ke Average. Bendera 1.2.0
> masih diterima sehingga skrip lama tetap berjalan, tetapi hanya menampilkan peringatan:
> penggabungan piksel tetap pada mode Average.

> **Perubahan pengelompokan, format piksel, dan pemicu pada kamera tidak berlaku.** Pada kamera di mana
> backend tetap terbuka dan melakukan streaming (`cam-connect`, antarmuka pengguna), `lattice resolution --binning`
> tidak berlaku (muncul pesan bahwa pengelompokan piksel masih berlaku, `Binning: 1x1`), dan
> `lattice format --set` serta `lattice trigger --mode` ditolak oleh kamera
> (`SC_ERR_ERROR -1001`). Lepaskan terlebih dahulu (`lattice cam-disconnect -s <SN>`), atau atur
> saat terhubung (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Membaca eksposur dan gain dalam mode otomatis

Saat `ExposureAuto` (`GainAuto`) kamera sedang berjalan, node `ExposureTime` (`Gain`) menyimpan **nilai terakhir yang ditulis**, bukan apa yang sedang dilakukan kamera saat ini. Untuk kamera yang backend-nya tetap terbuka (`cam-connect`, `array-connect`, GUI), `lattice configure --get`, tampilan default `lattice configure`, dan `lattice exposure` (tanpa bendera) oleh karena itu menampilkan nilai node dan, di bawahnya, catatan berisi bingkai terbaru kamera:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Baca nilai bingkai dari catatan; nilai node hanyalah penulisan terakhir. Melalui HTTP, respons `POST /api/camera/<serial>/read-nodes` yang sama mempertahankan `nodes` tanpa perubahan dan menambahkan `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, milik bingkai terbaru) dan `notes` (kalimat di atas). Keduanya hanya ada selama mode otomatis bukan `Off` dan bingkai sudah diketahui; kamera yang dibuka hanya untuk pembacaan tidak memiliki bingkai dan tidak mendapatkan keduanya. `lattice gain` (tanpa bendera) hanya mencetak simpul — gunakan `lattice configure --get Gain` untuk catatan.

##### Eksposur pada anggota array

Saat array melakukan streaming, setiap anggota dijalankan dengan **`AcquisitionFrameRateEnable`**, pada laju bingkai yang sedikit di atas laju pemicu array: periodenya adalah periode pemicu dikurangi 15% (minimal 3 ms), atau laju maksimum kamera itu sendiri jika pemicu sudah mendekati nilai tersebut. Oleh karena itu, pengaturan**`configure --get` pada anggota**akan membaca**`AcquisitionFrameRateEnable`**sebagai**`True`**dan**`AcquisitionFrameRate`**pada pengaturan tersebut; pengaktifan akan dinonaktifkan kembali saat array berhenti.**Jangan tulis kedua node ini pada anggota array** — array akan menyinkronkannya dengan pemicunya dan menuliskannya kembali.

Pengaturan tersebut juga merupakan batas maksimum eksposur anggota: sekitar 85 ms pada pemicu 10 fps (periode 100 ms), pada **ROI atau binning apa pun**. Sebelum versi 1.3.0, ROI kecil membatasi eksposur anggota pada periode free-run sensor (4,9 ms pada 768 × 304), terlepas seberapa lambat array tersebut memicu.**Nilai `ExposureTime` manual yang lebih lama dari batas maksimum tersebut akan memperlambat pemicu agar sesuai dengan batas tersebut** saat ditulis menggunakan `lattice exposure --set` atau `configure --set` (atau melalui antarmuka pengguna). Array tersebut terlebih dahulu membuka batas atas anggota tersebut untuk nilai tersebut dan membiarkan laju yang lebih lambat berjalan selama beberapa periode pemicu, sehingga penulisan dapat memakan waktu beberapa detik lebih lama; pemicu kemudian tidak akan berbunyi lebih cepat dari yang diizinkan oleh eksposur. Pada array laju otomatis, laju yang sebelumnya digunakan akan kembali setelah setiap anggota yang eksposur manualnya memperlambat laju tersebut kembali ke eksposur otomatis. Nilai yang tidak dapat diambil oleh kamera itu sendiri tetap ditolak, dengan rentang yang diterima tercantum dalam pesan kesalahan, dan eksposur otomatis anggota tersebut dikembalikan jika sebelumnya dimatikan hanya untuk penulisan tersebut. Resep eksposur `apply` tidak memperlambat pemicu: pemicu tersebut dibatasi pada batas atas. Chloros tidak pernah mempersingkat eksposur manual agar sesuai dengan pemicu yang lebih cepat; kecepatan yang dikunci (`array-connect --target-fps F`) yang lebih cepat daripada yang diizinkan oleh eksposur akan membuat anggota tersebut tetap pada batas atas selama waktu yang cukup untuk itu, dan anggota tersebut mengambil pemicu tidak lebih cepat daripada yang diizinkan oleh eksposurnya.

#### Kalibrasi & Sensor

| Subperintah | Tujuan |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Mengkalibrasi dari target reflektansi. Hingga `--attempts` bingkai (default `30`) dicari untuk itu; bingkai yang tidak lengkap (kehilangan paket GVSP) menghabiskan satu upaya, bukan seluruh proses. Proses ini gagal pada `no target detected in N attempts`, atau pada `no complete frame in N attempts: <last error>` jika tidak ada bingkai yang tiba secara utuh. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | Spektrum dan magnitudo pita DAQ-M, yang dibaca melalui Bluetooth (BLE) via backend. `--mac` adalah alamat MAC BLE sensor itu sendiri dan wajib disertakan pada ketiga tindakan tersebut: tidak ada nilai default, karena setiap unit memiliki alamatnya sendiri (`daq pool-discover --only ble` mencantumkannya); tanpa kode tersebut, perintah akan keluar dengan kode `1`. Jika tidak ada tindakan yang dipilih, perintah akan menampilkan panduan pengguna. `--camera SERIAL` mengintegrasikan magnitudo pita yang ditampilkan melalui respons spektral yang diukur oleh kamera tersebut, bukan kurva fleet dari model filter — kamera RGB, dengan paket kalibrasi kamera telah dimuat (terhubung) atau disimpan dalam cache di komputer ini. Judul menunjukkan kurva mana yang digunakan: `Band magnitudes (camera <SERIAL>):`, atau `Band magnitudes (model <filter>):` jika kamera tidak memilikinya (bukan kamera RGB, atau tidak ada paket yang dimuat atau disimpan dalam cache) dan kurva standar digunakan sebagai gantinya. Melalui HTTP, `POST /api/dls/cli-read` menerima `camera_serial` dan mengembalikan `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Menerapkan koreksi vignette pada gambar yang sudah ada, masing-masing dengan flat field lensa tersendiri (dari Model EXIF-nya). `--lens-model` diterapkan pada gambar yang Model-nya tidak menyebutkan kamera; gambar yang Model-nya bertentangan akan dilewati. |

#### Multi-Kamera (Sesi Sementara)

Perintah sementara membuka kamera untuk satu kali eksekusi dan menutupnya saat selesai: `multi-capture` tanpa array persisten yang terhubung, `multi-stream` dan `multi-test` di sini, dan `align-calibrate`, `align-apply`, `align-stream`, serta `index --live` di bawah ini. Aturan-aturan berikut berlaku untuk semuanya:

- **`--serials SN1,SN2,…` memilih kamera-kamera tersebut.** Hanya kamera-kamera itulah yang dibuka, dan yang pertama memicu pemicu (untuk `align-calibrate`, `--reference` jika diberikan). Kamera serial yang tidak terhubung ke jaringan akan menyebabkan eksekusi gagal sebelum kamera mana pun dibuka. Tanpa `--serials`, semua kamera di jaringan akan dibuka. Kamera yang sudah dipegang oleh Chloros (melalui antarmuka pengguna, `cam-connect`, `array-connect`) akan menolak proses eksekusi, baik disebutkan oleh `--serials` maupun tidak: kamera tersebut berbagi jaringan yang menjadi sasaran proses tersebut. (`index --live` tanpa `--serials` justru membuka kamera-kamera yang diaktifkan dalam profilnya: lihat [Indeks / Matematika Vegetasi](#index--vegetation-maths).)
- **Kamera-kamera tersebut berbagi jaringan dengan cara yang sama seperti yang dilakukan oleh `array-connect`.** Setiap kamera disesuaikan kecepatannya sesuai porsinya dari apa yang dibawa oleh tautan host (penundaan antar-paket, bukan batas throughput kamera, yang batas bawahnya sebesar 31,25 MB/s sudah terlampaui oleh empat kamera pada 1 GbE), sehingga empat kamera dapat beroperasi pada tautan 1 Gbps. Ketika kamera-kamera tersebut tidak dapat mengirim data secara bersamaan pada kecepatan terendah yang dapat mereka capai, mereka diaktifkan satu per satu, masing-masing dengan tautan eksklusif ke dirinya sendiri, dan perintah tersebut menampilkan `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Jika bahkan satu kamera pun tidak muat, proses tersebut ditolak sebelum kamera mana pun mulai streaming: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, keluar `1`.
- **Setiap kamera harus mengirimkan data, atau proses akan gagal.** Kamera yang tidak mengirimkan frame apa pun merupakan kesalahan: perintah menampilkan `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, di mana `<why>` adalah kesalahan pengambilan data kamera tersebut, diikuti dengan penyebabnya, lalu keluar dengan pesan `1`. Frame yang tiba dengan kehilangan paket GVSP (`IncompleteBufferError: GVSP packet loss …`) berarti jaringan tidak mampu menangani bandwidth gabungan kamera-kamera tersebut (`Every missing frame arrived with GVSP packet loss: …`): periksa kecepatan tautan host dan uplink switch (`lattice network`), gunakan bingkai jumbo, atau kurangi jumlah kamera yang diaktifkan dengan `--serials`. Kamera yang tidak mengirimkan data apa pun biasanya tidak menerima sinyal pemicu dari kamera utama: periksa kabel sinkronisasi M8-nya, lalu jalankan `lattice multi-detect`. Fungsi masing-masing perintah terhadap kelompok frame yang kehilangan satu kamera tercantum pada barisnya.
- **Setiap kamera dikembalikan ke kondisi semula.** Saat proses berjalan berakhir, setiap kamera akan kembali ke kondisi semula saat proses dimulai: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, dan `LineMode` / `LineSource` pada `Line2` dan `Line3` (dan sinyal masukan pemicu yang telah dibersihkan dari gangguan pada array memberikan sinyal pada jalur slave, `LineFilterSelector` / `LineFilterWidth`); `PtpEnable` (dan set `PtpSlaveOnly` / `PtpDomainNumber` yang menyertainya); `ExposureAuto` / `GainAuto` (dan `ExposureTime` / `Gain` di mana nilainya diatur secara manual); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (dan setiap `BalanceRatio` di mana keseimbangan putih diatur secara manual); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (dan `Gamma` di mana pengaturan gamma diaktifkan), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; ukuran paket (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); laju pengiriman, `DeviceLinkThroughputLimitMode` (dan `DeviceLinkThroughputLimit` di mana modenya adalah `On`), penundaan antar-paket `GevSCPD` dan selisih waktu mulai `GevSCFTD` yang diberikan oleh setiap kamera; data chunk yang diaktifkan oleh setiap aliran (`ChunkModeActive`, masing-masing `ChunkEnable` di bawah `ChunkSelector`-nya); `AcquisitionFrameRateEnable` (dan `AcquisitionFrameRate` di mana pengaktifan dalam keadaan aktif — jika dinonaktifkan, laju bersifat hanya baca dan membaca sebanyak yang diizinkan oleh kecepatan, sehingga ini bukan pengaturan). Hanya perbedaan yang ditulis, kamera berhenti terlebih dahulu, kecepatan dan laju bingkai sebelum eksposur yang membutuhkan periode bingkai (kamera membatasi `ExposureTime` pada satu periode bingkai). Dengan `--keep-settings`, kabel pemicu dan garis sinkronisasi diputus, kamera beroperasi secara bebas, dan sisanya disimpan ke memori kamera sesuai pengaturan saat dijalankan; `--clear-settings` mengembalikan pengaturan pabrik ke sana.
- **Eksposur diatur berdasarkan adegan sebelum bingkai pertama disimpan.** Proses pengambilan gambar ini mengambil kelompok bingkai yang tersinkronisasi dan membuangnya sambil mengatur eksposur dan gain setiap kamera berdasarkan bingkai-bingkai tersebut. Tujuannya adalah mencapai kecerahan rata-rata pada target kamera dengan tidak lebih dari 1% bingkai yang terpotong; adegan dengan sorotan terang akan berhenti sebelum mencapai target. Proses ini biasanya membutuhkan 2–6 kelompok bingkai, dan paling lama 5 detik.
  - Setiap kamera mengarahkan ke target yang ditentukan oleh array terhubung saat proses dimulai, melalui filternya: TargetBrightness **40**pada kamera multispektral (RGN, OCN, NGB) dan**60** pada kamera RGB. Angka tersebut setengah dari target eksposur yang diberikan oleh 80 (tiga perempat pada RGB), sehingga panel reflektansi yang terang cenderung tidak terpotong. `--brightness N` (0-255) justru menetapkan target setiap kamera (`multi-test` tidak memiliki `--brightness`: fitur eksposur otomatisnya dinonaktifkan). Nama log backend mencantumkan target setiap kamera: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - Eksposur didahulukan, hingga batas eksposur otomatis kamera itu sendiri. Gain hanya mencakup sisa yang tersisa.
  - Nilai eksposur dan gain yang ditentukan dipertahankan selama seluruh proses pengambilan gambar, sehingga setiap bingkai dari `multi-stream` diambil dengan eksposur yang sama. Pengambilan gambar yang harus mengikuti perubahan cahaya harus dilakukan pada array yang terhubung (`array-connect`), yang eksposur otomatisnya terus menyesuaikan.
  - `--exposure` dan `--gain` dipertahankan persis seperti yang ditentukan. Dengan keduanya diberikan, tidak ada bingkai yang dibuang.
  - Saat proses pengambilan gambar berakhir, pengaturan eksposur otomatis setiap kamera diaktifkan kembali.
  - Log backend melaporkan hasilnya: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …`, atau `exposure NOT solved after …` beserta alasannya, serta `(not settled)` pada setiap kamera yang tidak terdeteksi. Dalam kasus tersebut, bingkai tetap diambil.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` mengubah batas waktu 5 detik; `0` menonaktifkan fitur ini (lihat [Variabel Lingkungan](#environment-variables)).

**Selama Chloros sedang mengendalikan kamera apa pun** (melalui antarmuka pengguna, `cam-connect`, atau susunan kamera yang terhubung), `multi-stream`, `multi-test`, `multi-detect`, `align-apply`, `align-stream`, dan `index --live` menolak untuk dijalankan, karena mereka membuka setiap kamera di jaringan dan pembukaan kedua pembukaan ulang kamera yang sedang ditahan dapat membuat backend crash. (`index --live` menolak ketika kamera yang akan dibuka sedang ditahan: baik kamera `--serials` itu sendiri atau kamera-kamera dalam profilnya, serta kamera apa pun untuk profil yang tidak mencantumkan nama kamera.) Backend merespons HTTP 409 `CAMERA_HELD`, menyebutkan setiap kamera yang ditahan dan perintah pelepasannya (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), dan perintah tersebut akan keluar (`1`). Jika sebuah array terhubung, `array-capture --continuous` akan merekamnya menggantikan `multi-stream`, dan `array-capture` (sejajar secara default) menyimpan kumpulan yang sejajar menggantikan `align-apply`; `multi-capture` dan `align-calibrate` menggunakan array terhubung yang menampung kamera-kamera mereka alih-alih membukanya.

| Subperintah | Tujuan |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Menampilkan daftar semua kamera dengan peran sinkronisasi; dengan `--serials`, hanya yang memiliki peran tersebut, sedangkan yang pertama sebagai `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Satu bingkai yang disinkronkan dari setiap kamera. Menyimpan **semua jenis ekspor secara default**saat array persisten terhubung, dan pembacaan DAQ yang digunakan ([Pembacaan DAQ yang menyimpan tangkapan](#the-daq-reading-a-capture-saves)); fallback sementara tanpa array**hanya di-debayered** (jalankan `array-connect` terlebih dahulu untuk sisanya). `--brightness` hanya berlaku untuk mode cadangan: array yang terhubung mempertahankan pengaturan eksposur otomatisnya sendiri. Mode cadangan tidak menyimpan `.daq` (`--no-daq` / `--daq-tolerance-ms` tidak berpengaruh di sana), dan `multi-capture` tidak menerapkan `--force-daq`. Dalam mode cadangan, kamera yang tidak mengirimkan bingkai akan gagal dalam proses perekaman (`Error: multi-capture failed: no frame from …`, keluar dari `1`) dan tidak ada file dari kelompok bingkai tersebut yang disimpan. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Frame yang disinkronkan dalam aliran (sementara), `--fps` default `3`, hingga Ctrl+C atau `--count` kelompok. Kelompok bingkai yang tidak menghasilkan apa pun dari kamera tidak ditulis, dan tiga kali berturut-turut akan mengakhiri aliran. Jika ada kelompok yang dilewati, proses akan berakhir dengan mencetak `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, ditambah `Error: the stream stopped: …` yang menyebutkan nama kamera saat aliran berakhir, dan keluar dari `1`X. Program akan berhenti sendiri saat drive penuh, mencetak `Failed:` / `Disk full:` dan keluar dengan kode `1`; setiap file yang tidak dapat ditulis juga akan keluar dari `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Uji sinkronisasi waktu GPIO (sementara), pemicu default `--count` dan `10`. Menampilkan bagaimana jaringan dibagi (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, dan baris `Serial emit:` ketika kamera dipicu satu per satu, di mana selisih waktu tersebut kemudian tercermin dalam cap waktu), satu baris `Sync group: master SN:<serial> -> SN:<serial>, …` per grup sinkronisasi (`(no slaves: not synced)` untuk kamera yang tidak terhubung ke kabel sinkronisasi, yang dipicu secara mandiri), kemudian satu baris per pemicu: `[  0] SN:<serial>=<timestamp> \| …`, dengan `SN:<serial>=(no frame)` untuk kamera yang tidak mengirimkan apa pun pada pemicu tersebut. Kemudian baris `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` dan `SN:<serial>: mean <±x> us, jitter <y> us` per kamera, setiap kamera dibandingkan dengan master dalam satu siklus jam: pemicu yang cap waktunya mencampurkan jam perangkat kamera dan PTP (PTP mati atau masih mengunci) diabaikan, dengan baris `Note:` menghitungnya. Setiap frame yang terlewat akan membuat tes gagal (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, keluar dari `1`); tiga pemicu yang tidak lengkap berturut-turut akan mengakhiri tes lebih awal. Tes ini juga gagal (keluar dari `1`) jika kurang dari dua pemicu membuat semua kamera berada pada satu jam (`… the clocks cannot be compared`), atau jika jam-jam tersebut tidak pernah selaras: Jam PTP yang selisihnya lebih dari 1 ms (ditambah selisih pada lapisan serial) dari master (`… PTP is not locked`), atau jam perangkat yang selisihnya berubah lebih dari itu di antara pemicu. Proses ini akan memulai grandmaster PTP jika belum berjalan, mengaktifkan PTP pada setiap kamera (seperti yang dilakukan oleh `array-connect`, terlepas dari kondisi jaringan), dan sebelum pemicu pertama, menunggu hingga jam PTP setiap kamera terkunci pada grandmaster dan stabil: `PtpStatus` Slave pada satu grandmaster, dengan selisih waktu tidak lebih dari 100 us darinya dan stabil selama 3 detik, serta setiap kamera terkunci seperti itu, tanpa jeda, selama 120 detik — sekitar 2 menit sejak dinyalakan. Jam yang baru saja terkunci dapat mengalami selisih hingga ~200 us selama sekitar 2 menit setelah statusnya terbaca sebagai Slave, dan `PtpOffsetFromMaster` kamera itu sendiri tidak menunjukkannya; 120 s membuat kamera-kamera tersebut berada dalam selisih ~10 us dari jam yang telah stabil. Sebuah kamera masih terkunci sejak pengujian sebelumnya yang membiarkan PTP aktif, sebagaimana dilihat oleh backend ini, menghitung dari penguncian tersebut, sehingga pengujian yang dilakukan segera setelah pengujian sebelumnya tidak perlu menunggu lagi; kamera yang dihidupkan saat pengujian, atau kamera yang ditemukan dengan PTP aktif namun belum pernah dilihat oleh backend ini dalam keadaan terkunci (dihidupkan di tempat lain, atau sebelum backend di-restart), menunggu selama 120 s, dan dalam kumpulan kamera yang baru saja terkunci, kamera-kamera tersebut tetap menunggu giliran mereka. Kamera yang keluar dari status Slave, melewati batas 100 us, atau mengikuti grandmaster lain selama 3 pembacaan berturut-turut (1,5 s) selama masa tunggu, akan memulai hitungannya kembali; satu atau dua pembacaan semacam itu dicatat (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) dan penghitungan berlanjut, sedangkan ID grandmaster yang tidak dapat dibaca dianggap sebagai ID yang tidak dikenal, bukan sebagai perubahan. Jika hitungan setiap kamera dimulai ulang secara bersamaan (dalam waktu satu detik), hal itu menandakan pergerakan waktu grandmaster, bukan kamera: log backend mencatat `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` mencantumkannya dalam `grandmaster_steps`, dan penundaan yang kemudian habis waktunya memulai kesalahannya dengan `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Batas waktu tidak diperpanjang untuk hal ini. `--ptp-settle SECONDS` menetapkan waktu penyelesaian untuk eksekusi (0–600; `0` untuk tanpa penundaan penyelesaian). Waktu tunggu dibatasi pada waktu stabilisasi ditambah 90 detik (210 detik secara default, minimal 90 detik), dan perintah tersebut menunggu backend selama 240 detik ditambah waktu stabilisasi (secara default 360 detik). Perintah ini menampilkan pesan `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, dan log backend mencatat satu baris setiap 5 detik yang berisi status dan offset setiap kamera, berapa lama kamera tersebut telah terkunci, serta berapa lama waktu yang tersisa: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Setiap kamera dinyalakan dengan PTP dimatikan; dari situ, jam-jam tersebut terkunci dalam waktu sekitar 10 detik. Setelah itu, setiap kamera dikembalikan ke kondisi yang ditemukan saat pengujian — PTP-nya, eksposur otomatis, dan gain yang dinonaktifkan (Off) selama pengujian, serta sisanya (kecuali `--keep-settings`) — sehingga kamera yang dibiarkan dengan PTP aktif (`lattice configure -s <serial> --set PtpEnable=True`, atau pengujian dengan `--keep-settings`) tetap terkunci, dan setelah backend ini mendeteksi bahwa kamera tersebut telah terkunci selama waktu stabilisasi, pengujian selanjutnya tidak akan menunggunya. Jika jam tidak stabil selama masa tunggu, tes akan gagal sebelum dipicu (`PTP: NOT locked …`, keluar `1`) dan menjelaskan apa yang harus dilakukan: kamera yang tidak pernah terkunci (`PtpStatus Listening`) berarti grandmaster tidak dapat menjangkaunya (`time-sync status`); kamera yang terus terputus diberi nama sesuai dengan berapa lama ia telah terkunci (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); jam masih dalam proses konvergensi: periksa `time-sync status`, dan agar kamera terkunci sebelum pengujian, aktifkan PTP dan biarkan tetap aktif (`lattice configure -s <serial> --set PtpEnable=True` untuk setiap kamera, atau jalankan pengujian dengan `--keep-settings`; pengujian ini mengembalikan pengaturan yang ditemukan, sehingga PTP tetap aktif), lalu jalankan pengujian. |
| `lattice multi-detect [--line LINE] [--json]` | Temukan kamera-kamera pada jalur sinkronisasi M8, dan master default: kamera RGB (model yang berakhiran `-FRGB`) jika terhubung dengan kabel, jika tidak, kamera pertama yang terhubung dengan kabel berdasarkan nomor seri. Kabel-kabel M8 bertemu di satu titik persimpangan, sehingga **kamera yang terhubung apa pun dapat menjadi master** — pencetakan `MASTER` hanyalah pengaturan default, bukan fakta pemasangan kabel. Setiap kamera secara bergiliran mengendalikan jalur tersebut dan kamera lainnya diperiksa untuk mendeteksi pulsa; kamera yang sudah terdeteksi dalam kelompok yang terhubung dengan kabel tidak akan diperiksa lagi, sehingga susunan yang sepenuhnya terhubung melalui kabel hanya memerlukan satu putaran. Tanpa adanya `--line`, sistem akan memeriksa `Line2`, lalu `Line3`. Mencetak `MASTER: SN:…`, satu `SLAVE: SN:…` per kamera terhubung lainnya, dan `(probed on <line>)`; `--json` menambahkan hasil lengkapnya (`master`, `slaves`, `no_cable` — kamera yang tidak terjangkau pulsa — `line`, dan per kamera `results`). Setelah selesai, setiap kamera akan kembali ke pengaturan pemicu dan garis sinkronisasi semula (node yang sama seperti di atas). |

#### Penyesuaian

| Subperintah | Tujuan |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — ditambah kenop detektor/penyandingan `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, parameter RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, kombinasi multi-frame `[--averaging mean\|median\|inlier_weighted]`, batasan geometris `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, batasan spasial `[--roi X0,Y0,X1,Y1] [--mask PATH]`, danpenggantian per-slave `[--per-cam-override SN:KEY=VALUE]` (dapat diulang) | Hitung profil penyelarasan dari kamera langsung. Nilai default: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (di dalam folder tempat Anda menjalankannya), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (`project align calibrate` memiliki pengaturan default tersendiri; lihat [tabelnya](#project-align-calibrate-options).) `--prefilter` secara default menggunakan `gradient` (peta tepi; sesuai dengan GUI/penyelarasan array — tepi tetap ada di seluruh pita spektral). `--matcher flann` memberikan hasil yang baik di atas ~5000 fitur; `--averaging median` tahan terhadap satu pengambilan gambar yang buruk, `inlier_weighted` memberi bobot berdasarkan jumlah kecocokan; `--lock-scale` memproyeksikan ke rotasi terdekat (tanpa skala), `--lock-axis` men-nol-kan satu komponen translasi; `--mask` berlaku untuk setiap kamera (gunakan `--per-cam-override` untuk pengaturan per kamera, misalnya `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` menolak menyimpan kalibrasi yang RMS reproyeksinya melebihi batas. `--method manual` mengambil titik-titiknya dari `--points PATH`: sebuah objek JSON `{"<serial>": [[x, y], ...], ...}` yang memberikan koordinat piksel setiap kamera untuk fitur yang sama, termasuk kamera referensi (setidaknya 2 per kamera); setiap kalibrasi `--frames` diselesaikan berdasarkan titik-titik tersebut. Ketika sebuah array terhubung berisi kamera-kamera, kalibrasi menggunakan bingkai langsung dari array tersebut, dan profilnya juga menjadi penyelarasan array untuk tangkapannya. Jika tidak, sistem membuka [array sementara](#multi-camera-transient-sessions): kamera yang tidak mengirimkan bingkai apa pun dalam salah satu `--frames` akan gagal dalam kalibrasi (keluar dari `1`, dengan menyebutkan namanya), dan tidak ada profil yang disimpan. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Mengambil satu bingkai multi-band yang selaras. `--bit-depth` secara default menyesuaikan dengan kamera; `--no-crop` mempertahankan bingkai penuh (diisi dengan warna hitam); `--interpolation` (default `linear`) dan `--border-mode`/`--border-value` (default `constant`/0) mengontrol distorsi CPU — jalur GPU tetap bilinear terlepas dari pengaturan. Membuka [array sementara](#multi-camera-transient-sessions): kamera yang tidak mengirimkan frame apa pun akan gagal dalam proses perekaman (keluar dari `1`, dengan menyebut namanya), dan tidak ada data yang ditulis. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Frame multi-band yang diselaraskan dengan aliran (pengaturan warp yang sama seperti `align-apply`), `--fps` default `3`. Membuka [array sementara](#multi-camera-transient-sessions): kelompok frame yang tidak dikirimkan oleh kamera tidak disimpan, dan tiga frame berturut-turut akan mengakhiri aliran. Jika ada kelompok yang dilewati, proses akan berakhir dengan menampilkan pesan `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` dan `Error: …`, dan keluar dengan pesan `1`. Aliran yang berakhir karena kesalahan lain juga mencetak pesan `Error: …` dan keluar dengan pesan `1`. |
| `lattice align-info --profile PATH [--json]` | Menampilkan detail profil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Mengubah urutan lapisan. |

#### Indeks / Matematika Vegetasi

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` memetakan simbol preset ke pita** (NDVI: `red`, `nir`), masing-masing satu `--channel SYMBOL=BAND`.

- **`--live`: hanya untuk simbol yang tidak dapat dijawab oleh tumpukan.** Seperti halnya `--input`, simbol preset yang Anda lewatkan akan mengambil satu pita dari tumpukan (pita kamera yang diaktifkan dalam profil) yang namanya dimulai dengan simbol tersebut. Hal ini diperiksa sebelum kamera mana pun dibuka: simbol yang tidak dijawab oleh satu pun pita, atau beberapa pita, akan ditolak, kemudian dengan menyebutkan nama-nama pita tersebut, keluar dari `1`. Sebuah `--formula` kustom memerlukan `--channel` untuk setiap simbol.
- **Produk `--live` menunjukkan milik siapa produk tersebut.** `index.tif` dan `index_LUT.png` memuat informasi kamera dalam tumpukan (Merek, Model, nomor seri), waktu pengambilan gambarnya (waktu pemaparan kamera referensi, UTC) dan lensa, serta keduanya menamai indeks sebagai satu-satunya band mereka (`XMP-Camera:BandName` `NDVI`; nama PNG berwarna menamai indeks, bukan kanal warnanya), seperti yang dilakukan oleh produk `--input`. Sebuah indeks dari beberapa pita kamera justru menamai masing-masing pita tersebut, dan `Model`-nya adalah `LATTICE` — lihat [File yang dibuat dari beberapa kamera](#files-made-from-several-cameras).
- **`--input`: hanya untuk simbol yang tidak dapat diidentifikasi oleh berkas.** Simbol yang Anda abaikan akan mengambil satu band dari berkas yang namanya dimulai dengan simbol tersebut: `red` akan mengambil `Red_660` atau `Red_660_RGN`, sedangkan `nir` akan mengambil `NIR_850`. Nama-nama tersebut berasal dari file itu sendiri: daftar band dari stack itu sendiri, atau, pada tangkapan radian/reflektansi Chloros atau produk `process`, tag `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`-nya. Jadi, `lattice index --input <reflectance capture> --preset NDVI` tidak memerlukan `--channel`.
- **Dua band yang merujuk pada satu simbol tidak akan dibedakan** (tumpukan 9 kamera memiliki dua band NIR): beri nama salah satunya. Simbol yang tidak sesuai dengan apa pun akan ditolak dengan `channel_map missing entries`, keluar dari `1`.**Nama pita dalam tumpukan** mencakup filter kamera: pita kamera M3C adalah `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` pada RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` pada OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` pada RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` pada NGB — dan kamera M3Mpita tunggalnya adalah `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` mencantumkan pita-pita dari suatu profil.

Set bendera lengkap: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (dapat diulang), `--capture-level raw|debayered|radiance|reflectance|unknown` (menggantikan tingkat penangkapan yang tercatat dalam file TIFF sumber; default: dibaca dari metadata TIFF, dengan menggunakan nilai cadangan dari file Chloros `Chloros:ProcessingLevel`, sehingga penangkapan reflektansi melaporkan `reflectance`), `--output PATH` / `-o` (offline: sebuah jalur; dengan ekstensi gambar, nama file diambil dari bagian awal namanya — `-o ndvi_FRGN.png` menulis `ndvi_FRGN.tif` dan `ndvi_FRGN_LUT.png` — dan tanpa ekstensi, itu menjadi awalan, `-o ndvi` -> `ndvi.tif`, `ndvi_LUT.png`; default di samping input, seperti `<input name>_<preset>`; dengan `--live` dan tanpa `-o`, sebuah folder `output/` di bawah folder tempat Anda menjalankan perintah), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Dengan `--live`, kenop distorsi penyelarasan juga berlaku: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (target eksposur otomatis kamera, lihat aturan transien), `--serials SN1,SN2,…`. `--live` membuka [array transien](#multi-camera-transient-sessions): kamera yang tidak mengirimkan bingkai apa pun akan gagal dalam proses tersebut (keluar dari `1`, dengan menyebut namanya), dan tidak ada tumpukan atau indeks yang ditulis. Ia membuka `--serials` (yang pertama memicu pemicu), jika tidak, kamera-kamera dalam profil yang entri `layer_order`-nya diaktifkan, dengan referensinya terlebih dahulu — referensi tersebut bahkan jika band-nya dinonaktifkan, karena yang lain disejajarkan dengannya kamera tersebut dan kamera tersebut memicu pemicu. Kamera yang pita-pitanya dinonaktifkan tidak pernah dikalibrasi, sehingga kamera RGB yang dinonaktifkan dalam profil tidak lagi gagal dalam indeks reflektansi (`no-absolute-radiance-scale`). Profil yang tidak menyebutkan nama kamera apa pun akan membuka setiap kamera di jaringan, seperti sebelumnya.

Setiap formulir berjalan di backend, sehingga backend harus sedang berjalan: jalankan dari antarmuka pengguna Chloros, atau dengan perintah yang ditampilkan oleh CLI saat tidak dapat menjangkaunya. Dalam mode offline, backend membaca `--input` dan menulis file TIFF tipe float serta PNG berwarna secara otomatis, sehingga `--input` atau `-o` akan merujuk ke folder tempat Anda menjalankan perintah tersebut (lihat [Konvensi](#conventions)). Kedua produk offline tersebut menyertakan catatan kamera dari input, seperti yang dilakukan ekspor indeks `process`: Merek / Model / nomor seri, GPS, tag lensa, waktu pengambilan gambar (waktu dari input itu sendiri; jika EXIF-nya menyatakan tidak ada, maka menggunakan waktu eksposur yang terekam atau cap waktu pada nama file, dalam UTC) dan identitas kalibrasi MAPIR — bukan kalibrasi raw-DN, yang tidak berlaku untuk indeks — ditambah `Chloros:ProcessingLevel=index` dan ekspresi indeks; nilai float TIFF menamai satu band, yaitu preset. `--list-presets` dan `--list-gradients` mencetak tabel preset dan gradien backend. Rumus yang ditolak oleh backend, atau pita `--channel` yang tidak dimiliki oleh TIFF, akan keluar dari `1` dengan alasan dari backend.

> **Simbol-simbol `--channel` peka terhadap huruf besar-kecil.** Sisi simbol harus persis sama dengan nama kanal preset (preset menggunakan huruf kecil, misalnya NDVI = `red`,`nir` — periksa `--list-presets`), dan sisi band harus sesuai dengan nama band dalam tumpukan yang diselaraskan (atau berupa indeks band berbasis 0 dalam mode offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` berfungsi; `--channel RED=660` gagal dengan kesalahan `channel_map missing entries` .

#### `lattice index` — topeng tumpukan multiband

Pada `--capture-level radiance` atau `reflectance`, proses pemisahan 3×3 pada setiap kamera dapat membatasi nilai suatu band menjadi 0 (lihat [Masker pembatas pemisahan](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Tumpukan multiband berisi pita dari beberapa kamera, dan setiap kamera membatasi pita miliknya sendiri, sehingga `--live --save-multiband` menulis **satu masker per kamera sumber** di samping tumpukan:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Setiap masker berupa `uint8` satu-band pada kisi piksel **tumpukan** (melalui proses penyelarasan, distorsi, dan pemotongan yang sama), dengan bit-bit kamera tersebut: bit 0 = band saluran R-nya, bit 1 = G, bit 2 = B. Sebuah band tumpukan yang diambil dari saluran *c* suatu kamera dibatasi di mana bit *c* kamera tersebut (R = 0, G = 1, B = 2) ditetapkan.
- Provenance yang tertanam pada setiap band menyebutkan nama masker kameranya (`unmix_clamp_mask`, relatif terhadap folder tumpukan, atau `none`) serta bit yang akan dibaca (`unmix_clamp_mask_bit`).
- Kamera yang proses unmix-nya tidak membatasi nilai apa pun tidak menghasilkan berkas; kamera mono atau RGB yang tidak menjalankan unmix juga tidak menghasilkan berkas. CLI menampilkan setiap masker di bawah tumpukan (`clamp mask SN…: …`); respons mencantumkannya sebagai `clamp_masks`.
- File TIFF/PNG indeks dari proses yang sama dihitung dari tumpukan yang sama, pada grid yang sama, sehingga masker-masker tersebut juga menggambarkannya. `--capture-level raw` tidak menjalankan proses unmix dan tidak menulis masker apa pun.
- Tumpukan dan indeks berada langsung di dalam `--output`, sehingga masker-masker tersebut berada di dalam `clamp_masks/` yang merupakan bagian dari `--output` — tidak pernah berada di atas folder yang Anda beri nama. Folder tersebut mencampurkan jenis-jenis hasil (stack, indeks, render berwarna): pilih file-file di dalamnya, jangan serahkan seluruh folder ke perangkat lunak fotogrametri.

#### Koneksi Persisten (Smart-Prep, Alur Kerja Setara GUI)

Perintah-perintah ini menjaga kamera tetap terbuka di kolam backend di seluruh pemanggilan CLI.

| Subperintah | Tujuan |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Menambahkan satu kamera ke pool (kamera tunggal, bukan array). [Opsi umum](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` dan `--packet-size` (`standard` = 1500, `jumbo` = 9000, atau byte; probe `auto`) diterapkan saat koneksi (sakelar `--exposure` atau `--gain` yang mematikan mode otomatis). Perintah ini kembali setelah streaming kamera dimulai (menunggu sekitar 15 detik untuk itu, lalu 10 detik untuk buffer lengkap pertamanya) dan menampilkan `stream: streaming`, atau `stream: no_frames -- connected, no frames yet: …` ketika streaming tidak mengirimkan apa pun — untuk kamera yang menggunakan paket jumbo, disertai saran untuk terhubung kembali dengan kode `--packet-size standard`. Saat satu kamera sedang terhubung (koneksi pertama mengunduh kalibrasinya, sekitar 70 detik), perintah lain yang membutuhkan kamera tersebut akan menunggunya paling lama 5 detik, lalu gagal dengan kode HTTP 503 `CAMERA_POOL_BUSY` yang menyebutkan nama kamera yang sedang terhubung; jalankan kembali sebentar lagi. `lattice status` dan daftar kamera tidak pernah menunggu: kamera yang sedang dalam proses koneksi ditampilkan sebagai sedang terhubung, beserta tahapannya. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Lepaskan. Kamera dikembalikan ke kondisi saat ditemukan saat koneksi (pengaturan pemicu, PTP, pengaturan); `--keep-settings` menyimpan pengaturannya ke memori internalnya sendiri, `--clear-settings` mereset memori tersebut ke pengaturan pabrik (lihat [Memori kamera](#camera-memory----keep-settings----clear-settings)). Kamera yang merupakan anggota array akan melepaskan seluruh array. |
| `lattice cam-list` | Daftar kamera dalam pool. |
| **`lattice array-connect`**|**Menghubungkan array sinkronisasi persisten (titik masuk yang direkomendasikan).** Menjalankan alur persiapan cerdas (smart-prep) GUI secara penuh. Memerlukan waktu sekitar satu menit untuk 9 kamera; segera setelah kamera dinyalakan, proses ini bisa memakan waktu beberapa saat. Selama menunggu, sistem mencetak setiap langkah yang dicapai oleh backend, seperti `    <s> s: <step>` (misalnya `    41 s: open SN:214000535 (4 of 9)`), dan log backend memiliki satu baris waktu per langkah (`[array-connect <id>] <step>: <s> s …`) serta baris terakhir yang mencatat langkah-langkah paling lambat. Sistem menunggu hingga 600 detik. Jika koneksi belum terjalin hingga saat itu, backend akan membatalkan prosesnya: setiap kamera yang telah dibuka dikembalikan ke kondisi semula dan ditutup, perintah tersebut mencetak `Error:` dengan langkah yang telah dicapai dan keluar dengan `1`, serta tidak ada array yang terhubung (504 HTTP `CLIENT_DEADLINE_PASSED`). Ctrl+C membatalkan koneksi dengan cara yang sama (`Interrupted: …`); jika array tersebut sudah terhubung, baris tersebut akan menampilkannya dan menyebutkan `array-disconnect` yang melepaskannya. Perintah yang dihentikan saat sedang menunggu (`kill -9`) akan membuat koneksi tersebut dibatalkan jika backend tidak menerima respons darinya selama 30 detik. Sebuah array yang aktif tetapi belum memiliki grup frame yang disinkronkan tetap dilaporkan terhubung, dengan alirannya sebagai `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Melepaskan sebuah array. Setiap kamera dibiarkan berjalan bebas (`TriggerMode` `Off`), dengan semua pengaturan lainnya tetap seperti saat koneksi terjalin — pengaturan pemicu dan garis sinkronisasi (pengaturan master`TriggerSource` dikembalikan ke kondisi semula, garis sinkronisasinya kembali ke input), PTP-nya, dan pengaturan kamera yang diubah oleh sesi — sehingga perintah untuk satu kamera pada anggota jaringan langsung berfungsi. `--keep-settings` menyimpan setiap pengaturan kamerake dalam memori kamera itu sendiri sesuai dengan kondisi saat sesi berakhir; `--clear-settings` mengatur ulang memori tersebut ke pengaturan pabrik. Lihat [Memori kamera](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Menampilkan daftar array yang terhubung. |
| `lattice array-status [--array-id ID]` | fps langsung, PTP, kesalahan terakhir. `ptp_enabled` hanya sama dengan `True` jika setiap anggota membaca PTP dalam keadaan aktif; `ptp_off:` menyebutkan anggota yang tidak melakukannya. Baris `state:` membaca `restarting (applying settings)` selama pengaturan dimulai ulang, `recovering (…)` saat anggota yang kehilangan daya dibuka kembali (lihat catatan di bawah), `stalled (…)` saat loop pengambilan berjalan tetapi tidak menerbitkan grup frame, dan `stopped (…)` setelah loop pengambilan berhenti (sambungkan kembali array). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Satu penangkapan yang disinkronkan dari array langsung — Tunggal / Berkelanjutan / Interval / Tercepat. **Secara default menggunakan `all`**(satu file per jenis ekspor yang berlaku per kamera),**diselaraskan** sesuai dengan penangkapan aplikasi (`--no-aligned` untuk file asli). Kamera yang dilewati (mis. RGB) dikecualikan dari radian/reflektansi) dilaporkan dengan `Skipped: SN:<serial> (<reason>)`; pembacaan DAQ yang digunakan untuk perekaman disimpan bersamanya, perekaman mentah (raw-only) disertakan, dan dilaporkan dengan `DAQ: <path>` ([DAQ yang membaca perekaman menyimpan](#the-daq-reading-a-capture-saves)). Lihat [Mode Pengambilan Gambar, Perekam & Pemrosesan Ulang Offline](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Rekam tampilan indeks gabungan langsung ke video/GIF (kelas pemantauan; tanpa tampilan GUI yang terbuka, backend merender NDVI dari array, atau `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Burst raw-Bayer dengan fps tinggi (tingkat analisis; diproses ulang secara offline). Seperti `array-record` dan `array-capture --continuous`, proses ini melakukan check-in ke backend pada setiap polling: dihentikan tanpa dihentikan (`kill -9`), backend menghentikannya 30 detik setelah pengecekan terakhir (`stop_reason` `client_gone`; manifes burst diselesaikan dan pembuatan indeks gabungannya dimulai, seperti saat dihentikan) dan array tetap terhubung. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Memproses ulang burst mentah yang disimpan menjadi video yang telah dikalibrasi. |

> **Kamera yang kehilangan daya atau koneksinya akan kembali secara otomatis.** Ketika salah satu anggota berhenti merespons (siklus daya saklar akan me-reboot setiap kamera yang disuplai dayanya), array akan menunggunya, membukanya kembali saat kamera tersebut merespons lagi, dan mengembalikan pengaturan yang ditetapkan oleh `array-connect`: PTP, sinyal pemicu dan sinkronisasi, kecepatan jaringan, format piksel dan ukuran bingkai, serta eksposur otomatis. Kamera tersebut kemudian mulai melakukan streaming kembali tanpa pengaturan `array-connect` baru. Sementara itu, `array-status` menampilkan `state: recovering (...)` dan `last_error` menunjukkan kamera mana yang ditunggu; setelah itu, sistem mencetak `re-opened:`. Kamera yang telah di-reboot kembali dengan PTP dinonaktifkan, sehingga hingga dibuka kembali, `ptp_enabled` menjadi `False` dan `ptp_off:` menamainya.

> **Kamera yang ditahan tidak akan pernah dibuka untuk kedua kalinya.** Pembukaan kedua akan menghentikan aliran pemegang dan menulis ulang pengaturannya, serta dapat menyebabkan backend crash. Jadi, pada kamera yang ditahan oleh pool (GUI atau `cam-connect`), anggota array, atau kamera yang ditahan oleh tugas latar belakang (`lattice continuous` yang sedang berjalan, `multi-stream`, atau `align-stream`—baik miliknya sendiri maupun milik klien lain), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, satutembakan tunggal `white-balance` dan `presets --apply` ditolak dengan kode HTTP 409 `CAMERA_HELD`, menyebutkan pemegang dan perintah pelepasan (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>`, atau untuk suatu pekerjaan, ID-nya dan siapa yang memulainya), dan keluar `1`. Begitu pula dengan kamera yang sedang digunakan oleh perintah lain saat ini (seperti `multi-test` atau `multi-capture` yang membuka kameranya, satu kali lagi), untuk setiap perintah di sini: pesan kesalahan menyarankan untuk menunggu hingga perintah tersebut selesai. `lattice power` tanpa `--sleep` membaca melalui handle milik pemegang jika ada (pool, array, kamera terbuka dari suatu pekerjaan), dan ditolak jika kamera tersebut sedang digunakan oleh perintah lain. Setiap balasan `CAMERA_HELD` membawa kode tersebut baik di `code` maupun `error_code`. Untuk perintah yang membuka setiap kamera, lihat [Multi-Kamera (Sesi Sementara)](#multi-camera-transient-sessions).

##### Opsi `array-connect`

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--serials SN1,SN2,…` | temukan semua kamera LATTICE secara otomatis (diperlukan ≥2) | Kamera serial pertama adalah MASTER. Jika diabaikan, proses penemuan akan menyaring model LATTICE (M3C / M3M) dan menghubungkan semuanya. |
| `--line {Line0,Line2,Line3}` | `Line2` | Jalur sinkronisasi GPIO. |
| `--target-fps F` | otomatis | Kecepatan tembakan pemicu MASTER. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | otomatis | Mengganti pengatur tingkatan. |
| `--wire-ceiling-mbps MB_PER_S` | terdeteksi otomatis | **Anggaran bandwidth berkelanjutan host, dalam MB/s — angka yang menjadi acuan alokasi seluruh array.** Kurangi nilainya saat array melaporkan frame yang rusak akibat GVSP: nilai otomatis ini diperoleh dari kecepatan tautan yang diiklankan oleh NIC, yang cenderung melebih-lebihkan adaptor USB, jalur PCIe yang tipis, dan jaringan bersama yang sibuk. Nilai ini disimpan dalam blok penangkapan array proyek, sehingga pembukaan ulang / koneksi ulang melalui CLI / SDK akan mengembalikannya. Lihat [Kesehatan array](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | tidak ada (sensor penuh) | Pengelompokan perangkat keras (selalu Rata-rata). |
| `--width N` / `--height N` / `--pixel-format FMT` | sensor penuh; format per kamera | Ukuran bingkai atau format piksel yang eksplisit untuk setiap kamera, bukan sensor penuh dan pilihan format per kamera (Mono12 / BayerRG12). |
| `--no-recommend` | nonaktif | Lewati langkah analisis jaringan (setiap kamera kemudian mempertahankan ukuran aslinya, bukan disetel ke sensor penuh). |
| `--no-ptp` | nonaktif | Nonaktifkan PTP (cap waktu antar-kamera kemudian **tidak** dapat dibandingkan). |
| `--packed {auto,on,off}` | `auto` | Alirkan bingkai 12-bit dalam bentuk terkompresi dan dekompresi di komputer ini — lebih banyak bingkai per detik pada resolusi penuh sensor, dengan data yang identik. `auto` hanya mengompresi jika jika diperlukan dan komputer ini mampu melakukannya; `off` tidak pernah mengemas. Lihat [Transportasi 12-bit yang dikemas](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Ukuran paket GVSP untuk setiap kamera dalam susunan. `auto` memungkinkan setiap kamera melakukan pemeriksaan jaringan; `jumbo` = 9000, `standard` = 1500, atau ukuran dalam byte akan membatasinya. Sebelum versi 1.3.0, `array-connect` menerima bendera ini tetapi mengabaikannya. |

> **Sensor penuh secara default.** Tanpa `--width` / `--height` / `--binning` /
> `--pixel-format`, `array-connect`, setiap kamera diatur ke sensor penuh — tanpa
> binning, tanpa pemotongan ROI — dan membiarkan laju bingkai turun sesuai dengan kemampuan komputer dan
> jaringan ini, alih-alih memperkecil ukuran bingkai untuk mempertahankan laju tersebut. Baris
> analisis jaringan menunjukkan laju yang diproyeksikan, dan baris berikutnya mencantumkan
> opsi yang lebih cepat (binning atau ROI yang lebih kecil) beserta laju yang diproyeksikan; lewati
> `--binning` atau `--width`/`--height` untuk memilih salah satunya. Connect tidak pernah melakukan binning atau pemotongan
> secara otomatis.

##### Transportasi 12-bit terkemas (`--packed`)

Pada konfigurasi sensor penuh, laju bingkai suatu array ditentukan oleh seberapa cepat setiap kamera dapat mengirimkan
bingkainya. Format 12-bit (`BayerRG12`, `Mono12`) biasanya dikirim sebagai kata 16-bit;
transmisi **terkompresi** mengirimkan 12 bit yang sama dalam 1,5 byte per piksel dan
membukanya di komputer ini, sehingga setiap bingkai membutuhkan waktu 25% lebih sedikit untuk dikirim —
sekitar **33% lebih banyak bingkai per detik pada sensor penuh** (terukur: 10,7 → 14,0 fps
pada susunan 9 kamera). **Data yang disimpan identik**: nilai yang sama,
file 16-bit yang sama, metadata yang sama.

- `auto` (default) hanya melakukan pengemasan jika bermanfaat **dan** komputer ini mampu
  melakukan pembongkaran. Saat terhubung, backend mengukur kecepatan pengepakannya sendiri, jumlah CPU,
  beban CPU saat ini, dan anggaran jaringan, sehingga komputer desktop dengan inti yang berlebih akan mengompresi
  pada pengaturan sensor penuh, sedangkan komputer yang sibuk atau berdaya rendah akan melakukan streaming tanpa kompresi. ROI kecil dan bingkai yang digabungkan menghasilkan
  sedikit.
- `off` tidak pernah mengompres — gunakan ini untuk menonaktifkan pengompresan saat mendiagnosis, atau pada
  komputer yang harus mengalokasikan seluruh CPU-nya untuk pekerjaan lain.
- `on` selalu mengompresi format 12-bit.

Output connect menampilkan keputusan dan alasannya (`dikompresi 12-bit: aktif (otomatis:
diperkirakan 11,0 -> 14,4 fps, …)`); `/api/camera/array/status` melaporkan
`packed_transport` dan `packed_decision`. Untuk menetapkan nilai default di seluruh mesin
daripada harus memberikan bendera tersebut setiap kali, atur `CHLOROS_ARRAY_PACKED12=off` (atau
`on`) di lingkungan backend — pengaturan eksplisit `--packed on|off` tetap diutamakan.
Format 8-bit dan 16-bit tidak pernah dikemas.

### Satu catatan proyek, semua mode

Folder proyek menyimpan pengaturan setiap perangkat: `cameras.json` — pengaturan masing-masing kamera (`cameras[].settings`, untuk satu kamera dan untuk setiap anggota array) serta pengaturan yang berlaku untuk seluruh array (`arrays[].capture`: format, ROI, binning, laju pemicu, anggaran kabel) — dan `sensors.json`, setiap sensor cahaya (waktu integrasi, rata-rata bingkai, batas). Aplikasi, CLI, SDK, dan server MCP semuanya membaca dan menulis berkas yang sama, dengan cara yang sama:

- **Perangkat yang terhubung ke suatu proyek disimpan di dalamnya dan mencatat setiap perubahan ke dalamnya.** "Di bawah sebuah proyek" berarti dengan proyek yang terbuka di backend (aplikasi), atau — dari SDK — melalui `ChlorosProject`, yang menentukan foldernya sendiri. Kamera, array, atau sensor cahaya yang belum disimpan dalam proyek akan ditambahkan ke berkas-berkasnya saat terhubung (`lattice cam-connect`, `array-connect`, `daq pool-connect`, aplikasi, SDK, `chloros-mcp`), beserta pengaturannya. Perubahan dari mode apa pun (`lattice exposure`, `gain`, `configure --set`, `resolution`, pengaturan arraypengaturan dan laju pemicu, batas sensor cahaya, serta kontrol aplikasi) akan disimpan ke dalam berkas proyek tersebut beberapa saat kemudian. Pengaturan eksposur, gain, atau target AE yang ditetapkan pada kamera akan disimpan sebagai pengaturan yang disimpan, sehingga saat dibuka kembali, pengaturan tersebut akan diterapkan kembali.
- **Membuka sebuah proyek akan mengatur perangkat-perangkatnya berdasarkan proyek tersebut**, di setiap mode: kamera, array, atau sensor cahaya yang terhubung tanpa pengaturan sendiri akan menggunakan pengaturan yang disimpan dalam proyek tersebut (`lattice cam-connect` / `array-connect` saat proyek terbuka, `chloros-cli project connect`, `open_project().connect_all()`, saat aplikasi dibuka kembali). Sebuah array yang dihubungkan dengan ID baru akan memperbarui array yang disimpan proyek dengan kamera yang sama; array dengan kamera lain akan ditambahkan di sampingnya.
- **Fungsi penyimpanan aplikasi sendiri tidak pernah menimpa pengaturan langsung perangkat** dengan salinan lamanya, dan mempertahankan pengaturan khusus per-anggota yang disimpan oleh pemanggil CLI atau SDK.
- **Format piksel yang Anda tetapkan adalah yang disimpan oleh proyek.** Format yang ditetapkan di Pengaturan Array, melalui `--pixel-format` atau `connect_array(pixel_format=)`—termasuk 8-bit—akan disimpan dan diterapkan kembali persis seperti aslinya. Jika tidak ada yang ditetapkan, setiap kamera akan terhubung pada pengaturan default 12-bit (`BayerRG12`, `Mono12` pada sensor monokrom). Nama format menunjukkan kedalaman bitnya; keluarga format tersebut sesuai dengan sensornya, sehingga kamera warna tidak pernah disetel ke format Mono atau kamera mono ke format Bayer, dan baris `[PixelFormat <serial>] ... written` mencatat hal ini saat format dipetakan. Ekspor hasil pengambilan gambar 8-bit, ekspornya adalah 8-bit, termasuk reflektansi: diskalakan ulang dan diberi cap dengan `PixelScale`.
- Perangkat yang terhubung tanpa proyek tidak merekam apa pun.
- **Perekaman mengikuti proyek yang dibuat setelahnya; saat ditutup, perekaman tetap terikat pada proyeknya.**Saat sebuah proyek dibuat — baik melalui SDK `create_project()`, `chloros-mcp`, `create_project`, atau fitur**Proyek Baru**— atau diduplikasi (opsi**Duplicate**pada aplikasi, yang membuka salinannya) sementara perekaman masih menulis ke proyek yang terbuka sebelumnya, perekaman akan terus berlanjut, dan apa yang ditulisnya akan masuk ke proyek baru, di tempat yang sama di dalamnya (di bawah). Membuka proyek yang sudah ada — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, fitur**Buka Proyek** pada aplikasi — akan membuat perekaman tetap berjalan di proyek yang menjadi tujuannya: perekaman akan selesai di sana, dan berkas-berkasnya akan ditambahkan ke daftar berkas proyek tersebut (aplikasi akan menghentikan setiap perekaman sebelum membuka proyek).
  - `array-record` dan `array-burst`: video, atau folder burst yang berisi setiap bingkai, dipindahkan secara utuh ke sana saat perekaman berhenti (berkas `--duration`, Ctrl+C). `Burst dir` yang ditampilkan adalah yang baru, dan `--build` serta `array-build-video` beroperasi berdasarkan file tersebut. Jika pemindahan ditolak oleh drive, rekaman tetap berada di lokasi aslinya;log akhir menunjukkan alasannya, begitu pula respons HTTP saat berhenti (`relocation_error`). Frame mentah dari burst yang dihentikan dan `.daq` ditambahkan ke daftar file proyek tempatnya berakhir; sebuah video (sebuah `array-record`, video yang dihasilkan dari burst) tidak ditambahkan ke daftar file proyek.
  - `array-capture --continuous`: setiap rekaman setelah perubahan disimpan ke dalam proyek baru; rekaman yang disimpan sebelumnya tetap berada di tempatnya.
  - `daq pool-record` dan `project sensor log` (serta perekaman sensor cahaya bawaan aplikasi): `.daq` dipindahkan ke folder proyek baru`light_sensor` saat perekaman berhenti, dan bergabung dengan daftar file proyek tersebut.

  Perekaman ke dalam folder di luar proyek yang terbuka (seperti `-o` di tempat lain) tetap berada di tempatnya, sedangkan `lattice continuous` dan `multi-stream` tetap berada di folder yang telah ditentukan.
- **Di dalam aplikasi, hasil perekaman setelah proyek baru dibuat akan dimasukkan ke dalamnya.** Membuat atau menduplikasi proyek akan mempertahankan semua proses perekaman yang sedang berjalan, dan setiap perekaman yang dilakukan setelahnya (Capture All, Start Capturing) akan disimpan ke dalam proyek baru tersebut dan ditambahkan ke daftar file-nya. Membuka proyek yang sudah ada di dalam aplikasi akan menghentikan semuanya terlebih dahulu.
- **Sebuah proyek tidak akan dipindahkan saat sedang merekam.**Memindahkan proyek yang terbuka ke direktori kerja lain (melalui**Pengaturan → Direktori Kerja** di aplikasi; tidak ada perintah CLI yang memindahkan proyek, melainkan klien HTTP yang mengirim permintaan ke `/api/move-project`) akan ditolak dengan kode status HTTP 409 `PROJECT_RECORDING` selama perekaman array atau burst, perekaman array berkelanjutan, pekerjaan `lattice continuous` / `multi-stream`, atau perekaman sensor cahaya sedang menulis ke dalamnya: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` Aplikasi akan menolak dengan cara yang sama, sebelum meminta folder, saat fitur Capture All, Start Capturing, atau perekaman sensor cahaya miliknya sendiri sedang berjalan.

Hal ini terpisah dari pengaturan kamera saat terputus (lihat [Memori kamera](#camera-memory----keep-settings----clear-settings)): sebuah proyek mengatur kameranya sendiri saat dibuka.

### Memori kamera — `--keep-settings` / `--clear-settings`

Saat sebuah array terhubung, kamera-kameranya akan menggunakan pengaturan array tersebut, dan Anda kemudian dapat mengubahnya. Saat Chloros melepaskan kamera — `array-disconnect`, akhir dari perintah sementara (`multi-capture` tanpa array, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), penghentian `lattice continuous`, akhir dari `lattice capture` kamera yang tidak dimilikinya — ia mengembalikan masing-masing ke keadaan semula sebelum terhubung: kabel pemicu dan jalur sinkronisasi, serta pengaturan PTP dan kamera (daftar lengkapnya ada di [Multi-Kamera (Sesi Sementara)](#multi-camera-transient-sessions)).

Untuk menggunakan kamera di luar proyek, dua bendera pada salah satu perintah tersebut akan menulis ke **memori kamera itu sendiri** — pengaturan saat kamera dinyalakan, di perangkat lunak apa pun:

- **`--keep-settings`** menyimpan pengaturan setiap kamera sebagaimana adanya saat sesi berakhir ke dalam memorinya (UserSet2, dijadikan set boot). Kamera-kamera tersebut tetap dibiarkan berjalan bebas (`TriggerMode` `Off`), dengan pengkabelan jalur sinkronisasi tetap seperti semula. Kamera yang tidak memiliki UserSet2 akan menyimpan pengaturannya ke UserSet1; komputer ini mengingatnya, sehingga array Chloros dapat memperbaiki diri — yang mereset UserSet1 yang telah menyimpang dari garis dasar Chloros — membiarkannya apa adanya.
- **`--clear-settings`** mereset memori setiap kamera ke pengaturan pabrik: garis dasar Chloros, yang diintegrasikan ke dalam proses pemulihan otomatis array-nya — pengaturan default pabrik dengan `TriggerMode` `Off`, `TriggerSource` `Line2`, pencahayaan otomatis, keseimbangan putih yang telah dikalibrasi pada kamera RGB tetap — di UserSet1, dijadikan set boot.
- Keduanya tidak dapat digabungkan. Setiap kamera mendapat satu baris: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` atau `SN:…: camera memory reset to the factory settings.`; balasannya, `camera_memory`, sama untuk setiap kamera.

Aplikasi ini tidak menyediakan bendera-bendera tersebut.

**Setiap jalur keluar mengembalikan kamera ke kondisi semula.**Proses pengembalian ke kondisi semula yang sama seperti saat ditemukan dijalankan pada setiap jalur keluar, tidak hanya perintah di atas: `cam-disconnect` (dan kamera**X**dari aplikasi);**X** dari aplikasi pada sebuah array atau pada baris anggota array mana pun (sebuah array tidak dapat dijalankan tanpa anggota, sehingga seluruh array dilepaskan); menutup aplikasi (aplikasi memberikan waktu hingga 20 detik kepada backend untuk mengembalikan setiap kamera, array, dan menjalankan `lattice continuous` kembali sebelum menghentikannya; perintah yang masih berjalan saat itu — perintah sementara yang membuka kameranya atau menunggu jam PTP-nya, perintah satu kali `lattice capture`, `power`, `white-balance`, atau `presets --apply` — akan berhenti pada titik aman berikutnya, mengembalikan kameranya terlebih dahulu, selama maksimal 8 detik, dan merespons HTTP 503 `SHUTTING_DOWN`; koneksi array yang masih berjalan — `lattice array-connect`, koneksi aplikasi — berhenti sebelum kamera berikutnya dan mengembalikan kamera yang telah dibuka, selama maksimal 10 detik, dan merespons HTTP 503 `SHUTTING_DOWN`; dan yang tiba selama proses penutupan tidak membuka apa pun: HTTP 503 `SHUTTING_DOWN`); serta perintah CLI yang dihentikan paksa atau terminalnya ditutup (SIGTERM / SIGHUP menghentikan tugasnya — dan mengembalikan kameranya — sebelum keluar `143` / `129`, seperti Ctrl+C). Sebuah `lattice continuous`, `multi-stream`, atau `align-stream` yang dihentikan secara langsung (SIGKILL), yang tidak dapat menghentikan apa pun, akan dihentikan tugasnya dan kameranya dikembalikan oleh backend 30 detik setelah check-in terakhirnya. Backend yang crash atau dimatikan paksa tidak dapat mengembalikan apa pun; siklus daya akan mengembalikan setiap kamera ke memorinya, dan koneksi Chloros berikutnya akan menghubungkan ulang kamera yang ditemukan dalam keadaan aktif.**Sebuah proyek mengatur kameranya sendiri.** Ketika sebuah array terhubung saat proyek sedang terbuka, setiap anggotanya mendapatkan pengaturan yang disimpan proyek untuknya — target eksposur, gain, atau AE yang disimpan padanya (disimpan sebagai milik Anda, sehingga eksposur otomatis tidak mengubahnya) serta mode keseimbangan putihnya (koreksi Chloros; keseimbangan putih bawaan kamera tetap dinonaktifkan) — apa pun yang tersimpan dalam memori kamera.

### Smart-AE / Smart-Capture

Array LATTICE menjalankan AE berkelanjutan di latar belakang segera setelah terhubung, tetapi adegan yang baru diarahkan membutuhkan waktu sejenak untuk konvergensi. `array-capture --smart` adalah **fitur kemudahan terintegrasi**: fitur ini menunggu hingga AE stabil di semua kamera dalam array, lalu memicu pengambilan gambar. Gunakan fitur ini saat Anda mengganti pemandangan di tengah sesi.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Kebijakan stabilisasi secara default bersifat konservatif: batas waktu 5 detik, jendela stabilitas 1,5 detik, toleransi penyebaran eksposur ±5%. Sesuaikan melalui SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) jika Anda memerlukan perilaku yang berbeda dari otomatisasi.

#### Bagaimana perilaku eksposur otomatis

- **Pencahayaan terlebih dahulu, penguatan terakhir.** Dengan penguatan pada mode Otomatis, penguatan tetap pada 0 dB hingga pencahayaan mencapai nilai terpanjang yang diizinkan.
- **Pengaturan eksposur manual akan membiarkan gain tetap pada mode Otomatis** hingga Anda mengubah gain atau mengaturnya ke mode manual. Saat kembali ke mode Otomatis, batas gain akan kembali seperti semula.
- **Sorotan kecil tidak akan menggelapkan adegan.** Sorotan spekular hingga 1% dari bingkai akan diabaikan, dan pada panel target kalibrasi yang diketahui, hingga 0,5% dari satu panel.
- **Batas atas eksposur DAQ hanya berlaku saat target kalibrasi terlihat.** Hal ini mencegah panel putih mengalami clipping; jika tidak ada panel yang terlihat, kamera akan menyesuaikan eksposur sesuai dengan adegan.
- **Cahaya konstan tetap stabil, dan perubahan cahaya akan stabil dalam hitungan detik.** Pada cahaya konstan, eksposur dan batasnya tetap stabil, dan setelah terjadi perubahan cahaya yang besar, tampilan akan stabil dalam beberapa detik. Kliping tidak lagi membatasi eksposur secara permanen, dan perubahan ROI akan menyesuaikan eksposur kembali.
- **Target kecerahan tetap dalam jangkauan.** Target ini tidak naik saat kamera berada pada batas eksposurnya dan penguatannya tidak dapat meningkat, sehingga cahaya yang kembali tidak akan menyebabkan flare pada bingkai.
- **Kamera yang eksposur otomatisnya berhenti merespons akan dipertahankan pada tingkat yang sesuai dengan cahaya di sekitarnya.** Di bawah sinar matahari yang cerah, kamera bisa turun ke eksposur terpendeknya dan tetap di sana dengan bingkai yang gelap, meski semua pengaturannya menunjukkan nilai normal. Chloros memantau bingkai, mempertahankan kamera pada tingkat yang disesuaikan dengan cahaya tersebut, dan melepaskannya saat cahaya berubah. Sensor cahaya yang gelap atau tertutup tidak mengganggu proses ini.

### Tingkat Ekspor Pengambilan Gambar (pengaturan default `all`)

Sejak rilis ini, `lattice capture`, `lattice multi-capture`, dan `lattice array-capture` **secara default menggunakan `--processing all`** — satu berkas yang disimpan per jenis ekspor yang berlaku untuk setiap kamera, sesuai dengan perilaku "Capture All" pada antarmuka pengguna (GUI). Tingkat-tingkatnya adalah:

| Tingkat | Keluaran | Berlaku untuk |
| --- | --- | --- |
| `raw` | Bayer saluran tunggal (kamera monokrom: pita tunggal) langsung dari sensor. | Semua kamera. |
| `debayered` | Warna 3-saluran, disimpan dalam urutan saluran **RGB**(berkas TIFF dengan interpretasi fotometrik RGB; PNG/JPEG juga dibuka sebagai RGB): offset gelap (DSNU) dari paket kalibrasi dikurangi dari data mentah, kemudian demosaik yang mendukung deteksi tepi OpenCV— tanpa LUT linearitas, flat field, atau unmix (kamera monokrom: skala abu-abu 1-saluran). Frame yang telah di-debayered dalam memori yang dikembalikan oleh Python SDK (`grab()`, `frame_stream()`) berformat**BGR**, urutan OpenCV; hanya berkas yang disimpan yang berformat RGB. | Semua kamera. |
| `radiance` | float32 W/m²/sr/nm melalui rantai radiometrik lengkap. | Hanya multispektral (M3C/M3M) — **dilewati untuk kamera dengan filter RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), siap untuk Pix4D. | Hanya multispektral, dan **hanya jika DAQ terikat + kamera telah dikalibrasi**; jika tidak, dilewati. |
| `display` | Rantai pratinjau GUI lengkap (CCM + WB + gamma sesuai profil kamera). Disimpan dalam folder `preview/`. `lattice capture` juga menerima nama `preview` untuknya; `array-capture` / `multi-capture` hanya menerima `display`. | Semua kamera. |

Berikan satu level saja untuk menyimpan hanya level tersebut (`--processing debayered`). Saat Anda meminta `all`, level yang tidak berlaku untuk kamera tertentu akan dilewati (dan dilaporkan), bukan dianggap error — kamera yang tidak terhubung atau belum dikalibrasi tetap mendapatkan `raw` / `debayered` / `preview`.

#### Pembacaan DAQ yang digunakan dalam penangkapan disimpan

Setiap kali DAQ diikat ke kamera atau array, atau dimasukkan ke dalam pool, pembacaan downwelling yang digunakan dalam penangkapan ditulis ke berkas **`.daq`** di subfolder `daq/` dari folder captures, beserta nilai `.match.json` (sejauh mana pembacaan tersebut dari eksposur) dan `.cameras.json` (milik siapa pembacaan tersebut), serta dilaporkan pada baris `DAQ:`. Hal ini berlaku untuk setiap level, termasuk tangkapan mentah atau yang telah di-debayering, sehingga setiap tangkapan dapat diproses ulang menjadi radiance, reflektansi, atau indeks di kemudian hari. Ini adalah file yang sama, dengan nama dan sidecar yang sama, baik tangkapan tersebut berasal dari GUI, CLI, atau SDK. Tanpa pembacaan DAQ sama sekali, tidak ada `.daq`, dan `multi-capture` tanpa array yang terhubung (pengganti sementara) tidak akan menyimpan apa pun.

| Bendera | Efek |
| --- | --- |
| `--no-daq` | Jangan menyimpan pembacaan pada penangkapan yang levelnya tidak menggunakannya (`raw`, `debayered`). Tingkat `radiance`, `reflectance`, atau `display` tetap menuliskannya, karena proses impor ulang memerlukan pembacaan yang digunakan untuk membuat produk tersebut. |
| `--daq-tolerance-ms MS` | Sejauh mana pembacaan dapat menyimpang dari eksposur dan tetap dianggap dalam toleransi (default 500). Hanya hasilnya yang berubah (untuk `.match.json`, `in_tolerance`, dan `tolerance_ms`, serta `DlsInTolerance` untuk reflektansi), bukan nilai pembacaan yang digunakan. Lihat [Pembacaan DAQ yang Jauh dari Nilai Paparan](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | Cara lama untuk meminta `.daq` pada pengambilan gambar mentah saja. Ini sekarang menjadi pengaturan default, dan `lattice capture` serta `array-capture` masih menerima bendera tersebut (`multi-capture` tidak). `--no-daq` memiliki prioritas lebih tinggi daripada bendera tersebut. |

DAQ mana: yang terikat pada kamera atau array (pilihan sensor cahaya di GUI, pengikatan yang disimpan dalam proyek, atau `chloros-cli daq pool-bind`), atau yang terbaru yang dimiliki oleh backend. Sebuah burst mentah (`array-burst`) menyimpan `.daq`-nya sendiri untuk setiap pembacaan yang berbeda di dalam folder burst seperti sebelumnya.

**Format PNG dan JPEG menyimpan 8 bit per kanal, sehingga hanya menggunakan `display` / `preview` dan produk indeks saja.** `--format png` atau `jpg` dengan tingkat yang tidak dapat ditampung akan ditolak sebelum apa pun direkam, dengan menyebutkan tingkat tersebut dan TIFF sebagai format yang menampungnya:

- `radiance` (float32) dan `reflectance` (uint16, `32768` = 1,0) — selalu, sehingga `--processing all` (default) dengan `--format png`/`jpg` juga ditolak: gunakan `--format tiff`, atau `--levels display`.
- `raw` dan `debayered` dari kamera yang melakukan streaming lebih dari 8 bit (frame M3C 12-bit / Mono12 M3M) — CLI tidak dapat mendeteksi kedalaman bit kamera, sehingga backend menolak frame-frame ini (HTTP 400). Frame tunggal`lattice capture --processing debayered` yang hanya berisi satu frame mengambil frame 8-bit untuk PNG/JPEG dan tidak ditolak, begitu pula dengan `multi-capture` tanpa array persisten (8-bit debayered).

Setiap rute penangkapan memberikan respons yang sama (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst`, serta rute HTTP yang berada di belakangnya.

### Seperti apa tampilan folder hasil perekaman

Setiap jenis ekspor disimpan di **subfolder tersendiri**di bawah `-o`, sehingga hasil perekaman bertingkat tidak akan mencampurkan jenis-jenis tersebut. Sketsa tersebut menunjukkan**penangkapan default `all` dari satu kamera oleh `lattice capture`** — beberapa tingkat per kamera, sehingga setiap file memiliki akhiran `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Perbedaan nama file:

- **Bidang kamera — nomor seri, lalu filter:** setiap file perekaman mencantumkan nama kamera yang mengambilnya sebagai `SN<serial>-<filter>`, di mana `<filter>` adalah bagian filter dari model kamera: `SN261902142-F450` untuk `M3M-L87-F450`, `SN213800234-FRGN` untuk `M3C-L87-FRGN`, `SN214701292-FRGB` untuk kamera RGB. Filter tersebut digabungkan dengan nomor seri menjadi `-`, bukan dengan `_`, sehingga nama file memiliki bidang-bidang yang dipisahkan oleh `_` seperti sebelumnya (awalan, tanggal, waktu, milidetik, kamera, level). Kamera yang modelnya tidak dapat dibaca (DeviceUserID yang dihapus) diberi nama hanya `SN<serial>`. Rekaman dari rilis sebelumnya tidak memiliki filter dalam namanya; Chloros membaca keduanya.
- **Awalan:** `lattice capture` memberi nama berkas `capture_…`; `array-capture`, `multi-capture` dengan rangkaian yang terhubung, dan `project capture` / `burst` / `stream` dari sebuah array menamainya `sync_<ts>_SN<serial>-<filter>…`.
- **Sufiks level hanya berlaku untuk beberapa level.**Kamera yang menyimpan**beberapa**level dalam satu pengambilan (`all`, atau `--levels a,b`) menambahkan `_<level>`. Kamera yang menyimpan**satu** tingkat mempertahankan nama biasa, tanpa sufiks — `--processing reflectance` menulis `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — sehingga skrip yang ditulis untuk pengambilan gambar satu tingkat tetap berfungsi. Tingkat tersebut masih ditentukan oleh folder.
- **Masker clamp mengikuti nama produk:** `<product name>_clampmask.tif`. Pengambilan data reflektansi satu tingkat menghasilkan `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; pengambilan data `all` menghasilkan `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **Ekstensi** mengikuti `--format`: nilai default `tiff` menghasilkan `.tiff`, sedangkan `--format tif` menghasilkan `.tif` (masker penjepit selalu `.tif`).
- **Satu pengecualian:** `lattice capture --processing debayered` (atau `--levels debayered`) mengambil jalur-frame dan menulis `capture_<ts>_SN<serial>-<filter>.tiff` langsung ke `-o` — tanpa subfolder.
- **Aliran dan burst** membawa bidang yang sama: `lattice continuous` menulis `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff`, dan burst mentah `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` adalah cap waktu perekaman dan `<serial>` adalah nomor seri kamera, sehingga satu grup yang disinkronkan berbagi
cap waktu di seluruh kamera. **Perhatikan satu ketidaksimetrisan:** tingkat `display` disimpan dalam folder
bernama `preview/`, sedangkan berkas multi-tingkat mempertahankan `_display` dalam nama — folder dan sufiksnya berbeda
hanya untuk tingkat tersebut. Tingkat yang tidak dikenal akan dialihkan ke folder dengan nama yang sama, dan jika subfolder
tidak dapat dibuat, berkas tersebut akan ditulis ke akar output daripada hilang.

**Memproses ulang folder captures:**arahkan `chloros-cli process` ke**akar folder captures**
(`output/`). `process` biasanya hanya mengimpor folder yang Anda tentukan, tetapi jika folder tersebut tidak berisi
gambar dan memiliki subfolder, maka program akan menelusuri subfolder tersebut secara otomatis — sehingga subfolder pada setiap tingkat dan
subfolder `daq/` semuanya diambil sekaligus. Setiap tingkat dari hasil tangkapan diimpor sebagai satu gambar tunggal dengan
tingkat-tingkat lainnya tersedia sebagai mode, bukan sebagai satu gambar per tingkat.

Menamai **subfolder tingkat** secara langsung (misalnya `output/raw/`) juga dapat dilakukan. Melakukan hal tersebut akan meninggalkan `daq/`
tertinggal, jadi salin atau arahkan pembacaan DAQ tersebut bersamanya saat Andamengolah produk radiometrik
dari `raw/` — jika tidak, pencocokan cap waktu tidak akan memiliki acuan untuk diselesaikan.

**Pemrosesan selalu dimulai dari `raw`.** Di dalam setiap pengambilan data, bingkai mentah merupakan sumber pipa pemrosesan;
`debayered`, `radiance`, `reflectance`, dan `preview` muncul sebagai mode tampilan tetapi tidak pernah dimasukkan
kembali melalui jalur pemrosesan. Memproses ulang produk turunan akan menerapkan kembali efek vignette, CCM, dan
perhitungan radian yang sudah tertanam dalam pikselnya, sehingga Chloros memilih untuk tidak melakukannya daripada
melakukan pemrosesan ganda. Dua konsekuensi yang perlu diketahui:

- Hasil render `index/` dan `composite/` **tidak pernah** diproses. Keduanya merupakan keluaran, bukan tangkapan —
  render LUT NDVI tidak memiliki interpretasi radian yang bermakna.
- Folder tangkapan yang diekspor **tanpa** `raw` (misalnya `array-capture --processing reflectance`) tidak memiliki
  tidak memiliki sumber pipeline yang sah. Capture tersebut dapat diimpor dan ditampilkan secara normal, tetapi `process` akan melewatkannya
  dan menampilkannya sebagai berikut:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Jika Anda benar-benar perlu memproses produk turunan — sesi hub yang direkam dengan
  `demosaic` diaktifkan, atau folder warisan — `--input-level {raw,debayered,processed}` memaksa titik masuk
  dan mengabaikan perintah lewati. Bendera tersebut merupakan jalan keluar yang disengaja; `auto` (pengaturan default)
  tidak pernah memproses penangkapan yang tidak memiliki data mentah.

**Tag kalibrasi: hanya pada `raw`.** Berkas `raw` dari suatu penangkapan data berisi paket kalibrasi kamera: muatan `CalibrationJSON` dan tag yang kompatibel dengan MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), yang menggambarkan nilai mentah sensor. Setiap berkas pada level lainnya (`debayered`, `display`, `radiance`, `reflectance`, render `index/`) serta setiap bingkai `lattice continuous` / `multi-stream` membawa identitas kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) serta catatan bingkai itu sendiri, `XMP-mapir:FrameJSON` (pencahayaan, penguatan, suhu, format piksel, waktu pengambilan, kecocokan DAQ), karena setiap produk `process` memiliki identitas tersebut. `process` menemukan kalibrasi file tersebut berdasarkan nomor serinya dan `CalBundleSha256` di cache kalibrasi Chloros, mengunduh paket tersebut sekali jika komputer tidak memilikinya.

#### File yang dibuat dari beberapa kamera

File yang dibuat dari pita spektral lebih dari satu kamera — tumpukan multiband (`align-apply`, `align-stream`, `index --live --save-multiband`), indeks langsung yang dibuat dari beberapa kamera (`index.tif`, `index_LUT.png`), gambar `composite/` dari pengambilan array — mencantumkan nama setiap kamera yang digunakan:

- **Daftar per kamera, berdasarkan urutan pita** (komposit: lapisan latar belakang, lalu latar depan; lapisan indeks, kamera-kamera yang menjadi sumber pita-pitanya): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (paket kalibrasi yang dilalui oleh setiap pita kamera, atau — tumpukan `align-apply` / `align-stream`, yang pita-pitanya tidak melalui proses kalibrasi apa pun — paket kamera yang telah disimpan dalam cache oleh Chloros, yaitu yang namanya tercantum dalam berkas tangkapannya), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — daftar paralel, di mana item *n* dari masing-masing daftar menggambarkan kamera yang sama; `-` di mana sebuah kamera tidak memiliki nilai (tidak ada paket kalibrasi, lensa tidak dikenal). Setiap kamera adalah **satu** item: jika paketnya memiliki beberapa nilai untuk suatu bidang — misalnya sertifikat lampu halogen dalam paket — nilai-nilai tersebut digabungkan dengan `;` (`F-1479-1;F-1480-1;F-1481-1`), bukan koma, sehingga daftar tersebut menampilkan satu item per kamera terlepas dari cara pencetakannya.
- **EXIF `SerialNumber`** mengandung nomor seri yang sama, disatukan dengan koma (`214000535,214000551`).
- Data ini **tidak** mengandung identitas kamera tunggal (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): yang tersebut menamai satu kamera dan paketnya, dan `process` akan menentukan kalibrasi file dari data tersebut. Indeks yang dihitung dari pita beberapa kamera (`index.tif`, `index_LUT.png`) memiliki EXIF `Model` `LATTICE` (rig), bukan model `LATT-…` dari satu kamera.

Sebuah berkas yang berisi band dari **satu** kamera — baik berkas TIFF per-band (`--per-band`), sebuah bingkai `align-apply --per-camera`, tumpukan atau indeks langsung dari satu kamera — membawa tag identitas kamera tersebut sendiri, seperti halnya setiap berkas tangkapan lainnya. Tumpukan `align-apply`, berkas per-band dan per-kamera, serta setiap tumpukan `align-stream` dan berkas per-band-nya, membawa waktu pengambilan gambar kelompoknya (waktu pemaparan kamera referensi, UTC), dan bingkai `--per-camera` dari kamera mono merupakan satu bidang, pada `--bit-depth` seperti halnya tumpukan.**Orientasi kamera (balik / putar).** Pengaturan orientasi kamera (opsi Mirror H /
Mirror V / rotasi) menentukan apa yang disimpan dalam hasil pengambilan gambar:

- **`raw` dan `debayered` disimpan persis seperti yang dibaca oleh sensor**, apa pun pengaturannya. Sebuah
  Mosaik Bayer yang diputar atau dibalik akan menjadi pola filter warna yang berbeda, dan peta kalibrasi
  (dark offset, flat field) berada pada grid sensor itu sendiri.
- **`preview`/`display`, `radiance`, `reflectance` dan hasil render `index/` diorientasikan** — diputar
  sekali, setelah semua perhitungan kalibrasi, sehingga file-file tersebut terbuka sesuai dengan cara kamera membingkai objek. File-file
  `clamp_masks/…_clampmask.tif` berada pada kisi yang sama.
- **Setiap file tangkapan merekam pengaturan** dalam XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, searah jarum jam, setelah
  proses pembalikan) dan `ChlorosOrientationApplied` — `false` pada file mentah dan yang telah di-debayering (piksel asli),
  `true` pada hasil yang telah diorientasikan. `ChlorosProcessing` mencantumkan operasi yang diterapkan pada piksel file tersebut
  (`none` jika dalam format asli). Hal itu mencakup setiap bingkai yang dialirkan oleh `lattice continuous` (
  pengaturan kamera itu sendiri, belum diterapkan) dan setiap bingkai dari `lattice multi-stream` (identitas:
  bingkai-bingkai anggotanya bersifat asli, seperti pada pengambilan gambar array). Tumpukan `lattice align-stream` / `align-apply`
  mencatat pengaturan kamera utama, yang belum diterapkan: pita-pitanya diwarp ke dalam
  bingkai kamera utama tetapi tidak diputar.
- **`process` menerapkan orientasi yang direkam** saat memproses data mentah: offset gelap, flat
  field, demosaik, dan pemisahan warna dalam koordinat sensor terlebih dahulu, diikuti oleh distorsi penyelarasan array, dan
  orientasi terakhir — sehingga produk pratinjau, radian, reflektansi, dan indeks dihasilkan persis seperti
  pada pengambilan langsung, dan merekam `ChlorosOrientationApplied=true`. Produk cadangan yang belum dikalibrasi
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) diorientasikan dengan cara yang sama, setelah
  bidang datar. Produk yang telah melalui proses debayering tetap dalam format aslinya.
- **Penampil gambar Chloros menampilkan file mentah atau yang telah melalui proses debayering yang diputar sesuai urutan rekamannya** (hanya tampilan —
  file tidak diubah), sehingga tampilannya mirip dengan produk-produknya; thumbnail, tampilan ukuran penuh,
  tampilan kursor, dan kotak pasir Indeks/LUT semuanya menggunakan orientasi yang sama. File tanpa catatan, dan
  produk (yang sudah diorientasikan), ditampilkan sesuai penyimpanannya. Ekspor kotak pasir mencatat orientasi yang
  dibawa oleh pikselnya (`ChlorosOrientationApplied=true`; identitas untuk lapisan tanpa catatan).
- **Pengambilan gambar tanpa catatan diproses sebagai file asli** (perilaku tidak berubah). Sebelum versi 1.3.0,
  pengambilan gambar dengan satu kamera yang memiliki pengaturan pembalikan atau rotasi akan memutar sendiri file mentah dan file yang telah di-debayering;
  Chloros tidak dapat membedakan file mentah yang sudah-dari file mentah asli dan tidak menebak, sehingga
  memprosesnya sesuai dengan yang disimpan. Peta kalibrasi berada pada grid sensor itu sendiri, jadi untuk
  perekaman semacam itu, gunakan yang diambil dengan versi 1.3.0 jika kalibrasinya penting.
- **Array tidak memutar file anggotanya secara default** (pergeseran penyelarasan dan file XMP `Alignment*` berada dalam
  bingkai asli kamera yang belum diputar) dan mencatat identitasnya. `array-capture --orient` memutar hasil
  setiap anggota, setelah transformasi penyelarasan: berdasarkan orientasi **master** saat mereka
  diputar ke dalam bingkai master (sehingga anggota-anggota tetap terdaftar bersama), atau berdasarkan orientasi kamera masing-masing. Setiap
  file mencatat orientasi bingkai tempat pikselnya berada — sebuah file mentah, sesuai kamera masing-masing —
  dan file mentah yang membawa transformasi penyelarasan juga mencatat `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: milik master, untuk produk yang di-warp
  ke dalam bingkai master. `process` memutar setiap produk sesuai dengan yang dimiliki oleh pikselnya:
  yang telah diselaraskan saat memutar file mentah (`--array-alignment`, default — hasil penangkapan yang diselaraskan secara langsung
  produk tangkapan), produk kamera itu sendiri saat tidak dilakukan (`--no-array-alignment` — hasil
  perekaman langsung yang tidak disejajarkan). Opsi yang sama tersedia untuk `array-burst`, `project capture` / `burst` /
  `stream`, resep, dan SDK.

### Pengambilan Gambar yang Dilewati dalam Susunan Filter Campuran

Saat Anda mencampur kamera RGB dan multispektral dalam satu susunan, `array-capture --processing radiance` (atau `reflectance`) menyimpan bingkai multispektral dan **melewati** kamera RGB-nya — radian per-Bayer tidak bermakna untuk sensor broadband. CLI menampilkan setiap file yang disimpan (bersama tingkat ekspornya), setiap `.daq` yang ditulis, dan setiap pengabaian secara eksplisit, sehingga jumlah file tidak mengejutkan:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Token alasan lewati mengikuti pola `<level>-not-applicable-to-rgb-cam`. Kamera yang paket kalibrasinya tidak dimuat akan melewati `radiance` / `reflectance` dengan `<level>-needs-calibration (connect the camera so its cal pack loads)`, dan jika tidak ada pembacaan DAQ reflektansi yang dapat digunakan, akan melewati dengan `reflectance-skipped-no-dls`. Reflektansi juga dapat dilewati dengan `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, dan dengan `dls-uncalibrated-band-<nm>` ketika pita spektrum sebagian besar berada di luar rentang yang dikalibrasi secara radiometrik dari sensor cahaya DAQ (~374–974 nm) — di antara SKU yang tersedia, hanya F988 yang mendukung alur kerja panel reflektansi.

Gunakan `--processing debayered` (atau `display`) untuk menyertakan setiap kamera terlepas dari jenis filternya, atau `all` (default) untuk mendapatkan setiap level yang berlaku per kamera dalam satu kali pengambilan.

### Pembacaan DAQ yang Jauh dari Eksposur

Setiap `.daq` yang disimpan oleh penangkapan adalah pembacaan DAQ yang sesuai dengan eksposur tersebut, dan `.match.json` di sebelahnya mencatat seberapa jauh jarak antara keduanya (`gap_ms`) serta apakah selisih tersebut masih dalam batas toleransi (`in_tolerance`, 500 ms kecuali jika pengaturan penangkapan menentukan `--daq-tolerance-ms`). Pembacaan di luar batas tersebut tetap digunakan, sehingga tidak ada bingkai yang hilang, dan diberi label: file XMP berkas reflektansi memuat `DlsInTolerance=False` dan `DlsMatchGapS`. `lattice capture`, `multi-capture`, dan `array-capture` juga mencantumkannya di bawah `.daq` yang bersangkutan, dengan kesalahan aliran DAQ'ketika dijelaskan:

**Bagaimana pembacaan dipadukan dengan waktu pemaparan.** Sebuah penangkapan menunggu pembacaan pertama DAQ setelah waktu pemaparan, sehingga dapat menggabungkan pembacaan di kedua sisi waktu pemaparan tersebut (`method` `interpolated`). DAQ-E mengirimkan pembacaan setiap 0,4 detik di siang hari dan setiap 1–2 detik di dalam ruangan, sehingga waktu tunggu tersebut berjumlah sekitar satu setengah kali interval tersebut untuk perintah, bukan untuk eksposur. DAQ yang telah berhenti mengirimkan data tidak ditunggu. Pembacaan DAQ-E adalah rata-rata cahaya selama interval sejak pembacaan sebelumnya, sehingga ditempatkan di tengah interval tersebut: `gap_ms` diukur dari waktu pemaparan hingga titik tengah tersebut, dan `bracket_stamps_ns` memberikan cap waktu pembacaan itu sendiri. `array-capture --continuous` menunggu tanpa memperlambat proses pengambilan: berkas mentahnya langsung ditulis, dan `.daq` ditulis begitu pembacaan tiba. Dengan `--interval`, setiap putaran menunggu seperti pengambilan data tunggal.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Jika tidak ada berkas reflektansi yang ditulis dari pembacaan tersebut, baris pertama diakhiri dengan `saved anyway; its .match.json says in_tolerance: false`.

Pengambilan data tetap berakhir dengan `0`: setiap berkas telah ditulis dan mencantumkan isinya. Sebuah DAQ yang alirannya terhenti terus mengirimkan pembacaan terakhirnya, jadi periksalah dengan `chloros-cli daq pool-list` sebelum mengandalkan nilai reflektansi dari pengambilan data tersebut. Backend akan menghubungkan kembali DAQ-E yang digabungkan (pooled) yang telah berhenti mengirimkan data (lihat [DAQ-E yang Berhenti Mengirimkan Data](#a-daq-e-that-stops-delivering)). Pembacaan yang tidak memenuhi toleransi hanya mencetak baris `DAQ:`-nya, kecuali jika itu adalah pembacaan yang disimpan (di bawah).

**Pembacaan yang disimpan setelah DAQ-nya tidak tersedia.** Ketika tidak ada DAQ yang dapat menyediakan pembacaan untuk suatu eksposur (DAQ yang terikat tidak tersedia, atau tidak ada DAQ terpusat yang memilikinya, misalnya setelah `daq pool-disconnect`), proses penangkapan akan menggunakan kembali pembacaan terakhir yang dimiliki Chloros, dan baris `.match.json` menggambarkan pembacaan tersebut terhadap paparan ini: `method` `retained`, `gap_ms` dari eksposur ini ke sampel yang menjadi dasar pembacaan tersebut, `in_tolerance` dari celah tersebut, dan `retained_from_epoch_ns`, yaitu eksposur yang pertama kali dicocokkan dengan pembacaan tersebut. Oleh karena itu, penangkapan data yang diambil lama setelah DAQ dimatikan akan mencetak baris-baris di atas, dan di bawah garis-garis tersebut, yang dicetak bahkan saat pembacaan masih dalam batas toleransi:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

`.daq` masih mencantumkan nama DAQ yang mengambil pembacaan tersebut, beserta kalibrasi dan penutup yang digunakan saat pembacaan diambil, sehingga saat diimpor kembali, sistem mengetahui DAQ, kalibrasi, dan penutup mana yang menghasilkan pembacaan tersebut.

### File yang Gagal Ditulis

Kamera yang file-nya tidak dapat ditulis — misalnya karena drive penuh atau kesalahan izin — tidak akan menghentikan penyimpanan file dari kamera lainnya. Setiap perintah pengambilan gambar (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; serta `capture`, `burst`, dan `stream` dari sebuah resep) mencetak setiap berkas yang gagal ke stderr, kemudian **`1` keluar** meskipun sisanya tersimpan:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Setiap berkas yang menjadi bagian dari tangkapan telah tercakup, termasuk`lattice capture --levels debayered` yang telah melalui proses debayering satu frame. Sebuah pembacaan `.daq` yang sidecar asal-usulnya (`.match.json`) tidak dapat ditulis terdaftar sebagai `[match]` (`.daq` itu sendiri telah disimpan).

Pada drive yang penuh, penangkapan yang berulang (`array-capture --continuous` / `--interval`, `project burst` / `stream`, sebuah resep `burst` / `stream`) **berhenti pada tahap tersebut** alih-alih tidak menulis apa pun ke drive yang sama selama sisa proses. Kegagalan lainnya dilaporkan pada setiap tahap saat terjadi, dan proses berlanjut hingga akhir, lalu keluar dari `1`. `array-capture --continuous` (penangkapan berkelanjutan dari backend) tidak mencantumkan file apa pun, termasuk yang gagal: file-file tersebut dihitung pada baris kemajuan dan ringkasannya (`N failed`, `N file(s) failed to write.`) dan keluar dari `1` — lihat [Mode Berkelanjutan](#continuous-mode---continuous).

Perekam — `array-burst` dan `array-record` — melaporkan kegagalan mereka sendiri. `array-burst` menghentikan dirinya sendiri ketika drive penuh, atau tepat sebelum itu terjadi. Keduanya tetap berjalan meskipun terjadi kegagalan penulisan lainnya. Ketika salah satunya melaporkan kesalahan, pesan tersebut dicetak setelah penghitung, dan perintah **keluar dari `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` menggantikan `Disk full:` berarti proses perekaman berhenti sebelum drive drive. Frame yang direkam sebelum penghentian tetap disimpan. `array-burst --build` melewatkan proses pembuatan saat ruang penyimpanan habis, karena video akan disimpan ke drive yang sama. Sediakan ruang kosong terlebih dahulu, lalu jalankan `array-build-video --burst-dir …`. Setelah kesalahan apa pun, proses pembuatan tetap berjalan.

`array-record` yang tidak menulis bingkai sama sekali juga merupakan perekaman yang gagal (`Recorder error: no-frames: the combined view published no frame to record …`, keluar dari `1`) — lihat [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Mode Perekaman, Perekam, & Pemrosesan Ulang Offline

Semua ini beroperasi pada **array persisten** (jalankan `array-connect` terlebih dahulu). Fitur-fitur ini mencerminkan panel perekaman antarmuka pengguna (GUI).

### Mode `array-capture`

`array-capture` adalah perintah tunggal dengan empat mode rana ditambah serangkaian tombol pengaturan ekspor:

| Mode | Bendera | Perilaku |
| --- | --- | --- |
| **Tunggal** *(default)* | (tidak ada) | Satu grup penangkapan yang disinkronkan, lalu keluar. |
| **Berkelanjutan** | `--continuous` | Pengambilan gambar berturut-turut hingga `Ctrl+C`, `--count N`, `--duration S`, atau drive penuh. Menjalankan penangkapan berkelanjutan milik backend (mode Berkelanjutan pada GUI) — lihat [mode Berkelanjutan](#continuous-mode---continuous) di bawah ini. |
| **Interval** | `--interval S` | Satu putaran setiap `S` detik (diukur sejak awal setiap putaran), batas yang sama. |
| **Tercepat** | `--fastest` | Hanya data mentah-hanya + pembacaan DAQ yang ditetapkan + komposit indeks gabungan; melewati perhitungan radian/reflektansi/tampilan sehingga bingkai ditampilkan dengan cepat. Mengimplikasikan `--processing raw --force-daq`. Proses ulang `.daq` yang disimpan menjadi produk yang dikalibrasi nanti. |

Tombol pengaturan ekspor (dapat digabungkan dengan mode apa pun; semuanya menggunakan antarmuka pengguna/titik akhir SDK yang sama):

| Bendera | Efek |
| --- | --- |
| `--array-id ID` | Array yang akan diambil datanya (lewati jika hanya satu yang terhubung). |
| `-o, --output DIR` | Folder keluaran (default `output`, di bawah folder tempat Anda menjalankan perintah). |
| `--format {tiff,tif,png,jpg,jpeg}` | Format file (default `tiff`). PNG/JPEG hanya menggunakan `display` dan indeks render; lihat [Tingkat Ekspor Capture](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Tingkat ekspor tunggal (`raw`, `debayered`, `radiance`, `reflectance`, `display`), atau `all` (default). |
| `--levels L1,L2,…` | Subset eksplisit dari tipe ekspor (misalnya `raw,radiance,reflectance`); **menggantikan `--processing`**. |
| `--aligned` / `--no-aligned` | **Default: teralignasi, seperti penangkapan array yang sama oleh aplikasi Chloros**(pengaturan penangkapan**Teralignasi** per-array-nya, aktif kecuali Anda mematikannya). Ekspor non-raw setiap anggota (`debayered`, `display`, `radiance`, `reflectance`, render indeks) disesuaikan ke bingkai kamera utama dan dipotong sesuai tumpang tindih bersama antar anggota, sehingga file-file tersebut terdaftar pikselnya dan menghasilkan ukuran, pemotongan, serta tag `Alignment*` yang sama dengan yang digunakan aplikasi — lebih kecil dari sensor (atau ROI) sebesar offset anggota; di bawah ROI, transformasi disesuaikan kembali ke jendela. `XMP-mapir:AlignmentJSON` menyatakan `"applied": true` dan menghasilkan `crop_roi`. File mentah tetap tidak di-warp pada ukuran aslinya tetapi menyimpan transformasi dalam metadata (`"applied": false`). `--no-aligned` menyimpan setiap file pada ukuran aslinya, tanpa di-warp. `--aligned` adalah pengaturan default yang dijelaskan. Terlepas dari apakah transformasi diterapkan atau tidak, setiap file anggota mencatat transformasi kamera masing-masing (`Alignment*`) begitu array memiliki profil penyelarasan — kecuali kamera yang profilnya tidak dapat diselaraskan (dilewati, atau dipaksa menjadi identitas akibat pemecahan yang gagal), serta kamera referensi saat tidak ada kamera lain yang diselaraskan: keduanya tidak memiliki tag `Alignment*`. Jika tidak ada profil pada array, hasil pengambilan gambar disimpan tanpa penyelarasan, dan CLI mencantumkan alasannya pada baris `Alignment:`, seperti yang dilakukannya pada setiap kasus lain di mana distorsi tidak dapat diterapkan (anggota dengan resolusi berbeda, susunan yang tidak disinkronkan melalui kabel, atau bingkai yang profilnya tidak dapat dipetakan). Penyesuaian otomatis susunan akan membuat profil setelah eksposurnya stabil; `align-calibrate` membuatnya sesuai permintaan. |
| `--orient` | Ubah **produk**setiap anggota — `display`, `radiance`, `reflectance`, hasil render `index/`, dan `composite/` — sesuai orientasi kamera, setelah proses penyelarasan: produk yang diselaraskan ke bingkai master (pengaturan default) berdasarkan**orientasi master**, sehingga anggota-anggotanya tetap terdaftar bersama; yang tidak diselaraskan (`--no-aligned`, atau tangkapan yang tidak dapat diterapkan warp) berdasarkan orientasi kameranya sendiri.**Default: nonaktif — setiap berkas anggota tetap dalam format asli**, seperti halnya tangkapan array selalu demikian. `raw` dan `debayered` tetap dalam bingkai asli dalam kedua kasus. Setiap berkas mencatat orientasi yang dibawa oleh produk-produknya (lihat [Orientasi kamera](#what-a-captures-folder-looks-like)); file XMP `Alignment*` tetap berisi transformasi bingkai asli, yang diterapkan sebelum orientasi, sehingga `process` (warp, lalu orientasi) mereproduksi produk langsung. |
| `--index` / `--no-index` | Simpan / lewati overlay indeks vegetasi per kamera jika telah dikonfigurasi. Default: tampilkan. |
| `--no-daq` | Lewati file sidecar `.daq` jika tidak ada level yang dipilih menggunakan DAQ (pengambilan data mentah saja). Secara default, pembacaan disimpan, sehingga bingkai dapat diproses ulang menjadi reflektansi/indeks secara offline. `--force-daq` masih diterima (default). |
| `--daq-tolerance-ms MS` | Toleransi pencocokan DAQ untuk penangkapan ini (default 500 ms). |
| `--smart` | Tunggu hingga AE stabil di semua kamera sebelum memicu pemotretan (lihat [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Kompresi piksel TIFF. `none` (default sejak versi 1.3.0; versi 1.2.0 menulis `deflate`) = tidak terkompresi — piksel sensor persis seperti yang dibaca, ~6,3 MB per bingkai resolusi penuh, dan kecepatan berkelanjutan tercepat; `deflate` = zlib L1 tanpa kehilangan data + prediktor horizontal, ~4,1 MB per bingkai, lebih lambat. Keduanya tanpa kehilangan data dan dibaca secara identik saat diimpor. |

> **TIFF penulisan tunggal + model kecepatan berkelanjutan.**Setiap file ditulis**sekali**: piksel, paket XMP, IFD0 Make/Model/CameraSerialNumber, dan tag EXIF `ExposureTime` / `SerialNumber` dalam satu proses, tanpa ada yang menimpa data tersebut setelahnya. `--continuous`X menangkap kelompok-kelompok yang disinkronkan secara berurutan — setiap penangkapan mengambil kelompok setelah yang sebelumnya, termasuk satu yang dipublikasikan saat penangkapan tersebut masih disimpan — sehingga berjalan pada kecepatan array. DEFLATE menahan GIL Python, sehingga penulisan terkompresi**tidak**diparalelkan di seluruh utas penulis per-kamera; perekaman resolusi penuh yang berkelanjutan pada kecepatan sensor memerlukan penulisan tanpa kompresi (pengaturan default)**dan** disk kelas NVMe (~600 MB/s berkelanjutan untuk susunan 9 kamera pada ~10 fps). Opsi yang sama ditampilkan sebagai `compression` pada `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Mode Kontinu (`--continuous`)

`array-capture --continuous` (tanpa `--interval` atau `--smart`) menyerahkan proses perekaman ke **fitur perekaman berkelanjutan bawaan backend** — sesi yang dijalankan oleh mode Berkelanjutan pada antarmuka pengguna (GUI). Backend menyimpan setiap grup yang disinkronkan secara mandiri, dengan dua penangkapan yang sedang berlangsung dan tanpa permintaan dari CLI di antara grup-grup tersebut, sehingga dapat mengikuti kecepatan array yang tinggi di mana permintaan per penangkapan tertinggal (diukur 73–77 penangkapan/detik pada array 9 kamera dengan ROI 512 piksel). Setiap pengaturan ekspor di atas berlaku untuk setiap penangkapan.

- **Batas:** `--count N` berhenti setelah N penangkapan (satu penangkapan = satu grup yang disinkronkan: file setiap anggota); `--duration S` setelah S detik. Tanpa salah satu dari keduanya, proses akan berjalan hingga `Ctrl+C`, drive penuh, terjadi kesalahan, atau array tidak tersedia. `Ctrl+C` menghentikan sesi backend, lalu mencetak ringkasan.
- **Kemajuan** setiap 2 detik, dengan `, N failed` dan `, N groups skipped` ditambahkan jika nilainya bukan nol (kelompok yang dilewati adalah kelompok yang dipublikasikan oleh array tetapi tidak ada penangkapan yang mengambilnya):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Tidak ada daftar per berkas.** Proses berakhir dengan satu baris ringkasan — penangkapan, laju, berkas, dan folder keluaran — diikuti oleh `N file(s) failed to write.` / `N group(s) skipped.` jika nilainya bukan nol:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Berhenti** saat drive penuh dengan pesan `Continuous capture stopped: the drive is full.`, dan saat terjadi kesalahan atau array tidak tersedia dengan pesan `Continuous capture stopped: <reason>.` (keduanya ditampilkan di stderr).
- **Status keluar:** `0` saat tangkapan telah disimpan dan semua file berhasil ditulis, termasuk `Ctrl+C`; `1` jika ada file yang gagal ditulis, ruang penyimpanan penuh, kesalahan, array terputus, atau tidak ada rekaman sama sekali.
- **Microsoft Defender:** ketika Defender menggunakan satu inti CPU atau lebih untuk memindai file yang ditulis oleh rekaman, proses akan mencetak pesan sekali di stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Lihat [Pengaturan sistem](#windows-exe).
- Satu penangkapan berkelanjutan per array pada satu waktu; penangkapan kedua akan ditolak dengan pesan `a continuous capture is already running for this array`. Melalui HTTP, sesi tersebut adalah `POST /api/camera/array/continuous/start` (isi `/api/camera/array/capture` ditambah `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` dan `POST /api/camera/array/continuous/stop`.

**Interval**(`--interval S`, dengan atau tanpa `--continuous`) dan**`--continuous --smart`** tetap menggunakan loop per-permintaan: satu permintaan penangkapan per putaran — pada setiap waktu yang dijadwalkan untuk Interval, setelah AE stabil untuk `--smart` — dengan file setiap putaran tercantum seperti dalam mode Tunggal, baris `Done — N capture pass(es).` di bagian akhir, dan `--count N` menghitung jumlah putaran.

### `array-record` — video/GIF indeks gabungan (tingkat pemantauan)

Merekam **tampilan indeks gabungan langsung** dari array ke berkas `.avi` (dan opsional ke berkas `.gif`). Saat tampilan gabungan di antarmuka pengguna (GUI) terbuka, fitur ini merekam tampilan tersebut sebagaimana ditampilkan. Jika tidak ada tampilan yang terbuka, backend akan merender tampilan itu sendiri: indeks yang Anda berikan melalui `--expr`, atau secara default NDVI dari array, setelah array tersebut diselaraskan. NDVI default menggunakan pita merah dan NIR yang paling dekat dengan 660 nm dan 850 nm, seperti yang dilakukan tampilan langsung. Tampilan GUI yang dibuka selama perekaman akan direkam selama tampilan tersebut tetap terbuka. Perekam memeriksa kemajuan setiap 2 detik dan berhenti pada `--duration`, `Ctrl+C`, atau saat perekam berhenti secara otomatis.

Perekaman yang tidak menulis bingkai apa pun akan gagal: perekam melaporkan `no-frames: …` (array mungkin masih dalam proses penyelarasan atau penyesuaian eksposur) dan perintah **keluar dari `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--array-id ID` | hanya array | Array target (dihilangkan jika hanya satu yang terhubung). |
| `-o, --output DIR` | `output` | Direktori keluaran. Jalur relatif berada di bawah folder tempat Anda menjalankan perintah. |
| `--fps F` | `10` | Kecepatan bingkai perekaman. |
| `--duration S` | hingga Ctrl+C | Berhenti otomatis setelah `S` detik. |
| `--gif` | nonaktif | Juga simpan GIF animasi. |
| `--gif-only` | nonaktif | Hanya simpan GIF (tanpa `.avi`). |
| `--expr EXPR` | NDVI array | Ekspresi indeks yang akan direkam saat tidak ada tampilan GUI yang terbuka, dalam nama-nama band array (misalnya `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; lihat [nama band](#index--vegetation-maths)). Nama warna singkat (`NIR`, `Red`) hanya berfungsi jika salah satu band dalam array memilikinya. Tampilan GUI yang terbuka akan dicatat sesuai yang ditampilkan. |

### `array-burst` — burst raw-Bayer dengan fps tinggi (kelas analisis)

Membaca buffer grup sinkronisasi loop pengambilan gambar secara langsung — **tidak memerlukan rantai kalibrasi, exiftool, atau tampilan langsung** — sehingga berjalan pada kecepatan pengambilan gambar penuh kamera. Menulis bingkai mentah + manifes per bingkai + satu `.daq` per pembacaan DLS yang berbeda di bawah `<output>/bursts/<base>/`. Proses ulang secara offline (perintah berikutnya), atau berikan `--build` untuk melakukannya segera setelah berhenti.

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
| `--max-frames N` | tak terbatas | Berhenti otomatisberhenti setelah `N` frame mentah. |
| `--orient` | mati | Produk yang dihasilkan oleh proses build dari burst ini disesuaikan dengan orientasi kamera — produk per kamera disesuaikan dengan orientasi kameranya sendiri, indeks gabungan mengikuti orientasi kamera utama — seperti yang dilakukan oleh `array-capture --orient`. Frame mentah tetap dalam format aslinya; `manifest.json` dari rentetan tersebut mencatat orientasi setiap anggota dan opsi yang dipilih. |
| `--build` | nonaktif | Setelah berhenti, segera proses ulang rentetan tersebut (sama seperti `array-build-video`). |
| `--products …` | `combined:index` | Dengan `--build`: video mana yang akan dibuat (lihat di bawah). |
| `--fps F` | `10` | Bersama dengan `--build`: fps video keluaran. |
| `--save-tiffs` | nonaktif | Bersama dengan `--build`: juga simpan file TIFF yang telah dikalibrasi per-frame (masing-masing dengan [masker clamp unmix](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on) saat sebuah piksel di-clamp; hasil build mencantumkannya sebagai `clamp_masks`). |
| `--gif` | nonaktif | Dengan `--build`: juga menulis GIF animasi. |

### `array-build-video` — memproses ulang secara offline rangkaian gambar yang disimpan

Menyesuaikan waktu setiap bingkai mentah dengan pembacaan `.daq` terdekat yang tersimpan dan memprosesnya melalui **rantai radian / reflektansi / indeks yang sama seperti pipa impor**, menghasilkan satu atau lebih video.

`--products` adalah daftar yang dipisahkan koma dari item `kind:level`, di mana `kind` ∈ `per_cam` | `combined` dan `level` ∈ `radiance` | `reflectance` | `index`. `level` tanpa nilai (tanpa `kind:`) secara default akan menjadi `per_cam`. Nilai defaultnya adalah `combined:index`.

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
| `--products …` | `combined:index` | Daftar `kind:level`, seperti di atas. |
| `--fps F` | `10` | Kecepatan bingkai video keluaran (fps). |
| `--save-tiffs` | nonaktif | Juga menyimpan file TIFF yang telah dikalibrasi per bingkai bersama video(s). |
| `--gif` | off | Juga menulis GIF animasi. |

**Sebuah video tidak memiliki masker clamp** (satu aliran gambar, 8-bit). Sebagai gantinya, hasil kompilasi membawa `clamp_fractions`: per produk video, bagian piksel setiap bingkai yang tidak dicampur dalam rentang yang ditahan oleh kanal R / G / B-nya — nilai `Chloros:Band{R,G,B}UnmixClampFraction` yang akan dicantumkan pada produk radian/reflektansi bingkai tersebut (untuk indeks gabungan, dari setiap kamera anggota). Proses ini mencetak bingkai terburuk per produk pada baris `Clamp:`. `--save-tiffs` menambahkan masker per piksel untuk berkas TIFF, di `clamp_masks/<level>/` di samping folder `<level>/` berkas TIFF tersebut.**Proses build yang dihentikan sebelum selesai tidak akan meninggalkan video yang rusak.** Setiap video ditulis sebagai `<name>.partial.avi` dan baru mendapatkan nama akhirnya setelah selesai, sehingga video lama dengan nama tersebut tetap ada hingga penggantinya selesai sepenuhnya. Jalankan proses pembuatan ulang untuk mengganti file `.partial.avi` yang tertinggal.

Proses ini mencetak setiap video yang ditulis, kemudian setiap hasil `Skipped:` (kamera/level yang diminta — atau indeks gabungan — yang tidak menghasilkan video, beserta alasannya — misalnya tidak ada paket kalibrasi, tidak ada pembacaan DLS untuk reflektansi, atau tidak ada pasangan pita merah/NIR untuk indeks gabungan) serta setiap `Note:`. Kamera RGB yang diminta untuk radian atau reflektansi akan mendapatkan video tampilan sebagai gantinya, yang disesuaikan keseimbangan putihnya jika paket kalibrasi kamera tersedia; kamera yang ditampilkan tanpa kalibrasi tersebut disebutkan dalam `Note:`. Fungsi ini **mengembalikan nilai bukan nol** ketika produk yang diminta dilewati atau tidak dibangun sesuai permintaan (ekspresi indeks yang dikonfigurasi tidak dievaluasi secara offline); video yang berhasil dibangun tetap ditulis dan dicantumkan.

> **Pilih perekam yang tepat.** `array-record` adalah *kelas pemantauan* — ia merekam komposit langsung yang telah dirender: tampilan gabungan antarmuka pengguna (GUI), atau jika tidak ada GUI yang terbuka, tampilan NDVI (atau `--expr`) dari backend itu sendiri. `array-burst` → `array-build-video` adalah *tingkat analisis* — ia menyimpan data mentah sensor pada laju penuh dan merekonstruksi video radian/reflektansi/indeks yang telah dikalibrasi setelahnya, tanpa memerlukan tampilan langsung.

### Kamera Mono (M3M) Pita Tunggal

Seri **M3M**adalah varian mono dari seri Bayer**M3C**: satu filter interferensi pita sempit per kamera (`M3M-<lens>-F<wavelength>`, misalnya `M3M-L87-F685`), sehingga sensor menghasilkan**satu pita skala abu-abu** tanpa mosaik Bayer. Tidak ada proses demosaik yang perlu dilakukan, tidak ada crosstalk antar-saluran yang perlu dipisahkan, dan tidak ada pengaturan keseimbangan putih — seluruh alur kerja warna tampilan RGB sama sekali tidak berlaku.

Artinya pada antarmuka CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**mendeteksi kamera monokrom dan**melewatkannya dengan pesan satu baris** alih-alih menerapkan pengaturan yang tidak berguna. Keduanya tetap berjalan normal saat digunakan pada kamera RGB/Bayer M3C dalam sesi yang sama.
- **Format piksel Bayer menunjukkan kedalaman warnanya.** M3M hanya menawarkan format Mono, sehingga format Bayer dari `--pixel-format` atau `--preset` akan mengatur format Mono dengan kedalaman yang sama: `BayerRG8` dan `--preset high_speed` menghasilkan `Mono8`; `BayerRG12`, `high_quality`, dan `triggered` menghasilkan `Mono12`; nama yang dikemas tetap dikemas (`BayerRG12p` menghasilkan `Mono12p`). Berkas-berkas dan `lattice format` menggunakan format Mono. Sebuah array yang mencampurkan kamera M3C dan M3M menggunakan satu format dengan cara yang sama: Bayer pada setiap M3C, kedalaman Mono-nya pada setiap M3M. Sebuah nama adalah kedalaman bitnya: hal yang sama berlaku sebaliknya, sehingga M3C tidak pernah ditempatkan pada format Mono (`--pixel-format Mono12` adalah `BayerRG12` pada M3C, terlepas dari kamera mana yang menjadi master dalam susunan tersebut), dan `lattice format --set` dipetakan dengan cara yang sama serta mencetak format yang telah ditetapkan.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** masih berfungsi — radiance dan reflectance adalah peta radiometrik *per-band* dan didefinisikan dengan sempurna untuk satu band. Frame mono membawa matriks respons sensor **identitas** (tanpa pemisahan 3×3), sehingga perhitungan kalibrasi berjalan tanpa perubahan.
- **Satu kamera mono tidak dapat menghasilkan indeks vegetasi.**NDVI/NDRE/dll. membutuhkan setidaknya dua pita (mis. Red + NIR). Untuk mendapatkan indeks dari perangkat keras mono, arahkan**beberapa** kamera M3M pada panjang gelombang yang berbeda, gabungkan menjadi satu tumpukan multi-pita, dan hitung indeks *dari tumpukan tersebut*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` membuka kamera secara otomatis (sebuah [sesi sementara](#multi-camera-transient-sessions)), jadi jangan hubungkan apa pun terlebih dahulu: `--serials`, jika tidak, kamera yang diaktifkan dalam profil (lihat [Indeks / Perhitungan Vegetasi](#index--vegetation-maths)). Selama Chloros masih mengendalikan kamera mana pun (antarmuka pengguna, `cam-connect`, `array-connect`) , permintaan tersebut akan ditolak dengan kode status HTTP 409 `CAMERA_HELD` dan program akan keluar `1`; lepaskan kamera-kamera tersebut dengan `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

Simbol-simbol `--channel` harus **persis** sesuai dengan nama saluran preset (peka huruf besar/kecil; NDVI menggunakan huruf kecil `red`, `nir` — lihat `--list-presets`), dan nama sisi band harus sesuai dengan nama band dalam tumpukan yang diselaraskan (mode offline juga menerima indeks band berbasis 0, misalnya `--channel red=0 --channel nir=1`).

Pembeda di seluruh tumpukan adalah token `M3M` dalam string model (token ini tidak pernah muncul dalam string `M3C`), yang ditampilkan ke GUI/SDK sebagai `is_mono`.

---

## Pengaturan & Penyesuaian NIC Host (array LATTICE)

Kamera LATTICE mengalirkan GVSP melalui adaptor Ethernet host, sehingga untuk array multi-kamera, **driver**dan**ukuran receive-ring** sama pentingnya dengan kecepatan tautan. Pengaturan yang salah akan muncul sebagai `FRAMES WILL DROP` / `Reduce ROI to enable` di panel Pengaturan Array (dan di `lattice network-analysis` / `analyze_array_network()` dalam SDK), meskipun kamera-kamera tersebut dalam kondisi baik.

### Adaptor USB 10GbE — Realtek RTL8157 ("Realtek USB 10GbE Family Controller")

Pengaturan di bawah ini adalah properti driver Windows. Di Linux, adaptor USB juga memiliki batasan kecepatan (400 MB/s pada x86_64, 200 MB/s pada NVIDIA Jetson) (lihat [Model fps & burst array](#array-fps--burst-model)), dan buffer penerimaan soket kernel berpengaruh pada setiap adaptor — lihat [Host Linux](#linux-hosts--socket-receive-buffers).

| Item | Nilai yang diperlukan | Mengapa hal ini penting |
| --- | --- | --- |
| **Versi driver**|**≥ v10.67 (Jan 2026)**, INF `rtump64x64sta.inf` | Driver**2016**yang sudah usang (v10.65, `rtump64x64.inf`) tidak menangani proses pematian daya dan bugcheck dengan benar, yang menyebabkan**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**saat shutdown/restart/sleep. Proses transisi macet (timeout ~5 menit), pengguna mematikan daya secara paksa, dan pematian sistem yang tidak teratur yang berulang-ulang**merusak repositori WMI**(PowerShell/alat mulai gagal dengan kode `Invalid class`) serta**menyebabkan USB stack macet** pada booting berikutnya (NIC tidak dapat diaktifkan; drive USB berhenti melakukan enumerasi). Lakukan pembaruan dari realtek.com (atau vendor dongle) sebelum mengandalkan restart yang bersih. |
| **Buffer Penerimaan**— kata kunci `ReceiveBufferLen` |**256**(maksimum driver) | Cincin RX NIC. Nilai default driver sebesar**32**hanya menyisakan ~0,26 MB ruang cincin yang dapat digunakan — jauh terlalu kecil untuk burst multi-kamera — sehingga panel array melaporkan `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` dan menghalangi koneksi. Pada**256**cincinnya besar (**~13,5 MB yang diukur pada host 10GbE di lab**), memberikan ruang gerak yang cukup bagi pipa RX untuk burst GVSP multi-kamera. (Apakah konfigurasi tertentu benar-benar *terhubung* ditentukan oleh dua pemeriksaan — pemeriksaan **drain-aware**dan pemeriksaan**over-subscription agregat** — bukan perbandingan mentah antara burst dan ring; lihat [Model fps & burst Array](#array-fps--burst-model).) |
| **URB Penerima**— kata kunci `PendingReceives` |**64** (maks.) | Blok permintaan USB yang sedang dikirim; tingkatkan bersamaan dengan Buffer Penerimaan untuk penyerapan burst. |
| **Jumbo Frame** — kata kunci `*JumboPacket` | **9014** | Diperlukan untuk paket GVSP berukuran 9000 byte (jumlah paket per frame 6 kali lebih sedikit daripada 1500). |

> ⚠️ **Pembaruan driver NIC AKAN MENGEMBALIKAN properti lanjutan ini ke pengaturan default.**Setelah memperbarui atau mengganti driver adaptor,**terapkan kembali** `ReceiveBufferLen=256` dan `PendingReceives=64`, atau panel array akan memblokir koneksi lagi meskipun "tidak ada perubahan pada perangkat keras." Ini adalah penyebab utama rig yang sebelumnya berfungsi tiba-tiba menolak terhubung.

Terapkan dari PowerShell dengan **hak akses administrator** (ganti dengan nama adaptor Anda, misalnya `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` mencakup adaptor USB 10GbE.** Kini sistem mendeteksi jenis adaptor dan menyesuaikan kata kunci receive-ring yang tepat: `*ReceiveBuffers`→2048 untuk NIC PCIe (Intel I219, dll.), atau `ReceiveBufferLen`→256 + `PendingReceives`→64 untuk pengontrol **USB** 10GbE Realtek (yang tidak mengekspos `*ReceiveBuffers`). Nilai target dibatasi hingga batas maksimum yang dilaporkan oleh masing-masing driver (`NumericParameterMaxValue`), sehingga tidak akan pernah menulis nilai di luar rentang. Jalankan dari terminal dengan **hak akses administrator**; seperti pengaturan berbasis registri lainnya, perubahan ini berlaku setelah adaptor di-restart atau sistem di-reboot. Perintah manual `Set-NetAdapterAdvancedProperty` di atas tetap menjadi alternatif yang baik — perintah tersebut berlaku secara langsung (mengikat ulang adaptor) tanpa perlu restart.

### Host Linux — buffer penerimaan soket

Di Linux, termasuk NVIDIA Jetson, buffer penerimaan soket pada kernel sama pentingnya dengan ring penerimaan pada NIC. Pada pengaturan default kernel (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 byte), serangkaian paket GVSP dari kamera akan meluap dari soket aliran sebelum dibaca oleh Chloros. Paket-paket tersebut dibuang di kernel sementara NIC itu sendiri tidak melaporkan adanya kehilangan, dan sebuah array kehilangan kelompok-kelompok frame serta menurunkan laju bingkainya. Diukur pada array 4 kamera yang terhubung melalui adaptor USB 10 GbE, menaikkan kedua nilai tersebut menjadi 32 MB meningkatkan frame rate array dari 4,96 menjadi 7,52 fps setelah terhubung, serta meningkatkan kecepatan perekaman mentah berkelanjutan dari 4,7 menjadi 7,6 perekaman per detik, tanpa ada paket yang terbuang di kernel.

- **Paket .deb yang mengaturnya.** Menginstal paket tersebut akan menulis nilai `/etc/sysctl.d/60-chloros-gige.conf` (keduanya pada `33554432`) dan langsung menerapkannya. Nilai yang sudah diatur lebih tinggi akan dibiarkan apa adanya.
- **`lattice network` memeriksanya.** Jika salah satunya lebih rendah, `socket receive buffers too small for GigE Vision (…)` akan tercantum di bawah *Masalah yang ditemukan*, dan log backend akan menampilkan peringatan yang sama saat kamera atau array pertama kali terhubung.
- **Untuk menaikkan nilainya secara manual** (sampai reboot berikutnya; berkas paket akan menerapkannya pada setiap kali boot):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` juga menaikkan dan mempertahankan nilai-nilai tersebut, tetapi hanya jika backend dijalankan sebagai root. Backend yang dijalankan sebagai pengguna Anda (yang dimulai dengan `chloros-backend`, atau [`chloros-backend@<user>`](#menjalankan-backend-sebagai-layanan-tanpa-antarmuka-grafis)) tidak dapat mengubah pengaturan kernel, dan mencetak baris `sudo sysctl` sebagai gantinya.

**Penyesuaian adaptor otomatis hanya berlaku untuk adaptor yang terhubung.** Saat kamera terhubung untuk pertama kali, backend yang berjalan sebagai root akan menyesuaikan adaptor kabel yang memiliki ukuran ring penerima kecil. Kini, penyesuaian hanya dilakukan pada adaptor yang terhubung, sehingga port onboard yang dicabut tetap mempertahankan MTU-nya, dan buffer penerima tidak diubah. `lattice network` tidak lagi mencantumkan MTU adaptor yang dicabutsebagai masalah. `lattice network`, `--fix`, `lattice probe`, dan penyesuaian adaptor hanya mempertimbangkan port Ethernet kabel, bukan bus CAN Jetson (`can0`) atau tautan mode perangkat USB (`usb0`, `usb1`) yang ditawarkannya kepada PC host; adaptor Ethernet USB untuk kamera masih dihitung.

### Dasar-dasar jaringan (semua tautan LATTICE)

- **Penomoran alamat:** link-local `169.254.0.0/16` (GigE Vision LLA). Komputer host menggunakan alamat statis `169.254.x.x/16`; kamera dan DAQ-E menetapkan alamatnya sendiri dalam rentang yang sama. Tidak diperlukan DHCP/gateway.
- **Ukuran paket:**lebih disukai jumbo (9000), tetapi biarkan proses auto-probe menemukannya — proses ini mengabaikan batas ICMP 1500 byte kamera melalui probe GVSP, sehingga akan menggunakan ukuran jumbo di mana pun kabel tersebut benar-benar mendukungnya.**Setiap kamera diukur sekali per sesi backend, dan diukur lagi saat koneksinya berubah (adaptor lain, kecepatan koneksi berbeda, atau koneksi terputus, seperti saat mengganti switch); koneksi ulang akan menggunakan hasil yang sudah ada. Setelah terjadi perubahan di belakang switch yang tetap aktif, restart backend untuk mengukur ulang.**Koneksi pertama terkadang membutuhkan waktu beberapa menit pada pemeriksaan ini, dan log akan mencatatnya jika hal itu terjadi. Tetapkan nilai tetap (pin) dengan kode kesalahan `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` hanya jika Anda lebih tahu daripada alat penguji, dan utamakan pengaturan per-perintah daripada pengaturan permanen: pengaturan tetap akan melewati alat penguji, sehingga jika jalur tersebut sebenarnya tidak dapat mentransmisikan 9000,**setiap** penangkapan akan mengalami timeout dengan `SC_ERR_TIMEOUT -1011` (lihat [Variabel Lingkungan](#environment-variables)).
- **Cincin RX berskala dengan `ReceiveBufferLen`:**pada nilai default `32`, ring yang dapat digunakan adalah ~0,26 MB (terlalu kecil untuk burst multi-kamera apa pun); pada nilai maksimum `256`, ukurannya besar (~13,5 MB yang diukur pada host 10GbE di lab), sehingga memberikan ruang cadangan yang nyata. Apakah suatu konfigurasi dapat terhubung kemudian ditentukan oleh pemeriksaan penerimaan yang menyadari beban**dan** pemeriksaan kelebihan langganan agregat di bawah ini — bukan perbandingan mentah antara burst dan ring.

### Model fps & burst Array

Cara membaca panel Pengaturan Array (dan `lattice analyze-array` / `analyze_array_network` di SDK):

- **Burst dijumlahkan per kamera pada format piksel asli masing-masing kamera.**Kamera Mono**M3M**mengalirkan**Mono12 (2 B/px)**, atau dengan `--pixel-format`, format Mono dari kedalaman kameranya (`BayerRG8` adalah `Mono8`, 1 B/px); kamera**M3C**Bayer mengalirkan data 8- atau 12-bit (firmware M3C secara diam-diam mengeluarkan BayerRG12 bahkan ketika diminta BayerRG8). Jadi, satu bingkai resolusi penuh dari 4 kamera berukuran**~12,6 MB jika semuanya 8-bit, tetapi ~25 MB dengan tiga kamera mono 12-bit**. Proyeksi ini menentukan format masing-masing kamera berdasarkan modelnya (cache identitas), sehingga data burst sesuai dengan apa yang sebenarnya dikirimkan melalui kabel — bukan asumsi BayerRG8 yang seragam.
- **Adaptor Ethernet USB dibatasi hingga 200 MB/s terlepas dari spesifikasi yang tertera.** Tabel efisiensi yang mengubah laju tautan menjadi angka berkelanjutan berasal dari PCIe; sebuah NIC USB mengiklankan laju tautan *Ethernet*-nya tetapi dibatasi oleh bus USB dan drivernya. Sebuah dongle USB 10GbE pernah mencapai ~1.063 MB/s "berkelanjutan" — angka yang tidak pernah diuji — dan pengaturan kecepatan yang dihasilkan merusak 6–18 % frame meskipun masih melaporkan fps target yang baik. NIC yang terhubung via USB kini dibatasi secara mutlak: **200 MB/s** di Windows dan di NVIDIA Jetson, **400 MB/s** pada Linux x86_64 (pada Linux, adaptor yang terpasang di port USB 2.0 dialokasikan **30 MB/s**, dan catatan spesifikasinya menyarankan untuk mencolokkannya ke port USB 3) (batasnya ditentukan oleh bus USB dan driver host, sehingga tidak sebanding dengan spesifikasi tertera; adaptor USB 1 GbE menghasilkan ~80 MB/s dan tidak terpengaruh). Di Linux, adaptor diidentifikasi melalui sysfs (jika tidak ditemukan, akan menggunakan nama alternatif `ethtool -i` atau `bus-info`). Setiap angka diukur: di Windows, susunan 4 kamera yang sama mengalami kerusakan pada 6–10 % frame di atas 200 MB/s; di Linux x86_64, adaptor yang sama berjalan lancar pada setiap batas atas hingga 480 MB/s (273 MB/s yang dihasilkan, batas dari kamera itu sendiri). Pada NVIDIA Jetson, adaptor yang sama hanya beroperasi tanpa gangguan pada 200 MB/s dan kehilangan frame secara sporadis di atas kecepatan tersebut. `wire_ceiling_source` pada catatan kemampuan menyatakan hal ini secara tertulis, dan `nic_is_usb` menandainya. Ganti pengaturan ini dengan `--wire-ceiling-mbps`.
- **Admittance memperhatikan beban drain, bukan perbandingan antara burst utuh dan ring.** Sebuah burst simultan hanya perlu sesuai dengan *backlog sementara* = `max(0, Σ per-cam arrival − host drain) × emit_window`, bukan seluruh burst. Pada jaringan host cepat / kamera lambat (sebuah **PCIe**host 10G + 4× cam 1 GbE: kedatangan ≈ 320 MB/s, pengosongan ≈ 1063 MB/s), host mengosongkan lebih cepat daripada kamera mengisi, backlog ≈ 0, sehingga**sim-emit**resolusi penuh**diterima**meskipun burst 25 MB melebihi ring 13,5 MB. Letakkan keempat kamera yang sama di belakang adaptor**USB** 10GbE di Windows, dan laju pengosongan menjadi 200 MB/s, bukan 1063 — laju kedatangan melampaui laju pengosongan, dan kerugian tersebut muncul sebagai frame yang rusak, bukan sebagai penurunan laju frame. Pada host 1 GbE, batas bawah DLThr kamera sebesar 31,25 MB/s membuat kedatangan melebihi laju pembuangan → sistem dengan benar **memblokir** (untuk *kelas* pemblokiran ini, kurangi ROI atau gunakan binning ≥ 2). Admittance adalah salah satu dari **dua** gerbang koneksi — yang lainnya adalah pemeriksaan kelebihan langganan agregat di bawah ini.
- **FPS yang diproyeksikan adalah siklus kamera yang paling lambat.**Dalam mode emit simultan, setiap kamera mengirimkan framenya pada waktu yang sama, sehingga array berputar sesuai dengan waktu yang lebih lama antara waktu pembacaan sensor dan waktu emit dari anggota yang paling lambat (byte frame ÷ laju emitnya:**link akses** kamera, 1 GbE ≈ 80 MB/s, atau porsinya dari batas maksimum koneksi host jika nilainya lebih kecil). Setiap eksposur tumpang tindih dengan pembacaan frame sebelumnya, sehingga eksposur tidak masuk ke dalam siklus: pengaturan eksposur otomatis mengisi periode yang ditentukan oleh laju tersebut, kemudian menaikkan gain, dan pemandangan yang redup tidak lagi menurunkan laju (pada versi 1.2.0 hal ini terjadi). Hanya eksposur manual yang lebih lama dari periode tersebut yang memperlambat pemicu (lihat [Eksposur pada anggota array](#exposure-on-an-array-member)). Menambahkan kamera tidak akan memperlambat array hingga porsi jalurnya berkurang. Dalam mode emisi serial (`slip-emit-and-capture`), waktu emisi kamera justru dijumlahkan, dan eksposur setiap anggota diperhitungkan dalam stagger. Diukur pada sensor penuh, 12-bit, pada array 9 kamera dengan host PCIe 10 GbE: 10,7 fps, atau 14,0 fps dengan [transportasi terkompresi](#packed-12-bit-transport---packed).
- **Setelah perekaman berkelanjutan, proyeksi menunjukkan nilai yang dicapainya.** Perekaman selama 30 detik atau lebih pada laju otomatis mencatat laju yang stabil pada ukuran susunan kamera tersebut di komputer ini; uji coba singkat pengontrol laju pada laju yang lebih cepat diabaikan. Sejak saat itu, baris `array-connect` dan panel Pengaturan Array mencantumkan laju tersebut jika nilainya di bawah laju model, beserta faktor yang membatasinya: `clock-bound` (array berjalan pada laju yang telah stabil, loop pengambilan menunggu clock pemicunya), `loop-bound` (loop pengambilan kehabisan waktu), `capture-bound` (proses penangkapan tertinggal dan melewatkan beberapa kelompok) atau `measured` (tidak ada yang menyebutkan mana yang terjadi). Angka modelnya terdapat pada `wire_fps_max`. Array tersebut tetap naik hingga batas yang didukung oleh jaringan.
- **Sebuah array dapat beroperasi di jaringan apa pun — laju bingkai lah yang menentukan.**Setiap kamera mendapatkan porsi yang merata dari**batas maksimum saluran yang aman dari tabrakan**(`sustained × sim_emit_factor`), disinkronkan dengan GevSCPD, dan semuanya tetap memicu pada sinyal pemicu yang sama. Pada kabel yang lambat, porsinya kecil dan laju bingkai pun ikut menurun: 8 kamera resolusi penuh 12-bit pada 1 GbE tanpa jumbo frame masing-masing mendapatkan 52 / 8 = 6,5 MB/s, sekitar**1 fps**(sekitar 1,3 fps jika dikompresi), sedangkan host 10G menjalankannya pada 11–14 fps. Jika kamera tidak dapat disinkronkan sepelan itu — lakukan pembacaan rentang GevSCPD setiap kamera untuk mengetahuinya — array akan beralih ke mode**serial**(`slip-emit-and-capture`): kamera-kamera tersebut bergiliran di saluran dengan kecepatan tautan penuh, dipicu satu demi satu, sehingga setiap frame tetap tiba tetapi anggotanya**tidak terpapar secara bersamaan**. `analyze-array` dan output perintah `connect` menunjukkan mode tersebut; atur `--force-tier sim-capture-sim-emit` ke `refuse` untuk menolak mode cadangan. Ukuran frame tidak memengaruhi pembagian ini, sehingga**penggabungan (binning) dan ROI yang lebih kecil meningkatkan laju frame tetapi tidak mengubah apakah sebuah array muat**. Hanya kabel yang tidak dapat mentransmisikan array bahkan secara serial yang**kelebihan beban**— misalnya, jumlah kamera jauh lebih banyak daripada yang dapat ditangani oleh satu kabel, atau banyak kamera lain yang melakukan streaming di atasnya — dan di situ gejalanya adalah kehilangan paket GVSP, bukan kecepatan yang lebih rendah: `analyze-array` mengosongkan angka fps yang dapat dicapai dan menampilkan `** OVER-SUBSCRIBED**`, koneksi tetap dilanjutkan dengan peringatan keras, dan koneksi HTTP dengan `pin_resolution: true` (mode ketat) **menolak**. Solusinya adalah mengurangi jumlah kamera, menggunakan jumbo frames dari ujung ke ujung, atau menggunakan NIC yang lebih cepat. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` menurunkan status penolakan menjadi peringatan keras untuk pengujian — lihat [Variabel Lingkungan](#environment-variables).

### Kesehatan array — subsistem mana yang kehilangan frame

`GET /api/camera/array/<array_id>/capability` pada array yang terhubung menampilkan blok
`health` secara real-time, yang dievaluasi ulang dalam jendela bergulir **10 detik**. Blok ini membagi kehilangan frame
menjadi dua penyebab yang memerlukan perbaikan berlawanan, alih-alih melaporkan satu tingkat "tidak lengkap"
yang tidak menyebutkan keduanya:

| Kolom | Artinya | Subsistem mana |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per serial) | Frame **telah tiba namun secara struktural rusak**— kehilangan paket GVSP. |**Jaringan**: alokasi bandwidth, pengaturan kecepatan, cincin penerima NIC, MTU |
| `never_arrived_rate_pct` (per serial) | Frame **sama sekali tidak pernah tiba**— kamera tidak memicu, atau tidak ada data yang dikirim. |**Pemicu / sinkronisasi**: kabel M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Tingkat terburuk setiap kamera untuk masing-masing penyebab. | — |
| `per_cam_rate_pct` | Tingkat ketidaklengkapan gabungan per kamera (kedua penyebab tersebut digabungkan). | — |
| `stable_for_seconds` | Berapa lama setiap kamera berada di bawah 0,01 %. | — |

Di atas 5%, backend mencatat baris `[array-health <id>] WARN` yang menyebutkan pemisahan — pada
pelanggaran pertama, saat terjadi perubahan tingkat keparahan, sekali setiap menit selama kondisi tersebut berlanjut, dan sekali saat
kondisi tersebut teratasi. Bagian yang rusak menulis baris `[gvsp-corrupt <SN>]` ke berkas log backend pada
kejadian pertama per kamera dan alasan, kemudian ringkasan setiap 60 detik; panel log aplikasi menampilkan baris
baris `[array-health <id>]`. Setiap evaluasi tetap terekam dalam berkas log backend;
penghitung terus bertambah pada setiap buffer terlepas dari apa yang dicetak.

Catatan yang sama melaporkan angka yang menjadi acuan seluruh alokasi:

| Kolom | Artinya |
| --- | --- |
| `wire_ceiling_mbps` | Anggaran bandwidth yang berlaku secara berkelanjutan pada host, MB/s. |
| `wire_ceiling_source` | Asal usul angka tersebut, dalam bentuk teks — misalnya `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (pada Linux x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) atau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` ketika `--wire-ceiling-mbps` (atau bidang **Wire Budget** pada antarmuka grafis) menetapkannya. |
| `nic_is_usb` | `true` untuk adaptor Ethernet USB — lihat batasan USB di atas. |

**Penafsiran:** nilai `gvsp_corrupt_rate_pct` yang tidak nol dengan `never_arrived_rate_pct` bernilai 0
berarti pemicu dan sinkronisasi kabel sempurna, dan 100% kehilangan terjadi di jalur jaringan
— kurangi nilai `--wire-ceiling-mbps` dan sambungkan ulang. Pola sebaliknya menandakan masalah pada
kabel sinkronisasi atau jalur pemicu.

> **`--target-fps` bukanlah faktor penentu untuk frame yang rusak.** Pacing GevSCPD ditulis
> sekali saat koneksi, sehingga menurunkan laju pemicu mengubah siklus kerja dan bukan
> laju burst emisi simultan. Pengurangan permintaan sebesar 5× yang diukur tidak menghasilkan perbaikan;
> menurunkan batas maksimum kabel dari 240 menjadi 200 MB/s menurunkan tingkat kerusakan pada rig yang sama dari 10,4%
> menjadi 0,00%.

> **Sebuah array yang sedang berjalan tidak dapat merencanakan ulang dirinya sendiri pada firmware LATTICE (M3C / M3M).** Array tersebut
> tidak dapat memperbaiki masalah ini sendiri; putuskan sambungan dan sambungkan kembali agar pemilih waktu sambungan dapat
> merencanakan ulang dengan batas atas yang baru.

### Gejala → solusi

| Gejala (Pengaturan Array / koneksi / `analyze_array_network`) | Penyebab | Solusi |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` diatur ulang ke 32 (biasanya setelah pembaruan driver) | Atur `ReceiveBufferLen`→256, `PendingReceives`→64; buka kembali panel (mulai ulang backend jika telah menyimpan ukuran ring lama dalam cache) |
| Proses restart/shutdown macet; kemudian muncul kesalahan WMI `Invalid class`, NIC tidak dapat diaktifkan, drive USB hilang | Driver Realtek USB 10GbE versi lama tahun 2016 → BSOD `0x9F` → sistem mati paksa | Perbarui driver adaptor ke v10.67 atau lebih tinggi (2026), lalu terapkan kembali pengaturan receive-ring di atas |
| Array terhubung pada sensor penuh tetapi memiliki laju bingkai rendah | Mode Otomatis mempertahankan sensor penuh dan membiarkan laju bingkai turun sesuai kapasitas kabel | Pilih salah satu opsi yang lebih cepat dari daftar output koneksi (`--binning 2` / `4`, atau `--width`/`--height`), aktifkan jumbo frame, atau tingkatkan koneksi |
| Array melaporkan target fps yang normal tetapi hanya menghasilkan sebagian kecil darinya; `health.gvsp_corrupt_rate_pct` tidak nol, `never_arrived_rate_pct` 0 | Anggaran bandwidth yang diperkirakan pada host melebihi kapasitas yang sebenarnya dapat dipertahankan (umum terjadi pada adaptor Ethernet USB, jalur PCIe yang sempit, atau jaringan bersama) | Sambungkan kembali dengan nilai `--wire-ceiling-mbps` yang lebih rendah dan periksa kembali blok kesehatan. **Bukan** `--target-fps` — Pacing GevSCPD sudah ditetapkan saat koneksi |
| Linux: sebuah array kehilangan kelompok frame dan menurunkan laju bingkainya sementara NIC melaporkan tidak ada frame yang terlewat; `lattice network` mencantumkan `socket receive buffers too small` | Buffer penerimaan soket kernel berada pada nilai default, sehingga paket kamera terbuang di kernel | Naikkan nilai `net.core.rmem_max` / `rmem_default` menjadi 32 MB — lihat [host Linux](#linux-hosts--socket-receive-buffers). Paket .deb melakukan ini saat instalasi |
| Kamera tidak muncul di grup yang dipublikasikan; nilai `health.never_arrived_rate_pct` bukan nol, `gvsp_corrupt_rate_pct` 0 | Jalur pemicu/sinkronisasi — kamera tidak aktif, bukan masalah jaringan | Periksa kabel sinkronisasi M8 dan `--line`; pastikan setiap anggota dalam status siaga (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` melebihi batas di `analyze-array`, atau penolakan koneksi dalam mode ketat (`pin_resolution`; `array over-subscribes the wire`) | Kabel tidak mampu menangani array pada kecepatan apa pun yang dicapai kamera, baik secara bersamaan maupun bergantian — jumlah kamera jauh melebihi kapasitas kabel untuk dibagi, atau ada kamera lain yang sedang streaming di atasnya | Kurangi jumlah kamera, gunakan jumbo frames end-to-end, atau gunakan NIC yang lebih cepat. **ROI/binning TIDAK AKAN membantu** (pembagian tidak bergantung pada ukuran frame). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` mengabaikan batasan pada pengujian (menerima kehilangan paket) |
| Array terhubung melalui `slip-emit-and-capture` (serial) pada jaringan lambat | Kamera-kamera tidak dapat diatur kecepatannya cukup lambat untuk berbagi kabel secara bersamaan, sehingga mereka bergantian; frame tidak terkirim pada saat yang sama | Frame jumbo atau NIC yang lebih cepat memulihkan perekaman simultan; `--force-tier sim-capture-sim-emit` tetap memaksakannya (dan malah kehilangan paket) |

---

## `chloros-cli daq`

Perintah sensor spektral. Perintah `pool-*` adalah klien HTTP ringan yang mengendalikan sensor melalui pool persisten backend, sehingga memerlukan backend yang sedang berjalan (lihat [Mulai Backend](#start-the-backend)). Backend mengelola transportasi, sehingga skrip CLI dan SDK berbagi satu handle aktif alih-alih berebut port serial. `pool-discover` adalah satu-satunya cara untuk mengetahui alamat MAC BLE DAQ-M.

> **`chloros-cli daq --help`** (atau `-h`, atau `daq help`) mencantumkan subperintah `pool-*` (di Linux, bersama dengan perintah perangkat keras langsung yang disertakan dalam build), dan `chloros-cli daq <subcommand> --help` (atau `daq help <subcommand>`) memberikan opsi masing-masing. Opsi-opsi tersebut mencakup koneksi, stream, merekam berkas `.daq` yang telah dikalibrasi, menukar profil kapasitor, dan menghubungkan DAQ ke kamera, serta untuk DAQ-E, pengaturan jaringannya, PTP, restart, dan pembaruan firmware. DAQ juga dapat dikendalikan dari Python dengan `chloros_sdk.connect_daq_sensor()`, yang menggunakan jalur terpusat yang sama.
>
> **Gunakan perintah `pool-*` di setiap platform.** Perintah-perintah ini berperilaku sama di Windows, Linux, dan Jetson. Di Windows, `daq discover` dan `daq list` merupakan alias dari `pool-discover`, serta perintah `daq` lainnya yang bukan `pool-*` akan keluar dari `1` dengan `DAQ direct-hardware commands require the full daq package`. Di Linux dan Jetson, `daq discover`, `daq list`, dan perintah `pool-*` serta `daq` lainnya yang bukan `pool-*` merupakan perintah langsung-perangkat keras: perintah-perintah ini membuka sensor dalam proses CLI itu sendiri, di luar kumpulan proses backend (sehingga bukan pada sensor yang dikelola oleh backend, GUI, atau skrip), dan di sana menerima perintah `daq discover`, `--only`, serta `--timeout`, tetapi tidak menerima perintah `--json`. Referensi ini hanya mendokumentasikan perintah `pool-*`.

### Alur Kerja Koneksi Pertama Sensor DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referensi `pool-*`

| Subperintah | Tujuan |
| --- | --- |
| `daq pool-connect` (smart-detect) | Membuka sensor di kumpulan backend. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U pada port serial tertentu. |
| `daq pool-connect --ble` | DAQ-M melalui BLE, MAC dipindai secara otomatis. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M melalui BLE pada alamat MAC yang diketahui (mencakup `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E melalui Ethernet pada nama host atau alamat IP yang diketahui (menyiratkan `--eth`). |
| `daq pool-connect --eth` | DAQ-E melalui Ethernet, host terdeteksi otomatis (mDNS + cadangan ARP; berfungsi dari cache ARP yang kosong di Windows dan Linux). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Sesuaikan jendela integrasi / status AE. |
| `daq pool-connect --no-stream` | Terhubung tetapi belum mulai streaming (dilanjutkan dengan `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Profil koreksi batas. Nilai default di backend adalah `sunshine_cosine`; `none` adalah sensor tanpa penutup. Sebuah [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) mengabaikannya (diffuser tetap). **Sensor dikirim dalam kondisi polos, jadi beri nama penutup yang Anda pasang:** nilai default dicatat sebagai (`cap_id_source=auto_default`) dan sistem pemrosesan akan memberikan peringatan, karena DAQ-U tanpa penutup di bawah koreksi sinar matahari akan menunjukkan pembacaan ~30 kali lebih tinggi. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Pindai setiap transportasi untuk mencari sensor yang dapat Anda hubungkan, tanpa benar-benar menghubungkannya. **Inilah cara menemukan alamat MAC BLE DAQ-M.** `--timeout` berlaku per protokol (default 5 detik, 1–20). Sensor yang sudah terbuka di pool tidak tercantum — DAQ-M yang terhubung berhenti mengiklankan alamatnya — jadi gunakan `pool-list` untuk sensor-sensor tersebut. `usb` hanya mencantumkan perangkat serial USB: port serial bawaan komputer (`COM1`, `/dev/ttyS*` pada Linux) tidak dapat berfungsi sebagai sensor dan diabaikan. |
| `daq pool-list` | Menampilkan semua sensor di kolam backend. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Mengikat kamera atau array ke DAQ, seperti yang dilakukan oleh pemilihan sensor cahaya di antarmuka pengguna: penangkapan gambarnya menggunakan, mencocokkan, dan menyimpan pembacaan DAQ tersebut. `SENSOR` adalah ID sensor dari `pool-list` atau alamat tempat DAQ tersebut dapat dijangkau. `--clear` (tidak ada `SENSOR`) akan memutuskan pengikatan, dan proses perekaman akan beralih ke DAQ terbaru. Pengikatan ini berlangsung hingga backend dimulai ulang, atau hingga kamera, array, atau DAQ tersebut terputus: pemutusan koneksi (di sini, dari SDK, atau di dalam aplikasi) akan memutus semua pengikatan perangkat keras yang terputus, terlepas dari siapa yang mengaturnya; anggota array akan terputus bersamaan dengan array tersebut. Kamera harus terhubung, berada dalam proyek yang terbuka, atau pernah dibuka sebelumnya di komputer ini, dan array tersebut harus terhubung atau berada dalam proyek yang terbuka; serial atau ID lain apa pun akan ditolak berdasarkan namanya dan tidak ada yang terikat. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Pelepasan — siapa pun yang membuka sensor, sehingga juga membebaskan sensor yang ditinggalkan oleh skrip yang crash di dalam pool (perekaman apa pun di atasnya diselesaikan terlebih dahulu). Sensor yang tidak ada di dalam pool tidak memiliki apa pun untuk dilepaskan: ia mencetak `ID: already released (not in the pool)` dan keluar dengan kode `0`; pelepasan yang ditolak atau tidak dikonfirmasi oleh backend akan keluar dengan kode `1`. SDK: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N frame spektrum terbaru (default 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Lanjutkan / jeda streaming. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Mulai / hentikan perekaman .daq. `-o` secara default diatur ke `~/Documents/DAQ Live View` di komputer backend (jalur relatif berada di bawah folder tempat Anda menjalankan perintah); `--duration` (`-d`, default `0` = hingga `--stop`). `--confirm-after S` akan memberikan peringatan jika tidak ada pembacaan yang masuk dalam waktu S detik setelah dimulainya (default 20; 0 menonaktifkannya); dengan `--duration`, `--heartbeat S` mencetak jumlah pembacaan setiap S detik dan memberikan peringatan saat jumlahnya berhenti bertambah (default 30; 0 menonaktifkannya). Perekaman pada DAQ-E yang berhenti mengirim data akan dilanjutkan dalam berkas yang sama setelah backend terhubung kembali, dan `--stop` mencantumkan celah-celah tersebut ([DAQ-E yang Berhenti Mengirim Data](#a-daq-e-that-stops-delivering)). Ctrl+C menghentikan perekaman `--duration` lebih awal dan melaporkannya; tanpa `--duration`, perekaman tetap berjalan setelah Ctrl+C (keluar dari `130`) hingga `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Menukar profil koreksi kapasitor saat runtime. `--cap-id` diperlukan; `--cap-id none` menghapus koreksi kapasitor (sensor tanpa penutup). DAQ-E-S mempertahankan pengaturan `as_recorded` apa pun yang ditetapkan. |
| `daq pool-enable-ptp --sensor-id ID` | Aktifkan kembali PTP pada DAQ-E yang PTP-nya dalam keadaan mati. Pengaturan ini disimpan di unit, sehingga unit yang dimatikan tetap mati — cap waktu tidak disinkronkan dengan PTP — hingga perintah ini dijalankan. `pool-list` menandai unit tersebut (`!! PTP is switched OFF`) dan menampilkan perintah ini untuknya. Chloros tidak pernah mengaktifkan kembali unit secara otomatis, karena status mati bisa jadi disengaja. |
| `daq pool-info --sensor-id ID [--json]` | Pengaturan DAQ-E itu sendiri: firmware-nya (dan apakah rilis Chloros ini membawa versi yang lebih baru), alamatnya dan bagaimana alamat tersebut diperoleh (DHCP / link-local / statis), pengaturan jaringan yang disimpan hingga sensor menjalankannya (satu yang disimpan dengan `--no-reboot` disertakan), aliran data saat dinyalakan, dan PTP. Pada sensor yang alirannya tidak sampai ke komputer ini, sistem akan membaca sensor tersebut pada koneksi baru dan memberitahukannya. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Atur DAQ-E ke DHCP (`--auto`) atau alamat statis (`--mask` default `255.255.255.0`). Sensor akan restart untuk menerapkan pengaturan tersebut dan terhubung kembali dengan pengaturan sebelumnya, menunggu hingga `--timeout` detik (default 45) untuk proses tersebut; `--no-reboot` hanya menyimpannya. Lihat [Pengaturan Perangkat DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Apakah DAQ-E mulai melakukan streaming saat dinyalakan tanpa perangkat lunak yang terhubung (disimpan di sensor). Chloros akan memulai streaming secara otomatis dalam kedua kasus. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Mulai ulang DAQ-E dan sambungkan kembali dengan pengaturan sebelumnya (menunggu hingga `--timeout`, default 45 detik). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Mem-flash firmware DAQ-E yang disertakan dalam rilis Chloros ini ke sensor yang menjalankan versi lama, lalu menghubungkannya kembali (sekitar satu menit; jangan matikan daya; `--timeout` default 45 detik hingga kembali aktif). Sensor yang sudah menggunakan versi terbaru dibiarkan apa adanya. |

### DAQ-E yang Berhenti Mengirim Data

DAQ-E di kolam yang tidak mengirimkan spektrum selama 20 detik akan dihubungkan kembali oleh backend. Mematikan dan menghidupkan kembali sakelar yang menyalakan sensor akan melakukan hal ini: sensor akan restart bersamanya dan tidak akan mengirimkan data lagi sampai diperintahkan (kecuali jika `pool-set-auto-stream --on`).

- **Pertama-tama, sistem memeriksa apakah itu adalah sensor yang sama.** Sistem mengirim permintaan ke alamat yang sebelumnya dimiliki sensor, tanpa mengubah apa pun, untuk mengetahui nama, MAC, dan nomor seri sensor. Sensor ini juga mengirim permintaan menggunakan nama `daq-e-<id>.local` milik sensor tersebut, tetapi hanya mempercayai jawaban tersebut jika berasal dari adaptor jaringan (MAC) milik sensor itu sendiri. Respons lain apa pun tidak akan dihubungkan kembali atau ditulis ke dalamnya.
- **Sistem ini menghubungkan kembali dengan pengaturan sesi:** waktu integrasi, rata-rata bingkai, eksposur otomatis, dan profil cap tetap seperti semula, bukan pengaturan default saat sensor dinyalakan. Sensor yang sebelumnya mengaktifkan PTP dan kembali dalam keadaan nonaktif akan diaktifkan kembali. Kemudian, sistem ini memulai ulang aliran data dan penerima di komputer untuk sensor tersebut.
- **Sistem akan mencoba kembali dengan interval yang semakin lama:** segera, kemudian setelah 5, 10, 20, dan 40 detik, lalu setiap 60 detik selama sensor tidak terdeteksi. Setiap percobaan hanya menanyakan identitas sensor tersebut. Sensor yang merespons tetapi tetap tidak mengirimkan apa pun setelah tiga kali upaya koneksi ulang akan dibiarkan dan dilaporkan (`auto-reconnect GAVE UP`). `pool-stream --start` akan mencoba lagi.
- **`pool-list` menunjukkan statusnya.** Selama sensor tidak merespons, baris `STREAM FAULT`-nya menjelaskan apa yang sedang dilakukan proses koneksi ulang. Baris tersebut hilang pada spektrum pertama, dan sensor kemudian menampilkan `reconnected automatically N time(s)`.
- **Perekaman berlanjut pada baris yang sama `.daq`.** Berkas tersebut menandai setiap jeda dengan dua baris: `event_type` 5 pada spektrum terakhir sebelum jeda dan 6 pada spektrum pertama setelahnya, masing-masing hanya disertai cap waktu. `pool-record --duration` mencetak `!! STREAM LOST …` saat sensor tidak terdeteksi dan `stream resumed after …` saat sensor kembali terdeteksi. `pool-record --stop` mencantumkan setiap celah dan memberikan peringatan, seperti yang dilakukannya untuk file kosong (perekaman yang tidak menyimpan pembacaan akan keluar sebagai `1`).
- **Sebuah perekaman mempertahankan satu kalibrasi.** Jika sensor kembali dengan kalibrasi yang berbeda saat perekaman sedang berjalan, sisa perekaman akan tetap menggunakan kalibrasi yang digunakan saat dimulai, dan peringatan penghentian akan memberitahukannya. `pool-disconnect` + `pool-connect` akan menggunakan kalibrasi yang baru.
- **Perintah Anda yang berlaku.** Sensor yang Anda hentikan dengan `pool-disconnect`, atau jeda dengan `pool-stream --stop`, tidak akan pernah terhubung kembali, bahkan jika proses penyambungan ulang sudah berlangsung.
- **`pool-info` pada sensor yang tidak merespons** membaca sensor pada koneksi baru dan menambahkan `!! stream NOT reaching this computer`. Sensor yang tidak merespons akan segera dilaporkan. `pool-list` tidak lagi menampilkan status PTP untuk sensor yang telah berhenti merespons.
- DAQ-U dan DAQ-M tidak tercakup. DAQ-M akan terhubung kembali melalui Bluetooth secara otomatis, sedangkan DAQ-U yang dicabut dari USB memerlukan `pool-connect` lagi.

### DAQ-E-S — difuser tetap dan sensor kemiringan

**DAQ-E-S** adalah DAQ-E dengan dua perbedaan yang berpengaruh pada perangkat lunak. Selebihnya — penemuan, koneksi, PTP, perekaman, serta pengaturan perangkat di bawah ini — berfungsi persis seperti pada DAQ-E.**Diffuser-nya bersifat permanen, sehingga tidak memerlukan koreksi penutup.** Diffuser bergaya Sunshine sudah terpasang di dalam unit dan sudah ada saat kalibrasi pabrik diukur, sehingga kalibrasi tersebut sudah menunjukkan irradiansi turun yang sebenarnya. Chloros mengatur penutupnya ke `as_recorded` (tanpa profil) apa pun yang diminta: batas yang dinamai khusus untuknya (`sunshine_cosine`, `fov_*`, `none`) akan diabaikan disertai pesan peringatan, dan rekamannya menyimpan nilai `cap_id_source=model`, sehingga peringatan batas yang diasumsikan tidak pernah muncul untuknya. Jangan juga menerapkan batas pada datanya secara manual: `sunshine_cosine` di bagian atas akan menunjukkan nilai sekitar 11 kali lipat.

**Perangkat ini dilengkapi dengan akselerometer 3-sumbu.** Respon kosinus penyebar sinar matahari hanya berlaku saat perangkat dalam posisi horizontal, sehingga sensor yang miring melaporkan irradiansi yang salah — dan reflektansi mewarisi kesalahan tersebut — tanpa gejala lain. Oleh karena itu, setiap pembacaan DAQ-E-S menyertakan orientasinya:

| Kolom | Arti |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Sudut dalam derajat. `None` ketika firmware menandai sudut tersebut tidak terdefinisi (jatuh bebas, sumbu mati). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Akselerasi dalam mili-g. Nilai `mag_mg` yang jauh dari ~1000 berarti unit sedang bergerak dan sudut-sudut tersebut tidak bermakna untuk sampel tersebut. |
| `cal_applied` | Set: sumbu dan sudut berada dalam kerangka acuan diffuser (kalibrasi pemasangan unit telah diterapkan). Clear: kerangka acuan papan mentah, yang dapat menunjukkan kemiringan hingga ~2,3° yang sebenarnya tidak nyata. |
| `sample_age_ms` | Usia sampel akselerometer saat spektrum dikirim. |
| `temp_raw` | **Tren** suhu die yang belum dikalibrasi — hanya perbedaan, bukan pembacaan termometer. |

`daq pool-latest --json` mengembalikannya di bawah kunci `imu` pada setiap bingkai (`null` pada DAQ-U, DAQ-M, atau DAQ-E biasa). Data `.daq` yang direkam dari DAQ-E-S menyimpan nilai yang sama di kolom `imu_*`-nya (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) di samping setiap spektrum, sehingga pembaca dapat membedakan antara awan dan tripod yang terguncang. Pada semua model lainnya, kolom-kolom tersebut kosong.

Aliran data orientasi dilaporkan secara terpisah: `daq pool-list` mencetak `!! ATTITUDE STREAM FAULT (spectra unaffected)` saat unit mati, serta berapa banyak kesalahan yang berhasil dipulihkan setelahnya. Kesalahan di sana tidak pernah menyebabkan celah dalam perekaman spektrum. `daq pool-discover` mencantumkan unit tersebut sebagai `DAQ-E-S` (sebagai `DAQ-E` jika unit tersebut tidak merespons pemeriksaan identitas dalam waktu sekitar 2 detik).

### Pengaturan Perangkat DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **`--static` tanpa `--gateway` tidak menyimpan gateway apa pun** (gateway apa pun yang ditetapkan sebelumnya akan dihapus), yang sesuai untuk jaringan kamera tanpa router.
- **Alamat statis harus berada di jaringan komputer ini.** Chloros akan menolak alamat yang subnet-nya tidak mencakup alamat komputer ini di jaringan sensor: setelah restart, tidak ada yang dapat menjangkau sensor untuk mengembalikannya ke pengaturan semula. Gunakan `--force` hanya jika komputer ini akan dialihkan alamatnya, atau diarahkan, ke subnet baru.
- **Restart ditolak saat sensor sedang merekam.** Hentikan perekaman terlebih dahulu (`pool-record --stop`). Perekaman yang dimulai sesaat sebelum restart akan ditutup oleh restart tersebut, dan `pool-set-network`, `pool-reboot`, serta `pool-update-firmware` diakhiri dengan baris `Warning:` yang mencantumkan nama berkas perekaman (bahkan ketika sensor tersebut kemudian tidak kembali).
- **Setelah restart, Chloros hanya menghubungkan kembali sensor itu sendiri.** Ia menunggu di lokasi sensor tersebut berada — dan setelah `--auto` juga di bawah nama `daq-e-<id>.local` — untuk jawaban yang berisi nama sensor, nama host, dan modelnya, dari adaptor jaringan (MAC) yang dimilikinya, dan menghubungkannya kembali ke alamat yang merespons. Respons dengan nama tersebut hanya diterima jika berasal dari MAC yang sama, sehingga dari komputer yang mengakses sensor melalui router, atau dari WSL2 dalam konfigurasi NAT default-nya, sensor yang dipindahkan ke alamat baru oleh DHCP tidak akan terhubung kembali: perintah tersebut akan memberitahukannya dan menampilkan perintah `pool-connect --eth-host <its address> …` yang mengembalikannya dengan pengaturan cap dan eksposur yang sebelumnya dimilikinya.
- **`pool-update-firmware` menolak sensor yang versi firmware-nya tidak dapat dibaca.** Sensor yang membutuhkan waktu lebih lama dari batas tunggu untuk kembali setelah flash tidak dianggap gagal: begitu sensor merespons, koneksi akan dipulihkan dengan pengaturan firmware baru yang terisi, dan perintah tersebut akan memperingatkan bahwa responsnya lambat.
- **Pembaruan firmware memerlukan koneksi TCP masuk.** Sensor terhubung kembali ke komputer ini untuk mengumpulkan gambar. Chloros mendengarkan pada adaptor yang mengarahkan ke sensor (pada setiap adaptor hanya jika tidak dapat menentukan yang mana) dan hanya melayani alamat sensor itu sendiri; koneksi lain apa pun akan ditutup tanpa dilayani. Di Windows, aturan firewall `Chloros Backend (TCP-In)` dari penginstal mengizinkannya masuk (lihat [Windows (.exe)](#windows-exe)); firewall yang memblokirnya akan menyebabkan pembaruan gagal dengan kode kesalahan `… device never connected back on TCP <port> …`.
- **Di bawah WSL2, gunakan jaringan cermin** (`networkingMode=mirrored` dalam `%UserProfile%\.wslconfig`). Dalam mode NAT default, alamat komputer ini di jaringan sensor adalah alamat WSL itu sendiri, sehingga setiap alamat statis ditolak (kecuali `--force`), siaran langsung tidak sampai ke WSL, dan pembaruan firmware tidak dapat diselesaikan, karena sensor terhubung kembali ke komputer untuk menerima gambar.
- **ID sensor dan nama host tidak berubah.** Apa pun alamatnya, DAQ-E tetap ber-ID `daq-e-<id>` dan `daq-e-<id>.local`, sehingga `--eth-host daq-e-<id>.local` tetap dapat menemukannya.
- **GUI memiliki pengaturan yang sama**: tab Sensor Cahaya, pengaturan sensor (ikon roda gigi), Jaringan. GUI mempertahankan koneksinya sendiri ke sensor, jadi ubah pengaturan sensor yang sedang terbuka di GUI melalui GUI: restart dari CLI akan mengakhiri sesi antarmuka pengguna (GUI) dengan sensor tersebut.
- Setara SDK-nya adalah `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()`, dan `update_firmware()`.

---

## `chloros-cli project`

Buka, sambungkan, dan jalankan proyek Chloros yang telah disimpan (folder yang berisi `cameras.json` + `sensors.json` + `project.json`). Semua proses diproses melalui backend sehingga antarmuka grafis (GUI) dan antarmuka baris perintah (CLI) menghasilkan status perangkat keras yang identik.

### Referensi Subperintah

| Subperintah | Tujuan |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Menampilkan daftar perangkat proyek (kamera, array, sensor), membaca dari berkas-berkasnya, dan memeriksa berkas sumbernya (lihat [Proyek yang dipindahkan](#a-moved-project---relink)). Perintah ini tidak membuka proyek di backend: proyek yang terbuka di sana, beserta perangkat dan rekamannya, tetap seperti semula. Keluar dari `1` jika tidak ada file sumber proyek yang dapat ditemukan. |
| `project devices PATH [--reconnect]` | Menampilkan daftar atau menjalankan ulang proses penemuan. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Menghubungkan setiap kamera / array / sensor yang tersimpan. Setiap array terhubung seperti yang dilakukan oleh `lattice array-connect`: hingga 600 detik, langkah-langkahnya dicetak ke stderr (`[array <id>]    <s> s: <step>`; stdout adalah laporan JSON), dan array yang tidak terhubung dalam waktu tersebut akan dibatalkan — kameranya dikembalikan ke kondisi semula — dan dilaporkan di bawah `errors`, tidak pernah dibiarkan tetap terhubung. `--packed` mengatur transportasi 12-bit terkompresi untuk setiap array (default `auto`; sebuah proyek tidak menyimpan pengaturan per-array; lihat [Transportasi 12-bit terkompresi](#packed-12-bit-transport---packed)). Setiap sensor cahaya dibuka di kolam DAQ backend pada alamat yang disimpan dengan waktu integrasi, rata-rata bingkai, eksposur otomatis, dan batas yang disimpan: sensor yang tidak memilih batas waktu akan mendapatkan pengaturan `sunshine_cosine`, sensor yang disimpan tanpa pengaturan (`none`) tetap tanpa pengaturan, dan DAQ-E-S tidak pernah menetapkan batas waktu. Panel DAQ pada GUI bukan bagian dari pool: tutup sensor di sana terlebih dahulu. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Pengambilan tunggal dari kamera atau array yang diberi nama (`--prefix` default `capture`). `--orient` (array): array menyimpan data mentah, yang tetap dalam format asli — ini mencatat bahwa produk yang dihasilkan `process` darinya harus diorientasikan, seperti `array-capture --orient`. Default: asli. `project burst` / `stream` dan resep `capture` / `burst` / `stream` (`orient: true`) menggunakan opsi yang sama. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Rangkaian N-frame dari kamera atau array yang ditentukan (`-n/--count` default 5; `-i/--interval` jeda antar-frame dalam detik, default 0). Burst array menghilangkan duplikat dari kelompok sinkron yang berulang (pengawas kedaluwarsa) sehingga array yang berputar sebagian tidak dapat mengembalikan N salinan dari satu frame; mencetak hasil per iterasi. Penangkapan yang gagal di tengah jalan akan menghentikan burst dan keluar dari `1`, sambil menunjukkan berapa banyak yang telah disimpan sebelumnya. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Penangkapan ke disk dalam loop: `--fps F` menangkap satu detik (default 3), satu bingkai per penangkapan dari kamera, satu kelompok bingkai yang disinkronkan dari array (dengan watchdog kedaluwarsa yang sama seperti `project burst`), hingga `-n N` menangkap (default `0` = hingga Ctrl+C, yang mencetak berapa banyak yang telah disimpan dan keluar dari `0`, atau `1` jika tidak ada yang disimpan atau terjadi kegagalan file; Ctrl+C sebelum `-n N` menangkap akan menutup `130`). Penangkapan yang gagal di tengah jalan akan menghentikan aliran dan keluar dari `1`, sambil menunjukkan berapa banyak frame yang telah disimpan sebelumnya. `--poll-interval` sudah tidak direkomendasikan dan diabaikan: tetapi masih diterima sehingga skrip lama tetap berjalan, dengan catatan di stderr. |
| `project sensor read PATH NAME [--json]` | Frame spektrum terbaru. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Merekam berkas .daq untuk `--seconds` (default 10). Perekaman yang tidak berisi pembacaan akan dilaporkan di stderr dan program akan keluar dari `1`. |
| `project run PATH RECIPE.yaml` | Jalankan resep penangkapan YAML/JSON. `--dry-run` melakukan validasi tanpa menjalankan proses. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Hitung penyelarasan untuk sebuah array — lihat [tabel opsi di bawah ini](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Menampilkan profil penyelarasan saat ini. |
| `project align clear PATH NAME` | Menghapus profil yang disimpan dalam cache. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Menggeser transformasi satu slave (nilai default `0`, `0`, `0`, `1.0`: tidak ada perubahan). |
| `project align export PATH NAME --to FILE` | Simpan profil ke JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Muat profil yang telah disimpan. |

#### Opsi `project align calibrate`

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Metode penyelarasan. **Ejaan ini berbeda dari `lattice align-calibrate`**, yang menggunakan bentuk singkat `orb` / `akaze` / `phase`; kedua perintah tersebut tidak dapat digunakan secara bergantian pada bendera ini. |
| `--model {translation, rigid, affine, homography}` | `affine` | Mengubah model agar pas. |
| `--frames N` | `1` | Menyinkronkan snapshot bingkai ke rata-rata. |
| `--reference SN` | master | Nomor seri kamera referensi; setiap anggota lainnya diwarp ke kamera tersebut. |
| `--max-features N` | `5000` | Batas jumlah fitur ORB. |
| `--ratio-threshold F` | `0.75` | Uji rasio Lowe'. |
| `--ransac-threshold-px F` | `3.0` | Ambang batas inlier RANSAC. |
| `--min-matches N` | `15` | **Batasan kualitas** — tolak hasil pemecahan jika jumlah kecocokan inlier di bawah batas ini. |
| `--max-reproj-err-px F` | `4.0` | **Batasan kualitas** — tolak hasil pemecahan jika melebihi nilai kesalahan reproyeksi RMS ini. |
| `--checkerboard RxC` | — | Geometri papan untuk `--method checkerboard`, misalnya `9x6`. |
| `--name PROFILE` | kosong | Nama profil yang disematkan dalam JSON yang disimpan. **Bukan nama array** — itu adalah posisi `NAME`. |

Dua gerbang kualitas inilah yang menjadi alasan mengapa kalibrasi dapat berhasil dalam pemecahan masalah namun tetap
menolak untuk disimpan: profil yang gagal pada salah satu gerbang tersebut akan secara diam-diam menyebabkan kesalahan registrasi pada setiap
perekaman selanjutnya, sehingga profil tersebut ditolak daripada disimpan.

### Proyek yang dipindahkan (`--relink`)

Nama file sumber dalam proyek — setiap gambar dan rekaman sensor cahaya —
ditentukan oleh jalur lengkapnya. Jika folder proyek, atau seluruh folder proyek,
dipindahkan ke drive lain, jalur-jalur tersebut masih mengacu ke lokasi lama.

Membuka proyek akan memperbaiki apa yang bisa diperbaiki secara otomatis: berkas yang sebelumnya berada di dalam folder proyek
akan dicari di folder tempat proyek tersebut berada sekarang, dan setiap jalur yang ditemukan di sana
akan diarahkan ulang dan disimpan (`project.json` terlebih dahulu dicadangkan sebagai
`project.json.before-relink-<date>-<time>`). Hal ini terjadi setiap kali dibuka — baik aplikasi,
`chloros-cli project open`, `chloros-cli process` ke dalam proyek yang sudah ada, SDK dan `chloros-mcp`. Jalur hanya akan diarahkan ulang ke berkas yang ada di sana.

File yang berada di luar folder proyek — sesi hub, impor Tambahkan Folder —
tidak dapat ditemukan dengan cara tersebut. Beri nama folder tempat file tersebut berada saat ini:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` mengambil folder tujuan berkas-berkas tersebut, atau folder mana pun di atasnya: setiap
folder berkas yang hilang dicari di bawahnya berdasarkan nama folder akhirannya
(`Hub Projects/<session>/raw/<serial>`, lalu `<session>/raw/<serial>`, ...), dan setiap
file dalam folder tersebut diambil dari satu tempat yang menemukan file terbanyak. `project open` mencetak
apa yang telah diarahkan ulang, berapa banyak file sumber yang ditemukan, dan folder tempat sisanya
berada. Beberapa masih hilang: peringatan, keluar dari `0` -- proyek terbuka dengan sisanya. Tidak ada yang
ditemukan: kesalahan saat menamai folder-folder tersebut dan `--relink`, keluar dari `1`; backend juga
menolak membuka proyek semacam itu untuk `process` atau `project devices --reconnect` (HTTP 409 `PROJECT_SOURCES_MISSING`).

### Contoh

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

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

`project run RECIPE.yaml` menerima berkas YAML atau JSON yang menjelaskan urutan tindakan:

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
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
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

Tindakan yang didukung: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` mengambil kunci `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (nama-nama lama seperti `exposure_time`, `exposure_auto`, `gain`, dan `gain_auto` diterima sebagai alias). Pengaturan eksposur, gain, atau target yang dilakukan dengan cara ini akan tetap menjadi milik Anda: kontrol eksposur otomatis Chloros (termasuk eksposur yang diukur oleh DAQ) tidak akan mengubahnya. Eksposur manual membiarkan gain tetap seperti semula: dalam mode otomatis, kamera terus menyesuaikan gain (di seluruh rentangnya) menuju kecerahan target hingga Anda mengatur `gain_db` atau `auto_gain: "Off"`. Perintah `auto_exposure: "Continuous"` selanjutnya mengembalikan pengaturan eksposur ke mode otomatis, sedangkan `target_brightness_auto: true` mengembalikan target yang telah ditetapkan ke target otomatis Chloros. Tindakan `burst` memerlukan `name` (diperlukan), `count` (default 5), `interval` (detik, default 0), `output`, `format`, dan `settings` (bentuk pengaturan per kamera yang sama dengan `apply`); burst array menggunakan watchdog grup yang baru disinkronkan yang sama seperti `project burst`.

Pada langkah `capture`, `burst`, atau langkah `stream`, `output` secara default diatur ke `output`. Nilai relatif `output` — yang sudah disertakan secara default — diselesaikan berdasarkan folder tempat Anda menjalankan `chloros-cli project run`, bukan folder berkas resep atau folder proyek (pada versi 1.2.0, berkas tersebut berada di folder instalasi backend).

Jalankan perintah berikut:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Setiap eksekusi akan diperiksa terlebih dahulu, berdasarkan proyek.**Setiap tindakan harus memiliki salah satu kata kerja di atas sebagai kuncinya — `- capture: {name: FrontLeft}`, bukan `- action: capture` atau `- type: capture` — dengan `name`, proyek tersebut memiliki `cameras.json` / `sensors.json`, kamera atau array untuk `apply` / `capture` / `burst` / `stream` dan sensor cahaya untuk `sensor`; sebuah `stream` memerlukan `count` > 0 dan sebuah `wait` memerlukan durasi. Resep dengan kesalahan semacam itu**tidak** menjalankan tindakan apa pun (`[recipe] error: …`, `[recipe] not run: …`, keluar dari `1`), bahkan dengan `stop_on_error: false`. Kunci yang tidak digunakan oleh suatu langkah adalah `[recipe] warning:`, begitu pula setiap array atau sensor cahaya yang dikendalikan oleh resep tersebut yang saat ini tidak terhubung. Sebuah resep tidak pernah menghubungkan perangkat: jalankan `chloros-cli project connect PATH` terlebih dahulu. `--dry-run` mencetak resep, kemudian menampilkan kesalahan dan peringatan yang sama, serta `[recipe] valid: …` / `NOT valid: …`, tidak menjalankan apa pun, dan keluar dari `1` ketika resep mengalami kesalahan.

---

## Variabel Lingkungan

| Variabel | Efek |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Mengganti URL backend (default `http://127.0.0.1:5000`) — **hanya diakui oleh keluarga perintah `lattice`, `project`, dan `daq pool-*`** (`lattice network-analysis` menggunakan `--backend-url` sebagai gantinya). Perintah inti (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) mengabaikan `http://127.0.0.1:<port>` dan mengabaikan variabel ini (literal IPv4 menghindari penalti Windows `localhost`→`::1` ~2 detik per permintaan), sehingga mereka selalu menargetkan mesin lokal. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Ditetapkan di lingkungan backend. `1` menurunkan tingkat penolakan kelebihan langganan — koneksi array dengan `pin_resolution`, atau [perintah multi-kamera sementara](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — menjadi peringatan keras, dengan menerima kehilangan paket GVSP. Hanya untuk pengujian — lihat [Model fps & burst array](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Atur di lingkungan backend: `on` atau `off` menetapkan nilai default [transportasi 12-bit terkompresi](#packed-12-bit-transport---packed) untuk setiap array (default `auto`). Pengaturan eksplisit `--packed` tetap diutamakan. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (atau `true` / `yes`) menonaktifkan [pemeriksaan pembaruan otomatis](#chloros-cli-update) pada CLI Linux saat startup. |
| `CHLOROS_CLI_MODE` | Ditetapkan oleh CLI itu sendiri; memerintahkan backend untuk mengaktifkan pemrosesan paralel. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` melewatkan probe fallback GVSP (hanya hasil ICMP). **Ini menonaktifkan jumbo, bukan sekadar mengurangi log** — kamera hanya merespons ping DF hingga 1500 pada setiap jalur, sehingga pemeriksaan ini adalah satu-satunya cara untuk mendeteksi jumbo. Menghemat ~1 detik per kamera per koneksi; menghabiskan ~1,45× batas maksimum kabel jika jaringan *dapat* mentransmisikan paket jumbo. SDK akan memberikan peringatan saat Anda mengaturnya. |
| `CHLOROS_GVSP_SESSION_REUSE` | Ditetapkan di lingkungan backend: `0` mengukur ukuran paket pada setiap koneksi, bukan sekali per kamera per sesi backend. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Tetapkan ukuran paket GVSP ke N byte; melewati proses pengujian sepenuhnya. Lebih disarankan menggunakan pengaturan per-perintah (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) daripada mengaturnya secara permanen: ukuran yang ditetapkan secara tetap tidak lagi menyesuaikan dengan jaringan di depannya, dan menetapkan 9000 pada jalur yang tidak dapat mentransmisikan paket jumbo akan menyebabkan **setiap** penangkapan mengalami timeout dengan `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Atur di lingkungan backend: jumlah detik maksimum untuk [perintah multi-kamera sementara](#multi-camera-transient-sessions), atau `lattice capture` dari kamera yang tidak memiliki pengaturan apa pun, menghabiskan waktu untuk mengatur eksposur setiap kamera dari adegan sebelum menyimpan bingkai (default `5`). `0` menonaktifkannya: pengaturan eksposur otomatis masing-masing kamera kemudian berjalan mulai dari frame pertama, dimulai dari batas eksposurnya. |
| `CHLOROS_LOG_DIR` | Ditetapkan di lingkungan backend: folder tujuan log terperinci. Default `%LOCALAPPDATA%\Chloros\logs` di Windows, `~/.cache/chloros/logs` di Linux (`$XDG_CACHE_HOME/chloros/logs` jika diatur). Layanan root dari berkas .deb menetapkan `/var/log/chloros`, yang dapat dibaca oleh grup `adm` — lihat [Menjalankan backend sebagai layanan](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Lokasi penyimpanan berkas sementara. CLI secara otomatis menggunakan `/mnt/ssd/tmp` jika tersedia. |

---

## Kode Keluar

| Kode | Arti |
| --- | --- |
| `0` | Berhasil. |
| `1` | Kegagalan umum (kebanyakan kesalahan subperintah). Perintah perekaman juga akan keluar dengan kode `1` jika ada file yang diminta untuk ditulis gagal ditulis, meskipun file lainnya berhasil disimpan, dan `array-burst` / `array-record` akan keluar dari `1` ketika perekam melaporkan kesalahan — lihat [File yang Gagal Ditulis](#files-that-fail-to-write). `process` juga keluar dari `1` ketika ekspor dari suatu eksekusi tidak dapat diverifikasi, atau ketika kehilangan gambar, meskipun produk lainnya berhasil ditulis. |
| `2` | Kesalahan argumen (bendera yang tidak dikenal atau hilang; juga `time-sync set-priority` tanpa prioritas apa pun). |
| `130` | Terputus oleh Ctrl+C: perintah apa pun yang dihentikan secara paksa — `lattice`, `project`, `daq`, dan `process` (`process` setelah backend menghentikan eksekusi, lihat [Catatan `process`](#notes)). Perintah yang biasanya dihentikan dengan Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst`, dan `project stream` dengan `-n 0` — akan berhenti berdasarkan apa yang dilakukannya: `0` saat semua data telah ditulis, `1` saat terjadi kegagalan apa pun. |
| `143` / `129` | Dihentikan oleh SIGTERM / SIGHUP (128 ditambah sinyal), setelah proses pembersihan perintah selesai: pekerjaan yang sedang berjalan dihentikan dan kameranya dikembalikan ke posisi semula, proses pemrosesan di backend dihentikan. Sebuah `lattice continuous` atau perekaman lain yang biasanya dihentikan dengan Ctrl+C juga akan keluar dari `143`, bukan `0`. |

`lattice network-analysis` akan keluar dari `1` untuk keputusan `needs_force_slip` dan `error`, dan `selftest` akan keluar dari `1` kecuali jika ketujuh pemeriksaan tersebut lulus.

---

## Petunjuk Pemecahan Masalah

- **"Diperlukan login"** → Jalankan `chloros-cli login EMAIL PASSWORD` sekali pada mesin ini.
- **"Folder proyek terlalu dalam untuk komputer ini"** → Jalur panjang Windows dinonaktifkan pada mesin ini (pengaturan default Windows), sehingga tidak ada jalur yang boleh melebihi 260 karakter, dan produk berada hingga sekitar 130 karakter di bawah folder proyek. Tidak ada yang diproses. Pesan tersebut menunjukkan folder proyek terpanjang yang masih sesuai: gunakan yang lebih pendek (`-o`, atau `chloros-cli set-project-folder`), atau aktifkan jalur panjang Windows dan mulai ulang Chloros. Penginstal Windows menawarkan opsi ini sebagai "Aktifkan jalur panjang Windows", yang dicentang secara default; jika tidak, gunakan Kebijakan Grup: *Konfigurasi Komputer > Template Administratif > Sistem > Sistem Berkas > Aktifkan jalur panjang Win32*.
- **"backend tidak dapat dijangkau"** → Jalankan aplikasi desktop Chloros (Windows), atau jalankan backend secara manual: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` di Windows (folder `backend` di dalam folder instalasi Anda; tidak ada di `PATH`), `chloros-backend` di Linux. Atau periksa `CHLOROS_BACKEND_URL` jika menggunakan koneksi jarak jauh.
- **Perintah `lattice` gagal dengan pesan "Driver kamera LATTICE tidak ditemukan"** → Backend tidak dapat memuat runtime kamera LATTICE yang terpasang bersamanya. Setiap perintah `lattice` dijalankan di backend; CLI-nya sendiri tidak memuat driver kamera. Instal ulang Chloros: penginstal Windows dan berkas .deb akan menginstal runtime di samping backend.
- **Array connect / Array Settings menampilkan "FRAMES WILL DROP" atau "Reduce ROI to enable"** → Ring penerima NIC host terlalu kecil (biasanya diatur ulang menjadi 32 setelah pembaruan driver NIC). Lihat [Pengaturan & Penyesuaian NIC Host](#host-nic-setup--tuning-lattice-arrays) — atur `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Mesin macet saat restart/shutdown, kemudian WMI menampilkan kode kesalahan `Invalid class` / NIC tidak dapat diaktifkan / drive USB tidak terdeteksi** → Driver adaptor USB 10GbE yang sudah usang menyebabkan kode kesalahan `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Perbarui driver adaptor — lihat [Pengaturan & Penyesuaian NIC Host](#host-nic-setup--tuning-lattice-arrays).
- **Sistem Linux: data tangkapan milik pengguna root, sehingga Anda tidak dapat menghapusnya, dan log backend tidak ada di direktori Anda `~/.cache`** → Backend berjalan sebagai root melalui `chloros-backend.service`. Beralihlah ke `chloros-backend@<user>.service`, yang menjalankannya sebagai Anda — lihat [Menjalankan backend sebagai layanan](#running-the-backend-as-a-service-headless). Sampai saat itu, log layanan root berada di `/var/log/chloros` (dapat dibaca oleh grup `adm`).
- **Linux: unit backend adalah `failed` dengan pesan "Instan lain sudah berjalan" (status keluar 98)** → Port tersebut sudah digunakan oleh backend lain: hanya satu yang dapat berjalan pada satu waktu. Jika dua unit backend diaktifkan, pertahankan satu: `sudo systemctl disable --now <unit>` untuk yang tidak Anda gunakan — lihat [Menjalankan backend sebagai layanan](#running-the-backend-as-a-service-headless). Jika hanya unit ini yang diaktifkan, backend yang dimulai di luar systemd (oleh `chloros-backend`, `chloros-cli`, atau skrip SDK) sedang menggunakannya: hentikan backend tersebut, lalu jalankan `sudo systemctl start <unit>`.
- **Peringatan swap Jetson** → Tambahkan swap berbasis file; antarmuka baris perintah (CLI) akan menampilkan perintah `fallocate` / `swapon` secara tepat.
- **"backend tidak dapat dijangkau" dari perintah `lattice`, `project`, atau `daq`** → tidak ada backend yang berjalan: perintah-perintah tersebut tidak pernah memulai backend. Jalankan `chloros-cli backend status`, dan lihat [Memulai Backend](#start-the-backend).
- **"`daq net ...` tidak tersedia di Chloros"** (Linux dan Jetson) → Beberapa pengaturan DAQ-E sendiri dicadangkan untuk layanan MAPIR. Pengaturan jaringan, restart, dan pembaruan firmware adalah `pool-set-network`, `pool-reboot`, dan `pool-update-firmware`; hubungi dukungan MAPIR untuk yang lainnya.
- **"Perintah perangkat keras langsung DAQ memerlukan paket daq lengkap"** (Windows) → Gunakan `pool-*`, yang mengendalikan sensor yang sama melalui backend, atau `chloros_sdk.connect_daq_sensor()` dari Python.

---

## Lihat Juga

- [Referensi SDK Python](sdk-reference.md) — padanan programatik dari setiap perintah CLI.
- [Sensor Cahaya DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) dalam manual Chloros — sensor, batas dan rentang kalibrasinya, perekaman, jaringan DAQ-E.
- [Kamera LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) dalam manual Chloros — koneksi, pengaturan kamera, mode pengambilan gambar, dan susunan kamera.
- Dokumen daring: `https://mapir.gitbook.io/chloros/cli`

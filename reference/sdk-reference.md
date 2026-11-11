# Referensi Chloros Python SDK

**Versi:**

1.3.0**Dibuat:**29 Juli 2026 19:19 ·**Direvisi:** 6 November 2026**Paket:** `chloros-sdk` (PyPI)**Target:** Dioptimalkan untuk penggunaan LLM; mudah dibaca oleh manusia.**Cakupan:** Setiap kelas, fungsi, dan helper publik yang disediakan oleh `import chloros_sdk`, dengan contoh-contoh yang dapat disalin dan ditempelkan yang mencakup pemrosesan gambar, kontrol kamera tunggal, array tersinkronisasi, sensor DAQ, dan otomatisasi proyek, ditambah server `chloros-mcp` untuk asisten AI.

Jika Anda hanya membutuhkan poin-poin penting, langsung ke:
- [Instalasi &amp; Panduan Cepat](#instalasi)
- [Smart-Connect untuk LATTICE Arrays](#smart-connect-for-lattice-cameras)
- [Sesi Sensor DAQ](#daq-sensor-sessions)
- [Utilitas Kamera](#utilitas-kamera)
- [Array Transien, Profil Penyelarasan &amp; Indeks](#array-transien-profil-penyelarasan--indeks)
- [Folder Proyek &amp; Resep](#folder-proyek--resep)
- [Otomatisasi Proyek](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Bingkai Langsung dalam Pipa Kerja Anda Sendiri](#live-frames-in-your-own-pipeline)
- [Server MCP (Asisten AI)](#mcp-server-ai-assistants)

> **Perubahan pada versi 1.3.0** (yang terdeteksi oleh skrip 1.2.0 yang sudah ada).
>
> **Perubahan yang mungkin perlu ditangani oleh skrip 1.2.0:**
>
> - **Rilis yang tidak dikonfirmasi oleh backend kini akan memicu kesalahan.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` pada array yang dibuka oleh sesi, `DAQSensorSession.disconnect()` / `force_disconnect()` pada sensor yang dibuka oleh sesi tersebut, serta `ChlorosProject.disconnect_all()` akan memicu `ChlorosConnectError` ketika backend menolak pelepasan atau tidak mengonfirmasinya tepat waktu, karena perangkat mungkin masih dalam penguasaan. Pada versi 1.2.0, kode-kode tersebut dikembalikan tanpa pemberitahuan. Keluarnya blok `with` tetap tidak pernah memicu pengecualian.
> - **Perangkat yang dipinjam tidak pernah dihapus.** `with attach_array(...)` dan panggilan eksplisit `disconnect()` pada perangkat tersebut membuat array tetap berjalan (pada versi 1.2.0, keluar dari blok akan menghapus perangkat tersebut, bahkan ketika GUI masih mengendalikannya); panggil `force_disconnect()` jika Anda bermaksud mengakhirinya. Demikian pula, perintah `connect_daq_sensor()` yang mendeteksi sensor sudah ada di dalam pool—termasuk fitur smart-detect dengan bendera nol—akan membiarkannya tetap terbuka pada `disconnect()` dan saat meninggalkan blok `with`-nya.
> - **Format PNG dan JPEG hanya menerima produk `display` dan produk berindeks.** `capture()` dengan `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) dan tingkat `radiance` atau `reflectance`, atau `processing="all"`, akan memicu `ValueError` sebelum permintaan apa pun; `raw` / `debayered` 12-bit ditolak oleh backend. Lihat [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` membuka sensor tersebut atau memicu** `ChlorosConnectError` yang mencantumkan apa yang ditemukannya; ia tidak pernah beralih ke sensor lain. Lihat [Pinned Transport](#pinned-transport).
> - **`frame_stream()` memicu kesalahan alih-alih diam:** `FrameFetchError` pada penolakan, pada tick berikutnya akan mengulangi (pembacaan reflektansi tanpa DAQ sama sekali), setelah 60 detik tanpa frame, dan setelah 60 detik tanpa pemicu baru. Lihat [Satu pemicu per pembacaan](#one-trigger-per-read--group_id).
> - **Timeout pembacaan atau koneksi terputus memicu `ChlorosConnectError`** dari setiap panggilan smart-connect dan `analyze_array_network()`, di mana versi 1.2.0 memicu timeout atau kesalahan koneksi yang mendasarinya.
> - **`connect_array()` menunggu 600 detik secara default (`timeout`, 180 detik pada versi 1.2.0), dan koneksi yang gagal akan dibatalkan.** Panggilan ini mengirimkan `timeout` ke backend, yang mengembalikan setiap kamera ke kondisi semula dan tidak melakukan koneksi apa pun jika koneksi tidak selesai tepat waktu; panggilan ini memicu `ChlorosConnectError` (504 HTTP `CLIENT_DEADLINE_PASSED`). Pada versi 1.2.0, koneksi yang dilakukan segera setelah kamera dinyalakan dapat berlangsung lebih lama dari panggilan tersebut dan menghubungkan array setelah panggilan tersebut memicu kesalahan. `connect_all()` menunggu setiap array yang disimpan dengan cara yang sama (120 detik pada versi 1.2.0). Lihat [Tanda Tangan `connect_array()`](#connect_array-signature).
> - **`connect_array()` mempertahankan sensor secara utuh.** Tanpa `width` / `height` / `binning`, setiap kamera diatur ke sensor penuh dan laju bingkai turun sesuai dengan yang disediakan oleh host dan jaringan; pada versi 1.2.0 akan mengompres atau menggabungkan frame untuk menjaga laju tetap tinggi. Lihat [Synchronized Array](#synchronized-array--arraysession-smart-prep).
> - **Format piksel Bayer pada kamera M3M (mono) adalah kedalamannya.** Kamera M3M hanya menawarkan format Mono, dan nama presetnya menggunakan istilah Bayer: pada versi 1.2.0, kamera menolak format tersebut dan tetap menggunakan formatnya sendiri, sementara `settings.pixel_format` menamai format Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)`, dan sebuah array&#x27;format bersama kini menetapkan format Mono dengan kedalaman yang sama pada M3M: `"BayerRG8"` dan `high_speed` menghasilkan `Mono8`; `"BayerRG12"`, `high_quality`, dan `triggered` menghasilkan `Mono12`; `cam.settings.pixel_format` menamainya. Hal yang sama berlaku sebaliknya: M3C tidak pernah disimpan dalam format Mono, sehingga `"Mono12"` — dari salah satu di antaranya, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)`, atau format penyimpanan array — akan menghasilkan `BayerRG12` pada M3C, terlepas dari kamera mana yang menjadi master array tersebut. Sebuah `CameraSettings` yang Anda lewatkan mempertahankan namanya, sehingga objek yang sama tetap meminta kedalaman tersebut dari setiap kamera dalam kelompoknya sendiri. `connect_camera(preset=...)` mempertahankan format preset (versi 1.2.0 mengatur kamera ke 12-bit setelah menerapkan preset), dan kamera yang dibuka pada preset tidak lagi menulis hasil pengambilan gambarnya ke dalam `PRESETS`. Format lain apa pun yang ditolak oleh kamera (yang tidak tercantum dalam daftar `PixelFormat`-nya, seperti `RGB8` pada M3M atau `BayerRG16` pada firmware yang tidak mendukungnya) juga tidak diberi nama tersendiri: setelah koneksi `LatticeCamera` terhubung, `apply_settings()` atau `set_format()`, `cam.settings.pixel_format` dan `packed_transport` menyebutkan format yang disiarkan kamera, dan satu baris cetakan menyebutkan format yang ditolak.
> - **`analyze_array_network()` menunjukkan koneksi mana yang berlaku:** `pixel_format=None` (sebelumnya `"BayerRG8"`) berarti format yang ditetapkan oleh koneksi (Mono12 / BayerRG12), sedangkan `packed="auto"` menunjukkan keputusan pengiriman terkompresi, sehingga nilai fps yang disebutkan dapat berbeda dari 1.2.0&#x27;. Sebuah `pixel_format` yang Anda lewati menampilkan M3M pada kedalaman tersebut. Lihat [Network-Analysis Helper](#network-analysis-helper).
> - **`CameraSession.capture()` mengembalikan sebuah `CaptureResult`** (masih berupa daftar kamus bingkai) dengan `.skipped`, `.failed`, `.daq_files`, dan `.daq_matches`. Hasil `ArraySession.capture()` juga menyertakan `.alignment` (apakah penyelarasan warp diterapkan, dan mengapa tidak) serta `.oriented`.
> - **Penangkapan array secara default disejajarkan, sama seperti yang dilakukan oleh aplikasi.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` dan `ArrayHandle.capture()` / `capture_smart()` melakukan penyesuaian terhadap file `debayered`, `display`, `radiance`, `reflectance`, dan berkas indeks ke dalam bingkai kamera utama, lalu memotongnya sesuai tumpang tindih antar anggota (lebih kecil dari sensor sebesar offset kamera, `XMP-mapir:AlignmentJSON` `"applied": true`), sehingga file-file tersebut sama dengan hasil tangkapan aplikasi Chloros pada pemandangan yang sama; `raw` tetap dalam ukuran aslinya. Sebelumnya, pengambilan gambar tanpa `aligned=True` menyimpan setiap berkas dalam ukuran asli, tanpa distorsi. `aligned=False` mempertahankan hal tersebut. `chloros-mcp`&#x27;`capture_array` dan `start_capture_run` menggunakan pengaturan default yang sama. Lihat [Metode `ArraySession`](#arraysession-methods).
> - **`connect_all()` menyebutkan backend yang menolak SDK.** Backend yang sedang berjalan namun telah logout atau menggunakan paket di bawah Copper dilaporkan di bawah `errors` dengan alasan tersendiri (HTTP 401 / 403), bukan sebagai &quot;tidak ada backend Chloros yang sedang berjalan&quot;, dan setiap panggilan `connect_all()` memeriksa backend tersebut kembali, sehingga memanggilnya setelah masuk akan berfungsi pada objek proyek yang sama.
> - **Panggilan `output_dir` relatif berfungsi terhadap backend lokal.** Jalur relatif diubah menjadi absolut berdasarkan direktori kerja skrip Anda sebelum dikirim. Pada versi 1.2.0, backend menerjemahkannya di dalam folder instalasinya sendiri, yang gagal dengan pesan &quot;Akses ditolak&quot;.
> - **`numpy` kini menjadi persyaratan** dari SDK (panggilan live-frame mengembalikan array numpy).
> - **Nama tingkat ekspor yang tidak sesuai akan ditolak.** `processing=` dan `levels=` menerima `raw`, `debayered`, `radiance`, `reflectance`, atau `display` (atau alias seperti `preview`; `processing=` serta `"all"`). Nama lain apa pun akan ditolak sebelum kamera disentuh (HTTP 400 `UNKNOWN_LEVEL`: `capture()` memicu `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); versi 1.2.0 menyimpan atau mengembalikan `raw` untuk hal tersebut.
> - **Fitur Texture Aware ditolak untuk tangkapan LATTICE.** Pada proyek yang berisi tangkapan LATTICE, `configure(debayer="Texture Aware (Slow, Highest Quality)")` memicu `ChlorosConfigurationError`, begitu pula `process()` setelah `configure()` yang meminta sebelum impor. Nilai Texture Aware yang disimpan proyek sebelumnya diatur ke Standard, disertai dengan `UserWarning`. Rekaman LATTICE selalu menggunakan demosaic standar, sehingga versi 1.2.0 menerima pengaturan tersebut dan mengabaikannya; aplikasi Chloros juga tidak menawarkannya untuk rekaman tersebut. Lihat [Nilai yang Didukung](#supported-values).
> - **Kamera dikembalikan ke kondisi semula — kecuali jika terjadi `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, akhir dari setiap fungsi sementara (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) serta penghentian `start_camera_continuous()` akan mengembalikan setiap kamera ke kondisi semula: kabel pemicu dan garis sinkronisasinya, PTP-nya, serta setiap pengaturan kamera yang diubah selama sesi (pencahayaan dan penguatan, target AE, batas dan redaman, keseimbangan putih, tingkat hitam, format piksel, binning dan ROI, pembalikan, laju bingkai). Sebaliknya, `keep_settings=True` menyimpan setiap pengaturan kamera ke dalam memori kamera itu sendiri, sehingga kamera menyala dengan pengaturan tersebut; `clear_settings=True` mengatur ulang memori tersebut ke pengaturan pabrik. Dalam kedua kasus tersebut, kamera dibiarkan berjalan secara mandiri. Sebuah proyek mengatur kameranya berdasarkan pengaturan yang telah disimpan saat menghubungkannya. Lihat [Memori kamera](#camera-memory--keep_settings--clear_settings).
> - **Setiap mode menyimpan proyek dengan cara yang sama.** Setiap perubahan pada kamera, array, atau sensor cahaya yang terhubung ke suatu proyek — baik dari SDK, aplikasi, CLI, maupun server MCP — disimpan ke dalam folder `cameras.json` / `sensors.json`, dan membuka proyek tersebut dalam mode apa pun akan mengatur perangkat berdasarkan pengaturan tersebut; sebuah `ChlorosProject` menamai foldernya sendiri ke backend. Sebelumnya, hanya aplikasi yang menyimpan proyek. Lihat [Satu catatan proyek, semua mode](#one-project-record-every-mode).
> - **File yang dibuat dari beberapa kamera mencantumkan nama masing-masing kamera.** Tumpukan multiband, indeks langsung yang dibuat dari pita beberapa kamera, dan gambar tangkapan array `composite/` menyimpan nomor seri, model, dan paket kalibrasi masing-masing kamera sumber sebagai daftar (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), sedangkan daftar EXIF `SerialNumber` mencantumkan nomor seri yang dipisahkan dengan koma. Sebelumnya, file-file tersebut tidak mencantumkan kamera apa pun, atau hanya mencantumkan nomor seri kamera pertama saja. Lihat [File yang dibuat dari beberapa kamera](#files-made-from-several-cameras).
> - **Hanya file `raw` yang memuat tag kalibrasi kamera.** Set yang kompatibel dengan MicaSense (`BlackLevel`, `RadiometricCalibration`, set per-saluran `Band{R,G,B}RadiometricCalibration`, set `MicaSenseVignetting*`) serta muatan `CalibrationJSON` menggambarkan nilai mentah sensor, sehingga hanya disertakan dalam berkas penangkapan `raw`. Setiap berkas lainnya — berkas penangkapan`debayered`, `display`, `radiance`, `reflectance`, serta berkas indeks, rekaman berkelanjutan, dan, serta setiap produk `process()` — memuat identitas kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) dan, pada saat pengambilan gambar, catatan bingkai itu sendiri (`XMP-mapir:FrameJSON`: eksposur, gain, suhu, format piksel, waktu pengambilan). Pada versi 1.2.0, file-file tersebut juga membawa set data mentah, yang diterapkan oleh alat pengolah (Pix4D, Metashape) ke piksel yang bukan nilai mentah. Processing menentukan kalibrasi file tersebut berdasarkan nomor seri dan `CalBundleSha256`.
>
> **Perubahan lainnya:**
>
> - **Kecepatan bingkai otomatis suatu array disesuaikan dengan apa yang dihasilkan oleh array tersebut.** Kecepatan yang kehilangan lebih dari 1% dari pemicunya akan diturunkan, dinilai berdasarkan beberapa pengamatan jangka panjang daripada hanya satu kali, dan kecepatan yang sebelumnya diturunkan akan dicoba kembali sesekali. Kecepatan yang Anda atur sendiri tidak akan diubah.
> - **Pencahayaan otomatis kini lebih stabil.** Fitur ini mempertahankan pencahayaan konstan, menyesuaikan dalam hitungan detik setelah terjadi perubahan, mengabaikan sorotan kecil, dan mengaktifkan kembali kamera yang berhenti merespons di bawah sinar matahari yang terik. Lihat [Bagaimana Pencahayaan Otomatis Bekerja](#how-auto-exposure-behaves).
> - **`import_images(recursive=True)` hanya menghitung gambar dan rekaman Anda.** Folder yang berisi berkas kalibrasi `camera_cal/` dari suatu proyek tidak lagi menghitung berkas-berkas tersebut sebagai gambar dan rekaman, dan proyek yang sebelumnya telah memasukkannya akan menghapusnya.
> - **`configure(radiometric_output=...)` berlaku.** Sekarang, pengaturan ini menetapkan produk `Export radiance` / `Export reflectance` proyek; sebelumnya, pengaturan produk masing-masing proyek yang berlaku, sehingga `radiance`, `sensor-response`, dan `all` semuanya mengekspor radiance dan reflektansi.
> - **`test_array_sync()` dijalankan saat kamera dalam keadaan menyala.** Fungsi ini mengaktifkan PTP kamera untuk proses pengambilan data, menunggu jam internal kamera terkunci dan stabil sebelum pemicu pertama (`ptp_lock`) — setiap kamera terkunci selama 120 detik, sekitar 2 menit sejak dinyalakan (`ptp_settle_s=` mengaturnya), karena jam yang baru saja terkunci masih memiliki selisih hingga ~200 us selama sekitar 2 menit — dan mengembalikan PTP setiap kamera ke kondisi semula. Kamera yang masih terkunci dari pengujian sebelumnya tidak ditunggu lagi. Dulu, PTP dibiarkan mati pada jaringan dengan satu sakelar di mana tidak ada DAQ-E yang merespons, dan jam bebas (free-running clock) setiap kamera dibaca sebagai jam PTP yang tidak akan terkunci (&quot;PTP tidak terkunci&quot;, pada setiap eksekusi).
> - **Perekaman `.daq` menunjukkan apakah penutupnya dipilih atau diasumsikan.** DAQ-U, DAQ-M, dan DAQ-E dikirim tanpa penutup, dan Chloros tidak dapat mendeteksi apakah penutup terpasang, sehingga sensor yang terhubung tanpa penutup yang ditentukan akan diberi nama `sunshine_cosine` secara default, yaitu sebuah *asumsi*. Setiap rekaman Chloros (skema 1.30) kini menyimpan `cap_id_source` di samping `cap_id`: `operator` saat Anda memberi nama penutup (termasuk `none`), `auto_default` saat itu menjadi nilai default. Memproses rekaman yang kapasitornya diasumsikan, dengan pengaturan DAQ proyek dibiarkan pada &quot;Auto&quot;, akan menampilkan `[daq-dls] WARNING` yang mencantumkan nama file: DAQ-U tanpa filter di bawah koreksi sinar matahari menunjukkan nilai sekitar 30 kali lebih tinggi. **Beri nama kapasitor yang Anda pasang** saat menghubungkannya (`connect_daq_sensor(cap_id=...)` atau `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, atau menu dropdown kapasitor di GUI), atau atur kapasitor DAQ proyek, dan peringatan tersebut akan hilang.
> - **Tidak ada lagi jendela konsol pip di Windows.** Pada PC yang telah menginstal Python, backend dan setiap image worker-nya menjalankan `pip` sekali saat startup, dan masing-masing membuka jendela konsol. Kini hal tersebut tidak terjadi lagi.
> - **Rekaman langsung `.daq` tetap utuh meskipun terjadi pemadaman listrik.**Rekaman ditulis dengan jurnal dan sinkronisasi disk penuh pada setiap commit, di belakang sensor, sehingga disk yang lambat hanya menunda file, bukan pembacaan data. Rekaman yang terhenti di tengah proses commit (pemadaman listrik, penghentian paksa) dapat meninggalkan berkas `<name>.daq-journal` di sampingnya:**salin kedua berkas tersebut bersama-sama.** Chloros mengembalikan jurnal saat pertama kali membaca berkas, sehingga rekaman dibuka dengan semua data yang telah dikomit sebelum penghentian (sekitar 3 detik terakhir hilang); pada versi 1.2.0, pemadaman listrik pada waktu yang salah dapat membuat seluruh berkas tidak dapat dibaca (&quot;gambar disk basis data rusak&quot;).
> - **`DAQSensorSession.record_start()` pada disk yang sibuk tidak lagi membuat sensor macet**: pembacaan diantrekan dan ditulis pada thread tersendiri, dan pembacaan yang tidak dapat ditulis dihitung dalam `status()` (bersama `record_errors` dan `record_last_error`).
> - **`LatticeCamera` Langsung:**sebuah frame tanpa pengaturan eksposur/gain chunk kini melaporkan eksposur dan gain**sebenarnya** dari kamera (sebelumnya, pembacaan cadangan menggunakan nilai yang disimpan dalam cache yang bisa tetap beku selama seluruh proses eksposur otomatis), dan `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` menerapkan deglitch input 5 µs yang sama seperti yang diterima oleh slave kabel Chloros (tanpa penyaringan, slave memicu frame tanpa sinyal pemicu yang dikirim).
> - **Kamera yang dipicu melalui perangkat lunak tidak lagi melewatkan setiap pemicu kedua.** Kamera dalam mode pemicu perangkat lunak kini mengantrekan pemicu yang tiba selama proses pembacaan (atau Chloros menunggu hingga kamera siap), seperti yang sudah dilakukan oleh kamera yang disinkronkan melalui kabel.
> - **`discover_daq_sensors()` menamai DAQ-E-S sebagai `DAQ-E-S`.** Nama `model`-nya adalah `DAQ-E` untuk setiap sensor Ethernet; sensor yang tidak merespons pemeriksaan identitas dalam waktu sekitar 2 detik tetap terdaftar, sebagai `DAQ-E`.
> - **Baru: `chloros-mcp`, sebuah server MCP untuk asisten AI.** Claude Desktop, Claude Code, Cursor, dan klien MCP lainnya dapat memproses citra (membuat proyek, mengimpor tangkapan, memilih pengaturan, memproses, melaporkan hasil ekspor) serta mengoperasikan kamera LATTICE, array, dan sensor cahaya DAQ: terhubung, pratinjau, mengatur eksposur dan warna, menangkap, merekam. Alat-alatnya mencakup fungsi yang dilakukan oleh SDK ini. Perangkat lunak ini diinstal bersama SDK, dan paket .deb untuk Linux menempatkannya di `/usr/bin`. Lihat [Server MCP (Asisten AI)](#mcp-server-ai-assistants).
> - **Baru: utilitas kamera**, perintah `chloros-cli lattice` sebagai panggilan SDK: profil warna, saturasi/kontras, sorotan, keseimbangan putih (satu kali, otomatis, rasio tetap), preset, model filter, status dan pembaruan firmware, daya (probe, tidur, bangun), kalibrasi target, adaptor jaringan (periksa, perkirakan, perbaiki), koreksi vignette pada file, pembacaan DLS, dan probe host. Lihat [Utilitas Kamera](#camera-utilities).
> - **Baru: sisa perintah CLI sebagai panggilan SDK**, sehingga CLI, SDK, dan `chloros-mcp` menawarkan fitur yang sama (CLI hanya mempertahankan proses masuk, penyiapan DAQ-E, dan server DAQ langsung, demi keamanan): kamera dibuka dengan satu perintah (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), berkas profil penyelarasan (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), indeks (`index_catalog()`, `compute_index()`, `capture_live_index()`), grandmaster PTP (`time_sync_status()` dan yang lainnya), folder proyek (`project_folder()`, `set_project_folder()`, `reset_project_folder()`), serta resep (`run_project_recipe()`, runner bawaan CLI). Untuk sensor cahaya: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()`, dan `light_sensor_reflectance()` (reflektansi dan indeks Ambient + Objek dari dua sensor); untuk kamera: `CameraSession.status()` dan `capture_repeated()`; untuk sebuah array: `ArraySession.capability()` dan `set_member_settings()` (pencahayaan, gain, dan lain-lain dari sebuah anggota). Lihat [Array Transien, Profil Penyelarasan &amp; Indeks](#transient-arrays-alignment-profiles--indices), [Folder Proyek &amp; Resep](#project-folder--recipes), dan [Sesi Sensor DAQ](#daq-sensor-sessions).
> - **Produk indeks LATTICE yang ditingkatkan** (NDVI dan indeks lainnya) dari `process()`, di setiap mode pemrosesan.
> - **Kamera LATTICE OCN: produk radian, reflektansi, dan indeks yang ditingkatkan di bawah sinar matahari.** Koreksi armada diterapkan setelah pemisahan pita — secara langsung, dalam penangkapan, dan di `process()`; kamera RGN tidak mengalami perubahan. Lihat [Output Radiometrik](#radiometric-output-lattice-multispectral-pipeline).
> - **Setiap tangkapan menyimpan pembacaan DAQ yang digunakannya, seperti yang dilakukan oleh GUI.** `CameraSession.capture()` dan `ArraySession.capture()` / `continuous()` menulis `.daq` dan `.match.json` setiap kali DAQ diikat atau dimasukkan ke dalam pool, termasuk penangkapan mentah saja; pada versi 1.2.0, penangkapan mentah atau yang telah di-debayering hanya disimpan satu kali dengan `force_daq=True`. `daq=False` menonaktifkannya dan `daq_tolerance_ms=` mengatur toleransi pencocokan; kedua hasil tersebut menunjukkan apa yang disimpan dalam `.daq_files` dan `.daq_matches`. Sekarang, penangkapan menunggu pembacaan DAQ berikutnya agar dapat menggabungkan pembacaan di kedua sisi eksposur. Lihat [Pembacaan DAQ yang disimpan oleh penangkapan](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
> - **`ArraySession` memiliki bingkai langsung dan penangkapan berkelanjutan di sisi server.** `grab()` dan `frame_stream()` mengembalikan bingkai numpy dari satu pemicu (pembacaan yang sama seperti `ArrayHandle.grab()` / `frame_stream()` pada API proyek), dan `continuous()` menyimpan setiap grup yang disinkronkan di backend serta mengembalikan sebuah [`ContinuousHandle`](#continuoushandle). Lihat [Frame Langsung dalam Pipa Anda Sendiri](#live-frames-in-your-own-pipeline).
> - **Sebuah array `grab()` diikat ke satu pemicu** (`ArrayHandle.grab()`, dan `ArraySession.grab()` yang baru). Pada versi 1.2.0, 12 dari 20`ArrayHandle.grab()` mencakup dua pemicu. Setiap bingkai kini membawa `group_id`, dan sebuah array `frame_stream()` menghasilkan setiap pemicu sekali. Lihat [Satu pemicu per pembacaan](#one-trigger-per-read--group_id).
> - **Bingkai per-pita dan yang telah diselaraskan:** `grab()` / `frame_stream()` mengambil `band=`, `align=`, `bit_depth=`, dan `refl_max=` pada handle kamera dan array. Lihat [Mengekspor satu band](#exporting-a-single-band--band).
> - **Metode dan argumen `ArraySession` yang baru:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (bersama dengan `CaptureResult.group_seq` dan `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` merangkai kelompok-kelompok berturut-turut dan berhenti saat drive penuh.
> - **`ArraySession.record()` merekam tanpa GUI yang terbuka.** Saat tampilan gabungan GUI ditutup, backend merender tampilan itu sendiri:NDVI, atau indeks yang Anda berikan sebagai `expr=`. `RecorderHandle.stop()` memicu `ChlorosConnectError` untuk perekaman yang gagal, yaitu yang tidak menulis satu frame pun, dan mempertahankan hasil penghentian pada `.result` (meninggalkan blok `with` tidak memicu pengecualian: periksa `rec.result`). Lihat [`RecorderHandle`](#recorderhandle).
> - **Metode `DAQSensorSession` baru:** `enable_ptp()`, `force_disconnect()`, dan untuk pengaturan DAQ-E sendiri `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()`, dan `update_firmware()`. `record_stop()` mengembalikan `warning` dan `gaps`. Restart yang menutup perekaman akan menyebutkan namanya, dalam balasannya atau, jika restart gagal, pada `ChlorosConnectError` (`recording_closed`; lihat [Pengaturan Perangkat DAQ-E](#daq-e-device-settings)). Lihat [Metode `DAQSensorSession`](#daqsensorsession-methods).
> - **DAQ-E yang digabungkan (pooled) yang berhenti mengirim data akan dihubungkan kembali oleh backend**, dengan pengaturan yang dimilikinya sebelumnya, dan perekaman di atasnya berlanjut dalam `.daq` yang sama dengan celah yang ditandai dalam berkas; `record_stop()` mengembalikan celah-celah tersebut sebagai `gaps`. Pada versi 1.2.0, sensor tetap diam hingga terputus dan terhubung kembali, yang menghentikan perekaman. Lihat referensi CLI [DAQ-E yang Berhenti Mengirim Data](cli-reference.md#a-daq-e-that-stops-delivering).
> - **`open_project()` berjalan dari SDK yang sudah terpasang.** Wheel (penginstaldan PyPI) kini membawa pemuat proyek yang dibutuhkannya; sebelumnya, `open_project()` memicu `ImportError` yang menamai `project_cameras` / `project_sensors` di SDK apa pun yang diinstal melalui pip. `connect_all()` menghubungkan kamera, array, dan sensor cahaya yang telah disimpan melalui backend yang sedang berjalan (sensor cahaya melalui kumpulan DAQ-nya), dan perangkat yang disimpan yang tidak dapat dicari tercantum di bawah `errors` beserta alasannya, alih-alih dilewati tanpa penjelasan. Sebuah array yang disimpan yang sudah aktif di backend (GUI, atau antarmuka `chloros-cli project connect`) akan terhubung dan tetap berjalan, padahal sebelumnya hal ini dilaporkan sebagai kesalahan (“Kamera  <sn>sudah ada dalam array<id>”). Lihat [Metode `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()`, dan `process_lattice_capture()` menemukan Chloros di Windows.** Mereka mencari lokasi di mana penginstal menempatkan backend (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` secara default, folder khusus dari catatan penginstal itu sendiri), dan terhubung ke backend yang sudah berjalan tanpa perlu mencarinya sama sekali. Pada versi 1.2.0, mereka menampilkan pesan &quot;Backend Chloros tidak ditemukan&quot; pada mesin yang sudah terinstal kecuali jika Anda memberikan argumen `backend_exe=`.
> - **Backend yang mati saat startup akan segera dilaporkan.** Ketika `ChlorosLocal()` atau panggilan smart-connect memulai backend yang keluar sebelum merespons, panggilan tersebut akan memicu kesalahan dalam hitungan detik, yaitu `ChlorosBackendError` dari `ChlorosLocal` dan `ChlorosConnectError` dari panggilan smart--connect, beserta kode keluar, perintah yang dijalankan, dan baris terakhir yang dicetak oleh backend. Versi 1.2.0 menunggu hingga batas waktu startup 60 detik habis tanpa memberikan penjelasan. Lihat [Backend Auto-Start](#backend-auto-start).
> - **Backend tetap berjalan di antara skrip.** Keluar dari blok `with ChlorosLocal(...)`, `process_folder()` / `process_lattice_capture()` yang kembali dan `chloros-mcp` yang keluar tidak lagi menghentikan backend yang dimulai oleh SDK; skrip berikutnya akan menggunakannya kembali. Backend akan berhenti pada `logout()` (kecuali jika aplikasi Chloros terbuka atau proses sedang berjalan), pada `shutdown_backend()`, atau secara otomatis setelah 30 menit jika tidak ada yang menggunakannya (`CHLOROS_BACKEND_IDLE_MIN` dalam lingkungan backend mengubah hal tersebut; `0`: tidak pernah).
> - **Backend yang dimulai oleh SDK diminta untuk dimatikan, bukan dihentikan paksa.** `shutdown_backend()` memungkinkan backend melepaskan kamera dan sensor cahayanya serta menyelesaikan perekaman apa pun; backend tersebut hanya akan dihentikan jika masih berjalan 10 detik kemudian. Versi 1.2.0 menghentikannya seketika (di Windows, pemaksaan penutupan), yang dapat menyebabkan sensor cahaya DAQ yang terhubung ke backend tersebut menjadi tidak responsif. Backend yang ditemukan SDK sedang berjalan tetap dibiarkan berjalan.
> - **Di Linux, fitur auto-start tidak pernah menjalankan `backend_server.py` dari folder saat ini.** Tanpa backend yang terinstal, skrip yang dijalankan di folder yang berisi backend akan menjalankannya; kini hanya backend yang terinstal yang dijalankan (atau, untuk `ChlorosLocal`, yang disebutkan oleh `backend_exe=`).
> - **`configure(indices=...)`**menambahkan setiap preset ke proyek untuk rentang spektrum kameranya, termasuk proyek baru, dan memberikan peringatan (`UserWarning`) jika ada nama yang tidak dapat ditambahkan.**`import_images()`** melaporkan rekaman `.daq` sebagai `scan_count` dan tidak lagi memberikan peringatan mengenai folder penangkapan LATTICE.
> - **`read_image_audit_tags()` berjalan setelah diinstal melalui pip.** Program ini membaca metadata gambar secara mandiri; pada versi 1.2.0, program ini mengembalikan `{}` tanpa komponen internal instalasi desktop di jalur Python.
> - **`capture()` segera setelah `connect_array()`** menunggu hingga 20 detik untuk grup frame pertama dari array, alih-alih gagal dengan kode HTTP 503 &quot;Array masih dalam proses pemanasan&quot;.
> - **`status()` melaporkan kemacetan.** Field baru `stalled` bernilai True selama loop pengambilan berjalan tetapi tidak menerbitkan apa pun, dan `actual_fps` kemudian bernilai 0. Pada versi 1.2.0, `actual_fps` mempertahankan nilai terakhirnya selama kemacetan, sehingga array yang macet menampilkan `actual_fps > 0` sementara penangkapan gagal dengan kode 503 &quot;Tidak ada grup frame sinkron yang baru&quot;.
> - **Lebih sedikit set bingkai yang tidak lengkap dan kesalahan penangkapan 503:** masukan pemicu setiap kamera slave kini telah dibersihkan dari gangguan.
> - **Transportasi 12-bit Terkompresi** (`connect_array(packed="auto")`) meningkatkan laju bingkai array sensor penuh sekitar sepertiga. Lihat [Transportasi 12-bit Terkompresi](#packed-12-bit-transport-packed).
> - **Pencahayaan array di bawah ROI:** ROI kecil tidak lagi membatasi pencahayaan anggota pada periode free-run sensor (4,9 ms pada 768 × 304). Lihat [Pencahayaan pada Anggota Array](#exposure-on-array-members).
> - **Eksposur otomatis tidak lagi menurunkan laju bingkai otomatis array.** Sebuah array yang disinkronkan melalui kabel beroperasi pada laju yang diizinkan oleh jaringan dan pembacaan kamera; eksposur otomatis mengisi periode yang diberikan oleh laju tersebut, lalu menaikkan gain. Eksposur manual yang lebih lama dari periode tersebut akan memperlambat pemicu agar sesuai dengannya, bukan ditolak. Lihat [Eksposur pada Anggota Array](#exposure-on-array-members).
> - **`read_nodes()`**menambahkan `frame_values` dan `notes` saat eksposur otomatis berjalan (lihat [`read_nodes()` di bawah eksposur otomatis](#read_nodes-under-auto-exposure--frame_values-and-notes)). Ketika kamera menolak suatu eksposur atau gain,**`set_settings()`** menunjukkan rentang yang diterima dan mengaktifkan kembali mode otomatis yang sebelumnya dimatikan untuk nilai tersebut; `set_settings(target_brightness_auto=True)` adalah kode baru.
> - **`orient=`** pada `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` dan `ArrayHandle.capture()` / `capture_smart()` memutar produk sesuai orientasi kamera. Lihat [Metode `ArraySession`](#arraysession-methods).
> - **`DLS()` memerlukan `mac_address` dari sensor itu sendiri.** Tidak ada lagi MAC default (sebelumnya merupakan alamat satu unit); `DLS()` tanpa alamat tersebut akan memicu `DLSError`. Lihat [besaran pita DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **Penginstal Windows** menawarkan opsi &quot;Aktifkan jalur panjang Windows&quot; dan &quot;Kecualikan backend Chloros dari pemindaian Microsoft Defender&quot;, keduanya dicentang secara default. Lihat [Langkah 1](#step-1--install-the-chloros-platform-package).
> - **Paket .deb Linux** menginstal Python SDK untuk sistem `python3` (sebelumnya, paket ini hanya menempatkan wheel di disk), meningkatkan ukuran buffer penerimaan soket kernel yang dibutuhkan oleh aliran kamera, dan menambahkan layanan backend yang berjalan sebagai pengguna Anda. Paket Jetson adalah `chloros_1.3.0_arm64_jp6.deb`, untuk JetPack 6, dengan CUDA PyTorch dari JetPack. Lihat [Langkah 1](#step-1--install-the-chloros-platform-package) dan [Langkah 2](#linux-deb).

---

## Arsitektur dalam 60 Detik

SDK ini merupakan lapisan Python tipis di atas backend Chloros (server Flask yang sama yang digunakan oleh antarmuka desktop dan CLI). Untuk otomatisasi, Anda mengimpor `chloros_sdk` dan memanggil metode tingkat tinggi; di balik layar, setiap panggilan akan diubah menjadi permintaan HTTP ke backend lokal di port 5000 — `http://127.0.0.1:5000/api/...` (sengaja tidak menggunakan `localhost`, yang pada sistem Windows akan diarahkan terlebih dahulu ke `::1` dan memakan waktu ~2 detik per permintaan saat terhubung ke backend yang hanya mendukung IPv4). Backend mengelola kumpulan perangkat keras — kamera, sensor DAQ, profil penyelarasan, buffer bingkai — sehingga skrip SDKdapat berjalan bersamaan dengan antarmuka pengguna grafis (GUI) tanpa bersaing untuk port serial atau bandwidth NIC.

Ada tiga antarmuka yang akan Anda gunakan:

1. **`ChlorosLocal` + fungsi bebas** (`process_folder`, `process_lattice_capture`) — Jalur pemrosesan gambar. Jalankan seluruh folder melalui kalibrasi / debayer / ekspor indeks dari satu panggilan Python.
2. **Penanganan koneksi cerdas** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Buka sesi backend persisten untuk perangkat keras langsung. Alur &quot;smart-prep&quot; yang sama seperti GUI: probe jaringan, pemilihan tingkat otomatis, PTP, seeding AE, konfigurasi pemicu GPIO.
3. **`ChlorosProject` / `open_project`** — Memuat proyek yang telah disimpan (folder yang berisi `cameras.json` + `sensors.json` + `project.json`), menghubungkan kamera, array, dan sensor cahayanya sekaligus, serta mengendalikan pengambilan gambar menggunakan handle yang diberi nama.

Permukaan 1 dan 2 **secara otomatis memulai backend lokal** jika belum ada yang mendengarkan (biner terintegrasi yang sama yang dijalankan oleh GUI/CLI) — sehingga skrip sederhana dapat berjalan dari shell baru tanpa Anda perlu memulai backend terlebih dahulu: `ChlorosLocal` (dan `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` dan `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` dan `analyze_array_network()` tidak pernah dijalankan: keduanya memerlukan backend yang sedang berjalan. Berikan argumen `auto_start_backend=False` untuk menonaktifkannya (misalnya saat mengarah ke backend jarak jauh, yang tidak pernah dijalankan). Lihat [Backend Auto-Start](#backend-auto-start). Surface 3 berperilaku berbeda: `open_project()` tidak menerima parameter `auto_start_backend`, dan `connect_all()` tidak pernah menjalankan backend — ia memeriksa `http://127.0.0.1:5000` sekali dan, jika tidak ada respons, beralih ke kontrol perangkat langsung (tanpa backend) `lattice_sdk`. Hal ini memerlukan runtime Arena SDK dan OpenCV (`cv2`) yang dapat diimpor ke dalam Python Anda, yang tidak diinstal oleh wheel SDK; tanpa keduanya, setiap kamera dan array yang disimpan akan dilaporkan sebagai kesalahan, beserta alasannya. Sensor cahaya dalam sebuah proyek tidak memiliki jalur tanpa backend: tanpa backend yang berjalan, setiap sensor yang disimpan akan dilaporkan sebagai kesalahan, beserta alasannya (lihat [`SensorHandle`](#sensorhandle)). Hanya `proj.process()` dan `stream(..., overlays=True)` yang secara malas membangun sebuah `ChlorosLocal()` (yang melakukan auto-start).

Ketiganya memerlukan otentikasi-gated: jalankan `chloros-cli login` sekali pada mesin, atau masuk melalui GUI desktop. Apa yang ditimbulkan oleh panggilan tanpa sesi yang valid bergantung pada antarmuka (lihat [Pengecualian](#exceptions)): `ChlorosAuthenticationError` dari `ChlorosLocal` (dan `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` dari panggilan smart-connect dan sesi-sesinya, `status: "error"` dengan `error_code` dari `analyze_array_network()`, serta entri di bawah `errors` dalam laporan `connect_all()`.

Persyaratan:
- Python 3.7+ (sesuai yang dinyatakan oleh paket; dikembangkan/diuji pada versi 3.10)
- Chloros Desktop terinstal secara lokal (biner backend disertakan dalam penginstal)
- Akun Chloros+ yang aktif pada **tingkatan Chloros+ berbayar apa pun (Copper atau lebih tinggi)**; tingkatan gratis**Iron**tidak memiliki akses SDK/CLI. Hal ini diberlakukan**dari sisi server****: setiap permintaan yang ditandai dengan SDK/CLI harus disertai sesi aktif dan paket berbayar. Paket di bawah Copper akan mendapatkan kode kesalahan `403` bersama `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` dari panggilan `ChlorosLocal`, sedangkan `ChlorosConnectError` berasal dari panggilan smart-connect). Pemanggil yang telah keluar dari sistem akan menerima kode `401` / `AUTH_REQUIRED` sebagai gantinya (`ChlorosAuthenticationError` dari `ChlorosLocal`, `ChlorosConnectError` dari panggilan smart-connect) — keduanya berbeda karena menjalankan kembali `chloros-cli login` akan memperbaiki `401` dan tidak dapat memperbaiki `403`, yang memerlukan peningkatan paket.
- Penggunaan offline didukung selama masa tenggang paket: tingkatan dibaca dari cache validasi server (5 menit) atau cache lisensi yang ditandatangani dan terikat pada mesin (30 hari untuk paket bulanan, hingga masa berlangganan berakhir untuk paket tahunan). Ketika masa tenggang tersebut berakhir, paket akan beralih ke versi gratis dan akses SDK/CLI akan dihentikan hingga mesin dapat terhubung ke server sekali. `chloros-cli status` (`GET /api/license-status`) tetap dapat diakses pada tingkatan gratis sehingga penyebabnya tetap terlihat.
- Windows 10/11 64-bit, **Ubuntu 22.04 LTS atau yang lebih baru**, atau Jetson (JetPack 6). Ubuntu 20.04**tidak** didukung: ketergantungan `.deb` berasal dari apa yang dihubungkan oleh backend, termasuk `libc6 (>= 2.35)`, dan Focal menyertakan glibc 2.31.

---

## Instalasi

Python SDK adalah lapisan Python tipis di atas backend Chloros. Kecuali jika Anda menjalankan backend di mesin lain (lihat [Mode Backend Jarak Jauh](#remote-backend-mode-pip-only-host-via-tunnel)), Anda perlu **menginstal paket desktop Chloros secara lokal** (penginstal Windows atau Linux `.deb`), yang mencakup sensor cahaya DAQ. Paket ini menyediakan biner backend, runtime Arena SDK untuk kamera LATTICE, dan paket kalibrasi.

Unduhan terbaru: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Langkah 1 — Instal paket platform Chloros

#### Windows (.exe)

1. Unduh `Chloros Installer 1.3.0.exe` dari halaman unduhan.
2. Jalankan penginstal dan ikuti panduan wizard. Jalur instalasi default adalah `C:\Program Files\Chloros\`.
3. Jalankan Chloros setidaknya sekali dan masuk menggunakan akun Chloros+ Anda.

Penginstal mengizinkan backend (`chloros-backend.exe`, tidak ada yang lain) melewati Windows Firewall pada setiap profil jaringan: masuk UDP, untuk aliran kamera, PTP, mDNS, dan aliran DAQ-E, serta masuk TCP hanya dari subnet lokal, untuk koneksi yang dibuat DAQ-E kembali ke komputer selama pembaruan firmware ([`update_firmware()`](#daq-e-device-settings)). Penghapus instalasi akan menghapus kedua aturan tersebut. API HTTP HTTP backend tetap berada di `127.0.0.1` dalam kedua kasus.

Grup **Pengaturan Sistem** pada wizard memiliki dua opsi, keduanya dicentang secara default:

| Opsi | Fungsinya | Menonaktifkan instalasi diam-diam |
| --- | --- | --- |
| **Aktifkan jalur panjang Windows** | Memungkinkan folder proyek berada di tingkat yang lebih dalam daripada batas jalur Windows sebesar 260 karakter. Jika jalur panjang dinonaktifkan (pengaturan default Windows), folder proyek yang terlalu dalam untuk produknya akan ditolak sebelum proses apa pun dilakukan — lihat [Pemecahan Masalah](#troubleshooting-pointers). | `/MERGETASKS="!longpaths"` |
| **Kecualikan backend Chloros dari pemindaian Microsoft Defender** | Menghentikan pemindaian real-timepada setiap file yang ditulis oleh penangkapan kamera. Pada laju penangkapan tinggi, pemindaian ini menghabiskan sekitar 30% dari laju tersebut: diukur pada ROI 512 pada susunan 9 kamera, penangkapan raw berkelanjutan berjalan pada 52,7 penangkapan/detik dengan pemindaian dan 77,7 penangkapan/detik dengan backend dikecualikan. | `/MERGETASKS="!defenderexclusion"` |

Perekaman berkelanjutan melaporkan beban Defender sebagai `defender_cores` (lihat [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Instal dengan `apt install ./…`, bukan `dpkg -i`: apt juga menginstal persyaratan Python SDK, yang direkomendasikan oleh paket tersebut, bukan diwajibkan (lihat [Langkah 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Untuk Jetson Orin / Orin NX / Orin Nano pada JetPack 6 (L4T R36). PyTorch-nya adalah build CUDA JetPack 6, sehingga pemrosesan GPU berjalan di GPU Orin; PyTorch ini menggunakan pustaka CUDA dan cuDNN yang diinstal oleh JetPack 6 (`sudo apt install nvidia-jetpack` pada gambar yang tidak memiliki pustaka tersebut).

#### Linux: buffer penerimaan dan layanan backend

Kedua paket .deb tersebut meningkatkan ukuran buffer penerimaan soket kernel yang dibutuhkan oleh aliran kamera LATTICE menjadi 32 MB (`/etc/sysctl.d/60-chloros-gige.conf`). Pada pengaturan default kernel, paket kamera akan dibuang oleh kernel dan array akan kehilangan frame serta laju frame; `chloros-cli lattice network` melaporkan status buffer saat kapasitasnya rendah. Lihat referensi CLI pada bagian [Host Linux — buffer penerimaan soket](cli-reference.md#linux-hosts--socket-receive-buffers).

Pada mesin tanpa antarmuka grafis, jalankan backend sebagai layanan **dengan akun pengguna Anda**, sehingga skrip dapat mendeteksi bahwa backend sedang berjalan (`connect_all()` tidak pernah memulai backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

Backend tersebut kemudian berjalan sebagai Anda: data tangkapan yang ditulisnya adalah milik Anda, dan backend tersebut menggunakan `chloros-cli login` Anda. `chloros-backend.service` root masih terpasang untuk mesin yang mengaktifkannya, tetapi setiap berkas yang ditulis oleh backend root menjadi milik root. Lihat referensi CLI pada bagian [Menjalankan backend sebagai layanan](cli-reference.md#running-the-backend-as-a-service-headless).

Backend yang dijalankan oleh layanan bersifat bersama dan tidak pernah dihentikan oleh klien: skrip SDK dan `chloros-cli` menggunakannya sebagaimana adanya (`ChlorosLocal.shutdown_backend()` hanya menghentikan backend yang dimulai sendiri oleh `ChlorosLocal`), dan `GET /api/status` menamai unit tersebut sebagai `service`. Mulai ulang dengan `sudo systemctl restart chloros-backend@$USER.service`. Memperbarui paket akan menghentikan layanan backend selama proses instalasi, sehingga skrip yang sedang berjalan kehilangan akses ke kamera dan sensornya, lalu memulai kembali semua yang diaktifkan; menghapus paket tersebut menghentikan layanan dan menonaktifkan yang diaktifkan. Hanya satu backend yang berjalan pada satu waktu, jadi aktifkan satu unit saja: unit kedua akan menolak untuk dimulai dan tetap dalam status `failed` (status keluar 98, tidak dimulai ulang), dan pesan `systemctl status` menjelaskan alasannya. Untuk mengganti unit, nonaktifkan unit yang tidak Anda gunakan.</id></sn>

### Langkah 2 — Instal Python SDK

**Penginstal Chloros menyertakan file wheel SDK yang sesuai.** Setiap penginstal Windows dan paket .deb Linux akan menempatkan berkas `chloros_sdk-X.Y.Z-py3-none-any.whl` di disk yang persis sesuai dengan versi GUI / CLI / backend. Anda tidak perlu terus-menerus memeriksa PyPI untuk memastikan keselarasan.

#### Windows

Penginstal secara otomatis menjalankan `pip install` terhadap file wheel yang disertakan menggunakan Python sistem Anda (disarankan menggunakan peluncur `py.exe`, jika tidak tersedia akan menggunakan `python -m pip`). Tidak diperlukan tindakan apa pun — `import chloros_sdk` akan berfungsi di lingkungan Python Anda setelah instalasi berhasil. Jika tidak ada Python di sistem, penginstal akan melewati langkah ini tanpa pemberitahuan, dan GUI + CLI tetap berfungsi.

#### Linux (.deb)

Kedua paket .deb (x86_64 dan Jetson) menginstal SDK untuk sistem `python3`, sehingga `import chloros_sdk` dapat berfungsi setelah instalasi, tanpa memerlukan pip maupun koneksi jaringan. Paket-paket (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api`, dan pemuat proyek) dimasukkan ke dalam `/usr/lib/python3/dist-packages`, dan perintah `chloros-mcp` dari SDK (server [MCP](#mcp-server-ai-assistants)) ke dalam `/usr/bin`; semuanya dimiliki oleh paket tersebut: pembaruan akan menggantikannya, dan menghapus paket akan menghapusnya.

Persyaratan SDK itu sendiri, yaitu `python3-numpy` dan `python3-requests`, **disarankan, bukan wajib** (GUI, CLI, dan backend bersifat mandiri): `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` pada Jetson) menginstalnya, sedangkan `dpkg -i` tidak. Hasil instalasi melaporkan versi SDK yang diimpor oleh `python3`, persyaratan apa pun yang hilang, serta keberadaan `chloros_sdk` di lokasi lain pada jalur `python3` yang menimpa salinan paket tersebut (seperti yang diinstal sebelumnya dengan `pip install --user`).

Di dalam virtualenv, yang tidak dapat mengakses paket sistem, instal wheel yang disertakan ke dalamnya:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Untuk deployment Jetson yang terisolasi (air-gapped), proses ini sepenuhnya offline — SDK dan wheel sudah ada di disk.

#### PyPI Publik

Untuk host yang hanya menggunakan pip (tidak ada paket desktop Chloros yang terinstal; SDK kemudian mengendalikan backend di mesin lain, lihat [Mode Backend Jarak Jauh](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

PyPI menyediakan rilis stabil terbaru. Pastikan versi SDK-nya sama dengan versi backend Chloros yang dikendalikannya (lihat [Penomoran Versi &amp; Kompatibilitas](#versioning--compatibility)). Pembantu DAQ-M `DLS` juga memerlukan `bleak`: `pip install "chloros-sdk[camera]"` (atau `pip install bleak`).

#### Verifikasi

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Diperlukan langganan Chloros+.** Semua panggilan SDK memerlukan login Chloros+ yang aktif. Jalankan `chloros-cli login user@example.com 'YourPassword'` sekali per mesin; kredensial disimpan dalam cache di `~/.chloros/`.

### Apakah Saya Membutuhkan Paket Desktop?

Paket pip saja **tidak** cukup untuk sebagian besar alur kerja. Berikut adalah kebutuhan masing-masing antarmuka SDK:

| Antarmuka SDK | Membutuhkan Paket Desktop? | Mengapa |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Ya**(untuk memulainya)**/ Tidak** (backend sudah dapat diakses) | Gunakan backend yang sedang berjalan (dari aplikasi desktop, atau skrip lain yang telah dijalankan); jika tidak, jalankan secara otomatis yang sudah terinstal: `/usr/lib/chloros/chloros-backend` (Linux) atau `<install folder>\backend\chloros-backend.exe` (sistem Windows; `C:\Program Files\Chloros\backend\` secara default, folder instalasi khusus dibaca dari catatan penginstal). Berikan nama sendiri dengan menggunakan `backend_exe=`. |
| `chloros-mcp` ([server MCP](#mcp-server-ai-assistants)) | **Ya**(untuk memulai satu)**/ Tidak** (backend sudah dapat diakses) | Dibangun di atas `ChlorosLocal`, jadi sama dengan baris di atas. |
| `read_image_audit_tags` | **Tidak** | Membaca metadata dari gambar yang telah diproses (hanya perpustakaan standar Python). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Ya**(lokal)**/ Tidak**(jarak jauh) | Klien HTTP murni melalui backend. Backend lokal → memerlukan paket desktop. Backend jarak jauh → `backend_url=`**melalui terowongan** (lihat Mode Backend Jarak Jauh — backend bawaan hanya mengikat loopback). |
| `ChlorosProject` / `open_project` | **Ya** | Mengakses proyek yang disimpan melalui backend. |
| Kelas LATTICE langsung (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Ya** | Membutuhkan runtime Arena SDK, yang disertakan dalam paket desktop, serta OpenCV (`cv2`) di lingkungan Python Anda, yang tidak diinstal baik oleh paket desktop maupun SDK (`pip install opencv-python`). Jika tidak, `CAMERA_AVAILABLE` akan menjadi `False` saat diimpor. `DLS` juga memerlukan `bleak`. |

### Mode Backend Jarak Jauh (host hanya pip, melalui terowongan)

> **Backend yang terinstal tidak dapat dijangkau melalui LAN.** Backend tersebut hanya mendengarkan
> pada loopback (`127.0.0.1` / `::1`), sehingga `backend_url="http://<lan-ip>:5000"`
> **tidak dapat menjangkau Chloros yang terinstal**. Untuk mengoperasikan backend di
> mesin lain, alihkan port loopback-nya sendiri dan arahkan SDK ke
> terowongan:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Host headless / CI / robotika dapat mempertahankan satu mesin dengan instalasi desktop lengkap sebagai &quot;server Chloros&quot; dan `pip install chloros-sdk` di tempat lain — tetapi transportasi di antara keduanya adalah terowongan yang diatur pengguna di atas, bukan URL LAN langsung.

> **`ChlorosLocal` pada host yang hanya menggunakan pip.** `ChlorosLocal(backend_url=BACKEND)` terhubung ke backend yang dapat dijangkau tanpa instalasi desktop: ia hanya mencari biner backend lokal saat harus MEMULAI satu (tidak ada yang merespons dan `auto_start_backend=True`). Hingga versi 1.2.0, sistem terlebih dahulu mencari biner tersebut dan memunculkan pesan kesalahan `ChlorosBackendError` (“Backend Chloros tidak ditemukan…”) pada host yang hanya menggunakan pip, meskipun backend tersebut dapat dijangkau. Folder-folder yang Anda berikan (`process_folder`, `import_images`) diperiksa di mesin ini dan dibaca oleh backend, sehingga dengan backend jarak jauh, nama folder tersebut harus sama di kedua sisi, seperti pada drive bersama.

### Alur Kerja Khusus DAQ (host yang hanya menggunakan pip)

`pip install chloros-sdk` saja tidak dapat mengendalikan sensor DAQ: sensor DAQ selalu dikendalikan melalui backend Chloros dengan `connect_daq_sensor()`:

- **Sensor pada mesin ini:** instal paket desktop Chloros di sini. `connect_daq_sensor()` akan memulai backend-nya jika belum ada yang berjalan.
- **Sensor di mesin lain:** mesin tersebut memerlukan paket desktop; akses backend-nya melalui terowongan dalam [Mode Backend Jarak Jauh](#remote-backend-mode-pip-only-host-via-tunnel). Backend tersebut membuka sensor di mesinnya sendiri, sehingga DAQ-U terhubung ke mesin tersebut, DAQ-M berada dalam jangkauan Bluetooth-nya, dan DAQ-E berada di jaringan yang dapat dijangkau olehnya.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Bagaimanapun juga, backend menerapkan skema Chloros+: masuk ke mesin yang menjalankan backend (`chloros-cli login`, atau aplikasi desktop) dengan paket berbayar apa pun (Copper atau yang lebih tinggi). Backend yang tidak login dan paket di bawah Copper keduanya akan memicu `ChlorosConnectError` yang membawa alasan dari backend (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Panduan Cepat

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
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

## Indeks API Tingkat Atas

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
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Pemrosesan Gambar — `ChlorosLocal`

Kelas pipa utama. Memulai backend saat diinisialisasi (jika tidak ada yang merespons dan `auto_start_backend=True`), membuat/mengonfigurasi proyek, memantau kemajuan, dan mengembalikan ringkasan setelah eksekusi.

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
| `open_project(project_path, relink=None)` | Membuka proyek yang telah disimpan (foldernya, atau namanya di dalam folder proyek) dan menjadikannya proyek yang terbuka. Jalur proyek yang dipindahkan diarahkan ulang ke folder tempatnya berada saat ini; `relink` menyebutkan nama folder tempat file-file lainnya yang hilang berada saat ini. Mengembalikan jawaban backend dengan `source_files`; menimbulkan `ChlorosProjectFilesMissingError` jika tidak ada yang dapat ditemukan. Lihat **Proyek yang dipindahkan** di bawah ini. |
| `import_images(folder_path, recursive=False)` | Mengimpor gambar RAW/TIF/JPG/DNG **dan `.daq` rekaman sensor cahaya**. Mengembalikan `count` (gambar) dan `scan_count` (rekaman). Hanya memberikan peringatan jika folder tersebut tidak berisi keduanya. |
| `export_light_sensor(daq=True, csv=True)` | Menulis data yang telah dikalibrasi `.daq` + `.csv` untuk setiap rekaman sensor cahaya pada proyek, ke dalam `<project>/Light Sensor/`. Lihat [Rekaman Sensor Cahaya](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Atur kenop pemrosesan. `indices` adalah nama preset (`NDVI`, `GNDVI`, `OSAVI`, …), masing-masing ditambahkan ke proyek sesuai dengan rentang spektral kameranya; nama yang bukan preset, atau yang tidak dapat dipenuhi oleh rentang spektral kamera (`NDRE` memerlukan RE dan NIR, yang tidak disediakan oleh satu kamera pun), akan diberi nama `UserWarning` dan tidak memiliki gambar indeks. `daq_log_path` merujuk pada rekaman sensor cahaya (berupa berkas `.daq` atau `.csv`, atau folder yang berisi berkas-berkas tersebut) yang digunakan sebagai acuan untuk mencocokkan setiap bingkai, baik di LATTICE maupun di Survey3, alih-alih rekaman yang diimpor bersama gambar-gambar tersebut; jika jalur tersebut tidak ada atau tidak berisi rekaman, maka rekaman yang diimpor akan digunakan. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Jalankan pipeline. Mengembalikan `{"status": "complete", "async": False}`, ditambah kunci `summary` jika backend menyediakannya — lihat [Ringkasan Pasca-Eksekusi &amp; Petunjuk](#post-run-summary--hints) — `export_check_failure` jika ekspor dari eksekusi tidak dapat diverifikasi, dan `image_failure` jika ada gambar yang hilang. `wait=False` mengembalikan `{"status": "started", "async": True}` sekaligus. |
| `get_config()` / `get_status()` / `status()` | Periksa status backend. |
| `logout()` | Keluar: menghapus kredensial yang tersimpan dalam cache dan melepaskan slot perangkat mesin ini (`device_deregistration` dalam hasilnya). Proses yang sedang berjalan tidak dihentikan: proses tersebut selesai dalam mode yang sama saat dimulai (`run_in_progress: true` dan `run_mode` dalam hasilnya), dan proses berikutnya menggunakan pemrosesan standar. |
| `shutdown_backend()` | Hentikan backend jika instance ini yang memulainya; backend yang ditemukan sedang berjalan dibiarkan berjalan. Meninggalkan blok `with` atau instance yang sedang dibersihkan oleh garbage collector tidak akan menghentikannya: backend akan terus berjalan untuk skrip berikutnya dan akan berhenti sendiri setelah 30 menit jika tidak ada yang menggunakannya. Backend diminta untuk dimatikan, sehingga ia melepaskan kamera dan sensor cahayanya serta menyelesaikan perekaman apa pun terlebih dahulu; backend tersebut baru akan dihentikan jika masih berjalan 10 detik kemudian. |
| `discover_cameras()` | Temukan kamera LATTICE **melalui backend instance ini** (`/api/camera/discover`). Mengembalikan daftar kamus (`serial`, `model`, `ip`, …) — bentuknya sama seperti yang terlihat di GUI/CLI. Daftar kosong jika tidak ditemukan atau backend tidak dapat dijangkau. |
| `camera_capture(output_dir, format="tiff", **settings)` | Mengambil satu bingkai**melalui backend**(dimulai secara otomatis oleh handle ini) sehingga mendapatkan persiapan yang sama seperti yang terlihat di GUI/CLI (default 12-bit, penggunaan ulang pool, metadata kalibrasi tertanam). Tentukan target dengan `serial=` atau `device_index=`; lewatkan `exposure`/`gain`/`pixel_format`/`preset` sebagai `**settings`. Mengembalikan kamus metadata lama (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Menghasilkan bingkai pratinjau yang digabungkan dengan overlay dari kamera yang digabungkan — klien MJPEG ringan melalui rute `/api/camera/<serial>/stream-annotated` di backend (zebra / grid / crosshair / histogram / peaking / spot yang digambar di sisi server). `decode=True` menghasilkan array BGR dan memerlukan OpenCV (`cv2`), yang tidak diinstal oleh SDK; `False` menghasilkan byte JPEG mentah. Juga dapat diakses per proyek sebagai `ChlorosProject.stream(overlays=True)`. |

**Proses yang sedang berjalan mempertahankan proyeknya.** Selama proses pemrosesan sedang berlangsung — baik dari skrip ini, aplikasi, `chloros-cli process`, maupun `chloros-mcp` — backend menolak untuk membuat, membuka, mengimpor ke, atau mengubah pengaturan pemrosesan suatu proyek (409 HTTP `PROCESSING_IN_PROGRESS`): `create_project()` dan `import_images()` memicu `ChlorosProcessingError`, dan `configure()` memicu `ChlorosConfigurationError`, serta `process()` dengan mode yang berbeda dari mode yang sedang berjalan memicu `ChlorosProcessingError` (suatu eksekusi mempertahankan mode saat dimulai), masing-masing disertai pesan dari backend. Sebuah rekaman yang disimpan ke dalam proyek kemudian digabungkan ke dalamnya saat eksekusi berakhir (balasannya, `project_add`, menyatakan hal tersebut). Tunggu hingga proses berjalan selesai (`get_status()`), lalu lanjutkan.**Proyek yang dipindahkan.** `project.json` dari sebuah proyek menamai setiap berkas sumber (setiap gambar dan rekaman sensor cahaya) berdasarkan jalur lengkapnya, sehingga memindahkan folder proyek — atau seluruh folder proyek ke drive lain — membuat nama-nama tersebut tetap merujuk ke lokasi lama. Setiap kali dibuka, sistem akan memperbaiki apa yang bisa diperbaiki secara otomatis: berkas yang sebelumnya berada di dalam folder proyek akan dicari di folder tempat proyek tersebut berada sekarang, dan setiap jalur yang ditemukan di sana akan diarahkan ulang dan disimpan (`project.json` terlebih dahulu dicadangkan sebagai `project.json.before-relink-<date>-<time>`). Jalur hanya akan diarahkan ulang ke berkas yang memang ada di sana. File yang berada di luar folder proyek — sesi hub, impor Add Folder — hanya ditemukan di `relink=<the folder they are in now>` (atau folder mana pun di atasnya). `open_project()` mengembalikan `source_files`: `total`, `found`, `missing`, `missing_roots` (folder-folder tempat berkas yang hilang berada), `missing_outside_project` dan `relink` (apa yang dialihkan, dari mana ke mana). Sebuah proyek dengan beberapa berkas yang hilang akan terbuka bersama sisanya; proyek yang tidak menemukan berkas apa pun tidak akan terbuka, dan `open_project()` memicu `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). `project open --relink` dan `chloros-mcp` pada CLI`open_project` (`relink_folder`) pada CLI juga melakukan hal yang sama.

Gunakan sebagai pengelola konteks untuk pembersihan yang terjamin:

```python
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

### Perekaman Sensor Cahaya — dikalibrasi `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E dapat direkam **tanpa** paket kalibrasinya. Itulah
yang dilakukan perekam publik [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) lakukan secara default: mereka menulis data mentah hitungan sensor dan menandai
file tersebut sehingga Chloros mengambil kalibrasi pabrik sensor tersebut **berdasarkan nomor seri** — cache lokal
terlebih dahulu, kemudian MAPIR Cloud — dan menerapkannya saat impor.

Chloros menyimpan hasilnya kembali sebagai dua produk per perekaman, di bawah
`<project>/Light Sensor/`:

| Produk | Apa itu |
| --- | --- |
| `<name>_calibrated.daq` | Arsip yang dapat diproses ulang — skema yang sama dengan rekaman langsung, kini mencantumkan paket yang menghasilkannya. Mengimpornya kembali **tidak** akan mengkalibrasinya untuk kedua kalinya. |
| `<name>_calibrated.csv` | Irradian spektral dalam W/m²/nm pada kisi panjang gelombang sensor itu sendiri, satu baris per pembacaan, ditambah kolom fotometrik (daya total, lux fotopik/skotopik, PPFD dan pembagiannya ke biru/hijau/merah, panjang gelombang puncak). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Hanya untuk sensor tanpa bundel (DAQ-A).** Hitungan spektral mentah dari sensor — *bukan* irradiance. Lihat di bawah. |

`<name>` adalah nama file perekaman itu sendiri. Dua perekaman di folder berbeda dapat berbagi
satu — pembacaan per pengambilan (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) yang disimpan dalam
milidetik yang sama oleh dua kamera — dan masing-masing tetap mendapatkan hasilnya sendiri: yang pertama mempertahankan
`<name>`, sedangkan yang berikutnya menambahkan folder yang membedakannya **setelah** nama
(`<name>__<capture>_daq_calibrated.daq`), sehingga tanggal dan waktu perekaman tetap berada di depan.

`process()` melakukan ekspor ini sebagai salah satu tahapnya. Proses ini **tidak** memerlukan citra:
sensor cahaya yang diterbangkan sendiri merupakan alur kerja yang utuh, dan proyek semacam itu secara desain tidak memiliki
citra sama sekali.

**Rekaman DAQ-A diekspor sebagai hitungan mentah.** Keluarga DAQ-A sudah ada sebelum sistem
bundel per-serial dan tidak memiliki bundel yang perlu diambil — alih-alih, alat ini dikalibrasi di lapangan menggunakan
target reflektansi, itulah sebabnya alat ini tidak pernah membutuhkannya. Rekaman-rekaman tersebut diekspor
dengan awalan `_raw`, bukan `_calibrated`: nama file yang berbeda, bukan bendera
di dalam file, karena klaim tersebut harus tetap utuh saat dikirim melalui email sebagai nama file mentah. Header
Header `.csv` mencantumkan `raw spectral sensor counts (NOT irradiance)` dan memperingatkan bahwa
nilai-nilai tersebut dapat dibandingkan **di dalam** file — persis seperti yang digunakan oleh kalibrasi berbasis target
— dan bukan antar sensor. Kolom fotometrik yang bergantung pada daya (daya total,
lux fotopik/skotopik, PPFD) menghasilkan nilai **NULL**, bukan hasil integrasi dari hitungan.

DAQ-U / DAQ-M / DAQ-E yang bundelnya sama sekali tidak dapat diambil tetap **dilewati**,
bukan ditulis dalam bentuk mentah: dalam kasus tersebut, bundelnya memang ada dan &quot;sambungkan kembali dan proses ulang&quot; adalah saran yang tepat.

Perekaman **v1.01 / v1.02** versi lama (ditulis oleh DAQ-A-SD) tidak memiliki epoch per pembacaan,
hanya waktu penulisan berkas. Pencocok gambar↔sinar turun masih menolaknya — mencocokkan
bingkai dengan waktu penulisan akan menghasilkan kesalahan yang tidak terlihat — tetapi eksportir membacanya, dan
file CSV mencetak `clock=daq_created_on` sehingga produk tersebut menyatakan jam mana yang digunakan.

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
kalibrasi dalam berkas) dilaporkan di bawah `skipped` **bersama alasannya**. File tersebut tidak pernah
ditulis sebagai file &quot;terkalibrasi&quot; yang berisi data mentah — sambungkan ke internet dan
jalankan ulang, lalu proses ekspor akan selesai.

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

### Ringkasan Pasca-Eksekusi &amp; Petunjuk

Setelah selesai, `process()` mengambil `GET /api/processing-summary` dan melampirkan isinya sebagai `result["summary"]`. Pengambilan ini dilakukan dengan upaya terbaik dan tidak pernah menghalangi pengembalian yang berhasil — jika ringkasan tidak tersedia, `process()` akan kembali ke bentuk `{"status": "complete", "async": False}` biasa. Setiap entri dalam `summary["hints"]` — kalimat lengkap dengan saran perbaikan, misalnya, mengapa suatu eksekusi menghasilkan output nol — juga dikeluarkan kembali sebagai Python `UserWarning`, sehingga eksekusi dengan output nol dapat mendiagnosis dirinya sendiri meskipun Anda tidak pernah memeriksa kamus:

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
| `models` | Kelompok kamera dalam eksekusi. |
| `images_in_groups` | Gambar sumber di seluruh kelompok tersebut. |
| `targets_found` | Target reflektansi yang terdeteksi. |
| `images_calibrated` | Gambar yang digunakan oleh proses tersebut untuk menghasilkan produk reflektansi. |
| `images_failed` | Gambar yang gagal dan tidak diekspor. |
| `exported_files` | **File produk gambar yang dihasilkan oleh proses**: setiap produk (debayered, pratinjau, radiance, reflektansi, indeks, render target), dalam setiap format. |
| `reflectance_files` | Di antara produk reflektansi tersebut (`Reflectance_Calibrated_Images`), baik yang dikalibrasi oleh target maupun oleh irradiance sensor cahaya (DAQ). Proses tanpa target yang tetap menghasilkan data reflektansi akan mencantumkannya dalam petunjuk; hanya proses yang tidak menghasilkan apa pun yang diberi tahu bahwa keluaran reflektansinya hilang. |
| `clamp_masks_written` | [Masker penjepit unmix](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) yang tertulis di sampingnya — merupakan data pendamping, bukan hasil akhir, sehingga tidak termasuk dalam `exported_files` (CLI juga menghitungnya secara terpisah dengan cara yang sama). |
| `captures_left_out` | Frame mentah di folder `captures` proyek yang tidak diambil oleh proses ini. Sebuah eksekusi menunggu saat dimulai untuk data yang direkam sebelumnya, sehingga nilainya 0 kecuali jika perekaman disimpan selama eksekusi atau waktu tunggu habis. `summary["captures_left_out_note"]` menjelaskan apa yang harus dilakukan terhadapnya. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Rekaman sensor cahaya, dihitung secara terpisah dengan sengaja — data ini berasal dari tahap yang berbeda dan ada pada eksekusi yang sama sekali tidak menghasilkan gambar, sehingga menggabungkannya akan membuat eksekusi yang hanya menggunakan DAQ tampak seolah-olah telah mengekspor gambar. |

Di sampingnya: `summary["output_dirs"]` (setiap direktori yang ditulis),
`summary["light_sensor_export"]`, `summary["stopped"]` (bernilai benar jika pengguna menghentikan
proses, sehingga hitungan parsial tidak dianggap sebagai proses yang telah selesai namun hasilnya kurang), dan
`summary["groups"]` (rincian per kelompok).

`exported_files` dicatat oleh pipa **saat menulis**, bukan dipindai dari
objek gambar proyek setelahnya. Strategi paralel dan GPU membangun objek gambar
mereka sendiri (dalam subproses pekerja untuk jalur GPU), sehingga pemindaian lama melaporkan
`0 file(s) written` untuk setiap eksekusi semacam itu dan kemudian mengeluarkan petunjuk zero-exports — pada eksekusi
di mana semuanya telah berjalan dengan baik. Jika Anda membuat skrip berdasarkan angka ini, eksekusi paralel yang sehat kini
melaporkan jumlah yang tidak nol.

Laporan pengabaian sensor cahaya mencantumkan alasan yang sebenarnya ditetapkan oleh pembaca untuk setiap berkas —
skema yang tidak dapat dibaca, bundel yang hilang, kesalahan penulisan — **didesduplikasi**, sehingga dua puluh berkas
yang dilewati karena satu penyebab dicatat sebagai satu penyebab, bukan dua puluh pengulangan dari penyebab tersebut.

> **`process()` tidak terpicu ketika suatu proses tidak menghasilkan gambar apa pun.** Ini adalah salah satu dari dua hal di mana SDK dan
> CLI secara sengaja berbeda: `chloros-cli process` menganggap &quot;produk diminta, tetapi tidak ada yang
> ditulis&quot; sebagai kegagalan dan keluar dengan nilai bukan nol, sedangkan SDK kembali secara normal dan melaporkan
> kondisi tersebut melalui `summary` / petunjuk. Jika alur kerja Anda harus berhenti pada eksekusi kosong, periksalah
> sendiri — periksa `summary` (atau hitung file di bawah folder proyek) daripada mengandalkan
> tidak adanya pengecualian. Penyebab umumnya adalah folder input yang tidak dikenali sebagai
> penangkapan dan produk yang dilewati karena tidak berlaku untuk kamera yang ada (misalnya, radian dari kamera RGB saja
>).
>
> `process()` juga tidak terpicu ketika ekspor dari suatu eksekusi tidak dapat diverifikasi: sistem akan memberikan peringatan dan menetapkan
> `result["export_check_failure"]`. Produk-produk tersebut tetap ditulis. Untuk menghentikan proses pada kondisi ini, periksa kunci tersebut. Sebuah run yang
> kehilangan gambar juga sama: sistem akan memberikan peringatan dan menetapkan `result["image_failure"]`, yang akan membuat `chloros-cli process`
> keluar dengan nilai tidak nol.

### Fungsi-Fungsi Praktis

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
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

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Setiap proses berjalan di dalam `with ChlorosLocal(...)`-nya sendiri; backend yang harus dijalankan tetap berjalan saat proses tersebut kembali, untuk panggilan berikutnya, dan menghentikan dirinya sendiri setelah 30 menit jika tidak ada yang menggunakannya. Keduanya tidak menerima `auto_start_backend`, `radiometric_output`, atau `daq_log_path`; gunakan `ChlorosLocal` dan `configure()` untuk kasus-kasus tersebut. `process_lattice_capture()` tidak menerima `camera`, `debayer`, atau `ppk`: fungsi ini melewati deteksi panel dan menggunakan debayer standar.

### Nilai yang Didukung

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
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

Tingkat ekspor multispektral LATTICE (M3C/M3M) dari pipeline `process` — `reflectance` (default), `radiance`, `sensor-response`, atau `all` (setiap mode yang berlaku per gambar) — menentukan produk **&quot;Ekspor radian&quot;**dan**&quot;Ekspor reflektansi&quot;**: `radiance` hanya mengekspor radiance, `reflectance` hanya reflektansi, `sensor-response` tidak mengekspor keduanya (hanya produk debayered dan pratinjau), `all` mengekspor keduanya. Produk hasil debayering dan pratinjau tidak dapat diubah pada tingkat ini. `configure()` memiliki kata kunci khusus untuk itu:

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

Jalan keluar lanjutan — menulis produk melalui `custom_settings` — masih berfungsi, tetapi ingatlah bahwa ini akan menggantikan seluruh blok pengaturan (lihat peringatan di bawah). Tulis kunci `"Export <product>"`: pengaturan `"Export radiance"` / `"Export reflectance"` akan mengesampingkan kunci `"Radiometric output"` yang lebih lama, sehingga kunci tersebut sendiri tidak mengubah apa pun.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (pengaturan default) membagi radian kamera dengan **sinyal DAQ downwelling yang disesuaikan dengan cap waktu**, yang diselesaikan secara otomatis dari data tercatat `.daq` (DAQ-U/M/E)**atau produk tipe `.csv` asli DAQ-M**yang ditemukan bersama citra; setiap paket kalibrasi per-kamera atau DAQ yang tidak tersedia secara lokal akan**diunduh secara otomatis dari AWS** pada penggunaan pertama. Antarmuka CLI menampilkan ini sesuai dengantombol pengalih produk tipe pada `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**Siapa pemilik pembacaan ini.**Sebuah `.daq` yang disimpan oleh penangkapan adalah pembacaan dari kamera yang digambarkannya — yang disebutkan dalam `<file>.daq.cameras.json` sidecar di sampingnya (rekaman tunggal, sinkron, dan array; burst mentah mencantumkan pembacaan setiap anggotanya dalam manifesnya) — dan**hanya cocok dengan gambar dari kamera-kamera tersebut**. Oleh karena itu, kamera mandiri yang tidak memiliki pembacaan sendiri tidak lagi meminjam DAQ kamera lain; ikat DAQ ke kamera tersebut, atau rekam satu. Rekaman DAQ itu sendiri (`chloros-cli daq pool-record`, atau perekaman yang dimulai di antarmuka pengguna), atau file `.daq` apa pun tanpa file sidecar tersebut (termasuk perekaman lama), tetap berfungsi untuk semua kamera.**Kamera OCN di bawah sinar matahari.** Setiap paket kalibrasi OCN dibuat di bawah cahaya pita sempit, satu LED per pita, sedangkan sinar matahari mencampurkan pita-pita tersebut. Chloros 1.3.0 menerapkan koreksi berbasis sinar matahari secara massal setelah pemisahan pita, yang meningkatkan produk radian, reflektansi, dan indeks OCN di bawah sinar matahari. Koreksi ini berjalan di mana pun proses pemisahan pita dilakukan — `process()`, tangkapan gambar, `grab()` / `frame_stream()`, dan tampilan langsung — untuk paket OCN yang mendukungnya; kamera RGN tidak mengalami perubahan. Gambar OCN yang diambil dengan versi sebelumnya juga akan mendapatkan fitur ini saat diproses di komputer yang menyimpan kalibrasi yang digunakan saat pengambilan gambar (Chloros menyimpan kalibrasi kamera setelah kamera terhubung); jika tidak, gambar tersebut diproses seperti sebelumnya. Oleh karena itu, nilai radian, reflektansi, dan indeks OCN berbeda dari versi 1.2.0.

> `custom_settings` **menggantikan** seluruh blok pengaturan yang dihitung (secara desain, ini melewati kata kunci lain dan validasi dari `configure()`). Saat menggunakannya, sertakan setiap kunci `Project Settings` yang Anda perlukan, seperti pada contoh di atas.

#### Membaca piksel indeks (indeks NDVI dan indeks vegetasi lainnya)

**Raster indeks bilangan bulat merupakan penyesuaian kontras per gambar, bukan peta dengan rentang −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, dan `JPG (8-bit)` — termasuk file TIFF 8-bit yang dihasilkan dari pengambilan gambar 8-bit — menyimpan

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

di mana `IndexMin` / `IndexMax` adalah **nilai indeks terendah dan tertinggi yang terbatas dari bingkai tersebut**. DN 0 bukanlah −1, dan skala penuh bukanlah +1.

Stempel Chloros pada kedua titik ujung tersebut agar pemetaan tetap dapat dibalik:

| Tag | Arti |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — konvensi di bawah ini |
| `Chloros:IndexMin` | nilai indeks yang disimpan sebagai DN 0 |
| `Chloros:IndexMax` | nilai indeks yang disimpan sebagai skala penuh |

Baca nilai-nilai tersebut dan balikkan, lalu normalisasikan DN yang disimpan menjadi 0…1 berdasarkan nilai maksimum tipe data (dtype) yang disimpan terlebih dahulu, sehingga satu ekspresi dapat mencakup semua format bilangan bulat:

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

> **`IndexMin == IndexMax`** berarti setiap piksel pada bingkai tersebut memiliki nilai indeks yang sama. Raster diisi pada skala tengah dan tidak ada yang perlu dibagi — ambil titik akhir.

> **Tidak adanya tag `Chloros:Index*` berarti tidak ada pemetaan yang dinyatakan.** Ekspor LUT adalah hasil render berwarna yang piksel-pikselnya berupa warna gradien yang disesuaikan dengan jendela ambang batas LUT Anda, dan bingkai tanpa piksel indeks terbatas tidak memiliki rentang yang dapat dinyatakan. Jangan mengasumsikan adanya pemetaan dalam kedua kasus tersebut — aturan yang sama berlaku seperti pada kasus `PixelScale` yang hilang.**Karena peregangan dilakukan per gambar, dua bingkai dari satu penerbangan tidak dapat dibandingkan sebagai DN**— masing-masing memiliki titik akhir sendiri, sehingga nilai NDVI yang sama akan berada pada DN yang berbeda di gambar yang berbeda. Untuk pekerjaan kuantitatif di seluruh penerbangan (statistik zonal, ambang batas tetap, rangkaian waktu), ekspor**`TIFF (32-bit, Percent)`**: raster indeks ditulis di sana sebagai float32**nilai indeks itu sendiri**, tanpa penskalaan ulang dan tanpa memerlukan tag. Terlepas dari nama formatnya, produk indeks tidak dikalikan ×100 — penskalaan tersebut hanya berlaku untuk reflektansi bertipe data integer.

#### Masker pembatas pemisahan spektral — piksel mana saja yang nilai band-nya dibatasi

Pemisahan spektral 3×3 dapat membuat nilai band menjadi negatif — di mana cahaya hampir tidak mengandung band tersebut, atau di mana adegan berada di luar apa yang dijelaskan oleh matriks pemisahan kamera. Itu bukanlah nilai nyata, sehingga ekspor menetapkannya ke 0 — dan tampilan langsung juga menampilkan nilai 0 yang sama. Sebuah band dapat dibatasi pada beberapa piksel atau hampir seluruhnya; masker mencatat keduanya. `Chloros:Band{R,G,B}UnmixClampFraction` menunjukkan seberapa besar bagian frame yang mengalami hal tersebut; masker pembatasan menunjukkan **piksel mana saja**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— di dalam folder `clamp_masks/`**di samping** folder produk, bukan di dalamnya (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — untuk setiap produk radiance, reflectance, dan indeks di mana setidaknya satu piksel mengalami pembatasan. `uint8` pita tunggal, pada kisi piksel produk itu sendiri (termasuk orientasi, ROI, binning, dan distorsi penyelarasan array sudah termasuk). Setiap masker mencantumkan nama kamera produknya (`Make` / `Model` / serial), waktu pengambilan gambar, dan tag lensa, serta mencantumkan pita yang diwakili oleh bit-bitnya (`XMP-Camera:BandName`, bit 0 terlebih dahulu); `ImageDescription` dan `Software` menandainya sebagai masker clamp, dan tidak mengandung kalibrasi.
- **Bit**: bit 0 = pita pada saluran R produk (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Bit yang aktif berarti nilai piksel pada band tersebut telah dibatasi menjadi 0 — perlakukan nilai tersebut, serta indeks apa pun yang dibuat darinya, sebagai data dengan tingkat kepercayaan rendah.
- **`Chloros:UnmixClampMask`** dicantumkan pada setiap produk yang telah menjalankan proses pemisahan: jalur masker relatif terhadap folder produk itu sendiri (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, selalu dipisahkan dengan `/`— pisahkan — gabungkan dengan folder gambar untuk membuka masker), atau `none` jika tidak ada piksel yang dibatasi. Tidak adanya tag berarti produk tersebut tidak pernah menjalankan proses unmix (kamera RGB atau mono, level raw/debayered).
- Hasil perekaman menyertakannya sebagai **`clamp_mask`** pada frame (jalur masker, atau `None` jika tidak ada yang dibatasi) — dalam hasil `CameraSession.capture()`, `ArraySession.capture()`&#x27;dalam bingkai `CaptureResult` dan respons perekaman HTTP. Build offline mencantumkan maskernya sebagai `clamp_masks`.
- Tumpukan multiband indeks langsung (`/api/camera/cli-array-index` dengan `save_multiband`) mendapatkan satu masker per kamera sumber, di dalam folder keluaran `clamp_masks/` — lihat referensi CLI&#x27;`lattice index`. Sebuah video tidak memiliki masker: hasil build offline melaporkan fraksi clamp setiap frame sebagai gantinya.

Karena tidak ada folder produk yang menyimpan masker, pada tingkat kedalaman mana pun, folder produk dapat diserahkan ke perangkat lunak fotogrametri apa adanya — termasuk subfoldernya — dan perangkat lunak tersebut hanya memproses gambar. Serahkan folder produk tersebut, bukan folder di atasnya. Proses impor Chloros sendiri melewati masker-masker tersebut.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect untuk Kamera LATTICE

Sesi backend yang persisten untuk perangkat keras langsung. Menggunakan titik akhir yang sama dengan yang digunakan oleh GUI, sehingga perilakunya identik di seluruh SDK / CLI / GUI.

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

Pengaturan eksposur, gain, dan target yang ditetapkan di sini akan dipertahankan sebagai milik Anda: kontrol eksposur otomatis Chloros (termasuk eksposur yang diukur oleh DAQ) tidak akan mengubahnya. `cam.set_settings(exposure_auto="Continuous")` (atau `gain_auto=`) mengembalikan pengaturan eksposur (gain), dan `cam.set_settings(target_brightness_auto=True)` mengembalikan target ke sistem target otomatis Chloros.

#### `connect_camera()` Tanda Tangan

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

- **Fungsi ini kembali setelah aliran dimulai.** Backend menunggu, dengan batas waktu (sekitar 15 detik hingga streaming dimulai, lalu 10 detik untuk buffer pertamanya yang lengkap — bukan untuk frame pratinjau pertama, yang muncul sekitar 1,5 detik kemudian), sehingga `status()` muncul tepat setelahnya, diikuti oleh `streaming`. `connect_stream` dari sesi tersebut adalah blok balasan `stream`: `state` `streaming`, `no_frames` (aliran dimulai tetapi tidak mengirimkan apa pun — hal ini dinyatakan oleh `note` dan, untuk kamera yang menggunakan paket jumbo, disarankan untuk menyambung kembali dengan ukuran paket 1500 byte), `starting` atau `start_failed`, dengan `waited_s`. `None` untuk kamera yang sudah terhubung. Pratinjau (`get_camera_preview`, `/api/camera/jpeg/<serial>`) yang diminta segera setelah penundaan, yang dibatasi (4 detik), untuk frame pratinjau pertama kamera alih-alih menjawab &quot;Belum ada bingkai yang tersedia&quot;; kamera yang alirannya belum mengirimkan apa pun (`no_frames`, atau dalam mode siaga untuk pemicu perangkat keras) tetap langsung merespons.
- **Ukuran paket dapat Anda tentukan sendiri.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` membuka kamera dengan paket berukuran 1500 byte, tanpa pengujian ukuran paket (`chloros-cli lattice cam-connect --packet-size standard` mengirimkan hal yang sama).
- **Hanya satu koneksi sekaligus yang dapat menggunakan pool kamera**: pembukaan koneksi, pengujian ukuran paket, pengunduhan kalibrasi pertama (sekitar 70 detik untuk kamera yang baru di komputer ini), dan dimulainya aliran data. Permintaan yang membutuhkan kamera pada saat yang sama (`read_nodes`, `set_settings`, penangkapan, koneksi kedua) akan menunggunya paling lama 5 detik, lalu gagal dengan kode HTTP 503 `CAMERA_POOL_BUSY`, menyebutkan nama kamera yang terhubung (`ChlorosConnectError`); tanyakan lagi sebentar lagi. Daftar tersebut tidak pernah menunggu: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()`, dan `CameraSession.status()` langsung merespons. Sebuah kamera yang sedang terhubung terdaftar di bawah balasan kumpulan `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — dan `elapsed_s`), dan `status()`-nya menunjukkan `connecting: True` dengan `phase`-nya; kamera lainnya dibaca secara langsung saat pool kosong, jika tidak, dijawab berdasarkan pembacaan terakhirnya (`live: False`).

#### Metode `CameraSession`

| Metode | Deskripsi |
| --- | --- |
| `status(timeout=15.0)` | Status langsung kamera, sebagaimana dibaca oleh `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed`, atau `stopped`), `device` (model, nomor seri, firmware, suhu, `is_mono`) dan pengaturannya saat ini. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Membaca node GenICam; mengembalikan `{nodes, errors, enums, device, was_in_pool, in_array_id}`, ditambah `frame_values` dan `notes` saat eksposur otomatis berjalan (lihat di bawah). |
| `set_settings(**kwargs)` | Menulis node berdasarkan nama ramah (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Sebuah `exposure_time` (`gain`) merupakan pengaturan eksposur (gain) manual: ditulis tanpa `exposure_auto` (`gain_auto`), mode otomatis harus dimatikan terlebih dahulu, sehingga nilainya tidak ditolak oleh mode otomatis yang sedang berjalan. Hasilnya, `applied` menyimpan nilai yang diambil kamera; penulisan yang ditolak kamera terdapat di `errors` dan tidak disimpan. Pesan kesalahan tersebut menyebutkan rentang yang diterima kamera saat ini (nilai maksimum `ExposureTime` mengikuti laju bingkai kamera), dan mode otomatis yang dimatikan hanya untuk nilai yang ditolak tersebut diaktifkan kembali; `restored` mencantumkannya (misalnya `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Mengambil **satu** frame. Mengembalikan `CaptureResult`: daftar kamus metadata frame (satu per level yang disimpan), dengan `.skipped`, `.failed`, `.daq_files`, dan `.daq_matches`. (Fitur pengambilan gambar beruntun/multi-frame telah dihapus — panggil `capture()` dalam loop jika Anda memerlukan serangkaian gambar.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Pengambilan gambar berturut-turut (`interval_s=0`) atau dengan interval dari kamera ini, untuk penangkapan `count` dan/atau `duration_s` (setidaknya satu, agar proses berakhir), seperti yang dilakukan `ArraySession.capture_repeated()` untuk sebuah array. Mengembalikan daftar `CaptureResult`; proses berhenti setelah penangkapan yang melaporkan drive penuh. `on_capture(i, result)` dijalankan setelah masing-masing; kata kunci lainnya dialihkan ke `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Melepaskan dari pool, kamera dikembalikan ke kondisi saat ditemukan saat terhubung (pengaturan pemicu, PTP, pengaturan); `keep_settings=True` menyimpan pengaturannya ke dalam memori sendiri, sedangkan `clear_settings=True` mereset memori tersebut (lihat [Memori kamera](#camera-memory--keep_settings--clear_settings)); mengembalikan `camera_memory` saat salah satunya diminta. Kamera yang merupakan anggota array akan melepaskan seluruh array. Tidak melakukan apa-apa jika kita terhubung ke sesi yang sudah terbuka. Mengembalikan nilai setelah backend mengonfirmasi pelepasan, atau ketika tidak ada yang perlu dilepaskan (kamera sudah tidak ada, atau tidak ada backend yang berjalan); pelepasan yang melebihi batas waktu atau ditolak oleh backend akan memicu `ChlorosConnectError`, karena kamera mungkin masih dipegang dan koneksi berikutnya akan bertabrakan dengannya. Keluarnya blok `with` tidak pernah memicu kesalahan. |

Atribut: `serial`, `device` (catatan penemuan kamera) dan `backend_url`.

Kontrol ekspor `capture()` (model yang sama dengan array + GUI):

- `processing` / `levels` — `processing="all"` menyimpan setiap jenis ekspor yang berlaku; `levels=["raw","radiance"]` hanya menyimpan yang tersebut (menggantikan `processing`). Lewati keduanya untuk menggunakan pengaturan default backend.
- `daq` — pembacaan DAQ yang digunakan dalam penangkapan disimpan sebagai berkas tambahan (sidecar) `.daq` secara default, termasuk tangkapan mentah (raw-only), sehingga bingkai tersebut dapat diproses ulang menjadi reflektansi/indeks di kemudian hari. `daq=False` mengabaikannya jika tidak ada level yang menggunakannya. `force_daq=True` adalah ejaan lama dari pengaturan default. Tanpa DAQ, tidak ada `.daq`.
- `daq_tolerance_ms` — seberapa jauh pembacaan dapat berada dari waktu pemaparan dan tetap berada dalam batas toleransi (default 500). Hanya hasil penilaian yang berubah.

#### `read_nodes()` pada eksposur otomatis — `frame_values` dan `notes`

Saat `ExposureAuto` (`GainAuto`) berjalan, node `ExposureTime` (`Gain`) membaca kembali **nilai terakhir yang ditulis**, bukan apa yang sedang dilakukan kamera. `nodes` masih menyimpan nilai node tersebut. Saat Anda meminta `ExposureTime` atau `Gain` pada kamera yang terhubung yang mode otomatisnya bukan `Off`, balasannya menambahkan:

- `frame_values` — nilai yang dibawa oleh bingkai terbaru kamera, di bawah nama node (`ExposureTime` dalam µs, `Gain` dalam dB);
- `notes` — satu kalimat per node tersebut yang mencantumkan kedua nilai tersebut.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Kedua kunci tersebut tidak ada ketika mode otomatisnya adalah `Off` (node tersebut merupakan) dan untuk kamera yang tidak terhubung ke backend. Untuk sebuah array, `ArraySession.status()["member_ae"]` memberikan nilai bingkai setiap anggotanya sekaligus.

#### Batas atas eksposur yang diukur oleh DAQ (HTTP)

Dengan sensor cahaya DAQ yang terhubung, Chloros membatasi eksposur × gain kamera berdasarkan cahaya siang hari yang diukur agar panel referensi putih tidak mengalami clipping. Batas ini hanya berlaku selama target kalibrasi berada dalam jangkauan pandang kamera tersebut (terlihat dalam bingkainya dalam 10 detik terakhir); jika tidak ada target dalam jangkauan pandang, eksposur otomatis standar kamera, dengan perlindungan sorotan (highlight protection) bawaan, akan aktif sebagai gantinya, sehingga bagian lain dari pemandangan tidak menjadi lebih gelap dari yang seharusnya. Tidak ada pembungkus SDK; baca atau ubah pengaturan ini per kamera menggunakan `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (isi POST `{"enabled": false}` akan menonaktifkannya untuk kamera tersebut). Responsnya berisi `enabled`, `cap_us` (batas atas saat aktif, `None` saat tidak aktif, termasuk saat tidak ada target kalibrasi dalam jangkauan), `k_exp_g` (konstanta yang digunakan untuk menghitung batas atas) dan:

| Kolom | Arti |
| --- | --- |
| `k_exp_g_source` | Konstanta mana yang digunakan: `camera` — milik kamera ini sendiri, dihitung dari titik lab dalam paket kalibrasinya (kamera RGB); `fleet` — konstanta armada, digunakan jika paket tidak memiliki titik lab; `override` — penggantian bidang yang ditetapkan di backend. |
| `k_exp_g_fleet` | Konstanta armada, untuk perbandingan. |

### Synchronized Array — `ArraySession` (Smart-Prep)

`connect_array` adalah **titik awal yang direkomendasikan** untuk konfigurasi multi-kamera. Sistem ini menjalankan alur kerja Smart-Prep berbasis antarmuka grafis (GUI) secara penuh di balik layar:

1. **Analisis jaringan**(`/api/camera/array/recommend`) — mengatur**sensor secara penuh** (tanpa binning, tanpa pemotongan ROI) dan membiarkan laju bingkai turun sesuai dengan yang disediakan oleh host dan jaringan ini, diproyeksikan dengan format piksel dan pengemasan yang akan digunakan oleh Connect. Bingkai tidak pernah diperkecil untuk menjaga laju tetap tinggi: gunakan `width` / `height` atau `binning=2` / `4` untuk menukar resolusi dengan laju bingkai (daftar opsi tersebut beserta laju bingkai yang diproyeksikan terdapat di`alternatives` mencantumkan opsi-opsi tersebut beserta fps yang diproyeksikan).
2. **Pemilihan otomatis tingkat** — `sim-capture-sim-emit` (setiap kamera melakukan eksposur pada pemicu yang sama, emisi diatur agar dapat berbagi jalur, kecepatan bingkai menurun seiring berkurangnya pembagian) pada jaringan apa pun di mana kamera dapat diatur agar berbagi; `slip-emit-and-capture` (kamera bergiliran menggunakan saluran, tidak diekspos secara bersamaan) hanya jika hal tersebut tidak memungkinkan.
3. **PTP diaktifkan**secara default — cap waktu antar-kamera selisih**~1 ms**dari satu jam bersama, yaitu [PTP grandmaster](#ptp-grandmaster--time_sync_status) di backend. Eksposur simultan berasal dari pemicu perangkat keras M8 (**&lt; 100 µs** antar-modul), bukan dari PTP: PTP menyelaraskan *cap waktu*, bukan eksposur.
4. **Pemilihan otomatis format piksel per kamera** — mono → `Mono12`, warna → `BayerRG12`.
5. **Penetapan awal AE** — mengambil snapshot status AE terkini setiap kamera agar koneksi tidak mereset eksposur di tengah proses.
6. **Konfigurasi pemicu GPIO** — `connect_array` mengaktifkan setiap kamera (`TriggerMode=On`, `TriggerSource=Line2`) sehingga pulsa masterdapat menggerakkan kamera-kamera slave melalui kabel M8. Ini adalah langkah khusus untuk array: jika hanya satu kamera yang diaktifkan dengan `LatticeCamera`, kamera tersebut akan beroperasi secara bebas. Melepaskan array (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) akan mengembalikan setiap kamera ke mode operasi bebas (`TriggerMode=Off`), dengan semua pengaturan lainnya tetap seperti yang ditemukan saat koneksi: pengaturan pemicu dan jalur sinkronisasi (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; jalur sinkronisasi master dikembalikan ke input), PTP, dan pengaturan kamera yang diubah selama sesi — kecuali `keep_settings=True` / `clear_settings=True` (lihat [Memori kamera](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atribut: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport`, dan `packed_reason` (empat yang terakhir adalah `None` pada handle dari `attach_array`), `connect_stream`, dan `backend_url`. `connect_stream` adalah blok `stream` dari balasan koneksi, bentuk koneksi tunggal ditambah `members` (`{serial: {state, why?}}`): `streaming` setelah kelompok sinkronisasi pertama dari array tiba (backend menunggunya, dengan batas waktu sekitar 10 detik), atau `no_frames` dengan kegagalan pengambilan terakhir setiap anggota sebagai `why`-nya dan, untuk paket jumbo, fallback 1500 byte di `note`. `None` pada handle `attach_array`.

#### Tanda Tangan `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`timeout` menunjukkan berapa lama panggilan menunggu koneksi, dan nilai ini dikirim ke
backend sebagai batas waktunya. Koneksi yang belum selesai pada saat itu akan dibatalkan: setiap kamera yang
telah dibuka dikembalikan ke kondisi semula dan ditutup, tidak ada array yang terhubung, dan
panggilan tersebut memicu `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, yang menyebutkan
tahap yang telah dicapai oleh proses koneksi). Jadi, panggilan yang memicu kesalahan tersebut tidak pernah meninggalkan array
yang terhubung di belakangnya. Sebuah array 9 kamera terhubung dalam waktu sekitar satu menit; segera setelah
kamera dinyalakan, prosesnya bisa memakan waktu beberapa menit. Sebuah array yang sudah aktif tetapi belum memiliki
kelompok frame yang tersinkronisasi akan dikembalikan, dengan kode kesalahan `connect_stream["state"]`
`no_frames`. Log backend memiliki satu baris waktu per langkah koneksi
(`[array-connect <id>] <step>: <s> s …`) dan baris terakhir yang menyebutkan langkah yang paling lambat.

Nilai-nilai `force_tier`:
- `"sim-capture-sim-emit"` — benar-benar simultan (semua kamera memotret pada tepi clock yang sama).
- `"sim-capture-ftd-stagger"` — pengaturan waktu yang fleksibel (kamera memancarkan pada waktu yang sedikit bergeser sehingga paket diserialisasikan di kabel).
- `"slip-emit-and-capture"` — pengambilan berurutan per kamera (tanpa sinkronisasi waktu). Dipilih secara otomatis hanya ketika kamera tidak dapat diatur kecepatannya cukup lambat untuk berbagi saluran secara bersamaan.

`wire_ceiling_mbps` menggantikan **anggaran bandwidth berkelanjutan host** dalam MB/s — angka tunggal
yang menjadi acuan alokasi seluruh array. Biarkan tetap pada `None` untuk menggunakan nilai yang terdeteksi secara otomatis
. Kurangi nilainya ketika array melaporkan frame yang rusak akibat GVSP: nilai otomatis ini diperoleh
dari kecepatan tautan yang diiklankan oleh NIC, yang cenderung melebih-lebihkan adaptor USB, jalur PCIe yang sempit, dan
fabric bersama yang sibuk — dan perkiraan berlebih ini muncul sebagai frame yang rusak, bukan sebagai
tautan yang terlihat lambat. Nilai ini disimpan dalam blok penangkapan array proyek, sehingga
pembukaan ulang atau pengaturan `connect_array` di kemudian hari akan mengembalikannya seperti pengaturan array lainnya.
Lihat [Kesehatan Array](#array-health--which-subsystem-is-losing-frames).

#### Transportasi Terkemas 12-bit (`packed=`)

Pada konfigurasi sensor penuh, laju bingkai suatu array ditentukan oleh seberapa cepat setiap kamera dapat mengirimkan
bingkainya. Format 12-bit (`BayerRG12`, `Mono12`) biasanya dikirim sebagai kata 16-bit;
transportasi terkompresi mengirimkan 12 bit yang sama dalam 1,5 byte per piksel dan
mengekstraknya di host, sehingga setiap frame membutuhkan waktu 25% lebih sedikit untuk dikirim — sekitar
**33% lebih banyak frame per detik pada sensor penuh** (diukur: 10,7 → 14,0 fps pada
susunan 9 kamera). **Data yang disimpan identik**: nilai yang sama, file 16-bit
yang sama, metadata yang sama.

- `packed="auto"` (default) hanya melakukan pengemasan jika hal itu bermanfaat **dan** host mampu
  melakukan penguraian: saat terhubung, backend mengukur kecepatan penguraiannya sendiri,
  jumlah CPU, beban saat ini, dan anggaran jaringan. Komputer desktop dengan inti CPU yang berlebih
  melakukan pengemasan pada sensor penuh; host yang sibuk atau berdaya rendah (komputer papan tunggal,
  pengontrol tertanam) melakukan streaming tanpa pengemasan. ROI kecil dan bingkai yang digabungkan tidak banyak mendapat manfaat.
- `packed="off"` (atau `False`) tidak pernah mengompresi — untuk mengesampingkannya saat mendiagnosis,
  atau pada host yang harus mengalokasikan CPU-nya untuk pekerjaan lain.
- `packed="on"` (atau `True`) selalu mengemas format 12-bit.
- Nilai lain apa pun akan memicu `ValueError`. Format 8-bit dan 16-bit tidak pernah
  dikompresi.

`analyze_array_network(..., packed="auto")` menentukan keputusan yang sama,
sehingga fps-nya adalah laju eksekusi array tersebut (responsnya membawa
`packed_transport` dan `packed_reason`); nilai default `pixel_format`-nya adalah apa yang
diterapkan oleh koneksi (Mono12 / BayerRG12) alih-alih 8-bit.

Keputusan dan alasannya terdapat pada sesi:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` menerapkan penggantian yang sama ke setiap
array dalam sebuah proyek (default `None`, yang berarti `"auto"`: sebuah proyek tidak menyimpan
pengaturan packed per array).
Untuk pengaturan default di seluruh mesin, atur `CHLOROS_ARRAY_PACKED12=off` (atau `on`) di
lingkungan backend; pengaturan eksplisit `packed="on"` / `"off"` tetap berlaku.
`/api/camera/array/status` melaporkan `packed_transport` dan `packed_decision`.

#### Kecepatan Bingkai yang Diharapkan

Kecepatan bingkai array yang diukur (disinkronkan perangkat keras, 12-bit; sensor penuh = 2048 × 1536). Gunakan data ini untuk memeriksa keabsahan `status()["actual_fps"]`; `analyze_array_network()` memprediksi laju bingkai untuk host dan jaringan Anda sendiri.

| Konfigurasi | Laju bingkai array |
| --- | --- |
| Sensor penuh, NIC host PCIe 10 GbE, transportasi terkompresi (`packed="auto"` pada desktop) | ~14 fps |
| Sensor penuh, NIC host PCIe 10 GbE, tanpa pengemasan (`packed="off"`) | ~10,5 fps |
| Sensor penuh, 8 kamera pada adaptor USB 10 GbE | ~3,8 fps |
| Sensor penuh, koneksi host 1 GbE | 1,2–1,7 fps |
| Penggabungan 2 × 2 (`binning=2`), NIC host 10 GbE | ~30–33 fps |

- Pada koneksi PCIe 10 GbE, **kamera**-lah yang menjadi pembatas pada mode sensor penuh, bukan jaringan: 4 dan 8 kamera berjalan pada kecepatan yang sama.
- **`target_fps` adalah kecepatan pemicu, dan array tidak pernah berjalan lebih cepat.** `target_fps=1` menjalankan array pada 1 fps; `None` (pengaturan default) menjalankannya pada kecepatan tertinggi yang dapat ditangani oleh host dan jaringan. Kecepatan yang melebihi batas tersebut akan diturunkan ke batas tersebut (lihat [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` dari `4294` di `status()` bukanlah kecepatan tautan.** Artinya, Windows tidak dapat membaca kecepatan adaptor (driver melaporkan nilai 32-bit terbesar), yang umum terjadi pada adaptor Ethernet USB. Kecepatan yang dapat digunakan adaptor USB di Windows dibatasi hingga 200 MB/s terlepas dari apa pun (lihat [Array Health](#array-health--which-subsystem-is-losing-frames)); untuk menyetelnya, lihat bagian &quot;Adaptor USB 10GbE&quot; pada referensi CLI di bawah [Pengaturan &amp; Penyetelan NIC Host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Batas kecepatan USB juga berlaku di Linux: 400 MB/s pada x86_64, 200 MB/s pada NVIDIA Jetson.** Adaptor diidentifikasi dari sysfs (jika tidak ditemukan, menggunakan `ethtool -i` sebagai cadangan), sehingga adaptor USB 5 GbE atau 10 GbE dibatasi pada batas tersebut, bukan pada kecepatan nominalnya sebesar 500 / ~1060 MB/s. Angka pada x86_64 diukur di Linux: adaptor yang sama yang merusak frame di atas 200 MB/s di Windows dapat menjalankan susunan 4 kamera tanpa gangguan pada setiap batas atas hingga 480 MB/s. Pada NVIDIA Jetson, adaptor yang sama hanya berjalan lancar pada 200 MB/s dan kehilangan frame secara sporadis di atas kecepatan tersebut. Jika host Anda dapat menangani lebih banyak data tanpa kehilangan frame dengan `gvsp_corrupt`, naikkan batasnya dengan `wire_ceiling_mbps`. Di Linux, adaptor pada port USB 2.0 dibatasi hingga 30 MB/s; colokkan ke port USB 3.

#### Over-Subscription (batas bawah per kamera)

Sebuah array dapat berfungsi di jaringan apa pun; laju bingkai bergantung pada kondisi jaringan. Pacing Sim-emit mengalokasikan porsi yang merata kepada setiap kamera dari alokasi bandwidth yang aman dari tabrakan, dan semuanya tetap memicu pada sinyal pemicu yang sama; sehingga pada koneksi lambat, porsi tersebut — dan laju bingkai — menjadi kecil: 8 kameraresolusi penuh 12-bit pada 1 GbE tanpa jumbo frame masing-masing mendapatkan 6,5 MB/s, sekitar **1 fps**(sekitar 1,3 fps jika dikemas). Bagian tersebut dibatasi pada laju paling lambat yang dapat diatur untuk kamera-kamera tersebut (`per_cam_floor_bps`: dibaca dari rentang GevSCPD setiap kamera saat terhubung, dan tidak pernah di bawah 1 MB/s). Jika koneksi kabel memerlukan kecepatan lebih lambat dari itu, koneksi akan beralih ke pengiriman**serial**(`slip-emit-and-capture`): kamera bergantian pada kecepatan penuh koneksi, sehingga setiap frame tiba tetapi anggota**tidak terpapar secara bersamaan**— tingkatan respons dan pemblokir (`SERIAL: ...`) menyatakan hal tersebut; berikan `force_tier="sim-capture-sim-emit"` untuk menolak pengalihan tersebut. Ukuran frame tidak memengaruhi pembagian, sehingga**binning dan ROI meningkatkan laju frame tetapi tidak mengubah apakah sebuah array muat**. Hanya kabel yang tidak dapat mentransmisikan array pada kecepatan apa pun, baik secara bersamaan maupun bergantian, yang**melebihi kapasitas** — mode kegagalan adalah kehilangan paket GVSP, bukan laju bingkai yang lebih rendah. Solusi: kurangi jumlah kamera, gunakan bingkai jumbo dari ujung ke ujung, atau gunakan NIC yang lebih cepat.

- Respons `analyze_array_network()` dan `/api/camera/array/connect` membawa `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe`, dan `per_cam_floor_bps`. Ketika `oversubscribed` bernilai true, proyeksi **mengosongkan bidang fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) alih-alih melaporkan laju yang lambat namun berfungsi, yang dapat menyesatkan.
- `POST /api/camera/array/connect` menerima parameter body `pin_resolution` (**hanya HTTP — bukan argumen kata kunci SDK**; `connect_array` tidak mengeksposnya). Ini adalah mode ketat: koneksi yang kelebihan langganan dengan pengaturan `pin_resolution` akan**ditolak tegas** dengan pesan kesalahan yang menyebutkan setiap solusi. Tanpa parameter tersebut, koneksi akan dilanjutkan dengan peringatan bahwa array akan kehilangan paket. Dalam kedua kasus, koneksi tetap mempertahankan ukuran frame yang diminta — sistem tidak akan pernah melakukan pengelompokan atau pemotongan secara otomatis.
- Hanya untuk pengujian: atur `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` di lingkungan backend untuk menurunkan tingkat penolakan menjadi peringatan keras — Anda tetap dapat terhubung dan menerima kehilangan paket.

#### Kesehatan Array — subsistem mana yang kehilangan frame

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) membawa blok `health` yang aktif pada
array yang terhubung, dievaluasi ulang pada jendela bergulir **10 detik** . Ini membagi kehilangan frame
menjadi dua penyebab yang memerlukan perbaikan berlawanan, alih-alih satu tingkat &quot;tidak lengkap&quot; yang
tidak menyebutkan keduanya:

| Kolom | Artinya | Subsistem mana |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (per serial) | Frame **telah tiba namun secara struktural rusak**— kehilangan paket GVSP. |**Jaringan**: kapasitas kabel, kecepatan pengiriman, cincin penerima NIC, MTU |
| `never_arrived_rate_pct` (per serial) | Frame **sama sekali tidak pernah tiba**— kamera tidak memicu, atau tidak ada yang keluar darinya. |**Pemicu / sinkronisasi**: Kabel M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Tingkat terburuk kamera untuk masing-masing. | — |
| `per_cam_rate_pct` | Tingkat ketidaklengkapan gabungan per kamera (kedua penyebab digabungkan). | — |
| `stable_for_seconds` | Berapa lama setiap kamera berada di bawah 0,01 %. | — |

Selain `health`, catatan yang sama melaporkan angka alokasi keseluruhan yang terpakai:

| Kolom | Artinya |
| --- | --- |
| `wire_ceiling_mbps` | Anggaran bandwidth yang berlaku secara berkelanjutan pada host, MB/s. |
| `wire_ceiling_source` | Dari mana angka tersebut berasal, dalam bentuk deskripsi — misalnya `USB-capped 200 MB/s (was theoretical 1062; …)` atau `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` saat `wire_ceiling_mbps=` menetapkannya. |
| `nic_is_usb` | `true` untuk adaptor Ethernet USB. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Penafsiran:** nilai `gvsp_corrupt_rate_pct` yang tidak nol dengan `never_arrived_rate_pct` bernilai 0 berarti
pemicu dan sinkronisasi kabel sempurna, serta 100% kehilangan terjadi pada jalur jaringan — turunkan
`wire_ceiling_mbps` dan sambungkan kembali. Pola sebaliknya mengindikasikan masalah pada kabel sinkronisasi atau
jalur pemicu.

> **`target_fps` bukanlah faktor penentu untuk frame yang rusak.** Pacing GevSCPD ditetapkan sekali saat
> koneksi, sehingga menurunkan laju pemicu mengubah siklus kerja dan bukan
> laju burst emisi simultan. Pengurangan permintaan sebesar 5× yang diukur tidak menghasilkan perbaikan, sementara
> menurunkan batas kecepatan kabel dari 240 menjadi 200 MB/s menurunkan tingkat kerusakan pada rig yang sama dari 10,4% menjadi
> 0,00%.

> **Array yang sedang berjalan tidak dapat merencanakan ulang sendiri pada firmware LATTICE (M3C / M3M).** Ia tidak dapat
> memperbaiki masalah ini sendiri; putuskan sambungan dan sambungkan kembali agar pemilih waktu sambungan merencanakan ulang berdasarkan
> batas atas yang baru.

Sebuah **adaptor Ethernet USB dibatasi hingga 200 MB/s** oleh probe terlepas dari
labelnya: tabel efisiensi yang mengubah kecepatan tautan menjadi angka berkelanjutan berasal dari
PCIe, dan NIC USB mengiklankan kecepatan tautan Ethernet-nya sambil dibatasi oleh
bus USB dan drivernya. Batas ini bersifat absolut, bukan fraksi — adaptor USB 1 GbE
menghasilkan ~80 MB/s dan tidak terpengaruh. Hal ini berlaku di Windows dan Linux (termasuk Jetson);
lihat [Laju Frame yang Diharapkan](#expected-frame-rates) untuk mengetahui di mana pengukuran tersebut dilakukan.

#### Metode `ArraySession`

| Metode | Deskripsi |
| --- | --- |
| `status(timeout=10.0)` | Status real-time dari array yang sedang berjalan: `target_fps` / `target_fps_auto` (laju pemicu dan apakah pengontrol laju mengendalikannya), `target_fps_clamped_reason` (alasan laju yang dikunci dipertahankan lebih rendah, jika tidak maka `None`), `actual_fps` (terukur; 0 setelah loop pengambilan menyerah atau macet), `recommended_fps`, `streaming` (False setelah loop pengambilan menyerah — periksa ini, bukan hanya `last_error`; tetap True setelah pengaturan dimulai ulang, yang ditandai oleh `restarting`), `stalled` (True selama loop pengambilan berjalan tetapi belum menerbitkan grup frame selama sekitar sepuluh periode frame, 2–30 detik; `last_error` baru menandainya setelah 60 detik), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` hanya ketika array terhubung dengan PTP **dan** setiap anggota terakhir membaca `PtpEnable` dalam keadaan aktif; tidak sama dengan terkunci, yang diukur oleh `ptp_epoch`), `ptp_configured` (sesuai permintaan koneksi), `ptp_off_members` / `ptp_unread_members` (anggota yang terakhir membaca status `PtpEnable` dalam keadaan mati / tidak merespons), `member_recovery` (`None`, atau anggota yang kehilangan daya atau koneksinya: `state` `waiting` meskipun tidak merespons, `reopening`, kemudian `recovered`, bersama dengan `members` dan `why` — array tersebut akan mengaktifkannya kembali saat perangkat tersebut merespons dan mengembalikan waktu koneksi PTP-nya, pengaturan pemicu dan kabel sinkronisasi, kecepatan, serta kamera, kemudian melakukan streaming lagi tanpa `connect_array` baru), dan `ptp_epoch` (terukur), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` berarti Windows tidak dapat membaca kecepatan adaptor — lihat [Laju Frame yang Diharapkan](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (apakah rantai radian per-pita dapat digunakan, dan anggota mana yang merusaknya), dan `band_clamp` — sementara tampilan langsung menampilkan susunan tersebut: persentase piksel yang terang yang dibatasi oleh proses unmix menjadi 0, per anggota (`members`: `{serial: [band0, band1, band2]}` dalam urutan pita R, G, B kamera) dan per pita, komposit menunjukkan (`composite`: `{band name: share}`), dengan `warn`, persentase yang ditandai oleh GUI sebagai &quot;Data pita dengan tingkat kepercayaan rendah&quot; (diukur sekitar sekali per detik; `None` saat tampilan langsung tidak aktif), dan `member_ae` — per anggota, eksposur (`exposure_us`) dan penguatan (`gain_db`) yang dibawa oleh bingkai terakhir yang dipublikasikan, dari bingkai itu sendiri (`read_nodes` dari `ExposureTime` / `Gain` mengembalikan nilai terakhir yang ditulis saat eksposur otomatis berjalan, bukan apa yang sedang dilakukannya). |
| `set_member_settings(serial, **kwargs)` | Menulis pengaturan satu kamera anggota, sebagaimana `CameraSession.set_settings()` mengambilnya (pencahayaan, gain, kecerahan target, ...); kamera anggota tersebut tetap berada dalam array. Menghasilkan pengecualian `ChlorosConnectError` untuk serial yang bukan merupakan anggota. Lihat [Pencahayaan pada Anggota Array](#exposure-on-array-members). |
| `capability(timeout=10.0)` | Apa yang telah ditentukan oleh smart-prep (tingkat pengambilan gambar, ukuran bingkai, `pixel_format`), blok `health` yang aktif, dan anggaran bandwidth host. Lihat [Kesehatan Array](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Ubah laju pemicu saat array sedang berjalan, tanpa perlu menyambung ulang. Sebuah angka mengunci nilainya (pengontrol laju tidak mengubahnya); `None` mengembalikannya ke mode otomatis. Mengembalikan `{requested, target_fps, target_fps_auto, clamped}` — `clamped` bernilai True ketika laju telah diturunkan ke tingkat yang dapat ditangani oleh host dan jaringan. Lihat [Pengambilan Gambar Berinterval untuk Survei Drone](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Satu frame dari setiap kamera, **semua dari satu pemicu**, sebagai array numpy (tidak ada file yang ditulis). Mengembalikan `GrabResult` (sebuah `dict`): `{serial: array}`, atau `{serial: (array, metadata)}` dengan `with_metadata=True`. `.group_id` adalah pemicu yang digunakan bersama oleh bingkai-bingkai tersebut; `.errors` memetakan setiap kamera yang terlewatkan ke alasannya (kamera yang tidak dapat dilayani pada pemicu tersebut diabaikan, tidak pernah diisi dari kamera lain). `processing` adalah `raw` / `debayered` / `radiance` / `reflectance` / `display`. Pembacaan ini sama dengan [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) dalam API proyek — satu implementasi bersama — sehingga setiap kata kunci di sana (`band`, `align`, `bit_depth`, `refl_max`) berfungsi di sini. Membutuhkan `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterator yang menghasilkan satu hasil `grab()` per tick (hanya argumen kata kunci), diatur ke `fps`, dibatasi oleh `duration` (detik) dan/atau `count` (`None` = tidak terbatas). **Setiap himpunan yang dihasilkan merupakan pemicu baru:** tick yang pemicunya terbaru adalah himpunan yang telah dihasilkannya akan menunggu pemicu berikutnya alih-alih mengulanginya, sehingga aliran berjalan pada kecepatan yang lebih lambat di antara `fps` dan laju bingkai array itu sendiri. Menghasilkan `FrameFetchError` alih-alih diam: seketika ketika setiap kamera menolak karena suatu alasan sehingga tick berikutnya akan diulang, setelah 60 detik tick tanpa frame, dan setelah 60 detik tanpa pemicu baru (&quot;tidak ada pemicu baru selama 60 detik&quot;). Implementasi yang sama dengan `ArrayHandle.frame_stream()`. Lihat [Frame Langsung dalam Pipeline Anda Sendiri](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Memulai perekaman berkelanjutan milik backend (mode Continuous pada GUI) → [`ContinuousHandle`](#continuoushandle). Menyimpan **setiap** grup yang disinkronkan yang dipublikasikan oleh array, sebagaimana `capture()` akan menyimpannya (kontrol ekspor yang sama), hingga dihentikan atau hingga `max_captures` / `duration_s` (`None` = hingga dihentikan). Menghasilkan pengecualian `ChlorosConnectError` jika sudah ada yang berjalan untuk array ini. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Satu grup penangkapan yang disinkronkan. Mengembalikan sebuah `CaptureResult` (daftar kamus frame + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Berikan hasil sebelumnya`group_seq` sebagai `after_seq` untuk mendapatkan kelompok yang disinkronkan berikutnya setelahnya, alih-alih menunggu yang baru. Kontrol ekspor di bawah ini. |
| `capture(..., smart=True)` | **Perekaman cerdas** — menunggu AE stabil di semua kamera, lalu memicu perekaman. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Perekaman tercepat: hanya data mentah + pembacaan DAQ yang ditetapkan (+ indeks gabungan bebas). Mencerminkan tombol &quot;Fastest Capture&quot; pada GUI. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Tunggal / Berkelanjutan / Interval dalam satu loop terbatas. Mengembalikan `list[CaptureResult]`.**Membutuhkan `count` dan/atau `duration_s`** agar dapat dihentikan (SDK tidak memiliki Ctrl+C). Juga berakhir lebih awal, setelah satu putaran, ketika suatu putaran melaporkan drive penuh (entri `.failed` dengan `disk_full`); putaran tersebut adalah hasil terakhir. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Mulai merekam tampilan indeks gabungan langsung ke video/GIF → `RecorderHandle`. Video tersebut berformat Motion-JPEG `.avi` (tidak memerlukan ffmpeg). Jika tidak ada tampilan GUI yang terbuka, backend akan merendernya: `expr` (ekspresi indeks dalam nama band array, misalnya `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`), atau NDVI array secara default. Satu perekam komposit per array. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Mulai pengambilan gambar raw-Bayer berkecepatan tinggi → `RecorderHandle`. Proses ulang secara offline dengan `build_video()`. `orient=True`: produk yang dihasilkan oleh `build_video()` disesuaikan dengan orientasi kamera (produk per kamera mengikuti orientasi kameranya masing-masing, sedangkan indeks gabungan mengikuti orientasi kamera utama); frame mentah tetap dalam format aslinya dan manifes pengambilan gambar mencatat orientasi setiap anggota. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Memproses ulang secara offline rangkaian gambar mentah yang telah disimpan menjadi video yang telah dikalibrasi. Proses ini akan menunggu hingga selesai (`wait=True`) dan mengembalikan hasil proses: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — lihat [hasil proses](#the-build_video-result) di bawah ini. |
| `build_video_status(job_id, timeout=15.0)` | Memeriksa status pekerjaan pembangunan offline: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Melepaskan seluruh array — **jika sesi ini yang membukanya** (`connect_array`): maka menjadi `force_disconnect()`, dan memicu `ChlorosConnectError` jika pelepasan tidak dikonfirmasi (blok `with` tidak pernah memicu pengecualian saat keluar). Pada handle dari `attach_array`, array dibiarkan tetap berjalan dan mencetak satu baris ke stderr yang menyatakan hal tersebut: array tersebut mungkin milik GUI atau skrip lain. Setiap kamera dibiarkan berjalan bebas dan tetap dalam kondisi seperti saat koneksi menemukannya; `keep_settings=True` menyimpan pengaturannya ke dalam memori kamera itu sendiri, sedangkan `clear_settings=True` mengatur ulang memori tersebut ke pengaturan pabrik (lihat [Memori kamera](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Melepaskan seluruh array terlepas dari siapa yang membukanya. Gunakan perintah ini pada handle yang terhubung jika yang dimaksud adalah mengakhiri array (misalnya, untuk menghubungkannya kembali dengan pengaturan yang berbeda). Kembali setelah backend mengonfirmasi pelepasan, atau ketika tidak ada yang perlu dilepaskan (array sudah tidak ada, atau tidak ada backend yang berjalan); proses pelepasan yang habis waktunya atau ditolak oleh backend akan memicu `ChlorosConnectError`. Dengan bendera mana pun, fungsi ini mengembalikan apa yang diperoleh memori masing-masing kamera (`camera_memory`), jika tidak, mengembalikan `None`. |

Kontrol ekspor `capture()` (titik akhir yang sama dengan yang digunakan GUI/CLI):

- `processing` / `levels` — `processing="all"` (atau `levels=["raw","radiance",…]`) menyimpan setiap jenis ekspor yang berlaku per kamera; satu nilai `processing` hanya menyimpan tingkat tersebut.
- `aligned` — **aktif secara default (`None`), karena penangkapan array yang sama oleh aplikasi Chloros**(Pengaturan Pengambilan Gambar**Terjajar** per array). Ekspor non-raw setiap anggota disesuaikan ke dalam bingkai kamera utama berdasarkan [profil penjajaran](#array-alignment) dan dipotong sesuai tumpang tindih bersama antar anggota, sehingga file-file tersebut terdaftar pikselnya dan dihasilkan dengan ukuran, pemotongan, serta tag `Alignment*` yang sama seperti yang dilakukan aplikasi (di bawah ROI, transformasi disesuaikan kembali ke jendela); `XMP-mapir:AlignmentJSON` menyatakan `"applied": true` dengan `crop_roi`-nya. File mentah tetap tidak terwarp pada ukuran aslinya tetapi menyimpan transformasi dalam metadata. `aligned=False` menyimpan setiap file pada ukuran aslinya, tanpa terwarp; `aligned=True` adalah pengaturan default yang dijelaskan. `continuous()`, `capture(smart=True)`, `capture_repeated()`, serta `ArrayHandle.capture()` / `capture_smart()` di tingkat proyek menggunakan pengaturan default yang sama. Terlepas dari apakah warp diterapkan atau tidak, setiap file anggota mencatat transformasi `Alignment*` kameranya begitu array memiliki profil penyelarasan — kecuali kamera yang profilnya tidak selaras (ditinggalkan, atau dipaksa menjadi identitas karena penyelesaian yang gagal), serta kamera referensi ketika tidak ada kamera lain yang diselaraskan: kamera-kamera tersebut tidak memiliki tag `Alignment*`. Jika tidak ada profil pada array, hasil pengambilan gambar disimpan tanpa penyelarasan, dan hasilnya menunjukkan hal tersebut: `result.alignment` adalah `{"requested": True, "applied": False, "warnings": [...]}`, satu peringatan untuk setiap alasan mengapa warp tidak dapat diterapkan (tidak ada profil, anggota dengan resolusi berbeda, array yang tidak dapat, bingkai yang tidak dapat dipetakan ke profil). `applied` adalah `True` hanya jika setiap ekspor yang meminta warp telah mendapatkannya. Penyelarasan otomatis array membuat profil begitu eksposurnya stabil; `ArrayHandle.calibrate_alignment()` / `align-calibrate` di CLI membuatnya sesuai permintaan. `result.oriented` menunjukkan apakah `orient=True` telah memproses produk-produk tersebut.
- `render_index=False` — lewati overlay indeks vegetasi per kamera; secara default, overlay tersebut ditampilkan di lokasi yang telah dikonfigurasi.
- `daq` / `daq_tolerance_ms` — pembacaan DAQ disimpan sebagai file tambahan `.daq` secara default, bahkan jika tidak ada tingkat yang dipilih yang membutuhkannya; `daq=False` akan melewatkannya dalam hal ini, dan `daq_tolerance_ms` menetapkan toleransi pencocokan (default 500). `force_daq=True` adalah ejaan lama dari nilai default. Lihat [Pembacaan DAQ yang disimpan oleh penangkapan](#the-daq-reading-a-capture-saves--daq_files-and-daq_matches).
- `orient=True` — mengubah **produk**setiap anggota (`display`, `radiance`, `reflectance`, indeks per-kamera, dan komposit) dengan orientasi kamera, setelah proses warp penyelarasan: dengan orientasi**kamera utama**saat file-file tersebut diwarp ke dalam bingkai kamera utama (pengaturan default `aligned` — file anggota tetap terdaftar bersama), atau dengan orientasi kamera masing-masing.**Default: setiap file anggota dalam format asli**, sebagaimana selalu terjadi pada pengambilan gambar array; `raw` dan `debayered` tetap dalam format asli dalam kedua kasus. Setiap file mencatat orientasi bingkai tempat pikselnya berada (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — file raw, dari kamera aslinya), dan file XMP `Alignment*` tetap menyimpan transformasi bingkai asli, yang diterapkan sebelum orientasi; file RAW yang membawa transformasi tersebut juga mencatat `ChlorosAligned*` (dari bingkai utama), yaitu orientasi hasil yang telah diwarp ke bingkai utama. Proses pemrosesan memutar setiap hasil sesuai dengan transformasi yang dimiliki oleh pikselnya: transformasi yang disejajarkan saat file RAW diwarp (penyelarasan array aktif), atau transformasi kamera itu sendiri saat tidak. `capture_repeated()` dan `capture_fastest()` meneruskannya begitu saja; `ArrayHandle.capture()` / `capture_smart()` di tingkat proyek juga menggunakannya.

**Kompresi TIFF (pengaturan khusus HTTP):** `ArraySession.capture()` tidak mengirimkan kunci `compression`, sehingga pengaturan default backend berlaku — `POST /api/camera/array/capture` membaca parameter badan `compression`, `"none"` secara default: tidak terkompresi, piksel sensor persis seperti yang dibaca (~6,3 MB per bingkai resolusi penuh). `"deflate"` (zlib L1 tanpa kehilangan data + prediktor horizontal, ~4,1 MB/bingkai) menulis file yang lebih kecil dengan kecepatan lebih lambat — keduanya tanpa kehilangan data dan dibaca secara identik saat diimpor. SDK tidak menyediakan argumen kata kunci (kwarg) untuk ini; jalan keluarnya adalah `chloros-cli lattice array-capture --compression deflate` atau HTTP mentah. DEFLATE mengunci GIL Python, sehingga penulisan terkompresi tidak dapat dijalankan secara paralel di antara thread penulisan per kamera — perekaman resolusi penuh berkelanjutan pada kecepatan sensor memerlukan pengaturan default tanpa kompresi. Rincian: [Referensi CLI → array-capture](cli-reference.md).**Grup berturut-turut (HTTP):** titik akhir mengembalikan `group_seq`; loop yang mengirimkannya kembali sebagai `after_seq` pada permintaan berikutnya akan mendapatkan grup sinkron berikutnya — termasuk grup yang diterbitkan saat perekaman sebelumnya sedang disimpan — alih-alih menunggu grup yang diterbitkan setelah permintaan, dan `skipped_groups` melaporkan grup mana pun yang terlewatkan oleh loop. `capture_repeated`, `--continuous` di CLI dan mode Continuous di GUI semuanya melakukan hal ini; interval yang telah berlalu tetap diperbarui pada setiap waktu yang dijadwalkan.**Penggantian ekspor per anggota (hanya HTTP):**titik akhir yang sama juga menerima `exclude_serials` (daftar — menghapus anggota dari kumpulan yang disimpan; array tersebut tetap dipicu sebagai satu grup yang disinkronkan dan anggota yang dikecualikan dikembalikan dalam `excluded`), `serial_levels` (penggantian tingkat per-kamera `{serial: [level tokens]}`), dan `serial_index` (penggantian overlay indeks per-kamera `{serial: bool}`). Ini adalah parameter tubuh yang setara dengan GUI dan**belum berupa argumen kata kunci SDK**; anggota yang tidak ada dalam peta akan menggunakan nilai default dari `levels` / `render_index` yang berlaku untuk seluruh array.

##### Memeriksa Kamera yang Dilewati — `CaptureResult.skipped`

`ArraySession.capture()` mengembalikan `CaptureResult`, yang merupakan subkelas dari `list`: iterasi, indeks, `len()` — setiap pola yang ada tetap berfungsi. Kode baru dapat memeriksa atribut `.skipped` untuk melihat kamera mana yang dikecualikan dan alasannya. Kasus yang paling umum adalah kamera RGB dalam susunan filter campuran saat Anda meminta `processing="radiance"` atau `"reflectance"` — radian per-Bayer tidak bermakna bagi sensor broadband, sehingga backend mengabaikan kamera-kamera tersebut daripada menghasilkan data yang tidak bermakna.

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

Token alasan mengikuti pola `<level>-not-applicable-to-rgb-cam` (satu entri per tingkat yang dilewati, masing-masing berisi `level`). Kamera yang paket kalibrasinya tidak dimuat akan melewati `radiance` / `reflectance` dengan `<level>-needs-calibration (connect the camera so its cal pack loads)`, dan jika tidak ada pembacaan DAQ reflektansi yang dapat digunakan, akan melewati dengan `reflectance-skipped-no-dls`. Lewatan reflektansi lainnya adalah `reflectance-skipped-no-fresh-dls` (tidak ada pembacaan sinar turun baru yang tersedia), `reflectance-skipped-bound-daq-unavailable (…)` (DAQ yang terikat tidak dapat dijangkau), dan `dls-uncalibrated-band-<nm>` — pita tersebut sebagian besar berada di luar rentang yang dikalibrasi secara radiometrik dari sensor cahaya DAQ (~374–974 nm), sehingga pembagian reflektansi absolut berbasis DAQ ditolak dan bingkai tersebut diturunkan statusnya secara tegas ke respons sensor. Di antara SKU yang tersedia, hanya F988 yang memicunya; jalur yang didukung kamera tersebut adalah alur kerja panel reflektansi.

##### File yang Tidak Dapat Ditulis — `CaptureResult.failed`

Sebuah pengambilan gambar yang hanya dapat menulis sebagian file-nya tetap mengembalikan: daftar tersebut berisi setiap berkas yang **telah** ditulis, dan `.failed` mencantumkan sisanya, satu kamus per berkas dengan `serial` (`None` untuk gabungan komposit dan berkas pendamping array `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, atau `match` — riwayat `.daq` dari `.match.json`, ketika `.daq` ditulis tetapi sidecar-nya tidak) dan `error`. Pada drive yang penuh, entri tersebut juga mencakup `disk_full: True`; setiap penangkapan data selanjutnya ke drive tersebut akan gagal dengan cara yang sama, itulah sebabnya `capture_repeated()` berhenti di sana. Tidak ada pengecualian yang muncul untuk penangkapan parsial, dan berkas dalam `.failed` tidak tertinggal di disk dalam keadaan setengah tertulis. `.failed` kosong pada penangkapan yang bersih.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### Pembacaan DAQ saat penyimpanan penangkapan — `.daq_files` dan `.daq_matches`

Setiap kali DAQ diikat ke kamera atau array, atau digabungkan, setiap penangkapan menyimpan pembacaan downwelling yang digunakannya: sebuah berkas `.daq` di subfolder `daq/` dari folder penangkapan, beserta berkas `.match.json` dan `.cameras.json` (yang merupakan pembacaan asalnya). Hal ini berlaku juga untuk penangkapan mentah atau yang telah di-debayered, dan berkasnya sama terlepas dari apakah penangkapan tersebut berasal dari GUI, CLI, atau SDK. `daq=False` mengabaikannya untuk penangkapan yang levelnya tidak menggunakan DAQ (tingkat `radiance`, `reflectance`, atau `display` tetap menuliskannya); `daq_tolerance_ms` memindahkan garis batas toleransi untuk perekaman tersebut dan bukan pada pembacaan yang digunakan. DAQ mana: yang terikat pada kamera atau array (pilihan sensor cahaya di GUI, pengikatan proyek, atau `chloros-cli daq pool-bind`), atau yang terbaru yang dimiliki oleh backend. CLI menjelaskan kontrol `--no-daq` dan `--daq-tolerance-ms` ([Referensi CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Sebuah penangkapan menyimpan pembacaan sensor cahaya yang digunakannya sebagai `.daq` dengan `<file>.daq.match.json` di sampingnya, yang menjelaskan bagaimana pembacaan tersebut dikaitkan dengan eksposur: `method`, `gap_ms` (dari eksposur ke pembacaan DAQ terdekat) dan `in_tolerance` (dalam 500 ms, atau `daq_tolerance_ms` dari penangkapan). File XMP dari file reflektansi tersebut memuat hasil yang sama dengan `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS`, dan `Chloros:DlsInTolerance`.

Sebuah pengambilan data menunggu pembacaan pertama DAQ setelah eksposur, sehingga dapat menggabungkan pembacaan di kedua sisi pembacaan tersebut (`method` `interpolated`). Sebuah DAQ-E mengirimkan pembacaan setiap 0,4 detik di siang hari dan setiap 1–2 detik di dalam ruangan, sehingga `capture()` dikembalikan sekitar satu setengah interval setelahnya; pencahayaan itu sendiri tidak tertunda. DAQ yang telah berhenti mengirimkan data tidak ditunggu. Nilai pembacaan DAQ-E merupakan rata-rata cahaya selama interval sejak pembacaan sebelumnya, sehingga ditempatkan di tengah interval tersebut: `gap_ms` diukur hingga titik tengah tersebut, dan `.match.json`&#x27;`bracket_stamps_ns` memberikan cap waktu pembacaan itu sendiri. `ArraySession.continuous()` menunggu tanpa memperlambat pengambilan data. `capture_repeated()`&#x27;stidak menunggu, sehingga mempertahankan laju pembacaan, dan disesuaikan dengan pembacaan terdekat sebelum eksposur.

Sebuah pembacaan tidak pernah kadaluwarsa, sehingga tidak ada bingkai yang hilang akibat pembacaan terlambat — melainkan diberi label:

- **Aliran DAQ terhenti** — pembacaan terbarunya digunakan, dan `gap_ms` menunjukkan seberapa lama pembacaan tersebut (`in_tolerance: false` lebih dari 500 ms).
- **Tidak ada DAQ yang dapat dibaca**— terputus (`disconnect()`, `chloros-cli daq pool-disconnect`), atau DAQ yang terhubung tidak dapat dijangkau — pembacaan terakhir kamera digunakan kembali dan dijelaskan berdasarkan**pembacaan ini** : `method`, `retained`, dan `gap_ms` dari eksposur ini, `in_tolerance` dihitung ulang, dan `retained_from_epoch_ns` menyebutkan eksposur yang pertama kali dicocokkan dengannya. Kamera yang tidak pernah memiliki pembacaan tetap tidak mendapatkan reflektansi, dan hasilnya menjelaskan alasannya (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, atau penolakan pengambilan gambar).

File yang disimpan oleh penangkapan di dalam folder proyek yang terbuka di Chloros akan ditambahkan ke daftar file proyek tersebut, seperti halnya penangkapan di dalam aplikasi (file yang disimpan di tempat lain dibiarkan apa adanya): `CameraSession.capture()`, `ArraySession.capture()` (`smart=True` juga) dan `capture_cameras_transient()`. Hasilnya tercantum dalam `.project_add` (`capture_cameras_transient()`: `project_add` dari kamus) -- `held`, `files`, dan `message` di bagian backend; `held` adalah `True` selama proses sedang berjalan, dan berkas-berkas tersebut digabungkan setelah proses selesai. `None` ketika tidak ada berkas yang masuk ke dalam proyek.

`CameraSession.capture()` dan `ArraySession.capture()` mengembalikannya sebagai hasil `.daq_files` dan `.daq_matches`, sebagai respons penangkapan HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) mengandung `daq_files` dan `daq_matches`, masing-masing satu per `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms`, dan `stream_fault` — kesalahan aliran internal DAQ, yang hanya diperiksa jika pembacaan berada di luar batas toleransi. Sebelum mengandalkan nilai reflektansi dari hasil pengambilan data, periksa `in_tolerance` dalam `.match.json` atau XMP-nya, atau [`status()`](#daqsensorsession-methods).

Tingkat `processing`:

| Tingkat | Keluaran |
| --- | --- |
| `"raw"` | Bayer saluran tunggal (kamera monokrom: pita tunggal) langsung dari sensor. |
| `"debayered"` *(pengaturan default SDK)* | Warna 3-saluran: offset gelap (DSNU) dari paket kalibrasi yang diambil dari data mentah, kemudian demosaik berbasis tepi dari OpenCV — tanpa LUT linearitas, flat field, atau pemisahan (kamera monokrom: skala abu-abu 1-saluran). File TIFF yang disimpan memiliki urutan kanal **RGB**(RGB fotometrik, sehingga penampil gambar apa pun menampilkan warna asli); sedangkan array dalam memori yang dikembalikan oleh `grab()` / `frame_stream()` berada dalam urutan**BGR** — balik urutan kanal (`frame[..., ::-1]`) untuk membandingkannya dengan berkas yang disimpan. `chloros-cli process` menghasilkan keluaran yang sama dengan `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm melalui rantai radiometrik lengkap. Hanya multispektral — kamera RGB dilewati. |
| `"reflectance"` | uint16 0..32768 (siap untuk Pix4D); memerlukan pemasangan DAQ langsung sebagai referensi absolut. Hanya multispektral. |
| `"display"` | Rantai lengkap yang sesuai dengan pratinjau GUI (CCM + WB + gamma sesuai profil kamera). |
| `"all"` | **Satu berkas per tingkat yang berlaku** untuk setiap kamera (sesuai dengan pengaturan default &quot;Capture All&quot; di antarmuka pengguna / CLI). Berkas `CaptureResult` yang dihasilkan kemudian berisi satu kamus bingkai per `(cam, level)`, dengan tingkat yang tercantum dalam setiap kamus; tingkat yang tidak berlaku muncul di `.skipped`. Pembacaan DAQ yang digunakan untuk penangkapan disimpan sebagai sidecar `.daq`. |

> **Catatan — pengaturan default berbeda dari CLI.** `ArraySession.capture()` secara default diatur ke `processing="debayered"`; perintah `chloros-cli lattice array-capture` secara default diatur ke `processing="all"`. Berikan `processing="all"` secara eksplisit dari SDK untuk meniru penyimpanan multi-level pada CLI/GUI.

##### Nama berkas penangkapan array

Setiap tingkat disimpan dalam subfolder tersendiri di bawah `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` untuk `display`, `index/`, `composite/`). Setiap nama berkas mencantumkan nomor seri kamera sebagai `SN<serial>-<filter>`: nomor seri, kemudian bagian filter dari model kamera — `SN214000533-FOCN` untuk `M3C-L87-FOCN`, `SN261902142-F450` untuk `M3M-L87-F450` — digabungkan dengan `-`, sehingga nama file memiliki bidang-bidang yang dipisahkan oleh `_` seperti sebelumnya. Kamera yang modelnya tidak dapat dibaca hanya berupa `SN<serial>`, dan tangkapan dari rilis sebelumnya tidak memiliki filter dalam namanya. Nama file bergantung pada berapa banyak tingkat yang diminta oleh anggota:

| Tingkat yang diminta anggota | Nama file | Contoh (`reflectance`, default `format="tiff"`) |
| --- | --- | --- |
| **Satu tingkat** (`processing="reflectance"`, atau `levels` dengan satu item) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — tanpa akhiran tingkat, seperti pada rilis sebelumnya, sehingga skrip yang ada tetap berfungsi | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Beberapa tingkat** (`processing="all"`, atau `levels` dengan lebih dari satu item) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

[clamp mask](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) mengikuti nama produk, `clamp_masks/<product folder>/<product name>_clampmask.tif`: penangkapan reflektansipenangkapan reflektansi satu tingkat menulis `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, sedangkan `processing="all"` menulis `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. Aturan sufiks ditentukan per anggota berdasarkan level yang diminta, sehingga level yang dilewati (misalnya reflektansi pada kamera RGB) tidak mengubahnya. Bacalah jalur dari frame yang dikembalikan daripada membangunnya. Pengambilan gambar dengan satu kamera (`CameraSession.capture()`) selalu menambahkan tingkat: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG/JPEG: hanya menerima produk tampilan dan indeks

PNG atau JPEG memiliki 8 bit per kanal. Permintaan untuk tingkat yang tidak dapat ditampung oleh format tersebut akan ditolak sebelum penangkapan dilakukan, disertai pesan yang menyebutkan tingkat tersebut dan TIFF sebagai format yang dapat menampungnya:

- **`radiance`**(float32) dan**`reflectance`**(uint16, `32768` = 1,0) — serta `processing="all"`, yang mencakup keduanya. SDK memeriksa hal ini pada argumen: `capture()` memicu**`ValueError`** tanpa permintaan (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` dari kamera yang melakukan streaming lebih dari 8 bit** (12-bit M3C, Mono12 M3M) — backend menolak ini, karena mengetahui kedalaman bit masing-masing kamera: Kode status HTTP 400 dengan `error_code` `FORMAT_CANNOT_HOLD_LEVEL`, dilaporkan sebagai `ChlorosConnectError` (`CameraSession` / `ArraySession`) atau `RuntimeError` (`CameraHandle` / `ArrayHandle`). Pengambilan gambar satu tingkat `debayered` dari satu kamera menghasilkan bingkai 8-bit untuk PNG/JPEG dan tidak ditolak.

Simpan data radiometrik sebagai `format="tiff"` (`ext=".tiff"`); gunakan PNG/JPEG untuk `display` dan hasil render indeks.

#### Cara Kerja Eksposur Otomatis

- **Eksposur terlebih dahulu, gain terakhir.** Saat gain diatur ke Otomatis, gain tetap pada 0 dB hingga eksposur mencapai nilai terpanjang yang diizinkan.
- **Pengaturan eksposur manual akan membiarkan gain tetap pada mode Otomatis** hingga Anda mengubah pengaturan gain atau mengaturnya ke mode manual. Saat kembali ke mode Otomatis, batas gain akan kembali seperti semula.
- **Sorotan kecil tidak menggelapkan adegan.** Sorotan spekular hingga 1% dari bingkai diabaikan, dan pada panel target kalibrasi yang diketahui, hingga 0,5% dari satu panel.
- **Batas atas eksposur DAQ hanya berlaku saat target kalibrasi terlihat.** Hal ini mencegah panel putih mengalami clipping; jika tidak ada panel yang terlihat, kamera akan menyesuaikan eksposur sesuai dengan adegan.
- **Cahaya konstan tetap stabil, dan perubahan cahaya akan stabil dalam hitungan detik.** Pada cahaya konstan, eksposur dan batasnya tetap stabil, dan setelah terjadi perubahan cahaya yang besar, tampilan akan stabil dalam beberapa detik. Kliping tidak lagi membatasi eksposur secara permanen, dan perubahan ROI akan menyesuaikan eksposur kembali.
- **Target kecerahan tetap dalam jangkauan.** Target ini tidak meningkat saat kamera berada pada batas eksposurnya dan penguatannya tidak dapat naik, sehingga cahaya yang kembali tidak akan menyebabkan flare pada bingkai.
- **Kamera yang eksposur otomatisnya berhenti merespons akan dipertahankan pada tingkat yang sesuai dengan cahaya di sekitarnya.** Di bawah sinar matahari yang terik, kamera bisa turun ke eksposur terpendeknya dan tetap di sana dengan bingkai yang gelap, sementara semua pengaturannya menunjukkan nilai normal. Chloros memantau bingkai, mempertahankan kamera pada tingkat yang disesuaikan dengan cahaya tersebut, dan melepaskannya saat cahaya berubah. Sensor cahaya yang gelap atau tertutup tidak mengganggu proses ini.

#### Eksposur pada Anggota Array

Sebuah kamera LATTICE membatasi `ExposureTime` pada satu periode dari laju bingkainya. Saat sebuah array melakukan streaming, Chloros menjalankan setiap anggota dengan `AcquisitionFrameRateEnable` aktif, dengan laju yang sedikit di atas laju pemicu, sehingga batas atas paparan suatu anggota berada sedikit di bawah periode pemicu, **di ROI mana pun**:

- Periode bingkai suatu anggota adalah periode pemicu dikurangi kelonggaran sebesar 15% dari periode tersebut (minimal 3 ms, maksimal setengah periode), dan batas atas eksposurnya adalah periode bingkai tersebut dikurangi sekitar 0,1 ms. Contoh: pemicu 10 fps (100 ms) menghasilkan laju anggota sekitar 11,8 fps dan batas paparan sekitar 85 ms.
- **Pengaturan manual `ExposureTime` yang lebih lama dari batas tersebut akan memperlambat pemicu agar sesuai.** Chloros terlebih dahulu membuka batas anggota tersebut untuk nilai tersebut dan membiarkan laju yang lebih lambat berjalan selama beberapa periode pemicu (penulisan dapat memakan waktu beberapa detik lebih lama), kemudian menuliskannya, dan pemicu tidak akan bekerja lebih cepat dari yang diizinkan oleh batas waktu pemaparan. Pada susunan laju otomatis, laju yang sebelumnya berlaku akan kembali setelah setiap anggota yang laju pemicunya diperlambat oleh pengaturan manual kembali ke pemaparan otomatis. Nilai yang tidak dapat diambil oleh kamera itu sendiri tetap ditolak, dengan rentang yang diterima sekarang masuk ke dalam kesalahan, dan eksposur otomatis anggota tersebut dikembalikan jika sebelumnya dimatikan hanya untuk penulisan tersebut. Sebelum versi 1.3.0, ROI kecil membatasi setiap anggota pada periode free-run sensor itu sendiri, terlepas dari laju pemicu: 4,9 ms pada 768 × 304, sehingga tidak ada anggota yang dapat dipercerah.
- Chloros tidak pernah memperpendek eksposur manual agar sesuai dengan pemicu yang lebih cepat. Laju otomatis tetap berada dalam batas yang diizinkan oleh eksposur manual; laju yang dikunci yang lebih cepat daripada yang diizinkan oleh eksposur akan menetapkan batas atas yang cukup lama bagi anggota tersebut, dan pemicu yang diambil tidak akan lebih cepat daripada yang diizinkan oleh eksposurnya. Kurangi eksposur, atau laju yang dikunci, ketika setiap anggota harus merespons setiap pemicu.
- **Pembacaan Kembali:** Membaca node-node anggota menunjukkan `AcquisitionFrameRateEnable` True dan `AcquisitionFrameRate` sesuai laju anggota tersebut; izin (enable) akan dinonaktifkan kembali saat array berhenti.
- **Array tersebut memiliki kedua node tersebut. Jangan menulis `AcquisitionFrameRateEnable` atau `AcquisitionFrameRate` pada anggota array** — atur laju arraydengan `target_fps` / `set_target_fps()`.

Tulis eksposur anggota dengan `arr.set_member_settings(serial, exposure_time=20000)` (kata kunci dari `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` melakukan hal yang sama). Anggota tersebut tetap berada di dalam array. Jangan membuka anggota array dengan `connect_camera()`: connect menolak kamera yang sudah ada dalam array, dan `CameraSession` yang akan dikembalikan akan menghapus kamera dari array-nya saat terputus. `ArraySession.status()["member_ae"]` melaporkan eksposur setiap anggotapaparannya dan gain-nya dari frame terbarunya.

### Satu catatan proyek, semua mode

Folder proyek menyimpan pengaturan setiap perangkat: `cameras.json` — pengaturan setiap kamera (`cameras[].settings`, untuk satu kamera dan untuk setiap anggota array) serta pengaturan yang berlaku untuk seluruh array (`arrays[].capture`: format, ROI, binning, laju pemicu, anggaran kabel) — serta `sensors.json`, pengaturan setiap sensor cahaya (waktu integrasi, rata-rata bingkai, batas). Aplikasi, CLI, SDK, dan server MCP semuanya membaca dan menulis file yang sama, dengan cara yang sama:

- **Perangkat yang terhubung dalam suatu proyek akan disimpan di dalamnya dan mencatat setiap perubahan ke dalamnya.** &quot;Di bawah sebuah proyek&quot; berarti dengan proyek yang terbuka di backend (aplikasi), atau — melalui `ChlorosProject` (`open_project()`), yang menamai foldernya sendiri ke backend (`project_path`). Kamera, array, atau sensor cahaya yang belum dimiliki proyek akan ditambahkan ke berkas-berkasnya saat terhubung (`connect_camera()`, `connect_array()`, koneksi sensor cahaya, CLI, aplikasi, `chloros-mcp`), beserta pengaturannya. Perubahan dari mode apa pun (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, pengaturan array, `set_cap()`, CLI, kontrol aplikasi) akan disimpan ke dalam berkas proyek tersebut beberapa saat kemudian. Pengaturan eksposur, gain, atau target AE yang ditetapkan pada kamera disimpan sebagai pengaturan yang disimpan, sehingga saat dibuka kembali, pengaturan tersebut akan diterapkan kembali.
- **Membuka sebuah proyek akan mengatur perangkatnya berdasarkan proyek tersebut**, di setiap mode: kamera, array, atau sensor cahaya yang terhubung tanpa pengaturan tersendiri akan mendapatkanyang tersimpan (`lattice cam-connect` / `array-connect` saat proyek terbuka, `chloros-cli project connect`, `open_project().connect_all()`, saat aplikasi dibuka kembali). Sebuah array yang terhubung dengan ID baru akan memperbarui array yang disimpan proyek dengan kamera yang sama; array dengan kamera lain akan ditambahkan di sampingnya.
- **Penyimpanan aplikasi sendiri tidak pernah menimpa pengaturan langsung perangkat** dengan salinan lamanya, dan mempertahankan penggantian per-anggota yang disimpan oleh pemanggil CLI atau SDK.
- **Format piksel yang Anda tetapkan adalah yang disimpan oleh proyek.** Format yang ditetapkan di Pengaturan Array, melalui `--pixel-format` atau `connect_array(pixel_format=)`—termasuk 8-bit—akan disimpan dan dihubungkan kembali persis seperti aslinya. Jika tidak ada yang ditetapkan, setiap kamera akan terhubung dengan pengaturan default 12-bit (`BayerRG12`, `Mono12` pada sensor mono). Nama format menunjukkan kedalaman bitnya; keluarga format tersebut mengacu pada sensornya, sehingga kamera berwarna tidak pernah disetel ke format Mono atau kamera mono ke format Bayer, dan baris `[PixelFormat <serial>] ... written` menunjukkan hal tersebut saat format telah dipetakan. Ekspor dari perekaman 8-bit berformat 8-bit, termasuk reflektansi: diskalakan ulang dan diberi cap dengan `PixelScale`-nya.
- Perangkat yang terhubung tanpa proyek tidak mencatat apa pun.
- **Perekaman mengikuti proyek yang dibuat setelahnya; jika dibuka, perekaman tersebut tetap berada dalam proyeknya.**Saat sebuah proyek dibuat — `create_project()`, `chloros-mcp`, `create_project`, atau fitur**Proyek Baru**— atau diduplikasi (fitur**Duplicate**pada aplikasi, yang membuka salinannya) sementara rekaman masih menulis ke dalam proyek yang sebelumnya terbuka, rekaman tersebut tetap berlanjut, dan apa yang ditulisnya akan masuk ke dalam proyek baru, di tempat yang sama di dalamnya (di bawah). Membuka proyek yang sudah ada — `ChlorosProject.process()` (yang membuka proyeknya di bagian belakang), `chloros-cli process`, opsi**Buka Proyek** pada aplikasi — membuat perekaman tetap berjalan dengan proyek yang menjadi tujuannya: perekaman selesai di sana, dan berkas-berkasnya bergabung dengan daftar berkas proyek tersebut (aplikasi menghentikan setiap perekaman sebelum membuka sebuah proyek).
  - `ArraySession.record()` dan `ArraySession.burst()`: video, atau folder burst yang berisi setiap frame, dipindahkan secara utuh ke sana saat perekaman berhenti. Hasil penghentian tersebut melaporkan jalur baru (`out_dir`, `video_path`), dan proses pembuatan video dari burst tersebut dilakukan dari sana. Jika pemindahan ditolak oleh drive, rekaman tetap berada di lokasi aslinya, dan hasilnya menunjukkan hal tersebut (`relocation_error`). Frame mentah dari burst yang dihentikan serta `.daq` ditambahkan ke daftar file proyek tempatnya berakhir; video tidak ditambahkan ke daftar file proyek.
  - `ArraySession.continuous()`: setiap rekaman setelah perubahan disimpan ke dalam proyek baru; rekaman yang disimpan sebelumnya tetap berada di tempatnya.
  - `DAQSensorSession.record_start()` (dan rekaman sensor cahaya bawaan aplikasi): `.daq` dipindahkan ke folder `light_sensor` proyek baru saat perekaman berhenti; `record_stop()` mengembalikan jalur baru, dan berkas tersebut ditambahkan ke daftar berkas proyek tersebut.

  Perekaman ke dalam folder di luar proyek yang terbuka tetap berada di tempatnya, dan `start_camera_continuous()` / tugas multi-stream mempertahankan folder yang telah ditentukan.
- **Di dalam aplikasi, hasil perekaman setelah proyek baru dibuat akan masuk ke dalamnya.** Membuat atau menduplikasi proyek akan mempertahankan semua proses perekaman yang sedang berjalan, dan setiap perekaman yang dilakukan setelahnya (Capture All, Start Capturing) akan disimpan ke dalam proyek baru tersebut dan ditambahkan ke daftar berkasnya. Membuka proyek yang sudah ada di aplikasi akan menghentikan semuanya terlebih dahulu.
- **Proyek tidak dipindahkan saat sedang merekam.**Memindahkan proyek yang terbuka ke direktori kerja lain (melalui**Pengaturan → Direktori Kerja** di aplikasi; panggilan SDK tidak memindahkan proyek, klien HTTP yang mengirim data ke `/api/move-project` melakukannya) akan ditolak dengan kode HTTP 409 `PROJECT_RECORDING` selama perekaman array atau burst, pengambilan gambar beruntun array, pekerjaan kamera beruntun/multi-stream, atau perekaman sensor cahaya sedang menulis ke dalamnya: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Hal ini terpisah dari pengaturan kamera yang tersisa saat terputus (lihat [Memori Kamera](#camera-memory--keep_settings--clear_settings)): sebuah proyek mengatur kameranya sendiri saat dibuka.

### Memori kamera — `keep_settings` / `clear_settings`

Saat sebuah array terhubung, kamera-kameranya akan mengadopsi pengaturan array tersebut, dan Anda kemudian dapat mengubahnya. Ketika Chloros melepaskan kamera — `ArraySession.disconnect()` / `force_disconnect()`, akhir dari fungsi sementara, penghentian `start_camera_continuous()` — sistem akan mengembalikan setiap kamera ke kondisi semula sebelum terhubung: kabel pemicu dan jalur sinkronisasi, serta pengaturan PTP dan kamera — `ExposureAuto` / `GainAuto` (serta pengaturan manual `ExposureTime` / `Gain`), `TargetBrightness` serta batas AE dan redaman, keseimbangan putih, tingkat hitam, `PixelFormat`, penggabungan piksel dan ROI, alur kerja warna (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, `PtpSlaveOnly` dan `PtpDomainNumber` dari PTP, ukuran paket (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), kecepatan (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), data chunk (`ChunkModeActive`, masing-masing `ChunkEnable` di bawah `ChunkSelector`), penghilang gangguan pada input pemicu, dan sebuah array memberikan jalur sinkronisasi kepada perangkat slave (`LineFilterSelector` / `LineFilterWidth`) dan laju bingkai akuisisi (`AcquisitionFrameRate` hanya jika pengaktifannya menyala: jika dimatikan, laju yang terbaca adalah nilai maksimum yang diizinkan oleh pengaturan kecepatan, bukan pengaturan). Hanya perbedaan yang dicatat: kamera berhenti terlebih dahulu, pengaturan kecepatan, dan laju bingkai sebelum eksposur yang memerlukan periode bingkai tersebut.

Untuk menggunakan kamera di luar proyek, dua argumen menulis ke **memori internal** kamera — data yang digunakan saat dinyalakan, di perangkat lunak apa pun (pada `disconnect()` / `force_disconnect()`, setiap fungsi sementara, `start_camera_continuous()`, dan `ChlorosProject.disconnect_all()`; MCP: argumen yang sama dengan `disconnect_device` dan alat-alat sementara; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** menyimpan pengaturan setiap kamera sebagaimana adanya saat sesi berakhir ke dalam memorinya (UserSet2, dijadikan set boot). Kamera-kamera tersebut tetap dibiarkan berjalan bebas (`TriggerMode` `Off`), dengan pengkabelan jalur sinkronisasi seperti yang ditemukan. Kamera yang tidak memiliki UserSet2 akan menyimpan pengaturannya ke UserSet1; komputer ini mengingatnya, sehingga fitur pemulihan otomatis array Chloros — yang mereset UserSet1 yang telah menyimpang dari baseline Chloros — tidak akan mengubahnya.
- **`clear_settings=True`** mereset memori setiap kamera ke pengaturan pabrik: baseline Chloros, yang menjadi dasar proses pemulihan otomatis array-nya — pengaturan default pabrik dengan `TriggerMode` `Off`, `TriggerSource` `Line2`, eksposur otomatis, serta keseimbangan putih yang telah dikalibrasi pada kamera RGB dijaga — di UserSet1, dijadikan set boot.
- Tidak keduanya (`ValueError`; backend merespons dengan kode 400). Balasan tersebut berisi `camera_memory`, sesuai kamera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` atau `{"cleared": bool, "errors": [...]}` — `force_disconnect()` mengembalikannya; untuk suatu tugas, nilai tersebut disertai dengan `stop()`.

Aplikasi ini tidak menyediakan nilai-nilai tersebut.

**Setiap jalur keluar akan mengembalikan kamera ke posisi semula.**Proses pengembalian ke posisi semula dilakukan pada setiap jalur keluar: `CameraSession.disconnect()` (kamera**X**dari aplikasi),**X** pada sebuah array atau anggota array mana pun (seluruh array dilepaskan), menutup aplikasi (backend memiliki waktu hingga 20 detik untuk mengembalikan setiap kamera, array, dan perekaman berkelanjutan yang sedang berjalan; fungsi sementara atau panggilan satu kali yang masih berjalan akan berhenti pada titik aman berikutnya, mengembalikan kameranya terlebih dahulu, selama maksimal 8 detik, dan merespons HTTP 503 `SHUTTING_DOWN`; sebuah `connect_array()` yang masih berjalan akan berhenti sebelum kamera berikutnya dan mengembalikan kamera yang telah dibuka, selama maksimal 10 detik, serta memicu `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); dan yang tiba selama proses penutupan tidak membuka apa pun: HTTP 503 `SHUTTING_DOWN`), serta perintah `chloros-cli` yang dihentikan paksa atau terminalnya ditutup (tugasnya dihentikan dan kameranya dikembalikan terlebih dahulu). Backend yang mengalami crash atau dihentikan paksa tidak dapat mengembalikan apa pun; siklus daya akan mengembalikan setiap kamera ke memorinya.**Sebuah proyek mengatur kameranya sendiri.** Ketika sebuah array terhubung melalui sebuah proyek — proyek yang terbuka di aplikasi, atau sebuah `ChlorosProject`, yang mendaftarkan dirinya ke backend (`project_path`) — setiap anggota mendapatkan pengaturan yang disimpan proyek untuknya: target eksposur, gain, atau AE yang diterapkan padanya (disimpan sebagai milik Anda, sehingga eksposur otomatis tidak mengubahnya) dan mode white balance-nya (koreksi Chloros; keseimbangan putih bawaan kamera tetap dinonaktifkan), apa pun yang tersimpan dalam memori kamera.

### Mode Pengambilan Gambar &amp; Perekam

Antarmuka array mencerminkan panel pengambilan gambar di GUI: mode rana Tunggal / Berkelanjutan / Interval / Tercepat, ditambah dua perekam (video komposit langsung dan burst mentah → pemrosesan ulang offline).

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

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**adalah loop Kontinu/Interval dari SDK. Karena tidak ada `Ctrl+C` untuk menghentikannya dari skrip, Anda**harus** meneruskan `count` dan/atau `duration_s` (proses akan berhenti saat salah satunya tercapai). `interval_s` diukur sejak awal setiap putaran (sesuai dengan antarmuka pengguna). Putaran berkelanjutan (`interval_s=0`) menghubungkan `group_seq` → `after_seq`, sehingga merupakan kelompok-kelompok yang disinkronkan secara berurutan pada laju array. Kwargs yang tersisa diteruskan langsung ke `capture()`.
- **`continuous`**adalah penangkapan berkelanjutan bawaan backend — mode Berkelanjutan pada GUI.**Gunakan ini daripada `capture_repeated()` jika Anda ingin setiap pemicu disimpan ke disk dengan laju bingkai tinggi:** `capture_repeated()` melakukan satu permintaan HTTP per grup dari skrip Anda, sedangkan `continuous()` menjaga dua penangkapan tetap berjalan di dalam backend tanpa perjalanan bolak-balik antar grup. Diukur pada ROI 512 pada susunan 9 kamera: 73–77 penangkapan/detik, dibandingkan dengan susunan yang menghasilkan sekitar 78. Gunakan `capture_repeated()` untuk penangkapan interval (`interval_s`) dan untuk pengambilan gambar yang Anda periksa satu per satu. Di Windows, kecualikan backend dari Microsoft Defender untuk laju tinggi (lihat [Langkah 1](#step-1--install-the-chloros-platform-package)).
- **`record`** adalah *kelas pemantauan*: ia merekam komposit indeks gabungan langsung yang ditampilkan. Dengan tampilan gabungan antarmuka pengguna (GUI) yang terbuka, backend merekam tampilan tersebut sebagaimana ditampilkan. Jika tidak ada yang terbuka, backend merender tampilan itu sendiri: `expr` jika Anda memberikan satu, sebaliknya NDVI dari array (pita merah dan NIR terdekat dengan 660 nm dan 850 nm, sebagaimana ditampilkan dalam tampilan langsung), setelah array tersebut disejajarkan. Beri nama pita dalam `expr` sesuai dengan penamaan array: `<band>_<filter>` untuk kamera M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` untuk M3M (`Mono_850`); nama warna pendek (`NIR`, `Red`) hanya berfungsi jika satu band dari array tersebut memilikinya. Tampilan GUI yang dibuka selama perekaman akan direkam selama tetap terbuka. Perekaman yang tidak menulis frame apa pun akan gagal dengan `error` `"no-frames: …"` (array mungkin masih dalam proses penyelarasan atau penyesuaian eksposurnya), dan `stop()` akan memicu kesalahan (lihat [`RecorderHandle`](#recorderhandle)). Satu perekam komposit per array (akan memicu kesalahan jika sudah ada yang berjalan).
- **`burst` → `build_video`** adalah *tingkat analisis*: `burst` menulis frame mentah + manifes per frame + satu `.daq` per pembacaan DLS yang berbeda di bawah `<output>/bursts/<base>/` pada kecepatan penuh loop pengambilan (tanpa rantai, tanpa exiftool, tanpa tampilan langsung). `build_video` mencocokkan waktu setiap bingkai dengan `.daq` terdekat dan menjalankan kembali rantai radiance/reflektansi/indeks. `products` adalah daftar `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (default: indeks gabungan). `burst().stop()` juga secara otomatis memicu pembuatan indeks gabungan, yang dikembalikan sebagai `build_job` dalam hasil penghentian.

##### Hasil `build_video`

| Kunci | Arti |
| --- | --- |
| `success` | `True` hanya jika setiap produk yang diminta telah dibuat sesuai permintaan: setidaknya satu keluaran, tidak ada apa pun di `skipped`, dan tidak ada di `degraded`. |
| `outputs` | Jalur video (dan GIF) yang telah ditulis. |
| `errors` | Masalah, dalam bentuk teks. |
| `skipped` | Satu `{"serial", "level", "reason"}` per produk yang diminta yang **tidak** menghasilkan video — masalah kamera (tidak ada paket kalibrasi untuk kamera tersebut, tidak ada pembacaan DLS untuk reflektansi, rantai pemrosesan gagal) atau indeks gabungan, yang mencakup `"kind": "combined"` dan `serial` `None` (tidak ada pasangan pita merah/NIR di antara kamera-kamera tersebut, tidak ada kelompok dengan radian untuk kedua pita tersebut). Entri apa pun akan menghasilkan `success` `False`; video lainnya tetap ditulis dan dicantumkan. |
| `substitutions` | Kamera dirender pada tingkat yang berbeda dari yang diminta, dan alasannya — kamera RGB yang diminta memiliki radian/reflektansi akan mendapatkan video tampilan yang telah disesuaikan keseimbangan putihnya jika paket kalibrasi kamera tersedia. Bukan kegagalan. |
| `notes` | Teks informatif, bukan masalah — misalnya, kamera tanpa ekspresi indeks per-kamera tidak memiliki video indeks, atau video tampilan kamera RGB tidak diimbangi putih (tidak ada paket kalibrasi, atau paket kalibrasi tanpa keseimbangan putih pabrik), dengan menyebutkan nama kamera. Bukan kegagalan. |
| `degraded` | `True` ketika suatu produk telah ditulis tetapi bukan yang diminta: indeks gabungan offline adalah selisih ternormalisasi merah/NIR bawaan, sehingga ekspresi indeks yang dikonfigurasi tidak dievaluasi. Menyebabkan `success` `False`. |
| `combined` | Ringkasan indeks gabungan: pita yang digunakan, rentang LUT, bingkai yang dibuat. |
| `clamp_masks` | Jalur [masker penjepit unmix](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) dariTIFF yang dikalibrasi per-frame (`save_tiffs=True`), di `clamp_masks/<level>/` di samping folder `<level>/` milik TIFF tersebut, satu per TIFF di mana ada piksel yang di-clamp. Sebuah video tidak memuatnya. |
| `clamp_fractions` | Informasi yang disertakan dalam video sebagai pengganti masker: satu berkas `{"kind", "serial", "level", "frames"}` per produk video (`kind` `"combined"` untuk kamera-kamera anggota indeks gabungan), `frames` daftar `[frame, R, G, B]` — bagian piksel pada bingkai tersebut yang dibatasi oleh proses unmix dalam rentang yang ditampung oleh kanal R / G / B-nya (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Dikembalikan oleh `ArraySession.record()` dan `ArraySession.burst()`. Gunakan sebagai pengelola konteks untuk menghentikan secara otomatis saat keluar dari lingkup, atau jalankan secara manual.

| Anggota | Deskripsi |
| --- | --- |
| `job_id` | ID pekerjaan backend (str). |
| `kind` | `"composite"` (dari `record`) atau `"raw"` (dari `burst`). |
| `start_stats` | Kamus yang dikembalikan oleh panggilan `start`. |
| `result` | `None` selama proses berjalan; kamus hasil akhir setelah dihentikan. |
| `stats(timeout=10.0)` | Statistik pekerjaan secara real-time (frame yang ditulis, fps yang tercapai, waktu yang berlalu). |
| `stop(timeout=60.0)` | Menghentikan perekam; mengembalikan dan menyimpan hasil akhir (`.result` tetap menjadi `None` jika permintaan penghentian itu sendiri gagal). Untuk perekaman `record()` yang gagal — yaitu yang tidak menulis frame apa pun (`error` `"no-frames: …"`) — sistem menyimpan hasil di `.result`, lalu memicu `ChlorosConnectError`. Idempoten (panggilan kedua mengembalikan hasil yang disimpan, tanpa memicu kesalahan). Meninggalkan blok `with` akan menghentikan perekam tanpa memicu kesalahan, jadi periksa `rec.result.get("error")` setelahnya. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Dikembalikan oleh `ArraySession.continuous()`. Gunakan sebagai pengelola konteks — keluar dari blok `with` akan menghentikan penangkapan jika masih berjalan — atau jalankan dengan `status()` / `wait()` / `stop()`. Satu proses penangkapan berkelanjutan berjalan per array pada satu waktu.

| Anggota | Deskripsi |
| --- | --- |
| `start_status` | Kamus status yang dikembalikan oleh panggilan start. |
| `result` | `None` selama proses berjalan; kamus status akhir setelah proses penangkapan selesai (ditetapkan oleh `stop()`, atau oleh `status()` / `wait()` yang mendeteksi berakhirnya proses tersebut). |
| `status(timeout=10.0)` | Dikt status aktif (bidang di bawah ini). |
| `wait(timeout=None, poll_s=0.5)` | Menunggu hingga proses penangkapan berakhir dengan sendirinya — `max_captures` atau `duration_s` tercapai, array habis, drive penuh, atau terjadi kesalahan — dan mengembalikan status akhir. Dengan `timeout`, kembalikan status terbaru setelah sekian detik meskipun proses masih berjalan: periksa `running`. |
| `stop(timeout=30.0)` | Hentikan penangkapan dan kembalikan status akhirnya. Idempoten (panggilan kedua akan mengembalikan hasil yang disimpan dalam cache). |

Kolom status:

| Kolom | Arti |
| --- | --- |
| `running` | Benar selama proses penangkapan berjalan. |
| `captures` | Grup yang disinkronkan telah disimpan. |
| `files` | File yang telah ditulis. |
| `failed_files` | Berkas yang tidak dapat ditulis (seperti pada [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)). |
| `failed_captures` | Upaya penangkapan yang gagal secara keseluruhan, termasuk yang dicoba ulang (HTTP503 saat array tidak memiliki grup baru tepat waktu). |
| `skipped_groups` | Grup yang dipublikasikan oleh array tetapi tidak tersimpan (proses penangkapan tertinggal). |
| `captures_per_s` | Tingkat penyimpanan rata-rata sejak awal. |
| `elapsed_s` / `started_at` | Jumlah detik yang telah berjalan; waktu mulai (detik epoch Unix). |
| `last_group_seq` | Kelompok terbaru yang disinkronkan dan disimpan. |
| `last_error` | Kesalahan terbaru, atau `None`. |
| `stop_reason` | Setelah selesai: `count` (`max_captures` tercapai), `duration` (`duration_s` tercapai), `stopped` (`stop()` dipanggil), `array_gone` (array terputus), `error` (20 kegagalan berturut-turut, atau permintaan yang ditolak oleh backend), `disk_full` (file yang tidak muat di drive). `None` saat sedang berjalan. |
| `max_captures` / `duration_s` | Batas yang digunakan saat dimulai (`0` = tidak ada). |
| `defender_cores` | Jumlah inti CPU yang digunakan oleh Microsoft Defender selama jendela waktu 10 detik terakhir; `None` hingga terukur, dan selalu `None` pada host tanpa Defender. |
| `defender_warned` | Bernilai true setelah penangkapan mencatat bahwa Defender memperlambatnya (pada 1 inti atau lebih). Kecualikan backend dari pemindaian Defender — lihat [Langkah 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Frame Langsung dalam Pipa Pengolahan Anda Sendiri

Untuk memasukkan frame yang disinkronkan ke dalam kode Anda sendiri — pembelajaran mesin, penampil khusus, perekam Anda sendiri — gunakan `ArraySession.frame_stream()`. Setiap tick menghasilkan `GrabResult`: `{serial: numpy array}` untuk setiap kamera, **semuanya dari satu pemicu**, dengan ID pemicu tersebut terdapat pada `.group_id`. Ini merupakan implementasi yang sama dengan `ArrayHandle.frame_stream()` pada API proyek, sehingga semua yang tercantum di [Satu pemicu per pembacaan](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align), dan [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) berlaku untuk keduanya.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Setiap set yang dihasilkan merupakan pemicu baru.** `frames.group_id` tidak pernah berulang dalam satu aliran; ketika Anda meminta lebih banyak `fps` daripada yang dijalankan oleh array, aliran tersebut akan menunggu pemicu berikutnya, sehingga berjalan pada kecepatan yang lebih lambat di antara keduanya.
- **Kamera yang hilang disebutkan namanya, bukan diisi ulang.** Kamera yang tidak dapat dilayani pada pemicu tersebut tidak ada dalam kamus dan tercantum dalam `frames.errors`.
- **Proses ini berakhir secara tiba-tiba.** `FrameFetchError` akan diulang pada tick berikutnya jika terjadi penolakan, setelah 60 detik tanpa frame, atau setelah 60 detik tanpa pemicu baru.**Hasil yang dihasilkan.**`frame_stream()` mentransfer seluruh frame numpy melalui koneksi HTTP lokal dan mencapai kecepatan maksimum sekitar**170 MB/s**. Kecepatan tersebut cukup untuk pemrosesan langsung frame mentah, atau array yang telah dibin atau ROI, tetapi tidak cukup untuk setiap pemicu pada resolusi penuh — untuk menyimpan**setiap** pemicu, simpan ke disk dengan [`continuous()`](#continuoushandle). Diukur pada susunan empat kamera (FOCN, FRGN, FRGB, F450) dengan pengiriman terkompresi; setiap tick merupakan satu set pemicu lengkap:

| | Sensor penuh (array 14,2–14,4 fps) | Binning 2 × 2 (array ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 set/detik | 22,2 set/detik |
| `frame_stream(processing="debayered")` | 2,3 set/detik | 9,3 set/detik |
| `frame_stream(processing="display")` | 0,8 set/detik | 3,0 set/detik |
| MJPEG per kamera, 4 aliran pada 720p | masing-masing 13,5 fps; setiap bingkai disinkronkan dengan pemicu | masing-masing ~25 fps; ~45% pemicu ada di keempatnya |

Gunakan `processing="raw"` untuk throughput dan demosaic di sisi Anda; level `debayered` juga menghilangkan offset gelap dari paket kalibrasi, yang tidak dilakukan oleh demosaik pada `raw`.

#### Aliran MJPEG per kamera (tanpa RTSP)

**RTSP tidak didukung.** Antarmuka streaming menggunakan MJPEG melalui HTTP (`multipart/x-mixed-replace`, satu file JPEG per bagian, setiap bagian dengan `Content-Length`), satu URL per kamera:

| URL | Apa yang disajikan |
| --- | --- |
| `GET /api/camera/stream/<serial>` | Pratinjau 8-bit, hingga ~30 fps. Untuk anggota array, `max_h` (default 720, 64–2160) dan `jpeg_q` (default 80) menentukan ukuran dan kualitas. |
| `GET /api/camera/<serial>/stream-annotated` | Pratinjau yang sama dengan overlay GUI yang ditampilkan di atasnya (zebra, kisi, garis bidik, histogram, peaking, pengukur titik). `ChlorosLocal.camera_stream()` membacanya untuk Anda. |
| `GET /api/camera/<serial>/stream-reflectance` | Reflektansi 8-bit per-band — lihat [Aliran reflektansi langsung](#live-reflectance-stream-mjpeg). |

- **Header pemicu.** Untuk anggota array, setiap bagian dari ketiga aliran tersebut membawa `X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, dan `X-Chloros-Timestamp-Ns`. Bagian dari kamera berbeda dengan `X-Chloros-Group-Id` yang sama merupakan pemicu yang sama: kumpulkan bagian-bagian tersebut dan kelompokkan berdasarkan header tersebut untuk menyusun kembali satu set. Kamera mandiri tidak memiliki kelompok: bagian pratinjau tidak membawa header ini sama sekali (komponen reflektansinya memiliki header `X-Chloros-Group-Id` yang kosong).
- **Aliran pratinjau akan kehilangan frame saat beban tinggi.** Aliran ini menyajikan frame terbaru dan melewatkan frame yang tidak dapat dikirim tepat waktu, sehingga pada laju array yang tinggi, tidak semua pemicu mencapai aliran setiap kamera (lihat tabel di atas). Untuk setiap pemicu, gunakan `frame_stream()` atau `continuous()`.
- **`/api/camera/stream/<serial>` akan berhenti sementara saat aliran langsung gabungan array terbuka** (`/api/camera/array/stream/<array_id>` — tampilan array pada antarmuka pengguna (GUI) yang membukanya). Setiap aliran anggota kemudian mengirimkan satu bingkai penanda &quot;Paused (array stream active)&quot;, tanpa header `X-Chloros-*`, dan tidak ada lagi hingga aliran gabungan ditutup; aliran tersebut kemudian dilanjutkan secara otomatis. `stream-annotated` dan `stream-reflectance` tidak berhenti. Tutup tampilan array antarmuka pengguna (GUI) saat skrip membaca aliran pratinjau per kamera.
- **Untuk angka, bukan gambar, gunakan `frame_stream()`.**JPEG bersifat lossy dan 8-bit; `frame_stream()` mengembalikan hasil yang sama tanpa codec dalam jalurnya.

### Pengambilan Gambar Berinterval untuk Survei Drone

Penerbangan survei memerlukan satu kelompok gambar yang disinkronkan setiap beberapa meter perjalanan, bukan setiap bingkai yang dapat dihasilkan oleh array. Dua pengaturan mengontrol hal ini:**`target_fps`**mengatur seberapa sering kamera mengambil satu bingkai, dan**`capture_repeated(interval_s=…)`** mengatur seberapa sering satu bingkai disimpan.**Kamera terus melakukan streaming di antara proses penyimpanan.** Setiap interval menyimpan kelompok gambar sinkron terbaru; bingkai-bingkai di antaranya tidak disimpan. Pengaturan eksposur otomatis menyesuaikan pada setiap bingkai, sehingga eksposur sudah stabil saat penyimpanan terjadi. Tidak ada mode di mana kamera berhenti di antara pengambilan gambar dan diaktifkan kembali untuk setiap pengambilan: setiap pengambilan gambar harus menunggu aliran dimulai dan eksposur stabil.**Tetapkan laju bingkai sesuai kebutuhan interval.**Jika dibiarkan dalam mode otomatis, sistem akan berjalan pada laju tertinggi yang didukung oleh host dan jaringan. Menetapkan laju yang lebih rendah akan mengurangi panas kamera, beban jaringan, dan penggunaan CPU host tanpa mengubah apa yang disimpan. Pilih laju yang menghasilkan**2–4 bingkai per interval**:

- Setiap proses penyimpanan menunggu frame berikutnya setelah diminta, sehingga kelompok frame yang disimpan akan tiba hingga satu periode frame setelah waktu yang dijadwalkan. Kecepatan yang lebih tinggi akan memperkecil selisih waktu tersebut.
- Pengaturan eksposur otomatis dilakukan sekali per frame, sehingga kecepatan yang sangat rendah akan memperlambat respons eksposur terhadap awan yang melintas.

**Hitung interval berdasarkan rencana penerbangan:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Misalnya, area cakupan 60 m dengan tumpang tindih ke depan 80% berarti jarak antar baris 12 m; pada kecepatan 10 m/s, nilainya adalah `interval_s=1.2`, dan nilai `target_fps=3` yang dikunci menghasilkan antara 3 hingga 4 bingkai per interval. Pengambilan gambar didasarkan pada waktu: SDK tidak memiliki mode pemicu jarak atau GPS, jadi pertahankan kecepatan tanah yang stabil pada jalur survei.

**Simpan data mentah di pesawat dan kalibrasi nanti.** `interval_s` diukur dari awal setiap penyimpanan, sehingga penyimpanan yang memakan waktu lebih lama dari interval akan membuat penyimpanan berikutnya dimulai terlambat dan jarak antar-frame menjadi tidak konsisten. Menyimpan data mentah beserta pembacaan sensor cahaya (`processing="raw"`; pembacaan ini disimpan secara default) membuat setiap proses penyimpanan tetap singkat dan memastikan setiap frame dapat dikonversi menjadi reflektansi di kemudian hari.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**mengunci laju sejak awal;**`arr.set_target_fps(fps)`**mengubahnya di tengah penerbangan tanpa perlu terhubung ulang, dan**`arr.set_target_fps(None)`** mengembalikannya ke mode otomatis. Kecepatan di atas batas yang dapat ditangani oleh host dan jaringan akan diturunkan hingga batas tersebut — periksa `clamped` pada hasilnya — dan kecepatan terendah adalah 0,1 fps.
- **Jangan gunakan `burst()` untuk mengurangi jumlah frame.** Perintah ini menulis setiap frame yang dihasilkan oleh array; perintah ini mengikuti `target_fps` yang telah ditetapkan, sehingga hanya cocok digunakan jika setiap frame memang diinginkan.
- **`target_fps` tidak memperbaiki frame yang rusak.** Menurunkan nilainya akan mengurangi jumlah frame yang dihasilkan, bukan kecepatan pengiriman masing-masing frame; untuk frame yang rusak akibat GVSP, turunkan nilai `wire_ceiling_mbps` sebagai gantinya (lihat [Kesehatan Array](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Menyambungkan ke Array yang Sudah Terhubung — `attach_array`

Jika array sudah aktif (dibuka melalui GUI, atau sesi SDK sebelumnya telah memanggil `connect_array`), gunakan `attach_array` untuk mendapatkan handle ke array tersebut alih-alih terhubung ulang. `connect_array` selalu menghasilkan kesalahan &quot;Kamera  <sn>sudah ada dalam array<id>&quot; dalam situasi tersebut, karena mengirimkan POST ke `/array/connect` untuk anggota dalam kumpulan tidak bersifat idempoten; `attach_array` membaca `/api/camera/array/list` dan mencocokkannya berdasarkan array_id atau nomor seri.

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

Pola: Skrip SDK yang berbagi penyewa dengan GUI desktop sebaiknya mencoba `attach_array` terlebih dahulu dan beralih ke `connect_array` jika belum ada array di dalam pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Array yang terpasang dipinjam, bukan dimiliki.** Sebuah handle dari `attach_array` tidak pernah membongkar array: baik saat blok `with` berakhir maupun saat dilepaskan secara eksplisit oleh `disconnect()` (panggilan eksplisit tersebut mencetak satu baris ke stderr yang menyatakan bahwa masih menjalankan array tersebut). Aturan yang sama berlaku untuk `CameraSession` / `DAQSensorSession`. Jadi, penggunaan bersama dengan GUI aman baik dengan maupun tanpa `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Jika Anda memang bermaksud mengakhirinya — misalnya untuk terhubung kembali dengan `wire_ceiling_mbps` yang berbeda — panggil `arr.force_disconnect()`. Mencoba terhubung kembali setelah memanggil `disconnect()` biasa pada handle yang terhubung akan gagal dengan pesan &quot;Kamera <sn>sudah </sn>ada <sn>dalam array<id>&quot;, karena tidak ada yang dilepaskan.

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
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Backend tidak pernah dimulai: backend yang tidak merespons akan memicu `ChlorosConnectError`, sedangkan permintaan yang ditolak akan dikembalikan sebagai hasil. `packed` yang salah akan memicu `ValueError`.

`status` adalah salah satu dari `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (jika tidak, `error` — termasuk permintaan yang ditolak: belum masuk, `error_code: AUTH_REQUIRED`, atau paket di bawah batas minimum SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` berarti resolusi yang diminta hanya sesuai dengan cincin RX pada laju pemicu yang dibatasi — pertahankan resolusi dan lanjutkan dari `target_fps=result["recommended"]["recommended_target_fps"]` ke `connect_array` (lihat [Contoh 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` hanya muncul ketika frame yang diminta tidak dapat dikirim pada laju frame apa pun; connect tidak pernah memperkecil frame secara otomatis, jadi terapkan `recommended` sendiri.

**Cara membaca proyeksi** (model yang sama dengan panel Pengaturan Array di GUI):

- **Burst (`frame_bytes_total`) dijumlahkan per kamera pada format piksel asli masing-masing kamera.**Kamera**M3M**mono mengalirkan Mono12 (2 B/px), atau dengan `pixel_format`, format mono dari kedalaman kameranya (`"BayerRG8"` adalah `Mono8`, 1 bit per piksel), sehingga tanpa format kedalaman, bingkai resolusi penuh dari 4 kamera berukuran**~25 MB** dengan tiga kamera mono, bukan ~12,6 MB seperti yang diperkirakan dengan asumsi semua 8-bit. Backend menentukan format masing-masing kamera berdasarkan modelnya.
- **Admittance (`burst_fits_nic_ring`) memperhatikan beban drain**, bukan perbandingan burst penuh vs. ring: sim-emit berlaku saat host mengosongkan ring RX lebih cepat daripada kamera mengisinya. Host 10G + kamera 1 GbE**dapat menangani** resolusi penuh bahkan saat burst melebihi kapasitas ring; sedangkan host 1 GbE akan memblokir (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` adalah batas atas pengambilan serial yang konservatif** — `max(readout+emit, N×emit)` dengan emisi per kamera dibatasi pada tautan kamera 1 GbE, tidak bergantung pada eksposur. Misalnya, ~2,8 fps untuk susunan 4 kamera resolusi penuh 12-bit (sesuai dengan hasil pengukuran runtime ~2,7–3,0). Model lengkap: [Referensi CLI → Model fps &amp; burst array](cli-reference.md#array-fps--burst-model).
- **Setelah perekaman berkelanjutan, proyeksi menunjukkan nilai yang dicapai.** Perekaman selama 30 detik atau lebih pada laju otomatis mencatat laju yang stabil pada ukuran array tersebut di komputer ini; uji coba singkat pengontrol laju pada laju yang lebih cepat diabaikan. Mulai saat itu, baris `array-connect`&#x27;, analisis `connect_array()` dan panel Pengaturan Array akan mencantumkan laju tersebut jika nilainya di bawah laju model, beserta faktor yang membatasinya di `cycle_limiter`: `"clock"` (array berjalan pada laju yang telah ditetapkan, loop pengambilan menunggu clock pemicunya), `"loop"` (loop pengambilan tidak memiliki waktu tersisa), `"capture"` (proses perekaman tertinggal dan melewatkan beberapa kelompok) atau `"measured"` (tidak ada yang menyebutkan mana yang terjadi). Angka model terdapat di `wire_fps_max`. Array tersebut tetap meningkat hingga mencapai batas yang dapat ditangani oleh saluran.
- **Over-subscription (`oversubscribed: true`) berarti saluran tidak dapat menangani array tersebut pada kecepatan apa pun yang dicapai oleh kamera** — bidang fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) menunjukkan nilai 0, dan binning tidak dapat memperbaikinya (karena hal ini mengurangi jumlah byte per frame, bukan byte per detik yang diatur kecepatannya). Kabel yang lambat saja tidak menyebabkan hal ini: laju frame justru akan turun, dan jika kamera tidak dapat diatur kecepatannya cukup lambat, tingkatan menjadi `slip-emit-and-capture` (serial). Solusinya adalah mengurangi jumlah kamera, menggunakan jumbo frame, atau menggunakan NIC yang lebih cepat; `max_cams_collision_safe` melaporkan batas atas untuk koneksi kabel dan kamera Anda. Respons tersebut juga memuat `aggregate_demand_bps`, `collision_safe_ceiling_bps`, dan `per_cam_floor_bps`. Lihat [Over-Subscription](#over-subscription-the-per-cam-floor).

### Penemuan &amp; Pencatatan

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Masing-masing menerima `backend_url=` dan `timeout=30.0` yang hanya berisi kata kunci, dan mengembalikan daftar kamus. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) juga menerima `auto_start_backend=True`; panggilan `list_*` tidak pernah memulai backend.

---

## Smart-AE / Smart-Capture

Array LATTICE menjalankan AE secara terus-menerus di latar belakang segera setelah terhubung, tetapi adegan yang baru diarahkan memerlukan waktu sejenak untuk konvergensi. **Smart-capture** adalah fitur praktis yang telah dikemas: fitur ini memeriksa eksposur setiap kamera, menunggu hingga array stabil di seluruh jendela, lalu memicu proses pengambilan gambar. Fitur ini setara dengan antarmuka GUI: tombol &quot;smart&quot; capture pada aplikasi desktop memanggil titik akhir backend yang sama.

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

Saat mengoperasikan melalui `ChlorosProject` (bagian berikutnya), Anda akan mendapatkan lebih banyak opsi pengaturan:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Kebijakan smart-AE bersifat konservatif secara default. Perketat pengaturan `exposure_tolerance_pct` untuk pekerjaan radiometrik yang menuntut presisi tinggi; longgarkan untuk adegan yang berubah cepat di mana Anda hanya menginginkan hasil yang &quot;cukup mendekati&quot;.

---

## Sesi Sensor DAQ

Kumpulan backend persisten untuk sensor spektral (DAQ-U melalui USB, DAQ-M melalui BLE, DAQ-E melalui Ethernet). Mencerminkan permukaan kamera: smart-detect, penggunaan ulang kumpulan, pengikatan idempoten.

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

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` menyebutkan satu sensor, dan hanya sensor tersebut yang akan dibuka. Nama sensor tersebut dicocokkan dengan sensor yang sudah terbuka; kemudian, nama host atau alamat IP DAQ-E (DAQ-E yang hanya terdeteksi melalui IP dalam pemindaian jaringan akan diminta, dalam mode baca saja, untuk nama, MAC, dan nomor serinya, sehingga salah satu dari informasi tersebut juga dapat menemukannya), MAC BLE DAQ-M, serta ID yang dilaporkan oleh DAQ-U atau DAQ-M setelah dibuka (sensor lain yang dibuka untuk diperiksa akan ditutup kembali). Jika tidak ditemukan, `connect_daq_sensor` memicu `ChlorosConnectError` yang mencantumkan apa yang ditemukan; beserta alamatnya (`port=` / `mac=` / `eth_host=`) yang menampung sensor berbeda, hal ini juga memicu kesalahan. Sistem tidak pernah beralih ke sensor mana pun yang terlihat — itulah tujuan dari panggilan dengan bendera nol.</id></sn></id></sn>

### `connect_daq_sensor()` Tanda Tangan

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Sensor yang sudah ada di kumpulan dikembalikan apa adanya (dipinjam): `integration_time`, `frame_avg`, dan `enable_ae` tidak diterapkan padanya. Putuskan sambungannya terlebih dahulu untuk membukanya dengan pengaturan Anda. `cap_id` merupakan pengecualian: pada sensor yang dipinjam, pengaturan ini ditetapkan dengan `set_cap()`. Jika diabaikan, pengaturan default backend yang berlaku (lihat catatan di bawah tabel metode).

### Metode `DAQSensorSession`

| Metode | Deskripsi |
| --- | --- |
| `status(timeout=10.0)` | Ringkasan entri pool (status streaming/perekaman, rentang panjang gelombang, sha kalibrasi, waktu integrasi, frame_avg, status AE). Untuk DAQ-E yang alirannya terhenti: `reconnect` (di mana fitur koneksi ulang otomatis backend berlaku: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` saat sedang melakukan streaming), `stream_lost_s`, `reconnects`, dan `last_reconnect`, serta `record_gaps` untuk perekaman yang sedang berlangsung; `rx_stream_fault` menunjukkan status koneksi ulang hingga spektrum pertama tiba. |
| `latest(*, n=1, timeout=10.0)` | Mengembalikan hingga N frame spektrum terbaru (masing-masing `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Lanjutkan / jeda streaming (handle tetap terbuka). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Mulai merekam berkas .daq. Mengembalikan jalur berkas. Ditolak untuk DAQ-U/M tanpa paket kalibrasi AWS (DAQ-E dikecualikan). |
| `record_stop()` | Hentikan perekaman. Mengembalikan `{path, rows, warning, gaps}`; `warning` adalah `None` untuk perekaman yang bersih, jika tidak, menampilkan diagnosis dari backend (misalnya berkas kosong atau celah dalam aliran data), yang juga dicetak ke stderr. `gaps` mencantumkan setiap rentang di mana aliran tidak mengirimkan apa pun: `start_utc`, `end_utc` (`None`: masih hilang saat berhenti), `duration_s`, `reconnected`, `marked_in_file`. Kolom ini kosong untuk perekaman berkelanjutan. |
| `set_cap(cap_id, timeout=15.0)` | Tetapkan profil koreksi batas yang diterapkan pada pembacaan mulai dari yang berikutnya: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"`, atau `"none"` (atau `None`) untuk sensor tanpa penutup. Mengembalikan ringkasan sensorringkasan sensor, yang dibaca kembali oleh `cap_id`. DAQ-E-S mempertahankan nilai `as_recorded` apa pun yang diminta (diffuser-nya terintegrasi). Mengaktifkan `ChlorosConnectError` untuk profil yang tidak dapat dimuat. CLI: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Khusus DAQ-E: mengaktifkan kembali PTP untuk unit yang PTP-nya dalam keadaan mati (`status()["rx_ptp_enabled"]` sama dengan `False`; pengaturan disimpan di unit, sehingga tetap mati hingga diaktifkan kembali). Mengembalikan status yang telah diperbarui. Memicu `ChlorosConnectError` untuk DAQ-U / DAQ-M, atau ketika unit tidak memberikan konfirmasi. Chloros tidak pernah mengaktifkan kembali unit secara otomatis, karena status mati bisa jadi disengaja. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Hanya untuk DAQ-E: pengaturan unit itu sendiri. `firmware` (`running`, `bundled`, `update_available`), `ip` dan `ip_mode` (`dhcp` / `linklocal` / `static`: yang sedang berjalan saat ini), `network` (pengaturan yang disimpan: `mode`, ditambah `ip` / `mask` / `gateway` saat statis), `network_change_pending` (berlaku hingga unit menjalankan pengaturan yang disimpan, salah satu yang disimpan termasuk `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` adalah `None` ketikaversi unit tidak dapat dibaca. Meskipun aliran data unit tidak sampai ke komputer ini, data tersebut dibaca pada koneksi baru dan kamus menambahkan `stream_silent` (`True`), `stream_fault`, dan `reconnect`; unit yang tidak merespons akan memicu peringatan. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E saja: DHCP (`"auto"`) atau alamat statis. Memulai ulang unit untuk menerapkannya dan menghubungkannya kembali; sesi ini tetap berjalan dan `address` mengikuti unit tersebut. `reboot=False` hanya menyimpannya. Lihat [Pengaturan Perangkat DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Khusus DAQ-E: melakukan streaming saat dinyalakan tanpa perangkat lunak yang terhubung (disimpan di unit). Mengembalikan pengaturan yang telah dikonfirmasi. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Hanya DAQ-E: me-restart unit dan menghubungkannya kembali dengan pengaturan yang sebelumnya. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Hanya DAQ-E: menginstal firmware yang disertakan dalam rilis Chloros ini ke unit yang menjalankan versi lama, lalu sambungkan kembali (sekitar satu menit; jangan matikan daya). Mengembalikan `updated`, `fw_before`, `fw_after`, `bundled`, dan `warning` jika unit lambat kembali aktif atau kembali aktif pada versi yang berbeda; `updated` adalah `False` untuk unit yang sudah menggunakan versi terbaru. Menghasilkan `ChlorosConnectError` untuk unit yang versinya tidak dapat dibaca. CLI: `pool-update-firmware`. |
| `disconnect()` | Melepaskan dari pool. Tidakoperasi untuk handle yang terhubung tetapi bukan miliknya. Pada sensor yang dibuka oleh sesi ini, perintahnya adalah `force_disconnect()`, dan memicu `ChlorosConnectError` ketika pelepasan tidak dikonfirmasi (keluarnya blok `with` tidak pernah memicu kesalahan). |
| `force_disconnect(timeout=30.0)` | Melepaskan sensor terlepas dari siapa yang membukanya — misalnya sensor yang ditinggalkan oleh skrip yang crash di dalam pool, yang hanya dapat dijangkau oleh skrip baru sebagai handle terpasang yang `disconnect()`-nya tidak melakukan apa-apa. Mengakhiri sensor milik klien lain akan menghentikan perekaman apa pun yang sedang dilakukannya. Kembali setelah backend mengonfirmasi pelepasan, atau ketika tidak ada yang perlu dilepaskan (sensor tidak ada di pool, atau tidak ada backend yang berjalan); pelepasan yang habis waktunya atau ditolak oleh backend akan memicu `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atribut: `sensor_id`, `model`, `transport`, `address` (mengikuti DAQ-E yang dialamatkan ulang oleh `set_network()`), `info`, dan `backend_url`.

> **Sebutkan nama penutupnya.** DAQ-U, DAQ-M, dan DAQ-E dikirim tanpa kapasitor, dan Chloros tidak dapat mendeteksi apakah kapasitor terpasang: sensor yang dihubungkan tanpa `cap_id` akan mendapatkan nilai `sunshine_cosine`, sebuah asumsi yang menurut catatan perekamannya telah dibuat (`cap_id_source=auto_default`). DAQ-U tanpa penutup yang mengalami koreksi tersebut akan menampilkan pembacaan sekitar 30 kali lebih tinggi, jadi masukkan nilai `cap_id="none"` untuk sensor tanpa penutup (atau `set_cap("none")`), sedangkan profil penutup yang Anda pasang akan berbeda.

### DAQ-E-S — difuser tetap dan sensor kemiringan

**DAQ-E-S** adalah DAQ-E dengan dua perbedaan yang berpengaruh pada perangkat lunak. Selebihnya — pencarian perangkat, koneksi, PTP, perekaman, serta pengaturan perangkat di bawah ini — berfungsi persis seperti pada DAQ-E.**Diffuser-nya permanen, sehingga tidak memerlukan koreksi penutup.** Diffuser gaya Sunshine sudah terpasang di dalam unit dan sudah ada saat kalibrasi pabrik diukur, sehingga kalibrasi tersebut sudah menunjukkan irradiansi turun yang sebenarnya. Chloros menetapkan batasnya ke `as_recorded` (tanpa profil) apa pun yang diminta: batas yang dinamai sesuai namanya (`sunshine_cosine`, `fov_*`, `none`) akan diabaikan disertai pesan, dan rekamannya menyimpan nilai `cap_id_source=model`, sehingga peringatan batas yang diasumsikan tidak pernah muncul untuknya. Jangan juga menerapkan batas pada datanya secara manual: `sunshine_cosine` di bagian atas akan menunjukkan nilai sekitar 11 kali lipat.

**Perangkat ini dilengkapi dengan akselerometer 3-sumbu.** Respon kosinus penyebar sinar matahari hanya berlaku saat perangkat dalam posisi horizontal, sehingga sensor yang miring melaporkan irradiansi yang salah — dan reflektansi mewarisi kesalahan tersebut — tanpa gejala lain. Oleh karena itu, setiap pembacaan DAQ-E-S membawa informasi orientasinya:

| Kolom | Arti |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Sudut dalam derajat. `None` ketika firmware menandai sudut tersebut tidak terdefinisi (jatuh bebas, sumbu mati). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Akselerasi dalam mili-g. `mag_mg` yang jauh dari ~1000 berarti unit sedang bergerak dan sudut-sudut tersebut tidak bermakna untuk sampel tersebut. |
| `cal_applied` | Set: sumbu dan sudut berada dalam kerangka acuan diffuser (kalibrasi pemasangan unit telah diterapkan). Clear: kerangka acuan papan mentah, yang dapat menunjukkan kemiringan hingga ~2,3° yang sebenarnya tidak nyata. |
| `sample_age_ms` | Usia sampel akselerometer saat spektrum dikirim. |
| `temp_raw` | **Tren** suhu die yang belum dikalibrasi — hanya perbedaan, bukan pembacaan termometer. |

`DAQSensorSession.latest()` mengembalikannya di bawah kunci `imu` pada setiap bingkai (`None` pada DAQ-U, DAQ-M, atau DAQ-E biasa). `.daq` yang direkam dari DAQ-E-S menyimpan nilai yang sama di kolom `imu_*`-nya (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) di samping setiap spektrum, sehingga pembaca dapat membedakan antara awan dan tripod yang terguncang. Pada model lain, kolom-kolom tersebut kosong.

Laporan aliran sikap mencakup bidang-bidangnya sendiri dalam `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), yang dipisahkan dari kolom spektrum sehingga gangguan akselerometer tidak pernah dianggap sebagai celah dalam perekaman. `discover_daq_sensors()` dan `model` sesi tersebut menunjukkan `DAQ-E-S`.

### Pengaturan Perangkat DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Alamat statis harus ada di jaringan komputer ini.** `set_network` memicu `ChlorosConnectError` bagi perangkat yang subnet-nya tidak mencakupdi jaringan sensor: setelah restart, tidak ada yang dapat menjangkau sensor untuk mengembalikannya ke pengaturan semula. Lanjutkan ke `force=True` hanya jika komputer ini akan dialihkan alamatnya, atau diarahkan, ke subnet baru.
- **`set_network`, `reboot`, dan `update_firmware` me-restart sensor**, dan memicu `ChlorosConnectError` saat sensor sedang merekam, dan jika sensor tidak kembali dalam waktu `comeback_timeout` detik. ID sensor dan nama host `daq-e-<id>.local` tidak pernah berubah.
- **Perekaman yang dimulai tepat sebelum restart akan ditutup oleh proses tersebut, dan diberi nama.** Saat sensor kembali aktif, kamus yang dikembalikan berisi `recording_closed` (jalur berkas) dan nilai `warning` yang menyatakan hal tersebut. Jika sensor tidak kembali, atau transfer firmware gagal, nilai `ChlorosConnectError` akan menyatakan hal tersebut: pesannya diakhiri dengan peringatan, dan nilai `e.recording_closed` / `e.warning` ditetapkan (keduanya `None` jika tidak ada perekaman yang ditutup). Mulai perekaman kembali setelah sensor kembali terhubung.
- **Hanya sensor itu sendiri yang terhubung kembali.** Setelah restart, unit harus merespons di lokasi semula -- dan setelah `set_network("auto")` juga di bawah nama `.local` -- dengan nama, nama host, dan modelnya, berdasarkan adaptor jaringan (MAC) yang dimilikinya; `address` menjadi alamat IP yang merespons. Respons dengan nama tersebut hanya dihitung jika menggunakan alamat MAC yang sama, sehingga dari komputer yang mengakses sensor melalui router, atau dari jaringan NAT default WSL2, sensor yang dipindahkan oleh DHCP tidak akan terhubung kembali (`ChlorosConnectError`; sambungkan kembali ke alamat barunya).
- **Pembaruan firmware memerlukan sensor untuk terhubung kembali.** `update_firmware()` mengundang unit tersebut, dan unit tersebut membuka koneksi TCP kembali ke komputer ini untuk mengambil gambar, pada port yang dipilih oleh sistem operasi. Backend mendengarkan pada adaptor yang terhubung ke unit tersebut (pada setiap adaptor hanya jika tidak dapat menentukan yang mana) dan hanya melayani unit tersebut: koneksi lain apa pun akan ditolak (`[DAQ-FW] … refused a firmware connection from <ip>` dalam keluaran backend) selama backend masih menunggu. Firewall yang memblokir koneksi tersebut akan menyebabkan pembaruan gagal dengan pesan `device never connected back on TCP <port>`. Penginstal Windows menambahkan aturan untuk itu (lihat [Langkah 1](#windows-exe)); pada firewall lain, izinkan lalu lintas TCP masuk ke backend dari subnet sensor.
- **Di bawah WSL2, gunakan jaringan cermin** (`networkingMode=mirrored` dalam `.wslconfig`): dalam mode NAT default, setiap alamat statis ditolak (alamat WSL sendiri tidak berada di jaringan sensor), aliran data tidak sampai ke WSL, dan pembaruan firmware tidak dapat diselesaikan.
- Antarmuka pengguna (GUI) mempertahankan koneksinya sendiri ke sensor: memulai ulang melalui skrip akan mengakhiri sesi GUI dengan sensor tersebut. GUI memiliki pengaturan yang sama (tab Sensor Cahaya, pengaturan sensor, Jaringan).

### Penemuan — mencari alamat untuk terhubung

`discover_daq_sensors()` memindai USB / BLE / ETH untuk mencari sensor yang *dapat* Anda buka. Ini adalah versi DAQ dari `discover_lattice_cameras()`, dan satu-satunya cara untuk mendapatkan **MAC BLE DAQ-M** — DAQ-E memiliki nama host dan DAQ-U memiliki port COM, tetapi alamat MAC tidak tercetak pada perangkat maupun terdaftar oleh sistem operasi.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Kolom | Deskripsi |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Port COM / MAC BLE / nama host — diteruskan ke `connect_daq_sensor` sebagai `port=` / `mac=` / `eth_host=`. |
| `display` | Label yang dapat dibaca manusia. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, atau `None`. Sensor Ethernet adalah `DAQ-E-S` atau `DAQ-E` sebagaimana dilaporkan oleh sensor tersebut saat diminta, hanya baca, selama pemindaian (`DAQ-E` jika tidak merespons tepat waktu). Port serial USB menampilkan nilai `DAQ-U` jika chip jembatannya adalah salah satu yang digunakan oleh DAQ-U (FTDI, CH340, CP210x, Prolific) — petunjuk dari deskripsi port tersebut, bukan probe — dan `None` untuk perangkat USB lainnya, yang tetap terdaftar sehingga Anda dapat meneruskannya sebagai `port=`. Port serial tanpa ID vendor USB dan tanpa chip jembatan tersebut (seperti UART pada motherboard seperti `/dev/ttyS*` di Linux, tautan serial Bluetooth, atau Intel AMT) tidak dapat dianggap sebagai DAQ-U dan tidak terdaftar. |
| `extra` | Rincian per-transport (nama yang diiklankan BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Nilai kosong diabaikan. |

| Parameter | Default | Deskripsi |
| --- | --- | --- |
| `transports` | ketiga-tiganya | Urutan (atau string CSV) yang membatasi pemindaian. Sebaiknya ditentukan jika Anda tahu apa yang diinginkan — BLE adalah bagian yang lambat. |
| `scan_timeout` | 5 | Jendela pemindaian per transportasi dalam detik; backend membatasi nilainya antara 1–20. |
| `timeout` | 60,0 | Batas maksimum HTTP untuk seluruh panggilan (seperti di bagian lain dalam SDK). |
| `auto_start_backend` | `True` | Menjalankan backend lokal jika belum ada yang berjalan. Tidak pernah dijalankan untuk `backend_url` jarak jauh. |

> **Sensor yang sudah terbuka di pool tidak akan muncul.** Periferal BLE yang sudah terhubung akan berhenti mengiklankan diri, dan port COM yang terbuka tidak dapat diuji, sehingga daftar penemuan hanya menampilkan apa yang *tersedia untuk dihubungkan*. Hasil kosong segera setelah Anda menghubungkan sesuatu adalah hal yang diharapkan — gunakan `list_daq_sensors()` untuk perangkat yang sudah Anda miliki. Transportasi yang pemindaiannya tidak dapat dijalankan (tidak ada bleak / zeroconf yang terinstal) akan dilewati alih-alih memicu kesalahan, sehingga mesin tanpa Bluetooth tetap mendapatkan respons USB dan ETH-nya.

### Daftar

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### DAQ mana yang Menghubungkan ke Kamera mana — `daq_bindings()`

Reflektansi tangkapan, dan `.daq` yang disimpannya, menggunakan DAQ yang terikat pada kamera atau array tersebut, atau DAQ terbaru yang dimiliki oleh backend. Pemilihan sensor cahaya di antarmuka pengguna dan `chloros-cli daq pool-bind` menetapkannya; begitu pula:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` mengembalikan `{"cameras": {serial: b}, "arrays": {array_id: b}}`: setiap `b` menyimpan `binding` (nilai yang ditetapkan), `source` (`project`, dari `sensors.json` proyek tersebut, atau `runtime`), `known` / `known_as` (apakah nama kunci kamera atau array tersebut dikenal oleh Chloros, dan bagaimana caranya: `connected`, `project` — `cameras.json` dari proyek yang terbuka — atau, untuk kamera, `seen`: pernah dibuka di komputer ini sebelumnya; `known: False` untuk kunci yang tidak menamai apa pun) dan `resolved` (sensor pool yang sekarang dipetakan ke sana: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Pengikatan saat runtime mengesampingkan pengikatan proyekselama sesi; berkas proyek tidak diubah. Memutuskan sambungan kamera, array, atau DAQ akan menghapus semua pengikatan perangkat keras tersebut, terlepas dari siapa yang mengaturnya (anggota array terikat bersama array tersebut).

`set_daq_bindings()` hanya mengikat kamera yang terhubung, yang ada dalam proyek yang terbuka atau pernah dibuka di komputer ini sebelumnya, serta array yang terhubung atau ada dalam proyek yang terbuka (atau tombol yang sudah terikat). Tombol lain apa pun akan memicu `ChlorosConnectError` dengan menyebutkan namanya (`400`, `code`, `UNKNOWN_BINDING_TARGET`, dengan kunci `unknown`), dan **tidak ada** yang diikat dalam panggilan tersebut. `None` menghapus semua kunci.

### Reflektansi Lingkungan + Objek — `light_sensor_reflectance()`

Dua DAQ yang digabungkan, satu mengarah ke langit (lingkungan) dan satu ke target (objek), memberikan nilai reflektansi target dan indeks vegetasi berdasarkan pembacaan terbaru mereka: pasangan &quot;Ambient + Object&quot; pada antarmuka pengguna (GUI), serta `chloros-cli daq reflectance` tanpa perlu membuka sensor untuk kedua kalinya. Keduanya harus terhubung dan sedang melakukan streaming.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Pita.** Reflektansi setiap pita merupakan-terintegrasi dari objek tersebut di atas irradiance latar. `daq_band_presets(filter_code=None)` mencantumkan kumpulan pita: satu per filter LATTICE M3C (`RGN`, `OCN`, `NGB`, model pita kamera itu sendiri, sehingga NDVI DAQ dan NDVI kamera merupakan besaran yang sama) serta `GENERIC` pita lebar. `suggested`-nya adalah preset yang sesuai dengan `filter_code`, jika tidak, filter kamera pertama yang terhubung (`suggested_from` menunjukkan yang mana).
- **Mengembalikan** `bands` (per pita `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (yang terintegrasi-ins), `custom` (masing-masing dengan `value`, atau `valid: False` dan `error`), `invalid_bands`, `reflectance_type`, `preset`, serta dua pembacaan `timestamps` (`ts_ns`). Rentang yang tidak dapat diukur oleh sensor lingkungan (di luar rentang pengukurannya, pada tingkat kebisingan dasar) dianggap tidak valid, dengan alasan yang tercantum, dan setiap indeks yang membutuhkannya diabaikan, tidak pernah dihitung terhadap nol.
- **Tampak atau relatif.** Indeks yang bergantung pada skala (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) memerlukan rasio absolut, sehingga hanya dihasilkan jika kedua sensor telah dikalibrasi daya (masing-masing memiliki `calibration_bundle_sha`): `"Apparent Reflectance"`. Jika tidak, hasilnya adalah `"Relative Reflectance"`, yang hanya mencakup indeks-indeks yang tidak bergantung pada skala.
- **Dua kisi panjang gelombang yang berbeda** dibandingkan berdasarkan tumpang tindihnya: titik-titik sensor lingkungan di dalam jangkauan objek, spektrum objek diinterpolasi ke titik-titik tersebut.
- **Dua sensor, bukan satu.** Menamai satu sensor sebagai keduanya (dengan ID yang sama, dalam ejaan apa pun yang diterima oleh pool) akan memunculkan `ChlorosConnectError`: pembacaan sensor terhadap dirinya sendiri adalah 1,0 pada setiap pita, yang tidak mengukur apa pun. Backend juga menolaknya (`POST /api/daq/indices` merespons `400`, `code` `SAME_SENSOR`, padahal `ambient_sensor_id` dan `object_sensor_id` yang dikirim oleh SDK merupakan satu sensor yang digabungkan, misalnya ID dan portnya).

### Penggunaan Bersama dengan CLI, dan Skrip Lainnya

Kumpulan sensor DAQ di backend dibagikan oleh setiap klien CLI dan SDK di mesin tersebut. Jika sebuah sensor sudah ada dalam kumpulan — dibuka oleh `chloros-cli daq pool-connect` atau oleh skrip lain — memanggil `connect_daq_sensor(port="COM3")` dari Python akan mengembalikan handle yang dipinjam: panggilan `disconnect()`-nya tidak melakukan apa-apa, sehingga skrip Anda tidak akan merebut sensor tersebut dari klien yang membukanya saat keluar dari lingkup. Hal yang sama berlaku untuk `sensor_id=` (diketik dengan `:` atau `-`, keduanya cocok), untuk DAQ-E yang disebut berdasarkan IP-nya saat dibuka dengan nama host (atau sebaliknya), dan untuk deteksi cerdas dengan bendera nol ketika backend merespons dengan sensor yang sudah dimilikinya — blok `with` hanya akan melepaskan sensor yang dibuka olehnya sendiri.

GUI Chloros **bukan** bagian dari kumpulan ini: panel DAQ-nya menjalankan koneksi sendiri ke sensor, sehingga `connect_daq_sensor` tidak pernah meminjam sensor yang telah dibuka oleh GUI. Jangan gunakan sensor yang sama dari GUI dan dari skrip secara bersamaan. DAQ-U atau DAQ-M yang sedang digunakan oleh GUI biasanya gagal dibuka, tetapi DAQ-E dapat menerima sesi kedua: perintah connect dari skrip mengubah pengaturan akuisisi unit di bawah GUI, dan perintah `disconnect()` menghentikan aliran data unit, yang mengakibatkan tampilan langsung GUI dan perekaman apa pun yang sedang dilakukan terhenti. Tutup sensor di GUI terlebih dahulu.

### PTP Grandmaster — `time_sync_status()`

Backend menjalankan PTP grandmaster yang menjadi sumber cap waktu bagi kamera LATTICE dan unit DAQ-E. Backend ini dimulai sesuai permintaan: perintah `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, dan `connect_all()` akan menjalankannya jika belum berjalan — termasuk koneksi yang kemudian gagal, seperti `connect_all()` yang mencoba terhubung ke DAQ-E yang tidak ada — begitu pula dengan `test_array_sync()` dan setiap panggilan yang membuka kamera untuk satu perintah (`capture_cameras_transient()`, `start_multi_camera_stream()`, panggilan penyelarasan, `capture_live_index()`).

- **Di mana pengumuman dilakukan.** Pada adaptor jaringan kabel yang terhubung ke kamera (termasuk alamat link-local `169.254.x.x`), atau pada adaptor rute default jika adaptor tersebut terhubung melalui kabel. Tidak pernah melalui Wi-Fi atau adaptor virtual (Tailscale, WSL): tidak ada kamera atau DAQ-E yang terhubung di baliknya. Jika tidak ada adaptor semacam itu yang aktif, layanan ini berjalan tanpa mengumumkan alamat apa pun (`health` `NO_SOCKETS`) dan akan mengikat adaptor saat kamera atau DAQ berikutnya terhubung.
- **Setelah koneksi terputus.** Ketika adaptor kamera kehilangan koneksinya dan mendapatkannya kembali (kabel ditarik, sakelar diganti, adaptor USB diidentifikasi ulang), grandmaster membangun ulang soketnya sendiri, tanpa perlu restart, dan memeriksa ulang setiap DAQ-E yang terhubung, mengaktifkan kembali yang sebelumnya terjebak dalam keadaan terputus. Grandmaster juga akan membangun ulang koneksinya jika semua kamera dan DAQ-E yang dilayaninya tidak aktif selama 15 detik.
- **Apakah sinyal mencapai kabel?** Baca `time_sync_status()` (`GET /api/time-sync/status`). `state` hanya menunjukkan bahwa utas grandmaster berjalan; `health` menunjukkan apakah paketnya terkirim: `OK` (tidak ada kesalahan yang terdeteksi), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (beberapa adaptor mengalami kegagalan) atau `PEERS_SILENT` (kamera dan DAQ-E yang dilayaninya berhenti merespons), disertai kalimat di `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` hanya menghitung pesan yang terkirim; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` dan `last_delay_req_age_s` menggambarkan yang tidak berhasil, sedangkan `rebinds` / `last_rebind` menggambarkan proses rebuild. `sync_sends_slow` menghitung Syncs yang waktu pengirimannya melebihi 200 us, dan `sync_send_max_us` adalah yang terlama; waktu asal setiap Follow_Up dihitung maksimal 200 us dari waktu pengirimannya. Grandmaster berjalan dalam prosesnya sendiri (`own_process`, `process_pid`), dengan prioritas tinggi di Windows (`process_priority`), sehingga backend yang sibuk tidak dapat menunda waktu yang dicatat dan mengatur jam setiap kamera sekaligus; jika proses tersebut tidak dapat dijalankan, maka proses tersebut berjalan di dalam backend seperti sebelumnya (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Fungsi | Apa yang dilakukannya |
| --- | --- |
| `time_sync_status()` | Status grandmaster, kesehatan, identitas jam, prioritas, dan penghitung pesan (di atas). `chloros-cli time-sync status`. |
| `time_sync_peers()` | PTP slave yang dideteksinya (kamera, DAQ-E): `peers`, masing-masing `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Setiap kamera array yang terhubung: `PtpStatus`, `PtpOffsetFromMaster`, dan `PtpMeanPathDelay` (kamera yang firmware-nya tidak memiliki node PTP hanya memiliki nomor seri). `time-sync cameras`. |
| `restart_time_sync()` | Hentikan dan mulai ulang grandmaster; PTP setiap DAQ-E yang tergabung dalam pool akan diperiksa ulang setelahnya (`daq_ptp_rechecked`). Mengaktifkan `ChlorosConnectError` ketika tidak kembali. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Prioritas PTP-nya (0–255; nilai lebih rendah menang dalam pemilihan best-master), untuk jaringan dengan master PTP lain. Membutuhkan master tersebut dalam keadaan berjalan. `time-sync set-priority`. |

Masing-masing mengambil `backend_url=` dan `timeout=` serta memulai backend lokal yang belum berjalan.

---

## Utilitas Kamera

Apa yang dilakukan `chloros-cli lattice` selain menghubungkan dan merekam, sebagai fungsi biasa dalam `chloros_sdk` (modul `chloros_sdk.camera_utils`). Masing-masing memanggil rute backend yang digunakan perintah CLI, dengan aturan CLI, sehingga keduanya berperilaku serupa. Keduanya menerima **nomor seri** kamera, bukan sesi, serta kata kunci `backend_url=` / `timeout=` yang sama seperti pada panggilan smart-connect; backend yang tidak sedang berjalan akan dimulai, seperti yang dilakukan oleh `connect_camera()`. Penolakan akan memicu `ChlorosConnectError` dengan alasan dari backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Fungsi | Apa yang dilakukannya |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | Profil warna tampilan: `raw`, `linear`, `natural`, `enhanced` atau `custom_temp`, serta suhu warnanya (dibatasi antara 2000–10000 K; dibaca saat profilnya adalah `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Menampilkan saturasi dan kontras, masing-masing 0–200% (100 = tidak berubah); `reset=True` mengembalikan keduanya ke 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Menurunkan nilai piksel paling terang dan paling putih pada tampilan: -100..0 (dipotong ke arah nol, lalu dibatasi; 0 = mati). |
| `camera_white_balance_once(serial, settle_s=None)` | Penyeimbangan putih sekali jalan: kamera melakukan streaming sebentar, mencapai konvergensi, dan mengembalikan `red_ratio` / `blue_ratio` (`ratios_read` False jika tidak dapat melaporkannya). Kamera mempertahankan keseimbangan putih yang telah ditentukan; setiap pengaturan lain yang ditulis oleh open dikembalikan sebagaimana adanya. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (berkelanjutan) atau `"off"`; atau rasio keseimbangan tetap `red` / `blue`, yang akan mematikan mode otomatis terlebih dahulu. Rasio yang ditolak oleh kamera diberi nama di bawah `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | Pengaturan default kamera, dan saat menerapkan salah satunya: kamera mempertahankan pengaturan sesuai nama pengaturan default (balasannya adalah `settings`, karena kamera membacanya setelahnya, misalnya laju bingkai yang digunakannya; `None` untuk bidang yang tidak disebutkan dalam preset), dan setiap pengaturan lain yang ditulis oleh programnya akan dikembalikan seperti semula. `"default"` mengatur ulang kamera ke pengaturan default pabriknya (pengaturan pabrik dengan eksposur otomatis bebas, batas throughput tautan dan koreksi cacat dimatikan, keseimbangan putih yang dikalibrasi pada kamera RGB tetap dipertahankan), tidak disimpan ke memorinya: balasannya berisi `factory_defaults: True` dan setiap bidang dari `settings`. Dalam `camera_presets()`, `default` adalah `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Model filter, serta detail satu model (pita, bilangan kondisi respons spektral). Model filter kamera dibaca dari string modelnya; model ini tidak diatur secara manual. |
| `camera_firmware_status(serial=None)` | Firmware masing-masing kameramemiliki firmware: `current_version`, `available_version`, `update_offered`, `needs_minimum_update`, dan `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Instal firmware build Chloros ini ke satu kamera: reset pabrik, instal, reboot, pengaturan dasar Chloros ditulis ulang; kalibrasi kamera tetap terjaga. Memakan waktu beberapa menit; kamera harus tetap menyala. `force=True` menginstal ulang firmware pada kamera yang sudah terpasang. |
| `camera_power_probe(serial)` | Noda daya dan termal kamera serta suhunya. |
| `sleep_camera(serial)` / `wake_camera(serial)` | Mode siaga hemat daya kamera (kamera tetap dalam keadaan tidur hingga dibangunkan: batas throughput tautannya diaktifkan (`DeviceLinkThroughputLimitMode` `On`) pada nilai terendahnya, mode hemat daya aktif jika kamera memilikinya), beserta suhu sebelum dan sesudah; pembangkitan mengembalikan koneksi ke kondisi saat kamera masuk ke mode tidur (batas, mode, dan penundaan paket `GevSCPD`, yang dipertahankan kamera pada nilai 0 selama batas aktif; catatan ini disimpan di komputer ini, sehingga pembangkitan dari backend berikutnya juga menemukannya), atau, untuk kamera yang tidak dimasukkan ke mode tidur oleh komputer ini namun tampak dalam mode tidur (batasnya diaktifkan pada batas bawah), dengan batas dinonaktifkan dan nilai `GevSCPD` dibiarkan pada 0 pada 80 (pengaturan pabrik); batas yang ditetapkan di atas nilai minimum dibiarkan apa adanya. Hanya status daya yang disimpan; setiap pengaturan lain yang ditulis oleh open akan dikembalikan seperti semula. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Rekam bingkai hingga target kalibrasi MAPIR terlihat, dan sesuaikan koefisien per-saluran dari panelnya: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Adaptor Ethernet komputer ini untuk GigE Vision (MTU, kecepatan tautan, bingkai jumbo) dan semua `issues`; `estimate=True` menambahkan laju bingkai dan bandwidth yang akan diperoleh kamera `cameras`. |
| `fix_network_settings()` | Terapkan pengaturan adaptor GigE Vision ke adaptor kamera komputer ini. Opsi ini mengubah pengaturan adaptor dan hanya berfungsi saat Chloros dijalankan sebagai Administrator; jika tidak, program akan menampilkan log hasil penemuannya. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Koreksi vignetting pada gambar-gambar dalam sebuah folder, masing-masing menggunakan peta flat-field lensa yang sesuai: `processed`, `lens_models` (lensa yang digunakan untuk mengoreksi setiap file), `skipped` (beserta alasannya). Tanpa `lens_model`, lensa setiap file diambil dari Model EXIF-nya; `lens_model` berlaku untuk berkas yang Model-nya tidak menyebutkan nama kamera, dan berkas yang Model-nya bertentangan akan dilewati. Hanya berkas. |
| `read_dls(mac, filter=None, camera_serial=None)` | Satu pembacaan dari DLS (sensor cahaya turun Bluetooth): spektrum, irradiansi per-pita, dan magnitudo pita melalui respons kamera itu sendiri saat `camera_serial` diberikan. `discover_daq_sensors(transports=["ble"])` menemukan MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | Perangkat keras host untuk kamera dan konfigurasi yang direkomendasikan (`report_text`), serta CPU / GPU / runtime PyTorch sistem. |

- **Beberapa memerlukan kamera untuk diri mereka sendiri.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()`, dan `update_camera_firmware()` membuka kamera. Sementara Chloros sedang menggunakannya (terhubung di aplikasi, dengan `connect_camera()`, sebagai anggota array, atau sedang digunakan oleh perintah lain saat ini, seperti uji array atau penangkapan yang membuka kameranya), backend menolak dengan kode HTTP 409 `CAMERA_HELD` (baik di `code` maupun `error_code`), yang menjelaskan cara melepaskannya. Pengaturan warna, sorotan, `camera_power_probe()`, dan pembacaan berfungsi pada kamera yang terhubung.
- **Warna, sorotan, dan keseimbangan putih ditujukan untuk kamera RGB.** Pada kamera mono pita tunggal (M3M), pengatur tersebut tidak mengubah apa pun dan mengembalikan `{"skipped": True, "reason": ...}`, seperti yang dilakukan oleh CLI. Pengaturan tersebut adalah pengaturan tampilan: tampilan langsung dan tangkapan tingkat tampilan, bukan radian atau reflektansi.

---

## Array Transien, Profil Penyelarasan &amp; Indeks

Apa yang dilakukan `chloros-cli lattice` tanpa koneksi persisten, sebagaimana fungsi-fungsi dalam `chloros_sdk` (modul `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` dan `index`. Masing-masing mengirimkan permintaan yang dikirim oleh perintah CLI, dengan pengaturan default CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Mereka membuka kamera secara mandiri.** Setiap perintah membuka kameranya sendiri, menjalankan tugasnya, dan menutupnya kembali; oleh karena itu, perintah tersebut memerlukan kamera yang tidak sedang dipegang oleh Chloros (terhubung di aplikasi, dengan `connect_camera()`, atau sebagai anggota array); backend menolak dengan kode status HTTP 409 `CAMERA_HELD` dan menjelaskan cara melepaskan satu kamera. Pembukaan sementara memakan waktu (sekitar 40 detik untuk beberapa kamera). Untuk pengambilan berulang, `connect_array()` sekali adalah cara tercepat, dan ini menyimpan setiap jenis ekspor di mana pengambilan gambar sementara menyimpan gambar yang telah di-debayer. `calibrate_alignment_profile()` merupakan pengecualian: pada susunan kamera yang terhubung, ini memprosesnya dari bingkai milik susunan itu sendiri (`used_live_array`).
- **`serials`** adalah kamera-kamera yang akan dibuka, di mana yang pertama memicu pemicu; `None` membuka semua kamera di jaringan.
- **`settings`** adalah kamus pengaturan kamera yang diganti, bidang-bidang yang ditetapkan oleh bendera CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size`, dan `preset` (pengaturan kamera berdasarkan nama). Nilai eksposur atau gain yang diberikan akan menonaktifkan kontrol otomatis tersebut, sama seperti yang dilakukan oleh bendera.
- **`gpu`**: `True` memaksa penggunaan GPU, sedangkan `False` memaksa penggunaan CPU (`--gpu` / `--no-gpu`); `None` membiarkan backend yang memilih.
- **`keep_settings`**/**`clear_settings`** (setiap fungsi yang membuka kamera, default `False`): pada akhir proses, setiap kamera dikembalikan ke kondisi semula; `keep_settings=True` menyimpan pengaturan sesi ke memori kamera itu sendiri (mode bebas), `clear_settings=True` mengatur ulang memori tersebut ke pengaturan pabrik — lihat [Memori kamera](#camera-memory--keep_settings--clear_settings). Balasan (untuk suatu pekerjaan, milik `stop()`) membawa `camera_memory`.
- **Jalur adalah milik backend**: jalur relatif diubah menjadi absolut relatif terhadap direktori kerja Anda saat backend berada di mesin ini. Fungsi berkas profil penyelarasan (`alignment_profile_info()`, `reorder_alignment_profile()`) membaca berkas di komputer ini, sama seperti yang dilakukan oleh CLI.

| Fungsi | Apa yang dilakukannya |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Merekam satu kamera secara terus-menerus hingga dihentikan. Mengembalikan nilai `CliJob`. Frame yang dihasilkan oleh proses `tiff` disimpan dalam berkas `raw` sebagai array yang ditulis oleh kamera tersebut (satu bidang mosaik pada nilai sensor, tidak terkompresi); `png` dan `jpg` berisi 8 bit, sehingga rangkaian tersebut merupakan bingkai yang telah melalui proses demosaic untuk ditampilkan. DAQ yang terikat atau tergabung menulis satu berkas `.daq` per bingkai ke dalam(`daq_files` dalam statistiknya). Setiap bingkai ditulis satu kali dengan metadata lengkapnya (Model, waktu pengambilan, lensa, pita, kalibrasi pada bingkai mentah); `stop()` akan kembali setelah setiap frame yang telah diambil ditulis (hingga 25 detik), dan baru setelah itu kamera ditutup. Kamera yang dibiarkan dalam status siap untuk pemicu perangkat keras (`TriggerMode` `On`) diatur ke mode free-run selama sesi pengambilan gambar (balasan awal `trigger` menunjukkan apa yang ditemukan). Saat berhenti, kamera dikembalikan ke kondisi semula — pengaturan pemicunya dan setiap pengaturan yang diubah selama proses perekaman — kecuali `keep_settings=True`, yang menyimpan pengaturan perekaman ke dalam memori kamera itu sendiri (mode bebas), atau `clear_settings=True`, yang mereset memori tersebut. Sesi yang tidak merekam bingkai apa pun akan berakhir secara otomatis setelah 10 detik dengan `stop_reason` `no_frames` dan alasannya tercantum dalam `last_error`, dan perintah `stop()` mengembalikan alasan tersebut sebagai `error`. Frame yang terlewat karena penulisan data tidak dapat mengikuti kecepatan perekaman akan dihitung, dengan `drop_reason` menjelaskan alasannya. Untuk kamera yang diChloros, gunakan `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Kelompok frame yang disinkronkan dalam aliran pada `fps`, untuk kelompok `count` atau hingga dihentikan. Mengembalikan sebuah `CliJob` (nilai `start_reply`-nya mencakup `n_cams` dan saluran `pacing`); statistik akhirnya menghitung grup yang diambil, ditulis, dan dibuang, file yang tidak dapat ditulis oleh penulis, serta grup yang tidak menghasilkan apa pun dari kamera. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Satu pengambilan sinkron, satu gambar yang telah di-debayered per kamera: `results` (`filepath`, `serial`, `frame_id`) dan `pacing`. Sebuah grup yang kehilangan satu kamera muncul, menyebutkan berapa banyak file yang disimpan terlebih dahulu. `lattice multi-capture` tanpa array yang terhubung. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` pengambilan gambar yang disinkronkan pada eksposur tetap: `sync_groups`, `master`, satu baris `group_timestamps` per pemicu (kamera yang melewatkan satu pemicu tidak tercantum dalam barisnya) dan `sync`: offset setiap kamera lainnya dari kamera utama, pemicu demi pemicu (`offsets_us`: per kamera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), pada SATU siklus jam (`clock_domain`: `ptp`, atau `device` — PTP mati, jam uptime masing-masing kamera, sehingga offset-nya adalah saat kamera dinyalakan dan hanya jitter yang mengukur sinkronisasi). Sebuah pemicu yang cap waktunya merupakan campuran antara jam perangkat dan PTP (PTP dinonaktifkan atau masih dalam proses sinkronisasi) diabaikan dan dihitung (`groups_mixed_domain`), begitu pula pemicu di mana selisih jam PTP masih lebih dari `agree_within_us` dari master (`groups_clocks_disagree`); `notes` menunjukkan hal tersebut. `clock_offset_mean_us` / `clock_offset_jitter_us` adalah nilai rata-rata dan jitter terbesar per kamera. `success` bernilai salah, bersama dengan `error`, ketika pemicu terlewatkan, ketika kurang dari dua pemicu menyinkronkan semua kamera pada satu jam, atau ketika jam-jam tersebut tidak pernah sejalan (`sync.problems`); baris-baris tersebut tetap dikembalikan dalam kedua kasus tersebut. Ini memulai grandmaster PTP jika belum berjalan (`time_sync` menyatakan `grandmaster_running` dan `grandmaster_started_by_this_test`), mengaktifkan PTP pada setiap kamera (seperti yang dilakukan oleh `connect_array()`), dan sebelum pemicu pertama, menunggu hingga jam PTP setiap kamera terkunci pada grandmaster dan stabil: `PtpStatus` Slave pada satu grandmaster, dalam rentang 100 us darinya dan stabil selama 3 detik, serta setiap kamera terkunci seperti itu, tanpa jeda, selama 120 s — sekitar 2 menit sejak dinyalakan. Jam yang baru saja terkunci dapat memiliki selisih hingga ~200 us selama sekitar 2 menit setelah statusnya terbaca sebagai Slave, dan `PtpOffsetFromMaster` kamera itu sendiri tidak menunjukkannya; 120 detik membawa kamera-kamera tersebut ke dalam rentang ~10 us dari jam yang telah stabil. Sebuah kamera yang masih terkunci sejak pengujian sebelumnya yang membiarkan PTP aktif, sebagaimana dilihat oleh backend ini, menghitung waktu mulai dari penguncian tersebut, sehingga pengujian yang dilakukan segera setelah pengujian lain tidak perlu menunggu lagi; kamera yang dihidupkan oleh pengujian, atau kamera yang ditemukan dengan PTP aktif namun belum pernah dilihat terkunci oleh backend ini (dihidupkan di tempat lain, atau sebelum restart backend), menunggu 120 detik, dan dalam kumpulan kamera tersebut, kamera-kamera yang baru saja terkunci tetap menunggu giliran mereka. Kamera yang keluar dari mode Slave, melampaui 100 us, atau mengikuti grandmaster lain selama 3 pembacaan berturut-turut (1,5 detik) selama masa tunggu akan memulai hitungannya kembali; satu atau dua pembacaan semacam itu dicatat (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) dan hitungan berlanjut, sedangkan ID grandmaster yang tidak dapat dibaca dianggap sebagai &quot;tidak diketahui&quot;, bukan sebagai perubahan. Jika hitungan setiap kamera dimulai ulang secara bersamaan (dalam waktu satu detik), itu menandakan waktu grandmaster yang bergerak, bukan waktu kamera: log backendmencatat `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` mencantumkannya dalam `grandmaster_steps`, dan masa tunggu yang kemudian habis memulai catatannya dengan `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. Batas waktu tidak diperpanjang untuk hal ini. `ptp_settle_s` (0-600; `0` jika tidak ada penundaan penyelesaian) menetapkan waktu penyelesaian untuk proses tersebut; nilai lain akan ditolak (Kode Status HTTP 400). Waktu tunggu dibatasi pada waktu penyelesaian ditambah 90 detik (210 detik secara default, minimal 90 detik), dan panggilan menunggu selama `timeout` detik: secara default 240 ditambah waktu penyelesaian (360 detik), seperti yang dilakukan oleh `chloros-mcp` dan `test_array_sync`. `ptp_lock` memiliki `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (berapa lama setiap kamera telah terkunci dan stabil bersama-sama), `found_settled` (kamera-kamera tersebut sudah terkunci selama waktu stabilisasi saat ditemukan) dan, per kamera, `status`, `offsets_ns`, `slave_s` (berapa lama kamera tersebut telah terkunci; None: tidak terkunci) dan `remaining_s` (None jika PTP kamera tidak aktif). Log backend mencatat satu baris setiap 5 detik yang berisi status setiap kamera, berapa lama kamera tersebut telah terkunci, dan berapa lama waktu yang tersisa. Setiap kamera dinyalakan dengan PTP dalam keadaan mati; dari situ, jam-jam tersebut terkunci dalam waktu sekitar 10 detik. Setelah itu, setiap kamera dikembalikan ke kondisi seperti yang ditemukan dalam pengujian — PTP-nya, eksposur otomatis, dan penguatan (gain) yang dalam pengujian tersebut dinonaktifkan selama pengujian berlangsung (`ExposureAuto` / `GainAuto`, dan pengaturan manual `ExposureTime` / `Gain`), dan sisanya, kecuali `keep_settings=True` — sehingga kamera yang dibiarkan dengan PTP aktif (`PtpEnable=True`, atau pengujian dengan `keep_settings=True`) tetap terkunci, dan begitu backend ini melihatnya terkunci selama waktu penyetelan, pengujian selanjutnya tidak akan menunggunya. Jika jam tidak stabil, `success` bernilai salah sebelum pemicu apa pun, dan `error` menjelaskan apa yang harus dilakukan: kamera yang tidak pernah terkunci (`not_slave`) berarti grandmaster tidak dapat menjangkaunya (`time_sync_status()`); kamera yang terus terputus diberi nama berdasarkan berapa lama ia telah terkunci (`remaining_s`); jam masih dalam proses konvergensi (`far`): periksa `time_sync_status()`, dan agar kamera terkunci sebelum pengujian, aktifkan PTP dan biarkan tetap aktif (`PtpEnable=True` pada setiap kamera, atau `keep_settings=True`; pengujian akan mengembalikan pengaturan yang ditemukan, sehingga tetap aktif), lalu jalankan. `lattice multi-test`. |
| `detect_array_master(line=None)` | Kamera-kamera pada jalur sinkronisasi dan master default: `master_serial`, `slave_serials`, yang diuji oleh `line`. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Betapa beragamnya model kamera ini (nama produk, `"LATT-M3C-L87-FOCN"`) yang akan terhubung ke jaringan komputer ini: `tier`, `achievable_fps_max`, anggaran kabel (`max_cams_collision_safe`, `oversubscribed`), `cam_caps`, dan `fix_suggestions`. Secara default, model kamera yang terdeteksi; tidak ada kamera yang terbuka. Untuk kamera yang Anda miliki, beserta pengaturan yang akan digunakan saat terhubung, lihat [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Menyelesaikan profil penyelarasan dari bingkai langsung dan menyimpannya. `method`: `orb`, `akaze`, `phase`, `checkerboard` (bersama dengan `checkerboard="9x6"`, sudut-sudut bagian dalam) atau `manual` (bersama dengan `control_points={serial: [[x, y], ...]}`, kamera referensi juga). `model`: `translation`, `rigid`, `affine`, atau `homography`. Opsi yang lebih spesifik adalah bendera CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` adalah `{serial: {option: value}}`. `rms_threshold_px` menolak menyimpan solusi yang lebih buruk. Mengembalikan `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (per kamera `reprojection_error`, `num_matches`) dan `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Satu gambar multi-band yang diselaraskan melalui profil: `files` (TIFF multi-band dengan `bands` dan ukurannya, ditambah satu per pita dengan `per_band` dan satu per kamera dengan `per_camera`) serta `vignette_corrected`. `bands` memilih dan `order` mengurutkan pita berdasarkan nama (daftar atau string koma). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Mengalignasikan gambar multi-band (dengan opsi warp yang sama seperti `capture_aligned_multiband()`), untuk `count` atau hingga dihentikan. Mengembalikan `CliJob`; statistik akhirnya mencakup `frames_written` dan semua `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Berkas profil yang dirangkum sebagai `align-info` mencetaknya: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (per kamera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (per kamera `band_names`, `wavelengths_nm`, `enabled`); `profile` adalah file utuhnya. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Ubah berkas profil di tempat: `order` adalah nama-nama band (masing-masing memindahkan lapisan kameranya ke atas, sesuai urutan tersebut), `enable` / `disable` adalah nomor seri. Mengembalikan `layer_order` yang baru. `lattice align-reorder`. |
| `index_catalog()` | Preset indeks (`presets`: masing-masing `name`, `channels`, `formula`) serta warna `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Indeks atas file TIFF multi-band yang disejajarkan: sebuah `preset` (`NDVI`, `NDRE`, ...) atau sebuah `formula` atas simbol-simbol `channel_map` yang dipetakan ke pita (`{"NIR": "NIR_850", "Red": 0}`, berdasarkan nama atau indeks). Sebuah preset&#x27;simbol preset `channel_map` mengabaikan dan mengambil satu pita dari berkas yang namanya dimulai dengan simbol tersebut (`red` -&gt; `Red_660`, `nir` → `NIR_850_RGN`), yang dinamai berdasarkan file itu sendiri (daftar band-nya, atau `Chloros:Band{R,G,B}Name` dari tangkapan/produk Chloros), sehingga NDVI pada tangkapan reflektansi Chloros tidak memerlukan `channel_map`; dua band yang sesuai dengan satu simbol ditolak, bukan ditebak. `capture_level` menjelaskan isi band-band tersebut: `reflectance` atau `radiance` menjadikan nilai-nilai tersebut kuantitatif; tumpukan yang belum dikalibrasi (seperti `align-apply`) menghasilkan nilai kualitatif; `"unknown"` membacanya dari berkas, termasuk berkas Chloros&#x27;s `Chloros:ProcessingLevel`. Mengembalikan `stats` (bentuk, rentang persentil 1–99, rentang LUT, `capture_level` yang digunakan) dan `files` yang ditulis di samping input (`<input stem>_<preset>.tif`, `..._LUT.png`) atau di `output`, sebuah jalur: dengan ekstensi gambar, berkas-berkas tersebut diberi nama berdasarkan batangnya (`output="D:/out/ndvi_FRGN.png"` menulis `ndvi_FRGN.tif` dan `ndvi_FRGN_LUT.png`), tanpa ekstensi tersebut maka yang digunakan adalah awalan (`D:/out/ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`); baca jalur dari `files`. File-file tersebut memuat catatan kamera dari input (Merek / Model / nomor seri, GPS, lensa, waktu pengambilan gambar dalam UTC, identitas kalibrasi tetapi tidak pernah kalibrasi raw-DN-nya), `Chloros:ProcessingLevel=index`, serta indeks sebagai satu-satunya bandnya (`XMP-Camera:BandName`, baik TIFF maupun LUT PNG). Hanya berkas; tanpa data kamera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Pengambilan gambar, kalibrasi (`reflectance` atau `radiance`; `raw` melewatkannya), selaraskan, dan hitung indeks dalam satu langkah; `save_multiband=True` menyimpan tumpukan yang telah dikalibrasi, yang dapat diambil oleh `compute_index()` nanti. Mengembalikan `stats` (bersama dengan `serials` yang band-nya digunakan) dan `files`. Fungsi ini membuka `serials` (yang pertama memicu pemicu), atau kamera-kamera dalam profil yang entri `layer_order`-nya diaktifkan, dengan kamera referensi terlebih dahulu (dibuka meskipun pita-pitanya dinonaktifkan); kamera yang pita-pitanya dinonaktifkan tidak akan pernah dikalibrasi. `channel_map` hanya diperlukan untuk simbol yang tidak dapat dijawab oleh tumpukan, seperti pada `compute_index()`: simbol preset mengambil satu pita yang diaktifkan yang namanya dimulai dengan simbol tersebut, diperiksa sebelum kamera mana pun dibuka (jika tidak ada atau beberapa yang merespons ditolak, dengan menyebutkan nama-namanya); `formula` khusus memerlukan setiap simbol. `index.tif` dan LUT membawa informasi kamera tumpukan, waktu pengambilan gambar (waktu pemaparan kamera referensi, UTC), dan lensa, serta keduanya menamai indeks sebagai satu-satunya band mereka (`XMP-Camera:BandName`, misalnya `NDVI`: berkas PNG berwarna menyebutkan indeks, bukan kanal warnanya). `lattice index --live`. |

### `CliJob`

Handle yang dikembalikan oleh ketiga aliran tersebut. Tugas ini berjalan di bagian backend, yang mengelola kamera-kameranya hingga dihentikan.

| Anggota | Deskripsi |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (frame yang disimpan / ditulis, dibuang, `fps`, `elapsed`, kegagalan penulisan, `stop_reason`; pekerjaan berkelanjutan juga mencakup `frames_grabbed` dan `frames_queued` — setiap frame yang diambil ditulis, dimasukkan ke antrian, dibuang, atau gagal — serta `drop_reason`, `writers` dan `write_ms`, waktu rata-rata yang dibutuhkan untuk menulis satu bingkai) dan `output_dir`. `state` adalah `running`, `ended` (proses tersebut berhenti sendiri dan belum dihentikan), `stopping` (proses yang berhenti sedang menulis antriannya: statistik real-time, dan `stopped_by`) atau `finished` (statistik akhirnya, yaitu `error` dan `stopped_by` dari proses yang dihentikan). Catatan pekerjaan yang dihentikan disimpan selama 10 menit (paling banyak 32 catatan terakhir), untuk setiap klien: skrip lain atau `chloros-mcp` membaca status dan statistik akhir yang sama. Sebuah pekerjaan `chloros-cli` dimulai (`lattice continuous`, `multi-stream`, `align-stream`) dihentikan oleh backend ketika CLI berhenti melakukan check-in selama 30 detik — dihentikan paksa, mesinnya dalam mode tidur — dengan `stop_reason`, `client_gone`, dan `stopped_by` di backend; membaca statistiknya dari sini tidak membuatnya tetap aktif. Sebuah pekerjaan yang dimulai dari SDK berjalan hingga dihentikan. Setiap kali dimulai, pekerjaan tersebut mengirimkan `timeout` sebagai batas waktu backend (`client_deadline_s`): kamera yang dibuka setelahnya (satu masih dalam proses booting) dikembalikan dan ditutup, dan tidak ada tugas yang dimulai, serta panggilan tersebut memicu `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Kamera yang dipegang oleh suatu tugas akan ditolak oleh setiap pembukaan lainnya (`connect_camera()`, pembukaan baru, penangkapan sementara) dengan kode status HTTP 409 `CAMERA_HELD`, yang menyebutkan nama tugas dan siapa yang memulainya. |
| `stop(timeout=30.0)` | Hentikan tugas tersebut dan tutup kameranya: `final_stats`, `output_dir`, dan `error` jika penghentian tidak berjalan lancar. Panggilan kedua mengembalikan jawaban yang sama, begitu pula dengan perintah hentikan dari klien lain: yang dikirim saat klien pertama masih menulis antriannya akan menunggu hingga selesai (hingga 25 detik; setelah itu akan memicu kesalahan, dan statistik akhir akan muncul pada `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Tunggu hingga proses tersebut berakhir dengan sendirinya (ketika `count` tercapai, drive penuh, atau terjadi kesalahan), lalu jalankan `stop()` untuk mengumpulkan jumlah akhir. `timeout` memicu `TimeoutError` saat masih berjalan; sebuah pekerjaan tanpa `count` hanya berakhir ketika dihentikan. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | ID dan jenisnya (`continuous`, `multi-stream`, `align-stream`), lokasi penulisan, balasan rute awal, dan apakah `stop()` telah dijalankan. |

Gunakan sebagai pengelola konteks untuk menghentikannya saat keluar dari lingkup.

---

## Folder Proyek &amp; Resep

**Folder proyek** adalah folder tempat proyek baru dibuat, yang digunakan bersama oleh aplikasi Chloros, CLI, dan SDK (modul `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Folder yang telah dikonfigurasi namun tidak lagi ada akan kembali ke default (`Documents/MAPIR/Chloros Projects`), sama seperti yang dilakukan aplikasi. Jalur relatif yang diberikan ke `set_project_folder()` diambil dari direktori kerja Anda saat backend berada di mesin ini.

**Resep** adalah daftar tindakan YAML/JSON yang dijalankan oleh `chloros-cli project run PATH RECIPE` terhadap proyek yang telah disimpan. `run_project_recipe()` menjalankannya menggunakan runner bawaan CLI(yang diinstal bersama SDK), sehingga satu resep akan berperilaku sama baik dari:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** adalah berkas `.yaml` / `.yml` / `.json` atau resep itu sendiri: sebuah kamus (dict) yang berisi `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`), serta secara opsional `overrides` dan `stop_on_error` (default true: proses berhenti pada tindakan pertama yang gagal). Tindakan-tindakan tersebut merujuk pada [Recipe DSL](cli-reference.md#recipe-dsl) dalam dokumentasi CLI; nilai relatif `output` disesuaikan dengan direktori kerja skrip Anda.
- **Mengembalikan** `ok` (semua tindakan berhasil), `actions` (jumlahnya), `log` (keluaran runner; `echo=True` juga mencetaknya saat berjalan) dan `validation`. Berkas resep yang hilang atau tidak dapat diparsing akan memicu `ChlorosConnectError`.
- **Diverifikasi sebelum dijalankan.**`validation` adalah `{"ok", "errors", "warnings", "steps"}`: setiap tindakan harus memiliki tepat satu kata kerja yang dikenal sebagai kuncinya (`{"capture": {"name": ...}}`, bukan `{"action": "capture"}` atau `{"type": "capture"}`) yang menamai kamera, array, atau sensor yang dimiliki proyek, dengan jenis yang tepat. Resep yang mengandung kesalahan**tidak** menjalankan tindakan apa pun (`ok` false, kesalahan dalam `log`), apa pun yang dinyatakan oleh `stop_on_error`. Kunci yang tidak digunakan, serta setiap array atau sensor cahaya yang dikendalikan resep tersebut namun saat ini tidak terhubung, akan menjadi `warnings`. Sebuah resep tidak menghubungkan perangkat: hubungkan perangkat pertama proyek (`ChlorosProject.connect_all()`, `chloros-cli project connect`, atau alat MCP `connect_project_devices`). `dry_run=True` tidak menjalankan apa pun dan mengembalikan `{"recipe": ..., "validation": ...}`.
- Hanya satu resep yang dapat dijalankan sekaligus dalam suatu proses. Resep YAML memerlukan PyYAML.

---

## Otomatisasi Proyek — `ChlorosProject`

Proyek Chloros yang disimpan adalah sebuah folder yang berisi `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` memuat manifes (`relink`: folder tempat berkas sumber yang hilang dari proyek yang dipindahkan kini berada, digunakan saat backend memuatnya -- `process()` -- lihat **Proyek yang dipindahkan** di bawah `ChlorosLocal`), dan `connect_all` mengaktifkan setiap kamera, array, dan sensor cahaya yang telah disimpan beserta pengaturannya — keadaan perangkat keras yang sama seperti yang akan dihasilkan oleh GUI. Sensor cahaya terhubung melalui [kumpulan sensor DAQ](#daq-sensor-sessions), sehingga memerlukan backend yang sedang berjalan (lihat [`SensorHandle`](#sensorhandle)).

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

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
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
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Menemukan + menghubungkan setiap kamera, array, dan sensor cahaya yang tersimpan. Mengembalikan laporan koneksi per kelas (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, masing-masing dengan `connected`, `missing`, dan `errors`; `arrays` juga mencakup `alignment`). Dengan backend yang mendengarkan pada `127.0.0.1:5000`, kamera dan array yang tersimpan ditemukan dan dihubungkan melalui backend tersebut. Jika tidak ada, proses ini akan menemukan dan mengendalikannya sendiri melalui `lattice_sdk`, yang memerlukan runtime Arena SDK dan OpenCV (`cv2`) — paket SDK wheel tidak menginstal keduanya; jika tidak dapat menginstalnya, setiap kamera dan array yang tersimpan akan terdaftar di bawah `errors` beserta alasannya, dan `missing` tetap kosong. Backend yang sedang berjalan tetapi menolak SDK (keluar: HTTP 401; rencana di bawah Copper: 403) bukanlah &quot;tidak ada backend&quot;: setiap kamera, array, dan sensor cahaya yang disimpan tercantum di bawah `errors` beserta alasannya dari backend, dan tidak ada yang dibuka dalam proses ini. Setiap panggilan memeriksa backend lagi, sehingga `connect_all()` setelah masuk, atau setelah memulai Chloros, terhubung ke objek proyek yang sama. Sebuah array yang telah disimpan dan sudah aktif di backend dengan kamera yang sama (dari GUI, atau `chloros-cli project connect`) akan dihubungkan apa adanya, tanpa menerapkan ulang pengaturan yang disimpan, dan `disconnect_all()` membiarkannya tetap berjalan. Sensor cahaya selalu melewati backend (lihat [`SensorHandle`](#sensorhandle)). Ia tidak pernah membuat backend baru. |
| `disconnect_all()` | Hentikan semuanya. Setiap perangkat dilepaskan terlebih dahulu; kemudian, jika backend tidak mengonfirmasi pelepasan kamera, array, atau sensor cahaya (proses penghentian habis waktunya, atau ditolak), masing-masing diberi nama oleh satu `ChlorosConnectError`. Perangkat yang sudah tidak ada, atau tidak ada backend yang berjalan, tidak akan memberikan respons. Array atau sensor cahaya yang telah dibuka oleh klien lain tetap berjalan. Proses keluar proyek `with` tidak pernah memicu kesalahan. |
| `capture_all(output_dir=".")` | Satu bingkai dari setiap kamera + array yang terhubung, dan satu spektrum (`SensorHandle.read()`) dari setiap sensor cahaya yang terhubung, diambil secara bergantian (kamera, lalu array, lalu sensor), bukan pada satu waktu. |
| `stream(camera, overlays=False, fps=10.0)` | Generator yang menghasilkan frame BGR `numpy` dari kamera (atau array) yang disebutkan namanya. `overlays=False` membaca frame dengan handle&#x27;`grab()` pada tingkat `debayered`: melalui backend jika proyek terhubung melalui backend, jika tidak, menggunakan loop pengambilan langsung `lattice_sdk` (array menghasilkan kamus `{serial: frame}`). `overlays=True` meneruskan melalui `ChlorosLocal.camera_stream()` → umpan MJPEG backend `/api/camera/<serial>/stream-annotated`, dengan blok yang disimpan oleh kamera (`ui.overlay`) diteruskan sebagai parameter kueri. Membutuhkan mode backend, OpenCV (`cv2`), dan **kamera mandiri**: kamera mode langsung memicu `RuntimeError` (backend tidak dapat mengambil kamera yang dimiliki oleh proses ini) dan sebuah array memicu `NotImplementedError` (menumpuk komposit per kamera — mengalirkan anggota berdasarkan nama). Setara satu kali eksekusi: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Jalankan penyelarasan pada setiap array yang saat ini terhubung. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Jalankan pipa kalibrasi/indeks pada gambar-gambar proyek (mencakup `ChlorosLocal.process`; keempat ini adalah **satu-satunya** argumen kata kunci yang diterima — `indices=` dst. menimbulkan `TypeError`; atur indeks melalui `ChlorosLocal.configure()`). Membangun `ChlorosLocal()` secara malas, yang secara otomatis memulai backend. |

Atribut:
- `proj.cameras` — `Dict[str, CameraHandle]` yang diindeks berdasarkan nama DAN serial.
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
| `debayered` | Warna 3-saluran (**default** untuk `CameraHandle` dan setiap `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()`, dan `stream()` secara default menggunakan `raw`): BGR di memori (`grab()` / `frame_stream()`), disimpan sebagai RGB oleh `capture()` | Dalam mode backend, sebagai tangkapan: offset gelap (DSNU) dari paket kalibrasi pada data mentah, kemudian demosaik yang peka tepi dari OpenCV — tanpa LUT linearitas, flat field, atau unmix. Satu-satunya tingkat yang berfungsi tanpa mode backend adalah demosaik bilinear biasa dari pustaka kamera (tanpa offset gelap). File TIFF yang disimpan berformat RGB fotometrik, jadi balikkan saluran frame dalam memori (`frame[..., ::-1]`) untuk membandingkannya dengan file. |
| `radiance` | float32, W/m²/sr/nm, **satu bidang per pita** | Rantai radiometrik lengkap: demosaik + pemisahan 3×3 (multispektral) + DSNU + flat-field + skala NIST, dengan eksposur × gain dibagi sehingga nilainya menjadi absolut. |
| `reflectance` | uint16, 32768 = 1,0 (atau uint8, 255 = 1,0 — lihat `bit_depth`), **satu bidang per pita** | Radian dibagi dengan irradiansi yang turun (ρ = π·L/E). Membutuhkan pembacaan DLS/DAQ — lihat catatan di bawah. |
| `display` | 8-bit mirip sRGB | Render setara GUI: CCM + white balance + gamma melalui profil warna aktif kamera. |

Hanya dalam mode langsung (tanpa backend), `debayered` yang tersedia: `grab()` pada tingkat lain mengaktifkan
`NotImplementedError`, `capture()` mengabaikan `processing`, dan `frame_stream()` meningkatkan
`RuntimeError` di setiap tingkat. `reflectance` memerlukan pembacaan downwelling yang dapat digunakan — titik akhir bingkai menarik
DAQ yang digabungkan ke slot DLS kamera secara otomatis, tetapi tanpa DAQ yang terikat, rantai menolak
keluaran reflektansi dan secara jujur menandai penurunan kelas dalam metadata yang dikembalikan daripada diam-diam
mengembalikan produk yang lebih rendah.

> **Skala DN reflektansi — jangan mengkodekannya secara permanen.** Reflektansi LATTICE menggunakan `32768` = ρ 1,0 dan menandai
> XMP `Chloros:PixelScale=32768`; reflektansi Survey3 menggunakan `65535` = ρ 1,0 dan tidak memiliki
> tag `Chloros:*`. Baca tag tersebut dan bagi dengan nilainya. Nilai ini didefinisikan dalam domain uint16, sehingga tetap
> `32768` untuk setiap format yang melakukan penskalaan ulang (TIFF 16-bit, TIFF 8-bit, PNG/JPG 8-bit, 32-bit persentase)
> — normalisasikan tipe data yang disimpan kembali ke uint16 terlebih dahulu (×257 dari 8-bit, ×65535 dari float). Satu-satunya
> pengecualian adalah berkas dari build sebelum versi 1.3.0: berkas TIFF 8-bit pada saat itu *dipotong*, bukan diskalakan ulang, sehingga
> tidak ada skala yang mendeskripsikannya, dan file tersebut tidak mengandung tuple `PixelScale` atau MicaSense. Perlakukan tag yang hilang pada
> file reflektansi LATTICE sebagai &quot;tidak ada skala yang valid&quot;, bukan sebagai nilai default.

**Tag kalibrasi: hanya pada `raw`.** File `raw` dari suatu pengambilan data berisi paket kalibrasi kamera: muatan `CalibrationJSON` dan tag yang kompatibel dengan MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), yang menggambarkan nilai mentah sensor. Setiap berkas pada tingkat lainnya (`debayered`, `display`, `radiance`, `reflectance`, serta render `index/`) dan setiap bingkai perekaman berkelanjutan / aliran multi-kamera menyertakan identitas kamera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) serta catatan khusus bingkai tersebut, yaitu `XMP-mapir:FrameJSON` (pencahayaan, penguatan, suhu, format piksel, waktu pengambilan, kecocokan DAQ), karena setiap produk `process()` memiliki identitas tersebut. Pemrosesan menemukan kalibrasi berkas tersebut berdasarkan nomor serinya dan `CalBundleSha256` di cache kalibrasi Chloros, mengunduh paket tersebut sekali jika komputer ini tidak memilikinya.

#### File yang dibuat dari beberapa kamera

File yang dibuat dari pita spektral lebih dari satu kamera — tumpukan multiband (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), indeks langsung yang dibuat dari beberapa kamera (`index.tif`, `index_LUT.png`), gambar `composite/` dari pengambilan array — mencantumkan nama setiap kamera yang digunakan:

- **Per-daftar per kamera, berdasarkan urutan band** (komposit: lapisan latar belakang, lalu latar depan; lapisan indeks, kamera asal pita-pitanya): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (paket kalibrasi yang dilalui oleh setiap pita kamera, jika tidak — tumpukan `capture_aligned_multiband()` / `start_aligned_stream()`, yang pita-pitanya tidak melalui paket apa pun — paket kamera yang telah disimpan dalam cache oleh Chloros, yang namanya sama dengan nama file tangkapannya), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — daftar paralel, di mana item *n* dari masing-masing menggambarkan kamera yang sama; `-` di mana kamera tidak memiliki nilai (tidak ada paket kalibrasi, lensa tidak dikenal). Setiap kamera adalah **satu** item: jika paketnya memiliki beberapa nilai untuk suatu bidang — sertifikat lampu halogen dari paket tersebut — digabungkan dengan `;` (`F-1479-1;F-1480-1;F-1481-1`), tidak pernah menggunakan koma, sehingga daftar tersebut terbaca sebagai satu item per kamera terlepas dari cara pencetakannya.
- **EXIF `SerialNumber`** menyimpan nomor seri yang sama, yang disatukan dengan koma (`214000535,214000551`).
- Data ini **tidak** memuat identitas kamera tunggal (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): identitas tersebut merujuk pada satu kamera dan paketnya, dan proses pengolahan akan menentukan kalibrasi berkas berdasarkan data tersebut. Indeks yang dihitung dari rentang spektral beberapa kamera (`index.tif`, `index_LUT.png`) memiliki EXIF `Model` `LATTICE` (rig), bukan model `LATT-…` dari satu kamera.

Sebuah berkas yang berisi band dari **satu** kamera — berkas TIFF per band, bingkai `capture_aligned_multiband(per_camera=True)`, tumpukan, atau indeks langsung dari satu kamera — membawa tag identitas kamera tersebut sendiri, seperti halnya setiap berkas tangkapan lainnya. Tumpukan berkas `capture_aligned_multiband()`, berkas per-band, dan berkas per-kamera, serta setiap tumpukan berkas `start_aligned_stream()` dan berkas per-band-nya, membawa waktu pengambilan gambar kelompoknya (waktu pemaparan kamera referensi, UTC), dan bingkai `per_camera` dari kamera mono merupakan satu bidang, pada `bit_depth` seperti halnya tumpukan.**Orientasi kamera (balik / putar).** Frame dari `grab()` / `frame_stream()` selalu dalam
orientasi asli kamera, di setiap tingkatan. Hasil pengambilan gambar yang disimpan mengikuti pengaturan orientasi kamera
(Mirror H / Mirror V / rotasi pada GUI):

- **File `raw` dan `debayered` persis seperti yang dibaca oleh sensor**, apa pun pengaturannya — mosaik Bayer
  yang diputar atau dibalik akan menjadi pola filter warna yang berbeda, dan peta kalibrasi
  berdasarkan kisi sensor itu sendiri.
- **Berkas `display`, `radiance`, `reflectance`, dan berkas indeks telah diorientasikan** — diputar satu kali, setelah semua
  perhitungan kalibrasi. Masker penjepitnya berada pada kisi yang sama.
- **Setiap berkas tangkapan merekam pengaturannya** dalam XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, searah jarum jam, setelah
  pembalikan) dan `ChlorosOrientationApplied` — `false` pada data mentah dan yang telah di-debayering, serta `true` pada
  hasil orientasi. `ChlorosProcessing` mencantumkan operasi yang diterapkan pada piksel berkas tersebut (`none` jika
  dalam format asli). Hal itu mencakup setiap bingkai dari perekaman berkelanjutan (pengaturan kamera itu sendiri,
  belum diterapkan) dan dari aliran multi-kamera (identitas: bingkai-bingkai anggotanya bersifat native, seperti
  pada perekaman array).
- **Pemrosesan proyek menerapkan orientasi yang terekam** ke produk-produk yang dihasilkannya dari data mentah
  (kalibrasi dalam koordinat sensor terlebih dahulu, kemudian distorsi penyelarasan array, dan terakhir orientasi),
  sehingga hasilnya sesuai dengan produk perekaman langsung — termasuk produk cadangan yang belum dikalibrasi berdasarkan respons sensor / yang telah dikoreksi vignette;
  produk yang telah di-debayered tetap dalam format asli.
- **Penampil gambar Chloros menampilkan file mentah atau yang telah di-debayered yang diputar sesuai rekamannya** — hanya untuk tampilan,
  file tersebut tidak diubah; file tanpa rekaman ditampilkan sebagaimana disimpan.
- **Perekaman tanpa rekaman diproses sebagai file asli.** Sebelum versi 1.3.0, pengambilan gambar dengan satu kamera
  mengonversi file mentah dan file yang telah di-debayered-nya sendiri; Chloros tidak dapat membedakan file mentah semacam itu dari file asli dan
  tidak menebaknya.
- **Pengambilan gambar array mempertahankan setiap file anggotanya dalam format asli secara default** dan mencatat identitasnya;
  `ArraySession.capture(orient=True)` mengonversi hasilnya (lihat kontrol pengambilan gambar array).

#### Keluaran per-band dan array campuran

Pada tingkat `radiance` dan `reflectance`, sebuah kamera mengembalikan **satu bidang per band dari
kamera tersebut** — tiga untuk M3C (tiga pita tak tercampurnya, urutan saluran BGR), satu untuk M3M (
satu filter pita sempitnya). Bidang yang direplikasi dari kamera mono digabungkan, sehingga **array campuran
M3C/M3M menghasilkan bentuk susunan yang berbeda per serial**. Baca kunci `bands` dari
metadata bingkai untuk mengetahui apa saja setiap bidang tersebut; jangan menyimpulkannya dari jumlah saluran:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Selalu periksa `hdr["processing_level"] == "reflectance"` sebelum menggunakan nilai-nilai tersebut. Sebuah penurunan status
(tidak ada pembacaan DAQ, tidak ada paket kalibrasi, tata letak pita tidak diketahui) dilaporkan dalam `processing_level` dan
`radiometric_refused`, bukan peningkatan status — susunan yang Anda terima tetap merupakan bingkai yang valid, hanya saja
bukan hasil yang Anda minta. Rekaman yang disimpan yang telah diturunkan statusnya memiliki kode yang sama dalam XMP-nya
seperti `Chloros:RadiometricRefused` (misalnya `no-band-layout`, `no-dls-snapshot`).

#### Satu pemicu per pembacaan — `group_id`

**Anggota array disajikan dari grup bingkai yang disinkronkan array tersebut, bukan dari
buffer tangkapan masing-masing kamera.** Kamera-kamera dalam array yang disinkronkan secara perangkat keras memaparkan gambar secara bersamaan, tetapi N pembacaan per kamera
tidak *mengambil sampel* secara bersamaan — membaca bingkai terbaru yang tersimpan dalam buffer masing-masing kamera melintasi batas pemicu
, dan satu bingkai yang terlewat akan membuat set tersebut tidak sinkron tanpa ada yang menyadarinya.

Penyajian dari grup memang diperlukan tetapi tidak cukup jika dilakukan sendiri: setiap permintaan masih
mengambil *kelompok terbaru* pada saat itu juga, dan loop pengambilan dapat menerbitkan di antaranya.
Berdasarkan pengukuran pada susunan 4 kamera langsung, **12 dari 20 pembacaan paralel melintasi dua kelompok** — selalu
berdekatan secara tepat. Jadi, pembacaan tersebut **dijepit**:

1. Setiap frame membawa `group_id`, penghitung publikasi monotonik dari susunan kamera tersebut.
2. `grab()` membaca master (atau, jika master tidak dapat dilayani, kamera berikutnya yang bisa),
   lalu mengunci kamera-kamera lainnya ke ID grup yang dituju (`?group_id=N` pada titik akhir HTTP
  ).
3. Backend menyimpan grup terbaru selama **setidaknya setengah detik**, sehingga kamera sejenis yang sudah
   tergeser oleh publikasi selanjutnya tetap dilayani oleh pemicu tersebut — cukup lama untuk
   ekspor reflektansi per-kamera, bahkan pada susunan sensor penuh dengan 9 kamera. Penetapan yang tidak lagi
   disimpan oleh backend akan ditolak dengan kode `409` dan `group_aged_out: true`, bukan dijawab secara diam-diam
   dengan pemicu yang berbeda — `grab()` membaca ulang set tersebut.

Diukur ulang dengan penandaan: **20 dari 20 pembacaan mengembalikan satu pemicu.** `grab()` melakukan ini untuk Anda;
Anda hanya memerlukan `group_id=` saat mengakses titik akhir HTTP secara langsung.

- Kamera yang tidak dapat dilayani di grup yang di-pin — tidak ada di dalamnya, atau menolak permintaan
  (sebuah `band` yang tidak dimilikinya, level yang tidak dapat dihasilkannya) — **dilewati alih-alih diisi ulang**
  dari pemicu yang berbeda, dan kamera lainnya tetap dikembalikan.
- Dict yang dikembalikan oleh `grab()` adalah `GrabResult` (subkelas `dict` biasa;
  `from chloros_sdk import GrabResult`): `.errors` memetakan setiap
  nomor seri yang diabaikan ke pengecualian yang ditimbulkan saat pembacaan, dan `.group_id` adalah grup bersama. Sebuah
  `.errors` kosong berarti setiap kamera termasuk dalam grup ini.
- Kamera mandiri tidak memiliki grup, melaporkan `group_id: None`, dan **menolak** pin `group_id=`
  daripada mengabaikannya.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

Penghitung yang sama dipublikasikan pada **setiap aliran MJPEG per kamera** dari anggota array —
pratinjau (`/api/camera/stream/<serial>`), pratinjau beranotasi (`/stream-annotated`), dan
aliran reflektansi (`/stream-reflectance`) — sebagai `X-Chloros-Group-Id` pada setiap bagian, selain
`X-Chloros-Frame-Id` dan `X-Chloros-Timestamp-Ns`. Aliran bersifat berkelanjutan, bukan
pada titik waktu tertentu, sehingga pencocokannya dilakukan dengan mengumpulkan bagian-bagian dan mengelompokkannya berdasarkan header tersebut — tidak ada
pin yang perlu diatur. Kamera mandiri tidak memiliki grup. Lihat
[Aliran MJPEG per kamera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` tidak pernah diam.** Sebuah tick yang gagal karena alasan yang dapat teratasi dengan sendirinya
akan dilewati — aliran kamera tidak menghasilkan apa pun untuknya, sedangkan aliran array menghasilkan kamera-kamera yang
menjawab (mungkin tidak ada). Alasan-alasan tersebut adalah: belum ada frame; utas pengambilan kamera dijeda selama
satu atau dua detik (`404 not streaming`, yang dijawab oleh backend setiap kali kamera lain terhubung
atau terputus, atau perubahan pengaturan menyeimbangkan kembali kumpulan); backend dimulai ulang atau `5xx`; penyelarasan
masih dalam proses (`409 … retry shortly`); dan penolakan reflektansi karena DAQ belum memiliki pembacaan
(`no-dls-snapshot`, pembacaan yang kedaluwarsa atau kosong). Penolakan pada tick berikutnya hanya akan terulang — sebuah
`band` yang tidak dikenal, `bit_depth=8` pada radiance, paket kalibrasi tanpa skala radiance, penolakan reflektansi
dengan **tidak ada DAQ sama sekali** (tidak ada yang terikat ke kamera atau array-nya, tidak ada yang terhubung ke
backend: `dls_source` dari penolakan tersebut adalah `"none"`), `4xx` lainnya —
menimbulkan `chloros_sdk.FrameFetchError` (sebuah `RuntimeError`, yang membawa `.status` dan `.body`) secara
bersamaan. Demikian pula **60 detik** tik yang terlewat tanpa frame di antaranya, apa pun `fps` — cukup
lama untuk melewati penyelesaian penyelarasan dan restart backend, cukup singkat sehingga kamera yang
benar-benar mati dapat menghentikan aliran. Kesalahan &quot;give-up&quot; tersebut juga merupakan `FrameFetchError`, baik untuk kamera maupun
array, dengan pengecualian dari pembacaan terakhir itu sendiri pada `.errors` (dan, untuk kamera, sebagai `__cause__`). Untuk
sebuah array, &quot;ditolak&quot; berarti setiap kamera ditolak; pengecualian `.errors` memuat
alasan masing-masing kamera.

**Sebuah array `frame_stream()` menghasilkan setiap pemicu sekali** (baik `ArrayHandle` maupun `ArraySession`). Sebuah
tick yang set terbarunya adalah pemicu yang telah dihasilkannya akan menunggu pemicu berikutnya alih-alih
menghasilkan set yang sama lagi, sehingga aliran berjalan pada kecepatan yang lebih lambat di antara `fps` dan laju bingkai array itu sendiri
, dan set yang berurutan tidak pernah berbagi `group_id`. Sebuah array yang berhenti mempublikasikan akan memicu
`FrameFetchError` (“tidak ada pemicu baru selama 60 detik”) alih-alih mengulangi set terakhirnya. Tanpa ini, sebuah
aliran yang meminta lebih banyak daripada yang dipublikasikan oleh array — atau dimulai segera setelah terhubung — akan menghasilkan
pengulangan: dalam satu pengujian yang diukur, 20 set berisi 11 ID grup yang berbeda.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Mengekspor satu pita — `band`

`grab()` / `frame_stream()` ambil `band="<name>"` untuk mengekspor hanya satu pita dari kamera sebagai
array saluran tunggal `(H, W, 1)` — bentuk yang sama seperti yang dihasilkan kamera mono, sehingga keluaran satu band
ditangani dengan cara yang sama baik berasal dari M3M maupun hasil pemisahan M3C. Band merah FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

Ejaan yang diterima adalah nama tata letak lengkap (`Red_660`, `NIR_850`, `Green_550`) atau singkatan
kepalanya (`red`, `nir`, `green`), tidak membedakan huruf besar-kecil. Nama yang tidak dikenal akan ditolak dengan
daftar nama yang tersedia.

> **Pemilihan dilakukan berdasarkan nama, bukan nomor saluran — secara sengaja.** &quot;Saluran 1&quot; adalah `Green_550` dalam
> urutan saluran BGR array, tetapi *merah* dalam urutan huruf kode filterurutan huruf (`RGN`), dan keduanya
> tidak sesuai. Memilih yang salah akan menghasilkan bingkai yang tampaknya masuk akal namun berasal dari pita yang salah, yang
> tidak dapat dideteksi oleh apa pun di hilir. Oleh karena itu, tidak ada indeks numerik.

#### Menyelaraskan aliran — `align`

`align=True` menyesuaikan bingkai ke dalam bingkai referensi umum array-nya. Setiap kamera disesuaikan
melalui resolusi profil dan ROI pemotongan yang **sama**, sehingga aliran per-kamera yang dibuka secara terpisah
akan **terdaftar secara bersamaan** — Anda tidak perlu memintanya secara bersamaan:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Persyaratan dan perilaku:

- Kamera harus berada dalam susunan yang **disinkronkan secara perangkat keras** (kabel sinkronisasi M8) yang memiliki profil penyelarasan
  — penyelarasan bersama mengasumsikan anggota-anggotanya terpapar secara bersamaan, yang tidak disediakan oleh susunan
  array free-run tidak menyediakan. Jika tidak, akan ditolak dengan kode `409`.
- **Ditolak, tidak pernah secara diam-diam tidak selaras.** Jalur penangkapan mungkin menyimpan data yang tidak selaras dengan peringatan
  karena file mempertahankan tag asal-usulnya dan dapat didaftarkan ulang nanti; aliran langsung tidak memiliki
  kesempatan kedua seperti itu, sehingga profil yang hilang merupakan kesalahan, bukan solusi cadangan.
- **Meminta penyelarasan akan memulainya.** Meminta `align` pada array tanpa profil akan memicu
  kalibrasi dan mengembalikan `409` dengan apa yang sedang ditunggu (misalnya *&quot;menunggu eksposur
  menetap pada 214000551 (1/3)&quot;*) — coba lagi dan prosesnya akan berhasil, biasanya dalam hitungan puluhan detik.
  Hal ini penting dalam mode headless: penyelarasan otomatis saat koneksi akan berhenti setelah 30 detik, dan setiap upaya
  dalam rentang waktu tersebut akan dilewati selama smart-AE masih dalam proses stabilisasi, sehingga array yang eksposurnya
  konvergensi lambat biasanya berakhir tanpa profil untuk seluruh sesi dan tidak ada cara untuk memperoleh
  profil tersebut tanpa membuka tampilan gabungan di desktop. `alignment_status` juga melaporkan
  `ae_align_holdouts`, menyebutkan kamera-kamera yang masih dalam proses penyesuaian serta kemajuannya.
- Berikan setiap aliran dari satu susunan **`max_h` yang sama**, atau potongan-potongan gambar tersebut akan berada pada kisi piksel
  yang berbeda.
- Penyelarasan diterapkan *sebelum* pengkodean kedalaman bit, sehingga warp menginterpolasi nilai-nilai presisi penuh
  .

#### `bit_depth` — menukar presisi dengan bandwidth

`grab()` dan `frame_stream()` menerima `bit_depth=16` (default) atau `bit_depth=8`:

| `bit_depth` | Pengkodean reflektansi | Biaya |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, ruang kosong hingga ρ 2,0 | Presisi penuh. |
| `8` | uint8 mencakup `0..refl_max` (default ρ 1,0) | Setengah byte; ~0,004 kuantisasi ρ pada batas atas default. Nilai di atas batas atas akan dipotong — lihat di bawah. |

##### Batas atas 8-bit — `refl_max`

`refl_max` menetapkan nilai ρ yang dicakup oleh rentang 8-bit, sehingga `pixel_scale = 255 / refl_max` dan ρ tetap
`pixel / pixel_scale`. `grab()` dan `frame_stream()` menggunakannya sebagai kata kunci pada
penanganan array (`?refl_max=` pada titik akhir HTTP). Naikkan nilainya untuk mendapatkan ruang gerak bagi sorotan spekular
, dengan langkah-langkah yang secara proporsional lebih kasar:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | langkah |
| --- | --- | --- |
| `1.0` (default) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **Fitur eksposur otomatis tidak dapat memperbaiki kliping 8-bit, dan mencoba menggunakannya justru akan memperburuk data.**
> ρ = π·L/E dengan eksposur dan gain diabaikan, sehingga ρ merupakan sifat dari pemandangan, bukan dari
> pengaturan kamera — diukur pada array langsung, mengurangi eksposur setengahnya mengubah nilai rata-rata ρ sebesar 1,3%. Meningkatkan
> eksposur agar &quot;sesuai&quot; dengan rentang justru akan membuat piksel mencapai saturasi **sensor**, yang
> meremehkan nilai L dan menyebabkan bias ρ **rendah** tanpa menampilkan kesalahan (diukur: ρ maksimum 1,203 → 0,985 pada
> eksposur 2×, rata-rata −23%). Pilih rentang dengan `refl_max`; menjaga sensor agar tidak
> mengalami saturasi adalah tujuan AE, dan hal itu melindungi *validitas* ρ, bukan rentangnya.

**Baca `reflectance_clipped_fraction`, bukan bendera.** `reflectance_clipped_at_1_0` terpicu oleh
satu piksel — pada pemandangan nyata, ia melaporkan `True` sementara fraksinya adalah `0.000001`, satu piksel
dari sejuta. Fraksi itulah yang memberi tahu Anda apakah perlu menaikkan batas atas.

**`bit_depth=8` hanya mencakup reflektansi.** Permintaan untuk `radiance` akan ditolak, bukan
diproses: reflektansi memiliki rentang tetap 0..1 yang tetap utuh dalam pengkodean 8-bit, sedangkan radian tidak memiliki
rentang tetap, sehingga untuk memasukkannya ke dalam 8 bit diperlukan peregangan persentil per bingkai yang pemetaannya
berubah setiap bingkai — sebuah gambar, bukan pengukuran.

#### Aliran reflektansi langsung (MJPEG)

`GET /api/camera/<serial>/stream-reflectance` menyajikan reflektansi 8-bit per-band sebagai
`multipart/x-mixed-replace` — M3C sebagai JPEG 3-saluran, M3M sebagai skala abu-abu. Parameter kueri:
`fps` (default 10), `jpeg_q` (default 95), `max_h` (0 = asli), `band` (satu band bernama, seperti
di atas), `align` (`1` untuk pendaftaran bersama — aturan yang sama seperti `align` di SDK), `refl_max` (batas atas
pengkodean 8-bit, seperti di atas). Setiap bagian membawa
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned`, dan `X-Chloros-Reflectance-Clipped`.
**Bagian dari kamera yang berbeda yang membawa `X-Chloros-Group-Id` yang sama merupakan pemicu yang sama** —
begitulah cara beberapa aliran per-kamera dari satu array disatukan kembali, dan hal ini dijamin
di sisi server karena anggota array disajikan dari grup bingkai yang disinkronkan (lihat
[Satu pemicu per pembacaan](#one-trigger-per-read--group_id)). Fungsi ini mengembalikan `409` jika nilai reflektansi
tidak tersedia saat koneksi terjalin dan menghentikan aliran jika nilai tersebut tidak tersedia di tengah proses, sehingga aliran yang terhenti
berarti “reflektansi tidak tersedia lagi” alih-alih “masih baik-baik saja, hanya statis”.

> **Format JPEG bersifat lossy — piksel-piksel ini merupakan hasil pengukuran.** Kesalahan kompresi langsung memengaruhi
> nilai reflektansi dan paling parah terjadi di tepi, yang merupakan batas material. Kualitas
> secara default diatur ke 95 karena alasan tersebut. Jika Anda membutuhkan angka-angka yang utuh, gunakan
> `grab(processing="reflectance")` / `frame_stream(...)`, yang menghasilkan produk yang sama tanpa
> codec dalam jalurnya (dan gunakan `bit_depth=8` jika biaya koneksi, bukan codec, menjadi kendala).

> **Data EXIF diteruskan ke hasil ekspor.** `process()` menyalin blok GPS dari tangkapan sumber
> **dan ExifIFD-nya** ke setiap produk, sehingga ekspor membawa `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal`, dan `CameraSerialNumber` serta
> georeferensi. `FocalLength` adalah data yang digunakan Pix4D untuk menghitung jarak sampel permukaan (GSD) — tanpa data ini,
> rekonstruksi akan menghasilkan skala yang sangat salah (dalam sebuah kasus yang diukur, lokasi seluas 411 m
> berubah menjadi 47,8 km). Salinan ini sengaja tidak menggunakan `-all:all`: tag struktural IFD0 merusak
> keluaran LATTICE, dan `ExifImageWidth`/`Height` dikecualikan karena keduanya mendeskripsikan
> pengambilan sumber, bukan raster yang diekspor.

Sub-flag tahap penangkapan (berlaku untuk tingkat radiometrik — `radiance`, `reflectance`, `display`):

| Bendera | Default | Arti |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + flat-field + pemisahan 3x3 + skala radiometrik NIST. |
| `apply_white_balance` | `True` | LUT WB. Mendukung DLS saat DAQ terhubung ke kamera. |
| `apply_index` | `False` | Evaluasi indeks vegetasi. |
| `index_expression` | `None` | Mengganti rumus. Jika tidakkosong → mengaktifkan indeks secara otomatis. |
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

> **Nilai kembaliannya adalah `chloros_sdk.CapturePaths`, bertipe `CapturePathMap`, bukan `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` adalah `Dict[str, Union[str, List[str]]]`: satu tingkat
> `processing` memberikan satu jalur untuk setiap nomor seri, sedangkan `processing="all"` memberikan
> **daftar terurut** dari setiap produk yang disimpan untuk kamera tersebut. `ArrayHandle.capture()` tidak menerima
> tidak menerima `levels=`, dan mengabaikan argumen kata kunci yang tidak dikenalnya (`levels`, `daq`,
> `render_index`, `force_daq`), jadi berikan hanya yang ditampilkan di sini (`aligned=False`
> menyimpan file asli, seperti pada `ArraySession.capture()`); gunakan
> `ArraySession.capture()` untuk kontrol-kontrol tersebut. Komposit gabungan langsung, jika sedang streaming, akan tiba di bawah kunci tambahan
> kunci `"combined"`, bukan di bawah serial. Kode yang mengasumsikan `str` akan mengalami kesalahan pada
> bentuk daftar tanpa ada pemeriksa tipe yang keberatan — anotasi tersebut menyebutkan `Dict[str, str]`
> selama beberapa waktu setelah bentuk daftar dirilis, itulah sebabnya alias tersebut ada. Normalisasikan
> jika Anda menginginkan bentuk datar:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **Yang tidak bisa ditulis: `.failed`.** Peta yang dikembalikan oleh `capture()` adalah
> `chloros_sdk.CapturePaths` — subkelas `dict` biasa, sehingga semua yang disebutkan di atas tetap berfungsi
> tanpa perubahan — yang juga mencakup `.failed`: berkas-berkas yang tidak dapat ditulis oleh backend,
> entri yang sama dengan [`CaptureResult.failed`](#files-that-could-not-be-written--captureresultfailed)
> (`serial`, `level`, `error`, dan `disk_full: True` pada drive yang penuh). Penangkapan sebagian
> tetap mengembalikan setiap jalur yang telah ditulis; pada drive yang penuh, peta tersebut
> kosong dan `.failed` menjelaskan alasannya. `.skipped` mencantumkan lompatan per kamera dan
> `.group_seq` mencantumkan grup yang disinkronkan, dan `.alignment` / `.oriented` adalah milik
> backend, seperti pada `CaptureResult`. `capture_smart()` mengembalikannya sebagai
> `result["frames"].failed` dan `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Penyelarasan Array

`ArrayHandle` menampilkan seluruh permukaan penyelarasan. Profil bersifat hanya untuk sesi secara default — panggil `export_alignment()` secara eksplisit untuk menyimpannya secara permanen.

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

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Penyesuaian Saat Koneksi

`connect_all(align=...)` dapat menyesuaikan setiap array secara otomatis saat koneksi:

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

Jika tidak ditentukan, akan menggunakan `project.json["config"]["auto_align_on_connect"]` sebagai alternatif.

### `SensorHandle`

Sensor cahaya yang disimpan (DAQ-U, DAQ-M, atau DAQ-E), dibuka oleh `connect_all()` di [kumpulan sensor DAQ](#daq-sensor-sessions) — kumpulan yang sama yang digunakan oleh `connect_daq_sensor()` dan `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` menunggu spektrum **berikutnya** dari sensor — spektrum yang sudah disimpan dalam buffer saat Anda memanggil fungsi ini tidak dihitung — dan mengembalikan `(spectrum, is_saturated, integration_time, x, y, z)`, atau `None` jika tidak ada yang tiba dalam waktu 5 detik. Aliran yang dijeda dilanjutkan. `spectrum` dinyatakan dalam W/m²/nm saat sensor dikalibrasi.
- `.sensor` adalah [`DAQSensorSession`](#daq-sensor-sessions), untuk yang lainnya: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name`, dan `.settings` adalah entri yang disimpan dari `sensors.json`.

**Apa yang dipulihkan oleh `connect_all`.** Setiap sensor dibuka pada alamat yang disimpan — port COM-nya, alamat Bluetooth, atau host DAQ-E — dan, jika proyek telah mencatat ID sensor tersebut, maka haruslah sensor itu: unit lain di alamat tersebut akan ditolak, dianggap tidak terhubung. Waktu integrasi, rata-rata bingkai, dan eksposur otomatis yang disimpan diterapkan, serta koreksi cap-nya:

| Batas yang tersimpan | Terhubung dengan |
| --- | --- |
| Profil batas (mis. `fov_30`) | Profil tersebut. |
| Tidak dipilih | `sunshine_cosine`, pengaturan default koneksi, tercatat sebagai yang diasumsikan (`cap_id_source=auto_default`), sehingga pemrosesan memberikan peringatan. Sensor dikirim dalam kondisi polos: pilih penutup yang terpasang. |
| `none` — dipilih sebagai **Tidak Ada (sensor tanpa penutup)** di antarmuka pengguna | Tidak ada koreksi penutup. |

DAQ-E-S tidak memerlukan profil penutup, apa pun yang telah disimpan: difusornya merupakan bagian dari kalibrasinya (lihat [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Sebuah proyek yang disimpan oleh Chloros versi sebelumnya mencatat sensor tanpa penutup sebagai opsi yang tidak dipilih, sehingga terhubung kembali dengan `sunshine_cosine`; pilih **None (sensor tanpa penutup)** di antarmuka pengguna (GUI) sekali lagi dan simpan proyek tersebut.

Sensor yang sudah dibuka oleh klien lain di pool (CLI, skrip lain) akan terhubung apa adanya, tanpa menerapkan pengaturan ulang, dan `disconnect_all()` membiarkannya tetap berjalan; sensor yang dibuka oleh `connect_all()` dilepaskan. Seperti halnya `connect_daq_sensor()`, GUI Chloros **bukan** bagian dari pool: tutup sensor di GUI sebelum menghubungkan proyek dari skrip.**Persyaratan dan laporan.** Sensor cahaya memerlukan backend Chloros yang aktif dan masuk ke tingkat berlangganan Chloros+ berbayar (Copper atau lebih tinggi), dan `connect_all()` tidak pernah memulainya. Jika tidak ada yang berjalan, setiap sensor yang disimpan akan terdaftar di bawah `report["sensors"]["errors"]`, dengan keterangan tersebut. Sensor yang tidak dapat dibuka oleh backend juga tercantum di sana, beserta alasan dari backend; `report["sensors"]["missing"]` tetap kosong untuk sensor cahaya.

---

## Perangkat Keras Langsung (Tanpa Backend)

Jika Anda ingin tidak bergantung sama sekali pada backend untuk kamera LATTICE (CI, robot tanpa layar, tertanam), impor `lattice_sdk` secara langsung — modul ini diekspor ulang oleh `chloros_sdk`. Peringatan mengenai `CAMERA_AVAILABLE`: `lattice_sdk` terdapat dalam paket SDK, tetapi memerlukan runtime Arena SDK dan OpenCV (`cv2`) yang terpasang, sedangkan paket tersebut tidak menginstal keduanya. Sensor cahaya diproses melalui backend — [`connect_daq_sensor()`](#daq-sensor-sessions) atau `proj.sensors` dari suatu proyek — dengan satu pengecualian, yaitu helper magnitudo pita DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), yang juga memerlukan `bleak`.

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

Tiga dari empat preset beroperasi dalam mode **free-run**: kamera melakukan eksposur secara terus-menerus dan
`capture()` mengembalikan bingkai berikutnya. `triggered` merupakan pengecualian — preset ini mengaktifkan
kamera untuk mendeteksi sinyal tepi perangkat keras pada Jalur 2, sehingga tidak merekam apa pun hingga sinyal tersebut tiba.

| Preset | Pemicu | Digunakan saat |
| --- | --- | --- |
| `default` | free-run | penggunaan umum |
| `high_speed` | free-run | 8-bit, batas 60 fps, eksposur singkat |
| `high_quality` | free-run | 12-bit, tanpa batas fps — pilihan umum untuk foto |
| `triggered` | **siap, Line 2** | kamera terhubung ke kabel sinkronisasi M8 dan dipicu oleh perangkat lain |

Jika Anda memilih `triggered` (atau mengatur `trigger_mode="On"` sendiri) tanpa ada
yang mengendalikan Line 2, setiap `capture()` akan mengalami timeout — sebagaimana mestinya, karena Anda meminta
kamera untuk menunggu. SDK menjelaskan hal ini saat terjadi; lihat
[SC_ERR_TIMEOUT selama perekaman](#direct-hardware-backend-free).

> **Catatan — Pesan &quot;GVSP probe&quot; / `SC_ERR_TIMEOUT -1011` saat terhubung bukanlah kesalahan.**
> Saat terhubung, SDK mencoba menegosiasikan**jumbo frames** (paket GVSP 9000 byte) untuk throughput yang lebih tinggi. Pada tautan NIC titik-ke-titik langsung (misalnya, alamat `169.254.x.x` link-local), jaringan biasanya tidak dapat mentransmisikan jumbo frames, sehingga proses negosiasi ini mengalami timeout dan mencatat baris seperti:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Inilah **mekanisme cadangan yang dirancang**: SDK secara otomatis kembali ke paket standar 1500 byte dan kamera tetap terhubung seperti biasa (baris-baris `[chunk-enable …]` yang mengikuti merupakan bagian dari urutan koneksi normal). Perekaman tetap berfungsi.
>
> Anda dapat melewati probe ini, tetapi **ini bukan sekadar penekan log — probe ini mematikan dukungan jumbo frame.** Kamera hanya merespons ping &quot;Don&#x27;t-Fragment&quot; hingga 1500 byte terlepas seberapa baik jaringan Anda, sehingga uji ping saja tidak pernah dapat mendeteksi jumbo; probe ini adalah satu-satunya yang bisa. Nonaktifkan probe ini, dan kamera akan terus menggunakan paket standar 1500 byte selamanya, di jaringan mana pun:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Hanya berguna di jaringan yang Anda *tahu* tidak dapat menangani jumbo, di mana hal ini menghemat sekitar satu detik waktu koneksi per kamera. Karena ini merupakan pertukaran yang nyata, bukan sekadar perubahan kosmetik, SDK kini memberikan peringatan saat Anda menggunakannya:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Biarkan saja kecuali Anda memiliki alasan.**Jika dibiarkan aktif, setiap kamera diukur sekali per sesi backend, dan diukur lagi saat koneksinya berubah (adaptor lain, kecepatan koneksi berbeda, atau koneksi terputus, seperti saat mengganti switch); koneksi ulang akan menggunakan hasil pengukuran sebelumnya.**Setelah terjadi perubahan di belakang switch yang tetap aktif, restart backend untuk melakukan pengukuran ulang.** Koneksi pertama terkadang dapat memakan waktu beberapa menit pada pemeriksaan ini, dan log akan mencatatnya jika hal itu terjadi.
>
> Jika Anda *ingin* throughput jumbo, aktifkan jumbo end-to-end (MTU NIC 9000 + switch yang mendukungnya), atau tetapkan secara tetap dengan `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` saat Anda yakin koneksi tersebut mendukungnya — meskipun disarankan menggunakan perintah per-koneksi `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` daripada pengaturan permanen, karena ukuran yang dikunci akan melewati proses pengujian dan menghentikan adaptasi terhadap jaringan di depannya. **Setiap** perangkat di jalur tersebut harus dapat meneruskan paket jumbo — termasuk splitter atau injektor PoE, yang biasanya menjadi alasan mengapa konfigurasi yang seharusnya mendukung paket jumbo tidak dapat meneruskannya.

> **`SC_ERR_TIMEOUT -1011` selama `capture()` / `grab*()` adalah masalah yang berbeda — yang satu ini adalah kesalahan yang sebenarnya.**
> Catatan di atas hanya mengenai `-1011` yang tercatat oleh**probe waktu koneksi**. Kesalahan yang sama yang muncul dari**penangkapan** berarti kamera terhubung dengan baik tetapi tidak mengirimkan gambar apa pun:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Ciri khasnya adalah kamera yang saluran *kontrol*-nya berfungsi dengan baik — proses penemuan berhasil, pengaturan dan penulisan `[chunk-enable …]` semuanya berhasil — sementara *setiap* frame mengalami timeout.
>
> **Penyebab umumnya adalah kamera telah diaktifkan untuk pemicu perangkat keras.** Dengan kode kesalahan `trigger_mode="On"` dan `trigger_source="Line2"`, kamera tidak mengirimkan apa pun sama sekali hingga sinyal tepi listrik tiba pada kabel sinkronisasi M8. Jika tidak ada kabel yang menggerakkan jalur tersebut, setiap proses pengambilan data akan menunggu selamanya. Kamera tidak rusak dan jaringan berfungsi dengan baik — kamera hanya menjalankan perintah yang diberikan.
>
> `CameraSettings()` serta `default` / `high_speed` / `high_quality` diatur ke mode free-run, dan proses pengambilan gambar yang habis waktunya saat dalam keadaan siaga akan menampilkan penjelasan alih-alih hanya menampilkan kode `-1011`. `PRESETS["triggered"]` mengaktifkan Line2, sesuai desain.
>
> Untuk memaksa kamera apa pun agar beroperasi dalam mode free-run:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Jika masih mengalami timeout dengan `trigger_mode="Off"`, berarti kamera tersebut benar-benar tidak mengirimkan data — kirimkan log dan `ip link show` kepada kami.

#### Profil Warna (pratinjau langsung RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` menentukan profil warna layar untuk **pratinjau langsung** pada kamera RGB (kamera multispektrum mengabaikan pengaturan ini):

| Profil | Arti |
| --- | --- |
| `raw` | Melewati rantai radiometrik sepenuhnya. |
| `linear` | DSNU + flat + WB, tanpa CCM, tanpa gamma. |
| `natural` | Linear + CCM terukur + gamma sRGB, hanya dengan penyempurnaan sederhana (penghalusan kroma + desaturasi sorotan) — pengaturan default yang realistis. |
| `enhanced` | `natural` ditambah penyempurnaan hub-parity lengkap (penghilangan fringes, vibrance, kontras lokal CLAHE). Tampilan yang lebih kaya dengan biaya penyempurnaan per-frame sekitar **dua kali lipat**, sehingga framerate LIVE lebih rendah. |
| `custom_temp` | `natural` tetapi WB dikunci pada `custom_cct_k` Kelvin (DLS diabaikan; dibatasi pada 2000–10000 K di sisi backend). |

Profil ini adalah pengatur kecepatan/tampilan **hanya untuk pratinjau langsung**: tangkapan yang disimpan selalu mendapatkan hasil akhir yang kaya sepenuhnya terlepas dari profil yang dipilih, sehingga memilih `natural` untuk menghemat waktu per frame tidak menurunkan kualitas hasil yang tersimpan di disk. Profil yang tidak dikenal akan meningkatkan nilai `ValueError`; ketika backend chloros dapat dijangkau, perubahan tersebut juga dikirimkan (POST) ke backend tersebut sehingga bingkai pratinjau berikutnya akan mencerminkannya (pengguna SDK langsung tanpa backend tetap mendapatkan perubahan pengaturan).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Sorotan (layar RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` mengambil piksel paling terang dan paling putih dari **layar** pada kamera RGB — solusi untuk permukaan putih cerah yang ditampilkan sebagai gumpalan putih datar. `value` adalah `-100`..`0` (dibatasi); `0` dinonaktifkan, dan hasil rendernya kemudian persis seperti aslinya tanpa pengaturan tersebut. Pengaturan yang lebih kuat menurunkan tingkat putih layar (hingga sekitar 60% pada `-100`) dan memperluas kurva nada, sehingga permukaan yang kecerahan otomatis pratinjau dorong menjadi putih murni akan mendapatkan detailnya kembali.

Ini adalah pengaturan tampilan, tetapi tidak seperti saturasi dan kontras (yang hanya mengubah pratinjau langsung), pengaturan ini juga memengaruhi apa yang Anda simpan: pratinjau langsung dan tangkapan **tingkat tampilan** mengikuti pengaturan ini, sehingga file-file tersebut juga menjadi lebih gelap (mereka menggunakan nilai `Chloros:DisplayHighlights`); ekspor radian dan reflektansi tidak pernah terpengaruh, dan pengaturan ini tidak dapat memulihkan data sensor yang terpotong — kurangi eksposur untuk itu. Kamera multispektral dan monokrom mengabaikannya. Nilai ini disimpan pada `settings.highlights` dan dikirim melalui POST ke backend chloros yang dapat dijangkau, persis seperti `set_color_profile`; setara dengan CLI-nya adalah `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Kamera Mono (M3M) dan `Calibration`

Kamera mono **M3M** (`M3M-<lens>-F<wavelength>`) bersifat single-band: satu bidang skala abu-abu, tanpa mosaik Bayer, tanpa matriks crosstalk spektral 3×3. `Calibration` mengenali kamera ini dan menampilkan bendera `is_mono`. Reflektansi masih berlaku sebagai peta radiometrik per-pita (proses pemisahan warna menggunakan matriks identitas), namun perhitungan multi-pita pada satu kamera menghasilkan hasil yang valid, bukan nilai yang tidak bermakna:

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

Untuk membuat indeks vegetasi dari perangkat keras monokromatik, gabungkan beberapa kamera M3M pada panjang gelombang yang berbeda menjadi tumpukan multi-pita yang selaras (lihat [Penyelarasan Array](#array-alignment)) dan hitung indeks tersebut pada tumpukan tersebut, bukan pada satu kamera.

#### Magnitudo pita DAQ-M — `DLS.band_magnitudes`

`DLS(mac_address, filter_model="RGN-IMX265")` terhubung ke DAQ-M melalui Bluetooth menggunakan alamat MAC BLE-nya sendiri. Ini adalah satu-satunya kelas sensor cahaya yang berkomunikasi langsung dengan sensor, bukan melalui backend, sehingga memerlukan apa yang dibutuhkan oleh kelas kamera langsung (`CAMERA_AVAILABLE`: runtime Arena SDK dan OpenCV) ditambah `bleak` untuk Bluetooth (`pip install "chloros-sdk[camera]"`, atau `pip install bleak`; tanpa itu, `DLS` akan memicu `DLSError`). Jika DAQ-M yang terhubung di tempat lain (pool backend, sebuah proyek, GUI) akan berhenti mengiklankan, jadi lepaskan terlebih dahulu di sana. **`mac_address` diperlukan**: tidak ada nilai default, karena setiap unit memiliki nilai sendiri, dan `DLS()` tanpa nilai tersebut akan memicu `DLSError` sebelum upaya koneksi apa pun dilakukan. Temukan alamat MAC dengan `discover_daq_sensors(transports=["ble"])` (lihat [Penemuan](#discovery--finding-an-address-to-connect-with)) atau `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` mengintegrasikan spektrum DAQ-M (pembacaan terbaru saat `spectrum` adalah `None`; `DLSError` jika belum ada) melalui respons tiga kanal kamera dan mengembalikan `{"red": …, "green": …, "nir": …}` — nama kunci tersebut sesuai urutan kanal; pada kurva kamera RGB, yang ketiga (`nir`) adalah kanal birunya.

- Tanpa `spectral_response`, fungsi ini menggunakan kurva standar model filter (kurva yang digunakan saat membangun `filter_model` dan `DLS`), yang diukur pada satu unit.
- `spectral_response` justru menggunakan kurva yang diukur dari kamera **itu sendiri**: sebuah kamus (dict) `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, sebagaimana disimpan dalam paket kalibrasi kamera tersebut (`color_correction.spectral_response_smoothed` — kamera RGB; paket multispektral tidak menyertakan kurva semacam itu). Jika kamus tersebut tidak dapat digunakan, sistem akan kembali menggunakan kurva armada.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

`chloros-cli lattice dls --camera SERIAL` pada CLI melakukan hal yang sama melalui backend, dengan mencari kurva kamera dalam paket kalibrasi yang dimuat atau disimpan dalam cache, serta menyebutkan kurva mana yang digunakan. Melalui HTTP, hal tersebut dilakukan oleh `POST /api/dls/cli-read` bersama dengan `mac` (diperlukan: tanpa ini, balasannya adalah HTTP 400, `code` `MAC_REQUIRED`) dan `camera_serial` di badan pesan; `band_magnitudes_source` pada balasan menjadi `camera <serial>` jika kurva kamera itu sendiri digunakan, atau `model <filter>` untuk kurva armada (kamera tanpa kurva: bukan RGB, atau paket tidak dimuat atau disimpan dalam cache).

---

## Pengecualian

Tangkap kelas dasar untuk menangani &quot;segala sesuatu yang salah pada Chloros&quot;:

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

> `ChlorosAuthenticationError` dan `ChlorosConfigurationError` adalah atribut dari `chloros_sdk` (dan dari `chloros_sdk.exceptions`) tetapi tidak ada di `__all__`, sehingga `from chloros_sdk import *` tidak memasukkannya: impor berdasarkan namanya. Keluarga `ChlorosError` diturunkan dari `ChlorosLocal` dan fungsi-fungsi yang dibangun di atasnya (`process_folder()`, `process_lattice_capture()`); permukaan smart-connect memunculkan `ChlorosConnectError`.

Hierarki:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

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
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
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

### 4. Aliran Frame Multi-Kamera → Pipa NumPy

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

### 5. Skrip Pengambilan Data Langsung ke Perangkat Keras Tanpa Antarmuka Grafis (Tanpa Backend)

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

### 6. Pengujian Kemampuan Sebelum Menghubungkan Array 4 Kamera

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
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Setara Resep Perekaman (Python Murni)

`run_project_recipe()` menjalankan resep CLI apa adanya (lihat [Folder Proyek &amp; Resep](#project-folder--recipes)). Jika ditulis dalam Python, langkah-langkahnya sama:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

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

## Server MCP (Asisten AI)

`chloros-mcp` memungkinkan asisten AI yang mendukung [Model Context Protocol](https://modelcontextprotocol.io) (MCP), seperti Claude Desktop, Claude Code, atau Cursor, untuk mengendalikan Chloros: memproses citra (membuat proyek, mengimpor folder tangkapan, memilih pengaturan, memproses, melaporkan hasil ekspor), serta mengoperasikan kamera LATTICE, array, dan sensor cahaya DAQ (menghubungkan, melihat pratinjau, mengatur eksposur dan warna, mengambil gambar, merekam, memperbarui firmware). Alat-alatnya mencakup fungsi yang dilakukan oleh SDK ini. Anda cukup bertanya dalam bahasa sehari-hari, dan asisten tersebut akan memanggil alat-alat server:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

Server ini dibangun di atas SDK ini: alat pemrosesan memanggil [`ChlorosLocal`](#image-processing--chloroslocal) dan alat-alat perangkat keras yang dipanggil oleh [smart-connect](#smart-connect-for-lattice-cameras), [sesi DAQ](#daq-sensor-sessions), `ChlorosProject`, dan [utilitas kamera](#camera-utilities), yang semuanya terhubung ke backend yang sama yang digunakan oleh aplikasi Chloros. Permintaannya membawa kode `X-Chloros-Client: sdk`, sehingga backend menerapkan pemeriksaan masuk dan paket langganan dari SDK, serta aturan SDK yang menyertainya (misalnya, Texture Aware ditolak untuk tangkapan LATTICE).

### Instalasi

Server disertakan dalam paket `chloros-sdk`; tidak ada yang perlu diinstal lagi. SDK menginstal perintah `chloros-mcp`, yang sama dengan `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** penginstal Chloros menginstal SDK menggunakan pip ke dalam Python Anda, yang menempatkan `chloros-mcp.exe` di folder `Scripts` Python tersebut.
- **Linux (.deb):** paket ini menginstal SDK untuk sistem `python3` dan menempatkan perintah `chloros-mcp` di `/usr/bin`, sehingga perintah tersebut tersedia di direktori `PATH`. Di dalam virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` menambahkan `chloros-mcp` ke dalam `bin` virtualenv tersebut.

Server ini hanya menggunakan pustaka standar di atas SDK (Python 3.7+). Klien MCP akan menjalankannya dan berkomunikasi dengannya melalui stdin/stdout; Anda tidak perlu menjalankannya sendiri, kecuali dengan `--help`.

### Pengaturan Klien

**Claude Desktop:**tambahkan server ke `claude_desktop_config.json` (di Windows, gunakan `%APPDATA%\Claude\claude_desktop_config.json`; buka melalui**Pengaturan &gt; Pengembang &gt; Edit Konfigurasi**), lalu mulai ulang Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Kursor:** entri yang sama dengan Claude Desktop, yaitu `mcpServers`, di `~/.cursor/mcp.json` (untuk setiap proyek; `%USERPROFILE%\.cursor\mcp.json` di Windows) atau `.cursor/mcp.json` dalam satu proyek.

Opsi dimasukkan ke dalam `args`, atau setelah perintah untuk Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Untuk menggunakan SDK Python Python tertentu sebagai pengganti `chloros-mcp` pada `PATH`, jalankan modul tersebut dengan Python tersebut (jalur lengkapnya sebagai `command` jika bukan yang pertama di `python3` pada `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Klien memulai server tanpa shell Anda, jadi jika dilaporkan bahwa server gagal dimulai, kemungkinan besar `chloros-mcp` tidak ada di `PATH` yang dilihat klien. Berikan jalur lengkap sebagai gantinya (`where chloros-mcp` di Windows, `which chloros-mcp` di Linux, di mana berkas .deb berada di `/usr/bin/chloros-mcp`), atau jalankan modul tersebut dengan Python yang telah dilengkapi SDK.

### Opsi

| Opsi | Default | Fungsinya |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | Backend Chloros yang akan digunakan. |
| `--no-auto-start` | off | Tidak pernah memulai backend: gunakan backend yang sudah berjalan, seperti backend aplikasi Chloros, layanan backend Linux, atau terowongan ke (lihat [Mode Backend Jarak Jauh](#remote-backend-mode-pip-only-host-via-tunnel); jalur folder harus merujuk ke folder yang sama di kedua mesin). |
| `--backend-exe PATH` | tempat Chloros menginstalnya | Backend yang akan dijalankan jika tidak ada yang sedang berjalan, untuk instalasi Chloros di mana SDK tidak mencarinya. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` atau `ERROR`. Log dikirim ke stderr, yang biasanya ditampilkan oleh sebagian besar klien dalam log MCP mereka; stdout hanya menampilkan pesan protokol. |

### Alat

Argumen setiap alat adalah argumen panggilan SDK yang dibungkusnya; klien mencantumkan masing-masing argumen beserta skema lengkapnya. Jalur yang digunakan bersifat absolut.

**Pemrosesan**

| Alat | Panggilan SDK | Fungsinya |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Membuat proyek dan menjadikannya proyek yang terbuka, yang akan diproses oleh alat-alat pemrosesan. Nama biasa akan membuat folder di dalam folder proyek default; jalur absolut akan membuatnya di sana. |
| `open_project` | `ChlorosLocal.open_project` | Membuka proyek yang telah disimpan dan menjadikannya proyek yang terbuka. Jalur proyek yang dipindahkan diarahkan ulang ke folder tempatnya berada saat ini; `relink_folder` (absolut) menyebutkan nama folder tempat file-file lainnya yang hilang berada saat ini. Mengembalikan `source_files` dan `image_count`. Tidak ditemukan: muncul pesan kesalahan yang mencantumkan lokasi file-file tersebut dan menamai `relink_folder`; tidak ada yang terbuka. |
| `import_images` | `import_images` | Mengimpor pasangan file `.RAW`+Survey3`.JPG`, LATTICE `.tif` menangkap dan DAQ `.daq` merekam; mengembalikan `count` (gambar) dan `scan_count` (rekaman). |
| `configure_processing` | `configure` | Mengubah pengaturan yang diteruskan dan tidak yang lain (nilai: [Nilai yang Didukung](#supported-values)); `custom_settings` mengirimkan seluruh konfigurasi sebagaimana diberikan. |
| `get_project_settings` | `get_config` | Membaca pengaturan pemrosesan, ekspor, dan indeks yang disimpan dari proyek yang terbuka. |
| `start_processing` | `process(wait=False)` | Memulai pemrosesan proyek yang terbuka dan segera kembali. |
| `get_processing_status` | `get_status` | Melaporkan `state` (`idle`, `starting`, `processing`, atau `finished`) serta kemajuan; setelah selesai, `summary`, `export_check_failure` untuk ekpor yang tidak dapat diverifikasi, dan `image_failure` jika gambar hilang: produk lainnya ditulis; laporkan proses tersebut sebagai gagal. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Membuat proyek untuk sebuah folder, mengimpornya, mengonfigurasi, dan memulai eksekusi dalam satu panggilan (`lattice: true` untuk pengambilan data LATTICE). Selama proses berjalan, sistem akan menolak sebelum membuat apa pun, dan backend juga menolak perubahan proyek dari alat mana pun pada saat itu. |
| `export_light_sensor` | `export_light_sensor` | Menulis berkas `.daq` dan `.csv` yang telah dikalibrasi untuk rekaman sensor cahaya proyek tanpa memproses gambar, mengambil kalibrasi pabrik dari MAPIR Cloud jika tidak tersimpan dalam cache (lihat [Perekaman Sensor Cahaya](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Membaca metadata audit Chloros dari gambar yang telah diproses (sumber, kalibrasi, tahap yang diterapkan). |

**Kamera dan susunan sensor**

| Alat | Panggilan SDK | Fungsinya |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | Apa yang terhubung saat ini, di klien mana pun, masing-masing dengan `connected_here`; ditambah pekerjaan yang sedang berjalan di server. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Kamera dan sensor cahaya yang dapat dihubungkan. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Status langsung satu kamera, susunan kamera (beserta kemampuan dan kondisinya), atau sensor cahaya. |
| `analyze_array_network` | `analyze_array_network` | Kapasitas yang dapat ditangani oleh host dan jaringan ini untuk sebuah array yang diusulkan. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Menghubungkan sebuah kamera, atau susunan yang disinkronkan (master terlebih dahulu). |
| `connect_project_devices` | `open_project(...).connect_all` | Hubungkan setiap perangkat yang tercatat dalam proyek yang disimpan, beserta pengaturannya. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Lepaskan kamera, susunan, atau sensor cahaya (lihat Perilaku). `keep_settings: true` pada susunan menyimpan pengaturan setiap kamera ke dalam memorinya masing-masing, sedangkan `clear_settings: true` mereset memori tersebut (hasilnya menghasilkan `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Perangkat-perangkat dalam proyek tersebut, secara bersama-sama. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Membaca atau menulis pengaturan kamera, berdasarkan nama ramah atau nama node GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Menetapkan laju array, atau mengembalikannya ke laju otomatis. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Satu pengambilan gambar (`fastest` dan `wait_for_exposure` adalah `capture_fastest` dan pengambilan gambar dengan eksposur yang telah disesuaikan). Tanpa `output_dir`, file-file tersebut disimpan ke dalam folder proyek yang terbukafolder `captures`, dan file-file tersebut ditambahkan ke proyek (ditahan hingga proses pemrosesan selesai); balasan dari `project_add` menyatakan demikian. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Pengambilan gambar berkelanjutan atau berinterval, sebagai sebuah tugas. |
| `start_array_recording` | `ArraySession.record` / `burst` | Video dengan indeks gabungan, atau burst mentah, sebagai tugas. |
| `build_burst_video` | `ArraySession.build_video` | Memproses ulang serangkaian gambar menjadi video yang telah dikalibrasi, sebagai tugas. |
| `get_job_status` / `stop_job` | | Mengikuti atau menghentikan suatu pekerjaan. |
| `get_camera_preview` | | Pratinjau langsung setiap kamera, dikembalikan sebagai gambar. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Satu bingkai (atau satu set yang disinkronkan) pada tingkat pemrosesan, tanpa penyimpanan: statistik per-bidang dan file PNG untuk masing-masing. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Penjajaran array. |

**Sensor cahaya**

| Alat | Panggilan SDK | Fungsinya |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Menghubungkan DAQ-U, DAQ-M, atau DAQ-E (`cap_id` mengatur koreksi kapasitornya). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | Pembacaan terbaru: cap waktu, saturasi, rentang spektrum dan puncak, serta spektrum yang dirata-ratakan ke dalam kelompok panjang gelombang (`full_spectrum` untuk setiap titik). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Memulai atau menghentikan aliran data. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Merekam berkas `.daq` sebagai tugas; `duration_s` akan menghentikannya secara otomatis. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | Koreksi batas (`none` untuk sensor tanpa penutup). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Sensor mana yang menyalurkan reflektansi masing-masing kamera dan susunan. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Kumpulan pita, serta reflektansi dan indeks Ambient + Objek dari dua sensor streaming. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Pengaturan khusus DAQ-E, disimpan di dalam unit. |

**Utilitas kamera** ([Utilitas Kamera](#camera-utilities))

| Alat | Panggilan SDK | Fungsinya |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` dan pengatur nilainya | Warna tampilan kamera RGB: profil dan suhu warna, saturasi / kontras, sorotan. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Keseimbangan putih sekali jepret, atau otomatis / mati / rasio tetap. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Pengaturan awal kamera. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Model filter, atau satu per satu berdasarkan nama. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Versi firmware, dan flash (secara default). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Status daya dan termal; tidur atau aktif. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Koefisien dari target kalibrasi yang terlihat. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Adaptor jaringan kamera, perkiraan, dan perbaikan (Hanya untuk Administrator). |
| `apply_vignette_correction` | `apply_vignette_correction` | Koreksi vignette pada folder gambar. |
| `read_dls` | `read_dls` | Satu pembacaan DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Perangkat keras dan runtime host. |

**Kamera transien, profil penyelarasan, dan indeks** ([Array Transien, Profil Penyelarasan &amp; Indeks](#transient-arrays-alignment-profiles--indices)). Setiap alat di sini yang membuka kamera menggunakan `keep_settings` / `clear_settings` (default false): lihat [Memori kamera](#camera-memory--keep_settings--clear_settings).

| Alat | Panggilan SDK | Fungsinya |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Merekam satu kamera, atau melakukan streaming beberapa kamera secara sinkron, sebagai sebuah tugas (`duration_s` akan menghentikannya secara otomatis). |
| `capture_cameras_transient` | `capture_cameras_transient` | Satu perekaman yang disinkronkan tanpa menghubungkan array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Pemeriksaan sinkronisasi pemicu, dan kamera mana saja yang terhubung ke jalur sinkronisasi. |
| `analyze_array_models` | `analyze_array_models` | Apa yang akan diperoleh oleh sebuah array model-model ini pada jaringan ini. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Menyelesaikan dan menyimpan berkas profil penyelarasan (`options` berisi opsi pemecah masalah yang lebih rinci). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Gambar multi-band yang telah diselaraskan, atau serangkaian gambar tersebut sebagai sebuah pekerjaan. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | Isi berkas profil; urutan band dan kamera yang diaktifkan. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Preset indeks dan gradien; indeks atas berkas TIFF multi-band; pengambilan gambar langsung ke indeks dalam satu langkah. |

**Sinkronisasi waktu, folder proyek, dan resep** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Folder Proyek &amp; Resep](#project-folder--recipes))

| Alat | Panggilan SDK | Fungsinya |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | Kondisi PTP grandmaster, peer-nya, dan status PTP setiap kamera array. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Mulai ulang; atur prioritasnya. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | Folder tempat proyek baru dibuat. |
| `run_project_recipe` | `run_project_recipe` | Jalankan resep proyek yang telah disimpan, sebagai tugas beserta log-nya. Diperiksa terlebih dahulu: resep yang mengandung kesalahan akan ditolak dan tidak menjalankan apa pun; array / sensor yang tidak terhubung akan diberi peringatan (tidak menghubungkannya: `connect_project_devices` terlebih dahulu). `dry_run` mengembalikan hasil pemeriksaan dan tidak menjalankan apa pun. |

Tidak ada alat untuk membuka proyek yang disimpan guna diproses: setiap pekerjaan pemrosesan dimulai dengan `create_project` (atau `process_folder`).

### Persyaratan

- **Chloros terinstal di komputer yang sama.** Backend Chloros melakukan pemrosesan. Server dapat menggunakan backend yang diakses melalui terowongan (`--backend-url` dengan `--no-auto-start`, lihat [Opsi](#options)).
- **Masuk ke akun di komputer tersebut**, baik melalui aplikasi Chloros maupun dengan `chloros-cli login`. Server tidak dapat masuk atas nama Anda: alat yang dijalankan saat Anda tidak masuk akan menampilkan pesan &quot;Belum masuk ke Chloros&quot; beserta alasan dari backend, dan asisten akan memberitahukannya kepada Anda.
- **Tingkat langganan Chloros+ berbayar apa pun (Copper atau lebih tinggi).** Server ini memerlukan akses SDK, sedangkan tingkat langganan gratis Iron tidak memilikinya: alat yang dipanggil pada tingkat langganan gratis akan menampilkan pesan &quot;Tidak tersedia pada paket Chloros ini&quot; beserta alasan dari backend.
- Python 3.7 atau yang lebih baru dengan SDK, serta klien MCP yang memulai server lokal (stdio).

### Perilaku

- **Backend.** Menjalankan server atau menampilkan daftar alatnya tidak akan memulai apa pun; panggilan alat pertama yang dilakukan akan terhubung. Jika aplikasi Chloros (atau layanan backend Linux) sedang berjalan, server akan menggunakan backend tersebut dan membiarkannya tetap berjalan. Jika tidak ada backend yang berjalan, server akan memulai backend yang terinstal, tanpa jendela. Saat server keluar (Anda menutup klien, atau klien me-restart server), server akan menghentikan tugas penangkapan data dan perekaman sensor cahaya yang telah dimulai, serta melepaskan perangkat yang terhubung; backend tetap berjalan, sehingga proses pemrosesan yang belum selesai akan berlanjut hingga tuntas, dan backend akan berhenti sendiri setelah 30 menit jika tidak ada yang menggunakannya. Dengan `--no-auto-start`, server tidak pernah memulai backend: panggilan alat yang dilakukan saat tidak ada backend yang berjalan akan meminta Anda untuk membuka aplikasi Chloros, dan panggilan berikutnya setelah Anda melakukannya akan terhubung.
- **Satu proyek terbuka.** Backend memiliki satu proyek terbuka, yang dibagikan dengan aplikasi Chloros saat aplikasi tersebut terbuka, jadi jangan mengerjakan proyek lain di aplikasi tersebut saat asisten sedang bekerja.
- **Hanya pengaturan yang diteruskan yang berubah.** `configure_processing` membiarkan setiap pengaturan yang tidak diberikan tetap pada nilai yang disimpan (SDK `configure()` mengatur ulang debayer, koreksi vignette, kalibrasi reflektansi, PPK, dan format ekspor ke nilai defaultnya jika tidak diberikan; server akan mengisinya dari proyek). Jika tidak dapat membaca pengaturan yang disimpan dalam proyek, aplikasi ini tidak akan mengubah apa pun dan akan memberitahukannya, kecuali jika kelima pengaturan tersebut diberikan. `indices` mengganti daftar indeks proyek. Hasilnya mencantumkan apa yang dikirim sebagai `settings_sent`, dan mencantumkan nama-nama di bawah `warnings` untuk setiap indeks yang tidak dapat dibuat oleh pita kamera.
- **Pemrosesan dilakukan dengan metode start-then-poll.** `start_processing` kembali segera setelah proses dimulai, yang kemudian memakan waktu beberapa menit hingga beberapa jam, dan asisten akan melanjutkannya dengan `get_processing_status`. `state` adalah `idle` (tidak ada pemrosesan), `starting` (backend belum melaporkan proses tersebut, hingga 20 detik), `processing`, atau `finished`, beserta [ringkasan](#post-run-summary--hints): apa yang diekspor, dan petunjuk jika hasil ekspor kurang dari yang diharapkan (tidak ada gambar, tidak ditemukan target kalibrasi). Hasil disimpan di bawah folder proyek, seperti yang dilakukan oleh aplikasi. `start_processing` ditolak saat proses sedang berjalan, dan backend yang berhenti merespons dilaporkan sebagai kesalahan, bukan sebagai proses yang selesai.
- **Apa yang dilihat oleh asisten.** Peringatan dari SDK muncul kembali dalam hasil alat di bawah `warnings`, dan kegagalan (keluar dari sistem, rencana, jalur yang salah) ditampilkan sebagai teks kesalahan yang dapat dibaca dan diteruskan kepada Anda. Daftar yang berisi lebih dari 20 item dipotong menjadi 20 item pertama, dengan panjang lengkapnya sebagai `<key>_total`. Jalur harus berupa jalur absolut.
- **Perangkat dibagikan.** Kamera, array, dan sensor cahaya berada di backend, sehingga asisten dapat mengendalikan salah satu perangkat yang terhubung melalui aplikasi Chloros, dan perubahan yang dilakukan akan ditampilkan di aplikasi tersebut. `disconnect_device` menampilkan perangkat yang terhubung ke server; perangkat yang terhubung oleh klien lain (aplikasi, `chloros-cli`, skrip, atau `chloros-mcp` lainnya) hanya dilepaskan melalui `force: true`, yang merupakan `force_disconnect` dari SDK, dan nama-nama yang menolak koneksi tersebut dilaporkan saat backend mengetahuinya; perangkat yang sama sekali tidak terdaftar di backend dilaporkan sebagai tidak terhubung (`released: false`, `connected: false`, tidak ada yang perlu dilaporkan). Rekaman atau streaming yang dijalankan oleh backend menampilkan tampilan `get_job_status` dengan `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (kelompok bingkai untuk streaming multi-kamera) — serta satu baris `summary` (&quot;220 frame ditulis, 55 terlewat, 0 gagal: ...&quot;), saat sedang berjalan dan, setelah selesai, angka-angka akhirnya; daftar dari klien lain mengenai hal tersebut (`other_clients_jobs`) menampilkan hal yang sama (`list_cameras()`, `list_arrays()`, dan `list_daq_sensors()` masing-masing menampilkan data`connected_by`: `client`, `label`, `name`, `at`). Setiap `chloros-mcp` mengidentifikasi dirinya ke backend sebagai `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, dikirim sebagai `X-Chloros-Client-Label`; skrip SDK apa pun dapat mengaturnya). Menghentikan perekaman sensor cahaya klien lain juga memerlukan `force`.
- **Pekerjaan yang memakan waktu adalah sebuah tugas.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, dan setiap perintah capture, solve, atau index yang diminta untuk `background`, langsung mengembalikan `job_id`; `get_job_status` mengikutinya dan `stop_job` mengakhiri proses tersebut. Proses akan terus berjalan hingga dihentikan kecuali Anda memberikan perintah `max_captures`, `duration_s` (atau, untuk mode burst, `max_frames`); `min_free_gb` menolak untuk dimulai jika ruang kosong pada drive tidak mencukupi. Tidak ada batasan yang diterapkan kecuali jika Anda menetapkannya. Perekaman atau streaming yang dijalankan oleh backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) menyimpan ID tugas backend tersebut, sehingga `chloros-mcp` lain — yang dimulai kemudian — dapat menemukannya berdasarkan ID tersebut: `get_job_status` akan mengikutinya (dan mencantumkan pekerjaan-pekerjaan seperti `other_clients_jobs`, dari `GET /api/camera/cli-jobs`, masing-masing dengan `started_by`), dan `stop_job` menghentikannya hanya dengan `force: true`. Pekerjaan lain apa pun (proses penangkapan, perekaman array atau burst, penangkapan latar belakang) berjalan di `chloros-mcp` yang memulainya, dan hanya server tersebut yang mengetahuinya: server lain yang memintanya akan mengonfirmasinya. Tugas yang dihentikan membaca `stopping` saat menulis antriannya, kemudian `finished` dengan jumlah akhir, selama 10 menit dan dari setiap `chloros-mcp`: server yang memulainya melihat penghentian paksa server lain dengan cara yang sama. `get_job_status` tidak pernah menunggu penghentian: selama proses penghentian (baik miliknya sendiri `stop_job`, klien lain, atau penghentian yang melepaskan kamera dari suatu pekerjaan yang berakhir dengan sendirinya), ia langsung merespons dengan `stopping` beserta hitungan langsung, dan `stop_job` dari suatu pekerjaan yang sedang dihentikan akan mengonfirmasinya. Pekerjaan yang telah berakhir akan menjelaskan alasannya di `stop_reason`: alasannya sendiri (`no_frames`, `disk_full`, `member_missing`, atau untuk serangkaian eksekusi berkelanjutan `count`, `duration`, `array_gone`), `limit reached` hanya ketika hitungannya menunjukkan `count` (atau `max_frames` dari sebuah burst, atau`duration_s`) tercapai, `error`, `stopped` (dengan `stopped_by`) saat klien menghentikannya, jika tidak maka `finished`.
- **Tempat penyimpanan rekaman.** Ke folder `captures` proyek yang terbuka, tempat aplikasi menyimpannya, kecuali jika `output_dir` ditentukan.
- **Gambar.** `get_camera_preview` dan `grab_frames` mengembalikan gambar yang dapat dilihat oleh asisten, selain data JSON-nya. Gambar `grab_frames` telah disesuaikan kontrasnya untuk tampilan; statistiknya adalah nilai sebenarnya.
- **Saat server berhenti**, tugas-tugas yang dimulainya akan dihentikan dan perekaman sensor cahaya yang dimulainya akan ditutup. Jika server telah memulai backend, server juga akan melepaskan perangkat yang telah dihubungkannya sebelum menghentikan backend; di backend aplikasi, perangkat-perangkat tersebut tetap terhubung.
- **Protokol.** Hanya stdio (tanpa titik akhir HTTP), hanya alat (tanpa sumber daya atau prompt), hasil teks dan gambar, versi protokol MCP dari 2024-11-05 hingga 2025-11-25.**Yang tidak termasuk.** Masuk atau keluar (akun adalah milik Anda; server tidak dapat masuk), dan menghentikan backend yang tidak dimulai oleh server. Semua hal lain yang dilakukan SDK ini terkait pemrosesan dan perangkat keras memiliki alat tersendiri. (Penyiapan DAQ-E — namanya, token otentikasi, kata sandi OTA, persyaratan tanda tangan, reset pabrik — serta server DAQ langsung yang hanya dimiliki oleh `chloros-cli`, juga terdapat dalam SDK: nama atau token yang salah akan mengunci setiap instalasi Chloros sehingga tidak dapat mengakses unit.)

---

## Pengaktifan Otomatis Backend

Titik masuk smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras`, dan `discover_daq_sensors` — adalah klien HTTP ringan yang mengasumsikan adanya backend yang mendengarkan di `127.0.0.1:5000` (URL default antarmuka smart-connect). Jika GUI atau CLI sudah berjalan, salah satunya pasti sudah aktif. Dari skrip dasar, mungkin tidak ada — sehingga fungsi-fungsi ini **secara otomatis menjalankan biner backend yang disertakan** (tanpa jendela, sama seperti yang dilakukan oleh `ChlorosLocal`) sebelum panggilan pertama, lalu menunggu hingga 60 detik hingga server tersebut aktif. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()`, dan `analyze_array_network()` tidak pernah memulainya.

Aturan:

- **Hanya URL lokal yang akan dijalankan.** Sebuah `backend_url` yang mengarah ke `localhost` / `127.0.0.1` / `[::1]` yang memenuhi syarat; host lain dianggap sebagai mesin milik orang lain dan tidak akan pernah dijalankan.
- **Backend yang keluar sebelum merespons akan mengakhiri masa tunggu.** Panggilan tersebut langsung mengembalikan kode keluar, perintah yang dijalankan, dan baris terakhir yang dicetak oleh backend, alih-alih menunggu selama 60 detik. Kode keluar 98 merupakan pengecualian: backend lain sudah menguasai port atau berkas kunci (milik aplikasi Chloros, layanan backend, atau skrip lain yang masih dalam proses memulai), sehingga panggilan tersebut terus menunggu backend tersebut merespons. `ChlorosLocal` melakukan hal yang sama.
- **Backend dibiarkan tetap berjalan untuk digunakan kembali** (sama seperti di CLI) — tidak ada penutupan otomatis saat skrip Anda berakhir. Menjalankan ulang skrip akan menggunakan kembali backend yang masih aktif.
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

Jika biner yang disertakan tidak dapat ditemukan atau dijalankan, panggilan HTTP berikutnya akan memicu pesan kesalahan `ChlorosConnectError` yang dapat ditindaklanjuti dan **sesuai platform**, bukan sekadar jejak penolakan koneksi — di Windows, pesan ini mengarahkan Anda ke aplikasi desktop atau perintah `chloros-cli`; di Linux (tanpa GUI), pesan ini mengarahkan Anda ke perintah `chloros-cli` atau `.deb`.

---

## Lingkungan &amp; Header

SDK menandai setiap panggilan HTTP backend dengan `X-Chloros-Client: sdk`. Backend menerapkan aturan lisensi SDK/CLI (diperlukan login **dan** paket Chloros+ berbayar) alih-alih jalur tier gratis antarmuka grafis. Pengaturan ini dilakukan secara otomatis saat proses impor — Anda tidak perlu melakukan apa pun.

Header ini ditambahkan ke setiap panggilan `requests` / `urllib` yang URL-nya berisi `localhost` atau `127.0.0.1`, di port mana pun — termasuk layanan lokal Anda sendiri — dan menggantikan nilai yang Anda tetapkan sendiri. Panggilan ke host lain tidak terpengaruh.

Ganti URL backend dengan meneruskan `backend_url=` (`ChlorosLocal` menerima `backend_url=` atau `api_url=`; sedangkan `process_folder()` dan `process_lattice_capture()` hanya menerima `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Backend yang terpasang hanya mendengarkan pada loopback, sehingga `backend_url` di LAN tidak dapat menjangkaunya; lihat [Mode Backend Jarak Jauh](#remote-backend-mode-pip-only-host-via-tunnel) untuk pola terowongan.)

---

## Versi &amp; Kompatibilitas

- Versi SDK-nya ditampilkan sebagai `chloros_sdk.__version__`.
- SDK mengikat perilaku pada versi backend yang disertakan. Menggabungkan SDK versi lama dengan backend versi baru biasanya berfungsi (titik akhir yang kompatibel ke depan), tetapi menggabungkan SDK versi baru dengan backend versi lama dapat menimbulkan kesalahan `404` pada titik akhir baru — perbarui aplikasi desktop agar sesuai.
- Antarmuka smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) dan titik akhir analisis jaringan mengembalikan skema JSON yang stabil; bidang baru ditambahkan.

---

## Petunjuk Pemecahan Masalah

- **`ChlorosAuthenticationError: Login required`** (dari `ChlorosLocal`; panggilan Smart-Connect memicu kesalahan `ChlorosConnectError: Login required. …` karena penyebab yang sama) → Jalankan `chloros-cli login EMAIL PASSWORD` sekali pada mesin ini, atau masuk melalui aplikasi desktop Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (atau `Failed to create project: …` dari `create_project()`) → Penggunaan jalur panjang Windows dinonaktifkan pada mesin ini (pengaturan default Windows), sehingga tidak ada jalur yang boleh melebihi 260 karakter, dan produk-produk berada hingga sekitar 130 karakter di bawah folder proyek. Tidak ada yang diproses. Pesan ini menunjukkan folder proyek terpanjang yang masih muat: gunakan yang lebih pendek, atau aktifkan jalur panjang Windows dan mulai ulang Chloros. Penginstal Windows menawarkan opsi ini sebagai &quot;Aktifkan jalur panjang Windows&quot;, yang dicentang secara default; jika tidak, gunakan Kebijakan Grup: *Konfigurasi Komputer &gt; Template Administratif &gt; Sistem &gt; Sistem Berkas &gt; Aktifkan jalur panjang Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → Fitur smart-connect memanggil auto-start pada backend lokal, sehingga pesan ini hanya muncul ketika biner yang disertakan tidak dapat ditemukan/dijalankan (misalnya, host pip-tanpa paket desktop). Pesan ini bergantung pada platform: di Windows, buka aplikasi desktop atau jalankan perintah `chloros-cli` apa pun; di Linux, jalankan perintah `chloros-cli` (tidak ada antarmuka grafis) atau instal `.deb`. Untuk backend jarak jauh, berikan argumen `backend_url=` (dan `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** saat impor → `lattice_sdk` gagal dimuat: runtime Arena SDK, atau OpenCV (`cv2`), tidak dapat diimpor di Python ini. Permukaan yang digerakkan oleh backend (`connect_*`, diproyeksikan melalui backend, `ChlorosLocal`) masih berfungsi.
- **Array terhubung pada sensor penuh tetapi laju bingkai rendah**→ Sesuai desain: mode otomatis mempertahankan sensor penuh dan membiarkan laju bingkai turun sesuai kapasitas kabel. Gunakan `binning=2` / `4` atau ukuran yang lebih kecil seperti `width` / `height` untuk mendapatkan frame per detik yang lebih tinggi, atau aktifkan jumbo frames / tingkatkan koneksi. Binning dan ROI**tidak** mencakup over-subscription agregat (`oversubscribed: true`, bidang fps 0): terlalu banyak kamera untuk koneksi kabel tidak dapat diatasi dengan binning/ROI — kurangi jumlah kamera, aktifkan jumbo frames, atau pindah ke NIC yang lebih cepat (lihat [Over-Subscription](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` melaporkan ring penerima NIC sangat kecil (~0,26 MB) / gerbang koneksi menampilkan &quot;FRAMES WILL DROP&quot;** → Ring penerima NIC host berada pada nilai defaultnya (sering kali diatur ulang menjadi 32 setelah pembaruan driver NIC). Pada adaptor Realtek USB 10GbE, atur `ReceiveBufferLen=256` dan `PendingReceives=64` (ditingkatkan), lalu mulai ulang backend agar membaca ulang ring tersebut. Prosedur lengkap: [Referensi CLI → Pengaturan &amp; Penyesuaian NIC Host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Host macet saat restart/shutdown, kemudian muncul kesalahan WMI `Invalid class` / NIC tidak dapat diaktifkan** → Driver USB 10GbE yang sudah usang menyebabkan `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Perbarui driver adaptor ke versi terbaru (≥ 2026) dan terapkan kembali pengaturan receive-ring. Lihat [Referensi CLI → Pengaturan &amp; Penyesuaian NIC Host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Penangkapan array gagal dengan kode HTTP-503 &quot;Tidak ada grup frame yang baru disinkronkan&quot;** → Array tidak menerbitkan grup frame baru selama periode tunggu. Periksa `arr.status()`: `stalled: True` (atau `streaming: False`) dengan nilai `actual_fps` 0 yang berarti loop pengambilan telah berhenti menerbitkan — putuskan sambungan dan sambungkan kembali array. Jika `stalled` bernilai False dan `actual_fps` di atas 0, coba lagi. Tepat setelah `connect_array()`, penangkapan sudah menunggu hingga 20 detik untuk grup pertama.
- **`status()` menampilkan `link_speed_mbps: 4294`** → Windows tidak dapat membaca kecepatan adaptor; hal ini umum terjadi pada adaptor Ethernet USB, dan bukan merupakan kesalahan. Lihat [Kecepatan Frame yang Diharapkan](#expected-frame-rates).
- **`update_firmware()` gagal dengan `device never connected back on TCP <port>`** → Firewall memblokir koneksi DAQ-E kembali ke komputer ini. Penginstal Windows menambahkan aturan tersebut (lihat [Pengaturan Perangkat DAQ-E](#daq-e-device-settings)); jika menggunakan firewall lain, izinkan lalu lintas TCP masuk ke backend dari subnet sensor.
- **Reflektansi ditolak** → DAQ yang aktif harus diikat ke kamera (atau array) untuk mendapatkan reflektansi skala absolut. Lakukan pengikatan melalui antarmuka pengguna grafis (GUI) atau gunakan `processing="radiance"` (W/m²/sr/nm) yang tidak memerlukan sensor yang dipasangkan.
- **Perekaman `smart=True` memakan waktu lebih lama dari yang diharapkan** → Konvergensi AE bergantung pada dinamika pemandangan. Dengan `ArrayHandle.capture_smart()`, perluas `exposure_tolerance_pct`, perpendek `stability_window_s`, atau kurangi nilai `settle_timeout_s` untuk pemicu yang lebih cepat dan kurang stabil.

---

## Lihat Juga

- [Referensi CLI](cli-reference.md) — setiap subperintah CLI mencerminkan panggilan SDK.
- [Sensor Cahaya DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) dalam manual Chloros — sensor, batas dan rentang kalibrasinya, perekaman, jaringan DAQ-E.
- [Kamera LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) dalam manual Chloros — koneksi, pengaturan kamera, mode pengambilan gambar, dan susunan kamera.
- Dokumen daring: `https://mapir.gitbook.io/chloros/api-python-sdk`

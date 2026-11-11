# Alur Kerja Reflektansi

Sensor cahaya DAQ mengubah citra radiometrik menjadi reflektansi. Terdapat dua alur kerja yang berbeda:

1. **Sensor tunggal** — satu DAQ mengukur irradiansi turun sementara kamera merekam, dan Chloros
membagi radian kamera dengan nilai referensi tersebut.
2. **Sensor ganda** — dua sensor DAQ, satu mengamati langit dan satu lagi mengamati objek, menghasilkan kurva reflektansi spektral secara langsung tanpa melibatkan kamera.

## Sensor tunggal + kamera (referensi cahaya yang turun)

DAQ berfungsi sebagai sensor cahaya yang turun (DLS): kamera mengukur radiansi naik **L**(W/m²/sr/nm), DAQ mengukur irradiansi turun**E** (W/m²/nm), dan Chloros
menghitung reflektansi per pita sebagai:

> ρ = π · L / E

Pembacaan DAQ selalu **disinkronkan dengan waktu pemaparan** — inilah mengapa DAQ dan kamera berbagi jam yang diatur oleh PTP (lihat [Jaringan &amp; Sinkronisasi Waktu DAQ-E](ethernet-ptp.md)). Pasang penutup kosinus Sunshine untuk pekerjaan di luar ruangan dan nyatakan dengan benar; pernyataan penutup tersebut secara langsung menskalakan E (lihat [Profil Penutup &amp; Rentang yang Dikalibrasi](caps-and-range.md)). Untuk pekerjaan kuantitatif, ingatlah karakteristik instrumen: irradiansi kuantitatif diperoleh dari rata-rata pembacaan setidaknya selama 15 detik.

### Pengambilan langsung

Hubungkan DAQ ke kamera di tab &quot;Cameras&quot;: panel pengaturan setiap kamera memiliki menu tarik-turun **Light Sensor** yang mencantumkan setiap DAQ yang terhubung (DAQ-U/M/E) dari tab &quot;Light Sensors&quot;; untuk susunan yang disinkronkan, pemilihan Light Sensor untuk seluruh susunan akan diterapkan ke setiap anggotanya (kamera individu masih dapat menggantikannya). Setelah dihubungkan, spektrum sensor akan dimasukkan ke slot DLS kamera dan ekspor reflektansi dibagi dengan pembacaan yang sesuai. Dalam sebuah array, anggota yang memiliki sensor sendiri menggunakan pembacaan sensornya sendiri — untuk pita-pita dalam komposit langsung, ekspor data, dan eksposur otomatisnya — sedangkan anggota lainnya menggunakan pembacaan dari array.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Dua perilaku yang perlu diketahui:

* **Tidak terikat DAQ → reflektansi ditolak, bukan dipalsukan.** Chloros
menolak hasil reflektansi dan mencatat alasan penolakan, bukan diam-diam mengembalikan hasil yang lebih rendah.
* **Pembacaan yang digunakan tetap dipertahankan.** Untuk setiap bingkai reflektansi, pembacaan DAQ yang sebenarnya diterapkan ditulis sebagai berkas `.daq` di dalam folder `daq/` dari rekaman tersebut, sehingga rekaman tersebut dapat diproses ulang nanti ([Perekaman &amp; Format .daq](recording.md)). Berkas `.daq.cameras.json` di sampingnya mencantumkan nama kamera yang digambarkannya, dan pemrosesan ulang hanya akan mencocokkannya dengan gambar dari kamera-kamera tersebut.

### Pemrosesan citra yang direkam

Untuk pemrosesan pasca-penerbangan, rekam berkas `.daq` selama sesi dan simpan bersama citra — alur kerja akan secara otomatis mencocokkan citra yang sesuai dengan cap waktu, serta mengambil kalibrasi pabrik yang hilang dari cloud MAPIR
pada penggunaan pertama. Rekaman antarmuka pengguna (GUI) akan ditambahkan ke proyek yang terbuka secara otomatis saat proses perekaman berhenti.

Referensi reflektansi dapat dipilih pada saat pemrosesan — `--reflectance-source` pada `chloros-cli process`, atau pengaturan sumber reflektansi di Pengaturan Proyek antarmuka pengguna (GUI):

| Nilai | Perilaku |
| --- | --- |
| `auto` (default) | Target kalibrasi dalam bingkai yang lulus QA menjadi referensi absolut; sinar turun DAQ (ρ = π·L/E) menjadi alternatif. Target yang lulus QA dengan bendera (panel terlepas, atau pita tidak berurutan) tidak secara otomatis diutamakan: rantai target, target+DAQ, dan DAQ-saja dicoba sesuai urutan skor terukurnya pada panel yang disisihkan. |
| `daq` | DAQ-otoritatif |
| `target` | Target dalam bingkai yang ketat; tanpa penggantian DAQ |

Lihat [Target Kalibrasi](../calibration-targets.md) untuk alur kerja target dan [bab LATTICE](../lattice/README.md) serta [Referensi CLI
](../reference/cli-reference.md) untuk alur pemrosesan lengkap. Saat membaca piksel reflektansi yang diekspor, gunakan skala yang tercantum (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3
: 65535) — lihat [Format Gambar Keluaran](../output-image-formats.md).

### Pita di luar rentang yang dikalibrasi DAQ

Rentang yang dikalibrasi secara radiometrik pada DAQ adalah ~374–974 nm. Chloros
menolak data reflektansi berbasis DAQ untuk band kamera mana pun yang memiliki kurang dari setengah bobot spektralnya berada di dalam rentang tersebut, dengan melaporkan alasan penolakan `dls-uncalibrated-band-<nm>`. Di antara SKU yang tersedia, hal ini hanya memengaruhi F988: reflektansi F988 dikalibrasi menggunakan panel reflektansi di lokasi pengukuran; karena pita tersebut berada di luar rentang kalibrasi sensor cahaya DAQ, Chloros
menggunakan data panel terbaru yang Anda ambil dan menyimpannya antara pengukuran panel. Jika kamera F988 dijalankan hanya dengan DAQ, Chloros
menolak reflektansi berbasis DAQ untuk pita tersebut dengan alasan pengabaian `dls-uncalibrated-band-988` — alur kerja panel adalah jalur yang didukung.

## Sensor ganda (cahaya sekitar + objek)

Dua sensor DAQ — pasangan apa pun, di transportasi apa pun — memberikan spektrum reflektansi langsung tanpa kamera: satu sensor menghadap langit (**Sumber Cahaya Sekitar**), satu menghadap subjek (**Pemindai Objek**), dan Chloros
menghitung per panjang gelombang:

> R(λ) = objek(λ) / ambient(λ)

(nol jika ambient ≤ 0).

### Di antarmuka pengguna (GUI)

Setelah kedua sensor terhubung di tab Light Sensors, buka overlay tambah sensor (tombol &quot;+&quot; pada ubin grafik dalam tampilan kisi) dan pilih **Gabungkan Cahaya Sekitar + Objek**. Pilih kedua sensor pada menu tarik-turun**Sumber Cahaya Sekitar**dan**Pemindai Objek**, lalu klik**Buat**. Kelompok tersebut akan muncul sebagai grafik tersendiri dan sebagai baris di bilah samping dengan lencana hijau**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->


Di bawah grafik reflektansi (tampilan daftar), sebuah **tabel indeks vegetasi** real-time menghitung indeks dari kurva tersebut menggunakan pusat pita pada biru 450 / hijau 550 / merah 670 / NIR
800 nm. Indeks berbasis rasio yang mengabaikan skala absolut (NDVI
, GNDVI
, ENDVI, WDRVI
, GRVI
, CVI
, GCI
, MSR) selalu ditampilkan; indeks yang memerlukan reflektansi absolut (EVI
, SAVI
, OSAVI
, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI
, NLI, MNLI, FCI, GEMI
) hanya muncul jika kedua sensor merupakan model yang telah dikalibrasi daya.

### Tampak vs. Relatif — aturan penamaan

Chloros
memberi label pada keluaran sensor ganda berdasarkan apa yang sebenarnya dapat diklaim oleh pasangan sensor tersebut:

| Pasangan sensor | Label |
| --- | --- |
| Kedua sensor dikalibrasi — paket kalibrasi pabrik dimuat | **Reflektansi Tampak** |
| Salah satu sensor belum dikalibrasi | **Reflektansi Relatif** |

Ketiga model tersebut bersifat radiometrik: begitu paket kalibrasi pabrik suatu sensor dimuat, spektrumnya menjadi nilai absolut W/m²/nm, sehingga rasio pasangan sensor yang telah dikalibrasi menghasilkan reflektansi tampak absolut — transportasi tidak menentukan nilainya. Sensor yang masih mengalirkan data mentah (paket tidak dapat dijangkau) akan menurunkan hasil menjadi kurva relatif (bentuk spektrum tetap valid). Kedua sensor harus memiliki batas yang dinyatakan dengan benar ([Profil Batas &amp; Rentang yang Dikalibrasi](caps-and-range.md)).

### Dari Python


Tidak ada panggilan khusus untuk dua sensor di antarmuka SDK
terpadu: buka dua sesi dengan `chloros_sdk.connect_daq_sensor()` dan hitung rasio spektrumnya dengan `latest()` sendiri, dengan menerapkan konvensi penamaan yang sama. (Alat perekaman dua sensor juga tersedia di antarmuka perangkat keras langsung internal MAPIR
, yang tercantum dalam [Referensi CLI
](../reference/cli-reference.md) demi kelengkapan — alat ini bukan bagian dari CLI
yang disertakan; alur kerja GUI di atas adalah jalur yang didukung.)

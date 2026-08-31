# Alur Kerja Reflektansi

Sensor cahaya DAQ mengubah citra radiometrik menjadi reflektansi. Terdapat dua alur kerja yang berbeda:

1. **Sensor tunggal** — satu DAQ mengukur irradiansi yang turun ke bawah sementara kamera merekam, dan Chloros membagi radian kamera dengan referensi tersebut.
2. **Sensor ganda** — dua sensor DAQ, satu mengamati langit dan satu mengamati objek, menghasilkan kurva reflektansi spektral secara langsung tanpa melibatkan kamera.

## Sensor tunggal + kamera (referensi cahaya yang turun)

DAQ berfungsi sebagai sensor cahaya yang turun (DLS): kamera mengukur radian ke atas **L**(W/m²/sr/nm), DAQ mengukur irradiance ke bawah**E** (W/m²/nm), dan Chloros menghitung reflektansi per pita sebagai:

> ρ = π · L / E

Pembacaan DAQ selalu **disinkronkan dengan waktu pengambilan gambar** — inilah mengapa DAQ dan kamera menggunakan jam yang disinkronkan melalui PTP (lihat [Jaringan DAQ-E &amp; Sinkronisasi Waktu](ethernet-ptp.md)). Gunakan penutup kosinus Sunshine untuk pekerjaan di luar ruangan dan tentukan profilnya dengan benar; penetapan profil penutup ini secara langsung memengaruhi nilai E (lihat [Profil Penutup &amp; Rentang yang Dikalibrasi](caps-and-range.md)). Untuk pekerjaan kuantitatif, ingatlah karakteristik instrumen: irradiansi kuantitatif diperoleh dari rata-rata pembacaan selama minimal 15 detik.

### Pengambilan gambar langsung

Hubungkan DAQ ke kamera di tab &quot;Cameras&quot;: panel pengaturan setiap kamera memiliki menu tarik-turun **Light Sensor** yang mencantumkan semua DAQ yang terhubung (DAQ-U/M/E) dari tab &quot;Light Sensors&quot;; untuk susunan yang disinkronkan, pemilihan Light Sensor secara keseluruhan akan diterapkan ke setiap anggota (kamera individu masih dapat menggantikannya). Setelah dihubungkan, spektrum sensor akan dimasukkan ke slot DLS kamera dan ekspor reflektansi akan dibagi dengan pembacaan yang cocok.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dua perilaku yang perlu diketahui:

* **Tidak ada DAQ yang terikat → reflektansi ditolak, bukan dipalsukan.** Chloros menolak hasil reflektansi dan mencatat alasan penolakan, bukan sekadar mengembalikan hasil yang lebih rendah tanpa pemberitahuan.
* **Nilai pembacaan yang digunakan tetap dipertahankan.** Untuk setiap bingkai reflektansi, pembacaan DAQ yang sebenarnya diterapkan ditulis sebagai file tambahan (sidecar) `.daq` di samping citra, sehingga pengambilan gambar dapat diproses ulang nanti ([Perekaman &amp; Format .daq](recording.md)).

### Pemrosesan citra yang direkam

Untuk pemrosesan pasca-penerbangan, rekam file `.daq` selama sesi berlangsung dan simpan bersama citra — alur kerja akan secara otomatis memproses data downwelling yang sesuai dengan cap waktu, serta mengambil kalibrasi pabrik yang mungkin terlewat dari cloud MAPIR pada penggunaan pertama. Rekaman antarmuka pengguna (GUI) akan ditambahkan ke proyek yang terbuka secara otomatis saat proses perekaman berhenti.

Referensi reflektansi dapat dipilih pada saat pemrosesan — `--reflectance-source` pada `chloros-cli process`, atau pengaturan sumber reflektansi di Pengaturan Proyek antarmuka pengguna (GUI):

| Nilai | Perilaku |
| --- | --- |
| `auto` (default) | Target kalibrasi dalam bingkai yang lulus QA menjadi referensi absolut; DAQ downwelling (ρ = π·L/E) menjadi alternatif |
| `daq` | Mengutamakan data DAQ |
| `target` | Target dalam bingkai yang ketat; tidak ada penggantian dengan data DAQ |

Lihat [Target Kalibrasi](../calibration-targets.md) untuk alur kerja target dan [bab LATTICE](../lattice/README.md) serta [Referensi CLI](../reference/cli-reference.md) untuk alur pemrosesan lengkap. Saat membaca piksel reflektansi yang diekspor, gunakan skala yang tertera (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — lihat [Format Gambar Keluaran](../output-image-formats.md).

### Pita di luar rentang yang dikalibrasi DAQ

Rentang yang dikalibrasi secara radiometrik pada DAQ adalah ~374–974 nm. Chloros menolak data reflektansi berbasis DAQ untuk pita kamera mana pun yang memiliki kurang dari setengah bobot spektralnya di dalam rentang tersebut, dengan melaporkan alasan pengabaian `dls-uncalibrated-band-<nm>`. Di antara SKU yang tersedia, hal ini hanya memengaruhi F988: reflektansi F988 dikalibrasi menggunakan panel reflektansi dalam adegan: pita tersebut berada di luar rentang kalibrasi sensor cahaya DAQ, sehingga Chloros menerapkan penangkapan panel terbaru Anda dan menyimpannya di antara pengamatan panel. Jika kamera F988 dijalankan hanya dengan DAQ, Chloros menolak reflektansi berbasis DAQ untuk pita tersebut dengan alasan melewati `dls-uncalibrated-band-988` — alur kerja panel adalah jalur yang didukung.

## Sensor ganda (cahaya sekitar + objek)

Dua sensor DAQ — pasangan apa pun, di transportasi apa pun — memberikan spektrum reflektansi langsung tanpa kamera: satu sensor menghadap langit (**Sumber Cahaya Ambient**), satu menghadap subjek (**Pemindai Objek**), dan Chloros menghitung per panjang gelombang:

> R(λ) = objek(λ) / ambient(λ)

(nol jika ambient ≤ 0).

### Di antarmuka pengguna (GUI)

Setelah kedua sensor terhubung di tab Light Sensors, buka overlay tambah sensor (tombol &quot;+&quot; pada ubin grafik dalam tampilan kisi) dan pilih **Gabungkan Cahaya Sekitar + Objek**. Pilih kedua sensor pada menu tarik-turun Sumber Cahaya Sekitar dan Pemindai Objek, lalu klik Buat. Kelompok tersebut akan muncul sebagai grafik tersendiri dan sebagai baris di bilah samping dengan lencana hijau**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Di bawah grafik reflektansi (tampilan daftar), sebuah **tabel indeks vegetasi** secara real-time menghitung indeks dari kurva menggunakan pusat pita pada biru 450 / hijau 550 / merah 670 / NIR 800 nm. Indeks berbasis rasio yang menghilangkan skala absolut (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) selalu ditampilkan; indeks yang memerlukan reflektansi absolut (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) hanya muncul jika kedua sensor merupakan model yang telah dikalibrasi daya.

### Tampak vs. Relatif — aturan penamaan

Chloros memberi label pada keluaran sensor ganda berdasarkan apa yang sebenarnya dapat diklaim oleh pasangan sensor tersebut:

| Pasangan sensor | Label |
| --- | --- |
| Kedua sensor dikalibrasi — paket kalibrasi pabrik dimuat | **Reflektansi Tampak** |
| Salah satu sensor tidak dikalibrasi | **Reflektansi Relatif** |

Ketiga model ini bersifat radiometrik: begitu bundel kalibrasi pabrik dari suatu sensor dimuat, spektrumnya menjadi nilai absolut W/m²/nm, sehingga rasio dari sepasang sensor yang dikalibrasi menghasilkan reflektansi tampak absolut — transportasi data tidak menentukan nilainya. Sensor yang masih mengirimkan data mentah (paket tidak dapat dijangkau) akan menurunkan hasil menjadi kurva relatif (bentuk spektrumnya tetap valid). Kedua sensor harus memiliki batas yang dinyatakan dengan benar ([Profil Batas &amp; Rentang yang Dikalibrasi](caps-and-range.md)).

### Dari Python

Tidak ada panggilan khusus untuk dua sensor di permukaan terpadu SDK: buka dua sesi dengan `chloros_sdk.connect_daq_sensor()` dan hitung rasio spektrum `latest()`-nya sendiri, dengan menerapkan konvensi penamaan yang sama. (Alat perekaman dua sensor juga tersedia di permukaan perangkat keras langsung internal MAPIR, yang tercantum dalam [Referensi CLI](../reference/cli-reference.md) demi kelengkapan — alat ini bukan bagian dari CLI yang dikirimkan; alur kerja GUI di atas adalah jalur langsung yang didukung.)

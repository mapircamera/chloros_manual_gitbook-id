# Tab DAQ di Chloros

Tab DAQ — yang diberi label **Sensor Cahaya** di bilah samping Chloros — adalah antarmuka kontrol langsung untuk [sensor cahaya DAQ-U, DAQ-M, dan DAQ-E](README.md): menghubungkan sensor melalui protokol apa pun, memantau spektrum yang telah dikalibrasi secara real-time, menghitung reflektansi secara langsung dari sepasang sensor, dan merekam berkas `.daq` langsung ke proyek Anda.

Tab ini akan tersedia setelah backend Chloros selesai dijalankan. Grafik pada tab ini disuplai oleh layanan DAQ Chloros melalui koneksi langsung yang akan terhubung kembali secara otomatis (dengan jeda 2–10 detik) jika terputus; selama layanan tidak dapat dijangkau, baris Status sensor akan menampilkan **No Server**.

Tata letaknya terdiri dari **bilah samping sensor**(satu baris per sensor yang terhubung) ditambah**area grafik** (satu ubin grafik per sensor atau grup).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Menghubungkan sensor

Klik **Connect Sensor** di bagian atas bilah samping. Kotak dialog koneksi akan terbuka di area utama (atau sebagai overlay saat menambahkan sensor lain — tombol Batal akan muncul dalam kasus tersebut).

| Kontrol | Perilaku |
| --- | --- |
| **Jenis Perangkat** | `DAQ-U (USB)` (default), `DAQ-M (Bluetooth)`, atau `DAQ-E (Ethernet)`. Mengubah pilihan akan memulai ulang pemindaian untuk transportasi yang baru dipilih. |
| **Port / Perangkat BLE / Nama Host / IP** | Menampilkan daftar perangkat yang terdeteksi sebagai `device - description`; entri pertama yang dikenali sebagai sensor akan dipilih secara otomatis. Selama pemindaian, ditampilkan `Scanning...` (USB), `Scanning (N)...` dengan hitungan mundur 8 detik (BLE), atau `Discovering ethernet sensors (N)...` dengan hitungan mundur 5 detik (Ethernet). Hasil kosong ditampilkan sebagai `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Refresh** | Segera memindai ulang transportasi yang dipilih (dinonaktifkan saat pemindaian BLE/Ethernet sedang berlangsung). |
| **Connect Sensor** | Diaktifkan setelah perangkat dipilih; mengganti label menjadi `Connecting...` saat koneksi sedang dibuat. |

Proses pencarian hanya berjalan **selama dialog koneksi muncul di layar**, dan diulang setiap 15 detik hanya untuk transportasi yang dipilih — sekadar membuka tab tidak akan memicu pemindaian. Jika terjadi kegagalan, dialog akan menampilkan: *"Koneksi gagal. Coba cabut dan pasang kembali sensor, lalu klik 'Connect' lagi."*

Bilah samping akan terbuka secara otomatis saat sensor pertama Anda terhubung.

{% hint style="info" %}
**DAQ-E tidak muncul?**DAQ-E tidak memiliki LED status — periksa indikator PoE/link pada switch atau port injektor tempatnya terpasang, dan tunggu beberapa detik setelah dinyalakan agar perangkat tersebut dapat melakukan booting. Perangkat Chloros harus berada di domain siaran yang sama (mDNS tidak dapat menembus router). Aturan firewall**Chloros Backend (UDP-In)** pada penginstal mengizinkan mDNS (UDP 5353), data DAQ-E (UDP 5002), dan PTP (UDP 319/320); firewall pihak ketiga perlu mengizinkan port yang sama. Dua unit DAQ-E dalam satu jaringan LAN akan terdeteksi secara terpisah, masing-masing dengan nama host `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Tipe Perangkat mencakup DAQ-U (USB), DAQ-M (Bluetooth), dan DAQ-E (Ethernet)</figcaption></figure>

***

## Bilah samping sensor

Setiap sensor yang terhubung mendapatkan satu baris (ditambah satu baris per grup Ambient+Object). Baris-baris ini dapat diurutkan ulang dengan cara diseret, dan urutannya juga akan mengatur ulang susunan ubin grafik. Klik sebuah baris untuk menjadikan sensor/grup tersebut sebagai grafik aktif dalam tampilan daftar.

| Elemen | Arti |
| --- | --- |
| Garis tepi kiri berwarna | Warna grafik sensor. |
| Lencana transportasi | `DAQ-U` / `DAQ-M` / `DAQ-E`, atau lencana hijau `REF` untuk kelompok reflektansi Ambient+Objek. |
| Nama perangkat | Secara default menggunakan nomor seri sensor (identitas tetapnya untuk kalibrasi, nama file `.daq`, dan pencocokan impor); nama khusus tetap berlaku per proyek. |
| Pil **Terkalibrasi** (hijau) | Ditampilkan saat paket kalibrasi pabrik sensor dimuat, yaitu spektrumnya benar-benar dalam satuan W/m²/nm. |
| Pil **Pembaruan Tersedia** (kuning, hanya DAQ-E) | Firmware yang sedang berjalan lebih lama daripada gambar yang disertakan dalam build Chloros ini. Selama pembaruan, pil ini menampilkan kemajuan secara langsung (`Flashing… N%`, `Restarting sensor…`, lalu `Updated X → Y` atau `Failed`). |
| Mata | Mengaktifkan/menonaktifkan tampilan sensor ini pada grafiknya. |
| Roda Gigi | Membuka jendela pengaturan per-sensor (di bawah). |
| ✕ (merah) | Memutuskan koneksi sensor, atau menghapus grup Ambient+Object. |

Di atas baris-baris tersebut terdapat dua tombol:

* **Hubungkan Sensor** — membuka dialog koneksi (berubah menjadi `Connecting...` saat sedang aktif).
* **Rekam Semua / Hentikan Semua**— memulai atau menghentikan perekaman `.daq` pada**setiap**sensor yang terhubung. Membutuhkan setidaknya satu sensor**dan proyek yang terbuka** (tooltip: "Buka proyek untuk merekam"); tombol ini berubah menjadi merah saat perekaman sedang berjalan.

Dalam keadaan kosong, tertulis "Tidak ada sensor yang terhubung".

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Pengaturan per-sensor (jendela modal roda gigi)

Buka dengan ikon roda gigi pada baris sensor. Isi secara berurutan:

* **Baris Info** — Jenis Perangkat (DAQ-U/M/E), Koneksi (`Serial (USB)` / `Bluetooth` / `Ethernet`), Port (port COM, alamat BLE, atau host), dan Serial.
* **Laporan Kalibrasi: Unduh** — mengunduh sertifikat kalibrasi yang dapat dilacak ke NIST (PDF) dari unit ini dan membukanya di pembaca PDF Anda. Tersedia setelah nomor seri diketahui; sertifikat disimpan dalam cache pada koneksi pertama.
* **Nama Perangkat** — klik ikon pensil untuk mengganti nama; pengaturan ini berlaku per proyek.
* **Warna Garis Grafik** — palet warna; pengaturan ini berlaku per proyek.
* **Waktu Integrasi (ms)**— penggeser + angka,**1–500 ms**, default**32 ms**. Dinonaktifkan saat AE aktif.
* **Rata-rata Frame**— penggeser + angka,**1–50 frame**, default**20**.
* **AE: AKTIF/NONAKTIF**— tombol pengalih eksposur otomatis;**default AKTIF** saat terhubung. Matikan untuk mengatur waktu integrasi secara manual.
* **Hentikan Streaming / Mulai Streaming** — jeda atau lanjutkan siaran langsung.
* **Rekam / Hentikan Perekaman** — perekaman per sensor `.daq` (memerlukan proyek yang terbuka).
* **Cap** — profil koreksi cap (bagian berikutnya).
* **Baris informasi langsung** — Waktu Integrasi (ms), FPS, Sampel, Perekaman (merah `REC` atau `Off`), dan Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Khusus DAQ-E: baris jaringan, firmware, dan PTP

* **Nama Host / IP** — alamat unit saat ini.
* **Firmware** — versi firmware saat ini, ditambah sel tindakan:<version\>

tombol</version\>

**Perbarui ke \<version\>** akan muncul jika build Chloros ini menyertakan gambar firmware DAQ-E yang lebih baru. Pembaruan dikirim melalui jaringan dalam waktu sekitar 30 detik; sensor akan restart dan terhubung kembali secara otomatis, dan jika transfer terputus, firmware saat ini tetap utuh. Progres ditampilkan secara langsung (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), dan sel tersebut menampilkan `Up to date` saat versi terbaru.
* **Sinkronisasi PTP** — status PTP secara langsung (kembali ke `unknown`). Firmware DAQ-E v1.2.0+ berpartisipasi dalam IEEE 1588 PTPv2 sebagai jam yang hanya berfungsi sebagai slave; backend host Chloros adalah grandmaster PTP, dan setiap DAQ-E serta kamera LATTICE di LAN menjadi slave-nya di domain 0, menjaga cap waktu tetap dalam rentang sekitar 1 ms.
* **Jaringan**— alamat yang digunakan unit dan cara mendapatkannya (DHCP, link-local, atau statis), serta pengaturan yang disimpan dan berlaku setelah restart.**Ubah…**membuka formulir:**Otomatis (DHCP)**atau**IP Statis**dengan alamat IP, masker subnet, dan gerbang (gateway) opsional.**Terapkan dan restart** menyimpan pengaturan, me-restart sensor, dan menghubungkannya kembali. Alamat statis di luar jaringan komputer ini ditolak, karena Chloros tidak dapat menjangkau sensor setelah restart; kotak centang untuk mengizinkannya tetap muncul, untuk komputer yang akan dialihkan alamatnya atau diarahkan ke jaringan tersebut. **Streaming saat dinyalakan**menentukan apakah sensor mulai melakukan streaming segera setelah dinyalakan, sebelum perangkat lunak apa pun terhubung.**Restart sensor** akan me-restart sensor dan menghubungkannya kembali dengan pengaturan saat ini. Restart akan ditolak saat sensor sedang merekam. Fitur baru di versi 1.3.0.

Untuk grup Ambient+Object, jendela pengaturan (gear modal) hanya menampilkan sensor sumber grup tersebut, Nama Perangkat, dan Warna Garis Grafik.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Pemilihan tutup

Menu tarik-turun **Tutup** memberitahu Chloros tutup fisik mana yang terpasang di atas difuser sensor, dan menerapkan profil koreksi yang diukur di pabrik dari tutup tersebut ke setiap spektrum. Pilihan bergantung pada model:

| Model | Pilihan tutup |
| --- | --- |
| DAQ-U | Tidak ada (sensor telanjang), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (korektor kosinus) |
| DAQ-M | Tidak ada (sensor telanjang), Sunshine (korektor kosinus) |
| DAQ-E | Tidak ada (sensor telanjang), FOV 15°, FOV 45°, FOV 90°, Sunshine (korektor kosinus) |

**Pengaturan default untuk setiap model adalah Sunshine (korektor kosinus)** — MAPIR mengirimkan setiap DAQ dengan penutup Sunshine terpasang, dan ini merupakan konfigurasi luar ruangan standar: pandangan hemisferik 180° dengan kesalahan kosinus ≤ ±4 % hingga 60° dan ≤ ±4,5 % hingga 70° (tidak disarankan di bawah ketinggian matahari ~15°), dengan peredaman sesuai desain (~12×). Pilihan Anda akan tetap berlaku di proyek.

{% hint style="warning" %}
**Pilihan penutup harus sesuai dengan penutup fisik yang terpasang.**Baik sensor maupun perangkat lunak tidak dapat mendeteksi penutup mana yang terpasang. Pilihan ini menentukan baik koreksi real-time maupun cap yang ditulis ke dalam setiap berkas `.daq` — dengan atenuasi ~12× dari penutup Sunshine, perubahan penutup yang tidak dilaporkan akan menyebabkan koreksi spektrum yang salah kira-kira sebesar faktor tersebut. (Melepas dan memasang kembali penutup yang sama menghasilkan pengulangan hingga sekitar 1,5 %.) Pilih**None (sensor telanjang)** hanya ketika penutup telah dilepas secara fisik; pada DAQ-E, "None" tetap menerapkan profil geometri pabrik untuk difuser kaca yang tersembunyi — ini bukan operasi kosong — dan DAQ-E tanpa penutup merupakan konfigurasi meja kerja, bukan konfigurasi lapangan yang didukung.
{% endhint %}

{% hint style="info" %}
Pembaruan dari panduan sebelumnya: tombol pengalih "Sunshine Diffuser Terpasang" di sisi browser dari versi 1.1.0 telah dihapus. Pengelolaan penutup kini dilakukan melalui profil penutup per-sensor ini, yang diterapkan di sisi server.
{% endhint %}

***

## Area grafik

Sebuah bilah atas yang tetap menempel menampilkan **tombol pengalih tampilan daftar ⇄ kisi**dan penggeser**Zoom Grafik** (ukuran ubin 200–2000 px). Tampilan akan beralih secara otomatis ke kisi jika terdapat lebih dari satu kelompok grafik, dan kembali ke daftar jika hanya ada satu atau kurang. Mode tampilan dan ukuran grafik disimpan per proyek.**Grafik spektrum** untuk setiap sensor menampilkan:

* **Sumbu X** — Panjang gelombang (nm). Kisi sensor berkisar antara 340–1010 nm dengan selang 5 nm (135 titik), diinterpolasi menjadi 1 nm untuk tampilan.
* **Sumbu Y**— untuk sensor yang telah dikalibrasi (pil hijau**Terkalibrasi**),**Daya (W/m²/nm)**— irradiansi spektral yang dikalibrasi secara radiometrik pada ketiga sistem satuan — dengan awalan SI otomatis (m/µ/n) yang dipilih berdasarkan puncak. Sensor yang paket kalibrasinya tidak dimuat akan menampilkan pembacaan mentahnya sebagai**Intensitas (counts)**, tanpa penskalaan. Ketika sebuah grafik menampilkan kedua jenis data tersebut secara bersamaan, daya ditampilkan pada sumbu kiri dan hitungan pada sumbu kanan.
* Isi spektral pelangi di bawah satu jejak; beberapa sensor pada satu grafik ditampilkan sebagai garis berwarna dengan isi yang diredupkan.
* **Arahkan kursor**— kursor vertikal dengan nilai panjang gelombang dan nilai per sensor (W/m²/nm untuk sensor yang dikalibrasi, hitungan utuh jika tidak);**seret** untuk memperbesar (tombol memperkecil akan muncul saat diperbesar).
* Tombol **+** (hanya tampilan kisi) untuk menambahkan sensor ke grafik ini atau membuat grup (di bawah).
* Nama perangkat ditempatkan di tengah bagian atas, dan spinner akan muncul hingga frame pertama tiba.

**Saturasi** tidak ditandai pada grafik itu sendiri: sensor yang jenuh menampilkan teks status merah `SATURATED` dan baris merah `Saturated: Yes` di tabel data langsung. Kurangi waktu integrasi atau aktifkan kembali AE untuk menghilangkannya.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tabel data langsung (tampilan daftar)

Di bawah grafik dalam tampilan daftar, diperbarui setiap 500 ms:

* **Semua model**: Sampel Warna Cahaya (sRGB dari CIE XYZ), Jenuh (Ya/Tidak), CIE 1931 X/Y/Z, Kromatisitas x/y, CIE u′/v′, CCT (K), CRI (Ra), Panjang Gelombang Dominan (nm), Panjang Gelombang Puncak (nm), Kemurnian Eksitasi, Duv, CIE L\*/a\*/b\*, dan Munsell H/V/C.
* **Hanya sensor yang telah dikalibrasi**(DAQ-U / DAQ-M / DAQ-E setelah paket kalibrasi pabriknya dimuat — tanda**Terkalibrasi** berwarna hijau pada baris sensor menjadi penanda): Daya Total (W/m²), Lux Fotopik (lx), Lux Skotopik (lx), Rasio S/P, PPFD serta PPFD Red /Green /Blue (µmol/m²/s), dan irradiansi opik — S-cone, Melanopic, Rhodopic, M-cone, L-cone (semua dalam satuan W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Kelompok reflektansi (Lingkungan + Objek)

Dua sensor yang terhubung dapat digabungkan menjadi tampilan reflektansi real-time — tanpa melibatkan kamera:

1. Dalam tampilan kisi, klik **+**pada ubin grafik dan pilih**Gabungkan Lingkungan + Objek**.
2. Pilih sensor **Sumber Cahaya Sekitar**dan sensor**Pemindai Objek**(dua sensor yang berbeda), lalu**Buat**.

Chloros menghitung R(λ) = objek(λ) / sekitar(λ) per panjang gelombang dari dua aliran data langsung (0 jika sekitar ≤ 0). Label grup mengikuti kelas kalibrasi sensor:

* Kedua sensor telah dikalibrasi (bundel dimuat) → **"Reflektansi Tampak"**.
* Salah satu sensor belum dikalibrasi → **"Reflektansi Relatif"**.

Grup ini muncul sebagai baris hijau `REF` di bilah samping dan grafiknya sendiri (isi pelangi, nilai saat kursor melayang hingga 4 desimal, zoom dengan menyeret).

Menu **+**juga menawarkan opsi**Tambahkan Sensor Baru** dengan tiga pilihan penempatan: *Gabungkan Sensor Baru* (gabungkan ke grafik ini), *Pindahkan Sensor yang Sudah Ada ke Sini*, atau *Lihat Sensor Baru* (grafik tersendiri).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tabel indeks vegetasi

Dalam tampilan daftar, tabel indeks vegetasi terletak di bawah grafik kelompok reflektansi, yang dihitung dari reflektansi real-time pada pusat pita **biru 450 / hijau 550 / merah 670 / NIR 800 nm** (nilai hingga 4 desimal, `---` jika tidak dapat dihitung; arahkan kursor ke nama indeks untuk melihat nama lengkapnya):

* **Selalu ditampilkan** (tidak bergantung pada skala, kombinasi sensor apa pun): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Hanya jika kedua sensor telah dikalibrasi daya** (kedua paket telah dimuat): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Merekam berkas `.daq`

* Perekaman memerlukan **proyek yang terbuka** — jika tidak, tombol "Record All" (di bilah samping) dan tombol "Record" per sensor akan dinonaktifkan.
* File ditulis ke **`<project folder>/light_sensor/`**; nama file berisi ID sensor dan cap waktu, serta nama perangkat disimpan bersama rekaman.
* Saat perekaman berhenti (Stop, Stop All, atau terputus di tengah perekaman), berkas `.daq` yang telah selesai **ditambahkan ke proyek yang terbuka secara otomatis** — berkas tersebut muncul dalam daftar berkas proyek tanpa perlu ditambahkan secara manual, siap digunakan sebagai data radiasi turun untuk [pemrosesan reflektansi](README.md).
* Indikator merah `REC` akan muncul di baris langsung pada jendela pengaturan selama perekaman.

Untuk angka irradiansi kuantitatif, ambil rata-rata data setidaknya selama 15 detik — ini merupakan karakteristik instrumen, bukan cacat.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Tata letak multi-sensor dan penyimpanan proyek

* Gabungkan beberapa sensor dalam satu grafik (sumbu bersama), pertahankan grafik terpisah (tata letak kisi otomatis), pindahkan sensor antar grafik, seret dan atur ulang baris/kotak, serta sembunyikan sensor individu dengan tombol mata.
* Per proyek, Chloros menyimpan: nama perangkat, warna grafik, ukuran grafik, mode tampilan, dan pengaturan setiap sensor (waktu integrasi, rata-rata bingkai, status AE, pemilihan penutup).
* **Membuka kembali proyek akan menghubungkan kembali sensor-sensornya secara otomatis** berdasarkan alamat — port COM untuk DAQ-U, perangkat BLE untuk DAQ-M, nama host mDNS untuk DAQ-E (tetap terdeteksi meskipun IP unit berubah) — serta menerapkan kembali profil cap yang disimpan, rata-rata bingkai, status AE, dan waktu integrasi manual masing-masing sensor.

***

## Pemasangan kamera (DLS)

Tidak ada yang perlu dipasangkan. Berbeda dengan alur kerja DLS drone yang mengikat sensor cahaya ke kamera sejak awal, Chloros mencocokkan data DAQ dengan citra pada tahap selanjutnya: saat impor/pemrosesan, pembacaan `.daq` diinterpolasi ke cap waktu eksposur setiap pengambilan gambar. Rekam dengan sensor apa pun yang terhubung (`.daq` akan masuk ke proyek secara otomatis), dan pemrosesan reflektansi akan menemukan pembacaan yang tepat berdasarkan waktu — lihat [Sensor Cahaya DAQ](README.md) untuk mengetahui bagaimana data downwelling digunakan. Untuk pengambilan gambar LATTICE **langsung**, pilih sensor pada menu tarik-turun**Light Sensor** di tab Cameras; setiap perekaman kemudian menyimpan pembacaan yang digunakannya, yang hanya disesuaikan dengan kamera pada perekaman tersebut (lihat [Alur Kerja Reflektansi](reflectance.md)).</version\>

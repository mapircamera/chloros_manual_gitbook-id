# Tab DAQ di Chloros

Tab DAQ — yang diberi label **Sensor Cahaya** di bilah samping Chloros — adalah antarmuka kontrol langsung untuk [sensor cahaya DAQ-U, DAQ-M, dan DAQ-E](README.md): hubungkan sensor melalui protokol apa pun, pantau spektrum yang telah dikalibrasi secara real-time, hitung reflektansi secara langsung dari sepasang sensor, dan rekam berkas `.daq` langsung ke dalam proyek Anda.

Tab ini akan tersedia setelah backend Chloros selesai dijalankan. Grafik pada tab ini disuplai oleh layanan DAQ Chloros melalui koneksi langsung yang akan terhubung kembali secara otomatis (waktu tunggu 2–10 detik) jika terputus; saat layanan tidak dapat dijangkau, baris Status sensor akan menampilkan **No Server**.

Tata letaknya berupa **bilah samping sensor**(satu baris per sensor yang terhubung) ditambah**area grafik** (satu ubin grafik per sensor atau grup).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Menghubungkan sensor

Klik **Connect Sensor** di bagian atas sidebar. Kotak dialog koneksi akan terbuka di area utama (atau sebagai overlay saat menambahkan sensor lain — tombol Cancel akan muncul dalam kasus tersebut).

| Kontrol | Perilaku |
| --- | --- |
| **Jenis Perangkat** | `DAQ-U (USB)` (default), `DAQ-M (Bluetooth)`, atau `DAQ-E (Ethernet)`. Mengubah pilihan akan memulai ulang pemindaian untuk protokol yang baru dipilih. |
| **Port / Perangkat BLE / Nama Host / IP** | Menampilkan daftar perangkat yang terdeteksi sebagai `device - description`; entri pertama yang dikenali sebagai sensor akan dipilih secara otomatis. Selama pemindaian, ditampilkan `Scanning...` (USB), `Scanning (N)...` dengan hitung mundur 8 detik (BLE), atau `Discovering ethernet sensors (N)...` dengan hitung mundur 5 detik (Ethernet). Hasil kosong ditampilkan sebagai `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Refresh** | Segera memindai ulang transportasi yang dipilih (dinonaktifkan saat pemindaian BLE/Ethernet sedang berlangsung). |
| **Connect** | Diaktifkan setelah perangkat dipilih; mengganti label menjadi `Connecting...` saat koneksi sedang dibuat. |

Pencarian hanya berjalan **saat dialog sambungan muncul di layar**, dan diulang setiap 15 detik hanya untuk transportasi yang dipilih — sekadar membuka tab tidak akan memicu pemindaian. Jika terjadi kegagalan, dialog akan menampilkan: *&quot;Sambungan gagal. Coba cabut dan pasang kembali sensor, lalu klik Sambungkan lagi.&quot;*

Bilah samping akan terbuka secara otomatis saat sensor pertama Anda terhubung.

{% hint style="info" %}
**DAQ-E tidak muncul?** DAQ-E tidak memiliki LED status — periksa indikator PoE/link pada port switch atau injektor tempat perangkat tersebut terpasang, dan tunggu beberapa detik setelah dinyalakan agar perangkat dapat melakukan booting. Perangkat Chloros harus berada di domain siaran yang sama (mDNS tidak menembus router). Pada Windows, terima prompt firewall Defender saat pertama kali Chloros mengikat soket multicast-nya (mDNS UDP 5353, data DAQ-E UDP 5002, PTP UDP 319/320). Dua unit DAQ-E pada satu LAN akan terdeteksi secara terpisah, masing-masing dengan nama host `daq-e-<id>.local`-nya sendiri.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Jenis Perangkat menawarkan DAQ-U (USB), DAQ-M (Bluetooth), dan DAQ-E (Ethernet)</figcaption></figure>***

## Bilah samping sensor

Setiap sensor yang terhubung mendapatkan satu baris (ditambah satu baris per grup Ambient+Object). Baris-baris ini dapat diurutkan ulang dengan cara diseret, dan urutannya juga akan mengatur ulang susunan ubin grafik. Klik sebuah baris untuk menjadikan sensor/grup tersebut sebagai grafik aktif dalam tampilan daftar.

| Elemen | Arti |
| --- | --- |
| Garis batas kiri berwarna | Warna grafik sensor. |
| Lencana transportasi | `DAQ-U` / `DAQ-M` / `DAQ-E`, atau lencana hijau `REF` untuk grup reflektansi Ambient+Objek. |
| Nama perangkat | Secara default menggunakan nomor seri sensor (identitas tetapnya untuk kalibrasi, nama file `.daq`, dan pencocokan impor); nama kustom akan tetap berlaku per proyek. |
| Tanda **Telah Dikalibrasi** (hijau) | Ditampilkan saat paket kalibrasi pabrik sensor dimuat, yaitu spektrumnya merupakan nilai W/m²/nm yang sebenarnya. |
| Tanda **Pembaruan Tersedia** (kuning, hanya DAQ-E) | Firmware yang sedang berjalan lebih lama daripada gambar yang disertakan dalam build Chloros ini. Selama pembaruan, ini menampilkan kemajuan secara langsung (`Flashing… N%`, `Restarting sensor…`, lalu `Updated X → Y` atau `Failed`). |
| Mata | Mengaktifkan atau menonaktifkan tampilan sensor ini pada grafiknya. |
| Roda Gigi | Membuka jendela pengaturan per-sensor (di bawah). |
| ✕ (merah) | Memutuskan koneksi sensor, atau menghapus grup Ambient+Object. |

Di atas baris-baris tersebut terdapat dua tombol:

* **Hubungkan Sensor** — membuka dialog koneksi (berubah menjadi `Connecting...` saat sedang diproses).
* **Rekam Semua / Hentikan Semua**— memulai atau menghentikan perekaman `.daq` pada**setiap**sensor yang terhubung. Membutuhkan setidaknya satu sensor**dan proyek yang terbuka** (tips alat: &quot;Buka proyek untuk merekam&quot;); tombol ini berubah menjadi merah saat perekaman sedang berjalan.

Status kosong menampilkan &quot;Tidak ada sensor yang terhubung&quot;.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Pengaturan per-sensor (jendela pengaturan berbentuk roda gigi)

Buka dengan mengklik ikon roda gigi pada baris sensor. Isi dalam urutan:

* **Baris Info** — Jenis Perangkat (DAQ-U/M/E), Koneksi (`Serial (USB)` / `Bluetooth` / `Ethernet`), Port (port COM, alamat BLE, atau host), dan Serial.
* **Laporan Kalibrasi: Unduh** — mengunduh sertifikat kalibrasi yang dapat dilacak ke NIST (PDF) dari unit ini dan membukanya di pembaca PDF Anda. Tersedia setelah nomor seri diketahui; sertifikat disimpan dalam cache saat koneksi pertama kali.
* **Nama Perangkat** — klik ikon pensil untuk mengganti nama; pengaturan ini berlaku per proyek.
* **Warna Garis Grafik** — palet warna; pengaturan ini berlaku per proyek.
* **Waktu Integrasi (ms)**— penggeser + angka,**1–500 ms**, default**32 ms**. Dinonaktifkan saat AE aktif.
* **Rata-rata Frame**— bilah geser + angka,**1–50 frame**, default**20**.
* **AE: AKTIF/NONAKTIF**— tombol pengaturan eksposur otomatis;**default AKTIF** saat terhubung. Matikan untuk mengatur waktu integrasi secara manual.
* **Hentikan Streaming / Mulai Streaming** — jeda atau lanjutkan siaran langsung.
* **Rekam / Hentikan Perekaman** — perekaman `.daq` per sensor (membutuhkan proyek yang terbuka).
* **Cap** — profil koreksi cap (bagian berikutnya).
* **Baris info langsung** — Waktu Integrasi (ms), FPS, Sampel, Perekaman (merah `REC` atau `Off`), dan Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Khusus DAQ-E: baris jaringan, firmware, dan PTP

* **Nama Host / IP** — alamat saat ini dari unit tersebut.
* **Firmware** — versi firmware saat ini, ditambah sel tindakan:<version\>

tombol</version\>

**Perbarui ke \<version\>** muncul ketika build Chloros ini menyertakan gambar firmware DAQ-E yang lebih baru. Pembaruan dilakukan melalui jaringan dalam waktu sekitar 30 detik; sensor akan restart dan terhubung kembali secara otomatis, dan jika transfer terputus, firmware saat ini tetap utuh. Kemajuan ditampilkan secara langsung (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), dan sel tersebut menampilkan `Up to date` saat versi terbaru.
* **Sinkronisasi PTP** — status PTP secara real-time (kembali ke `unknown`). Firmware DAQ-E v1.2.0+ berpartisipasi dalam IEEE 1588 PTPv2 sebagai jam yang hanya berfungsi sebagai slave; backend host Chloros adalah grandmaster PTP, dan setiap DAQ-E serta kamera LATTICE di LAN menjadi slave-nya di domain 0, menjaga cap waktu tetap dalam kisaran sekitar 1 ms.

Untuk grup Ambient+Object, jendela modal gear hanya menampilkan sensor sumber grup, Nama Perangkat, dan Warna Garis Grafik.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Pemilihan tutup

Menu tarik-turun **Cap** memberi tahu Chloros tutup fisik mana yang dipasang di atas difuser sensor, dan menerapkan profil koreksi yang diukur di pabrik dari tutup tersebut ke setiap spektrum. Pilihan bergantung pada model:

| Model | Pilihan tutup |
| --- | --- |
| DAQ-U | Tidak ada (sensor telanjang), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (korektor kosinus) |
| DAQ-M | Tidak ada (sensor telanjang), Sunshine (korektor kosinus) |
| DAQ-E | Tidak ada (sensor telanjang), FOV 15°, FOV 45°, FOV 90°, Sunshine (korektor kosinus) |

**Pengaturan default untuk setiap model adalah Sunshine (korektor kosinus)** — MAPIR mengirimkan setiap unit DAQ dengan penutup Sunshine terpasang, dan ini merupakan konfigurasi standar untuk penggunaan di luar ruangan: pandangan hemisferis 180° dengan kesalahan kosinus ≤ ±4 % hingga 60° dan ≤ ±4,5 % hingga 70° (tidak disarankan di bawah ketinggian matahari ~15°), dengan peredaman sesuai desain (~12×). Pilihan Anda akan tetap berlaku dalam proyek.

{% hint style="warning" %}
**Pilihan penutup harus sesuai dengan penutup fisik.**Baik sensor maupun perangkat lunak tidak dapat mendeteksi penutup mana yang terpasang. Pemilihan ini memengaruhi baik koreksi real-time maupun cap yang ditulis ke setiap berkas `.daq` — dengan atenuasi ~12× pada penutup Sunshine, perubahan penutup yang tidak dilaporkan akan mengoreksi spektrum secara salah dengan faktor sekitar itu. (Melepas dan memasang kembali penutup yang sama menghasilkan pengulangan hingga sekitar 1,5 %.) Pilih**None (sensor telanjang)** hanya jika penutup telah dilepas secara fisik; pada DAQ-E, &quot;None&quot; tetap menerapkan profil geometri pabrik untuk difuser kaca yang tersembunyi — ini bukan operasi kosong — dan DAQ-E tanpa penutup merupakan konfigurasi bangku kerja, bukan konfigurasi lapangan yang didukung.
{% endhint %}

{% hint style="info" %}
Pembaruan dari manual sebelumnya: tombol pengalih &quot;Sunshine Diffuser Terpasang&quot; di sisi browser dari versi 1.1.0 telah dihapus. Pengelolaan penutup kini menggunakan profil penutup per sensor ini, yang diterapkan di sisi server.
{% endhint %}

***

## Area grafik

Sebuah bilah atas yang melekat menampung **tombol pengalih tampilan daftar ⇄ kisi**dan penggeser**Zoom Grafik** (ukuran ubin 200–2000 piksel). Tampilan akan otomatis beralih ke mode kisi jika terdapat lebih dari satu grup grafik, dan kembali ke mode daftar jika hanya ada satu atau kurang. Mode tampilan dan ukuran grafik disimpan per proyek.**Grafik spektrum** untuk setiap sensor menampilkan:

* **Sumbu X** — Panjang gelombang (nm). Grid sensor berkisar 340–1010 nm dengan selang 5 nm (135 titik), diinterpolasi menjadi 1 nm untuk tampilan.
* **Sumbu Y** — Daya (W/m²), dengan awalan SI otomatis (m/µ/n) yang dipilih berdasarkan nilai puncak. Spektrum dikalibrasi secara radiometrik menjadi irradiance spektral (W/m²/nm) pada ketiga sistem transportasi.
* Isi spektrum pelangi di bawah satu jejak; beberapa sensor pada satu grafik ditumpangkan sebagai garis berwarna dengan isi yang diredupkan.
* **Arahkan kursor**— kursor vertikal dengan panjang gelombang dan nilai per sensor;**seret** untuk memperbesar (tombol memperkecil akan muncul saat diperbesar).
* Tombol **+** (hanya pada tampilan kisi) untuk menambahkan sensor ke grafik ini atau membuat grup (di bawah).
* Nama perangkat ditempatkan di tengah bagian atas, dan muncul spinner hingga frame pertama tiba.

**Saturasi** tidak ditandai pada grafik itu sendiri: sensor yang jenuh menampilkan teks status merah `SATURATED` dan baris merah `Saturated: Yes` di tabel data langsung. Kurangi waktu integrasi atau aktifkan kembali AE untuk menghilangkannya.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Tabel data langsung (tampilan daftar)

Di bawah grafik dalam tampilan daftar, diperbarui setiap 500 ms:

* **Semua model**: Sampel Warna Cahaya (sRGB dari CIE XYZ), Jenuh (Ya/Tidak), CIE 1931 X/Y/Z, Kromatisitas x/y, CIE u′/v′, CCT (K), CRI (Ra), Panjang Gelombang Dominan (nm), Panjang Gelombang Puncak (nm), Kemurnian Eksitasi, Duv, CIE L\*/a\*/b\*, dan Munsell H/V/C.
* **Hanya sensor yang telah dikalibrasi**(DAQ-U / DAQ-M / DAQ-E mana pun setelah paket kalibrasi pabriknya dimuat — tanda**Terkalibrasi** berwarna hijau pada baris sensor menjadi penanda): Daya Total (W/m²), Lux Fotopik (lx), Lux Skotopik (lx), Rasio S/P, PPFD serta PPFD Red/Green/Blue (µmol/m²/s), serta irradiansi opik — S-cone, Melanopic, Rhodopic, M-cone, L-cone (semua dalam satuan W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Kelompok Reflektansi (Ambient + Objek)

Dua sensor yang terhubung dapat digabungkan menjadi tampilan reflektansi real-time — tanpa melibatkan kamera:

1. Dalam tampilan kisi, klik **+**pada ubin grafik dan pilih**Gabungkan Lingkungan + Objek**.
2. Pilih sensor **Sumber Cahaya Lingkungan**dan sensor**Pemindai Objek**(dua sensor yang berbeda), lalu**Buat**.

Chloros menghitung R(λ) = objek(λ) / ambient(λ) per panjang gelombang dari dua aliran data langsung (0 jika ambient ≤ 0). Label kelompok mengikuti kelas kalibrasi sensor:

* Kedua sensor telah dikalibrasi (bundel dimuat) → **&quot;Reflektansi Tampak&quot;**.
* Salah satu sensor belum dikalibrasi → **&quot;Reflektansi Relatif&quot;**.

Grup ini muncul sebagai baris berwarna hijau `REF` di bilah samping dan memiliki grafik tersendiri (isi warna pelangi, nilai saat mengarahkan kursor hingga 4 desimal, zoom dengan menyeret).

Menu **+**juga menawarkan opsi**Tambahkan Sensor Baru** dengan tiga pilihan penempatan: *Gabungkan Sensor Baru* (gabungkan ke grafik ini), *Pindahkan Sensor yang Sudah Ada ke Sini*, atau *Lihat Sensor Baru* (grafik tersendiri).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Tabel indeks vegetasi

Dalam tampilan daftar, tabel indeks vegetasi terletak di bawah grafik kelompok reflektansi, yang dihitung dari reflektansi real-time pada pusat pita **biru 450 / hijau 550 / merah 670 / NIR 800 nm** (nilai hingga 4 desimal, `---` jika tidak dapat dihitung; arahkan kursor ke nama indeks untuk melihat nama lengkapnya):

* **Selalu ditampilkan** (tidak bergantung pada skala, kombinasi sensor apa pun): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Hanya jika kedua sensor telah dikalibrasi daya** (kedua paket telah dimuat): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

.***

## Merekam berkas `.daq`

* Perekaman memerlukan **proyek yang terbuka** — jika tidak, baik tombol &quot;Record All&quot; (di bilah samping) maupun tombol &quot;Record&quot; per sensor akan dinonaktifkan.
* Berkas disimpan ke **`<project folder>/light_sensor/`**; nama berkas berisi ID sensor dan cap waktu, serta nama perangkat disimpan bersama rekaman.
* Saat perekaman dihentikan (Stop, Stop All, atau terputus saat perekaman berlangsung), berkas `.daq` yang telah selesai akan **ditambahkan ke proyek yang terbuka secara otomatis** — berkas tersebut muncul dalam daftar berkas proyek tanpa perlu ditambahkan secara manual, siap digunakan sebagai data downwelling untuk [pemrosesan reflektansi](README.md).
* Indikator merah `REC` muncul di baris langsung jendela pengaturan selama perekaman.

Untuk angka irradiansi kuantitatif, ambil rata-rata data setidaknya selama 15 detik — ini merupakan karakteristik instrumen, bukan cacat.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Tata letak multi-sensor dan persisten proyek

* Gabungkan beberapa sensor dalam satu grafik (sumbu bersama), pertahankan grafik terpisah (tata letak kisi otomatis), pindahkan sensor antar grafik, seret dan atur ulang baris/kotak, serta sembunyikan sensor individu dengan tombol mata.
* Per proyek, Chloros menyimpan: nama perangkat, warna grafik, ukuran grafik, mode tampilan, dan pengaturan setiap sensor (waktu integrasi, rata-rata bingkai, status AE, pemilihan batas).
* **Membuka kembali sebuah proyek akan menghubungkan kembali sensor-sensornya secara otomatis** berdasarkan alamat — port COM untuk DAQ-U, perangkat BLE untuk DAQ-M, nama host mDNS untuk DAQ-E (tetap terdeteksi meskipun alamat IP unit berubah) — serta menerapkan kembali profil cap yang disimpan, rata-rata bingkai, status AE, dan waktu integrasi manual masing-masing sensor.***

## Pemasangan kamera (DLS)

Tidak ada yang perlu dipasangkan. Berbeda dengan alur kerja DLS drone yang mengikat sensor cahaya ke kamera sejak awal, Chloros mencocokkan data DAQ dengan citra pada tahap selanjutnya: saat impor/pemrosesan, pembacaan `.daq` diinterpolasi ke cap waktu eksposur setiap pengambilan gambar. Rekam dengan sensor apa pun yang terhubung (`.daq` akan masuk ke proyek secara otomatis), dan pemrosesan reflektansi akan menemukan pembacaan yang tepat berdasarkan waktu — lihat [Sensor Cahaya DAQ](README.md) untuk mengetahui bagaimana data downwelling digunakan.</version\>

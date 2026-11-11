# Perekaman &amp; Format .daq

Berkas `.daq` adalah format perekaman sensor cahaya dari Chloros

: sebuah **basis data SQLite** yang berisi bingkai spektral yang telah dikalibrasi dari satu sensor DAQ. Rekam satu berkas selama sesi pengambilan data, dan pipa pengolahan reflektansi nantinya dapat membagi setiap gambar dengan irradiansi turun yang diukur pada saat yang tepat tersebut.

## Isi berkas .daq

| Properti | Nilai |
| --- | --- |
| Kontainer | Basis data SQLite, satu berkas per sensor per perekaman |
| Nama berkas | Mencakup **ID sensor**dan**cap waktu**, misalnya `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum per bingkai | 135 titik, 340–1010 nm dengan selang 5 nm, ditambah tristimulus CIE XYZ |
| Satuan | Irradian spektral yang dikalibrasi, **W/m²/nm** (paket kalibrasi pabrik + profil penutup diterapkan) |
| Metadata yang dicantumkan | ID sensor (kunci untuk mengambil kalibrasi pabrik unit tersebut) dan profil penutup yang berlaku — lihat [Profil Penutup &amp; Rentang yang Dikalibrasi](caps-and-range.md) |

Formatnya identik di seluruh DAQ-U, DAQ-M, DAQ-E, dan DAQ-E-S, sehingga pemrosesan lanjutan tidak mempermasalahkan perangkat mana yang merekamnya. Rekaman DAQ-E-S juga menyimpan data kemiringan, rotasi, dan sudut kemiringan unit di samping setiap spektrum.

**Rekaman tetap utuh meskipun terjadi pemadaman listrik (1.3.0).**Rekaman langsung ditulis dengan jurnal dan sinkronisasi disk penuh pada setiap commit, sehingga pemadaman listrik atau proses yang dihentikan hanya mengakibatkan hilangnya data sekitar 3 detik terakhir, bukan berkasnya. Perekaman yang terhenti di tengah proses penulisan dapat meninggalkan berkas `<name>.daq-journal` di samping berkas `.daq`:**simpan dan salin kedua berkas tersebut bersama-sama.** Chloros

akan memutar balik jurnal saat pertama kali membaca berkas tersebut. Disk yang sibuk akan menunda proses penulisan file, bukan pembacaan sensor.

**DAQ-E yang berhenti mengirim data akan dihubungkan kembali (1.3.0).** Ketika aliran data dari DAQ-E terhenti (sakelar daya dimatikan dan dinyalakan kembali, atau koneksi terputus), backend akan menghubungkannya kembali dengan pengaturan sebelumnya, dan perekaman di DAQ-E tersebut akan dilanjutkan dalam berkas `.daq` yang sama dengan celah yang ditandai dalam berkas.

Perekaman yang dikalibrasi memerlukan paket kalibrasi pabrik dari sensor. Untuk DAQ-U dan DAQ-M, backend mengambil paket tersebut dari cloud MAPIR

berdasarkan ID sensor (perekaman akan ditolak jika tidak dapat diambil); unit DAQ-E dikecualikan karena menyimpan kalibrasinya di perangkat.

## Perekaman dari GUI

Perekaman di GUI memerlukan **proyek yang terbuka** (tombol Rekam akan dinonaktifkan jika tidak):

* **Rekam Semua / Hentikan Semua** — bagian atas bilah samping Sensor Cahaya; memulai atau menghentikan perekaman `.daq` pada setiap sensor yang terhubung secara bersamaan.
* **Rekam / Hentikan Perekaman** — per sensor, di jendela pengaturan roda gigi. Indikator &quot;REC&quot; berwarna merah akan muncul di baris info langsung sensor selama perekaman berlangsung.

File disimpan ke `<project>/light_sensor/`, dan saat perekaman dihentikan — baik melalui tombol Hentikan, Hentikan Semua, atau pemutusan koneksi sensor perekaman — file `.daq` yang telah selesai akan **secara otomatis ditambahkan ke proyek yang terbuka**. File tersebut muncul dalam daftar file proyek tanpa perlu langkah penambahan manual, dan sudah siap untuk pemrosesan reflektansi.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Perekaman dari CLI



CLI

melakukan perekaman melalui kumpulan sensor backend (backend harus sedang berjalan — perintah-perintah ini adalah klien HTTP

ringan):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Mendapatkan nilai `--sensor-id` dari `chloros-cli daq pool-list`. Dua nilai default yang perlu diketahui:

| Opsi | Default |
| --- | --- |
| `--duration` | `0` — merekam hingga `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` di sistem berkas **backend**, bukan di sistem berkas CLI

|

Perbedaan direktori keluaran ini penting ketika CLI

menargetkan backend di mesin lain: berkas tersebut akan disimpan di tempat backend dijalankan. Jika backend berada di komputer yang sama, `-o` relatif (seperti `./out` di atas) akan disesuaikan dengan folder tempat Anda menjalankan perintah tersebut.

## Perekaman dari Python

`DAQSensorSession` (yang dikembalikan oleh `chloros_sdk.connect_daq_sensor()`) menampilkan rekaman terpusat yang sama: `record_start(output_dir=None, device_name=None)` mengembalikan jalur file, sedangkan `record_stop()` mengembalikan `{path, rows}`. Lihat [Referensi SDK

](../reference/sdk-reference.md) untuk API

sesi lengkap. Kelas perangkat keras langsung dalam SDK

(hanya untuk instalasi desktop) secara default menulis rekaman ke `~/Documents/DAQ/`; untuk rilis resmi, jalur terpusat di atas adalah metode yang didukung.

## Menggunakan berkas .daq pada saat pemrosesan

Untuk menghasilkan nilai reflektansi dari citra, Chloros

memerlukan irradiansi turun yang disesuaikan dengan setiap eksposur:

* **Simpan berkas `.daq` bersama citra.**Pada saat pemrosesan, alur kerja akan menentukan**intensitas radiasi ke bawah yang sesuai dengan cap waktu** secara otomatis dari file `.daq` yang direkam (model DAQ apa pun) — atau dari file `.csv` asli DAQ-M — yang terdapat bersama gambar-gambar tersebut. Rekaman GUI secara otomatis memenuhi persyaratan ini, karena rekaman tersebut ditambahkan ke proyek begitu proses perekaman berhenti.
* **Kalibrasi diambil sesuai permintaan.** Jika paket kalibrasi pabrik per kamera atau per DAQ belum disimpan dalam cache lokal, Chloros

akan mengambilnya secara otomatis dari cloud MAPIR

pada penggunaan pertama (diperlukan koneksi internet sekali; disimpan dalam cache sebagai berkas `~/.chloros/`).
* **Perekaman langsung menulis berkas sidecar-nya sendiri.** Untuk setiap bingkai reflektansi yang direkam secara langsung, pembacaan DAQ yang sebenarnya digunakan disimpan sebagai berkas `.daq` di folder `daq/` perekaman tersebut, sehingga perekaman tersebut dapat diproses ulang nanti tanpa rekaman aslinya.
* **Rekaman dari perekam `chloros_scripts` (1.3.0).**Sebuah berkas `.daq` yang dibuat oleh perekam umum [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) terdaftar sebagai model DAQ yang membuatnya, dan batasnya dapat diatur di**Pengaturan Proyek → Sensor Cahaya DAQ** seperti perekaman lainnya. Perekam tersebut mencatat hitungan sensor mentah; Chloros

mengambil kalibrasi pabrik sensor melalui serial dan menerapkannya.
* **Kepada siapa pembacaan tersebut (1.3.0).**Di samping setiap penangkapan `.daq`, `.daq.cameras.json` menyebutkan nama kamera yang bingkai-bingkainya dijelaskan oleh pembacaan tersebut, dan pemrosesan mencocokkannya**hanya dengan gambar dari kamera-kamera tersebut**. Oleh karena itu, kamera mandiri yang tidak memiliki sensor cahaya sendiri tidak lagi meminjam pembacaan dari kamera lain — hubungkan sensor ke kamera tersebut, atau rekam pembacaan sensor tersebut. Perekaman yang dibuat dengan DAQ itu sendiri (GUI Record, `daq pool-record`), dan setiap `.daq` tanpa berkas tersebut (termasuk tangkapan lama), tetap berfungsi untuk semua kamera.

## Mengambil kembali data irradiansi

Memproses sebuah proyek juga mengekspor setiap rekaman sensor cahaya yang dimilikinya, ke dalam folder
`Light Sensor/` di samping produk citra. Hal ini **tidak** memerlukan citra:
rekaman sensor cahaya yang direkam sendiri merupakan penangkapan yang lengkap, dan folder yang hanya berisi berkas `.daq`
merupakan masukan yang valid. Sebuah proyek yang hanya berisi rekaman sensor cahaya akan mencantumkan dan
memprosesnya baik dari aplikasi maupun CLI

, dan daftar tersebut tetap ada setelah proses selesai. Proses tersebut
melaporkan berapa banyak produk sensor cahaya yang telah ditulis.

| Produk | Apa itu |
| --- | --- |
| `<name>_calibrated.daq` | Arsip yang dapat diproses ulang dengan skema yang sama seperti rekaman langsung, kini menyertakan paket kalibrasi yang menghasilkannya. Mengimpor ulang arsip ini **tidak** akan mengkalibrasinya untuk kedua kalinya. |
| `<name>_calibrated.csv` | Irradian spektral dalam W/m²/nm pada kisi panjang gelombang sensor itu sendiri, satu baris per pembacaan, ditambah kolom fotometrik: daya total, lux fotopik dan skotopik, PPFD dengan pembagian biru/hijau/merah, serta panjang gelombang puncak. |

DAQ-U atau DAQ-M yang paket kalibrasinya tidak dapat diambil — Anda sedang offline, atau
sensor tersebut tidak memiliki file kalibrasi — akan **dilewati dengan alasan**, dan tidak pernah disimpan
sebagai file &quot;terkalibrasi&quot; yang berisi hitungan mentah. Hubungkan ke internet dan jalankan kembali. DAQ-E
memiliki kalibrasi sendiri, sehingga hanya memerlukan ini ketika unit tidak terhubung dan
tidak ada yang disimpan dalam cache lokal.

### DAQ-A: hitungan mentah, dan mengapa itu adalah jawaban yang tepat

**DAQ-A** muncul sebelum sistem paket kalibrasi per-serial dan tidak memiliki paket yang
dapat diambil. Itu bukanlah kelalaian: DAQ-A dikalibrasi di lapangan menggunakan
target reflektansi, dan kalibrasi berbasis target hanya memerlukan respons *relatif*
sensor — yang persis sama dengan hitungan mentahnya. Chloros

saat ini mengkalibrasinya dengan cara tersebut.

Jadi, rekaman DAQ-A dapat diekspor, tetapi dengan nama yang berbeda:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, bukan `_calibrated` — nama berkas yang berbeda, bukan penanda di dalam berkas,
karena klaim tersebut harus tetap utuh saat berkas dikirim melalui email hanya dengan nama berkasnya saja. Header `.csv`
menunjukkan `raw spectral sensor counts (NOT irradiance)` dan memperingatkan bahwa nilai-nilai tersebut
dapat dibandingkan **di dalam** file itu sendiri, bukan antar sensor. Kolom-kolom yang hanya memiliki arti
untuk irradiansi nyata — daya total, lux, PPFD — dibiarkan kosong daripada
dihitung dari jumlah hitungan.

Rekaman DAQ-A-SD yang lebih lama (skema v1.01 / v1.02) hanya mencatat waktu penulisan berkas, bukan
cap waktu per pembacaan. Chloros

tidak akan mencocokkan gambar dengan data tersebut — mengaitkan sebuah bingkai dengan
waktu penulisan akan salah meskipun tampaknya tidak ada yang salah — tetapi ekspor membacanya dengan baik dan
berkas CSV

mencantumkan jam mana yang digunakan.

Untuk penjelasan lengkap mengenai reflektansi — sensor tunggal dengan kamera, dan sensor ganda (lingkungan/objek) — lihat [Alur Kerja Reflektansi](reflectance.md).

# Perekaman &amp; Format .daq

Berkas `.daq` merupakan format perekaman sensor cahaya dari Chloros: sebuah **basis data SQLite** yang berisi bingkai spektral yang telah dikalibrasi dari satu sensor DAQ. Rekam satu berkas selama sesi perekaman, dan pipa kerja reflektansi kemudian dapat membagi setiap gambar dengan irradiansi turun yang diukur pada saat itu juga.

## Isi berkas .daq

| Properti | Nilai |
| --- | --- |
| Kontainer | Basis data SQLite, satu berkas per sensor per perekaman |
| Nama berkas | Mencakup **ID sensor**dan**cap waktu**, misalnya `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum per bingkai | 135 titik, 340–1010 nm dengan selang 5 nm, ditambah tristimulus CIE XYZ |
| Satuan | Irradian spektral yang dikalibrasi, **W/m²/nm** (paket kalibrasi pabrik + profil penutup diterapkan) |
| Metadata yang dicantumkan | ID sensor (kunci untuk mengambil kalibrasi pabrik unit tersebut) dan profil penutup yang berlaku — lihat [Profil Penutup &amp; Rentang yang Dikalibrasi](caps-and-range.md) |

Formatnya identik di seluruh DAQ-U, DAQ-M, dan DAQ-E, sehingga pemrosesan lanjutan tidak mempermasalahkan perangkat apa yang merekamnya.

Perekaman yang dikalibrasi memerlukan paket kalibrasi pabrik sensor. Untuk DAQ-U dan DAQ-M, backend mengambil paket tersebut dari cloud MAPIR berdasarkan ID sensor (perekaman akan ditolak jika tidak dapat dilakukan); unit DAQ-E dikecualikan karena menyimpan kalibrasinya di perangkat.

## Perekaman dari GUI

Perekaman di GUI memerlukan **proyek yang terbuka** (tombol Rekam akan dinonaktifkan jika tidak):

* **Rekam Semua / Hentikan Semua** — bagian atas bilah samping Sensor Cahaya; memulai atau menghentikan perekaman `.daq` pada setiap sensor yang terhubung sekaligus.
* **Rekam / Hentikan Perekaman** — per sensor, di jendela pengaturan roda gigi. Indikator &quot;REC&quot; berwarna merah akan muncul di baris info langsung sensor selama perekaman berlangsung.

File disimpan ke `<project>/light_sensor/`, dan saat perekaman berhenti — baik melalui tombol Berhenti, Berhenti Semua, atau pemutusan koneksi sensor perekaman — file `.daq` yang telah selesai akan **secara otomatis ditambahkan ke proyek yang terbuka**. File tersebut muncul dalam daftar file proyek tanpa langkah penambahan manual, dan sudah siap untuk pemrosesan reflektansi.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Perekaman dari CLI

CLI merekam melalui kumpulan sensor backend (backend harus sedang berjalan — perintah-perintah ini adalah klien HTTP ringan):

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

Dapatkan nilai `--sensor-id` dari `chloros-cli daq pool-list`. Dua nilai default yang perlu diketahui:

| Opsi | Default |
| --- | --- |
| `--duration` | `0` — rekam hingga `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` pada sistem berkas **backend**, bukan pada sistem berkas CLI |

Perbedaan direktori keluaran menjadi penting ketika CLI menargetkan backend di mesin lain: file akan disimpan di tempat backend tersebut berjalan.

## Perekaman dari Python

`DAQSensorSession` (yang dikembalikan oleh `chloros_sdk.connect_daq_sensor()`) menampilkan perekaman yang sama dari pool: `record_start(output_dir=None, device_name=None)` mengembalikan jalur berkas, sedangkan `record_stop()` mengembalikan `{path, rows}`. Lihat [Referensi SDK](../reference/sdk-reference.md) untuk sesi lengkap API. Kelas perangkat keras langsung SDK (hanya untuk instalasi desktop) secara default menulis rekaman ke `~/Documents/DAQ/`; untuk rilis resmi, jalur terpusat di atas adalah rute yang didukung.

## Menggunakan berkas .daq pada saat pemrosesan

Untuk menghasilkan reflektansi dari citra, Chloros memerlukan irradiansi turun yang disesuaikan dengan setiap eksposur:

* **Simpan berkas `.daq` bersama citra tersebut.**Pada saat pemrosesan, alur kerja akan secara otomatis menentukan**intensitas radiasi ke bawah yang sesuai dengan cap waktu** dari rekaman `.daq` (model DAQ apa pun) — atau dari berkas `.csv` — yang ditemukan bersama gambar-gambar tersebut. Rekaman GUI secara otomatis memenuhi persyaratan ini, karena rekaman tersebut ditambahkan ke proyek begitu proses perekaman berhenti.
* **Kalibrasi diambil sesuai permintaan.** Jika paket kalibrasi pabrik per kamera atau per DAQ belum disimpan dalam cache lokal, Chloros akan mengambilnya secara otomatis dari cloud MAPIR pada penggunaan pertama (diperlukan koneksi internet sekali saja; disimpan dalam cache di bawah `~/.chloros/`).
* **Perekaman langsung menulis sidecar-nya sendiri.** Untuk setiap bingkai reflektansi yang direkam secara langsung, pembacaan DAQ yang sebenarnya digunakan disimpan sebagai sidecar `.daq` di samping citra, sehingga perekaman tersebut dapat diproses ulang nanti tanpa rekaman aslinya.

## Mengambil kembali data irradiance

Memproses sebuah proyek juga mengekspor setiap rekaman sensor cahaya yang dimilikinya, ke dalam
folder `Light Sensor/` di samping produk citra. Hal ini **tidak** memerlukan citra:
sensor cahaya yang diterbangkan sendiri merupakan perekaman lengkap, dan folder yang hanya berisi file `.daq`
merupakan input yang valid. Laporan pengoperasian mencatat berapa banyak produk sensor cahaya yang telah ditulis.

| Produk | Apa itu |
| --- | --- |
| `<name>_calibrated.daq` | Arsip yang dapat diproses ulang dengan skema yang sama seperti perekaman langsung, kini mencantumkan paket kalibrasi yang menghasilkannya. Mengimpornya kembali **tidak** akan mengkalibrasinya untuk kedua kalinya. |
| `<name>_calibrated.csv` | Irradiansi spektral dalam W/m²/nm pada kisi panjang gelombang sensor itu sendiri, satu baris per pembacaan, ditambah kolom fotometrik: daya total, lux fotopik dan skotopik, PPFD dengan pembagian biru/hijau/merah, serta panjang gelombang puncak. |

DAQ-U atau DAQ-M yang paket kalibrasinya tidak dapat diambil — Anda sedang offline, atau
sensor tersebut tidak memiliki file kalibrasi — akan **dilewati dengan alasan**, dan tidak pernah disimpan
sebagai file &quot;telah dikalibrasi&quot; yang berisi hitungan mentah. Hubungkan ke internet dan jalankan kembali. DAQ-E
memiliki kalibrasi sendiri, sehingga hanya memerlukan ini ketika unit tidak terhubung dan
tidak ada yang disimpan dalam cache lokal.

### DAQ-A: hitungan mentah, dan mengapa itu adalah jawaban yang tepat

**DAQ-A** muncul sebelum sistem paket kalibrasi per-serial dan tidak memiliki paket yang
dapat diambil. Itu bukanlah kelalaian: DAQ-A dikalibrasi di lapangan menggunakan
target reflektansi, dan kalibrasi berbasis target hanya memerlukan respons *relatif*
sensor — yang persis sama dengan hitungan mentahnya. Chloros melakukan kalibrasi dengan data tersebut hingga saat ini.

Jadi, rekaman DAQ-A dapat diekspor, tetapi dengan nama yang berbeda:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, bukan `_calibrated` — nama berkas yang berbeda, bukan penanda di dalam berkas,
karena klaim tersebut harus tetap berlaku meskipun berkas dikirim melalui email hanya dengan nama berkasnya saja. Header `.csv`
menunjukkan `raw spectral sensor counts (NOT irradiance)` dan memperingatkan bahwa nilai-nilai tersebut
dapat dibandingkan **di dalam** file tersebut, bukan antar sensor. Kolom-kolom yang hanya memiliki arti
untuk irradianse aktual — daya total, lux, PPFD — dibiarkan kosong daripada
dihitung dari jumlah hitungan.

Rekaman DAQ-A-SD yang lebih lama (skema v1.01 / v1.02) hanya mencatat waktu penulisan berkas, bukan
cap waktu per pembacaan. Chloros tidak akan mencocokkan gambar dengan data tersebut — mengaitkan sebuah frame dengan
waktu penulisan akan salah meskipun tampaknya tidak ada yang salah — tetapi ekspor dapat membacanya dengan baik dan
CSV menyatakan jam mana yang digunakan.

Untuk penjelasan lengkap mengenai reflektansi — sensor tunggal dengan kamera, dan sensor ganda (lingkungan/objek) — lihat [Alur Kerja Reflektansi](reflectance.md).

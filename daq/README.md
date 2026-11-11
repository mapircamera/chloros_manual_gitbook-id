# Sensor Cahaya DAQ

> **Mencari perangkat kerasnya?**Sensor-sensor itu sendiri — model, pemasangan, penutup, port, daya, dan aplikasi SCANNER — dijelaskan dalam**[panduan pengguna DAQ](https://mapir.gitbook.io/daq)**. Bab ini membahas penggunaannya dari Chloros. Sensor cahaya**DAQ** dari

MAPIR mengukur cahaya sekitar sebagai spektrum yang dikalibrasi secara radiometrik. Di Chloros, sensor ini memiliki dua peran:

* **Alat spektral mandiri** — grafik spektrum langsung, data kolorimetrik, dan rekaman `.daq`, semuanya dari [tab Sensor Cahaya](gui.md), [CLI](cli-quick-start.md), atau Python SDK.
* **Sumber iradiasi ke bawah untuk reflektansi** — selama pemrosesan, Chloros menginterpolasi pembacaan `.daq` Anda ke cap waktu eksposur setiap pengambilan gambar dan menggunakan cahaya yang mengarah ke bawah yang terukur untuk mengubah radian kamera menjadi reflektansi (`--reflectance-source daq`), tidak diperlukan panel dalam pemandangan untuk pita yang telah dikalibrasi.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tiga model, satu format data

| Model | Transport | Discovery |
| --- | --- | --- |
| **DAQ-U** | USB (serial) | pemindaian port serial |
| **DAQ-M** | Bluetooth Low Energy | pemindaian BLE berdasarkan nama |
| **DAQ-E** | Ethernet (IPv4, bertenaga PoE) | mDNS `_daq-e._tcp` (nama host `daq-e-<id>.local`) |**DAQ-E-S (baru di versi 1.3.0).** DAQ-E dengan diffuser permanen dan sensor kemiringan. Chloros mencantumkannya sebagai `DAQ-E-S` dan menjalankannya persis seperti DAQ-E, kecuali bahwa perangkat ini tidak memerlukan koreksi cap dan setiap pembacaan menyertakan data kemiringan, roll, dan pitch unit. Lihat [DAQ-E-S: difuser tetap dan kemiringan](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Ketiga perangkat tersebut menggunakan protokol kabel yang sama dan menghasilkan data yang identik:

* **Spektrum 135 titik dari 340 hingga 1010 nm dengan langkah 5 nm**, ditambah nilai tristimulus CIE XYZ, pada setiap bingkai.
* **Irradian spektral yang dikalibrasi secara radiometrik dalam W/m²/nm** — paket kalibrasi pabrik setiap unit (ditambah profil koreksi tutup aktifnya) diterapkan sebelum data sampai ke Anda.
* **Format perekaman `.daq`** yang sama (berupa berkas SQLite). Pemrosesan lanjutan akan identik terlepas dari transport mana yang menghasilkan berkas tersebut.

Stack transportasi (USB serial, BLE, mDNS/zeroconf) terintegrasi di dalam backend Chloros — tidak perlu menginstal apa pun untuk berkomunikasi dengan ketiga model tersebut melalui antarmuka grafis (GUI) atau perintah `pool-*` di antarmuka baris perintah (CLI).

***

## Rentang yang dikalibrasi: dilaporkan 340–1010 nm, dikalibrasi ~374–974 nm

Sensor melaporkan rentang penuh 340–1010 nm, tetapi gain radiometrik yang dapat dilacak ke NIST mencakup rentang sekitar **374–974 nm**. Chloros menolak pembagian reflektansi absolut untuk pita kamera mana pun yang memiliki kurang dari setengah bobot spektralnya di dalam rentang yang dikalibrasi tersebut; pita yang dilewati dilaporkan dengan alasan pelompatan `dls-uncalibrated-band-<nm>`.

Di antara SKU filter LATTICE yang tersedia, hanya **F988** yang terpengaruh:

Reflektansi F988 dikalibrasi menggunakan panel reflektansi di lokasi: pita tersebut berada di luar rentang kalibrasi sensor cahaya DAQ, sehingga Chloros menerapkan data panel terbaru yang Anda ambil dan mempertahankannya di antara pengukuran panel.

Jika penangkapan data F988 diproses dengan hanya data DAQ yang tersedia, Chloros akan menolak reflektansi berbasis DAQ untuk band tersebut dengan alasan pengabaian `dls-uncalibrated-band-988` — [alur kerja panel reflektansi](../calibration-targets.md) adalah jalur yang didukung untuk F988.

***

## ID Sensor

Setiap DAQ melaporkan ID sensor yang stabil. Bentuknya berbeda-beda tergantung model:

| Model | Bentuk ID | Contoh |
| --- | --- | --- |
| DAQ-U | 5-octet dengan tanda hubung | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktet dengan tanda hubung | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID sensor adalah:

* tercantum dalam setiap berkas `.daq` yang direkamnya,
* kunci yang digunakan Chloros untuk mengambil paket kalibrasi pabrik unit tersebut,
* nilai yang Anda berikan ke `--sensor-id` dalam perintah CLI `pool-*`, dan
* untuk DAQ-E, juga nama host mDNS-nya (`daq-e-def330.local`) — nilai yang diterima oleh `--eth-host`.

***

## Kalibrasi pabrik dan cloud

Setiap unit DAQ dikalibrasi secara individual di pabrik menggunakan rantai radiometrik yang dapat dilacak ke NIST, dan Chloros memuat paket kalibrasi setiap unit yang diidentifikasi berdasarkan ID sensornya. Laporan kalibrasi per unit (PDF) dapat diunduh dari pengaturan sensor di [tab Sensor Cahaya](gui.md).

{% hint style="warning" %}
**DAQ-U dan DAQ-M memerlukan akses ke cloud untuk kalibrasi.**Kedua model ini tidak menyimpan apa pun di perangkat: paket kalibrasi pabriknya disimpan di cloud MAPIR dan diambil berdasarkan ID sensor (lalu disimpan dalam cache lokal). Chloros memerlukan koneksi internet untuk mengirimkan data W/m²/nm yang telah dikalibrasi dari DAQ-U atau DAQ-M.**DAQ-E merupakan pengecualian** — kalibrasinya disimpan langsung di perangkat.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## Tempat penyimpanan rekaman

| Permukaan | Tujuan default `.daq` |
| --- | --- |
| GUI — Tab Sensor Cahaya | `<project folder>/light_sensor/` (rekaman yang sudah selesai ditambahkan ke proyek secara otomatis) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` di mesin yang menjalankan backend (sebuah jalur relatif `-o` akan mengacu pada folder tempat Anda menjalankan perintah tersebut jika backend berada di komputer yang sama) |

Setiap nama file `.daq` mencakup ID sensor dan cap waktu.

***

## Dalam bab ini

* [**Tab DAQ di Chloros**](gui.md) — panduan lengkap antarmuka pengguna grafis (GUI): menghubungkan setiap model, pengaturan per-sensor, grafik spektrum, data kolorimetrik real-time, reflektansi dua sensor, dan perekaman.
* [**Panduan Cepat CLI (pool-\*)**](cli-quick-start.md) — mengendalikan sensor DAQ dari `chloros-cli daq pool-*`, jalur baris perintah yang didukung.
* [**Profil Batas & Rentang yang Dikalibrasi**](caps-and-range.md) — batas apa saja yang ada per model, cara mendeklarasikannya, serta rentang spektral yang dikalibrasi secara terperinci.
* [**Perekaman & Format .daq**](recording.md) — format SQLite `.daq` dan alur kerja perekaman.
* [**Jaringan DAQ-E & Sinkronisasi Waktu**](ethernet-ptp.md) — mode transmisi DAQ-E dan sinkronisasi waktu PTP.
* [**Alur Kerja Reflektansi**](reflectance.md) — menggunakan data downwelling DAQ untuk menghasilkan reflektansi.
* Untuk dokumentasi lengkap tingkat flag, lihat [Referensi CLI](../reference/cli-reference.md) (bagian `chloros-cli daq`) dan [Referensi SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), yang keduanya ditulis agar dapat langsung digunakan oleh asisten AI.

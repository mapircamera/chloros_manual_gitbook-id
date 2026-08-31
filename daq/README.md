# Sensor Cahaya DAQ

> **Mencari informasi tentang perangkat kerasnya?**Sensor-sensor tersebut — model, pemasangan, penutup, port, catu daya, dan aplikasi SCANNER — dijelaskan dalam**[panduan pengguna DAQ](https://mapir.gitbook.io/daq)**. Bab ini membahas penggunaannya mulai dari Chloros.

Sensor cahaya **DAQ** pada MAPIR mengukur cahaya sekitar sebagai spektrum yang dikalibrasi secara radiometrik. Dalam Chloros, sensor ini memiliki dua fungsi:

* **Alat spektral mandiri** — grafik spektrum real-time, data kolorimetrik, dan rekaman `.daq`, semuanya dari [tab Sensor Cahaya](gui.md), [CLI](cli-quick-start.md), atau Python SDK.
* **Sumber irradiansi turun untuk reflektansi** — selama pemrosesan, Chloros menginterpolasi pembacaan `.daq` Anda ke setiapstempel waktu eksposur dan menggunakan cahaya turun yang diukur untuk mengubah radian kamera menjadi reflektansi (`--reflectance-source daq`), tidak diperlukan panel dalam adegan untuk pita yang dikalibrasi.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Tiga model, satu format data

| Model | Transport | Discovery |
| --- | --- | --- |
| **DAQ-U** | USB (serial) | pemindaian port serial |
| **DAQ-M** | Bluetooth Low Energy | pemindaian BLE berdasarkan nama |
| **DAQ-E** | Ethernet (IPv4, bertenaga PoE) | mDNS `_daq-e._tcp` (nama host `daq-e-<id>.local`) |

Ketiga perangkat tersebut menggunakan protokol kabel yang sama dan menghasilkan data yang identik:

* **Spektrum 135 titik dari 340 hingga 1010 nm dengan selang 5 nm**, ditambah nilai tristimulus CIE XYZ, pada setiap frame.
* **Iradian spektral yang dikalibrasi secara radiometrik dalam W/m²/nm** — paket kalibrasi pabrik setiap unit (ditambah profil koreksi tutup aktifnya) diterapkan sebelum data sampai ke Anda.
* **Format perekaman `.daq`** yang sama (berupa berkas SQLite). Pemrosesan lanjutan akan sama terlepas dari transportasi mana yang menghasilkan berkas tersebut.

Tumpukan transportasi (serial USB, BLE, mDNS/zeroconf) digabungkan di dalam backend Chloros — tidak perlu menginstal apa pun untuk berkomunikasi dengan ketiga model tersebut melalui antarmuka pengguna (GUI) atau perintah `pool-*` dari CLI.

***

## Rentang yang dikalibrasi: 340–1010 nm dilaporkan, ~374–974 nm dikalibrasi

Sensor melaporkan rentang penuh 340–1010 nm, tetapi gain radiometrik yang dapat dilacak ke NIST mencakup kisaran sekitar **374–974 nm**. Chloros menolak pembagian reflektansi absolut untuk pita kamera mana pun yang memiliki kurang dari setengah bobot spektralnya di dalam rentang yang dikalibrasi tersebut; pita yang dilewati dilaporkan dengan alasan pelompatan `dls-uncalibrated-band-<nm>`.

Di antara SKU filter LATTICE yang tersedia, hanya **F988** yang terpengaruh:

Reflektansi F988 dikalibrasi menggunakan panel reflektansi dalam adegan: pita tersebut berada di luar rentang yang dikalibrasi oleh sensor cahaya DAQ, sehingga Chloros menerapkan penangkapan panel terbaru Anda dan menyimpannya di antara pengamatan panel.

Jika penangkapan data F988 diproses dengan hanya data DAQ yang tersedia, Chloros menolak reflektansi berbasis DAQ untuk pita tersebut dengan alasan pengabaian `dls-uncalibrated-band-988` — [alur kerja panel reflektansi](../calibration-targets.md) adalah jalur yang didukung untuk F988.

***

## ID Sensor

Setiap DAQ melaporkan ID sensor yang stabil. Bentuknya berbeda-beda tergantung model:

| Model | Bentuk ID | Contoh |
| --- | --- | --- |
| DAQ-U | 5-oktet dengan tanda hubung | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktet dengan tanda hubung | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

ID sensor adalah:

* tercantum dalam setiap berkas `.daq` yang direkamnya,
* kunci yang digunakan Chloros untuk mengambil paket kalibrasi pabrik unit tersebut,
* nilai yang Anda berikan ke `--sensor-id` dalam perintah CLI `pool-*`, dan
* untuk DAQ-E, juga nama host mDNS-nya (`daq-e-def330.local`) — nilai yang diterima oleh `--eth-host`.

***

## Kalibrasi pabrik dan cloud

Setiap unit DAQ dikalibrasi secara individual di pabrik menggunakan rantai radiometrik yang dapat dilacak ke NIST, dan Chloros memuat paket kalibrasi masing-masing unit yang diidentifikasi berdasarkan ID sensornya. Laporan kalibrasi per unit (PDF) dapat diunduh dari pengaturan sensor di [tab Sensor Cahaya](gui.md).

{% hint style="warning" %}
**DAQ-U dan DAQ-M memerlukan akses cloud untuk kalibrasi.**Kedua model ini tidak menyimpan apa pun di perangkat: paket kalibrasi pabriknya disimpan di cloud MAPIR dan diambil berdasarkan ID sensor (lalu disimpan dalam cache lokal). Chloros memerlukan koneksi internet untuk mengirimkan data W/m²/nm yang telah dikalibrasi dari DAQ-U atau DAQ-M.**DAQ-E merupakan pengecualian** — perangkat ini menyimpan kalibrasinya di dalam perangkat itu sendiri.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Tempat penyimpanan rekaman

| Permukaan | Tujuan default `.daq` |
| --- | --- |
| GUI — Tab Sensor Cahaya | `<project folder>/light_sensor/` (rekaman yang sudah selesai ditambahkan ke proyek secara otomatis) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` pada mesin yang menjalankan backend |

Setiap nama file `.daq` mencakup ID sensor dan cap waktu.

***

## Dalam bab ini

* [**Tab DAQ di Chloros**](gui.md) — panduan lengkap antarmuka pengguna grafis (GUI): menghubungkan setiap model, pengaturan per-sensor, grafik spektrum, data kolorimetrik langsung, reflektansi dua sensor, dan perekaman.
* [**Panduan Cepat CLI (pool-\*)**](cli-quick-start.md) — mengendalikan sensor DAQ dari `chloros-cli daq pool-*`, jalur baris perintah yang didukung.
* [**Profil Batas &amp; Rentang yang Dikalibrasi**](caps-and-range.md) — batas apa saja yang ada per model, cara mendeklarasikannya, serta rentang spektral yang dikalibrasi secara terperinci.
* [**Perekaman &amp; Format .daq**](recording.md) — format SQLite `.daq` dan alur kerja perekaman.
* [**Jaringan DAQ-E &amp; Sinkronisasi Waktu**](ethernet-ptp.md) — mode transportasi DAQ-E dan sinkronisasi waktu PTP.
* [**Alur Kerja Reflektansi**](reflectance.md) — penggunaan data downwelling DAQ untuk menghasilkan reflektansi.
* Untuk dokumentasi lengkap tingkat flag, lihat [Referensi CLI](../reference/cli-reference.md) (bagian `chloros-cli daq`) dan [Referensi SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), yang keduanya ditulis agar dapat langsung digunakan oleh asisten AI.

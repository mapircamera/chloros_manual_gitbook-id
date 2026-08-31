# Menghubungkan Kamera

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Tab Kamera sebelum ada perangkat yang terhubung</p></figcaption></figure>Chloros secara otomatis mendeteksi kamera LATTICE pada tautan — dari tab Kamera di GUI, dari `chloros-cli lattice`, atau dari Python SDK. String model kamera menentukan semua proses selanjutnya: Chloros menentukan profil sensor, tata letak band, dan kalibrasi pabrik dari data `DeviceUserID` + `DeviceSerialNumber` kamera tersebut, sehingga **tidak perlu melakukan konfigurasi apa pun per kamera**.

Sebelum menghubungkan, pastikan jaringan host telah disiapkan — pengalamatan link-local, jumbo frames, dan, untuk array, pengaturan buffer penerimaan NIC. Itu adalah pengaturan sisi perangkat keras dan dijelaskan dalam manual LATTICE: [**Pengaturan Jaringan**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Menghubungkan dari GUI

Buka tab **Kamera**di bilah samping Chloros (tab perangkat keras akan muncul setelah backend selesai dimulai), atau gunakan menu utama →**Hubungkan ke Kamera**. Keduanya akan membuka dialog**Hubungkan Kamera**.

### Kotak dialog **Connect Camera(s)**Kotak dialog ini akan memindai jaringan begitu dibuka (&quot;Scanning network...&quot;), dan menampilkan daftar semua kamera yang ditemukan. Setiap baris menampilkan**model**kamera (mis. `LATT-M3M-L41-F550`),**nomor seri**, dan**alamat IP**.

* **Klik sebuah baris untuk memilihnya**(ditandai hijau). Anda dapat memilih**beberapa kamera** dan menghubungkannya sekaligus — Chloros akan menghubungkannya secara berurutan.
* Baris dengan lencana **&quot;Terhubung&quot;** sudah terhubung dan tidak dapat dipilih kembali.
* Baris dengan lencana **&quot;Dalam Array&quot;** merupakan bagian dari array kamera yang saat ini terhubung. Putuskan koneksi array terlebih dahulu untuk menggunakan kamera tersebut secara mandiri.
* **Connect** — menghubungkan kamera yang dipilih; tombol ini menampilkan jumlah, misalnya &quot;Connect (3)&quot;, jika lebih dari satu kamera dipilih.
* **Rescan** — menjalankan pencarian ulang.
* **Tutup** — menutup kotak dialog.
* Jika pemindaian selesai tanpa hasil, kotak dialog akan menampilkan **&quot;Tidak ada kamera yang ditemukan di jaringan&quot;** — lihat [Pemecahan Masalah](connecting.md#troubleshooting) di bawah ini.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Kotak dialog Hubungkan Kamera — ditampilkan di sini tanpa adanya kamera di jaringan</p></figcaption></figure>### Koneksi pertama: pengunduhan paket kalibrasi

Pada **koneksi pertama**kamera tertentu ke mesin, Chloros mengunduh paket kalibrasi pabrik kamera (\~3,8 MB) langsung dari kamera tersebut melalui GigE. Selama proses ini berlangsung, kotak dialog menampilkan panel hijau**&quot;Mengunduh data kalibrasi dari kamera&quot;**dengan bilah kemajuan per nomor seri — perkirakan waktu sekitar**70 detik** per kamera. Paket tersebut disimpan dalam cache di host, sehingga saat kamera yang sama dihubungkan kembali, proses pengunduhan akan dilewati sepenuhnya (dan panel tersebut tidak akan ditampilkan).

### Analisis Sistem

Tombol **Analisis Sistem** pada kotak dialog ini memeriksa host dan jaringan (labelnya bertuliskan &quot;Menganalisis...&quot; saat proses berjalan) dan menampilkan laporan diagnostik:

* **Host** — inti CPU dan RAM; nama GPU dan memori, atau &quot;GPU: Tidak terdeteksi&quot;.
* **Antarmuka Jaringan** — nama setiap NIC, kecepatan koneksi, MTU (dengan label &quot;jumbo&quot; jika aktif), status aktif/tidak aktif, dan apakah terpasang pada bus USB.
* **Kamera**— nomor seri, model, alamat IP, dan**NIC mana yang digunakan oleh masing-masing kamera**.
* **Kinerja** — perbandingan fps saat ini vs. fps ideal per kamera untuk format piksel tertentu, disertai garis hijau bertuliskan &quot;Potensi: Peningkatan hingga N× mungkin terjadi&quot; jika nilai ideal melebihi nilai saat ini.
* **Peringatan dan rekomendasi bernomor** — atau &quot;Sistem tampak baik untuk jumlah kamera saat ini.&quot; jika tidak ada yang perlu diperbaiki.

Jalankan fitur ini setiap kali proses pencarian atau streaming berperilaku tidak terduga — fitur ini mengidentifikasi sebagian besar masalah di sisi NIC (MTU salah, kamera terpasang pada antarmuka yang salah, batasan adaptor USB) tanpa perlu keluar dari jendela dialog.

### Menghubungkan susunan

Untuk menghubungkan dua kamera atau lebih sebagai **susunan yang tersinkronisasi**, gunakan wizard penghubung susunan (**Connect Camera Array**) sebagai gantinya: wizard ini akan memandu Anda melalui pemilihan master/slave (yang telah diisi sebelumnya oleh probe pengkabelan GPIO), pilihan mode tampilan (Ubin Terpisah vs. Gabungan), dan adegan pengaturan array dengan proyeksi langsung fps yang dapat dicapai serta bandwidth kabel sebelum Anda mengonfirmasi. Panduan dan alur kerja array dibahas dalam bagian array multi-kamera pada manual ini; setara dengan CLI adalah &quot;Alur Kerja Koneksi Pertama Kamera LATTICE&quot; dalam [Referensi CLI](../reference/cli-reference.md).

## Menghubungkan dari CLI dan SDK

Akses ke CLI dan SDK memerlukan langganan berbayar tingkat Chloros+ dan harus sudah masuk; hal ini diterapkan di sisi server (`401 AUTH_REQUIRED` jika belum masuk, `403 PLAN_UPGRADE_REQUIRED` pada tingkatan gratis).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Tanda tangan lengkap, opsi, dan alur kerja penangkapan: [Referensi CLI](../reference/cli-reference.md) § `chloros-cli lattice`, [Referensi SDK](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Bagaimana kalibrasi diselesaikan saat terhubung

Setiap kamera LATTICE membawa paket kalibrasi pabriknya **di dalam kamera**, dan Chloros juga memeriksa cloud MAPIR saat kamera terhubung:

| Situasi   | Apa yang digunakan oleh Chloros                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Kalibrasi terbaru yang diterbitkan untuk nomor seri tersebut** — salinan di cloud akan menggantikan salinan di kamera. Oleh karena itu, kamera yang telah dikalibrasi ulang atau diperbarui oleh MAPIR akan memperbarui dirinya secara otomatis; tidak diperlukan tindakan pengguna. |
| **Offline**|**Paket kalibrasi di dalam kamera**, apa adanya. Alur kerja sepenuhnya offline tetap berfungsi; alur kerja tersebut hanya tidak akan mengambil kalibrasi yang lebih baru hingga kamera terhubung ke internet sekali (atau di-flash ulang ke pengaturan pabrik).                                                  |

Pada saat pengambilan gambar, koefisien yang sebenarnya diterapkan **disimpan secara permanen dalam metadata XMP setiap gambar**. Pembaruan kalibrasi di kemudian hari tidak akan pernah mengubah gambar yang telah Anda ambil tanpa pemberitahuan — pemrosesan ulang gambar lama menggunakan koefisien yang tercantum dalam metadata XMP-nya, bukan yang terbaru saat ini.

## Pemecahan Masalah

* **&quot;Tidak ditemukan kamera di jaringan&quot;**— periksa pengaturan link-local di [Pengaturan Jaringan](https://mapir.gitbook.io/lattice-camera/setup/network-setup): NIC host statis `169.254.x.x/16`, kamera berada pada tautan yang sama, tidak diharapkan ada DHCP/gateway. Kemudian gunakan**Analisis Sistem**di dialog koneksi untuk memeriksa NIC mana yang dapat (atau tidak dapat) mendeteksi setiap kamera. Lakukan**Pemindaian Ulang** setelah ada perubahan kabel atau NIC.
* **Perangkat yang sebelumnya berfungsi kini menolak terhubung** (panel array dengan `FRAMES WILL DROP` / `Reduce ROI to enable`) — pembaruan driver NIC telah secara diam-diam mereset pengaturan receive-ring. Terapkan kembali pengaturan tersebut atau jalankan `chloros-cli lattice network --fix` dari terminal dengan hak akses tinggi; lihat [Pengaturan Jaringan](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Sebuah kamera menampilkan &quot;In Array&quot;** — kamera tersebut merupakan bagian dari sesi array yang terhubung. Putuskan koneksi array untuk menggunakan kamera secara mandiri.

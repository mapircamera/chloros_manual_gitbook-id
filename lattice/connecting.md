# Menghubungkan Kamera

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Tab Kamera sebelum ada perangkat yang terhubung</p></figcaption></figure>

Chloros secara otomatis mendeteksi kamera LATTICE pada koneksi — baik dari tab "Kamera" di antarmuka pengguna (GUI), dari `chloros-cli lattice`, maupun dari Python SDK. String model kamera menentukan semua hal selanjutnya: Chloros menentukan profil sensor, tata letak pita, dan kalibrasi pabrik dari `DeviceUserID` + `DeviceSerialNumber` kamera, sehingga **tidak ada yang perlu dikonfigurasi per kamera**.

Sebelum menghubungkan, pastikan jaringan host telah disiapkan — pengalamatan link-local, jumbo frames, dan, untuk array, pengaturan buffer penerimaan NIC. Itu adalah pengaturan sisi perangkat keras dan tercantum dalam manual LATTICE: [**Pengaturan Jaringan**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Menghubungkan dari GUI

Buka tab **Kamera** di bilah samping Chloros (tab perangkat keras akan muncul setelah backend selesai dimulai), atau gunakan menu utama → **Hubungkan ke Kamera**. Keduanya akan membuka dialog**Hubungkan Kamera**.

### Kotak dialog **Connect Camera(s)**Kotak dialog ini akan memindai jaringan begitu dibuka ("Scanning network..."), dan menampilkan daftar semua kamera yang ditemukan. Setiap baris menampilkan**model**kamera (mis. `LATT-M3M-L41-F550`),**nomor seri**, dan**alamat IP**.

* **Klik sebuah baris untuk memilihnya**(disorot hijau). Anda dapat memilih**beberapa kamera** dan menghubungkannya sekaligus — Chloros akan menghubungkannya secara berurutan.
* Baris dengan label **"Connected"** sudah terhubung dan tidak dapat dipilih kembali.
* Baris dengan label **"In Array"** merupakan bagian dari kelompok kamera yang saat ini terhubung. Putuskan koneksi array terlebih dahulu untuk menggunakan kamera tersebut secara mandiri.
* **Connect** — menghubungkan kamera yang dipilih; tombol ini menampilkan jumlah kamera, misalnya "Connect (3)", jika lebih dari satu kamera dipilih.
* **Rescan** — menjalankan proses pencarian ulang.
* **Tutup** — menutup kotak dialog.
* Jika pemindaian selesai tanpa hasil, kotak dialog akan menampilkan **"Tidak ditemukan kamera di jaringan"** — lihat [Pemecahan Masalah](connecting.md#troubleshooting) di bawah ini.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>Kotak dialog Hubungkan Kamera — ditampilkan di sini tanpa adanya kamera di jaringan</p></figcaption></figure>

### Koneksi pertama: pengunduhan paket kalibrasi

Pada **koneksi pertama** suatu kamera ke mesin, Chloros mengambil paket kalibrasi pabrik kamera (\~3,8 MB) langsung dari kamera tersebut melalui GigE. Selama proses ini berlangsung, kotak dialog menampilkan panel hijau **"Mengunduh data kalibrasi dari kamera"**dengan bilah kemajuan per nomor seri — perkirakan sekitar**70 detik** per kamera. Jika ada lebih dari enam kamera, baris kemajuan akan bergulir di dalam panel. Paket tersebut disimpan dalam cache di host, sehingga saat kamera yang sama dihubungkan kembali, proses pengunduhan akan dilewati sepenuhnya (dan panel tersebut tidak akan pernah ditampilkan).

### Analisis Sistem

Tombol **Analisis Sistem** pada dialog tersebut memeriksa host dan jaringan (labelnya bertuliskan "Menganalisis..." saat proses berjalan) dan menampilkan laporan diagnostik:

* **Host** — inti CPU dan RAM; nama GPU dan memori, atau "GPU: Tidak terdeteksi".
* **Antarmuka Jaringan** — nama setiap NIC, kecepatan koneksi, MTU (dengan label "jumbo" jika aktif), status aktif/tidak aktif, dan apakah terhubung ke bus USB.
* **Kamera**— nomor seri, model, alamat IP, dan**NIC mana yang digunakan oleh masing-masing kamera**.
* **Kinerja** — perbandingan fps saat ini vs. fps ideal per kamera untuk format piksel, dengan garis hijau bertuliskan "Potensi: Peningkatan N× mungkin terjadi" jika nilai ideal melebihi nilai saat ini.
* **Peringatan dan rekomendasi bernomor** — atau "Sistem tampak baik untuk jumlah kamera saat ini." jika tidak ada yang perlu diperbaiki.

Jalankan fitur ini setiap kali proses penemuan atau streaming berperilaku tidak terduga — fitur ini mengidentifikasi sebagian besar masalah di sisi NIC (MTU salah, kamera terpasang pada antarmuka yang salah, batasan adaptor USB) tanpa perlu keluar dari jendela dialog.

### Menghubungkan susunan kamera

Untuk menghubungkan dua atau lebih kamera sebagai **susunan yang tersinkronisasi**, gunakan wizard penghubung susunan (**Connect Camera Array**): wizard ini memandu Anda melalui pemilihan master/slave (yang telah diisi sebelumnya oleh probe kabel sinkronisasi — kamera mana pun yang terhubung dengan kabel dapat menjadi master, dan kamera RGB `-FRGB` sudah dipilih secara otomatis saat terhubung dengan kabel), pemilihan mode tampilan (Ubin Terpisah vs. Ubin Gabungan), serta halaman pengaturan array dengan proyeksi langsung mengenai fps yang dapat dicapai dan bandwidth kabel sebelum Anda mengonfirmasi. Wizard dan alur kerja array dibahas dalam bagian array multi-kamera pada manual ini; padanan CLI-nya adalah "LATTICE Camera First-Connect Workflow" dalam [Referensi CLI](../reference/cli-reference.md).

## Menghubungkan dari CLI dan SDK
Akses

CLI dan SDK memerlukan langganan Chloros+ berbayar dan Anda harus sudah masuk; hal ini diberlakukan di sisi server (`401 AUTH_REQUIRED` jika belum masuk, `403 PLAN_UPGRADE_REQUIRED` pada langganan gratis).

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

## Cara kalibrasi diselesaikan saat terhubung

Setiap kamera LATTICE menyimpan paket kalibrasi pabriknya **di dalam kamera**, dan Chloros juga memeriksa cloud MAPIR saat kamera terhubung:

| Situasi   | Apa yang digunakan Chloros                                                                                                                                                                                                           |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**|**Kalibrasi terbaru yang diterbitkan untuk nomor seri tersebut** — salinan di cloud akan menggantikan salinan di kamera. Oleh karena itu, kamera yang telah dikalibrasi ulang atau diperbarui oleh MAPIR akan memperbarui dirinya secara otomatis; tidak diperlukan tindakan apa pun dari pengguna. |
| **Offline**|**Paket kalibrasi di dalam kamera**, apa adanya. Alur kerja sepenuhnya offline tetap berfungsi; alur kerja tersebut hanya tidak akan mendeteksi kalibrasi yang lebih baru hingga kamera terhubung ke internet sekali (atau di-reflash ke pengaturan pabrik).                                                  |

Pada saat pengambilan gambar, koefisien yang sebenarnya diterapkan **disimpan secara permanen dalam metadata XMP setiap gambar**. Pembaruan kalibrasi di kemudian hari tidak akan pernah secara diam-diam mengubah gambar yang telah Anda ambil — pemrosesan ulang gambar lama menggunakan koefisien yang tercantum dalam metadata XMP-nya, bukan yang terbaru saat ini.

## Pemecahan Masalah

* **"Tidak ditemukan kamera di jaringan"**— verifikasi pengaturan link-local di [Pengaturan Jaringan](https://mapir.gitbook.io/lattice-camera/setup/network-setup): NIC host statis `169.254.x.x/16`, kamera berada di tautan yang sama, tidak diharapkan ada DHCP/gateway. Kemudian gunakan**Analisis Sistem**di dialog koneksi untuk memeriksa NIC mana yang dapat (atau tidak dapat) mendeteksi setiap kamera. Lakukan**Pemindaian Ulang** setelah ada perubahan kabel atau NIC.
* **Perangkat yang sebelumnya berfungsi kini menolak terhubung** (panel array dengan `FRAMES WILL DROP` / `Reduce ROI to enable`) — pembaruan driver NIC telah secara diam-diam mereset pengaturan receive-ring. Terapkan kembali pengaturan tersebut atau jalankan `chloros-cli lattice network --fix` dari terminal dengan hak akses administrator; lihat [Pengaturan Jaringan](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Sebuah kamera menampilkan "In Array"** — kamera tersebut merupakan bagian dari sesi array yang terhubung. Putuskan koneksi array untuk menggunakan kamera secara mandiri.

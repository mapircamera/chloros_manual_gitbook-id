# Profil Tutup &amp; Rentang yang Telah Dikalibrasi

> Tutup-tutup tersebut — termasuk tutup mana yang disertakan dengan sensor mana, cara pemasangannya, dan perilaku optiknya — didokumentasikan dalam **[panduan pengguna DAQ](https://mapir.gitbook.io/daq)**. Halaman ini membahas *penetapan* penutup yang terpasang ke Chloros, yang memastikan koreksi dilakukan dengan benar.

Setiap kalibrasi radiometrik pabrik sensor cahaya DAQ menggambarkan sensor *tanpa penutup*. Penutup fisik yang dipasang di atas diffuser mengubah jenis cahaya yang ditangkap oleh sensor, sehingga Chloros menerapkan **profil koreksi penutup** yang diukur di pabrik di atas paket kalibrasi. Menentukan tutup yang benar merupakan bagian dari proses memperoleh data yang terkalibrasi — halaman ini membahas tutup apa saja yang tersedia untuk setiap model, cara menentukan tutup tersebut, serta rentang spektral terkalibrasi sensor yang sebenarnya.

## Ketersediaan penutup berdasarkan model

| Profil penutup (`cap_id`) | Penutup fisik | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tutup korektor kosinus sinar matahari (**standar pada setiap model**) | Ya | Ya | Ya |
| `fov_15` / `fov_45` / `fov_90` | Kerucut pembatas FOV (15° / 45° / 90°) | Ya | — | Ya |
| `fov_30` / `fov_60` | Kerucut pembatas FOV (30° / 60°) | Ya | — | — |
| `none` | Tanpa penutup | — | — | Ya |

Catatan khusus model:

* **DAQ-M memiliki satu profil penutup: `sunshine_cosine`.** Definisi produknya adalah &quot;Bare-plus-Sunshine-cap&quot;, dan DAQ-M tanpa penutup tidak memerlukan profil geometri.
* **DAQ-U tanpa penutup adalah &quot;true-bare&quot;** — tidak memerlukan profil geometri sama sekali, itulah sebabnya tidak ada profil `none` untuknya.
* **`none` pada DAQ-E BUKANlah operasi kosong.** Diffuser DAQ-E yang tersembunyi dan tertutup kaca memiliki koreksi geometri tersendiri, sehingga &quot;tanpa penutup&quot; itu sendiri merupakan profil yang terukur pada model ini.
* **DAQ-E tanpa penutup tidak dapat mengukur sinar matahari langsung pada ketinggian apa pun** — penutup Sunshine adalah konfigurasi lapangan. Jangan merencanakan pekerjaan di luar ruangan dengan menggunakan DAQ-E tanpa penutup.

Di pengaturan per-sensor pada GUI (ikon roda gigi di tab Light Sensors), menu tarik-turun **Cap** juga menawarkan opsi &quot;None (bare sensor)&quot; pada DAQ-U dan DAQ-M — pada kedua model tersebut, &quot;bare&quot; hanya berarti tidak ada koreksi penutup yang diterapkan, sesuai catatan di atas. Pilih opsi ini hanya jika penutup telah dilepas secara fisik.

## Menyatakan penggunaan penutup — dan mengapa hal ini penting

**Kode `cap_id` yang dinyatakan harus sesuai dengan penutup yang secara fisik terpasang pada sensor.** Baik sensor maupun perangkat lunak tidak dapat mendeteksi penutup yang terpasang. Penentuan ini memengaruhi dua hal:

1. **Koreksi real-time** yang diterapkan pada setiap spektrum.
2. **Stempel penutup yang ditulis ke dalam setiap rekaman `.daq`**, yang menjadi acuan bagi pemrosesan reflektansi selanjutnya.

Tutup Sunshine meredam cahaya sekitar **12× sesuai desainnya**, sehingga perekaman dengan deklarasi tutup yang salah akan menyebabkan skala spektrum menjadi salah sekitar faktor tersebut. Segera deklarasikan perubahan tutup.

### Mengatur tutup

GUI: Tab Light Sensors → ikon roda gigi pada baris sensor → menu tarik-turun **Cap**. Nilai default untuk setiap model adalah `sunshine_cosine` (semua sensor DAQ dikirimkan dengan korektor kosinus terpasang), dan pilihan ini tetap berlaku untuk proyek tersebut.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (backend harus sedang berjalan):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI secara sintaksis menerima daftar lengkap `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); setiap profil diverifikasi berdasarkan model sensor saat terhubung, sehingga ID kap yang tidak tersedia (misalnya ID E-only pada DAQ-U) akan gagal dengan pesan kesalahan yang jelas daripada melakukan koreksi yang salah. Nilai default backend saat tidak ada yang diteruskan adalah `sunshine_cosine`.

Python SDK catatan: `cap_id` **bukan** knob SDK — `connect_daq_sensor()` / `DAQSensorSession` tidak mengekspos parameter cap. Pilih batas melalui perintah CLI di atas atau menu tarik-turun antarmuka pengguna; lihat [Referensi SDK](../reference/sdk-reference.md).

Lanjutan: profil disertakan dalam instalasi Chloros di `daq/cap_profiles/<u|m|e>/<cap_id>.json` dan dapat diganti per pengguna di `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Terlepas dari batas nilai, sensor yang belum pernah dikalibrasi ulang secara otomatis akan menerima penyesuaian offset gelap kecil yang diperoleh dari data armada — tanpa memerlukan tindakan pengguna.

## Kinerja batas nilai sinar matahari (konfigurasi luar ruangan)

Angka-angka yang dapat digunakan sebagai dasar prosedur:

| Sifat | Nilai |
| --- | --- |
| Bidang pandang | 180° hemisferis |
| Kesalahan respons kosinus | ≤ ±4 % hingga sudut insidensi 60°; ≤ ±4,5 % hingga sudut insidensi 70° |
| Batas matahari rendah | Tidak disarankan di bawah ~15° elevasi matahari |
| Atenuasi | ~12× (sesuai desain) |
| Pengulangan pemasangan ulang penutup | ≈ 1,5 % |
| Intensitas radiasi kuantitatif | Rata-rata pembacaan **≥ 15 detik** (karakteristik instrumen, bukan cacat) |

Untuk angka intensitas radiasi kuantitatif apa pun — termasuk referensi reflektansi — gunakan rata-rata pembacaan setidaknya selama 15 detik, bukan hanya satu bingkai.

## Rentang spektral yang dikalibrasi

| Sifat | Nilai |
| --- | --- |
| Pengambilan sampel spektral | 340–1010 nm dengan langkah 5 nm (135 titik) |
| Rentang yang dikalibrasi secara radiometrik | **~374–974 nm** (diterapkan dalam perangkat lunak) |

Sensor melaporkan grid penuh 340–1010 nm, tetapi gain radiometrik yang dapat dilacak ke NIST mencakup rentang ~374–974 nm. Chloros **menolak pembagian reflektansi absolut** untuk pita kamera mana pun yang kurang dari setengah bobot spektralnya berada di dalam rentang tersebut, dan melaporkan alasan pengabaian `dls-uncalibrated-band-<nm>` alih-alih menghasilkan produk yang tidak dikalibrasi. Di antara SKU kamera yang tersedia, hanya filter F988 yang berada di luar rentang ini; filter tersebut menggunakan alur kerja panel reflektansi sebagai gantinya — lihat [Alur Kerja Reflektansi](reflectance.md).

Untuk model sensor, transport, dan ID sensor, lihat [Ikhtisar DAQ](README.md). Untuk mengetahui bagaimana cap stamp digunakan selama pemrosesan, lihat [Perekaman &amp; Format .daq](recording.md).

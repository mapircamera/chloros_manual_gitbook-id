# Profil Tutup &amp; Rentang yang Telah Dikalibrasi

> Tutup-tutup tersebut — termasuk tutup mana yang disertakan dengan sensor mana, cara pemasangannya, dan perilaku optiknya — didokumentasikan dalam **[panduan pengguna DAQ](https://mapir.gitbook.io/daq)**. Halaman ini membahas *penyampaian informasi* mengenai tutup yang terpasang kepada Chloros
, yang merupakan kunci agar koreksi menjadi akurat.

Setiap kalibrasi radiometrik pabrik sensor cahaya DAQ menggambarkan sensor *tanpa penutup*. Penutup fisik yang dipasang di atas difuser mengubah jenis cahaya yang ditangkap oleh sensor, sehingga Chloros
menerapkan **profil koreksi penutup** yang diukur di pabrik di atas paket kalibrasi. Menyatakan tutup yang benar merupakan bagian dari mendapatkan data yang dikalibrasi — halaman ini membahas tutup apa saja yang tersedia per model, cara menyatakannya, dan apa sebenarnya rentang spektral yang dikalibrasi dari sensor tersebut.

## Ketersediaan penutup berdasarkan model

| Profil penutup (`cap_id`) | Penutup fisik | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tutup korektor kosinus sinar matahari (**standar pada setiap model**) | Ya | Ya | Ya |
| `fov_15` / `fov_45` / `fov_90` | Kerucut pembatas bidang pandang (15° / 45° / 90°) | Ya | — | Ya |
| `fov_30` / `fov_60` | Kerucut pembatas FOV (30° / 60°) | Ya | — | — |
| `none` | Tanpa penutup | — | — | Ya |

Catatan khusus model:

* **DAQ-M memiliki satu profil penutup: `sunshine_cosine`.** Definisi produknya adalah &quot;Bare-plus-Sunshine-cap&quot;, dan DAQ-M tanpa penutup tidak memerlukan profil geometri.
* **DAQ-U tanpa penutup adalah &quot;true-bare&quot;** — tidak memerlukan profil geometri sama sekali, itulah sebabnya tidak ada profil `none` untuknya.
* **`none` pada DAQ-E BUKANlah operasi kosong.** Diffuser DAQ-E yang tersembunyi dan dilapisi kaca memiliki koreksi geometri nyata tersendiri, sehingga &quot;tanpa penutup&quot; itu sendiri merupakan profil yang terukur pada model ini.
* **DAQ-E tanpa penutup tidak dapat mengukur sinar matahari langsung pada ketinggian apa pun** — penutup Sunshine adalah konfigurasi lapangan. Jangan merencanakan pekerjaan di luar ruangan dengan menggunakan DAQ-E tanpa penutup.

Di pengaturan per-sensor pada GUI (ikon roda gigi di tab Light Sensors), menu tarik-turun **Cap** juga menawarkan opsi &quot;None (sensor tanpa penutup)&quot; pada DAQ-U dan DAQ-M — pada kedua model tersebut, &quot;tanpa penutup&quot; berarti tidak ada koreksi penutup yang diterapkan, sesuai catatan di atas. Pilih opsi ini hanya jika penutup telah dilepas secara fisik.

## Menentukan status penutup — dan mengapa hal ini penting

**Kode `cap_id` yang ditentukan harus sesuai dengan penutup yang secara fisik terpasang pada sensor.** Baik sensor maupun perangkat lunak tidak dapat mendeteksi penutup yang terpasang. Penentuan ini memengaruhi dua hal:

1. **Koreksi real-time** yang diterapkan pada setiap spektrum.
2. **Kode penutup yang ditulis ke dalam setiap rekaman `.daq`**, yang menjadi acuan bagi pemrosesan reflektansi selanjutnya.

Tutup Sunshine meredam cahaya sekitar **12× sesuai desain**, sehingga perekaman dengan deklarasi tutup yang salah akan menyebabkan skala spektrum melenceng sekitar faktor tersebut. Segera deklarasikan perubahan tutup.

### Mengatur tutup

GUI: Tab Light Sensors → ikon roda gigi pada baris sensor → menu tarik-turun **Cap**. Nilai default untuk setiap model adalah `sunshine_cosine`, sebuah asumsi (DAQ-U, DAQ-M, dan DAQ-E dikirim tanpa nilai batas, dan perekaman menyatakan bahwa nilai batas diasumsikan hingga Anda memilih salah satunya), dan pilihan tersebut tetap berlaku untuk proyek tersebut.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(backend harus sedang berjalan):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

CLI
menerima daftar `cap_id` secara sintaksis (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); setiap profil diverifikasi berdasarkan model sensor saat terhubung, sehingga ID cap yang tidak tersedia (misalnya ID khusus E pada DAQ-U) akan gagal dengan pesan kesalahan yang jelas, bukan melakukan koreksi yang salah. Nilai default backend jika tidak ada yang diberikan adalah `sunshine_cosine`.

Python
SDK
: tentukan ID cap saat koneksi menggunakan `connect_daq_sensor(cap_id="sunshine_cosine")`, atau ganti ID tersebut saat runtime menggunakan `DAQSensorSession.set_cap("fov_45")`; lihat [Referensi SDK
](../reference/sdk-reference.md#daq-sensor-sessions).

Tingkat Lanjut: profil disertakan dalam instalasi Chloros
di `daq/cap_profiles/<u|m|e>/<cap_id>.json` dan dapat diganti per pengguna di `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Terlepas dari batas atas, sensor yang belum pernah dikalibrasi ulang secara otomatis akan menerima penyempurnaan offset gelap kecil yang diperoleh dari armada — tanpa memerlukan tindakan pengguna.

## DAQ-E-S: difuser dan kemiringan tetap

**DAQ-E-S** adalah DAQ-E dengan dua perbedaan yang penting bagi Chloros
. Seluruh fitur lainnya (penemuan, koneksi, PTP, perekaman, pengaturan perangkat) berfungsi persis seperti pada DAQ-E.

* **Diffuser-nya permanen, sehingga tidak memerlukan koreksi batas.** Diffuser tersebut sudah terpasang pada unit saat kalibrasi pabrik diukur, sehingga kalibrasi tersebut sudah menunjukkan irradiansi downwelling yang sebenarnya. Chloros
tidak menerapkan profil cap apa pun padanya, apa pun yang diminta: cap yang dinamai untuknya akan diabaikan dengan pesan peringatan, perekaman datanya menunjukkan bahwa cap berasal dari model, dan peringatan cap yang diasumsikan tidak pernah muncul untuknya. Di Pengaturan Proyek → Sensor Cahaya DAQ, barisnya tidak menampilkan batas, dan pengaturan batas yang berlaku untuk seluruh proyek tidak memengaruhinya. Jangan menerapkan batas pada datanya secara manual: koreksi Sunshine di atasnya akan menunjukkan nilai sekitar 11 kali lebih tinggi.
* **Perangkat ini dilengkapi dengan akselerometer 3-sumbu.** Respon kosinus diffuser hanya berlaku saat sensor dalam posisi horizontal, sehingga sensor yang miring akan melaporkan irradiansi yang salah tanpa gejala lain. Oleh karena itu, setiap pembacaan DAQ-E-S menyertakan data kemiringan, roll, dan pitch-nya, dan rekaman `.daq` yang diambil dari satu sensor menyimpannya di samping setiap spektrum (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), sehingga Anda dapat membedakan antara awan dan dudukan yang terguncang.

Untuk setiap bidang sikap dan cara pelaporan kesalahan dalam aliran data sikap, lihat [DAQ-E-S dalam Referensi CLI
](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Kinerja Sunshine cap (konfigurasi luar ruangan)

Angka-angka yang dapat Anda gunakan sebagai dasar prosedur:

| Sifat | Nilai |
| --- | --- |
| Bidang pandang | 180° hemisferis |
| Kesalahan respons kosinus | ≤ ±4 % hingga sudut insiden 60°; ≤ ±4,5 % hingga 70° |
| Batas matahari rendah | Tidak disarankan di bawah ~15° elevasi matahari |
| Atenuasi | ~12× (sesuai desain) |
| Pengulangan pemasangan ulang penutup | ≈ 1,5 % |
| Intensitas radiasi kuantitatif | Rata-rata pembacaan **≥ 15 detik** (karakteristik instrumen, bukan cacat) |

Untuk angka intensitas radiasi kuantitatif apa pun — termasuk referensi reflektansi — gunakan rata-rata pembacaan setidaknya 15 detik, bukan satu bingkai saja.

## Rentang spektral yang dikalibrasi

| Sifat | Nilai |
| --- | --- |
| Pengambilan sampel spektral | 340–1010 nm dengan langkah 5 nm (135 titik) |
| Rentang yang dikalibrasi radiometrik | **~374–974 nm** (diterapkan dalam perangkat lunak) |

Sensor melaporkan grid penuh 340–1010 nm, tetapi gain radiometrik yang dapat dilacak ke NIST mencakup rentang ~374–974 nm. Chloros
**menolak pembagian reflektansi absolut** untuk pita kamera mana pun yang memiliki kurang dari setengah bobot spektralnya di dalam rentang tersebut, dan melaporkan alasan pengabaian `dls-uncalibrated-band-<nm>` alih-alih menghasilkan produk yang tidak dikalibrasi. Di antara SKU kamera yang tersedia, hanya filter F988 yang berada di luar rentang ini; filter tersebut menggunakan alur kerja panel reflektansi sebagai gantinya — lihat [Alur Kerja Reflektansi](reflectance.md).

Untuk model sensor, transportasi, dan ID sensor, lihat [Ikhtisar DAQ](README.md). Untuk mengetahui bagaimana cap stamp digunakan selama pemrosesan, lihat [Perekaman &amp; Format .daq](recording.md).

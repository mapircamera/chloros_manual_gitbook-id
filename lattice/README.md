# Kamera LATTICE

LATTICE adalah sistem kamera multispektral modular dari MAPIR untuk pencitraan pertanian dan ilmiah. Setiap kamera LATTICE dibangun berdasarkan sensor global-shutter Sony IMX265 (**3,1 MP, piksel 3,45 µm**) dan terhubung melalui Ethernet sebagai perangkat**GigE Vision**.

Chloros 1.3.0 mengendalikan kamera LATTICE secara langsung — pencarian, pratinjau langsung, pengambilan gambar, dan susunan multi-kamera yang tersinkronisasi — dari tiga antarmuka:

| Antarmuka    | Di mana                                                          | Platform                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Tab **Kamera** di bilah samping Chloros                         | Windows 10/11 x64                                        |
| CLI        | Keluarga perintah `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Mencari perangkat kerasnya?**Modul kamera, lensa, filter dan pita, bingkai dan dudukan, kabel, PoE, serta kabel pemicu didokumentasikan dalam [**panduan pengguna LATTICE**](https://mapir.gitbook.io/lattice-camera). Bab ini membahas cara mengoperasikan kamera dari Chloros.

Rekaman LATTICE berupa berkas standar `.tif`/`.tiff`, dan Chloros selalu memprosesnya mulai dari rekaman mentah. Lihat [Referensi CLI](../reference/cli-reference.md) dan [Referensi SDK](../reference/sdk-reference.md) untuk daftar lengkap perintah dan antarmuka API.

## Dua konfigurasi sensor

| Konfigurasi | Sensor       | Filter                                | Hasil yang dihasilkan oleh satu kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Warna Bayer | filter triple-bandpass                |**Tiga pita yang telah dikalibrasi dari satu eksposur**                 |
| **M3M**| Monokrom   | filter interferensi pita sempit tunggal |**Satu pita yang dikalibrasi**; gabungkan beberapa kamera M3M untuk indeks |

Karena kamera M3M bersifat monokrom di balik satu filter, setiap pita mendapatkan eksposurnya sendiri. Kamera M3C mencakup ketiga pita frekuensinya dengan satu eksposur sensor.

## String model dan penamaan

Setiap kamera menyimpan identitasnya dalam GenICam `DeviceUserID` sebagai string model:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros menampilkannya dengan awalan `LATT-` (misalnya `LATT-M3M-L87-F450`). String yang sama, yaitu `LATT-…`, ditulis ke dalam tag EXIF `Model` pada setiap ekspor dan digunakan sebagai nama folder keluaran kamera dalam proyek yang telah diproses.

| Komponen | Nilai                                                   | Arti                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Warna Bayer / monokrom                                                                          |
| Lensa      | `L41` / `L87`                                            | Angka tersebut merupakan **sudut pandang horizontal dalam derajat**: L41 = sempit (41°), L87 = lebar (87°)    |
| Filter    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) atau `F<nm>` (M3M) | Lihat [Filter &amp; Pita Spektral](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

String model mengendalikan semua proses selanjutnya: Chloros menentukan profil sensor, tata letak pita, dan kalibrasi pabrik dari `DeviceUserID` + `DeviceSerialNumber`. Tidak ada yang perlu dikonfigurasi per kamera — lihat [Menghubungkan Kamera](connecting.md).

## Filter dan pita

Pusat pita, tepi FWHM, dan katalog lengkap 23-SKU M3M merupakan spesifikasi produk, sehingga tercantum dalam manual perangkat keras: [**Filter &amp; Pita Spektral**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Yang penting dari sisi perangkat lunak: kode filter dalam string model menentukan produk apa saja yang dapat dibuat oleChloros. Kamera dengan filter RGB (`FRGB`) hanya menghasilkan produk yang telah melalui proses debayering dan pratinjau — radian dan reflektansi per-pita tidak relevan untuk sensor pita lebar, sehingga Chloros mengabaikannya dan menyatakan hal tersebut. Setiap filter lainnya menghasilkan rantai lengkap radian → reflektansi → indeks.

## Sekilas tentang kalibrasi radiometrik

Setiap kamera LATTICE dikalibrasi secara individual di pabrik berdasarkan rantai yang dapat ditelusuri ke NIST dan dikirimkan bersama sertifikat khusus untuk setiap kamera. Hal-hal yang dicakup, cara pengukurannya, serta tingkat akurasi yang dapat Anda sebutkan tercantum dalam manual perangkat keras: [**Kalibrasi Radiometrik Pabrik**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

Dari sisi perangkat lunak, yang penting adalah bahwa Chloros menentukan kalibrasi yang tepat saat kamera terhubung dan mengunci koefisien yang diterapkan ke setiap ekspor — lihat [Menghubungkan Kamera](connecting.md).

## Dalam bab ini

* [Menghubungkan Kamera](connecting.md) — penemuan otomatis, dialog koneksi antarmuka pengguna (GUI), setara CLI/SDK, dan bagaimana kalibrasi pabrik ditentukan (paket di kamera vs. cloud) saat kamera terhubung.
* [Pengaturan Kamera](camera-settings.md) — setiap kontrol di tab Kamera: daftar kamera, panel pengaturan per kamera, dan panel pengaturan array (termasuk editor ROI dan penyelarasan).
* [Pengaturan &amp; Mode Pengambilan Gambar](capture.md) — Pengambilan Semua, pengambilan Tunggal / Berkelanjutan / Interval, jenis ekspor, dan tata letak folder hasil pengambilan gambar.
* [Susunan Multi-Kamera](arrays.md) — wizard koneksi susunan, probe kabel sinkronisasi, bandwidth, PTP, dan tampilan susunan langsung.
* [Kamera Mono &amp; Indeks Vegetasi](mono-indices.md) — Kamera M3M serta alur kerja selaraskan → tumpuk → indeks.

Daftar perintah lengkap tersedia di [Referensi CLI](../reference/cli-reference.md) dan [Referensi SDK](../reference/sdk-reference.md).

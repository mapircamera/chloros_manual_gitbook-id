---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Format Gambar Keluaran

Chloros mengekspor hasil pemrosesan dalam empat format berkas. Pilih format di Pengaturan Proyek (GUI), dengan `--format` (CLI), atau dengan `export_format` (SDK). CLI dan SDK menerima string persis seperti di bawah ini.

| String format | Ekstensi | Jenis piksel | Rentang piksel | Catatan |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(default)* | `.tif` | bilangan digital uint16 | 0 – 65535 | Direkomendasikan untuk fotogrametri / GIS. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = reflektansi 100%. Beberapa aplikasi tidak dapat membaca berkas TIFF dengan bilangan floating-point; ukuran berkas lebih besar. |
| `PNG (8-bit)` | `.png` | bilangan digital uint8 | 0 – 255 | Kompresi tanpa kehilangan data, cocok untuk penayangan dan visualisasi di web. |
| `JPG (8-bit)` | `.jpg` | angka digital uint8 | 0 – 255 | Kompresi lossy, ukuran file terkecil. |

## Lokasi penyimpanan berkas keluaran

Produk disimpan di dalam folder proyek, dikelompokkan berdasarkan kamera dan kemudian berdasarkan format berkas:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Folder kamera adalah `LATT-<sensor>-<lens>-F<filter>` untuk LATTICE dan `<model>_<filter>` (misalnya `Survey3N_RGN`) untuk Survey3. **Setiap produk yang diekspor tetap menggunakan nama file aslinya — folderlah yang mengidentifikasi produk, bukan sufiks nama file.** Lihat [Tempat penyimpanan hasil](reference/cli-reference.md) dalam Referensi CLI untuk aturan lengkapnya.

## Produk LATTICE (tingkat perekaman dan ekspor)

Satu bingkai mentah LATTICE dibagi menjadi setiap produk yang diminta dalam satu kali proses. Setiap jenis produk memiliki pengaturan tersendiri (kotak centang GUI, atau CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, semuanya dalam keadaan AKTIF secara default):

| Tingkat | Konten | Tipe data |
| --- | --- | --- |
| `raw` | Data Bayer langsung dari sensor (kamera monokrom: pita tunggal). Pemrosesan selalu dimulai dari data mentah. | Seperti yang direkam |
| `debayered` | Demosaik linier — 3 saluran untuk M3C, skala abu-abu 1 saluran untuk M3M. | DN linier |
| `radiance` | Radian spektral absolut dari rantai radiometrik lengkap, dalam **W/m²/sr/nm**. Selalu ditulis sebagai 32-bit TIFF (`tiff32/Radiance_Images/`), terlepas dari format ekspor yang dipilih. | float32 |
| `reflectance` | Reflektansi ρ, di mana **DN 32768 = ρ 1,0 (100%)** dengan ruang gerak hingga ρ 2,0. Siap untuk Pix4D. | uint16 |
| `preview` | Render siap tampilan: RGB = white balance + gamma; multispektral = peregangan warna palsu. | tampilan 8-bit |

## Membaca nilai piksel reflektansi

Reflektansi disimpan sebagai bilangan digital bulat, dan **DN yang berarti ρ = 1,0 (reflektansi 100%) bergantung pada kamera sumber**:

| Kamera sumber | ρ = 1,0 adalah DN | Cara mengetahuinya |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ruang headroom hingga ρ 2,0) | Tag XMP `Chloros:PixelScale=32768` tercantum pada file. |
| Survey3 | `65535` (dipotong pada ρ 1,0) | Tidak ada tag XMP `Chloros:*` — ketiadaan tersebut adalah tanda. |

**Baca tag XMP `Chloros:PixelScale` dan bagi dengan nilainya** daripada mengasumsikan nilai konstan. Tag tersebut didefinisikan dalam domain uint16, sehingga nilainya tetap sama di seluruh format keluaran yang melakukan penskalaan ulang — normalisasikan tipe data yang disimpan kembali ke uint16 terlebih dahulu (×257 dari 8-bit, ×65535 dari float32).

{% hint style="warning" %}
**Satu kasus tidak memiliki skala, sesuai desain.** Ketika perekaman sumber 8-bit (BayerRG8) ditulis sebagai 8-bit TIFF, pipeline akan memotong nilainya menjadi 0–255 alih-alih melakukan penskalaan ulang, sehingga tidak ada skala yang menggambarkan berkas tersebut — Chloros sengaja menghilangkan `Chloros:PixelScale` di sana. Jika tag tersebut tidak ada pada berkas reflektansi LATTICE, jangan mengasumsikan adanya skala; ekspor ulang dengan resolusi 16-bit atau 32-bit sebagai gantinya.
{% endhint %}

Untuk aturan lengkap (termasuk tag yang kompatibel dengan MicaSense), lihat **&quot;Membaca piksel reflektansi&quot;** di [Referensi CLI](reference/cli-reference.md).

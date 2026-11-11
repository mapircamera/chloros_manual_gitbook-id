---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Format Gambar Keluaran

Chloros mengekspor hasil pemrosesan dalam empat format berkas. Pilih formatnya di Pengaturan Proyek (GUI), dengan `--format` (CLI), atau dengan `export_format` (SDK). CLI dan SDK menerima string persis seperti di bawah ini.

| String format | Ekstensi | Jenis piksel | Rentang piksel | Catatan |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(default)* | `.tif` | bilangan digital uint16 | 0 – 65535 | Direkomendasikan untuk fotogrametri / GIS. |
| `TIFF (8-bit)` | `.tif` | bilangan digital uint8 | 0 – 255 | Untuk semua pengambilan gambar 8-bit, yang hanya menawarkan format 8-bit: Chloros tidak meningkatkan kedalaman bit. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = reflektansi 100%. Beberapa aplikasi tidak dapat membaca berkas TIFF bilangan floating-point; ukuran berkas lebih besar. |
| `PNG (8-bit)` | `.png` | bilangan digital uint8 | 0 – 255 | Kompresi tanpa kehilangan data, cocok untuk tampilan web dan visualisasi. |
| `JPG (8-bit)` | `.jpg` | bilangan digital uint8 | 0 – 255 | Kompresi dengan kehilangan data, ukuran berkas terkecil. |

## Lokasi penyimpanan file keluaran

Hasil pengolahan disimpan di dalam folder proyek, dikelompokkan berdasarkan kamera dan kemudian berdasarkan format file:

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

Folder kamera adalah `LATT-<sensor>-<lens>-F<filter>` untuk LATTICE dan `<model>_<filter>` (misalnya `Survey3N_RGN`) untuk Survey3. **Setiap produk yang diekspor mempertahankan nama berkas sumber — folderlah yang mengidentifikasi produk, bukan sufiks nama berkas.** Lihat [Lokasi penyimpanan hasil](reference/cli-reference.md) dalam Referensi CLI untuk aturan lengkapnya.

## Produk LATTICE (tingkat penangkapan dan ekspor)

Satu bingkai mentah LATTICE diolah menjadi setiap produk yang diminta dalam satu kali proses. Setiap jenis produk memiliki tombol pengaturan tersendiri (kotak centang di antarmuka pengguna, atau perintah CLI: `--debayered` / `--preview` / `--radiance` / `--reflectance`, semuanya aktif secara default):

| Tingkat | Konten | Tipe data |
| --- | --- | --- |
| `raw` | Data Bayer langsung dari sensor (kamera monokrom: pita tunggal). Pemrosesan selalu dimulai dari data mentah. | Sesuai yang direkam |
| `debayered` | Demosaik linier — 3 saluran untuk M3C, skala abu-abu 1 saluran untuk M3M: offset gelap dari paket kalibrasi dihilangkan, kemudian dilakukan demosaik yang memperhatikan tepi, tanpa koreksi linearitas, flat field, atau unmix. Ekspor `process` menulis file yang sama dengan yang disimpan oleh penangkapan pada tingkat ini, di setiap mode pemrosesan. | DN Linear |
| `radiance` | Radian spektral absolut dari rantai radiometrik lengkap, dalam **W/m²/sr/nm**. Selalu ditulis sebagai TIFF 32-bit (`tiff32/Radiance_Images/`), terlepas dari format ekspor yang dipilih. | float32 |
| `reflectance` | Reflektansi ρ, di mana **DN 32768 = ρ 1,0 (100%)** dengan ruang gerak hingga ρ 2,0. Siap untuk Pix4D. | uint16 |
| `preview` | Render siap tampil: RGB=penyeimbangan putih + gamma; multispektral = peregangan warna palsu. | tampilan 8-bit |

### Orientasi

Berkas `raw` dan `debayered` selalu mempertahankan orientasi **asli** sensor; pengaturan pembalikan/rotasi kamera hanya diterapkan pada produk lainnya (pratinjau, radian, reflektansi, indeks). Penangkapan array mempertahankan file setiap anggotanya dalam orientasi asli kecuali Anda meminta perubahan orientasi (`--orient` di CLI, `orient=True` di SDK). Setiap berkas mencatat orientasi yang dimiliki oleh pikselnya dalam XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — dan `ChlorosOrientationApplied` menunjukkan apakah pikselnya sendiri menyertakan informasi tersebut (`false` pada berkas mentah dan berkas yang telah di-debayering). Chloros Image Viewer mengubah file mentah dan file yang telah melalui proses debayering berdasarkan catatan tersebut hanya untuk tujuan tampilan; file aslinya tidak pernah ditimpa. File yang tidak memiliki catatan (dibuat sebelum versi 1.3.0, atau oleh alat lain) ditampilkan sesuai dengan kondisi penyimpanannya.

## Masker Clamp

Produk radian dan reflektansi dari kamera multispektral M3C menjalankan pemisahan band 3×3. Jika pemisahan tersebut menghasilkan nilai negatif pada suatu band, nilai tersebut dipotong menjadi nol (tampilan langsung dipotong dengan cara yang persis sama). Fitur baru di versi 1.3.0: produk yang setidaknya satu pikselnya dipotong akan disertai dengan **masker pembatas** yang menunjukkan secara tepat piksel dan band mana saja yang terpengaruh:

* **Di mana:**di dalam folder `clamp_masks/`**di samping** folder produk, bukan di dalam folder tersebut: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`, dan `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **Apa:** file TIFF `uint8` satu pita pada kisi piksel produk itu sendiri (termasuk orientasi, ROI, binning, dan penyelarasan array). Bit 0 (`mask & 1`) ditetapkan di mana pita pada saluran R produk tersebut terpotong, bit 1 (`& 2`) pada saluran G, bit 2 (`& 4`) pada saluran B. Porsi piksel yang terpotong per band juga dicantumkan pada produk sebagai `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction`, dan `Chloros:BandBUnmixClampFraction`.
* **Bagaimana produk tersebut menampilkannya:** tag XMP `Chloros:UnmixClampMask` menyimpan jalur masker relatif terhadap folder produk (`../clamp_masks/<product folder>/<name>_clampmask.tif`, selalu dipisahkan oleh `/`), `none` jika tidak ada piksel yang terpotong (tidak ada masker yang ditulis), atau `unavailable` jika masker tidak dapat ditulis. Produk tanpa tag berarti proses unmix tidak pernah dijalankan (kamera RGB dan mono, level raw dan debayered).

Karena tidak ada folder produk yang menyimpan masker, Anda dapat menyerahkan folder produk seperti `Reflectance_Calibrated_Images/` ke Pix4D, Metashape, atau perangkat lunak fotogrametri lainnya apa adanya, termasuk subfoldernya — serahkan folder produk tersebut, bukan folder format di atasnya. Impor Chloros sendiri melewatkan masker clamp. CLI menghitungnya secara terpisah: `Image products written: N (plus M clamp masks)`.

## Produk indeks

Setiap indeks berada di foldernya sendiri, misalnya `<INDEX>_Index_Images/` (misalnya `NDVI_Index_Images/`).

* **Format bilangan bulat**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) menyimpan**perpanjangan linier per gambar**, bukan pemetaan −1…1: DN 0 adalah nilai indeks terendah pada bingkai tersebut dan skala penuh adalah nilai tertinggi. File tersebut berisi `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin`, dan `Chloros:IndexMax`, sehingga juga berisi `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Karena setiap bingkai memiliki titik akhir (endpoint) masing-masing, bandingkan bingkai berdasarkan nilainya, bukan berdasarkan DN.
* **`TIFF (32-bit, Percent)`** menyimpan nilai indeks itu sendiri sebagai float32, tanpa penskalaan ulang — pilihan terbaik untuk pekerjaan kuantitatif sepanjang penerbangan.

Fitur baru di versi 1.3.0: produk indeks mendeskripsikan **dirinya sendiri** alih-alih mengulang tag data mentah sumber. Tag GPS dan kameranya disalin dari produk yang digunakan untuk menghitungnya (produk reflektansi jika ada), dan kemudian:

| Tag | Nilai |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (sehingga berkas tersebut tidak pernah diimpor ulang sebagai data pengambilan) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / rumus yang digunakan |
| `Chloros:IndexLevel` | `reflectance` atau `radiance` — dari mana indeks tersebut dihitung (tidak ada jika keduanya tidak ada) |
| `Chloros:IlluminationInvariant` | `True` untuk indeks yang dihitung dari reflektansi, `False` dari radian atau nilai sensor |
| `Chloros:ChlorosPipelineVersion` | versi Chloros yang menulis indeks tersebut |

`Chloros:PixelScale` dihapus dari produk indeks — gunakan `IndexMin` / `IndexMax` di atas. Resep pembacaan lengkap terdapat di bawah "Raster indeks bilangan bulat adalah peregangan kontras per gambar" dalam [Referensi CLI](reference/cli-reference.md).

## Membaca nilai piksel reflektansi

Reflektansi disimpan sebagai angka digital bilangan bulat, dan **DN yang berarti ρ = 1,0 (reflektansi 100%) bergantung pada kamera sumber**:

| Kamera sumber | ρ = 1,0 adalah DN | Cara mengetahuinya |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (ruang kepala hingga ρ 2,0) | Tag XMP `Chloros:PixelScale=32768` tercantum pada file. |
| Survey3 | `65535` (dipotong pada ρ 1,0) | Tidak ada tag `Chloros:PixelScale` — ketiadaan tersebut merupakan tanda. (Produk Survey3 memang menyertakan tag pemrosesan `Chloros:*` lainnya, seperti `Chloros:CalibrationApplied`, di setiap mode pemrosesan mulai versi 1.3.0.) |

**Baca tag XMP `Chloros:PixelScale` dan bagi dengan nilainya** daripada mengasumsikan nilai konstan. Tag ini didefinisikan dalam domain uint16, sehingga nilainya tetap `32768` di seluruh format keluaran yang melakukan penskalaan ulang — normalisasikan tipe data yang disimpan kembali ke uint16 terlebih dahulu (×257 dari 8-bit, ×65535 dari float32).

{% hint style="warning" %}
**Pengambilan data 8-bit.** Ekspor dari pengambilan data 8-bit berformat 8-bit. Reflektansinya diskalakan ulang dan diberi label `Chloros:PixelScale=32768`, sehingga skalanya tetap terjaga; Produk DN 8-bit, serta berkas reflektansi yang ditulis sebelum versi 1.3.0, dipotong menjadi rentang 0–255 dan tidak menyertakan tag skala. Jika tag tersebut tidak ada pada berkas reflektansi LATTICE, jangan mengasumsikan adanya skala; ekspor ulang dalam format 16-bit atau 32-bit sebagai gantinya.
{% endhint %}

Untuk aturan lengkap (termasuk tag yang kompatibel dengan MicaSense), lihat **"Membaca piksel reflektansi"** dalam [Referensi CLI](reference/cli-reference.md).

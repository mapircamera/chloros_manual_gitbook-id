---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Kamera yang Didukung

Chloros memproses citra dari dua seri kamera MAPIR di **semua platform** (Windows, Linux amd64, dan Linux arm64/Jetson):

* **Survey3** — Kamera Survey3W (lebar) dan Survey3N (sempit). Masukan: `RAW+JPG`.
* **LATTICE**— modul kamera multispektral M3C dan M3M. Masukan: tangkapan `.tif`/`.tiff`. Kamera LATTICE juga dapat**dikendalikan secara langsung** dari Chloros — melalui tab Kamera di antarmuka pengguna grafis (GUI) (Windows) atau `chloros-cli lattice` / Python SDK (Windows dan Linux) — termasuk susunan multi-kamera yang tersinkronisasi. Lihat [panduan LATTICE](lattice/).

Pipa pemrosesan ini juga mendukung berkas masukan `.dng`.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Produsen</th><th width="250">Model Kamera</th><th width="138">Model Filter</th><th width="187">Jenis Gambar</th></tr></thead><tbody><tr><td><strong>Produsen</strong></td><td><strong>Model Kamera</strong></td><td><strong>Model Filter</strong></td><td><strong>Jenis Gambar</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

Seri LATTICE adalah sistem kamera multispektral modular yang dibangun berdasarkan sensor global-shutter Sony IMX265 (3,1 MP, piksel 3,45 µm). Setiap kamera menyimpan identitasnya sebagai string model:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
```

Chloros menampilkannya dengan awalan `LATT-` (misalnya `LATT-M3M-L41-F550`), dan string model ini mengendalikan seluruh proses selanjutnya — profil sensor, tata letak pita spektral, dan kalibrasi diselesaikan secara otomatis; tidak ada yang perlu dikonfigurasi per kamera. Nomor lensa menunjukkan **sudut pandang horizontal dalam derajat**: `L41` = sempit 41°, `L87` = lebar 87°.

Terdapat dua konfigurasi sensor:

| Konfigurasi | Sensor      | Jenis filter                           | Pita frekuensi per kamera                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Warna Bayer | Tiga bandpass                       | 3 pita spektral dari satu eksposur                                 |
| **M3M**       | Monokrom  | Filter interferensi pita sempit tunggal | 1 pita yang dikalibrasi — gabungkan beberapa kamera M3M untuk indeks vegetasi |

### Opsi filter M3C (Bayer)

| Filter | Pita (nama @ pusat nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 615/21 · Cyan 490/38 · NIR 808/14 |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

### Katalog filter M3M (mono) — 23 SKU

Nilai F adalah label SKU; rentang yang diukur (dicantumkan pada setiap produk yang telah dikalibrasi dan diekspor) adalah hasil pemindaian filter per lot:

| SKU    | Pusat (nm, diukur) | Tepi FWHM (nm) | Lebar (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (nominal)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (nominal)           | 867–893         | 26         |
| F905   | — (nominal)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_&quot;Batas pita diukur berdasarkan nilai lebar penuh pada setengah maksimum dari pemindaian filter per lot MAPIR — nilai yang sama yang dicantumkan Chloros dalam setiap ekspor yang telah dikalibrasi.&quot;_ &quot;— (nominal)&quot; = belum ada pemindaian lot; untuk SKU tersebut, titik tengah yang disebutkan adalah nomor SKU dan lebarnya adalah angka dari pabrikan.

† &quot;Reflektansi F988 dikalibrasi menggunakan panel reflektansi dalam adegan: pita tersebut berada di luar rentang terkalibrasi sensor cahaya DAQ, sehingga Chloros menerapkan tangkapan panel terbaru Anda dan mempertahankannya di antara pengamatan panel.&quot; Lihat [Target Kalibrasi](calibration-targets.md).

Untuk pengendalian kamera secara langsung, susunan panel, pengaturan jaringan, dan rantai pemrosesan radiometrik, lihat [panduan LATTICE](lattice/).

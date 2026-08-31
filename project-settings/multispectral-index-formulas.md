---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Rumus Indeks Multispektral

Rumus indeks di bawah ini menggunakan kombinasi rentang transmisi rata-rata filter Survey3:

<table><thead><tr><th align="center">Warna Filter Survey3</th><th width="196.199951171875" align="center">Nama Filter Survey3</th><th width="159.800048828125" align="center">Rentang Transmisi (FWHM)</th><th align="center">Transmisi Rata-rata</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Saat rumus-rumus ini digunakan, nama mungkin diakhiri dengan &quot;\_1&quot; atau &quot;\_2&quot;, yang menunjukkan filter mana yang digunakan, yaitu NIR, NIR1, atau NIR2.

Untuk kamera LATTICE M3C (Bayer triple-bandpass), mesin indeks yang sama menggunakan pita filter M3C:

| Filter M3C | Pita 1 (pusat/FWHM) | Pita 2 (pusat/FWHM) | Pita 3 (pusat/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

Kamera LATTICE M3M adalah kamera single-band (satu filter pita sempit per kamera), sehingga indeks multi-band tidak dihitung untuk gambar M3M tunggal. Untuk menghitung indeks dengan M3M, gabungkan dua atau lebih kamera menjadi tumpukan multi-band yang selaras dan gunakan mesin indeks LATTICE (`chloros-cli lattice index`, atau Kalkulator Indeks langsung pada antarmuka pengguna grafis).

***

## Di mana setiap nama indeks dapat digunakan

Chloros memiliki **tiga** permukaan indeks, dan daftar prasetelnya tidak identik. Gunakan bagian ini untuk memeriksa apakah suatu nama akan berfungsi di tempat yang Anda rencanakan untuk menggunakannya.

| Di mana Anda berada | Daftar mana yang berlaku | Jumlah |
| --- | --- | --- |
| Pengaturan Proyek → Indeks → Tambahkan indeks (GUI) | Permukaan 1 | 27 |
| Penampil Gambar [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Permukaan 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Permukaan 2 | 22 |
| SDK `process_folder(indices=[...])` | Permukaan 2 | 22 |
| `chloros-cli lattice index --preset` | Permukaan 3 | 22 (angka 22 yang berbeda) |
| Tab Kamera - Kalkulator Indeks Langsung | Surface 3 | 22 (22 yang berbeda) |

Surface 1 dan 2 memproses **satu gambar sekaligus dari satu kamera**, menggunakan slot simbol `x`/`y`/`z`(/`a`) yang terikat pada saluran filter kamera tersebut. Permukaan 3 bekerja pada**tumpukan multi-band yang diselaraskan** — beberapa kamera LATTICE yang didaftarkan bersama menjadi satu kubus — dan merujuk pada saluran berdasarkan nama huruf kecil.

### 1. Pengaturan Proyek GUI / Menu tarik-turun sandbox Penampil Gambar — 27 rumus

Menu tarik-turun tersebut mencantumkan rumus-rumus tersebut dalam urutan berikut (ini adalah urutan penyisipan, bukan urutan alfabetis):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

Di GUI, Anda menyeret saluran filter kamera ke slot band rumus, sehingga rumus apa pun dapat digunakan dengan penugasan band apa pun yang didukung kamera Anda. Rumus kustom yang telah Anda simpan ditambahkan di bawah daftar ini.

**Lima rumus khusus GUI** — yang tidak diterima oleh daftar CLI/SDK `--indices` — diimplementasikan sebagai:

| Preset khusus GUI | Rumus (sebagaimana diimplementasikan) | Slot |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (empat slot) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Pemetaan yang dimaksud untuk masing-masing diberikan pada bagian tersendiri di bagian bawah halaman ini (misalnya, GARI mengharapkan x=Green, y=NIR, z=Blue, a=Red). GARI adalah satu-satunya rumus dalam Chloros yang menggunakan slot keempat.

### 2. Perluasan nama CLI / SDK `--indices` — 22 preset

Opsi `chloros-cli process --indices` (dan parameter SDK `indices`) menerima nama preset berikut:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Nama indeks yang tidak dikenal akan dilewati tanpa pemberitahuan.** Nama di luar daftar ini (termasuk lima rumus khusus GUI: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`, dan rumus kustom apa pun yang Anda simpan di antarmuka pengguna) akan diabaikan dengan hanya pemberitahuan log — proses berjalan terus tanpa indeks tersebut, dan proses itu sendiri tetap melaporkan keberhasilan. Pemberitahuan tersebut ditampilkan sebagai:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Nama-nama tersebut dicocokkan tanpa membedakan huruf besar/kecil setelah spasi dihapus, sehingga `ndvi`, `NDVI`, dan ` NDVI ` merupakan preset yang sama. Sebuah preset juga akan dilewati jika memerlukan rentang spektral yang tidak disediakan oleh filter kamera Anda.
{% endhint %}

Rumus-rumus yang tepat sebagaimana diimplementasikan (simbol `x`/`y`/`z` adalah slot pita; pemetaan default ditampilkan per preset):

| Preset | Rumus (sebagaimana diterapkan) | Filter default | Slot (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Bagaimana nama preset menjadi posisi kanal

Saat Anda memasukkan nama mentah seperti `NDVI`, Chloros harus menentukan saluran mana dari file mana yang dibaca oleh setiap simbol. Sistem ini menggunakan tabel ini, yang memetakan kode filter ke posisi array setiap saluran:

| Kode filter | Saluran → indeks array |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` diterima sebagai alias untuk Orange, juga 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

**Filter default** preset (kolom &quot;Filter Default&quot; di atas) digunakan jika proyek berisi gambar dengan filter tersebut. Jika tidak, Chloros akan memeriksa filter-filter yang sebenarnya ada dalam proyek sesuai urutan `RGN, OCN, NGB, RGB, RE, NIR` dan memilih yang pertama yang dapat menyediakan semua kanal yang dibutuhkan oleh preset tersebut. Jika tidak ada yang bisa, preset tersebut tidak digunakan untuk proses tersebut. Inilah sebabnya mengapa `NDVI` yang diminta pada dataset yang hanya berisi OCN tetap menghasilkan hasil yang masuk akal — preset tersebut terikat pada posisi Orange dan NIR dari OCN.

String model LATTICE M3C membawa filter dengan awalan `F` (`LATT-M3C-L41-FRGN`), tetapi awalan tersebut dihilangkan saat kode filter dibaca dari gambar, sehingga kamera FRGN dapat mengidentifikasi melalui baris `RGN` di atasnya dan tidak memerlukan penanganan khusus.

### 3. Mesin indeks LATTICE (`lattice index --preset`, Kalkulator Indeks langsung) — 22 preset

Mesin LATTICE bekerja pada tumpukan multi-band yang telah disejajarkan (array langsung atau file TIFF multi-band yang diekspor) dan menggunakan nama saluran huruf kecil (`red`, `green`, `blue`, `red_edge`, `nir`). Daftar presetnya berbeda dari dua mesin di atas:

| Preset | Rumus | Saluran |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | merah, nir |
| GNDVI | `(nir - green) / (nir + green)` | hijau, nir |
| BNDVI | `(nir - blue) / (nir + blue)` | biru, nir |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | merah\_tepi, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | biru, hijau, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | merah, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | merah, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | merah, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | biru, merah, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | merah, nir |
| CVI | `(nir / green) - (red / green)` | merah, hijau, nir |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | merah, nir |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | merah, nir |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | merah, hijau, nir |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | merah, hijau, biru |
| NGRDI | `(green - red) / (green + red)` | merah, hijau |
| VARI | `(green - red) / (green + red - blue)` | merah, hijau, biru |
| TGI | `green - 0.39*red - 0.61*blue` | merah, hijau, biru |
| EXG | `2*green - red - blue` | merah, hijau, biru |
| CIRE | `(nir / red_edge) - 1` | tepi merah, nir |
| CIGREEN | `(nir / green) - 1` | hijau, nir |
| NDWI | `(green - nir) / (green + nir)` | hijau, nir |

Jalankan perintah `chloros-cli lattice index --list-presets` untuk mencetak tabel ini dari build yang terinstal, dan `--list-gradients` untuk gradien warna yang tersedia. Simbol saluran peka huruf besar-kecil dan harus sesuai dengan nama preset dalam huruf kecil (misalnya `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Sebagaimana diimplementasikan dalam antarmuka pengguna (GUI) dan daftar preset CLI/SDK, CVI adalah rumus rasio-dari-rasio:

$$
CVI = {(z / y) \over (x / y)}
$$

dengan pemetaan saluran default RGB: x=Red, y=Green, z=Blue. Di antarmuka pengguna grafis (GUI), Anda dapat menyeret saluran kamera mana pun ke slot x/y/z. Perhatikan bahwa preset `CVI` dari mesin indeks LATTICE menggunakan rumus yang berbeda, `(NIR / Green) - (Red / Green)` — periksa tabel di atas untuk permukaan yang Anda gunakan.

***

## ENDVI - Indeks Vegetasi Perbedaan Ternormalisasi yang Ditingkatkan

Indeks ini menggunakan saluran biru selain NIR dan hijau, serta populer di kalangan kamera yang difilter dengan NGB di mana pita biru menggantikan pita merah.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

Implementasinya adalah rumus simbol `((x+y)-(2*z))/((x+y)+(2*z))` — tetapkan kanal NIR dan Green kamera Anda ke slot x/y, serta Blue ke z (untuk kamera NGB: x=NIR, y=Green, z=Blue).

***

## EVI - Indeks Vegetasi yang Disempurnakan

Indeks ini awalnya dikembangkan untuk digunakan dengan data MODIS sebagai penyempurnaan dari NDVI dengan mengoptimalkan sinyal vegetasi di area dengan indeks luas daun (LAI) yang tinggi. Indeks ini paling berguna di wilayah dengan nilai LAI tinggi di mana NDVI mungkin mengalami saturasi. Indeks ini menggunakan wilayah reflektansi biru untuk mengoreksi sinyal latar belakang tanah dan mengurangi pengaruh atmosfer, termasuk hamburan aerosol.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Nilai EVI seharusnya berkisar antara 0 hingga 1 untuk piksel vegetasi. Fitur-fitur terang seperti awan dan bangunan putih, serta fitur-fitur gelap seperti air, dapat menyebabkan nilai piksel yang tidak normal dalam citra EVI. Sebelum membuat citra EVI, Anda harus menyaring awan dan fitur terang dari citra reflektansi, serta secara opsional menetapkan ambang batas nilai piksel dari 0 hingga 1.

_Referensi: Huete, A., dkk. &quot;Gambaran Umum Kinerja Radiometrik dan Biofisik Indeks Vegetasi MODIS.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indeks Tutupan Hutan 1

_Hanya tersedia di GUI — tidak tersedia sebagai preset CLI/SDK `--indices`._

Indeks ini membedakan kanopi hutan dari jenis vegetasi lainnya menggunakan citra reflektansi multispektral yang mencakup pita tepi merah.

$$
FCI1 = Red * RedEdge
$$

Kawasan berhutan akan memiliki nilai FCI1 yang lebih rendah karena reflektansi pohon yang lebih rendah dan adanya bayangan di dalam kanopi.

_Referensi: Becker, Sarah J., Craig S.T. Daughtry, dan Andrew L. Russ. &quot;Indeks tutupan hutan yang andal untuk citra multispektral.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indeks Tutupan Hutan 2

_Hanya tersedia di antarmuka pengguna (GUI) — tidak tersedia sebagai preset CLI/SDK `--indices` preset._

Indeks ini membedakan kanopi hutan dari jenis vegetasi lainnya menggunakan citra reflektansi multispektral yang tidak mencakup pita tepi merah.

$$
FCI2 = Red * NIR
$$

Kawasan berhutan akan memiliki nilai FCI2 yang lebih rendah karena reflektansi pohon yang lebih rendah dan adanya bayangan di dalam kanopi.

_Referensi: Becker, Sarah J., Craig S.T. Daughtry, dan Andrew L. Russ. &quot;Indeks tutupan hutan yang andal untuk citra multispektral.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Indeks Pemantauan Lingkungan Global

_Hanya tersedia di GUI — tidak tersedia sebagai preset CLI/SDK `--indices`._

Indeks vegetasi non-linier ini digunakan untuk pemantauan lingkungan global dari citra satelit dan berusaha mengoreksi efek atmosfer. Indeks ini mirip dengan NDVI tetapi kurang sensitif terhadap efek atmosfer. Indeks ini terpengaruh oleh tanah terbuka; oleh karena itu, tidak disarankan untuk digunakan di daerah dengan vegetasi yang jarang atau cukup padat.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Dimana:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referensi: Pinty, B., dan M. Verstraete. GEMI: Indeks Non-Linier untuk Memantau Vegetasi Global dari Satelit. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indeks Tahan Atmosfer

_Hanya tersedia di antarmuka pengguna (GUI) — tidak tersedia sebagai preset CLI/SDK `--indices`._

Indeks ini lebih sensitif terhadap rentang konsentrasi klorofil yang luas dan kurang sensitif terhadap pengaruh atmosfer dibandingkan NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konstanta gamma adalah fungsi pembobotan yang bergantung pada kondisi aerosol di atmosfer. ENVI menggunakan nilai 1,7, yang merupakan nilai yang direkomendasikan oleh Gitelson, Kaufman, dan Merzylak (1996, halaman 296).

_Referensi: Gitelson, A., Y. Kaufman, dan M. Merzylak. &quot;Penggunaan Saluran Green dalam Penginderaan Jauh Vegetasi Global dari EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Indeks Klorofil

Indeks ini digunakan untuk memperkirakan kandungan klorofil daun pada berbagai spesies tumbuhan.

$$
GCI = {NIR \over Green} - 1
$$

Penggunaan rentang panjang gelombang yang luas, termasuk NIR dan panjang gelombang hijau, memberikan prediksi kandungan klorofil yang lebih akurat sekaligus memungkinkan sensitivitas yang lebih tinggi dan rasio sinyal-ke-noise yang lebih baik.

_Referensi: Gitelson, A., Y. Gritz, dan M. Merzlyak. &quot;Hubungan Antara Kandungan Klorofil Daun dan Reflektansi Spektral serta Algoritma untuk Penilaian Klorofil Tanpa Merusak pada Daun Tumbuhan Tinggi.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## Indeks Daun GLI - Green

Indeks ini awalnya dirancang untuk digunakan dengan kamera digital RGB guna mengukur tutupan gandum, di mana nilai-nilai digital (DN) merah, hijau, dan biru berkisar antara 0 hingga 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Nilai GLI berkisar antara -1 hingga +1. Nilai negatif mewakili tanah dan fitur non-hidup, sedangkan nilai positif mewakili daun dan batang hijau.

_Referensi: Louhaichi, M., M. Borman, dan D. Johnson. &quot;Platform Berbasis Lokasi dan Fotografi Udara untuk Dokumentasi Dampak Penggembalaan terhadap Gandum.&quot; Geocarto International 16, No. 1 (2001): 65-70._

***

## GNDVI - Green Indeks Vegetasi Perbedaan Terstandarisasi

Indeks ini serupa dengan NDVI, kecuali bahwa indeks ini mengukur spektrum hijau dari 540 hingga 570 nm, bukan spektrum merah. Indeks ini lebih sensitif terhadap konsentrasi klorofil dibandingkan dengan NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referensi: Gitelson, A., dan M. Merzlyak. &quot;Penginderaan Jauh Konsentrasi Klorofil pada Daun Tumbuhan Tingkat Tinggi.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Indeks Vegetasi yang Disesuaikan dengan Tanah dan Dioptimalkan

Indeks ini awalnya dirancang menggunakan fotografi warna-inframerah untuk memprediksi kebutuhan nitrogen jagung. Indeks ini mirip dengan OSAVI, tetapi mengganti pita hijau dengan pita merah.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referensi: Sripada, R., dkk. &quot;Penentuan Kebutuhan Nitrogen Selama Musim Tanam untuk Jagung Menggunakan Fotografi Udara Berwarna-Inframerah.&quot; Disertasi Doktor, Universitas Negeri Carolina Utara, 2005._

***

## Rasio Indeks Vegetasi GRVI - Green

Indeks ini sensitif terhadap laju fotosintesis di kanopi hutan, karena reflektansi hijau dan merah sangat dipengaruhi oleh perubahan pigmen daun.

$$
GRVI = {NIR \over Green }
$$

_Referensi: Sripada, R., dkk. &quot;Fotografi Warna-Inframerah dari Udara untuk Menentukan Kebutuhan Nitrogen Awal Musim pada Jagung.&quot; Jurnal Agronomi 98 (2006): 968-977._

***

## GSAVI - Green Indeks Vegetasi yang Disesuaikan dengan Tanah

Indeks ini awalnya dirancang menggunakan fotografi warna-inframerah untuk memprediksi kebutuhan nitrogen pada jagung. Indeks ini mirip dengan SAVI, tetapi mengganti pita hijau dengan pita merah.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referensi: Sripada, R., dkk. &quot;Menentukan Kebutuhan Nitrogen Selama Musim Tanam untuk Jagung Menggunakan Fotografi Warna-Inframerah dari Udara.&quot; Disertasi Ph.D., Universitas Negeri Carolina Utara, 2005._

***

## LAI - Indeks Luas Daun

Indeks ini digunakan untuk memperkirakan tutupan dedaunan serta memprediksi pertumbuhan dan hasil panen tanaman. ENVI menghitung LAI hijau menggunakan rumus empiris berikut dari Boegh dkk. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Di mana EVI adalah:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Nilai LAI yang tinggi biasanya berkisar antara sekitar 0 hingga 3,5. Namun, ketika adegan mengandung awan dan fitur-fitur terang lainnya yang menghasilkan piksel jenuh, nilai LAI dapat melebihi 3,5. Idealnya, Anda harus menyembunyikan awan dan fitur-fitur terang dari adegan Anda sebelum membuat gambar LAI.

_Referensi: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, dan A. Thomsen. &quot;Data Multispektral dari Udara untuk Mengukur Indeks Luas Daun, Konsentrasi Nitrogen, dan Efisiensi Fotosintesis dalam Pertanian.&quot; Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI - Indeks Klorofil Daun

_Hanya tersedia di GUI — tidak tersedia sebagai preset CLI/SDK `--indices`._

Indeks ini digunakan untuk memperkirakan kandungan klorofil pada tumbuhan tingkat tinggi, yang sensitif terhadap variasi reflektansi yang disebabkan oleh penyerapan klorofil.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referensi: Datt, B. &quot;Penginderaan Jauh Kandungan Air pada Daun Eucalyptus.&quot; Journal of Plant Physiology 154, no. 1 (1999): 30-36._

***

## MNLI - Indeks Non-Linier yang Dimodifikasi

Indeks ini merupakan penyempurnaan dari Indeks Non-Linier (NLI) yang menggabungkan Indeks Vegetasi yang Disesuaikan dengan Tanah (SAVI) untuk memperhitungkan latar belakang tanah. ENVI menggunakan nilai faktor penyesuaian latar belakang kanopi (_L_) sebesar 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referensi: Yang, Z., P. Willis, dan R. Mueller. &quot;Dampak Citra AWIFS yang Disempurnakan dengan Rasio Pita terhadap Akurasi Klasifikasi Tanaman.&quot; Prosiding Simposium Penginderaan Jauh Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Indeks Vegetasi yang Disesuaikan dengan Tanah Modifikasi 2

Indeks ini merupakan versi yang lebih sederhana dari indeks MSAVI yang diusulkan oleh Qi, dkk. (1994), yang merupakan penyempurnaan dari Indeks Vegetasi yang Disesuaikan dengan Tanah (SAVI). Indeks ini mengurangi gangguan tanah dan meningkatkan rentang dinamis sinyal vegetasi. MSAVI2 didasarkan pada metode induktif yang tidak menggunakan nilai _L_ konstan (seperti pada SAVI) untuk menonjolkan vegetasi yang sehat.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referensi: Qi, J., A. Chehbouni, A. Huete, Y. Kerr, dan S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR - Modified Simple Ratio

Indeks ini merupakan modifikasi dari rasio sederhana NIR/Red yang dirancang untuk melinearkan hubungannya dengan parameter biofisik, dan lebih sensitif daripada NDVI pada kepadatan vegetasi yang lebih tinggi.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referensi: Chen, J. &quot;Evaluasi Indeks Vegetasi dan Rasio Sederhana yang Dimodifikasi untuk Aplikasi di Kawasan Boreal.&quot; Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE - Perbedaan Terstandarisasi RedEdge

Indeks ini mirip dengan NDVI, tetapi membandingkan kontras antara NIR dengan RedEdge, bukan dengan Red, yang sering mendeteksi vegetasi lebih awal.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indeks Perbedaan Terstandarisasi Vegetasi

Indeks ini merupakan ukuran vegetasi hijau yang sehat. Kombinasi antara rumus selisih ternormalisasi dan penggunaan wilayah penyerapan serta reflektansi klorofil tertinggi membuatnya tangguh dalam berbagai kondisi. Namun, indeks ini dapat mengalami saturasi pada kondisi vegetasi yang lebat ketika nilai LAI menjadi tinggi.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Nilai indeks ini berkisar antara -1 hingga 1. Rentang nilai yang umum untuk vegetasi hijau adalah 0,2 hingga 0,8.

_Referensi: Rouse, J., R. Haas, J. Schell, dan D. Deering. Pemantauan Sistem Vegetasi di Great Plains dengan ERTS. Simposium ERTS Ketiga, NASA (1973): 309-317._

***

## NLI - Indeks Non-Linier

Indeks ini mengasumsikan bahwa hubungan antara banyak indeks vegetasi dan parameter biofisik permukaan bersifat non-linier. Indeks ini melinierkan hubungan dengan parameter permukaan yang cenderung bersifat non-linier.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referensi: Goel, N., dan W. Qin. &quot;Pengaruh Arsitektur Kanopi terhadap Hubungan antara Berbagai Indeks Vegetasi dan LAI serta Fpar: Simulasi Komputer.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Indeks Vegetasi yang Disesuaikan dengan Tanah yang Dioptimalkan

Indeks ini didasarkan pada Indeks Vegetasi yang Disesuaikan dengan Tanah (SAVI). Indeks ini menggunakan nilai standar 0,16 sebagai faktor penyesuaian latar belakang kanopi. Rondeaux (1996) menetapkan bahwa nilai ini memberikan variasi tanah yang lebih besar daripada SAVI pada tutupan vegetasi rendah, sekaligus menunjukkan sensitivitas yang lebih tinggi terhadap tutupan vegetasi di atas 50%. Indeks ini paling baik digunakan di daerah dengan vegetasi yang relatif jarang di mana tanah terlihat melalui kanopi.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referensi: Rondeaux, G., M. Steven, dan F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Indeks Vegetasi Perbedaan yang Dinormalisasi Kembali

Indeks ini menggunakan selisih antara panjang gelombang inframerah dekat dan merah, bersama dengan NDVI, untuk menonjolkan vegetasi yang sehat. Indeks ini tidak terpengaruh oleh efek tanah dan geometri pengamatan matahari.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referensi: Roujean, J., dan F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indeks Vegetasi yang Disesuaikan dengan Tanah

Indeks ini mirip dengan NDVI, tetapi menekan pengaruh piksel tanah. Indeks ini menggunakan faktor penyesuaian latar belakang kanopi, _L_, yang merupakan fungsi dari kepadatan vegetasi dan sering kali memerlukan pengetahuan sebelumnya mengenai jumlah vegetasi. Huete (1988) menyarankan nilai optimal _L_=0,5 untuk memperhitungkan variasi latar belakang tanah orde pertama. Indeks ini paling baik digunakan di daerah dengan vegetasi yang relatif jarang, di mana tanah terlihat melalui kanopi.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referensi: Huete, A. &quot;A Soil-Adjusted Vegetation Index (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Indeks Vegetasi Selisih yang Ditransformasikan

Indeks ini berguna untuk memantau tutupan vegetasi di lingkungan perkotaan. Indeks ini tidak mengalami saturasi seperti NDVI dan SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referensi: Bannari, A., H. Asalhi, dan P. Teillet. &quot;Indeks Perbedaan Vegetasi yang Ditransformasikan (TDVI) untuk Pemetaan Tutupan Vegetasi&quot; Dalam Prosiding Simposium Geosains dan Penginderaan Jauh, IGARSS &#x27;02, IEEE International, Jilid 5 (2002)._

***

## VARI - Indeks Visible yang Tahan Terhadap Pengaruh Atmosfer

Indeks ini didasarkan pada ARVI dan digunakan untuk memperkirakan fraksi vegetasi dalam suatu adegan dengan sensitivitas rendah terhadap pengaruh atmosfer.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referensi: Gitelson, A., dkk. &quot;Garis Vegetasi dan Tanah dalam Ruang Spektral Cahaya Tampak: Sebuah Konsep dan Teknik untuk Estimasi Jarak Jauh Fraksi Vegetasi.&quot; Jurnal Internasional Penginderaan Jauh 23 (2002): 2537−2562._

***

## WDRVI - Indeks Vegetasi Rentang Dinamis Luas

Indeks ini serupa dengan NDVI, namun menggunakan koefisien bobot (_a_) untuk mengurangi perbedaan kontribusi sinyal inframerah dekat dan merah terhadap NDVI. WDRVI sangat efektif pada pemandangan dengan kepadatan vegetasi sedang hingga tinggi ketika nilai NDVI melebihi 0.6. Nilai NDVI cenderung mendatar saat fraksi vegetasi dan indeks luas daun (LAI) meningkat, sedangkan nilai WDRVI lebih sensitif terhadap rentang fraksi vegetasi yang lebih luas serta terhadap perubahan pada LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Koefisien bobot (_a_) dapat berkisar antara 0,1 hingga 0,2. Nilai 0,2 direkomendasikan oleh Henebry, Viña, dan Gitelson (2004).

_Referensi_

_Gitelson, A. &quot;Indeks Vegetasi Rentang Dinamis Luas untuk Kuantifikasi Jarak Jauh Karakteristik Biofisik Vegetasi.&quot; Journal of Plant Physiology 161, No. 2 (2004): 165-173._

_Henebry, G., A. Viña, dan A. Gitelson. &quot;Indeks Vegetasi Rentang Dinamis Luas dan Potensi Kegunaannya untuk Analisis Kesenjangan.&quot; Gap Analysis Bulletin 12: 50-56._

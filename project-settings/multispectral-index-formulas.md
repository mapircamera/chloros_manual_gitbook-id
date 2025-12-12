---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Rumus Indeks Multispektral

Rumus indeks di bawah ini menggunakan kombinasi rentang transmisi rata-rata filter Survey3:

<table><thead><tr><th align="center">Survey3 Warna Filter</th><th width="196.199951171875" align="center">Nama Filter Survey3</th><th width="159.800048828125" align="center">Rentang Transmisi (FWHM)</th><th align="center">Transmisi Rata-Rata</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850nm</td></tr></tbody></table>

Ketika rumus-rumus ini digunakan, nama mungkin berakhir dengan &quot;\_1&quot; atau &quot;\_2&quot;, yang sesuai dengan filter NIR mana yang digunakan, baik NIR1 atau NIR2.

***

## EVI - Indeks Vegetasi Terperbaiki

Indeks ini awalnya dikembangkan untuk digunakan dengan data MODIS sebagai perbaikan atas NDVI dengan mengoptimalkan sinyal vegetasi di daerah dengan indeks luas daun tinggi (LAI). Indeks ini paling berguna di wilayah dengan LAI tinggi di mana NDVI mungkin mengalami saturasi. Indeks ini menggunakan wilayah reflektansi biru untuk mengoreksi sinyal latar belakang tanah dan mengurangi pengaruh atmosfer, termasuk penyebaran aerosol.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Nilai EVI harus berkisar antara 0 hingga 1 untuk piksel vegetasi. Fitur terang seperti awan dan bangunan putih, serta fitur gelap seperti air, dapat menyebabkan nilai piksel abnormal dalam gambar EVI. Sebelum membuat gambar EVI, Anda harus menyaring awan dan fitur terang dari gambar reflektansi, dan secara opsional menetapkan ambang batas nilai piksel dari 0 hingga 1.

_Referensi: Huete, A., dkk. &quot;Ringkasan Kinerja Radiometrik dan Biofisik Indeks Vegetasi MODIS.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Indeks Tutupan Hutan 1

Indeks ini membedakan kanopi hutan dari jenis vegetasi lain menggunakan citra reflektansi multispektral yang mencakup band tepi merah.

$$
FCI1 = Red * RedEdge
$$

Area hutan akan memiliki nilai FCI1 yang lebih rendah karena reflektansi pohon yang lebih rendah dan adanya bayangan di dalam kanopi.

_Referensi: Becker, Sarah J., Craig S.T. Daughtry, dan Andrew L. Russ. &quot;Indeks tutupan hutan yang andal untuk citra multispektral.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Indeks Tutupan Hutan 2

Indeks ini membedakan kanopi hutan dari jenis vegetasi lain menggunakan citra reflektansi multispektral yang tidak termasuk band merah tepi.

$$
FCI2 = Red * NIR
$$

Area hutan akan memiliki nilai FCI2 yang lebih rendah karena reflektansi pohon yang lebih rendah dan adanya bayangan di dalam kanopi.

_Referensi: Becker, Sarah J., Craig S.T. Daughtry, dan Andrew L. Russ. &quot;Indeks tutupan hutan yang andal untuk citra multispektral.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Indeks Pemantauan Lingkungan Global

Indeks vegetasi non-linier ini digunakan untuk pemantauan lingkungan global dari citra satelit dan berusaha memperbaiki efek atmosfer. Indeks ini mirip dengan NDVI tetapi kurang sensitif terhadap efek atmosfer. Indeks ini terpengaruh oleh tanah terbuka; oleh karena itu, tidak disarankan untuk digunakan di area dengan vegetasi jarang atau sedang.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Di mana:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referensi: Pinty, B., dan M. Verstraete. GEMI: Sebuah Indeks Non-Linier untuk Memantau Vegetasi Global dari Satelit. Vegetation 101 (1992): 15-20._

***

## GARI - Green Indeks Tahan Atmosfer

Indeks ini lebih sensitif terhadap rentang konsentrasi klorofil yang luas dan kurang sensitif terhadap efek atmosfer dibandingkan NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

Konstanta gamma adalah fungsi bobot yang bergantung pada kondisi aerosol di atmosfer. ENVI menggunakan nilai 1,7, yang merupakan nilai yang direkomendasikan oleh Gitelson, Kaufman, dan Merzylak (1996, halaman 296).

_Referensi: Gitelson, A., Y. Kaufman, dan M. Merzylak. &quot;Penggunaan Saluran Green dalam Pemantauan Jarak Jauh Vegetasi Global dari EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Indeks Klorofil

Indeks ini digunakan untuk memperkirakan kandungan klorofil daun pada berbagai spesies tanaman.

$$
GCI = {NIR \over Green} - 1
$$

Memiliki rentang gelombang NIR dan hijau yang luas memberikan perkiraan kandungan klorofil yang lebih akurat sambil memungkinkan sensitivitas yang lebih tinggi dan rasio sinyal-ke-noise yang lebih baik.

_Referensi: Gitelson, A., Y. Gritz, dan M. Merzlyak. &quot;Hubungan Antara Kandungan Klorofil Daun dan Reflektansi Spektral serta Algoritma untuk Penilaian Klorofil Non-Destruktif pada Daun Tumbuhan Tinggi.&quot; Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI - Green Indeks Daun

Indeks ini awalnya dirancang untuk digunakan dengan kamera digital RGB untuk mengukur tutupan gandum, di mana angka digital merah, hijau, dan biru (DN) berkisar antara 0 hingga 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Nilai GLI berkisar antara -1 hingga +1. Nilai negatif mewakili tanah dan fitur non-hidup, sementara nilai positif mewakili daun dan batang hijau.

_Referensi: Louhaichi, M., M. Borman, dan D. Johnson. &quot;Platform dan Fotografi Udara yang Terletak Secara Spasial untuk Dokumentasi Dampak Pemotongan pada Gandum.&quot; Geocarto International 16, No. 1 (2001): 65-70._

***

## GNDVI - Green Indeks Vegetasi Perbedaan Normalized

Indeks ini serupa dengan NDVI, namun mengukur spektrum hijau dari 540 hingga 570 nm alih-alih spektrum merah. Indeks ini lebih sensitif terhadap konsentrasi klorofil dibandingkan NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referensi: Gitelson, A., dan M. Merzlyak. &quot;Pemantauan Jarak Jauh Konsentrasi Klorofil pada Daun Tumbuhan Tinggi.&quot; Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Indeks Vegetasi yang Dioptimalkan dengan Penyesuaian Tanah

Indeks ini awalnya dirancang menggunakan fotografi warna-inframerah untuk memprediksi kebutuhan nitrogen pada jagung. Indeks ini mirip dengan OSAVI, tetapi mengganti band hijau dengan band merah.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referensi: Sripada, R., dkk. &quot;Menentukan Kebutuhan Nitrogen Selama Musim Tanam untuk Jagung Menggunakan Fotografi Warna-Infrared Udara.&quot; Disertasi Ph.D., Universitas Negeri Carolina Utara, 2005._

***

## GRVI - Green Rasio Indeks Vegetasi

Indeks ini sensitif terhadap laju fotosintesis di kanopi hutan, karena reflektansi hijau dan merah sangat dipengaruhi oleh perubahan pigmen daun.

$$
GRVI = {NIR \over Green }
$$

_Referensi: Sripada, R., dkk. &quot;Fotografi Warna-Infrared Udara untuk Menentukan Kebutuhan Nitrogen Awal Selama Musim Tanam pada Jagung.&quot; Jurnal Agronomi 98 (2006): 968-977._

***

## GSAVI - Green Indeks Vegetasi yang Disesuaikan dengan Tanah

Indeks ini awalnya dirancang dengan fotografi warna-inframerah untuk memprediksi kebutuhan nitrogen jagung. Mirip dengan SAVI, tetapi mengganti band hijau dengan band merah.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referensi: Sripada, R., dkk. &quot;Menentukan Kebutuhan Nitrogen Selama Musim Tanam untuk Jagung Menggunakan Fotografi Warna-Inframerah Udara.&quot; Disertasi Ph.D., Universitas Negeri Carolina Utara, 2005._

***

## LAI - Indeks Luas Daun

Indeks ini digunakan untuk memperkirakan tutupan daun dan memprediksi pertumbuhan dan hasil panen tanaman. ENVI menghitung LAI hijau menggunakan rumus empiris berikut dari Boegh dkk. (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Dimana EVI adalah:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Nilai LAI yang tinggi biasanya berkisar antara 0 hingga 3,5. Namun, ketika adegan mengandung awan dan fitur terang lainnya yang menghasilkan piksel jenuh, nilai LAI dapat melebihi 3,5. Sebaiknya Anda menyaring awan dan fitur terang dari adegan Anda sebelum membuat gambar LAI.

_Referensi: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde, dan A. Thomsen. &quot;Data Multispektral Udara untuk Mengukur Indeks Luas Daun, Konsentrasi Nitrogen, dan Efisiensi Fotosintesis dalam Pertanian.&quot; Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI - Indeks Klorofil Daun

Indeks ini digunakan untuk memperkirakan kandungan klorofil pada tumbuhan tingkat tinggi, yang sensitif terhadap variasi reflektansi yang disebabkan oleh penyerapan klorofil.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referensi: Datt, B. &quot;Penginderaan Jauh Kandungan Air pada Daun Eucalyptus.&quot; Journal of Plant Physiology 154, no. 1 (1999): 30-36._

***

## MNLI - Indeks Non-Linier Modifikasi

Indeks ini merupakan peningkatan dari Indeks Non-Linier (NLI) yang menggabungkan Indeks Vegetasi yang Disesuaikan Tanah (SAVI) untuk memperhitungkan latar belakang tanah. ENVI menggunakan faktor penyesuaian latar belakang kanopi (_L_) sebesar 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referensi: Yang, Z., P. Willis, dan R. Mueller. &quot;Dampak Rasio Band yang Ditingkatkan pada Gambar AWIFS terhadap Akurasi Klasifikasi Tanaman.&quot; Prosiding Simposium Penginderaan Jauh Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Indeks Vegetasi yang Disesuaikan Tanah yang Dimodifikasi 2

Indeks ini adalah versi yang lebih sederhana dari indeks MSAVI yang diusulkan oleh Qi, dkk. (1994), yang memperbaiki Indeks Vegetasi yang Disesuaikan Tanah (SAVI). Indeks ini mengurangi kebisingan tanah dan meningkatkan rentang dinamis sinyal vegetasi. MSAVI2 didasarkan pada metode induktif yang tidak menggunakan nilai _L_ konstan (seperti pada SAVI) untuk menonjolkan vegetasi yang sehat.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referensi: Qi, J., A. Chehbouni, A. Huete, Y. Kerr, dan S. Sorooshian. &quot;Indeks Vegetasi yang Disesuaikan Tanah yang Dimodifikasi.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE- Indeks Perbedaan Normalized RedEdge

Indeks ini mirip dengan NDVI tetapi membandingkan kontras antara NIR dengan RedEdge daripada Red, yang sering mendeteksi stres vegetasi lebih awal.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Indeks Perbedaan Normalized Vegetasi

Indeks ini merupakan ukuran vegetasi hijau yang sehat. Kombinasi formulasi perbedaan normalisednya dan penggunaan wilayah penyerapan dan refleksi tertinggi klorofil membuatnya tangguh dalam berbagai kondisi. Namun, indeks ini dapat jenuh dalam kondisi vegetasi padat ketika LAI menjadi tinggi.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Nilai indeks ini berkisar antara -1 hingga 1. Rentang umum untuk vegetasi hijau adalah 0,2 hingga 0,8.

_Referensi: Rouse, J., R. Haas, J. Schell, dan D. Deering. Pemantauan Sistem Vegetasi di Great Plains dengan ERTS. Simposium ERTS Ketiga, NASA (1973): 309-317._

***

## NLI - Indeks Non-Linier

Indeks ini mengasumsikan bahwa hubungan antara banyak indeks vegetasi dan parameter biofisik permukaan bersifat non-linier. Indeks ini melinierkan hubungan dengan parameter permukaan yang cenderung non-linier.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referensi: Goel, N., dan W. Qin. &quot;Pengaruh Arsitektur Kanopi terhadap Hubungan Antara Berbagai Indeks Vegetasi dan LAI serta Fpar: Simulasi Komputer.&quot; Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Indeks Vegetasi yang Dioptimalkan dengan Penyesuaian Tanah

Indeks ini didasarkan pada Indeks Vegetasi yang Disesuaikan dengan Tanah (SAVI). Ia menggunakan nilai standar 0,16 untuk faktor penyesuaian latar belakang kanopi. Rondeaux (1996) menentukan bahwa nilai ini memberikan variasi tanah yang lebih besar daripada SAVI untuk tutupan vegetasi rendah, sambil menunjukkan sensitivitas yang lebih tinggi terhadap tutupan vegetasi di atas 50%. Indeks ini paling baik digunakan di daerah dengan vegetasi yang relatif jarang di mana tanah terlihat melalui kanopi.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referensi: Rondeaux, G., M. Steven, dan F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Indeks Vegetasi Perbedaan Renormalisasi

Indeks ini menggunakan perbedaan antara panjang gelombang inframerah dekat dan merah, bersama dengan NDVI, untuk menyoroti vegetasi yang sehat. Indeks ini tidak terpengaruh oleh efek tanah dan geometri pemantauan matahari.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referensi: Roujean, J., dan F. Breon. &quot;Estimating PAR Absorbed by Vegetation from Bidirectional Reflectance Measurements.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Indeks Vegetasi yang Disesuaikan dengan Tanah

Indeks ini mirip dengan NDVI, tetapi menekan efek piksel tanah. Ia menggunakan faktor penyesuaian latar belakang kanopi, _L_, yang merupakan fungsi dari kepadatan vegetasi dan seringkali memerlukan pengetahuan sebelumnya tentang jumlah vegetasi. Huete (1988) menyarankan nilai optimal _L_=0.5 untuk memperhitungkan variasi latar belakang tanah tingkat pertama. Indeks ini paling baik digunakan di area dengan vegetasi yang relatif jarang di mana tanah terlihat melalui kanopi.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referensi: Huete, A. &quot;Indeks Vegetasi yang Disesuaikan dengan Tanah (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Indeks Vegetasi Perbedaan yang Diubah

Indeks ini berguna untuk memantau tutupan vegetasi di lingkungan perkotaan. Ia tidak mengalami saturasi seperti NDVI dan SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referensi: Bannari, A., H. Asalhi, dan P. Teillet. &quot;Transformed Difference Vegetation Index (TDVI) untuk Pemetaan Tutupan Vegetasi&quot; Dalam Prosiding Simposium Geosains dan Penginderaan Jauh, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Indeks Visibel yang Tahan Atmosfer

Indeks ini didasarkan pada ARVI dan digunakan untuk memperkirakan fraksi vegetasi dalam suatu adegan dengan sensitivitas rendah terhadap efek atmosfer.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referensi: Gitelson, A., dkk. &quot;Garis Vegetasi dan Tanah dalam Ruang Spektral Visibel: Konsep dan Teknik untuk Estimasi Jarak Jauh Fraksi Vegetasi. Jurnal Internasional Penginderaan Jauh 23 (2002): 2537−2562._

***

## WDRVI - Indeks Vegetasi Rentang Dinamis Lebar

Indeks ini mirip dengan NDVI, tetapi menggunakan koefisien bobot (_a_) untuk mengurangi perbedaan kontribusi sinyal inframerah dekat dan merah terhadap NDVI. WDRVI sangat efektif dalam adegan dengan kepadatan vegetasi sedang hingga tinggi ketika NDVI melebihi 0,6. NDVI cenderung stabil saat fraksi vegetasi dan indeks luas daun (LAI) meningkat, sedangkan WDRVI lebih sensitif terhadap rentang fraksi vegetasi yang lebih luas dan perubahan dalam LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

Koefisien bobot (_a_) dapat berkisar antara 0,1 hingga 0,2. Nilai 0,2 direkomendasikan oleh Henebry, Viña, dan Gitelson (2004).

_Referensi_

_Gitelson, A. &quot;Indeks Vegetasi Rentang Dinamis Luas untuk Kuantifikasi Jarak Jauh Karakteristik Biofisik Vegetasi.&quot; Jurnal Fisiologi Tumbuhan 161, No. 2 (2004): 165-173._

_Henebry, G., A. Viña, dan A. Gitelson. &quot;Indeks Vegetasi Rentang Dinamis Luas dan Potensinya untuk Analisis Celah.&quot; Gap Analysis Bulletin 12: 50-56._

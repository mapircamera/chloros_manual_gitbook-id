---
description: Panel yang diukur di laboratorium digunakan untuk mengkalibrasi data yang diambil dalam pasca pemrosesan
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Target Kalibrasi

MAPIR menawarkan berbagai target kalibrasi untuk mencakup berbagai aplikasi. T4-R50 kompak yang terlihat di bawah berisi 4 panel yang telah diukur reflektansi cahayanya dari 250 - 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>

Target referensi difus T4 memiliki kurva reflektansi berikut, [unduh datanya di sini](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR T4 Reflektansi :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR T4 Reflektansi :: 400-1000nm</p></figcaption></figure>

Melihat grafik reflektansi, Anda dapat melihat bahwa nilainya adalah panjang gelombang (sumbu x) versus persen reflektansi (sumbu y). Saat kami menangkap gambar target kalibrasi, kami kemudian membuat hubungan antara nilai piksel dan persen reflektansi, dalam spektrum yang sensitif terhadap masing-masing pita sensor kamera.

Artinya, pada setiap gambar yang Anda ambil dengan kamera kami, Anda dapat menggunakan foto target reflektansi kami, seperti [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) atau [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) untuk mengkalibrasi gambar untuk reflektansi. Setelah dikalibrasi, setiap piksel dalam gambar sama dengan persen reflektansi.

Jika Anda mengeluarkan gambar yang dikalibrasi dalam Chloros sebagai JPG atau TIFF pada umumnya, maka persen reflektansi dihitung dengan membagi nilai piksel dengan kedalaman bit format gambar. Jadi untuk JPG dibagi 255, dan untuk TIFF dibagi 65.535. Anda juga dapat memilih output format PERCENT di Chloros, dan kemudian setiap piksel akan berkisar dari nilai persen 0,0 hingga 1,0 (reflektansi 0% hingga 100%). Perlu diingat bahwa beberapa aplikasi gambar tidak dapat menerima gambar persentase (floating point), dan ukurannya besar dalam hal penyimpanan.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

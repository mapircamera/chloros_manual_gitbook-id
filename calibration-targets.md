---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Target Kalibrasi

MAPIR menawarkan berbagai target kalibrasi untuk berbagai aplikasi. T4-R50 yang ringkas seperti yang terlihat di bawah ini terdiri dari 4 panel yang telah diukur reflektansi cahayanya pada rentang 250–2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Target referensi difus T4 memiliki kurva reflektansi sebagai berikut, [unduh data di sini](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Reflektansi T4 :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Reflektansi T4 :: 400-1000nm</p></figcaption></figure>Target referensi difus T4P memiliki kurva reflektansi sebagai berikut, [unduh data di sini](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Reflektansi T4P :: 250-2500nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Reflektansi T4P :: 400-1000nm</p></figcaption></figure>Dengan melihat grafik reflektansi, Anda dapat melihat bahwa nilainya adalah panjang gelombang (sumbu x) versus persentase reflektansi (sumbu y). Saat kami mengambil gambar target kalibrasi, kami kemudian membuat hubungan antara nilai piksel dan persentase reflektansi, dalam spektrum yang sensitif terhadap setiap pita sensor kamera.

Ini berarti bahwa dengan setiap gambar yang Anda ambil menggunakan kamera kami, Anda dapat menggunakan foto target reflektansi kami, seperti [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) atau [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) untuk mengkalibrasi gambar berdasarkan reflektansi. Setelah dikalibrasi, setiap piksel dalam gambar setara dengan persentase reflektansi.

Jika Anda mengekspor gambar yang telah dikalibrasi dalam format JPG standar atau format lain, persentase reflektansi dihitung dengan membagi nilai piksel dengan kedalaman bit format gambar. Jadi, untuk JPG dibagi 255, dan untuk format lain dibagi 65.535. Anda juga dapat memilih format keluaran PERCENT dalam Chloros, dan kemudian setiap piksel akan berkisar dari nilai persentase 0,0 hingga 1,0 (reflektansi 0% hingga 100%). Perlu diingat bahwa beberapa aplikasi gambar tidak dapat menerima gambar persentase (floating point), dan ukurannya besar dalam hal penyimpanan.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Target Kalibrasi

MAPIR menawarkan berbagai target kalibrasi untuk berbagai aplikasi. T4-R50 yang ringkas seperti yang terlihat di bawah ini terdiri dari 4 panel yang telah diukur reflektansi cahayanya pada rentang 250 - 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Target referensi difus T4 memiliki kurva reflektansi sebagai berikut, [unduh data di sini](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Reflektansi T4 :: 250-2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Reflektansi T4 :: 400–1.000 nm</p></figcaption></figure>Target referensi difus T4P memiliki kurva reflektansi sebagai berikut, [unduh data di sini](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Reflektansi T4P :: 250–2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Reflektansi T4P :: 400–1000 nm</p></figcaption></figure>Dengan melihat grafik reflektansi, Anda dapat melihat bahwa nilai-nilai tersebut menunjukkan panjang gelombang (sumbu x) terhadap persentase reflektansi (sumbu y). Saat kami mengambil gambar target kalibrasi, kami kemudian membuat hubungan antara nilai piksel dan persentase reflektansi, dalam spektrum yang sensitif terhadap setiap pita sensor kamera.

Ini berarti bahwa pada setiap gambar yang Anda ambil dengan kamera kami, Anda dapat menggunakan foto target reflektansi kami, seperti [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) atau [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125) untuk mengkalibrasi gambar berdasarkan reflektansi. Setelah dikalibrasi, setiap piksel dalam gambar akan setara dengan persentase reflektansi.

Untuk **Survey3** , jika Anda mengekspor gambar yang telah dikalibrasi dalam format Chloros sebagai JPG standar atau TIFF, maka persentase reflektansi dihitung dengan membagi nilai piksel dengan kedalaman bit format gambar. Jadi, untuk JPG dibagi 255, dan untuk TIFF dibagi 65.535. Anda juga dapat memilih format keluaran PERCENT pada Chloros, dan setiap piksel akan berkisar antara nilai persentase 0,0 hingga 1,0 (reflektansi 0% hingga 100%). Perlu diingat bahwa beberapa aplikasi gambar tidak dapat menerima gambar dalam format persentase (bilangan floating point), dan ukuran penyimpanannya pun besar.

{% hint style="info" %}
**Reflektansi LATTICE menggunakan skala piksel yang berbeda.** Reflektansi LATTICE disimpan dengan DN 32768 = 100% reflektansi (bukan 65535), dan setiap file menyertakan tag XMP `Chloros:PixelScale` yang menyatakan skalanya. Baca tag tersebut dan bagi dengan nilainya, bukan mengasumsikan nilai konstan — lihat [Format Gambar Keluaran](output-image-formats.md).
{% endhint %}

## Target kalibrasi dengan kamera LATTICE

Pada kamera LATTICE, target kalibrasi bersifat **opsional** untuk reflektansi: Chloros dapat merujuk reflektansi ke irradian turun yang diukur oleh sensor cahaya DAQ (ρ = π·L/E). Referensi ini dipilih melalui pengaturan sumber reflektansi (Pengaturan Proyek di antarmuka pengguna; `--reflectance-source` di CLI; `reflectance_source` di SDK):

| Nilai | Perilaku |
| --- | --- |
| `auto` *(default)* | Target dalam bingkai yang lulus QA adalah **referensi absolut**; jika tidak ada target atau QA gagal, Chloros beralih ke pembagian sinyal turun DAQ. |
| `target` | Hanya target ketat — tidak ada penggantian DAQ. |
| `daq` | Otoritatif DAQ — pengukuran downwelling selalu menjadi referensi. |

Perilaku target tambahan untuk LATTICE:

* **Geometri target** — Panel bertanda ArUco, panel ROI tetap, dan target strip semuanya didukung; geometri berasal dari konfigurasi target proyek.
* **Data target yang diukur per unit** — `--target-reflectance-dir DIR` mengarah ke direktori pemindaian reflektansi target yang diukur per unit (`<serial>.csv`, dicari berdasarkan nomor seri/QR unit target). Jika tidak ditemukan, Chloros akan menggunakan spektrum T3/T4P nominal sebagai pengganti.
* **Penetapan waktu** — target yang terdeteksi mengkalibrasi bingkai di sekitarnya dan dipertahankan di antara pengamatan target.

Semantik dan contoh lengkap bendera terdapat di [Referensi CLI](reference/cli-reference.md) (lihat &quot;Toggle Ekspor Per-Produk&quot;).

### F988

&quot;Reflektansi F988 dikalibrasi menggunakan panel reflektansi dalam adegan: pita tersebut berada di luar rentang kalibrasi sensor cahaya DAQ, sehingga Chloros menerapkan penangkapan panel terbaru Anda dan mempertahankannya di antara pengamatan panel.&quot;

Jika F988 dijalankan dengan kalibrasi berbasis DAQ saja, Chloros akan menolak nilai reflektansi berbasis DAQ untuk pita tersebut dan menjelaskan alasannya (alasan pengabaian `dls-uncalibrated-band-988`); alur kerja panel adalah metode yang didukung.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>

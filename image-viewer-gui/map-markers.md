# Penanda Peta

Tab Peta menampilkan gambar Anda pada peta interaktif 2D berdasarkan koordinat GPS-nya. Hal ini memberikan gambaran geografis tentang sesi pengambilan gambar Anda dan membantu Anda memvisualisasikan cakupan spasial. Fitur ini juga berguna saat pertama kali mengimpor gambar untuk dengan cepat menghapus gambar yang tidak perlu diproses.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Akses Tab Peta

1. Buka atau buat proyek di Chloros
2. Impor gambar yang mengandung metadata GPS
3. Klik tab **Peta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> di bilah sisi kiri
4. Peta akan menampilkan penanda di setiap lokasi GPS gambar

{% hint style="info" %}
**GPS Diperlukan**: Hanya gambar yang memiliki koordinat GPS tertanam dalam metadata EXIF-nya yang akan muncul di peta. Pastikan kamera Anda memiliki GPS yang diaktifkan saat pengambilan gambar.
{% endhint %}

***

## Mengatur Gambar dari Tab Peta

Tab **Peta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> memiliki fungsi tambah  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  dan hapus  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  seperti yang ada di [**File Browser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Tab ini juga menampilkan daftar tabel file proyek yang sama tetapi dengan header kolom yang berbeda:

### Nama File

* Nama file asli dari kamera
* Mempertahankan konvensi penamaan kamera (misalnya, IMG\_0001.RAW)

### Lintang

* Lintang gambar

### Bujur

* Bujur gambar

### Ketinggian

* Ketinggian gambar

{% hint style="info" %}
Mengklik header kolom tabel juga akan mengurutkan data baris
{% endhint %}

***

## Penanda Gambar

Setiap gambar dengan data GPS diwakili oleh penanda di peta:

### Tampilan Penanda

* Penanda menunjukkan koordinat GPS tepat di mana setiap gambar diambil
* Penanda yang berdekatan mungkin digabungkan saat diperbesar
* Perbesar untuk melihat lokasi gambar individu

{% hint style="success" %}
SUPER-ZOOM: Saat Anda mencapai tingkat zoom maksimum dari penyedia ubin peta, ubin tersebut akan diperbesar saat diperbesar lebih lanjut, memungkinkan Anda melihat penanda yang berdekatan.
{% endhint %}

### Pratinjau Saat Mengarahkan Kursor

* **Arahkan kursor mouse** ke atas penanda mana pun untuk melihat pratinjau miniatur gambar tersebut
* Ini memungkinkan identifikasi visual cepat tanpa meninggalkan tampilan peta
* Berguna untuk menemukan gambar spesifik dalam sesi pengambilan gambar yang besar

***

## Penyedia Ubin Peta

{% hint style="success" %}
**Pemilihan Otomatis**: Chloros secara otomatis memilih layanan ubin yang menyediakan tingkat zoom terbaik untuk lokasi peta Anda saat ini. Anda dapat beralih secara manual antara penyedia jika diinginkan.
{% endhint %}

Tab Peta mendukung dua penyedia ubin untuk gambar latar belakang peta:

### Google Maps

* Gambar satelit dan peta standar dari Google
* Terbaik untuk cakupan global yang luas

### ESRI

* Gambar satelit dan udara dari ESRI ArcGIS
* Seringkali menyediakan gambar beresolusi lebih tinggi di wilayah tertentu

***

## Jenis Ubin Peta

Anda dapat memilih jenis lapisan peta (dari kiri ke kanan):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="line">### Terrain

Menampilkan profil elevasi dan ubin peta dengan detail (jalan, dll)

### Map

Menampilkan ubin peta standar (bandwidth rendah) dengan detail (jalan, dll)

### Satellite

Menampilkan ubin peta satelit dengan detail tinggi (bandwidth tinggi)

### Hybrid

Menampilkan ubin peta satelit dengan detail tambahan (jalan, dll)

***

## Navigasi Peta

### Kontrol Zoom

* **Zoom In/Out**: Gunakan roda gulir mouse atau tombol zoom
* **Layar Penuh**: Tampilkan peta dalam mode layar penuh

### Kontrol Geser

* **Geser**: Klik dan seret untuk berpindah di peta

***

## Kasus Penggunaan

### Visualisasi Jalur Terbang

* Lihat area cakupan sesi penangkapan drone
* Identifikasi celah dalam cakupan gambar
* Verifikasi pelaksanaan jalur terbang

### Ulasan Survei Darat

* Lihat distribusi spasial penangkapan berbasis darat
* Temukan gambar target kalibrasi relatif terhadap area survei
* Rencanakan lokasi penangkapan tambahan

### Kontrol Kualitas

* Identifikasi dengan cepat gambar yang diambil di lokasi yang tidak terduga
* Verifikasi akurasi GPS di seluruh dataset
* Bandingkan lokasi gambar dengan catatan lapangan

***

## Pemecahan Masalah

### Tidak Ada Marker yang Muncul

**Penyebab kemungkinan:**

* Gambar tidak mengandung metadata GPS
* GPS dinonaktifkan pada kamera selama pengambilan
* Data EXIF dihapus oleh perangkat lunak eksternal

**Solusi:** Verifikasi GPS diaktifkan pada kamera dan impor ulang file asli

### Marker di Lokasi yang Salah

**Penyebab kemungkinan:**

* GPS kamera memiliki sinyal satelit yang buruk
* Pergeseran GPS selama pengambilan

**Solusi:** Ini biasanya masalah saat pengambilan; pertimbangkan menggunakan GPS PPK/RTK untuk aplikasi presisi

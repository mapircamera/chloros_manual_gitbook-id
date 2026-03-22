# Penanda Peta

Tab Peta menampilkan gambar-gambar Anda pada peta 2D interaktif berdasarkan koordinat GPS-nya. Fitur ini memberikan gambaran geografis tentang sesi pengambilan gambar Anda dan membantu Anda memvisualisasikan cakupan spasial. Fitur ini juga berguna saat pertama kali mengimpor gambar untuk dengan cepat menghapus gambar-gambar yang tidak perlu diproses.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Mengakses Tab Peta

1. Buka atau buat proyek di Chloros
2. Impor gambar yang mengandung metadata GPS
3. Klik tab **Peta** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> di bilah sisi kiri
4. Peta akan menampilkan penanda di setiap lokasi GPS gambar

{% hint style="info" %}
**GPS Diperlukan**: Hanya gambar dengan koordinat GPS yang tertanam dalam metadata EXIF-nya yang akan muncul di peta. Pastikan kamera Anda mengaktifkan GPS saat pengambilan gambar.
{% endhint %}

***

## Mengatur Gambar dari Tab Peta

Tab **Peta**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> memiliki tombol tambah  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  dan hapus  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  seperti yang ada di tab [**File Browser**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Tab ini juga menampilkan daftar tabel file proyek yang sama, namun dengan judul kolom yang berbeda:

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
Mengklik judul kolom tabel juga akan mengurutkan data baris
{% endhint %}

***

## Penanda Gambar

Setiap gambar dengan data GPS diwakili oleh penanda di peta:

### Tampilan Penanda

* Penanda menunjukkan koordinat GPS tepat di mana setiap gambar diambil
* Penanda yang berdekatan mungkin akan digabungkan saat diperkecil
* Perbesar untuk melihat lokasi gambar secara individual

{% hint style="success" %}
SUPER-ZOOM: Saat Anda mencapai tingkat perbesaran maksimum dari penyedia ubin peta, ubin tersebut akan diperbesar saat Anda memperbesar lebih lanjut, sehingga Anda dapat melihat penanda yang berdekatan.
{% endhint %}

### Pratinjau Saat Mengarahkan Kursor

* **Arahkan kursor mouse** ke penanda mana pun untuk melihat pratinjau miniatur gambar tersebut
* Ini memungkinkan identifikasi visual cepat tanpa meninggalkan tampilan peta
* Berguna untuk menemukan gambar spesifik dalam sesi pengambilan gambar yang besar

***

## Penyedia Ubin Peta

{% hint style="success" %}
**Pemilihan Otomatis**: Chloros secara otomatis memilih layanan ubin yang menyediakan tingkat zoom terbaik untuk lokasi peta Anda saat ini. Anda dapat beralih secara manual antar penyedia jika diinginkan.
{% endhint %}

Tab Peta mendukung dua penyedia ubin untuk gambar latar belakang peta:

### Google Maps

* Gambar satelit dan peta standar dari Google
* Terbaik untuk cakupan global secara umum

### ESRI

* Gambar satelit dan udara dari ESRI ArcGIS
* Seringkali menyediakan gambar dengan resolusi lebih tinggi di wilayah tertentu

***

## Jenis Ubin Peta

Anda dapat memilih jenis lapisan peta (dari kiri ke kanan):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terrain

Menampilkan profil ketinggian dan ubin peta dengan detail (jalan, dll)

### Map

Menampilkan ubin peta standar (bandwidth rendah) dengan detail (jalan, dll)

### Satellite

Menampilkan ubin peta satelit detail (bandwidth tinggi)

### Hybrid

Menampilkan ubin peta satelit dengan detail tambahan (jalan, dll)

***

## Navigasi Peta

### Kontrol Zoom

* **Zoom In/Out**: Gunakan roda gulir mouse atau tombol zoom
* **Layar Penuh**: Tampilkan peta dalam layar penuh

### Kontrol Geser

* **Geser**: Klik dan seret untuk bergerak di sekitar peta***

## Kasus Penggunaan

### Visualisasi Jalur Penerbangan

* Lihat area cakupan sesi pengambilan gambar drone
* Identifikasi celah dalam cakupan gambar
* Verifikasi pelaksanaan jalur penerbangan

### Tinjauan Survei Darat

* Lihat distribusi spasial pengambilan gambar berbasis darat
* Temukan gambar target kalibrasi relatif terhadap area survei
* Rencanakan lokasi pengambilan gambar tambahan

### Kontrol Kualitas

* Identifikasi dengan cepat gambar yang diambil di lokasi yang tidak terduga
* Verifikasi akurasi GPS di seluruh dataset
* Bandingkan lokasi gambar dengan catatan lapangan

***

## Pemecahan Masalah

### Tidak Ada Penanda yang Muncul

**Penyebab kemungkinan:**

* Gambar tidak mengandung metadata GPS
* GPS dinonaktifkan pada kamera selama pengambilan
* Data EXIF dihapus oleh perangkat lunak eksternal

**Solusi**: Pastikan GPS diaktifkan pada kamera Anda dan impor ulang file asli

### Penanda Berada di Lokasi yang Salah

**Penyebab yang mungkin:**

* GPS kamera memiliki sinyal satelit yang lemah
* GPS mengalami drift selama pengambilan gambar

**Solusi**: Ini biasanya merupakan masalah saat pengambilan gambar; pertimbangkan untuk menggunakan GPS PPK/RTK untuk aplikasi presisi

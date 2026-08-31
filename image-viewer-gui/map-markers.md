# Penanda Peta

Tab Peta menampilkan gambar-gambar Anda pada peta 2D interaktif berdasarkan koordinat GPS-nya. Tab ini memberikan gambaran geografis secara keseluruhan mengenai sesi pengambilan gambar dan merupakan cara tercepat—segera setelah impor—untuk menghapus gambar yang tidak ingin Anda olah.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Mengakses Tab Peta

1. Buka atau buat proyek di Chloros
2. Impor gambar yang mengandung metadata GPS
3. Klik tab **Peta** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> di bilah sisi kiri
4. Peta akan menampilkan penanda di setiap lokasi GPS gambar

{% hint style="info" %}
**GPS wajib**: hanya gambar dengan koordinat GPS dalam metadata EXIF-nya yang muncul di peta. Gambar tanpa koordinat tetap ada di dalam proyek dan tetap diproses seperti biasa — hanya saja tidak memiliki penanda.
{% endhint %}

***

## Menyesuaikan Gambar dari Tab Peta

Tab **Peta**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> memiliki tombol-tombol file yang sama, yaitu tombol tambah <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> dan hapus <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">, seperti pada tab [**Penjelajah File**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Tab ini menampilkan daftar file proyek yang sama, dengan kolom-kolom geografis:

| Kolom        | Isi                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nama**      | Nama berkas sebagaimana diambil dari kamera                             |
| **Lintang**  | Derajat desimal, enam tempat desimal                                |
| **Bujur** | Derajat desimal, enam tempat desimal                                |
| **Ketinggian** | Meter, satu tempat desimal — `-` jika gambar tidak memiliki data ketinggian |

{% hint style="info" %}
Klik judul kolom mana pun untuk mengurutkan berdasarkan kolom tersebut; klik lagi untuk membalik urutannya.
{% endhint %}

{% hint style="warning" %}
**Ketinggian adalah ketinggian di atas permukaan laut, bukan ketinggian di atas tanah.** Nilai ini berasal dari tag EXIF `GPSAltitude` gambar, yang merujuk pada permukaan laut rata-rata. Ini bukan ketinggian penerbangan di atas permukaan tanah, dan Chloros tidak akan menghitung jarak sampel di atas permukaan tanah (GSD) darinya — di atas ladang yang berada 300 m di atas permukaan laut, drone yang terbang pada ketinggian 100 m AGL akan mencatat jarak sekitar 400 m di sini. Gunakan kolom ini untuk mengidentifikasi nilai yang menyimpang dan memastikan ketinggian penerbangan yang konsisten, bukan sebagai pengukuran AGL.
{% endhint %}

***

## Penanda Gambar

Setiap gambar dengan data GPS akan memiliki penanda pada koordinatnya.

### Tampilan penanda

* Penanda berada tepat pada koordinat yang tercatat untuk setiap pengambilan gambar
* Penanda yang berdekatan mungkin tampak tumpang tindih saat diperkecil — perbesar tampilan untuk memisahkannya
* Penanda yang dipilih dan disorot ditampilkan di atas penanda lainnya

### Pratinjau saat mengarahkan kursor

* **Arahkan kursor** ke penanda mana pun untuk menampilkan pratinjau gambar tersebut beserta nama filenya
* **Klik**penanda untuk memilih gambar dan**menjepit** jendela popup agar tetap terbuka — jendela tersebut akan tetap ada hingga Anda mengklik di tempat lain. Selama jendela popup dijepit, mengarahkan kursor ke penanda lain tidak akan menutupnya
* Ini adalah cara cepat untuk menemukan satu bingkai tertentu dalam sesi besar tanpa meninggalkan peta

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>Tab Peta menampilkan setiap gambar yang dilengkapi geotag dalam proyek</p></figcaption></figure>### Super-zoom

{% hint style="success" %}
**SUPER-ZOOM**: ketika Anda mencapai tingkat zoom maksimum yang didukung oleh penyedia ubin, memperbesar lebih lanjut akan memperbesar ubin alih-alih berhenti, sehingga Anda dapat membedakan penanda yang letaknya hampir saling bertumpukan.
{% endhint %}

* Fitur super-zoom hanya aktif ketika Anda berada **tepat pada** tingkat zoom maksimum penyedia untuk lokasi tersebut dan ubin telah selesai dimuat. Di bawah tingkat tersebut, perbesaran berfungsi seperti biasa
* Rentang perbesaran adalah **1× hingga 32×** di atas tingkat zoom maksimum penyedia itu sendiri
* Indikator di sudut menunjukkan tingkat super-zoom saat ini dalam bentuk persentase, dan tombol **×** di sebelahnya akan mengembalikan Anda ke zoom normal dengan satu klik
* Memperkecil tampilan selalu diteruskan ke peta itu sendiri, sehingga Anda tidak akan pernah terjebak dalam mode super-zoom
* Memperbesar dan menggeser peta saat dalam mode super-zoom akan meneruskan pergeseran yang dihasilkan ke peta, sehingga area di luar pusat yang Anda pindahkan akan terus meminta ubin alih-alih menjadi kosong
* Penanda digambar sebagai elemen vektor, bukan raster, sehingga tetap tajam di setiap tingkat super-zoom

***

## Penyedia Ubin Peta

{% hint style="success" %}
**Pemilihan otomatis**: Chloros memilih layanan ubin yang menawarkan tingkat zoom terbaik untuk lokasi gambar Anda. Anda dapat beralih secara manual kapan saja.
{% endhint %}

| Penyedia        | Catatan                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Cakupan luas di seluruh dunia; mendukung keempat jenis ubin                                                                                                            |
| **Esri ArcGIS**| Seringkali memiliki citra udara beresolusi lebih tinggi di wilayah tertentu. Jenis ubin**Terrain** tidak ditawarkan untuk Esri dan tombolnya dinonaktifkan saat Esri dipilih |***

## Jenis Ubin Peta

Pilih jenis lapisan peta dengan tombol-tombol berikut (dari kiri ke kanan):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Jenis                 | Menampilkan                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terrain**          | Bayangan ketinggian dengan detail peta (jalan, label). Hanya Google       |
| **Map**              | Ubin peta jalan standar — opsi dengan bandwidth terendah              |
| **Satelit**        | Citra satelit terperinci, tanpa label — opsi dengan bandwidth tertinggi |
| **Hibrida** (default) | Citra satelit dengan jalan dan label yang ditampilkan di atasnya                |

Tab Peta terbuka pada mode **Hibrida**. Pilihan Anda akan diterapkan saat berpindah penyedia, jika penyedia tersebut mendukungnya.***

## Navigasi Peta

* **Zoom**: roda gulir mouse, atau tombol zoom di peta
* **Geser**: klik dan seret
* **Layar Penuh**: kontrol layar penuh memperluas peta hingga memenuhi seluruh jendela***

## Kasus Penggunaan

### Peninjauan Jalur Penerbangan

* Lihat area cakupan sesi drone secara sekilas
* Temukan celah di mana lintasan penerbangan terlewat
* Pastikan penerbangan mengikuti pola yang direncanakan

### Peninjauan survei darat

* Lihat bagaimana distribusi tangkapan gambar berbasis darat
* Temukan bingkai target kalibrasi relatif terhadap area survei
* Tentukan di mana tangkapan tambahan diperlukan

### Pengendalian Kualitas

* Temukan gambar yang diambil di lokasi tak terduga dan hapus sebelum diproses
* Urutkan berdasarkan Ketinggian untuk menemukan bingkai yang diambil pada ketinggian yang salah, atau yang sinyal GPS-nya buruk
* Bandingkan lokasi gambar dengan catatan lapangan

***

## Pemecahan Masalah

### Tidak ada penanda yang muncul

**Kemungkinan penyebab**

* Gambar tidak memiliki metadata GPS
* GPS dinonaktifkan pada kamera saat pengambilan gambar
* Data EXIF dihapus oleh perangkat lunak lain sebelum diimpor

**Apa yang harus dilakukan**: pastikan GPS diaktifkan pada kamera dan impor ulang file aslinya. Anda dapat memeriksa apakah file tertentu memiliki koordinat dengan mencarinya di tabel file pada tab Peta — gambar tanpa koordinat tidak akan muncul di sana.

### Penanda berada di lokasi yang salah

**Penyebab yang mungkin**: sinyal satelit yang buruk saat pengambilan gambar, atau penyimpangan GPS selama sesi.**Apa yang harus dilakukan**: ini adalah masalah pada saat pengambilan gambar, bukan sesuatu yang dapat diperbaiki oleh Chloros setelahnya. Untuk pekerjaan yang membutuhkan presisi, gunakan alur kerja GPS PPK/RTK — lihat pengaturan**Terapkan koreksi PPK** di [Pengaturan Proyek](../project-settings/project-settings.md).

### Peta kosong atau ubin berhenti dimuat

Penyedia ubin adalah layanan daring. Jika ubin berhenti masuk, periksa koneksi jaringan perangkat, lalu coba ganti penyedia. Jika Anda sedang memperbesar peta secara ekstrem, tekan tombol reset **×** untuk kembali ke tingkat zoom normal dan biarkan peta meminta ulang ubin.***

## Halaman terkait

* [**Kisi Gambar**](image-grid.md) — kumpulan gambar yang sama dengan thumbnail
* [**Membuka Gambar dalam Layar Penuh**](opening-an-image-full-screen.md) — memeriksa satu gambar secara detail
* [**Menambahkan File ke Proyek**](../processing-images-gui/adding-files-to-a-project.md) — tombol tambah/hapus file yang terdapat di tab ini

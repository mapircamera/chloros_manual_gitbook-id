# Menambahkan File ke Proyek

Setelah Anda membuat atau membuka proyek di Chloros, langkah selanjutnya adalah menambahkan gambar multispektral Anda untuk memulai pemrosesan. Tab File Browser<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> memudahkan Anda mengimpor gambar dan mengelola kumpulan data Anda.

## Mengakses File Browser

1. Buka atau buat proyek di Chloros
2. Klik ikon **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> di bilah sisi kiri
3. Panel Penjelajah File akan menampilkan daftar file proyek Anda

{% hint style="info" %}
**Tipe File yang Didukung**: Chloros mendukung file gambar RAW+JPG dan JPG dari kamera MAPIR, Survey3W, dan Survey3N. Hanya RAW+JPG yang direkomendasikan.
{% endhint %}

***

## Menambahkan Gambar ke Proyek Anda

Ada dua cara utama untuk menambahkan gambar ke proyek Anda:

### Metode 1: Tambahkan File

Gunakan opsi ini untuk mengimpor file gambar tunggal atau sejumlah kecil file.

1. Klik tombol **&quot;Tambahkan File&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> di bagian atas panel File Browser
2. Arahkan ke folder yang berisi gambar Anda
3. Pilih satu atau lebih file gambar (tahan **Ctrl** untuk memilih beberapa file)
4. Klik **&quot;Open&quot;** untuk mengimpor file yang dipilih

### Metode 2: Tambahkan Folder

Gunakan opsi ini untuk mengimpor semua gambar dari sebuah folder sekaligus.

1. Klik tombol **&quot;Tambahkan Folder&quot;** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> di bagian atas panel Penjelajah File
2. Buka dan pilih folder yang berisi gambar sesi pemotretan Anda
3. Klik **&quot;Pilih Folder&quot;** untuk mengimpor semua gambar yang didukung dari folder tersebut***

## Memahami Tabel Penjelajah File

Setelah gambar diimpor, gambar tersebut akan muncul dalam tabel dengan kolom-kolom berikut:

### Nama File

* Nama file asli dari kamera
* Mempertahankan konvensi penamaan kamera (misalnya, IMG\_0001.RAW)

### Cap Waktu

* Tanggal dan waktu gambar diambil
* Diambil dari metadata EXIF gambar
* Digunakan untuk sinkronisasi PPK dan deteksi target kalibrasi

### Model Kamera

* Konfigurasi kamera dan filter yang terdeteksi secara otomatis
* Contoh: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Digunakan untuk menerapkan profil pemrosesan yang tepat

### Kolom Target (Kotak Centang)

* Centang kotak ini untuk gambar yang mengandung target kalibrasi
* Sangat mempercepat deteksi target selama pemrosesan
* Lihat [Memilih Gambar Target](choosing-target-images.md) untuk detailnya

### Menampilkan Metadata Gambar

Mengklik tombol toggle di pojok kanan atas di atas tabel akan menampilkan metadata gambar yang dipilih di area grid gambar.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Mengelola File dalam Proyek Anda

### Menghapus File

Untuk menghapus gambar yang tidak diinginkan dari proyek Anda:

1. Pilih satu atau lebih gambar di tabel File Browser
2. Klik tombol **&quot;Hapus yang Terpilih&quot;** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> 3. Konfirmasikan penghapusan (file tidak dihapus dari disk, hanya dihapus dari proyek)

### Penyortiran dan Penyaringan

* **Penyortiran berdasarkan kolom**: Klik header kolom mana pun untuk menyortir gambar
* **Penyortiran berdasarkan tanggal dan waktu**: Berguna untuk mengatur urutan pengambilan gambar secara kronologis
* **Penyaringan berdasarkan model kamera**: Kelompokkan gambar berdasarkan jenis kamera jika menggunakan beberapa kamera***

## Pratinjau Gambar

### Melihat Gambar Penuh

Klik thumbnail gambar apa pun di File Browser untuk menampilkannya di area pratinjau utama:

1. Gambar muncul di panel pratinjau tengah
2. Gunakan kontrol zoom untuk memeriksa detail gambar
3. Berpindah antar gambar menggunakan tombol panah

### Navigasi Cepat

* **Gambar Sebelumnya**: Klik panah kiri atau tekan tombol ←
* **Gambar Berikutnya**: Klik panah kanan atau tekan tombol →
* **Perbesar/Perkecil**: Gunakan roda mouse atau tombol zoom
* **Geser**: Klik dan seret pada gambar saat diperbesar***

## Penanganan File Duplikat

Chloros secara otomatis mendeteksi dan mengabaikan file duplikat:

* File dengan nama file yang identik akan dilewati
* Mencegah pemrosesan ganda yang tidak disengaja
* Pesan peringatan ditampilkan saat file duplikat terdeteksi

{% hint style="warning" %}
**Penting**: Jangan mengganti nama atau memodifikasi file gambar asli Anda sebelum mengimpor. Chloros bergantung pada nama file asli dan metadata untuk pemrosesan yang benar.
{% endhint %}

***

## Kumpulan Data Kamera Campuran

Jika proyek Anda berisi gambar dari beberapa kamera MAPIR:

1. Chloros secara otomatis mendeteksi setiap model kamera
2. Setiap jenis kamera diproses dengan profil kalibrasi yang sesuai
3. File Browser menampilkan model kamera di kolom Model Kamera
4. Pemrosesan menerapkan pengaturan yang benar untuk setiap jenis kamera

**Contoh skenario**: Konfigurasi dua kamera Survey3W RGN + Survey3N OCN***

## Praktik Terbaik

### Atur Sebelum Impor

* Simpan gambar target kalibrasi di folder yang sama dengan gambar survei
* Pertahankan struktur folder asli dari kamera/kartu SD Anda
* Jangan mencampur dataset dari sesi yang berbeda dalam satu proyek

### Penamaan File

* Pertahankan nama file kamera asli (IMG\_0001.RAW, dll.)
* Jangan mengganti nama file sebelum impor
* Nama asli mengandung metadata penting

### Gambar Target Kalibrasi

* Selalu sertakan 1-2 gambar target kalibrasi per sesi
* Ambil gambar target sebelum dan setelah sesi pengambilan gambar
* Letakkan target dalam kondisi pencahayaan yang sama dengan area pengambilan gambar
* Tandai gambar target menggunakan kotak centang Target untuk mempercepat pemrosesan

***

## Masalah Umum dan Solusinya

### Gambar Tidak Muncul Setelah Impor

**Penyebab yang mungkin:**

* Format file tidak didukung (hanya RAW+JPG dan JPG dari kamera MAPIR)
* Gambar berasal dari kamera non-MAPIR (lihat [Kamera yang Didukung](../supported-cameras.md))
* File rusak atau transfer dari kartu SD tidak lengkap

**Solusi**: Periksa kompatibilitas format file dan model kamera

### Model Kamera Tidak Terdeteksi

**Penyebab yang mungkin:**

* Metadata EXIF dimodifikasi
* Gambar diedit menggunakan perangkat lunak eksternal
* Transfer file tidak lengkap

**Solusi**: Impor ulang file asli yang belum dimodifikasi dari kamera/kartu SD

### Stempel Waktu Hilang

**Penyebab yang mungkin:**

* Jam kamera tidak disetel dengan benar
* Data EXIF dihapus oleh perangkat lunak eksternal

**Solusi**: Pastikan pengaturan waktu kamera benar saat pengambilan gambar***

## Langkah Selanjutnya

Setelah file Anda diimpor:

1. **Periksa daftar file** - Pastikan semua gambar dimuat dengan benar
2. **Periksa model kamera** - Pastikan deteksi kamera benar
3. **Tandai gambar target** - Lihat [Memilih Gambar Target](choosing-target-images.md)
4. **Sesuaikan pengaturan** - Konfigurasikan opsi pemrosesan di [Pengaturan Proyek](adjusting-project-settings.md)
5. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)

Untuk informasi terperinci tentang konfigurasi proyek, lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md).

# Menambahkan Berkas ke Proyek

Setelah Anda membuat atau membuka proyek di Chloros, langkah berikutnya adalah menambahkan gambar multispektral Anda untuk memulai pemrosesan. Pengelola Berkas<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> memudahkan Anda untuk mengimpor gambar dan mengelola dataset Anda.

## Mengakses File Browser

1. Buka atau buat proyek di Chloros
2. Klik ikon **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> di bilah sisi kiri
3. Panel File Browser akan menampilkan daftar file proyek Anda

{% hint style=&quot;info&quot; %}
**Tipe File yang Didukung**: Chloros mendukung file gambar RAW+JPG dan JPG dari kamera MAPIR Survey3W dan Survey3N. Hanya RAW+JPG yang direkomendasikan.
{% endhint %}

***

## Menambahkan Gambar ke Proyek Anda

Ada dua cara utama untuk menambahkan gambar ke proyek Anda:

### Metode 1: Tambahkan Berkas

Gunakan opsi ini untuk mengimpor berkas gambar individu atau sejumlah kecil berkas.

1. Klik tombol **&quot;Tambahkan File&quot;** di bagian atas panel File Browser
2. Navigasi ke folder yang berisi gambar Anda
3. Pilih satu atau lebih file gambar (tahan **Ctrl** untuk memilih beberapa file)
4. Klik **&quot;Buka&quot;** untuk mengimpor file yang dipilih

### Metode 2: Tambahkan Folder

Gunakan opsi ini untuk mengimpor semua gambar dari folder sekaligus.

1. Klik tombol **&quot;Tambahkan Folder&quot;** di bagian atas panel File Browser
2. Navigasi ke dan pilih folder yang berisi gambar sesi penangkapan Anda
3. Klik **&quot;Pilih Folder&quot;** untuk mengimpor semua gambar yang didukung dari folder tersebut

***

## Memahami Tabel File Browser

Setelah gambar diimpor, mereka akan muncul dalam tabel dengan kolom-kolom berikut:

### Thumbnail

* Pratinjau kecil dari setiap gambar
* Klik thumbnail untuk melihat gambar penuh di area pratinjau utama

### Nama File

* Nama file asli dari kamera
* Mempertahankan konvensi penamaan kamera (misalnya, IMG\_0001.RAW)

### Tanggal dan Waktu

* Tanggal dan waktu gambar diambil
* Diambil dari metadata EXIF gambar
* Digunakan untuk sinkronisasi PPK dan deteksi target kalibrasi

### Model Kamera

* Konfigurasi kamera dan filter yang terdeteksi secara otomatis
* Contoh: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Digunakan untuk menerapkan profil pemrosesan yang benar

### Kolom Target (Kotak Centang)

* Centang kotak ini untuk gambar yang mengandung target kalibrasi
* Sangat mempercepat deteksi target selama pemrosesan
* Lihat [Memilih Gambar Target](choosing-target-images.md) untuk detail

***

## Mengelola File dalam Proyek Anda

### Menghapus File

Untuk menghapus gambar yang tidak diinginkan dari proyek Anda:

1. Pilih satu atau lebih gambar di tabel File Browser
2. Klik tombol **&quot;Hapus yang Terpilih&quot;**
3. Konfirmasi penghapusan (file tidak dihapus dari disk, hanya dihapus dari proyek)

### Pengurutan dan Penyaringan

* **Urutkan berdasarkan kolom**: Klik header kolom mana pun untuk mengurutkan gambar
* **Pemisahan berdasarkan waktu**: Berguna untuk mengorganisir urutan pengambilan gambar secara kronologis
* **Filter model kamera**: Kelompokkan gambar berdasarkan jenis kamera jika menggunakan beberapa kamera

***

## Pratinjau Gambar

### Menampilkan Gambar Penuh

Klik thumbnail gambar apa pun di File Browser untuk menampilkannya di area pratinjau utama:

1. Gambar muncul di panel pratinjau tengah
2. Gunakan kontrol zoom untuk memeriksa detail gambar
3. Navigasi antar gambar menggunakan tombol panah

### Navigasi Cepat

* **Gambar Sebelumnya**: Klik panah kiri atau tekan tombol ←
* **Gambar Berikutnya**: Klik panah kanan atau tekan tombol →
* **Zoom In/Out**: Gunakan roda mouse atau tombol zoom
* **Pan**: Klik dan seret pada gambar saat diperbesar

***

## Penanganan File Duplikat

Chloros secara otomatis mendeteksi dan mengabaikan file duplikat:

* File dengan nama file yang identik akan dilewati
* Mencegah pemrosesan ganda yang tidak disengaja
* Pesan peringatan ditampilkan saat duplikat terdeteksi

{% hint style=&quot;warning&quot; %}
**Penting**: Jangan mengganti nama atau memodifikasi file gambar asli sebelum mengimpor. Chloros bergantung pada nama file asli dan metadata untuk pemrosesan yang benar.
{% endhint %}

***

## Kumpulan Data Kamera Campuran

Jika proyek Anda mengandung gambar dari beberapa kamera MAPIR:

1. Chloros secara otomatis mendeteksi setiap model kamera
2. Setiap jenis kamera diproses dengan profil kalibrasi yang sesuai
3. File Browser menampilkan model kamera di kolom Model Kamera
4. Pengolahan menerapkan pengaturan yang benar untuk setiap jenis kamera

**Contoh skenario**: Survey3W RGN + Survey3N OCN konfigurasi kamera ganda

***

## Praktik Terbaik

### Organisasi Sebelum Impor

* Simpan gambar target kalibrasi di folder yang sama dengan gambar survei
* Pertahankan struktur folder asli dari kamera/kartu SD Anda
* Jangan mencampur dataset dari sesi yang berbeda dalam satu proyek

### Penamaan File

* Pertahankan nama file kamera asli (IMG\_0001.RAW, dll.)
* Jangan mengganti nama file sebelum impor
* Nama asli mengandung metadata penting

### Gambar Target Kalibrasi

* Selalu sertakan 1-2 gambar target kalibrasi per sesi
* Ambil target sebelum dan setelah sesi pengambilan gambar
* Letakkan target dalam kondisi pencahayaan yang sama dengan area pengambilan gambar
* Tandai gambar target menggunakan kotak centang Target untuk mempercepat pemrosesan

***

## Masalah Umum dan Solusinya

### Gambar Tidak Muncul Setelah Impor

**Penyebab kemungkinan:**

* Format file tidak didukung (hanya RAW+JPG dan JPG dari kamera MAPIR)
* Gambar berasal dari kamera non-MAPIR (lihat [Kamera yang Didukung](../supported-cameras.md))
* File rusak atau transfer tidak lengkap dari kartu SD

**Solusi:** Verifikasi kompatibilitas format file dan model kamera

### Model Kamera Tidak Terdeteksi

**Penyebab kemungkinan:**

* Metadata EXIF yang dimodifikasi
* Gambar diedit menggunakan perangkat lunak eksternal
* Transfer file tidak lengkap

**Solusi:** Impor ulang file asli yang belum dimodifikasi dari kamera/kartu SD

### Tanggal dan Waktu Hilang

**Penyebab kemungkinan:**

* Jam kamera tidak disetel dengan benar
* Data EXIF dihapus oleh perangkat lunak eksternal

**Solusi**: Verifikasi pengaturan waktu kamera benar saat pengambilan gambar

***

## Langkah Selanjutnya

Setelah file diimpor:

1. **Periksa daftar file** - Pastikan semua gambar dimuat dengan benar
2. **Periksa model kamera** - Verifikasi deteksi kamera yang benar
3. **Tandai gambar target** - Lihat [Memilih Gambar Target](choosing-target-images.md)
4. **Sesuaikan pengaturan** - Konfigurasikan opsi pemrosesan di [Pengaturan Proyek](adjusting-project-settings.md)
5. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)

Untuk informasi detail tentang konfigurasi proyek, lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md).

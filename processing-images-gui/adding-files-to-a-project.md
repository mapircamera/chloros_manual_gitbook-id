# Menambahkan File ke Proyek

Setelah Anda membuat atau membuka proyek di Chloros, langkah selanjutnya adalah menambahkan citra multispektral Anda untuk memulai pemrosesan. Tab **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> memudahkan Anda mengimpor citra dan mengelola kumpulan data Anda.

## Mengakses File Browser

1. Buka atau buat proyek di Chloros
2. Klik ikon **File Browser** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> di bilah sisi kiri
3. Panel File Browser akan menampilkan daftar file proyek Anda

{% hint style="info" %}
**Tipe File yang Didukung**:

* **Survey3W / Survey3N**: Pasangan RAW+JPG dan gambar JPG (disarankan RAW+JPG)
* **LATTICE**: Rekaman `.tif` / `.tiff` — direkam oleh kontrol kamera Chloros atau oleh hub LATTICE
* **Data sensor cahaya**: Rekaman `.daq` (DAQ-U/M/E) dan log sinar turun DAQ-M `.csv` — diimpor bersamaan dengan citra untuk mengkalibrasi reflektansi
{% endhint %}

***

## Menambahkan Gambar ke Proyek Anda

Ada dua cara utama untuk menambahkan gambar ke proyek Anda:

### Metode 1: Tambahkan File

Gunakan opsi ini untuk mengimpor file gambar individual atau sejumlah kecil file.

1. Klik tombol **&quot;Tambahkan File&quot;** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> di bagian atas panel Penjelajah File
2. Buka folder yang berisi gambar Anda
3. Pilih satu atau lebih file gambar (tahan tombol **Ctrl** untuk memilih beberapa file)
4. Klik **&quot;Buka&quot;** untuk mengimpor file yang dipilih

### Metode 2: Tambahkan Folder

Gunakan opsi ini untuk mengimpor semua gambar dari sebuah folder sekaligus. Anda dapat memilih **beberapa folder** dalam satu jendela dialog.

1. Klik tombol **&quot;Add Folder&quot;** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> di bagian atas panel File Browser
2. Buka dan pilih folder yang berisi gambar sesi pengambilan Anda
3. Klik **&quot;Select Folder&quot;** untuk mengimpor semua gambar yang didukung

{% hint style="info" %}
**File yang gagal dimuat akan dilaporkan.** Jika sebuah folder berisi file yang dikenali oleh Chloros tetapi tidak dapat dimuat, peringatan akan muncul — gambar tidak akan hilang begitu saja dari kisi.
{% endhint %}

***

## Mengimpor Folder LATTICE Capture

Hasil tangkapan LATTICE disimpan dengan **satu subfolder per tingkat ekspor** — misalnya `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — dengan berkas downwelling yang sesuai, yaitu `.daq`, di direktori akar:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Arahkan Folder ke direktori akar captures** (`output/` di atas). Jika folder yang dipilih tidak berisi gambar apa pun tetapi memiliki subfolder, Chloros akan menelusuri subfolder tersebut secara otomatis — subfolder di tingkat tersebut dan folder akar `.daq` semuanya diambil sekaligus.**Cara impor tangkapan:*** Setiap tangkapan diimpor sebagai **satu gambar tunggal**, dikelompokkan berdasarkan tangkapan (bukan satu entri per tingkat). Tingkat-tingkat lain dari tangkapan yang sama muncul sebagai mode tampilan dari gambar tersebut.
* **Pemrosesan selalu dimulai dari bingkai mentah.** Tingkat-tingkat lainnya dapat dilihat, tetapi hanya `raw` yang diproses melalui alur kerja — memproses ulang produk yang sudah diproses akan menerapkan koreksi dua kali, sehingga Chloros ditolak. Ekspor yang diimpor ulang tidak pernah dapat mengambil slot mentah suatu pengambilan gambar.
* Folder pengambilan gambar yang disimpan **tanpa** impor mentah akan ditampilkan secara normal, tetapi pemrosesan akan melewatinya dan mencatatnya dalam log. (Bendera CLI `--input-level` dapat memaksa titik masuk untuk kasus ini — lihat [Referensi CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**Sesi hub LATTICE** melakukan impor dengan cara yang sama: arahkan &quot;Add Folder&quot; ke folder sesi yang disalin dari hub (folder tersebut berisi `raw/` dan `previews/`), beserta log downwelling DAQ-M `.csv` apa pun. Jika kalibrasi kamera atau DAQ belum disimpan dalam cache di komputer Anda, Chloros akan mengambilnya secara otomatis berdasarkan nomor seri saat impor (memerlukan koneksi internet sekali saja).***

## Memahami Tabel Penjelajah File

Setelah gambar diimpor, gambar tersebut akan muncul dalam tabel dengan kolom-kolom berikut:

### Nama File

* Nama file asli dari kamera
* Mempertahankan konvensi penamaan kamera (misalnya, IMG\_0001.RAW atau capture\_20260816\_101500\_SN213800234\_raw.tif)

### Cap Waktu

* Tanggal dan waktu pengambilan gambar
* Diambil dari metadata EXIF gambar
* Digunakan untuk pencocokan sensor cahaya, sinkronisasi PPK, dan penjadwalan target kalibrasi

### Model Kamera

* Konfigurasi kamera dan filter yang terdeteksi secara otomatis
* Contoh Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Contoh LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Digunakan untuk menerapkan profil pemrosesan yang benar

### Kolom Target (Kotak Centang)

* Centang kotak ini untuk gambar yang berisi target kalibrasi
* Jika setidaknya satu gambar dicentang, **hanya gambar yang dicentang yang akan dipindai** untuk mencari target
* Lihat [Memilih Gambar Target](choosing-target-images.md) untuk detailnya

### Melihat Metadata Gambar

Mengklik tombol toggle di sudut kanan atas di atas tabel akan menampilkan metadata gambar yang dipilih di area kisi gambar.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Berkas Sensor Cahaya dalam Proyek Anda

* Berkas `.daq` dan `.csv` muncul dalam daftar Penjelajah Berkas tetapi bukan gambar yang dapat diklik — berkas-berkas ini menyediakan irradiansi ke bawah untuk kalibrasi reflektansi.
* Setiap berkas `.daq`/`.csv` yang diimpor tercantum di **Pengaturan Proyek → Sensor Cahaya DAQ**, di mana Anda dapat memeriksa koreksi tutup difuser yang berlaku untuk setiap berkas. Lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md).
* Rekaman yang Anda buat di tab **Sensor Cahaya** akan ditambahkan ke proyek yang terbuka secara otomatis — tidak perlu mengimpor secara manual.***

## Mengelola Berkas dalam Proyek Anda

### Menghapus File

Untuk menghapus gambar yang tidak diinginkan dari proyek Anda:

1. Pilih satu atau lebih gambar di tabel Penjelajah File
2. Klik tombol **&quot;Hapus yang Dipilih&quot;** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Konfirmasikan penghapusan (file tidak dihapus dari disk, hanya dihapus dari proyek)

### Mengurutkan dan Memfilter

* **Mengurutkan berdasarkan kolom**: Klik header kolom mana pun untuk mengurutkan gambar
* **Pengurutan berdasarkan tanggal dan waktu**: Berguna untuk mengatur urutan pengambilan gambar secara kronologis
* **Filter model kamera**: Kelompokkan gambar berdasarkan jenis kamera jika menggunakan beberapa kamera***

## Pratinjau Gambar

### Melihat Gambar Penuh

Klik thumbnail gambar mana pun di File Browser untuk menampilkannya di area pratinjau utama:

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
* Pesan peringatan ditampilkan saat duplikat terdeteksi

{% hint style="warning" %}
**Penting**: Jangan mengganti nama atau memodifikasi berkas gambar asli Anda sebelum mengimpor. Chloros mengandalkan nama berkas dan metadata asli untuk pemrosesan yang tepat.
{% endhint %}

***

## Kumpulan Data Kamera Campuran

Jika proyek Anda berisi gambar dari beberapa kamera MAPIR:

1. Chloros secara otomatis mendeteksi setiap model kamera — Survey3, LATTICE, atau campuran
2. Setiap jenis kamera diproses dengan profil kalibrasi yang sesuai
3. File Browser menampilkan model kamera di kolom Model Kamera
4. Setiap kamera mendapatkan pohon folder outputnya sendiri saat diproses

**Contoh skenario**: Konfigurasi dua kamera Survey3W dan RGN + Survey3N dan OCN, atau susunan LATTICE dengan master RGB dan beberapa modul pita sempit***

## Praktik Terbaik

### Atur Sebelum Mengimpor

* Simpan gambar target kalibrasi di folder yang sama dengan gambar survei
* Simpan file sensor cahaya `.daq` / `.csv` dari setiap sesi pengambilan gambar bersama dengan citra dari sesi tersebut
* Pertahankan struktur folder asli dari kamera/kartu SD/hub Anda
* Jangan mencampur kumpulan data dari sesi yang berbeda dalam satu proyek

### Penamaan File

* Pertahankan nama file asli dari kamera (IMG\_0001.RAW, capture\_..., dll.)
* Jangan mengganti nama file sebelum mengimpor
* Nama asli berisi metadata penting

### Gambar Target Kalibrasi

* Selalu sertakan 1–2 gambar target kalibrasi per sesi (Survey3; untuk LATTICE, rekaman DAQ dapat digunakan sebagai pengganti — lihat [Memilih Gambar Target](choosing-target-images.md))
* Ambil gambar target sebelum dan sesudah sesi pengambilan gambar
* Tempatkan target pada kondisi pencahayaan yang sama dengan area pengambilan gambar
* Tandai gambar target menggunakan kotak centang Target

***

## Masalah Umum dan Solusinya

### Gambar Tidak Muncul Setelah Diimpor

**Kemungkinan penyebab:**

* Format file tidak didukung (lihat daftar jenis yang didukung di bagian atas halaman ini)
* Gambar berasal dari kamera non-MAPIR (lihat [Kamera yang Didukung](../supported-cameras.md))
* File rusak atau transfer dari kartu SD tidak lengkap

**Solusi**: Pastikan kompatibilitas format file dan model kamera, serta periksa peringatan pemuatan file untuk file-file yang gagal

### Model Kamera Tidak Terdeteksi

**Penyebab yang mungkin:**

* Metadata EXIF dimodifikasi
* Gambar diedit menggunakan perangkat lunak eksternal
* Transfer file tidak lengkap

**Solusi**: Impor ulang file asli yang belum dimodifikasi dari kamera/kartu SD

### Stempel Waktu Hilang

**Penyebab yang mungkin:**

* Jam kamera tidak diatur dengan benar
* Data EXIF dihapus oleh perangkat lunak eksternal

**Solusi**: Pastikan pengaturan waktu kamera sudah benar saat pengambilan gambar

### Laporan Proyek yang Dibuka Kembali Menunjukkan File Hilang

Jika file sumber dipindahkan atau dihapus sejak proyek terakhir kali dibuka, Chloros akan memberitahu Anda **file mana saja** yang hilang, bukan membuka ke kisi kosong. Kembalikan berkas ke jalur aslinya, atau hapus entri yang hilang dan impor ulang.***

## Langkah Selanjutnya

Setelah berkas Anda diimpor:

1. **Periksa daftar berkas** - Pastikan semua gambar dimuat dengan benar
2. **Periksa model kamera** - Pastikan deteksi kamera sudah benar
3. **Tandai gambar target** - Lihat [Memilih Gambar Target](choosing-target-images.md)
4. **Sesuaikan pengaturan** - Konfigurasikan opsi pemrosesan di [Pengaturan Proyek](adjusting-project-settings.md)
5. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)

Untuk informasi terperinci mengenai konfigurasi proyek, lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md).

# Memantau Proses Pemrosesan

Setelah proses pemrosesan dimulai, Chloros menyediakan beberapa cara untuk memantau kemajuan, memeriksa adanya masalah, dan memahami apa yang terjadi pada dataset Anda. Halaman ini menjelaskan cara melacak proses pemrosesan Anda dan menafsirkan informasi yang disediakan oleh Chloros.

## Gambaran Umum Bilah Kemajuan

Bilah kemajuan di bagian atas header menampilkan status pemrosesan secara real-time dan persentase penyelesaian.

### Bilah Kemajuan Mode Gratis

Untuk pengguna tanpa lisensi Chloros+:

**Tampilan Kemajuan 2 Tahap:**

1.**Deteksi Target** - Mencari target kalibrasi dalam gambar
2. **Pemrosesan** - Menerapkan koreksi dan mengekspor**Bilah kemajuan menampilkan:**

* Persentase penyelesaian keseluruhan (0-100%)
* Nama tahap saat ini
* Visualisasi bilah horizontal sederhana

### Bilah Kemajuan Chloros+

Untuk pengguna dengan lisensi Chloros+:

**Tampilan Progres 4 Tahap:**

1.**Deteksi** - Mencari target kalibrasi
2. **Analisis** - Memeriksa gambar dan menyiapkan alur kerja
3. **Kalibrasi** - Menerapkan koreksi vignette dan reflektansi
4. **Ekspor** - Menyimpan file yang telah diproses**Fitur Interaktif:*** **Arahkan kursor** ke bilah kemajuan untuk melihat panel 4 tahap yang diperluas
* **Klik** bilah kemajuan untuk membekukan/menancapkan panel yang diperluas
* **Klik lagi** untuk membatalkan pembekuan dan menyembunyikan secara otomatis saat kursor keluar
* Setiap tahap menampilkan kemajuan individu (0-100%)

***

## Memahami Setiap Tahap Pemrosesan

{% hint style="info" %}
**Arsitektur Pipeline**: Keempat tahap antarmuka pengguna ini sesuai dengan [pipeline pemrosesan 4-thread](../processing-architecture/processing-pipeline.md). Pada sistem dengan akselerasi GPU, Thread 3 (Kalibrasi) memanfaatkan [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) yang mengoptimalkan pemrosesan untuk perangkat keras spesifik Anda.
{% endhint %}

### Tahap 1: Deteksi (Deteksi Target)

**Apa yang terjadi:**

* Chloros memindai gambar yang ditandai dengan kotak centang Target
* Algoritma penglihatan komputer mengidentifikasi 4 panel kalibrasi
* Nilai reflektansi diekstraksi dari setiap panel
* Cap waktu target dicatat untuk penjadwalan kalibrasi yang tepat

**Durasi:**

* Dengan target yang ditandai: 10–60 detik
* Tanpa target yang ditandai: 5–30+ menit (memindai semua gambar)

**Indikator kemajuan:**

* Deteksi: 0% → 100%
* Jumlah gambar yang dipindai
* Jumlah target yang ditemukan

**Hal yang perlu diperhatikan:**

* Harus selesai dengan cepat jika target ditandai dengan benar
* Jika memakan waktu terlalu lama, target mungkin tidak ditandai
* Periksa Log Debug untuk pesan &quot;Target ditemukan&quot;

### Tahap 2: Analisis

**Apa yang terjadi:**

* Membaca metadata EXIF gambar (cap waktu, pengaturan eksposur)
* Menentukan strategi kalibrasi berdasarkan cap waktu target
* Mengatur antrian pemrosesan gambar
* Mempersiapkan pekerja pemrosesan paralel (hanya Chloros+)

**Durasi:** 5-30 detik**Indikator kemajuan:**

* Menganalisis: 0% → 100%
* Tahap cepat, biasanya selesai dengan cepat

**Hal yang perlu diperhatikan:**

* Harus berjalan lancar tanpa jeda
* Peringatan tentang metadata yang hilang akan muncul di Debug Log

### Tahap 3: Kalibrasi

**Apa yang terjadi:*** **Debayering**: Mengonversi pola Bayer RAW menjadi 3 kanal
* **Koreksi vignette**: Menghilangkan penggelapan tepi lensa
* **Kalibrasi reflektansi**: Menormalisasi dengan nilai target
* **Perhitungan indeks**: Menghitung indeks multispektral
* Memproses setiap gambar melalui seluruh alur kerja

**Durasi:** Sebagian besar waktu pemrosesan total (60-80%)**Indikator kemajuan:**

* Kalibrasi: 0% → 100%
* Gambar saat ini sedang diproses
* Gambar yang telah selesai / Total gambar

**Perilaku pemrosesan:*** **Mode bebas**: Memproses satu gambar pada satu waktu secara berurutan
* **Mode Chloros+**: Memproses hingga 16 gambar secara bersamaan
* **Akselerasi GPU**: Secara signifikan mempercepat tahap ini**Hal yang perlu diperhatikan:**

* Kemajuan yang stabil berdasarkan jumlah gambar
* Periksa Log Debug untuk pesan penyelesaian per gambar
* Peringatan mengenai kualitas gambar atau masalah kalibrasi

### Tahap 4: Ekspor

**Apa yang terjadi:**

* Menulis gambar yang telah dikalibrasi ke disk dalam format yang dipilih
* Mengekspor gambar indeks multispektral dengan warna LUT
* Membuat subfolder model kamera
* Mempertahankan nama file asli dengan sufiks yang sesuai

**Durasi:** 10-20% dari total waktu pemrosesan**Indikator kemajuan:**

* Ekspor: 0% → 100%
* File sedang ditulis
* Format ekspor dan tujuan

**Hal yang perlu diperhatikan:**

* Peringatan ruang disk
* Kesalahan penulisan file
* Selesainya semua output yang dikonfigurasi

***

## Tab Log Debug

Log Debug menyediakan informasi terperinci tentang kemajuan pemrosesan dan masalah apa pun yang ditemui.

### Mengakses Log Debug

1. Klik ikon **Log Debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> di bilah sisi kiri
2. Panel log terbuka menampilkan pesan pemrosesan secara real-time
3. Otomatis menggulir untuk menampilkan pesan terbaru

### Memahami Pesan Log

#### Pesan Informasi (Putih/Abu-abu)

Pembaruan pemrosesan normal:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Pesan Peringatan (Kuning)

Masalah non-kritis yang tidak menghentikan pemrosesan:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Tindakan:** Periksa peringatan setelah pemrosesan selesai, tetapi jangan hentikan

#### Pesan Kesalahan (Red)

Masalah kritis yang dapat menyebabkan pemrosesan gagal:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Tindakan:** Hentikan pemrosesan, selesaikan kesalahan, mulai ulang

### Pesan Log Umum

| Pesan                          | Arti                                | Tindakan yang Diperlukan                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| &quot;Target terdeteksi di \[nama_file]&quot; | Target kalibrasi berhasil ditemukan  | Tidak ada - normal                                         |
| &quot;Memproses gambar X dari Y&quot;        | Pembaruan kemajuan saat ini                | Tidak ada - normal                                         |
| &quot;Tidak ada target yang ditemukan&quot;               | Tidak ada target kalibrasi yang terdeteksi        | Tandai gambar target atau nonaktifkan kalibrasi reflektansi |
| &quot;Ruang disk tidak mencukupi&quot;        | Penyimpanan tidak cukup untuk output          | Kosongkan ruang disk                                    |
| &quot;Melewati file yang rusak&quot;        | File gambar rusak                  | Salin ulang file dari kartu SD                             |
| &quot;Data PPK diterapkan&quot;               | Koreksi GPS dari file .daq diterapkan | Tidak ada - normal                                         |

### Menyalin Data Log

Untuk menyalin log untuk pemecahan masalah atau dukungan:

1. Buka panel Debug Log
2. Klik tombol **&quot;Copy Log&quot;** (atau klik kanan → Pilih Semua)
3. Tempel ke berkas teks atau email
4. Kirim ke dukungan MAPIR jika diperlukan

***

## Pemantauan Sumber Daya Sistem

### Penggunaan CPU

**Mode Gratis:**

* 1 inti CPU pada ~100%
* Inti lainnya idle atau tersedia
* Sistem tetap responsif

**Chloros+ Mode Paralel:**

* Beberapa inti pada 80-100% (hingga 16 inti)
* Penggunaan CPU keseluruhan tinggi
* Sistem mungkin terasa kurang responsif

**Untuk memantau:**

* Windows Task Manager (Ctrl+Shift+Esc)
* Tab Kinerja → Bagian CPU
* Cari proses &quot;Chloros&quot; atau &quot;chloros-backend&quot;

### Penggunaan Memori (RAM)

**Penggunaan tipikal:**

* Proyek kecil (&lt; 100 gambar): 2-4 GB
* Proyek sedang (100-500 gambar): 4-8 GB
* Proyek besar (500+ gambar): 8-16 GB
* Mode paralel Chloros+ menggunakan lebih banyak RAM

**Jika memori rendah:**

* Proses dalam batch yang lebih kecil
* Tutup aplikasi lain
* Tingkatkan RAM jika sering memproses dataset besar

### Penggunaan GPU (Chloros+ dengan CUDA)

Saat akselerasi GPU diaktifkan:

* GPU NVIDIA menunjukkan pemanfaatan tinggi (60-90%)
* Penggunaan VRAM meningkat (membutuhkan 4GB+ VRAM)
* Tahap kalibrasi jauh lebih cepat

**Untuk memantau:**

* Ikon NVIDIA di System Tray
* Task Manager → Kinerja → GPU
* GPU-Z atau alat pemantauan serupa

### I/O Disk

**Yang diharapkan:**

* Pembacaan disk tinggi selama tahap Analisis
* Penulisan disk tinggi selama tahap Ekspor
* SSD jauh lebih cepat daripada HDD

**Tips kinerja:**

* Gunakan SSD untuk folder proyek jika memungkinkan
* Hindari drive jaringan untuk dataset besar
* Pastikan ruang disk tidak mendekati kapasitas penuh (mempengaruhi kecepatan penulisan)

***

## Mendeteksi Masalah Selama Pemrosesan

### Tanda-tanda Peringatan

**Kemajuan terhenti (tidak ada perubahan selama 5+ menit):**

* Periksa Log Debug untuk kesalahan
* Verifikasi ruang disk yang tersedia
* Periksa Task Manager untuk memastikan Chloros sedang berjalan

**Pesan kesalahan muncul secara berulang:**

* Hentikan pemrosesan dan tinjau kesalahan
* Penyebab umum: ruang disk, file rusak, masalah memori
* Lihat bagian Pemecahan Masalah di bawah

**Sistem tidak merespons:**

* Mode paralel Chloros+ menggunakan terlalu banyak sumber daya
* Pertimbangkan untuk mengurangi tugas bersamaan atau meng-upgrade perangkat keras
* Mode bebas lebih hemat sumber daya

### Kapan Harus Menghentikan Pemrosesan

Hentikan pemrosesan jika Anda melihat:

* ❌ Kesalahan &quot;Disk penuh&quot; atau &quot;Tidak dapat menulis file&quot;
* ❌ Kesalahan file gambar rusak berulang
* ❌ Sistem benar-benar macet (tidak merespons)
* ❌ Menyadari pengaturan yang salah telah dikonfigurasi
* ❌ Gambar yang salah diimpor

**Cara menghentikan:**

1. Klik tombol**Stop/Cancel** (menggantikan tombol Start)
2. Pemrosesan berhenti, progres hilang
3. Perbaiki masalah dan mulai ulang dari awal

***

## Pemecahan Masalah Selama Pemrosesan

### Pemrosesan Sangat Lambat

**Penyebab kemungkinan:**

* Gambar target tidak ditandai (memindai semua gambar)
* Penyimpanan HDD alih-alih SSD
* Sumber daya sistem tidak mencukupi
* Banyak indeks yang dikonfigurasi
* Akses drive jaringan

**Solusi:**

1. Jika baru dimulai dan berada di tahap Deteksi: Batalkan, tandai target, mulai ulang
2. Untuk ke depannya: Gunakan SSD, kurangi indeks, tingkatkan perangkat keras
3. Pertimbangkan CLI untuk pemrosesan batch dataset besar

### Peringatan &quot;Ruang Disk&quot;

**Solusi:**

1. Kosongkan ruang disk segera
2. Pindahkan proyek ke drive dengan ruang lebih besar
3. Kurangi jumlah indeks yang akan diekspor
4. Gunakan format JPG alih-alih TIFF (berkas lebih kecil)

### Pesan &quot;Berkas Rusak&quot; yang Sering Muncul

**Solusi:**

1. Salin ulang gambar dari kartu SD untuk memastikan integritas
2. Periksa kartu SD untuk kesalahan
3. Hapus berkas rusak dari proyek
4. Lanjutkan pemrosesan gambar yang tersisa

### Sistem Overheating / Throttling

**Solusi:**

1. Pastikan ventilasi yang memadai
2. Bersihkan debu dari ventilasi komputer
3. Kurangi beban pemrosesan (gunakan mode Free alih-alih Chloros+)
4. Lakukan pemrosesan pada waktu yang lebih sejuk

***

## Pemberitahuan Pemrosesan Selesai

Saat pemrosesan selesai:

* Bilah kemajuan mencapai 100%
* Pesan **&quot;Pemrosesan Selesai&quot;** muncul di Log Debug
* Tombol Mulai kembali aktif
* Semua file output berada di subfolder model kamera

***

## Langkah Selanjutnya

Setelah pemrosesan selesai:

1. **Periksa hasil** - Lihat [Menyelesaikan Pemrosesan](finishing-the-processing.md)
2. **Periksa folder keluaran** - Pastikan semua file diekspor dengan benar
3. **Periksa Log Debug** - Periksa apakah ada peringatan atau kesalahan
4. **Pratinjau gambar yang telah diproses** - Gunakan Image Viewer atau perangkat lunak eksternal

Untuk informasi tentang cara meninjau dan menggunakan hasil pemrosesan Anda, lihat [Menyelesaikan Pemrosesan](finishing-the-processing.md).

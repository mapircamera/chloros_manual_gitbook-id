# Memantau Proses Pemrosesan

Setelah proses pemrosesan dimulai, Chloros

menyediakan beberapa cara untuk memantau kemajuan, memeriksa adanya masalah, dan memahami apa yang terjadi pada dataset Anda. Halaman ini menjelaskan cara melacak proses pemrosesan Anda dan menafsirkan informasi yang disediakan oleh Chloros

.

## Gambaran Umum Bilah Kemajuan

Bilah kemajuan di bagian atas header menampilkan status pemrosesan secara real-time dan persentase penyelesaian. Kemajuan ditayangkan langsung dari backend melalui Server-Sent Events (SSE), sehingga bilah tersebut mencerminkan apa yang sebenarnya dilakukan oleh pipeline.

### Bilah Kemajuan Mode Gratis

Untuk pengguna tanpa lisensi Chloros

+:

**Tampilan Kemajuan 2 Tahap:**

1.**Deteksi Target** - Mencari target kalibrasi dalam gambar
2. **Pemrosesan** - Menerapkan koreksi dan mengekspor**Bilah kemajuan menampilkan:**

* Persentase penyelesaian keseluruhan (0-100%)
* Nama tahap saat ini
* Visualisasi bilah horizontal sederhana

### Bilah Kemajuan Chloros

+

Untuk pengguna dengan lisensi Chloros

+:

**Tampilan Kemajuan 4 Tahap:**

1.**Deteksi** - Mencari target kalibrasi
2. **Analisis** - Memeriksa gambar dan mempersiapkan pipeline
3. **Kalibrasi** - Menerapkan koreksi vignette dan reflektansi
4. **Ekspor** - Menyimpan file yang telah diproses**Fitur Interaktif:*** **Arahkan kursor** ke bilah kemajuan untuk melihat panel 4 tahap yang diperluas
* **Klik** bilah kemajuan untuk membekukan/menancapkan panel yang diperluas
* **Klik lagi** untuk melepaskan pembekuan dan menyembunyikan secara otomatis saat kursor mouse berpindah
* Setiap tahap menampilkan kemajuan masing-masing (0-100%)

{% hint style="info" %}
**Kesetaraan CLI**: selama proses `chloros-cli process` berjalan, keempat utas tersebut melaporkan status sebagai &quot;Mendeteksi&quot;, &quot;Menganalisis&quot;, &quot;Memproses&quot;, dan &quot;Mengekspor&quot;, sedangkan `chloros-cli export-status` menampilkan kemajuan ekspor Utas-4 secara langsung dari terminal lain. Lihat [Referensi CLI

](../reference/cli-reference.md).
{% endhint %}

***

## Memahami Setiap Tahap Pemrosesan

{% hint style="info" %}
**Arsitektur Pipa**: Keempat tahap GUI ini sesuai dengan [pipa pemrosesan 4-thread](../processing-architecture/processing-pipeline.md). Pada sistem dengan akselerasi GPU, Thread 3 (Kalibrasi) memanfaatkan [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) yang mengoptimalkan pemrosesan untuk perangkat keras spesifik Anda.
{% endhint %}

### Tahap 1: Deteksi (Deteksi Target)

**Apa yang terjadi:**

* Chloros

memindai gambar-gambar yang Anda centang dengan kotak centang Target (semua gambar jika tidak ada yang dicentang)
* Algoritma penglihatan komputer mengidentifikasi panel kalibrasi
* Nilai reflektansi diekstraksi dari setiap panel
* Cap waktu target dicatat untuk penjadwalan kalibrasi yang tepat

**Durasi:**

* Dengan target yang ditandai: 10–60 detik
* Tanpa target yang ditandai: 5–30+ menit (memindai semua gambar)

**Indikator kemajuan:**

* Sedang mendeteksi: 0% → 100%
* Jumlah gambar yang dipindai (hanya menghitung gambar yang benar-benar dipindai)
* Jumlah target yang ditemukan

**Hal yang perlu diperhatikan:**

* Seharusnya selesai dengan cepat jika target ditandai dengan benar
* Jika memakan waktu terlalu lama, mungkin target tidak ditandai
* Periksa Log Debug untuk pesan &quot;Target ditemukan&quot;

### Tahap 2: Analisis

**Proses yang berlangsung:**

* Membaca metadata EXIF gambar (cap waktu, pengaturan eksposur)
* Menentukan strategi kalibrasi berdasarkan cap waktu target dan data DAQ downwelling yang tersedia
* Mengatur antrian pemrosesan gambar
* Mempersiapkan pekerja pemrosesan paralel (hanya Chloros

+)

**Durasi:** 5–30 detik**Indikator kemajuan:**

* Sedang dianalisis: 0% → 100%
* Tahap yang cepat, biasanya selesai dalam waktu singkat

**Hal yang perlu diperhatikan:**

* Harus berjalan lancar tanpa jeda
* Peringatan mengenai metadata yang hilang akan muncul di Debug Log

### Tahap 3: Kalibrasi

**Proses yang terjadi:*** **Debayering**: Mengonversi pola RAW Bayer menjadi 3 kanal (dilewati untuk modul mono LATTICE, dengan catatan)
* **Koreksi vignette**: Menghilangkan penggelapan tepi lensa
* **Kalibrasi reflektansi**: Menormalkan dengan nilai target dan/atau downwelling DAQ
* **Perhitungan indeks**: Menghitung indeks multispektral
* Memproses setiap gambar melalui seluruh alur kerja

**Durasi:** Sebagian besar dari total waktu pemrosesan (60–80%)**Indikator kemajuan:**

* Kalibrasi: 0% → 100%
* Gambar saat ini sedang diproses
* Gambar yang telah selesai / Total gambar

**Perilaku pemrosesan:*** **Mode bebas**: Memproses satu gambar pada satu waktu secara berurutan
* **Mode Chloros

+**: Menjalankan kumpulan pekerja yang adaptif terhadap perangkat keras — 1–4 pekerja secara bersamaan pada sistem GPU (tergantung VRAM), satu pekerja per inti fisik (dikurangi satu) pada sistem berbasis CPU saja. Lihat [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md)
* **Akselerasi GPU**: Secara signifikan mempercepat tahap ini**Hal yang perlu diperhatikan:**

* Kemajuan yang stabil berdasarkan jumlah gambar
* Periksa Log Debug untuk pesan penyelesaian per gambar
* Peringatan mengenai kualitas gambar atau masalah kalibrasi

### Tahap 4: Ekspor

**Yang terjadi:**

* Menyimpan gambar yang telah diproses ke disk dalam format yang dipilih, seiring selesainya pemrosesan
* **LATTICE**: setiap bingkai dibagi ke setiap produk yang diaktifkan (debayered / pratinjau / radiance / reflektansi)
* Mengekspor gambar indeks multispektral dengan warna LUT
* Membuat struktur folder keluaran `<project>/<camera>/<format>/<Product>_Images/` — file yang diekspor mempertahankan nama file sumber; folder tersebut mengidentifikasi produk

**Durasi:** 10–20% dari total waktu pemrosesan**Indikator kemajuan:**

* Ekspor: 0% → 100%
* File sedang ditulis
* Format ekspor dan tujuan

**Hal yang perlu diperhatikan:**

* Peringatan ruang disk
* Kesalahan penulisan file
* Selesainya semua keluaran yang dikonfigurasi

***

## Tab Log Debug

Log Debug menyediakan informasi terperinci mengenai kemajuan pemrosesan dan masalah apa pun yang ditemui. Pesan-pesan saat startup backend juga ditampilkan kembali di konsol log, sehingga log tersebut menceritakan seluruh proses meskipun Anda membukanya belakangan.

### Mengakses Log Debug

1. Klik ikon **Log Debug** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

di bilah sisi kiri
2. Panel log akan terbuka dan menampilkan pesan pemrosesan secara real-time
3. Layar akan bergulir otomatis untuk menampilkan pesan terbaru

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Memahami Pesan Log Baris log

Chloros

diawali dengan tag dalam kurung yang menamai subsistem — misalnya `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. Yang paling penting untuk diketahui adalah **ringkasan eksekusi**, yang ditampilkan di akhir setiap eksekusi (termasuk eksekusi yang dihentikan):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Baris petunjuk tambahan `[RUN-SUMMARY]` akan muncul setiap kali ada hal yang perlu dijelaskan — misalnya, proses yang tidak menghasilkan apa-apa, atau kamera yang produk yang diminta dilewati karena tidak berlaku. Baris `[EXPORT-CHECK]` menjelaskan alasan dilewati per kamera (misalnya, mengapa kamera RGB

tidak mendapatkan produk radian).

Tingkat keparahan pesan secara umum (contoh di bawah ini bersifat ilustratif, bukan kata demi kata):

#### Pesan Informasi (Putih/Abu-abu)

Pembaruan pemrosesan normal: pemrosesan dimulai, target terdeteksi (dengan jumlah panel), kemajuan kalibrasi per gambar, file diekspor, pemrosesan selesai.

#### Pesan Peringatan (Kuning)

Masalah non-kritis yang tidak menghentikan pemrosesan — misalnya data GPS hilang dalam satu bingkai, selisih waktu yang besar antara gambar target, atau kontras rendah pada panel kalibrasi.

**Tindakan:** Periksa peringatan setelah pemrosesan selesai, tetapi jangan hentikan

#### Pesan Kesalahan (Warna Red

)

Masalah kritis yang dapat menyebabkan kegagalan pemrosesan — misalnya, disk penuh, file gambar rusak, atau tidak ada target yang terdeteksi saat kalibrasi reflektansi diminta.

**Tindakan:** Hentikan pemrosesan, atasi kesalahan, lalu mulai ulang

### Situasi Umum dalam Log

| Situasi                             | Arti                                       | Tindakan yang Diperlukan                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Target terdeteksi di \[nama_file]        | Target kalibrasi berhasil ditemukan         | Tidak ada - normal                                         |
| Garis kemajuan per gambar              | Pembaruan kemajuan saat ini                       | Tidak ada - normal                                         |
| Tidak ditemukan target                      | Tidak terdeteksi target kalibrasi               | Tandai gambar target atau nonaktifkan kalibrasi reflektansi |
| Ruang disk tidak mencukupi               | Penyimpanan tidak cukup untuk hasil                 | Kosongkan ruang disk                                    |
| Melewati berkas yang rusak               | Berkas gambar rusak                         | Salin ulang berkas dari kartu SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Pengambilan gambar tanpa bingkai mentah tidak dapat diproses | Ambil ulang dengan bingkai mentah, atau gunakan CLI

`--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | Proses tidak menghasilkan produk gambar — dilaporkan sebagai kegagalan dengan petunjuk | Baca baris petunjuk; periksa apa yang dilewati dan alasannya |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Jalur panjang Windows

dinonaktifkan dan folder proyek terlalu dalam untuk batas jalur 260 karakter; tidak ada yang diproses | Gunakan folder proyek yang lebih pendek (pesan tersebut menunjukkan panjang maksimum yang masih bisa muat), atau aktifkan jalur panjang Windows

dan mulai ulang Chloros

— lihat [Memulai Pemrosesan](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Masker clamp produk LATTICE tidak dapat ditulis; produknya sendiri baik-baik saja dan menampilkan pesan `Chloros:UnmixClampMask=unavailable` | Periksa ruang disk dan izin folder, lalu proses ulang jika Anda memerlukan masker tersebut |

### Menyalin Data Log

Untuk menyalin log guna pemecahan masalah atau dukungan:

1. Buka panel Debug Log
2. Klik tombol **Salin**(menyalin log ke clipboard), atau**Unduh** untuk menyimpannya sebagai berkas teks
3. Tempelkan ke berkas teks atau email
4. Kirim ke dukungan MAPIR

jika diperlukan

### Melaporkan bug dari tab log

Baru di versi 1.3.0: ikon bug (**Laporkan bug**) di bagian atas tab Log Debug akan mengirimkan laporan langsung ke MAPIR

.

1. Klik ikon bug di bagian atas tab Log Debug
2. Jelaskan masalahnya: apa yang sedang Anda lakukan, apa yang Anda harapkan, dan apa yang terjadi sebaliknya (wajib diisi)
3. Secara opsional, centang **Tambahkan beberapa gambar (maksimal 10)**dan pilih gambar dari proyek yang terbuka, atau gunakan**Telusuri...** untuk file gambar lainnya
4. Klik **Kirim**

Laporan tersebut mencakup deskripsi Anda, gambar yang Anda lampirkan, log Chloros

, dan detail sistem (versi Chloros

, OS, GPU, dan pengaturan proyek). Anda harus masuk ke Chloros

(tab Pengguna) untuk mengirim laporan. Anda dapat menutup jendela saat proses pengiriman berlangsung; setelah selesai, referensi seperti `Bug report sent. Reference: BR-…` akan tercatat dalam log — cantumkan referensi tersebut saat Anda menghubungi MAPIR

. Jika laporan tidak dapat diunggah (misalnya, tidak ada koneksi internet), Chloros

akan menyimpannya di komputer Anda dan menampilkan jalurnya sehingga Anda dapat mengirimkannya melalui email ke support@mapir.camera.

***

## Pemantauan Sumber Daya Sistem

### Penggunaan CPU

**Mode Bebas:**

* 1 inti CPU pada ~100%
* Inti lainnya dalam keadaan idle atau tersedia
* Sistem tetap responsif

**Mode Paralel Chloros

+:**

* Beberapa inti CPU dengan pemanfaatan tinggi — jumlahnya tergantung pada strategi yang dipilih oleh [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md)
* Sistem mungkin terasa kurang responsif

**Untuk memantau:**

* Task Manager Windows

(Ctrl+Shift+Esc)
* Tab Kinerja → Bagian CPU
* Cari proses &quot;Chloros

&quot; atau &quot;chloros-backend&quot;

### Penggunaan Memori (RAM)

**Penggunaan umum:**

* Proyek kecil (&lt; 100 gambar): 2–4 GB
* Proyek sedang (100–500 gambar): 4–8 GB
* Proyek besar (500+ gambar): 8–16 GB
* Mode paralel Chloros

+ menggunakan lebih banyak RAM

**Jika memori rendah:**

* Proses dalam batch yang lebih kecil
* Tutup aplikasi lain
* Tingkatkan kapasitas RAM jika sering memproses dataset besar

### Penggunaan GPU (Chloros

+ dengan CUDA)

Saat akselerasi GPU diaktifkan:

* GPU NVIDIA menunjukkan tingkat pemanfaatan yang tinggi (60–90%)
* Penggunaan VRAM meningkat (membutuhkan 4 GB+ VRAM; 7 GB+ untuk debayering Texture Aware secara bersamaan)
* Tahap kalibrasi menjadi jauh lebih cepat

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
* Pastikan ruang disk tidak hampir penuh (mempengaruhi kecepatan penulisan)

***

## Mendeteksi Masalah Selama Pemrosesan

### Tanda-tanda Peringatan

**Kemajuan terhenti (tidak ada perubahan selama 5+ menit):**

* Periksa Log Debug untuk mencari kesalahan
* Pastikan ruang disk masih tersedia
* Periksa Task Manager untuk memastikan Chloros

sedang berjalan

**Pesan kesalahan sering muncul:**

* Hentikan pemrosesan dan periksa kesalahan
* Penyebab umum: ruang disk, file rusak, masalah memori
* Lihat bagian Pemecahan Masalah di bawah ini

**Sistem menjadi tidak responsif:**

* Mode paralel Chloros

+ menggunakan terlalu banyak sumber daya
* Pertimbangkan untuk mengurangi tugas yang berjalan bersamaan atau meningkatkan perangkat keras
* Mode bebas lebih hemat sumber daya

### Kapan Harus Menghentikan Pemrosesan

Hentikan pemrosesan jika Anda melihat:

* ❌ Kesalahan &quot;Disk penuh&quot; atau &quot;Tidak dapat menulis file&quot;
* ❌ Kesalahan file gambar rusak yang berulang
* ❌ Sistem benar-benar macet (tidak merespons)
* ❌ Menyadari bahwa pengaturan yang salah telah dikonfigurasi
* ❌ Gambar yang salah diimpor

**Cara menghentikan:**

1. Klik tombol**Stop** (menggantikan tombol Start) — sekali saja sudah cukup
2. Bilah kemajuan akan menampilkan &quot;Sedang dihentikan...&quot; sementara gambar yang sedang diproses diselesaikan, lalu proses berakhir dalam keadaan terhenti
3. Produk yang sudah diekspor tetap tersimpan di disk; log mencatat kode kesalahan `[RUN-SUMMARY]` yang menunjukkan proses yang telah selesai
4. Perbaiki masalah dan mulai ulang — proses akan dimulai dari awal

***

## Pemecahan Masalah Selama Pemrosesan

### Pemrosesan Sangat Lambat

**Kemungkinan penyebab:**

* Gambar target yang tidak ditandai (memindai semua gambar)
* Menggunakan HDD alih-alih penyimpanan SSD
* Sumber daya sistem tidak mencukupi
* Banyak indeks yang dikonfigurasi
* Akses drive jaringan

**Solusi:**

1. Jika baru dimulai dan masih dalam tahap Deteksi: Hentikan, tandai target, lalu mulai ulang
2. Untuk ke depannya: Gunakan SSD, kurangi jumlah indeks, tingkatkan perangkat keras
3. Pertimbangkan penggunaan antarmuka baris perintah (CLI

) untuk pemrosesan batch dataset besar

### Peringatan &quot;Ruang Disk&quot;

**Solusi:**

1. Segera kosongkan ruang disk
2. Pindahkan proyek ke drive dengan ruang yang lebih besar
3. Kurangi jumlah indeks yang akan diekspor
4. Nonaktifkan produk ekspor LATTICE yang tidak Anda butuhkan (Pengaturan Proyek → Pemrosesan)
5. Gunakan format JPG alih-alih TIFF

(ukuran file lebih kecil)

### Pesan &quot;File Rusak&quot; yang Sering Muncul

**Solusi:**

1. Salin ulang gambar dari kartu SD untuk memastikan integritasnya
2. Periksa kartu SD untuk mendeteksi kesalahan
3. Hapus berkas yang rusak dari proyek
4. Lanjutkan pemrosesan gambar yang tersisa

### Sistem Terlalu Panas / Pembatasan Kecepatan

**Solusi:**

1. Pastikan ventilasi yang memadai
2. Bersihkan debu dari ventilasi komputer
3. Kurangi beban pemrosesan (gunakan mode Free alih-alih Chloros

+)
4. Lakukan pemrosesan pada waktu-waktu yang lebih sejuk dalam sehari

***

## Pemberitahuan Pemrosesan Selesai

Saat pemrosesan selesai:

* Bilah kemajuan mencapai 100%
* Baris `[RUN-SUMMARY]` muncul di Log Debug bersama jumlah akhir
* Tombol Mulai kembali aktif
* Semua file keluaran berada di pohon keluaran per kamera proyek: `<project>/<camera>/<format>/<Product>_Images/`

***

## Langkah Selanjutnya

Setelah pemrosesan selesai:

1. **Tinjau hasil** - Lihat [Menyelesaikan Pemrosesan](finishing-the-processing.md)
2. **Periksa folder keluaran** - Pastikan semua berkas diekspor dengan benar
3. **Tinjau Log Debug** - Periksa apakah ada peringatan atau kesalahan
4. **Pratinjau gambar yang telah diproses** - Gunakan Image Viewer atau perangkat lunak eksternal

Untuk informasi mengenai cara meninjau dan menggunakan hasil pemrosesan Anda, lihat [Menyelesaikan Pemrosesan](finishing-the-processing.md).

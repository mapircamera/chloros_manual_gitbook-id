# Memulai Pemrosesan

Setelah Anda mengimpor gambar, menandai target kalibrasi, dan mengonfigurasi pengaturan proyek, Anda siap untuk memulai pemrosesan. Halaman ini akan memandu Anda dalam memulai alur pemrosesan Chloros.

## Daftar Periksa Pra-Pemrosesan

Sebelum mengklik tombol Mulai, pastikan semuanya sudah siap:

* [ ] **File telah diimpor** - Semua gambar muncul di File Browser
* [ ] **Gambar target telah ditandai** - Kolom Target telah dicentang untuk gambar kalibrasi (atau rekaman `.daq` telah diimpor untuk LATTICE)
* [ ] **Model kamera terdeteksi** - Kolom Model Kamera menampilkan kamera yang benar
* [ ] **Pengaturan telah dikonfigurasi** - Pengaturan Proyek telah ditinjau dan disesuaikan
* [ ] **Indeks dipilih** - Indeks multispektral yang diinginkan telah ditambahkan (jika diperlukan)
* [ ] **Format ekspor telah dipilih** - Format keluaran yang sesuai dengan alur kerja Anda

{% hint style="info" %}
**Tip**: Klik beberapa gambar di Penjelajah File untuk memverifikasi apakah gambar-gambar tersebut telah dimuat dengan benar sebelum diproses.
{% endhint %}

***

## Memulai Pemrosesan

### Temukan Tombol Mulai

Tombol Mulai/Putar terletak di bilah header atas Chloros:

* Posisi: Bagian tengah atas jendela
* Ikon: **Tombol Putar/Mulai** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: Tombol aktif (bersinar) saat siap diproses

### Klik untuk Memulai

1. Klik **Tombol Putar/Mulai** di bilah header atas
2. Pemrosesan segera dimulai
3. Tombol tersebut berubah menjadi tombol **Berhenti** selama pemrosesan
4. Bilah kemajuan diperbarui, menampilkan status pemrosesan

{% hint style="success" %}
**Pemrosesan Dimulai**: Setelah diklik, Chloros secara otomatis menangani semua langkah pemrosesan — deteksi target, debayering, kalibrasi, perhitungan indeks, dan ekspor. Program ini secara otomatis mendeteksi apakah proyek Anda menggunakan Survey3, LATTICE, atau kombinasi keduanya, dan menerapkan alur kerja yang tepat untuk setiap kamera.
{% endhint %}

***

## Memahami Mode Pemrosesan

Chloros beroperasi dalam dua mode pemrosesan yang berbeda tergantung pada lisensi Anda:

### Mode Gratis (Pemrosesan Berurutan)

**Tersedia untuk semua pengguna**

**Cara kerjanya:**

* Memproses gambar satu per satu, secara berurutan
* Operasi single-threaded
* Penggunaan memori lebih rendah

**Bilah kemajuan menampilkan 2 tahap:**

1.**Deteksi Target** - Memindai target kalibrasi
2. **Pemrosesan** - Menerapkan kalibrasi dan mengekspor gambar**Waktu pemrosesan:**

* Jauh lebih lambat daripada mode paralel Chloros+
* Cocok untuk kumpulan data kecil hingga menengah (&lt; 200 gambar)

### Mode Chloros+ (Pemrosesan Paralel)

**Membutuhkan lisensi Chloros+**

**Cara kerjanya:**

* Memproses beberapa gambar secara bersamaan menggunakan [pipa pemrosesan 4-thread](../processing-architecture/processing-pipeline.md)
* [Adaptasi Komputasi Dinamis](../processing-architecture/dynamic-compute-adaptation.md) secara otomatis memilih strategi optimal untuk perangkat keras Anda saat program dimulai
* Akselerasi GPU (CUDA) dengan kartu grafis NVIDIA (desktop dan Jetson)
* **Jumlah pekerja (worker) menyesuaikan dengan perangkat keras**: Strategi GPU menjalankan**1–4 pekerja secara bersamaan** (diskalakan berdasarkan VRAM — Jetson dengan memori rendah menjalankan 1, sedangkan GPU desktop 12GB+ dapat menjalankan hingga 4); sistem berbasis CPU saja menjalankan satu pekerja per inti fisik, dikurangi satu**Bilah kemajuan menampilkan 4 tahap** (sesuai dengan 4 utas pipa):

1. **Deteksi** (Utas 1) - Mencari target kalibrasi
2. **Analisis** (Utas 2) - Memeriksa metadata gambar dan menghitung kalibrasi
3. **Kalibrasi** (Thread 3) - Debayering, koreksi vignette, kalibrasi, perhitungan indeks
4. **Ekspor** (Thread 4) - Menyimpan gambar yang telah diproses dan indeks**Interaksi bilah kemajuan:*** **Arahkan kursor mouse** ke bilah untuk melihat panel dropdown 4 tahap yang terperinci
* **Klik** bilah kemajuan untuk membekukan panel dropdown di tempatnya
* **Klik lagi** untuk melepaskan pembekuan dan menyembunyikan panel**Waktu pemrosesan:**

* Jauh lebih cepat daripada mode gratis
* Akselerasi GPU semakin meningkatkan kecepatan

{% hint style="info" %}
**Chloros+ Kecepatan**: Pemrosesan paralel dapat 5-10 kali lebih cepat daripada mode sekuensial untuk kumpulan data besar. Proyek dengan 500 gambar yang membutuhkan waktu 2 jam dalam mode gratis dapat diselesaikan dalam 15–20 menit dengan Chloros+.
{% endhint %}

***

## Apa yang Terjadi Selama Pemrosesan

### Tahap 1: Deteksi Target

**Apa yang dilakukan Chloros:**

* Memindai gambar yang Anda centang di kolom Target (semua gambar jika tidak ada yang dicentang)
* Mengidentifikasi panel kalibrasi di setiap target
* Mengekstrak nilai reflektansi dari panel target
* Mencatat cap waktu target untuk penjadwalan kalibrasi

**Durasi:** 1–30 detik (dengan target yang ditandai), 5–30+ menit (tanpa tanda)

### Tahap 2: Debayering (Konversi RAW)

**Fungsi Chloros:**

* Mengonversi data pola Bayer RAW menjadi gambar 3-saluran penuh (modul mono LATTICE tetap single-band — proses debayering dilewati untuk modul tersebut dengan catatan di log)
* Menerapkan algoritma demosaicing yang dipilih
* Mempertahankan kualitas dan detail gambar maksimal

**Durasi:** Bervariasi tergantung jumlah gambar dan kecepatan CPU/GPU

### Tahap 3: Kalibrasi

**Fungsi Chloros:*** **Koreksi vignette**: Menghilangkan penggelapan lensa di tepi
* **Kalibrasi reflektansi**: Menormalkan menggunakan nilai reflektansi target dan/atau data downwelling DAQ
* Menerapkan koreksi pada semua band/saluran
* Menggunakan referensi kalibrasi yang sesuai untuk setiap gambar berdasarkan cap waktu

**Durasi:** Sebagian besar waktu pemrosesan

### Tahap 4: Perhitungan Indeks

**Fungsi Chloros:**

* Menghitung indeks multispektral yang telah dikonfigurasi (NDVI, NDRE, dll.)
* Menerapkan perhitungan matematika pita ke citra yang telah dikalibrasi
* Menghasilkan citra indeks untuk setiap indeks yang dipilih

**Durasi:** Beberapa detik per citra

### Tahap 5: Ekspor

**Apa yang dilakukan oleh Chloros:**

* Menyimpan citra yang telah diproses dalam format yang dipilih
* **LATTICE fan-out**: setiap bingkai LATTICE mentah diekspor sebagai setiap produk yang diaktifkan dalam satu proses — debayered, pratinjau, radiance (selalu float32), reflektansi
* Menulis berkas ke dalam pohon keluaran proyek: `<project>/<camera>/<format>/<Product>_Images/`
* **Mempertahankan nama file sumber** — folder mengidentifikasi produk, tidak ada sufiks yang ditambahkan**Durasi:** Bervariasi tergantung pada format ekspor dan ukuran file***

## Perilaku Pemrosesan

### Jalur Pemrosesan Otomatis

Setelah dimulai, seluruh jalur pemrosesan berjalan secara otomatis:

* Tidak memerlukan interaksi pengguna
* Semua langkah yang dikonfigurasi dijalankan secara berurutan
* Pembaruan kemajuan ditampilkan secara real-time
* File yang diekspor ditulis ke disk saat selesai — Anda dapat membuka hasil yang sudah selesai sementara proses masih berjalan

### Penggunaan Komputer Selama Pemrosesan

**Mode Bebas:**

* Penggunaan CPU relatif rendah (single-threaded)
* Komputer tetap responsif untuk tugas-tugas lain
* Aman untuk meminimalkan jendela Chloros dan bekerja di aplikasi lain

**Mode Paralel Chloros+:**

* Penggunaan CPU tinggi di seluruh kumpulan pekerja strategi
* Dengan akselerasi GPU: Penggunaan GPU tinggi
* Komputer mungkin kurang responsif selama pemrosesan
* Hindari memulai tugas lain yang memakan banyak CPU

{% hint style="warning" %}
**Tips Kinerja**: Untuk kinerja Chloros+ terbaik, tutup aplikasi lain dan biarkan Chloros menggunakan seluruh sumber daya sistem.
{% endhint %}

### Pemrosesan Tidak Dapat Dijeda (Tetapi Penghentian Dilakukan dengan Tuntas)

* Setelah dimulai, pemrosesan tidak dapat dijeda dan dilanjutkan kembali nanti
* Mengklik **Stop** akan menghentikan proses secara tuntas pada klik pertama
* Produk yang telah diekspor sebelum penghentian tetap tersimpan di disk
* Proses yang dihentikan akan melaporkan secara jujur apa yang telah diselesaikannya (lihat baris `[RUN-SUMMARY]` dalam log)
* Proses baru akan memulai alur kerja dari awal

**Tips perencanaan:** Untuk proyek yang sangat besar, pertimbangkan untuk memproses secara bertahap atau menggunakan CLI untuk kontrol yang lebih baik.***

## Memantau Pemrosesan Anda

Saat pemrosesan sedang berjalan, Anda dapat:

* **Memantau bilah kemajuan** - Lihat persentase penyelesaian secara keseluruhan
* **Melihat tahap saat ini** - Deteksi, Analisis, Kalibrasi, atau Ekspor
* **Memeriksa tab log** - Lihat pesan dan peringatan pemrosesan secara terperinci
* **Melihat pratinjau gambar yang telah selesai** - File ekspor muncul di disk selama pemrosesan

Untuk informasi terperinci mengenai pemantauan, lihat [Memantau Pemrosesan](monitoring-the-processing.md).

***

## Menghentikan Pemrosesan

Jika Anda perlu menghentikan pemrosesan:

### Cara Menghentikan

1. Temukan **tombol Stop** (menggantikan tombol Mulai selama pemrosesan)
2. Klik sekali — bilah akan menampilkan **&quot;Sedang dihentikan...&quot;** sementara gambar yang sedang diproses selesai
3. Pemrosesan berakhir dalam keadaan terhenti secara definitif dan log mencatat laporan `[RUN-SUMMARY]` yang jujur tentang apa yang telah diselesaikan

### Kapan Harus Menghentikan

**Alasan yang sah untuk menghentikan:**

* Menyadari bahwa pengaturan yang digunakan salah
* Lupa menandai gambar target
* Gambar yang diimpor salah
* Sistem berjalan terlalu lambat atau tidak merespons

**Setelah menghentikan:**

* Produk yang diekspor sebelum penghentian tetap tersimpan di disk
* Periksa dan perbaiki masalah apa pun, sesuaikan pengaturan sesuai kebutuhan
* Mulai ulang pemrosesan — proses dimulai dari awal

***

## Perkiraan Waktu Pemrosesan

Waktu pemrosesan aktual sangat bervariasi tergantung pada:

* Jumlah gambar
* Resolusi gambar
* Format input RAW vs JPG
* Mode pemrosesan (Free vs Chloros+)
* Kecepatan CPU dan jumlah inti
* Ketersediaan GPU (hanya Chloros+)
* Jumlah indeks yang harus dihitung
* Jumlah produk ekspor yang diaktifkan (LATTICE)

### Perkiraan Kasar (Chloros+, gambar 12MP, CPU modern)

| Jumlah Gambar | Mode Gratis | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 gambar   | 15–20 menit | 5–8 menit        | 3–5 menit        |
| 100 gambar  | 30–40 menit | 10–15 menit      | 5–8 menit        |
| 200 gambar  | 1-1,5 jam | 20-30 menit      | 10-15 menit      |
| 500 gambar  | 2-3 jam   | 45-60 menit      | 20-30 menit      |
| 1.000 gambar | 4–6 jam   | 1,5–2 jam      | 40–60 menit      |

{% hint style="info" %}
**Penggunaan Pertama**: Pemrosesan awal mungkin memakan waktu lebih lama karena Chloros sedang membangun cache dan profil. Pemrosesan selanjutnya pada dataset serupa akan lebih cepat.
{% endhint %}

***

## Masalah Umum Saat Memulai

### Tombol Mulai Dinonaktifkan (Berwarna Abu-abu)

**Penyebab yang mungkin:**

* Belum ada gambar yang diimpor
* Backend belum sepenuhnya dimulai
* Pemrosesan sebelumnya masih berjalan
* Proyek belum sepenuhnya dimuat

**Solusi:**

1. Tunggu hingga backend sepenuhnya diinisialisasi (periksa ikon menu utama)
2. Pastikan gambar telah diimpor di File Browser
3. Mulai ulang Chloros jika tombol tetap dinonaktifkan
4. Periksa Log Debug untuk pesan kesalahan

### Pemrosesan Dimulai Namun Segera Gagal

**Kemungkinan penyebab:**

* Tidak ada gambar yang valid dalam proyek
* File gambar rusak
* Ruang disk tidak mencukupi
* Memori (RAM) tidak mencukupi

**Solusi:**

1. Periksa Log Debug <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> untuk pesan kesalahan
2. Pastikan ruang disk tersedia
3. Coba proses subset gambar yang lebih kecil
4. Pastikan gambar tidak rusak

### Pemrosesan Selesai Namun Tidak Menulis Gambar Apa Pun

Pemrosesan yang meminta produk gambar namun tidak menulis apa pun dianggap sebagai **kegagalan, bukan keberhasilan** — Chloros melaporkannya dengan jelas:

* Log GUI menampilkan petunjuk `[RUN-SUMMARY]` yang menyebutkan kemungkinan penyebabnya — tidak ada gambar yang diimpor, tidak ada target yang terdeteksi, atau setiap produk yang diminta dilewati karena tidak berlaku (misalnya, meminta radiance/reflectance dari kamera yang hanya mendukung RGB)
* Versi setara CLI (`chloros-cli process`) menampilkan pesan `Processing finished but wrote no image products.` dan **keluar dengan nilai non-nol**, sehingga skrip dapat mendeteksinya
* Eksekusi yang sengaja hanya menghasilkan metadata (semua produk ekspor dinonaktifkan, tanpa indeks) tetap dianggap berhasil

Lihat [Referensi CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) untuk penjelasan lengkapnya.

### Peringatan &quot;Tidak Ada Target yang Terdeteksi&quot;

**Kemungkinan penyebab:**

* Lupa menandai gambar target
* Gambar target tidak mengandung target yang terlihat
* Pengaturan deteksi target terlalu ketat

**Solusi:**

1. Tinjau [Memilih Gambar Target](choosing-target-images.md)
2. Tandai gambar yang sesuai di kolom Target
3. Pastikan target terlihat di gambar yang ditandai
4. Sesuaikan pengaturan deteksi target jika diperlukan

***

## Tips untuk Pemrosesan yang Sukses

### Sebelum Memulai

1. **Uji coba dengan subset kecil terlebih dahulu** - Olah 10-20 gambar untuk memverifikasi pengaturan
2. **Periksa ruang disk yang tersedia** - Pastikan tersedia ruang kosong sebesar 2-3 kali ukuran dataset (lebih banyak jika semua produk LATTICE diaktifkan)
3. **Tutup aplikasi yang tidak diperlukan** - Bebaskan sumber daya sistem
4. **Pastikan kualitas gambar target** - Pratinjau target yang telah ditandai untuk memastikan kualitasnya
5. **Simpan proyek** - Proyek akan disimpan secara otomatis, namun disarankan untuk menyimpannya secara manual

### Selama Pemrosesan

1. **Hindari mode tidur sistem** - Nonaktifkan mode hemat daya
2. **Jaga agar Chloros tetap di latar depan** - Atau setidaknya tetap terlihat di bilah tugas
3. **Pantau kemajuan sesekali** - Periksa adanya peringatan atau kesalahan
4. **Jangan jalankan aplikasi berat lainnya** - Terutama saat menggunakan mode paralel Chloros+

### Akselerasi GPU Chloros+

Jika menggunakan akselerasi GPU NVIDIA:

1. Perbarui driver NVIDIA ke versi terbaru
2. Pastikan GPU memiliki VRAM 4 GB atau lebih (7 GB atau lebih untuk debayering Texture Aware secara bersamaan)
3. Tutup aplikasi yang memakan banyak GPU (game, pengeditan video)
4. Pantau suhu GPU (pastikan pendinginan memadai)

***

## Langkah Selanjutnya

Setelah pemrosesan dimulai:

1. **Pantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)
2. **Tunggu hingga selesai** - Pemrosesan berjalan secara otomatis
3. **Tinjau hasilnya** - Lihat [Menyelesaikan Pemrosesan](finishing-the-processing.md)

Untuk informasi tentang apa yang harus dilakukan selama pemrosesan, lihat [Memantau Pemrosesan](monitoring-the-processing.md).

# GUI: Navigasi

Saat Anda pertama kali menjalankan Chloros, sistem akan memulai proses backend-nya. Setelah backend siap, ikon menu utama di pojok kiri atas akan muncul <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> dan tab "Kamera" serta "Sensor Cahaya" di bilah sisi kiri akan aktif (sebelumnya keduanya dalam keadaan tidak aktif).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Dari kiri ke kanan, bilah atas berisi:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu Utama

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>Dari menu utama, Anda dapat:

* **Proyek Baru**— membuat proyek baru. Jika Anda telah menyimpan templat proyek, menu tarik-turun**Pilih Templat** akan muncul sehingga proyek baru dimulai dari pengaturan templat tersebut.
* **Buka Proyek**— membuka proyek yang sudah ada. Daftar tersebut mencakup tombol**Buka Folder Proyek** yang membuka folder proyek di penjelajah file Anda.
* **Duplikat Proyek** — menyalin proyek yang sedang terbuka dengan nama baru (disarankan menggunakan nama bebas seperti "MyProject (2)") dan membuka salinannya. _(tampak setelah proyek dibuka)_
* **Tambahkan File** — tambahkan file gambar satu per satu ke proyek saat ini _(tampak setelah proyek dibuka)_
* **Tambahkan Folder** — tambahkan satu atau lebih folder gambar ke proyek saat ini _(tampak setelah proyek dibuka)_
* **Mulai Pemrosesan / Hentikan Pemrosesan** — mulai atau hentikan alur pemrosesan gambar _(diaktifkan setelah berkas ditambahkan)_
* **Hubungkan ke Kamera** — buka [tab Kamera](lattice/) untuk menghubungkan kamera atau susunan kamera LATTICE. Dapat digunakan tanpa harus membuka proyek.
* **Hubungkan ke Sensor Cahaya** — beralih ke [tab Sensor Cahaya](daq/) untuk menghubungkan sensor cahaya DAQ. Dapat digunakan tanpa harus membuka proyek.

{% hint style="info" %}
**Hanya Windows**: Antarmuka Pengguna Grafis (GUI) Chloros Desktop tersedia di Windows. Pengguna Linux disarankan untuk melihat dokumentasi [CLI](CLI.md) dan [Python SDK](api-python-sdk.md) untuk pemrosesan tanpa antarmuka grafis.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Tombol Putar/Mulai

Jika diaktifkan, tombol mulai pemrosesan akan menjalankan alur kerja pemrosesan gambar.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Bilah Kemajuan <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



Dalam mode Chloros gratis, yang memproses semua file secara berurutan, bilah kemajuan akan menampilkan 2 tahap: Deteksi Target dan Pemrosesan.

Dalam mode berlisensi Chloros+ berbayar, yang memproses semua file secara bersamaan, bilah kemajuan menampilkan 4 tahap: Deteksi, Analisis, Kalibrasi, dan Ekspor. Jika Anda mengarahkan kursor mouse ke atas bilah kemajuan Chloros+, panel bilah kemajuan yang diperluas akan muncul sehingga Anda dapat memantau prosesnya. Mengklik bilah kemajuan bagian atas akan membekukan panel dropdown tersebut, sedangkan mengkliknya lagi akan mencairkannya.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu Samping

Menu bilah samping kiri berisi berbagai ikon yang dapat diinteraksikan, dalam urutan dari atas ke bawah:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Pengaturan Proyek](project-settings/project-settings.md)

Tab Pengaturan Proyek memungkinkan Anda menyesuaikan pengaturan global proyek dan pengaturan pemrosesan proyek. Sesuaikan pengaturan ini sebelum mulai memproses file Anda.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Penjelajah File

Tambahkan file/folder dan hapus file dari proyek. File duplikat akan diabaikan. Centang kotak kolom target untuk gambar target mana pun, dan pemrosesan hanya akan memeriksa gambar yang dicentang sebagai target, sehingga sangat mempercepat waktu pemrosesan Anda. Gunakan tombol beralih Gambar/Metadata untuk beralih antara tampilan kisi thumbnail gambar yang dipilih dan tabel metadata terperinci.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Penampil Gambar](image-viewer-gui/opening-an-image-full-screen.md)

Saat sebuah gambar diklik di penampil gambar utama, gambar tersebut akan dibuka dalam layar penuh di tab Penampil Gambar.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Penampil Peta](image-viewer-gui/map-markers.md)

Lihat gambar-gambar Anda pada peta 2D interaktif berdasarkan koordinat GPS-nya. Mendukung penyedia peta Google Maps dan ESRI, serta secara otomatis memilih layanan terbaik untuk lokasi Anda. Arahkan kursor ke penanda untuk melihat pratinjau thumbnail gambar.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Kamera](lattice/)

Hubungkan dan kendalikan kamera LATTICE secara langsung — satu per satu atau sebagai susunan multi-kamera yang tersinkronisasi. Tab ini menampilkan ubin pratinjau langsung dengan overlay dan histogram, pengaturan per-kamera dan per-susunan, serta Pengaturan Pengambilan yang menentukan kamera dan jenis ekspor mana yang dihasilkan oleh "Capture All". Tersedia setelah backend siap; lihat [bagian LATTICE](lattice/) untuk panduan lengkapnya.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Sensor Cahaya](daq/)

Hubungkan sensor cahaya DAQ — DAQ-U (USB), DAQ-M (Bluetooth), dan DAQ-E (Ethernet) — dan lihat grafik spektrum terkalibrasi secara langsung dalam satuan W/m²/nm. Dari sini, Anda dapat merekam berkas `.daq` ke dalam proyek yang terbuka, mengganti nama sensor, memilih profil koreksi kap, dan memperbarui firmware DAQ-E. Fitur ini tersedia setelah backend siap; lihat [bagian DAQ](daq/) untuk panduan lengkapnya.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Log Debug

Periksa log untuk pesan debug saat terjadi masalah. Salin/unduh log tersebut dan kirimkan ke [Dukungan MAPIR](https://www.mapir.camera/community/contact) untuk mendapatkan bantuan.

Yang baru di versi 1.3.0: ikon bug di bagian atas tab log (**Laporkan bug**) akan mengirimkan laporan ke MAPIR yang berisi deskripsi Anda, hingga 10 gambar, log Chloros, dan detail sistem. Anda harus masuk (sign in) untuk mengirimkan laporan tersebut. Lihat [Melaporkan bug dari tab log](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Masuk Pengguna](chloros+-login.md)

Bilah samping masuk pengguna memungkinkan Anda masuk ke akun Chloros+ Anda untuk membuka fitur-fitur lanjutan. Anda juga dapat melihat versi aplikasi saat ini, serta menyesuaikan bahasa teks yang ditampilkan di antarmuka pengguna grafis (GUI) dan antarmuka baris perintah (CLI) Chloros.

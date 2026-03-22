# GUI: Navigasi

Saat pertama kali membuka Chloros dan Chloros (Browser), sistem backend-nya akan dimulai. Setelah siap, ikon menu utama di pojok kiri atas akan muncul <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Dari kiri ke kanan, bagian header atas berisi:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu Utama

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

Dari menu utama, Anda dapat:

* **Proyek Baru** — membuat proyek baru
* **Buka Proyek** — membuka proyek yang sudah ada
* **Buka Folder Proyek** — membuka folder proyek di penjelajah file Anda
* **Tambahkan File** — menambahkan file gambar individual ke proyek saat ini _(tampak setelah proyek dibuka)_
* **Tambahkan Folder** — tambahkan folder gambar ke proyek saat ini _(tampak setelah proyek dibuka)_
* **Mulai Pemrosesan / Hentikan Pemrosesan** — mulai atau hentikan alur pemrosesan gambar _(diaktifkan setelah berkas ditambahkan)_

{% hint style="info" %}
**Hanya Windows**: Antarmuka Pengguna Desktop Chloros tersedia di Windows. Pengguna Linux harus melihat [CLI](CLI.md) dan [Python SDK](api-python-sdk.md) untuk pemrosesan tanpa antarmuka grafis.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Tombol Putar/Mulai

Jika diaktifkan, tombol mulai pemrosesan akan memulai alur pemrosesan gambar.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Bilah Kemajuan <img src=".gitbook/assets/image (5).png" alt="" data-size="line">Dalam mode gratis Chloros, yang memproses semua file secara berurutan, bilah kemajuan akan menampilkan 2 tahap: Deteksi Target dan Pemrosesan.

Dalam mode berlisensi berbayar Chloros+, yang memproses semua file secara bersamaan, bilah kemajuan menampilkan 4 tahap: Deteksi, Analisis, Kalibrasi, Ekspor. Jika Anda mengarahkan kursor mouse ke atas bilah kemajuan Chloros+, panel bilah kemajuan yang diperluas akan muncul sehingga Anda dapat mengikuti prosesnya. Mengklik bilah kemajuan bagian atas akan membekukan panel dropdown, mengklik lagi akan membukanya kembali.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu Samping

Menu bilah samping kiri berisi berbagai ikon untuk berinteraksi:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Pengaturan Proyek](project-settings/project-settings.md)

Tab Pengaturan Proyek memungkinkan Anda menyesuaikan pengaturan global proyek dan pengaturan pemrosesan proyek. Sesuaikan pengaturan ini sebelum memulai pemrosesan file Anda.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Penjelajah File

Tambahkan file/folder dan hapus file dari proyek. File duplikat diabaikan. Centang kotak kolom target untuk gambar target mana pun, dan pemrosesan hanya akan memeriksa gambar yang dicentang sebagai target, sehingga sangat mempercepat waktu pemrosesan Anda. Gunakan tombol beralih Gambar/Metadata untuk beralih antara tampilan kisi thumbnail gambar yang dipilih dan tabel metadata terperinci.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Penerima Gambar](image-viewer-gui/opening-an-image-full-screen.md)

Saat gambar diklik di penerima gambar utama, gambar tersebut akan dibuka dalam layar penuh di tab Penerima Gambar.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Peta](image-viewer-gui/map-markers.md)

Lihat gambar Anda pada peta 2D interaktif berdasarkan koordinat GPS-nya. Mendukung penyedia peta Google Maps dan ESRI, secara otomatis memilih layanan terbaik untuk lokasi Anda. Arahkan kursor ke penanda untuk melihat pratinjau thumbnail gambar.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Log Debug

Periksa log untuk cetakan debug saat terjadi masalah. Salin/unduh log dan kirimkan ke [Dukungan MAPIR](https://www.mapir.camera/community/contact) untuk mendapatkan bantuan.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Masuk Pengguna](chloros+-login.md)

Panel samping masuk pengguna memungkinkan Anda masuk ke akun Chloros+ Anda untuk membuka fitur-fitur lanjutan. Anda juga dapat melihat versi aplikasi saat ini, serta menyesuaikan bahasa teks yang ditampilkan di antarmuka pengguna (GUI) Chloros dan CLI.

# Membuka Gambar dalam Layar Penuh

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Gambar yang dibuka dalam mode layar penuh, dengan pemilih lapisan di pojok kanan atas</p></figcaption></figure>

Chloros Image Viewer adalah antarmuka layar penuh untuk melihat, memeriksa, dan mengukur gambar Anda. Di sinilah Anda dapat membaca **nilai piksel sebenarnya** — DN per kanal, persentase reflektansi, atau radian dalam W/m²/sr/nm — alih-alih pratinjau yang diperbesar yang ditampilkan di layar.

## Mengakses Image Viewer

### Dari Penjelajah File

1. Buka tab **Penjelajah File** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Klik **thumbnail** mana pun di [kisi gambar](image-grid.md)
3. Gambar akan terbuka dalam layar penuh di tab **Penampil Gambar**

Gambar akan terbuka pada produk apa pun yang ditampilkan oleh kisi tersebut. Jika kisi diatur ke `RAW (Reflectance)`, maka lapisan itulah yang akan Anda buka.

### Membuka panel samping **Image Viewer**Klik ikon**Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di panel samping kiri untuk membuka panel analisis. Panel tersebut berisi, dari atas ke bawah:

* nama gambar dan model kameranya
* tombol **Ekspor/Simpan Gambar** (hanya aktif jika Indeks atau LUT sedang aktif)
* kotak centang **Index**dan**LUT** serta panel konfigurasi indeks — lihat [Index/LUT Sandbox](index-lut-sandbox.md)
* panel **Nilai Kursor**: pembacaan per saluran, histogram lapisan, dan pengaturan GSD***

## Menavigasi dan memperbesar/memperkecil

### Menelusuri gambar

* **Gambar berikutnya**: tombol →, atau tombol**→** (Panah Kanan)
* **Gambar sebelumnya**: tombol ←, atau tombol**←** (Panah Kiri)
* **Lompat ke gambar tertentu**: kembali ke kisi dan klik thumbnail-nya

Perbesaran dan pergeseran tetap berlaku saat Anda berpindah antar gambar, sehingga Anda dapat menelusuri serangkaian gambar sambil tetap berada di bagian yang sama dari bingkai.

### Perbesaran

Perbesaran dikendalikan oleh **roda mouse**, dengan langkah 15%, dengan titik acuan pada kursor — titik di bawah penunjuk tetap berada di bawah penunjuk. Rentang zoom dibatasi oleh ukuran gambar dan jendela: Anda tidak dapat memperkecil hingga melebihi batas &quot;sesuai jendela&quot;, dan batas atas ditentukan oleh resolusi asli gambar.

Tidak ada tombol zoom khusus di penampil layar penuh. (Di kisi, **Ctrl + `+` / `−`** mengubah ukuran thumbnail — kontrol yang berbeda.)

### Menggeser saat diperbesar

Klik dan tahan tombol kiri mouse di atas gambar, lalu seret. Pergerakan horizontal dibatasi sehingga gambar tidak dapat diseret keluar layar.

### Pemeriksaan per piksel pada pembesaran tinggi

Setelah pembesaran efektif melebihi **60×**, Chloros menampilkan kotak sorotan di sekitar piksel yang ditampilkan di bawah kursor serta nilai mengambang di sebelahnya.

Pembesaran &quot;efektif&quot; menghitung ukuran blok GSD: dengan ukuran blok 8, sorotan muncul pada pembesaran 7,5×, bukan 60×, karena satu piksel yang ditampilkan sudah terdiri dari 8 × 8 piksel sumber. Kecilkan kembali pembesaran hingga di bawah ambang batas tersebut dan sorotan akan menghilang.

### Pintasan keyboard

| Tombol                             | Di mana       | Tindakan                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Layar Penuh | Gambar berikutnya                          |
| **←**                           | Layar Penuh | Gambar sebelumnya                      |
| **Ctrl + R**                    | Layar Penuh | Atur ulang kotak pasir indeks/LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Kisi            | Gambar mini yang lebih besar (4 piksel per tekan)  |
| **Ctrl + `−`**                  | Kisi        | Gambar mini yang lebih kecil (4 piksel per tekan) |***

## Nilai Kursor

Arahkan kursor ke atas gambar dan panel **Nilai Kursor** akan menampilkan nilai setiap kanal di bawahnya.

{% hint style="success" %}
**Ini adalah angka sebenarnya dari berkas tersebut.** Kanvas di layar adalah pratinjau yang diperbesar 8-bit dan tidak dapat menampilkannya, sehingga Chloros mengambil sampel dari file produk yang sebenarnya untuk menampilkan nilainya. Itulah sebabnya bingkai mentah 12-bit menampilkan nilai di atas 255, dan lapisan radian float32 menampilkan satuan fisik.
{% endhint %}

### Arti kolom-kolom tersebut

Panel ini menyesuaikan dengan lapisan yang sedang Anda lihat:

| Lapisan yang Anda lihat              | Kolom yang ditampilkan    | Catatan                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflektansi                        | **DN**dan**%** | Persentase dihitung berdasarkan skala file tersebut — lihat di bawah                                      |
| Radian                           | **W/m²/sr/nm**   | Nilai fisik tipe float; tidak ada kolom DN, karena DN tidak relevan di sini                           |
| Raw / Debayered / pratinjau / JPG    | **DN**           | Bilangan digital bilangan bulat                                                                         |
| Ekspor reflektansi persentase 32-bit | **%** saja       | Bilangan desimal yang disimpan bukanlah DN, sehingga pembulatan ke bilangan bulat akan menghasilkan nilai yang tidak bermakna seperti `0` atau `1` |

Setiap baris diberi label dengan nama kanal sesuai filter kamera Anda — `Red / Green / NIR` untuk RGN, `Orange / Cyan / NIR` untuk OCN, `NIR / Green / Blue` untuk NGB, `Red / Green / Blue` untuk RGB, serta nama pita tunggal untuk kamera RE, NIR dan kamera mono M3M. Setiap label memiliki titik berwarna yang sesuai dengan lingkaran saluran yang digunakan dalam editor rumus indeks.

Gambar **indeks dan LUT** yang disimpan merupakan kasus khusus: gambar-gambar tersebut berisi komponen peta warna alih-alih pita spektral, sehingga baris-barisnya diberi label `Red / Green / Blue` (atau `Index` untuk berkas indeks saluran tunggal) alih-alih menggunakan nama filter kamera.

Saat sebuah indeks aktif di kotak pasir, baris tambahan akan muncul di bawah saluran yang menunjukkan **nilai indeks** pada kursor, bersama dengan nama indeks dan titik putih yang sesuai dengan penanda pada histogram.

### Persentase reflektansi menggunakan skala masing-masing file

{% hint style="warning" %}
**Jangan berasumsi bahwa 65535 = 100%.** Chloros menyimpan reflektansi pada skala yang berbeda-beda tergantung pada kamera yang menghasilkannya, dan penampil akan menentukan skala yang benar untuk setiap file.
{% endhint %}

| Sumber                  | DN yang setara dengan reflektansi 1,0 | Cara mengidentifikasinya                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Tag XMP `Chloros:PixelScale=32768` ditulis ke dalam setiap ekspor nilai reflektansi LATTICE. Ruang kosong sebesar 2× memungkinkan berkas tersebut menampung ρ di atas 1,0 tanpa pemotongan |
| **Survey3**|**65535**                      | Tidak ada tag skala XMP Chloros — kalibrasi Survey3 menulis ρ × dtype-max dan memotong pada 1,0                                                               |

Penampil, kotak pasir indeks/LUT, dan ekspor indeks semuanya menyelesaikan skala melalui implementasi tunggal yang sama, sehingga nilai yang Anda baca di kursor adalah nilai yang sama dengan yang digunakan oleh perhitungan indeks.

Dua konsekuensi yang perlu diketahui:

* Sebuah **persentase 32-bit**TIFF menyimpan DN/65535 sebagai bilangan floating-point, sedangkan ekspor**8-bit** PNG/JPG menyimpan DN × 255/65535 — penampil akan mengonversi keduanya kembali sebelum menampilkan persentase.
* Satu kasus tidak dapat dipulihkan: **ekspor 8-bit TIFF dari tangkapan sumber 8-bit** dipotong menjadi 0–255 alih-alih diskalakan ulang, dan secara sengaja tidak menyertakan tag skala. Untuk file-file tersebut, panel hanya mencetak DN, tanpa kolom persentase. Ini adalah jawaban yang jujur, bukan bug.***

## Histogram lapisan

Di bawah baris kursor terdapat histogram langsung dari lapisan yang Anda lihat, dalam **256 bin**. Secara default, histogram ini menggambar satu kurva gabungan, yang diberi bobot `(R + 2G + B) / 4` — ruang pengukuran yang sama dengan yang digunakan histogram kamera LATTICE. Mengaktifkan**RGB**, kurva tersebut akan diganti dengan kurva per-saluran dalam warna saluran masing-masing, yang dicampur secara aditif sehingga tumpang tindih tetap dapat dibaca. Lapisan mono selalu menampilkan kurva tunggal.

Sumbu horizontal menggunakan satuan lapisan itu sendiri:

| Lapisan       | Satuan sumbu  | Nilai maksimum sumbu                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Reflektansi | persen    | 125% — ruang kosong produk memungkinkan ρ di atas 1,0           |
| Radian    | W/m²/sr/nm | Puncak bingkai itu sendiri, dibulatkan ke atas hingga dua angka signifikan |
| Data 8-bit  | DN         | 255                                                        |
| Data 12-bit | DN         | 4095                                                       |
| Data 16-bit | DN         | 65535                                                      |

Ketika sumbu berada dalam satuan DN dan berada pada salah satu dari tiga batas atas tersebut, Chloros juga mengetahui kedalaman bit dari apa yang Anda lihat.

Tiga tombol terletak di atas histogram:

| Tombol     | Default | Efek                                                                                                                                                                                                                                                                                                           |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Aktif      | Menggambar garis penanda pada histogram pada nilai yang tepat seperti yang ditampilkan pada baris di atas, sehingga Anda dapat melihat di mana piksel di bawah kursor Anda berada dalam distribusi bingkai. Dalam mode RGB, terdapat satu penanda per kanal dengan warnanya masing-masing; sebaliknya, hanya ada satu penanda putih pada nilai gabungan |
| **INDEX**| Aktif      | Hanya muncul saat indeks aktif. Mengalihkan histogram dari pita sumber ke**distribusi nilai indeks**, dengan dua ambang batas klip digambar sebagai garis putus-putus oranye dan nilai indeks kursor sebagai garis putih                                                          |
| **RGB**| Nonaktif     | Beralih dari kurva gabungan ke kurva per kanal. Pada sensor mono, tombol ini bertuliskan**MONO** dan dinonaktifkan — hanya ada satu kanal yang ditampilkan                                                                                                                                  |

Histogram dihitung berdasarkan **blok yang terlihat**, bukan piksel sumber di belakangnya: ubah ukuran blok GSD dan distribusinya akan dihitung ulang, sehingga histogram, penanda kursor, dan gambar yang ditampilkan selalu sesuai.***

## Ukuran blok GSD

Di bagian bawah panel terdapat kontrol **GSD (px)**: kotak angka, penggeser dari**1 hingga 256**, dan tombol**RESET**.

Kontrol ini memperkasar gambar yang _ditampilkan_ dengan merata-ratakan blok N × N piksel sumber menjadi satu piksel yang ditampilkan. `1` adalah resolusi asli.

* Pengaturan ini memengaruhi **tampilan layar penuh, thumbnail kisi, pembacaan kursor, dan kedua histogram** — semua elemen yang menampilkan gambar akan menggunakan resolusi dasar yang sama.
* Pengaturan ini **hanya berlaku untuk tampilan**. Pemrosesan dan ekspor tidak terpengaruh. Satu-satunya pengecualian adalah hal yang disengaja: ekspor [Index/LUT Sandbox](index-lut-sandbox.md) menyimpan apa yang Anda lihat, sehingga membawa ukuran blok saat ini, dan panel ekspor akan memperingatkan Anda ketika ukuran blok melebihi 1.
* Nilai tersebut disimpan **per proyek** sebagai `viewer_display.gsd_bin` di `project.json`, sehingga tetap tersimpan meskipun aplikasi ditutup dan dibuka kembali.
* Tampilan kursor melaporkan blok, bukan piksel sumber, setiap kali ukuran blok melebihi 1 — nilai yang ditampilkan adalah rata-rata blok di bawah kursor Anda.

{% hint style="info" %}
**Mengapa &quot;ukuran blok&quot; dan bukan sentimeter per piksel?** Angka cm/px memerlukan ketinggian di atas permukaan tanah. EXIF pada satu bingkaiEXIF hanya memuat ketinggian GPS di atas permukaan laut rata-rata, bukan di atas medan yang menjadi sasaran, sehingga Chloros tidak akan menampilkan jarak ke permukaan tanah yang tidak dapat dihitung secara akurat. Ukuran blok dalam piksel sumber merupakan alternatif yang sama yang digunakan oleh alat awan MAPIR ketika jarak sampel permukaan tanah tidak diketahui.
{% endhint %}

***

## Jenis gambar yang dapat Anda lihat

Menu tarik-turun lapisan di pojok kanan atas penampil mencantumkan setiap versi dari gambar saat ini. Entri mana yang muncul bergantung pada kamera dan apa yang telah diproses — lihat [Lapisan Gambar](image-layers.md) untuk daftar lengkap dan cara kerja menu tarik-turun tersebut.

### Survey3

* **JPG** — berkas pratinjau dari kamera itu sendiri
* **RAW (Asli)** — sumber `.RAW`, telah melalui proses debayering untuk tampilan, tanpa koreksi
* **RAW (Target)** — bingkai yang diidentifikasi mengandung target kalibrasi
* **RAW (Reflektansi)** — produk reflektansi yang telah dikalibrasi (65535 = ρ 1,0)
* **Vignette Corrected**/**Sensor Response** — produk cadangan yang belum dikalibrasi
* **White Balanced** — produk yang telah disesuaikan keseimbangan putihnya
* **RAW (Indeks `<INDEX>`)**dan**LUT `<INDEX>`** — gambar indeks yang dihitung

### LATTICE

Pengambilan gambar LATTICE menggunakan menu tarik-turun yang sama, dengan nama-nama level dalam alur kerja:

| Lapisan                 | Isi lapisan                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Asli)**    | Frame RAW sumber sebagaimana direkam                                     |
| **RAW (Debayered)**   | Gambar linear yang telah melalui proses debayering                                           |
| **RAW (Preview)**     | Pratinjau tampilan — rentang warna palsu untuk kamera multispektral |
| **White Balanced**    | Pratinjau tampilan untuk kamera utama RGB (white balance + gamma)   |
| **RAW (Radiance)**    | Radian spektral Float32 dalam W/m²/sr/nm                              |
| **RAW (Reflektansi)** | reflektansi uint16, 32768 = ρ 1,0                                    |

Radiance dan reflektansi hanya tersedia dalam mode multispektral: kamera utama RGB tidak memiliki radiometri per-band, sehingga lapisan-lapisan tersebut tidak dihasilkan untuk kamera tersebut.

***

## Penerapan Indeks dan LUT

Terapkan indeks multispektral dan Tabel Pencarian Warna (Look-Up Tables) dari bilah samping:

1. Buka **Image Viewer** di bilah sisi <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Centang **Index**

3. Pilih filter kamera Anda dan rumus indeks, lalu seret lingkaran kanal ke slot rumus tersebut
4. Tambahkan LUT dan pilih gradien, ambang batas, serta mode pemotongan
5. Baca nilai di posisi kursor, lalu simpan hasilnya dengan **Export/Save Image(s)**Lihat [Index/LUT Sandbox](index-lut-sandbox.md) untuk panduan lengkapnya.***

## Pemecahan Masalah

### Gambar tidak dapat dibuka

**Kemungkinan penyebab**: file dipindahkan atau dihapus setelah diimpor; produk tidak pernah disimpan; memori tidak cukup untuk gambar yang sangat besar.**Apa yang harus dilakukan**:

1. Periksa apakah berkas lapisan tersebut masih ada di pohon keluaran proyek
2. Buka berkas tersebut di penampil eksternal untuk memastikan berkas tersebut utuh
3. Tutup aplikasi lain untuk membebaskan memori

### Gambar berwarna hitam, putih, atau berwarna-warni secara tidak wajar

**Kemungkinan penyebab**: fitur &quot;display stretch&quot; tidak memiliki data untuk diproses (frame yang hampir konstan); lapisan float32 dengan nilai yang tidak biasa; indeks yang tidak menghasilkan data yang valid.**Apa yang harus dilakukan**:

1. Periksa nilai kursor — jika setiap kanal bernilai nol atau mendekati nol, masalahnya ada pada data, bukan tampilan
2. Periksa histogram: lonjakan tunggal di salah satu ujung menandakan bingkai terpotong atau kosong
3. Periksa log pemrosesan untuk eksekusi yang menghasilkan lapisan tersebut

### Nilai-nilainya tampak salah

**Penyebab yang mungkin**: Anda berada di lapisan yang berbeda dari yang Anda kira; Anda membandingkan persentase dengan DN mentah; Anda membandingkan berkas LATTICE dengan berkas Survey3 menggunakan pembagi yang sama.**Apa yang harus dilakukan**:

1. Pastikan lapisan yang dipilih di menu tarik-turun — satuan pada panel mengikuti lapisan tersebut
2. Untuk reflektansi, gunakan kolom **%** daripada membagi nilai DN sendiri; jika Anda harus membagi, gunakan nilai `Chloros:PixelScale` dari berkas tersebut (32768 untuk LATTICE, jika tidak ada berarti 65535 untuk Survey3)
3. Atur kembali ukuran blok GSD ke 1 — jika nilainya di atas 1, Anda membaca rata-rata blok, bukan nilai piksel
4. Pastikan kalibrasi reflektansi benar-benar telah dijalankan untuk bingkai tersebut; produk cadangan yang belum dikalibrasi (Sensor Response / Vignette Corrected) bukanlah reflektansi

***

## Langkah Selanjutnya

* [**Lapisan Gambar**](image-layers.md) — nama setiap lapisan, jika ada, dan arti nilainya
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — buat, sesuaikan, dan ekspor visualisasi indeks
* [**Penanda Peta**](map-markers.md) — kumpulan gambar yang sama pada peta
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) — referensi indeks

Untuk alur kerja pemrosesan, lihat [Pemrosesan Gambar (GUI)](../processing-images-gui/adding-files-to-a-project.md).

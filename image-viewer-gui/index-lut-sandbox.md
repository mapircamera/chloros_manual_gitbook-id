# Kotak Pasir Indeks/LUT

Kotak Pasir Indeks/LUT adalah ruang kerja interaktif di dalam Chloros Image Viewer yang memungkinkan Anda bereksperimen dengan perhitungan indeks multispektral dan visualisasi warna secara real-time. Alat canggih ini membantu Anda menguji berbagai indeks, menyempurnakan rentang nilai, dan membuat visualisasi yang siap dipublikasikan tanpa perlu memproses ulang seluruh kumpulan data Anda.

## Apa itu Index/LUT Sandbox?

### Tujuan

Sandbox menyediakan:

* **Perhitungan indeks secara real-time** - Terapkan indeks vegetasi apa pun secara instan
* **Penyesuaian LUT interaktif** - Sesuaikan gradasi dan rentang warna
* **Optimasi alur kerja** - Tentukan pengaturan terbaik sebelum pemrosesan batch

### Sandbox vs. Pemrosesan Proyek

**Index/LUT Sandbox (Interaktif):**

* Satu gambar pada satu waktu
* Umpan balik instan
* Eksperimental dan berulang
* Tidak ada perubahan permanen pada file
* Sempurna untuk eksplorasi dan pengujian

**Pemrosesan Proyek (Batch):**

* Seluruh dataset sekaligus
* Pengaturan yang telah dikonfigurasi sebelumnya
* File output permanen
* Memakan waktu
* Terbaik saat pengaturan telah final

{% hint style="success" %}
**Alur Kerja Terbaik**: Gunakan Sandbox untuk bereksperimen dan menemukan pengaturan indeks dan LUT yang optimal, lalu terapkan pengaturan tersebut selama Pemrosesan Proyek untuk seluruh dataset Anda.
{% endhint %}

***

## Bekerja dengan Sandbox Indeks/LUT

### Memahami Indeks yang Telah Dihitung

Di Chloros, indeks dapat diterapkan selama pemrosesan proyek. Untuk menentukan pengaturan indeks dan LUT mana yang ingin Anda terapkan pada ekspor, cara termudah adalah menggunakan sandbox penampil gambar.

Sandbox memungkinkan Anda untuk:

* **Menerapkan indeks baru dan gradien warna (LUT)** untuk memvisualisasikan data
* **Menyesuaikan pengaturan visualisasi** secara interaktif
* **Melihat** gambar indeks yang sudah dihitung
* **Memeriksa** nilai piksel pada semua tingkat zoom

### Membuka Sandbox

Sandbox Indeks/LUT dapat diakses di tab <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab bilah samping:

1. Klik gambar di kisi gambar penjelajah file, gambar tersebut akan terbuka di tab **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> tab
2. Klik tab **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> untuk membuka bilah samping pop-out di sebelah kiri jika belum terbuka

### Memilih Gambar untuk Menerapkan Indeks/LUT

Untuk bekerja dengan indeks di <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Buka gambar** dari kisi gambar utama dengan mengkliknya
2. Tab **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> akan terbuka
3. Klik **menu tarik-turun Lapisan** (pojok kanan atas penampil)
4. Pilih lapisan dari menu tarik-turun:
   * RAW (Reflektansi)

### Menerapkan Indeks ke Gambar

Setelah gambar ditampilkan dalam layar penuh dan bilah samping tab **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sudah terbuka:

1. Centang kotak Indeks di bagian atas sidebar
2. Pilih filter kamera Anda dari menu tarik-turun sebelah kiri
3. Pilih rumus indeks yang diinginkan dari menu tarik-turun sebelah kanan
4. Seret lingkaran warna kanal filter ke lokasi yang sesuai dalam rumus indeks di bawah
5. Setelah rumus valid, gambar akan diperbarui dan menampilkan nilai indeks
6. Gerakkan kursor mouse Anda untuk melihat nilai di lokasi kursor
7. Perbesar tampilan untuk melihat piksel individu dan nilai yang terkait

Setiap indeks memiliki rentang nilai dan arti yang spesifik:

#### Contoh NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Untuk dokumentasi lengkap rumus indeks, lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md).

***

## Bekerja dengan LUT (Tabel Pencarian)

### Apa itu LUT?

Sebuah **Tabel Pencarian (LUT)** memetakan nilai indeks numerik ke warna untuk visualisasi:

* **Masukan**: Nilai piksel indeks (misalnya, NDVI 0,65)
* **Output**: Warna (misalnya, hijau cerah)
* **Tujuan**: Memudahkan pengamatan dan interpretasi pola**LUT Skala Abu-abu vs. LUT Warna:**

* Skala Abu-abu: Ilmiah dan netral, menampilkan data mentah
* LUT Warna: Intuitif dan berdampak, menonjolkan pola dan perbedaan

{% hint style="success" %}
**Kekuatan Visualisasi**: Menerapkan LUT warna pada gambar indeks skala abu-abu membuatnya jauh lebih mudah untuk mengidentifikasi pola, anomali, dan area yang menarik perhatian sekilas.
{% endhint %}

### Menerapkan LUT pada Gambar Indeks

Setelah Anda memiliki gambar indeks yang menampilkan

1. Klik tombol <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> tombol &quot;+Tambahkan LUT&quot;
2. Pilih gradien warna
3. Sesuaikan titik akhir minimum/maksimum pemotongan
4. Sesuaikan Mode Pemotongan
5. Centang kotak Indeks di bilah samping **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> di bilah samping untuk menerapkan LUT

### Memilih Gradien Warna

**Memilih gradien:**

1. Di panel LUT, temukan**bilah gradien berwarna**

2. Arahkan kursor mouse ke atasnya untuk melihat preset gradien yang tersedia
3. Pilih gradien yang diinginkan
4. Gambar **akan diperbarui secara instan** dengan warna baru saat kotak Indeks dicentang

{% hint style="success" %}
**Praktik Terbaik**: Untuk indeks vegetasi seperti NDVI, gradien Red-Kuning-Green paling intuitif karena selaras dengan asosiasi warna alami (hijau=sehat, kuning=sedang, merah=tertekan).
{% endhint %}

### Menyesuaikan Kelas Warna

Kontrol **Kelas**menentukan berapa banyak langkah warna terpisah yang muncul dalam gradien Anda:**Opsi jumlah kelas:*** **2-5 kelas**: Kategori yang sangat luas, zona yang jelas
* **6-10 kelas**: Seimbang, baik untuk klasifikasi
* **11-20 kelas**: Gradien halus, tampilan yang terus-menerus
* **20+ kelas**: Hampir terus-menerus, kelembutan maksimal**Cara menyesuaikan:**

1. Di panel LUT, temukan**kotak swatch warna di bawah bilah gradien**

2. Sesuaikan jumlah kelas dengan menambahkan menggunakan tombol +
3. Hapus jumlah kelas dengan mengklik ganda pada swatch warna
4. Gradien diperbarui **secara real-time** pada gambar**Dampak pada visualisasi:*** **Kelas lebih sedikit** (3-5): Membuat zona yang jelas, klasifikasi sederhana, lebih mudah membedakan kategori
* **Kelas sedang** (6-10): Pendekatan seimbang, cocok untuk sebagian besar aplikasi
* **Kelas lebih banyak** (15-20): Transisi halus, variasi detail, tampilan seperti foto**Kapan digunakan:*** **Sedikit kelas (3-5)**: Slide presentasi, peta klasifikasi, laporan sederhana
* **Kelas sedang (6-10)**: Analisis umum, detail seimbang, laporan standar
* **Banyak kelas (15-20)**: Analisis ilmiah, pemeriksaan terperinci, hasil berkualitas publikasi

### Penyesuaian Rentang Nilai

**Kontrol rentang nilai**menentukan nilai indeks mana yang dipetakan ke warna mana dalam gradien Anda:**Kontrol rentang di panel LUT:*** **Nilai minimum**: Batas bawah skala warna
* **Nilai maksimum**: Batas atas skala warna
* **Nilai antara**: Didistribusikan secara otomatis antara nilai minimum dan maksimum (berdasarkan jumlah kelas)

#### Menyesuaikan Nilai Minimum/Maksimum

**Untuk menyesuaikan rentang nilai:**

1. Di panel LUT, temukan bidang input**Nilai Minimum**dan**Nilai Maksimum**

2. Klik bidang**Nilai Minimum**

3. Ketik nilai minimum yang diinginkan (misalnya, `0.2`)
4. Tekan **Enter** atau klik di luar bidang
5. Ulangi untuk bidang **Nilai Maks** (misalnya, `0.9`)
6. Visualisasi **segera diperbarui**{% hint style="info" %}**Penyesuaian Otomatis**: Saat pertama kali menerapkan LUT, Chloros secara otomatis menetapkan nilai min/maks sesuai rentang data aktual pada gambar. Anda kemudian dapat mempersempit rentang ini untuk fokus pada rentang nilai tertentu yang diinginkan.
{% endhint %}

**Contoh penyesuaian rentang NDVI:*** **Rentang penuh**: `-1.0` hingga `1.0` (menampilkan semua nilai yang mungkin)
* **Berfokus pada vegetasi**: `0.2` hingga `0.9` (mengecualikan tanah telanjang dan air)
* **Hanya vegetasi sehat**: `0.5` hingga `0.9` (menyoroti hanya tanaman yang subur)
* **Deteksi stres**: `0.2` hingga `0.5` (tonjolkan area bermasalah)
* **Rentang kustom**: Sesuaikan berdasarkan nilai piksel yang Anda amati**Mengapa menyesuaikan rentang?*** **Meningkatkan kontras** di area yang menjadi fokus Anda
* **Mengecualikan nilai yang tidak relevan** (misalnya, badan air, tanah telanjang)
* **Menstandarkan visualisasi** di antara beberapa gambar atau tanggal
* **Menonjolkan perbedaan halus** dalam rentang nilai yang sempit

### Memotong Nilai di Luar Rentang

Ketika nilai piksel berada di luar rentang min/maks yang Anda tentukan, Anda dapat mengontrol cara penampilannya menggunakan **mode pemotongan**.

#### **Opsi mode pemotongan yang tersedia:**

#### 1. Minimum dan Maksimum

* Piksel **di bawah minimum**→ ditampilkan menggunakan**warna pertama** dalam gradien (misalnya, merah)
* Piksel **di atas maksimum**→ ditampilkan menggunakan**warna terakhir** dalam gradien (misalnya, hijau)
* **Kasus penggunaan**: Menonjolkan nilai ekstrem, menampilkan rentang data penuh dengan warna jenuh di batas-batas
* **Contoh**: Nilai NDVI di bawah 0,2 semuanya berwarna merah, nilai di atas 0,9 semuanya berwarna hijau

#### 2. Latar Belakang Transparan

* Piksel **di luar rentang**menjadi**sepenuhnya transparan*** Hanya piksel **dalam rentang** yang menampilkan gradien warna
* **Kasus penggunaan**: Overlay GIS, mengisolasi rentang nilai tertentu, menyoroti hanya area yang diminati
* **Contoh**: Tampilkan hanya NDVI 0,4–0,7 dalam warna, sisanya transparan

{% hint style="warning" %}
**Batasan Transparansi**: Piksel transparan akan ditampilkan sebagai warna latar belakang di pemirsa. Saat diekspor selama pemrosesan, transparansi dipertahankan dalam format PNG tetapi tidak dalam JPG.
{% endhint %}

#### 3. Latar Belakang Indeks

* Piksel **di luar rentang**ditampilkan dalam**skala abu-abu** (menampilkan nilai indeks mentah)
* Piksel **dalam rentang**menampilkan**gradien warna*** **Kasus penggunaan**: Penyorotan halus, mempertahankan konteks sambil menonjolkan area yang menarik
* **Contoh**: Menyorot vegetasi yang tertekan dengan warna (NDVI 0,3–0,5) sambil menampilkan area sehat dalam abu-abu

#### 4. Latar Belakang Asli

* Piksel **di luar rentang**menampilkan**gambar multispektral asli*** Piksel **dalam rentang**menampilkan**gradien warna*** **Kasus penggunaan**: Yang paling intuitif - menggabungkan konteks gambar alami dengan overlay warna analitis
* **Contoh**: Lihat penampilan lapangan/tanaman yang sebenarnya dengan area stres yang diberi kode warna ditampilkan di atasnya

### Memilih Mode Pemotongan yang Tepat

| Mode Pemotongan              | Cocok Untuk                                   | Gaya Visualisasi          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum dan Maksimum**    | Tampilan data lengkap, analisis ilmiah     | Semua piksel berwarna           |
| **Latar Belakang Transparan** | Overlay GIS, mengisolasi rentang tertentu    | Warna pada rentang, kosong di luar |
| **Latar Belakang Indeks**       | Penekanan halus, mempertahankan konteks data  | Berwarna pada rentang, abu-abu di luar  |
| **Latar Belakang Asli**    | Laporan, presentasi, analisis intuitif | Berwarna pada rentang, foto di luar |

### Membuat Warna LUT Kustom

Untuk kontrol penuh atas visualisasi Anda, Anda dapat membuat **gradien warna kustom** dengan mengedit titik warna individu.**Untuk membuat gradien kustom:**

1. Di panel LUT, temukan**bilah pratinjau gradien**

2. Cari**kotak contoh warna** di bawah gradien
3. **Klik titik warna** untuk memilihnya
4. **Pilih warna** akan terbuka
5. Pilih warna baru menggunakan:
   * **Roda warna**: Pemilihan warna visual
   * **Penggeser RGB/HSV**: Kontrol warna yang presisi
   * **Masukan kode hex**: Spesifikasi warna yang tepat (misalnya, `#FF0000` untuk merah)
6. Klik di luar pemilih warna **untuk menerapkan warna baru**

7. Gradien**terperbarui secara instan** pada gambar**Menambahkan atau menghapus titik warna:*** **Tambahkan titik warna**: Klik ikon + untuk menambahkan swatch baru di bagian akhir
* **Hapus titik warna**: Klik ganda kotak warna untuk menghapus swatch**Strategi penyesuaian:*** **Balikkan gradien**: Balik urutan warna untuk membalikkan arti (misalnya, hijau=rendah, merah=tinggi)
* **Warna merek**: Sesuaikan dengan palet warna organisasi Anda untuk laporan
* **Ramah bagi penderita buta warna**: Gunakan kombinasi oranye-biru atau ungu-kuning
* **Optimasi cetak**: Pilih warna yang berfungsi baik pada cetakan berwarna maupun skala abu-abu
* **Multi-ambang batas**: Gunakan warna berbeda pada ambang batas nilai tertentu untuk klasifikasi

{% hint style="info" %}
**Menyimpan Gradien Kustom**: Gradien kustom dapat disimpan dan digunakan kembali. Klik ikon simpan di panel LUT untuk menyimpan skema warna kustom Anda untuk penggunaan di masa mendatang.
{% endhint %}

***

## Alur Kerja Interaktif

### Pembaruan Real-Time

Semua penyesuaian LUT di area uji coba memperbarui gambar **secara instan dan interaktif**:

* **Ganti lapisan** → Gambar berubah seketika
* **Pilih gradien** → Warna diperbarui secara instan
* **Sesuaikan rentang nilai** → Kontras berubah secara real-time
* **Ubah kelas** → Kelancaran gradien diperbarui seketika
* **Modifikasi pemotongan** → Tampilan latar belakang berubah seketika
* **Edit warna** → Gradien kustom diterapkan seketika**Tidak perlu tombol &quot;Terapkan&quot;** - semua perubahan langsung dan interaktif!

{% hint style="success" %}
**Umpan Balik Langsung**: Umpan balik visual instan memungkinkan Anda bereksperimen dengan cepat menggunakan pengaturan berbeda hingga menemukan visualisasi optimal untuk kebutuhan analisis Anda.
{% endhint %}

### Alur Kerja Penyempurnaan Iteratif

**Alur kerja optimasi LUT yang umum:**

1.**Pilih lapisan indeks** (misalnya, RAW (Reflektansi))
2. **Terapkan indeks** - Pilih filter kamera dan rumus indeks, seret lingkaran berwarna ke lokasi yang sesuai dalam rumus indeks
3. **Terapkan gradien LUT** - Mulai dengan preset Red-Yellow-Green
4. **Periksa nilai piksel** - Gerakkan kursor ke berbagai arah, catat rentang nilai
5. **Sesuaikan min/max** - Perkecil rentang untuk fokus pada vegetasi (misalnya, 0,2 hingga 0,9)
6. **Pilih pemotongan** - Coba &quot;Original Background&quot; untuk konteks
7. **Perbaiki warna** - Sesuaikan gradien jika diperlukan untuk penekanan tertentu
8. **Finalisasi pengaturan**- Catat pengaturan dan salin ke Pengaturan Proyek untuk pemrosesan ekspor

### Pemeriksaan Nilai Piksel

Memahami nilai piksel aktual sangat penting untuk menetapkan rentang LUT yang efektif:**Cara memeriksa nilai:**

1. Nilai piksel ditampilkan saat gambar memiliki kotak**Index**, atau kotak**Index**dan**LUT** yang dicentang.
2. **Gerakkan kursor** ke area berbeda pada gambar
3. **Perhatikan nilai piksel** yang ditampilkan di legenda saat Anda mengarahkan kursor
4. Perbesar untuk melihat piksel individu yang disorot dengan nilai mengambang
5. **Catat** rentang nilai untuk fitur yang berbeda:
   * **Vegetasi sehat**: misalnya, NDVI 0,55–0,85
   * **Vegetasi tertekan**: misalnya, NDVI 0,30–0,50
   * **Tanah telanjang**: mis., NDVI 0,05–0,25
   * **Air** (jika ada): mis., NDVI -0,05 hingga 0,10**Menggunakan nilai piksel untuk mengatur rentang LUT:**Setelah memeriksa nilai piksel, sesuaikan nilai min/max LUT Anda:**Contoh skenario:*** **Pengamatan**: Nilai tanah = 0,05–0,25, Tertekan = 0,25–0,50, Sehat = 0,50–0,85
* **Tujuan**: Menampilkan hanya kesehatan tanaman (mengecualikan tanah)
* **Pengaturan LUT**: Min = `0.25`, Max = `0.85`
* **Clipping**: &quot;Original Background&quot; untuk melihat tanah dalam warna alami
* **Hasil**: Gradien warna hanya berlaku pada vegetasi, tanah ditampilkan sebagai gambar asli

{% hint style="info" %}
**Rentang Dinamis**: Tanaman yang berbeda, musim, dan tahap pertumbuhan akan memiliki rentang nilai yang berbeda. Selalu periksa nilai piksel pada dataset spesifik Anda sebelum menetapkan rentang LUT.
{% endhint %}

***

## Indeks Kustom (Chloros+)

### Membuat Rumus Indeks Kustom

{% hint style="info" %}
**Tempat Pembuatan**: Indeks kustom dapat dikonfigurasi di**Pengaturan Proyek** sebelum pemrosesan, serta di bilah samping sandbox Penampil Gambar.
{% endhint %}

**Untuk membuat indeks kustom:**

1.**Buka Pengaturan Proyek** (sebelum pemrosesan) atau bilah samping sandbox Penampil Gambar
2. Arahkan ke **menu tarik-turun rumus indeks**

3. Cari opsi**&quot;Kustom&quot;** (harus masuk dengan lisensi Chloros+)
4. **Tentukan rumus Anda** menggunakan variabel band:
   * Nama band: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, dll.
   * Operator: `+`, `-`, `*`, `/`, `^` (eksponen)
   * Fungsi: `sqrt()`, `abs()`, dll. (jika didukung)
   * Tanda kurung: `()` untuk urutan operasi
5. **Beri nama indeks Anda** (misalnya, &quot;MyIndex&quot; atau &quot;CustomNDVI&quot;)
6. **Simpan konfigurasi**

**Contoh rumus kustom:**

```

Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style="warning" %}
**Validasi Rumus**: Pastikan rumus Anda menggunakan band yang tersedia di kamera Anda. Misalnya, RedEdge hanya tersedia pada kamera dengan filter RedEdge.
{% endhint %}

***

## Langkah Selanjutnya

Sekarang setelah Anda memahami Sandbox Indeks/LUT:

* **Terapkan pada pemrosesan**: Gunakan pengaturan yang ditemukan di [Pengaturan Proyek](../project-settings/project-settings.md)
* **Pemrosesan batch**: Terapkan indeks yang dioptimalkan ke dataset lengkap
* **Pelajari lebih lanjut**: Baca [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md)

Dokumentasi terkait:

* [**Lapisan Gambar**](image-layers.md) - Pengelolaan dan visualisasi lapisan
* [**Membuka Gambar dalam Layar Penuh**](opening-an-image-full-screen.md) - Dasar-dasar Penampil Gambar
* [**Pemrosesan Gambar (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Alur kerja pemrosesan lengkap

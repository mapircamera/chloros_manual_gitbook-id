# Index/LUT Sandbox

Index/LUT Sandbox adalah ruang kerja interaktif di bilah samping Chloros Image Viewer. Anda memilih rumus, menghubungkan saluran kamera Anda ke rumus tersebut, mewarnainya dengan gradien, dan menyesuaikan rentang nilainya — dan gambar akan diperbarui secara langsung saat Anda melakukannya. Sejak versi 1.2.0, Anda juga dapat **menyimpan hasil karya Anda**, baik untuk satu gambar maupun untuk seluruh proyek, tanpa perlu memproses ulang.

## Kegunaan Sandbox

| Sandbox Indeks/LUT (interaktif)        | Pemrosesan Proyek (batch)       |
| -------------------------------------- | -------------------------------- |
| Satu gambar sekaligus, umpan balik instan | Seluruh kumpulan data dalam satu proses     |
| Eksperimental dan berulang             | Pengaturan yang telah dikonfigurasi sebelumnya          |
| Dirender secara langsung; hanya disimpan saat Anda meminta  | Selalu menulis file hasil      |
| Sempurna untuk menemukan pengaturan yang tepat | Paling baik setelah pengaturan final |

{% hint style="success" %}
**Alur kerja biasa**: sesuaikan di Sandbox hingga visualisasinya sesuai keinginan Anda, lalu ekspor langsung dari Sandbox, atau salin pengaturan indeks dan LUT yang sama ke [Pengaturan Proyek](../project-settings/project-settings.md) agar proses pengolahan berikutnya menerapkan pengaturan tersebut ke setiap gambar.
{% endhint %}

***

## Membuka Sandbox

1. Klik sebuah gambar di kisi — gambar tersebut akan terbuka dalam layar penuh di tab **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Klik ikon **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> untuk menampilkan bilah sisi kiri jika belum terbuka
3. Pilih lapisan multi-band dari menu tarik-turun lapisan di kanan atas — **RAW (Reflectance)** biasanya menjadi pilihan, karena nilai indeks yang dihitung berdasarkan reflektansi yang telah dikalibrasi dapat dibandingkan antar gambar

Bilah samping menampilkan, dari atas ke bawah:

* nama gambar dan model kameranya
* tombol **Ekspor/Simpan Gambar**— muncul setelah kotak centang**Indeks**atau**LUT** dicentang
* kotak centang **Indeks**dan**LUT**
* panel konfigurasi indeks
* panel **Nilai Kursor** dengan pembacaan, histogram, dan kontrol GSD

{% hint style="warning" %}
**Tidak tersedia untuk kamera mono.** Pada gambar LATTICE M3M pita tunggal, kedua kotak centang tersebut dinonaktifkan, dengan keterangan _"Tidak tersedia untuk sensor mono (M3M)"_ — indeks multi-pita tidak terdefinisi pada satu pita. Untuk menghitung indeks dari kamera M3M, gabungkan dua atau lebih menjadi tumpukan multi-pita yang disejajarkan dan gunakan mesin indeks LATTICE.
{% endhint %}

***

## Menerapkan indeks

1. Centang kotak **Indeks** di bagian atas bilah samping
2. Pilih filter kamera Anda dari menu tarik-turun sebelah kiri (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Pilih rumus indeks dari menu tarik-turun di sebelah kanan — 27 rumus bawaan, ditambah rumus kustom apa pun yang telah Anda simpan
4. Rumus tersebut ditampilkan sebagai persamaan matematika di bawah ini, dengan lingkaran kosong di setiap slot pita. **Seret lingkaran saluran berwarna ke dalam slot** untuk mengikatnya
5. Setelah setiap slot yang digunakan rumus tersebut terikat, gambar akan diperbarui dan menampilkan nilai indeks
6. Arahkan kursor ke atas gambar untuk membaca nilai; panel **Nilai Kursor** akan menambahkan baris indeks dengan nilai di bawah kursor

Klik ganda slot yang telah diikat untuk menghapusnya. Rumus yang belum lengkap adalah keadaan normal saat sedang menyeret, bukan kesalahan — gambar hanya tidak akan diperbarui hingga rumus selesai.

Lingkaran saluran diberi kode warna: merah = Red, hijau = Green, biru = Blue, oranye = Orange, sian = Cyan, ungu = NIR, magenta = RE. Warna yang sama digunakan untuk titik-titik saluran dan kurva histogram di panel Nilai Kursor.

### Contoh NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Untuk referensi rumus lengkap — ketiga daftar preset dan nama mana yang berfungsi di mana — lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md).

### Dengan opsi Indeks dicentang tetapi tanpa LUT

Gambar ditampilkan dalam **skala abu-abu**, diregangkan di antara dua nilai ambang batas. Hal ini disengaja: gambar indeks merupakan data skalar, dan skala abu-abu adalah representasi yang sebenarnya dari data tersebut. Tambahkan LUT jika Anda menginginkan warna.

***

## Bekerja dengan LUT (Tabel Pencarian)

Sebuah **Tabel Pencarian** memetakan nilai indeks ke warna: masukan NDVI 0,65, keluaran warna hijau tertentu. Tabel ini tidak mengubah data — melainkan mengubah cara Anda membacanya.

### Menambahkan LUT

1. Klik tombol **"+ Add LUT"** di bawah rumus <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">
2. Pilih gradien warna
3. Atur batas minimum dan maksimum pemotongan
4. Pilih Mode Pemotongan
5. Centang kotak **LUT** di bilah samping untuk menampilkannya

Kotak centang LUT akan tetap dinonaktifkan hingga LUT benar-benar telah dikonfigurasi pada indeks.

### Memilih gradien warna

Arahkan kursor ke **bilah gradien**untuk membuka daftar preset — Chloros menyediakan**tujuh** preset gradien:

| # | Gradien                            | Bentuk                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Kuning → Green (**default**)  | Bercabang — sesuai dengan intuisi umum tentang vegetasi, hijau = sehat |
| 2 | Ungu → Kuning → Green             | Bercabang, dengan ujung bawah yang jelas                                  |
| 3 | Cokelat → Putih → Blue                | Berbeda di sekitar titik tengah yang terang                                   |
| 4 | Hitam → Ungu → Merah muda → Kuning pucat | Berurutan, dari gelap ke terang                                           |
| 5 | Red → Kuning → Blue                 | Bercabang di sekitar titik tengah yang terang                                   |
| 6 | Ungu → Blue → Green → Kuning      | Berurutan, dari gelap ke terang                                           |
| 7 | Orange → Putih → Ungu             | Bercabang di sekitar titik tengah terang                                   |

Gradien **bercabang**menempatkan warna netral di tengah jendela Anda, yang mudah dibaca ketika titik tengah memiliki arti tertentu (ambang batas, tanggal dasar). Gradien**berurutan** berjalan secara monoton dari gelap ke terang, yang mudah dibaca untuk kuantitas yang hanya memiliki "lebih" dan "kurang".

Setiap preset memiliki tujuh titik warna. Klik sebuah preset dan gambar akan segera diperbarui (jika kotak LUT dicentang).

### Mengedit titik warna

Di bawah bilah gradien terdapat deretan contoh warna, satu per titik:

* **Mengubah warna**: klik swatch untuk membuka pemilih warna (roda warna, penggeser RGB/HSV, atau kode heksadesimal seperti `#FF0000`)
* **Menambahkan titik**: klik tombol**+** di ujung baris — titik berwarna putih akan ditambahkan
* **Menghapus titik**:**klik dua kali** swatch tersebut
* **Simpan gradien yang telah diedit**: klik ikon simpan di sebelah bilah gradien untuk menambahkan gradien yang telah diedit ke daftar preset sehingga Anda dapat memilihnya lagi

Gradien yang telah Anda konfigurasikan pada suatu indeks disimpan bersama indeks tersebut di pengaturan proyek, sehingga tetap ada meskipun proyek ditutup dan dibuka kembali.

**Jumlah titik yang lebih sedikit**menghasilkan zona-zona yang jelas yang terlihat seperti klasifikasi;**jumlah titik yang lebih banyak** menghasilkan transisi yang halus dan hampir seperti foto. Tiga hingga lima titik cocok untuk slide presentasi dan peta klasifikasi; enam hingga sepuluh cocok untuk analisis umum; lima belas atau lebih cocok untuk pemeriksaan terperinci dan gambar publikasi.

### Menentukan rentang nilai

Kontrol ambang batas berupa **penggeser dengan dua pegangan**yang berkisar dari −1 hingga +1, dilengkapi kotak teks yang dapat diedit di setiap ujungnya untuk memasukkan nilai tepat, serta tombol**AUTO**.

* Seret salah satu pegangan, atau ketik angka ke dalam kotaknya dan tekan Enter
* **AUTO**mengatur rentang ke**persentil ke-2 dan ke-98** dari nilai indeks yang valid pada gambar — titik awal yang baik yang mengabaikan nilai yang menyimpang. Chloros membulatkan hasil secara adaptif, hingga 4 tempat desimal untuk rentang yang sangat sempit, 3 untuk rentang yang sempit, dan 2 untuk yang lainnya
* Penyesuaian manual apa pun akan didahulukan daripada AUTO sampai Anda menekan AUTO lagi

Contoh jendela NDVI:

| Tujuan                                    | Min  | Maks |
| --------------------------------------- | ---- | --- |
| Tampilkan semuanya                         | −1,0 | 1,0 |
| Hanya vegetasi, kecualikan tanah dan air | 0,2  | 0,9 |
| Hanya vegetasi sehat                 | 0.5  | 0.9 |
| Tonjolkan stres                        | 0.2  | 0.5 |

Memperkecil jendela akan meningkatkan kontras di dalam area yang Anda minati dan mendorong semua elemen lain ke luar rentang — di mana **Mode Pemotongan** menentukan apa yang terjadi pada elemen tersebut.

***

## Mode Pemotongan

Ketika nilai indeks piksel berada di luar jendela min/maks, Mode Pemotongan menentukan bagaimana piksel tersebut digambar.

| Label menu tarik-turun                  | Nilai yang disimpan      | Piksel di luar rentang digambar sebagai                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum & Maksimum** (default) | `clip`            | Warna ujung terdekat dari gradien — nilai di bawah minimum akan mengambil warna pertama, nilai di atas maksimum akan mengambil warna terakhir |
| **Latar Belakang Transparan**      | `transparent`     | Sepenuhnya transparan (alpha sebenarnya)                                                                                                  |
| **Latar Belakang Indeks**| `indexColor`      | Skala abu-abu, direntangkan melintasi rentang indeks**penuh** gambar, sehingga struktur di luar rentang tetap terlihat dalam warna abu-abu                |
| **Latar Belakang Asli**         | `backgroundColor` | Gambar dasarnya sendiri, sehingga lapisan warna berada di atas pemandangan asli                                                |

| Mode                       | Cocok untuk                               | Tampilan                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimum & Maksimum**      | Tampilan data lengkap, analisis ilmiah | Setiap piksel diberi warna                      |
| **Latar Belakang Transparan** | Lapisan GIS, mengisolasi rentang nilai   | Warna di dalam jendela, kosong di luar |
| **Latar Belakang Indeks**       | Penekanan sambil mempertahankan konteks data    | Berwarna di dalam, abu-abu di luar               |
| **Latar Belakang Asli**    | Laporan dan presentasi              | Berwarna di dalam, foto di luar         |

{% hint style="info" %}
**Piksel tanpa data selalu transparan, di semua mode.** Piksel yang indeksnya tidak terbatas (pembagian 0/0) atau tepat −1,0 atau +1,0 (penanda saturasi, di mana satu pita bernilai nol sementara yang lain tidak) diperlakukan sebagai tidak ada data, bukan sebagai nilai ekstrem. Hal ini mencegah sorotan yang terbakar dan bayangan mati masuk ke dalam skala warna Anda, alih-alih menampilkannya sebagai pembacaan paling ekstrem dalam bingkai. Aturan yang sama menentukan piksel mana yang menjadi masukan untuk ambang batas AUTO dan histogram indeks, sehingga ketiganya selaras.
{% endhint %}

Transparansi tetap terjaga saat ekspor disimpan dalam format PNG. Hal ini tidak dapat direpresentasikan dalam format JPG.

***

## Membaca nilai saat Anda menyesuaikan

Panel **Nilai Kursor** di bawah panel konfigurasi berfungsi sebagai alat ukur untuk Sandbox:

* Arahkan kursor ke atas gambar dan baca nilai sumber per kanal, serta nilai indeks pada barisnya sendiri
* Aktifkan tombol **INDEX** di atas histogram untuk melihat distribusi nilai indeks dalam bingkai, dengan dua ambang batas klip Anda digambarkan sebagai garis putus-putus oranye dan nilai kursor sebagai garis putih — ini adalah cara tercepat untuk memilih jendela yang benar-benar berisi data Anda
* Aktifkan **CURSOR** untuk melihat garis penanda pada nilai-nilai di bawah penunjuk
* Perbesar hingga melebihi 60× (lebih kecil jika ukuran blok GSD telah ditetapkan) untuk menyorot piksel-piksel yang ditampilkan secara individual dengan nilai mengambang

Rutinitas praktis:

1. Catat nilai-nilai di atas vegetasi sehat, vegetasi yang tertekan, tanah gundul, dan air
2. Perhatikan di mana kelompok-kelompok tersebut berada pada histogram indeks
3. Atur nilai min/maks untuk membatasi kelompok yang Anda minati
4. Pilih mode pemotongan — _Original Background_ mempertahankan pemandangan di sekitarnya tetap terlihat

***

## Mengekspor dari Sandbox

Semua yang ditampilkan di atas hanyalah pratinjau langsung sampai Anda menyimpannya. Tombol **Ekspor/Simpan Gambar** di bagian atas bilah samping akan membuka panel yang meluncur di atas bilah samping (bukan menutupi gambar, sehingga Anda masih dapat melihat apa yang sedang Anda tentukan).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opsi

| Opsi                          | Efek                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Terapkan ke gambar saat ini**      | Menyimpan gambar yang ditampilkan persis seperti aslinya, dengan pengaturan ini                                                                                                |
| **Terapkan ke semua gambar proyek** | Menjalankan kembali konfigurasi yang sama pada setiap gambar dalam proyek. Gambar yang tidak memiliki pita yang dibutuhkan indeks ini akan dilewati, bukan dianggap gagal |
| **Bilah gradien indeks/LUT**      | Juga menyimpan gambar legenda terpisah untuk setiap ekspor, dengan rentang nilai yang diberi label                                                                     |
| **Histogram indeks**             | Juga menyimpan gambar histogram terpisah untuk setiap ekspor, yang menunjukkan nilai minimum/maksimum data dan ambang batas pemotongan                                               |

Jika **ukuran blok GSD** pada tab gambar lebih besar dari 1, panel akan menampilkannya sebelum Anda mengonfirmasi: ekspor akan menyimpan apa yang Anda lihat, termasuk rata-rata blok. Atur kontrol GSD kembali ke 1 terlebih dahulu jika Anda ingin resolusi penuh.

### Lokasi penyimpanan berkas

Setiap klik tombol **Ekspor**akan mengalokasikan**folder baru yang tidak pernah digunakan ulang**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Contoh: `Sandbox_Exports/NDVI_LUT_001/`, lalu `Sandbox_Exports/NDVI_LUT_002/` untuk proses berikutnya. Penomoran ini diperoleh dengan memindai apa yang sudah ada di disk, sehingga tetap ada meskipun sistem di-restart atau folder dihapus secara manual. Tidak ada yang pernah ditimpa — inti dari Sandbox adalah membandingkan satu upaya dengan upaya sebelumnya.

Di dalam folder, per gambar:

| File                                                   | Isi                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | Gambar yang telah dirender, persis seperti yang ditampilkan oleh penampil, piksel demi piksel |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | File tambahan bilah gradien, jika diminta                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | File tambahan histogram indeks, jika diminta                  |

Kedua sidecar tersebut selalu ditulis pada **resolusi penuh**, bahkan ketika gambar utama dirata-ratakan per blok: ukuran blok sama dengan resolusi tampilan, dan kedua sidecar tersebut membaca nilai indeks sebenarnya per piksel. Keduanya juga mencetak lebih banyak informasi daripada versi yang ditampilkan di layar — keduanya mencantumkan jendela rentang _dan_ nilai minimum/maksimum data sebenarnya, sehingga legenda yang disimpan tetap dapat dibaca berbulan-bulan kemudian tanpa perlu membuka proyek.

### Kemajuan dan hasil

Ekspor seluruh proyek hanya memakan waktu beberapa menit, sehingga proses ini melaporkan kemajuannya melalui saluran kemajuan langsung alih-alih memblokir:

* Sebuah bilah kemajuan menampilkan `current / total` dan file yang sedang ditulis
* Saat selesai, panel tersebut melaporkan berapa banyak gambar yang diekspor, berapa banyak yang dilewati, dan jalur folder keluaran
* Gambar yang dilewati dicantumkan beserta alasannya (ditampilkan hingga lima, lalu diikuti baris "+N lagi"). Alasan umumnya adalah lapisan yang tidak memiliki saluran yang dibutuhkan oleh indeks ini
* Jika **tidak ada** gambar dalam proyek yang dapat menggunakan indeks tersebut, proses akan melaporkan kegagalan alih-alih meninggalkan folder kosong

Hanya satu proses ekspor sandbox yang dapat berjalan pada satu waktu. Memulai proses kedua saat proses pertama masih berjalan akan ditolak dengan pesan yang jelas, alih-alih membiarkan dua proses berebut file proyek yang sama.

### Kisi akan menampilkan proses tersebut

Setiap proses yang telah selesai akan muncul sebagai tombol tersendiri di bilah alat [kisi gambar](image-grid.md), berlabel `<IndexName> <Index|LUT> <NNN>`. Begitulah cara Anda membandingkan proses: lakukan ekspor dua kali dengan gradien atau ambang batas yang berbeda, lalu beralih di antara kedua tombol pada grid.

***

## Rumus indeks kustom (Chloros+)

{% hint style="info" %}
**Tempat membuatnya**: di bilah samping Sandbox, atau di**Pengaturan Proyek** sebelum pemrosesan. Keduanya akan ditulis ke daftar tingkat proyek yang sama.
{% endhint %}

1. Buka kalkulator rumus khusus dari menu tarik-turun rumus indeks (memerlukan masuk dengan langganan Chloros+ yang memenuhi syarat)
2. Tulis rumus menggunakan **simbol slot rentang** `x`, `y`, `z`, `a`, `b`, `c` — bukan nama band
3. Operator yang tersedia: `+`, `-`, `*`, `/`, `^`, dan `()` untuk pengelompokan
4. Fungsi yang tersedia: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Beri nama dan simpan — rumus tersebut akan muncul di bagian bawah menu tarik-turun rumus, dan Anda dapat mengikat slot-slotnya dengan menyeret lingkaran saluran, persis seperti preset bawaan

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Rumus kustom hanya tersedia di antarmuka pengguna (GUI).** Opsi CLI/SDK `--indices` memperluas 22 nama preset bawaan dan secara otomatis melewati yang lainnya, termasuk rumus kustom Anda. Untuk memproses rumus kustom secara massal, konfigurasikan di Pengaturan Proyek dan jalankan pemrosesan, atau gunakan ekspor "Terapkan ke semua gambar proyek" di Sandbox.
{% endhint %}

***

## Pemecahan Masalah

### "Lapisan ini tidak memiliki saluran yang dibutuhkan oleh indeks ini"

Rumus tersebut membaca posisi saluran yang tidak dimiliki oleh lapisan saat ini — misalnya, indeks tiga slot pada file satu atau dua saluran. Beralihlah ke lapisan multi-band (reflektansi atau debayered), atau pilih indeks yang sesuai dengan filter kamera Anda.

### "Tidak dapat menghubungi backend pemrosesan gambar"

Backend tidak merespons. Periksa tab Log; jika backend sedang dimulai ulang, Sandbox akan pulih dengan sendirinya begitu backend kembali aktif.

### Gambar tidak berubah saat saya menyeret lingkaran

Rumus tersebut belum lengkap. Rumus yang belum lengkap diperlakukan sebagai keadaan normal saat sedang diseret — tidak ada yang dirender dan tidak ada yang dilaporkan sebagai kesalahan. Isi setiap slot yang digunakan rumus.

### Seluruh gambar berwarna sama

Jendela klip Anda mungkin berada jauh di luar rentang data. Tekan **AUTO**untuk menyesuaikannya ke persentil ke-2/ke-98, atau aktifkan histogram**INDEX** untuk melihat di mana data sebenarnya berada.

### Warna yang diekspor tidak sesuai dengan yang saya lihat

Seharusnya sama — jalur ekspor dirancang secara sengaja untuk mencerminkan pratinjau langsung, termasuk alpha mode pemotongan, dan rata-rata blok diterapkan _setelah_ pewarnaan, persis seperti yang dilakukan oleh penampil. Jika ada perbedaan, pastikan ukuran blok GSD tidak berubah antara saat melihat dan mengekspor.

***

## Langkah Selanjutnya

* [**Lapisan Gambar**](image-layers.md) — lapisan mana yang akan digunakan untuk indeks, dan apa arti nilainya
* [**Membuka Gambar dalam Layar Penuh**](opening-an-image-full-screen.md) — pembacaan kursor, histogram, dan pengaturan GSD secara terperinci
* [**Rumus Indeks Multispektral**](../project-settings/multispectral-index-formulas.md) — setiap preset, pada setiap permukaan
* [**Pengaturan Proyek**](../project-settings/project-settings.md) — menyimpan pengaturan yang telah Anda tentukan ke dalam proses pemrosesan

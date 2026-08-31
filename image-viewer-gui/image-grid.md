# Kisi Gambar

Setelah mengimpor gambar ke dalam proyek, Anda akan melihat gambar-gambar tersebut tersusun dalam bentuk kisi di area utama. Kisi ini adalah tempat Anda memilih **versi mana dari setiap gambar yang ingin Anda lihat** — tombol-tombol di atasnya akan mengganti semua thumbnail sekaligus antara file sumber dan setiap hasil pengolahan.

## Ukuran Gambar Mini

Gunakan bilah geser zoom di pojok kanan atas untuk menyesuaikan ukuran gambar mini. Bilah geser ini berkisar dari **64 px hingga 1200 px**.

* **Ctrl + roda mouse** juga dapat mengubah skala gambar mini.
* **Ctrl + `+`**/**Ctrl + `=`**dan**Ctrl + `−`** mengubah ukuran secara bertahap sebesar 4 px setiap kali ditekan. Rentang ukuran yang dapat diatur melalui keyboard berhenti pada 64 px di ujung terkecil dan, di ujung terbesar, pada ukuran apa pun yang pas untuk menampung tepat dua thumbnail per baris di jendela saat ini.
* Ukuran yang Anda tentukan akan disimpan bersama proyek (`UI → Grid thumbnail size` di `project.json`, default `160`), sehingga membuka kembali proyek akan mengembalikannya.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>*Resolusi* thumbnail adalah pengaturan yang terpisah dari *ukuran* thumbnail: lihat **Tampilan → Resolusi Thumbnail Gambar** di [Pengaturan Proyek](../project-settings/project-settings.md) (default 512 px pada sisi terpanjang). Ukuran menunjukkan seberapa besar ubin digambar; resolusi menunjukkan seberapa banyak detail yang diambil untuk mengisinya.***

## Bilah alat kisi

Baris tombol di atas kisi memiliki hingga tiga kelompok, dari kiri ke kanan:

1. **Per Pemicu / Per Kamera** — mode pengelompokan. Hanya muncul untuk proyek yang berisi hasil pengambilan gambar LATTICE.
2. **Tombol filter kamera** — satu per kamera LATTICE. Hanya muncul dalam mode Per Kamera.
3. **Tombol mode ekspor/tampilan** — produk mana yang ditampilkan oleh setiap thumbnail.

Jika jendela terlalu sempit untuk menampung semuanya, kelompok-kelompok tersebut akan dilipat dari kanan ke kiri menjadi menu tarik-turun saat kursor diarahkan: tombol ekspor/tampilan dilipat terlebih dahulu, diikuti oleh tombol kamera. Kelompok yang dilipat akan meninggalkan satu tombol yang diberi label sesuai pilihan yang sedang aktif, dan mengarahkan kursor ke tombol tersebut akan menampilkan seluruh set tombol. **Per Trigger / Per Kamera tidak pernah dilipat.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

*****

## Tombol Ekspor dan Tampilan

Tombol-tombol ini beralih antara jenis gambar pada kisi thumbnail. **Sebuah tombol muncul segera setelah produk yang disebutkannya ada** — yang untuk berkas sumber berarti segera saat impor, bukan setelah pemrosesan. Chloros memindai ulang produk-produk proyek saat proses sedang berjalan, sehingga tombol-tombol tersebut muncul selama pemrosesan saat setiap produk mulai disimpan ke disk.

### Tombol dasar

Tombol ekspor paling kiri diberi label sesuai dengan **apa yang sebenarnya Anda impor**:

| Apa yang Anda impor | Label tombol |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Rekaman LATTICE dengan pratinjau tampilan di samping bingkai RAW | `PNG` atau `TIFF`, tergantung pratinjau mana yang ditampilkan |
| Pengambilan gambar LATTICE di mana file dasarnya **adalah** bingkai mentah | *tidak ada tombol* — `RAW (Original)` sudah menampilkan file tersebut |

Dalam proyek campuran, label mengikuti ekstensi yang paling banyak digunakan oleh gambar-gambar tersebut.

### Tombol produk

| Tombol | Menampilkan | Kapan muncul |
| --- | --- | --- |
| **Target** | Gambar dengan target kalibrasi yang terdeteksi | Setelah proses yang mendeteksi target |
| **Reflektansi** | Gambar reflektansi yang telah dikalibrasi | Hanya proyek Survey3 — proyek LATTICE menggunakan `RAW (Reflectance)` sebagai gantinya, sehingga kisi tidak pernah menampilkan dua tombol reflektansi |
| **Keseimbangan Putih** | Produk dengan keseimbangan putih (kamera RGB) | Setelah pemrosesan |
| **Koreksi Vignette** | Opsi cadangan yang dikoreksi vignette tanpa kalibrasi | Setelah proses di mana kalibrasi reflektansi tidak dapat diterapkan dan *Koreksi Vignette* diaktifkan |
| **Respon Sensor** | Fallback respon sensor yang belum dikalibrasi | Sama, tetapi dengan *koreksi vignette* dinonaktifkan |
| **`RAW (<INDEX> Index)`** | Satu tombol per indeks yang dihitung | Setelah proses dengan indeks yang dikonfigurasi |
| **`<INDEX> LUT`** | Satu tombol per indeks yang dipetakan warna | Setelah proses dengan LUT yang dikonfigurasi |
| **`<Index> <Index\|LUT> <NNN>`** | Satu tombol per proses ekspor [Index/LUT Sandbox](index-lut-sandbox.md) | Saat proses ekspor sandbox selesai |

### Tombol tingkat LATTICE

Proyek yang berisi tangkapan LATTICE menambahkan tombol-tombol ini, yang diberi label dengan nama tingkat alih-alih nama produk:

| Tombol | Tingkat |
| --- | --- |
| **RAW (Asli)** | Bingkai RAW sumber, sebagaimana diimpor |
| **RAW (Radiance)** | Radian spektral Float32, W/m²/sr/nm |
| **RAW (Reflectance)** | Reflektansi uint16, 32768 = ρ 1,0 |

`RAW (Original)` tersedia sejak saat impor — tidak memerlukan pemrosesan. Ketika impor LATTICE sama sekali tidak memiliki tombol dasar (file dasar setiap tangkapan adalah bingkai mentahnya), kisi akan berpindah sendiri ke tombol level pertama yang tersedia sehingga sorotan bilah alat sesuai dengan apa yang Anda lihat.

Ekspor Chloros dua tingkat **tidak memiliki tombol kisi tersendiri**:

* **Debayered** — tampilan `RAW (Original)` sudah menampilkan hasil debayering, sehingga tombol kedua pada gambar yang secara visual identik hanya akan menimbulkan gangguan. Produk `RAW (Debayered)` tetap ditulis ke disk dan masih dapat dipilih dari menu tarik-turun lapisan layar penuh.
* **Pratinjau** — pada kamera RGB, pratinjau terdaftar sebagai lapisan `White Balanced`, yang memang memiliki tombol. Pada kamera multispektral, tampilan pratinjau terdaftar sebagai `RAW (Preview)` dan dapat diakses dari menu tarik-turun lapisan layar penuh.

{% hint style="info" %}
Tombol-tombol tingkat ini hanya ditampilkan untuk proyek yang benar-benar mengandung bingkai LATTICE. Proyek-proyek Survey3 mendaftarkan beberapa nama lapisan internal yang sama, dan tombol-tombol tersebut disaring untuk proyek-proyek tersebut sehingga kisi Survey3 tetap mempertahankan set `JPG / Targets / Reflectance` yang sudah dikenal.
{% endhint %}

Mengklik thumbnail grid akan membuka [Image Viewer](opening-an-image-full-screen.md) layar penuh pada **produk yang sama dengan yang ditampilkan oleh grid tersebut** — jika kisi diatur ke `Targets`, thumbnail tersebut akan membuka gambar target yang diekspor.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Mengelompokkan proyek LATTICE: Per Pemicu vs Per Kamera

Pengambilan gambar array menghasilkan beberapa gambar dari momen yang sama dari modul kamera yang berbeda. Pengelompokan menentukan cara grid menumpuk gambar-gambar tersebut. Kedua mode menampilkan bilah header yang dapat dilipat dengan lebar penuh; **setiap grup awalnya dalam keadaan terbuka**, dan Chloros mengingat grup yang Anda tutup. Status pelipatan dilacak secara terpisah per mode, sehingga menutup grup dalam mode Per Kamera tidak akan menutup apa pun dalam mode Per Pemicu.

### Per Kamera (default)

Satu grup per modul kamera. Header menampilkan model kamera dan nomor seri (`LATT-M3M — <serial>`) serta jumlah foto. Ubin di dalam grup diurutkan secara kronologis berdasarkan peristiwa pengambilan gambar.

Dalam mode ini, bilah alat juga menampilkan satu **tombol filter kamera per kamera**, berlabel `MODEL (SERIAL)`. Semua kamera awalnya dalam keadaan terpilih; mengklik tombol tersebut akan membatalkan pemilihan kamera tersebut dan menghapus grupnya dari kisi. Ini adalah cara cepat untuk meninjau satu rentang spektrum di seluruh penerbangan.

### Per Pemicu

Satu grup per peristiwa pengambilan gambar — kumpulan bingkai yang diambil oleh semua modul pada pemicu yang sama. Header menampilkan waktu pengambilan gambar, jumlah kamera yang berkontribusi, dan lencana per model kamera dalam grup tersebut. Ubin di dalam sebuah grup diurutkan berdasarkan nomor seri kamera, sehingga pita yang sama berada di kolom yang sama untuk setiap pemicu.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Gambar non-LATTICE dalam proyek campuran tidak dikelompokkan — gambar tersebut ditampilkan sebagai ubin biasa setelah grup-grup tersebut.

***

## Gambar mini kisi mengikuti ukuran blok GSD

Jika Anda telah mengatur ukuran blok **GSD (px)** di bilah samping tab gambar, thumbnail kisi akan ditampilkan pada resolusi permukaan yang sama — tidak hanya pada tampilan layar penuh. Ukuran blok 8 berarti setiap piksel yang ditampilkan merupakan rata-rata dari blok 8 × 8 piksel sumber, di mana pun dalam aplikasi yang menampilkan gambar tersebut.

Karena lebar ubin awalnya hanya beberapa ratus piksel, ukuran blok yang kasar tidak lagi terlihat perbedaannya pada grid jauh sebelum terlihat perbedaannya pada tampilan layar penuh: bingkai 4000 px yang digambar ke dalam ubin 160 px sudah memiliki rasio sekitar 25 piksel sumber per piksel yang ditampilkan. Lihat [Membuka Gambar dalam Layar Penuh](opening-an-image-full-screen.md#gsd-block-size) untuk kontrolnya sendiri.

***

## Halaman terkait

* [**Membuka Gambar dalam Layar Penuh**](opening-an-image-full-screen.md) — penampil layar penuh, nilai kursor, dan histogram
* [**Lapisan Gambar**](image-layers.md) — menu tarik-turun lapisan di dalam penampil layar penuh
* [**Sandbox Indeks/LUT**](index-lut-sandbox.md) — membuat dan mengekspor visualisasi indeks
* [**Pengaturan Proyek**](../project-settings/project-settings.md) — tombol pengaturan ekspor yang menentukan produk mana saja yang tersedia

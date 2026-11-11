# Kamera Mono & Indeks Vegetasi

## Satu kamera = satu pita

Kamera **M3M**adalah versi mono dari kamera Bayer**M3C**: sensor monokrom IMX265 yang dilengkapi dengan satu filter interferensi pita sempit. Nama string model menunjukkan nama pita — `M3M-<lens>-F<wavelength>`, misalnya `M3M-L87-F685` (ditampilkan di Chloros sebagai `LATT-M3M-L87-F685`). Sensor ini menghasilkan **satu pita skala abu-abu** tanpa mosaik Bayer: tidak ada yang perlu didemosaik, tidak ada crosstalk antar-saluran yang perlu dipisahkan, dan tidak ada white balance yang perlu diatur.

Hal-hal yang perlu diketahui sebelum merencanakan sistem monokrom:

* **Radian dan reflektansi didefinisikan sepenuhnya per band.**Ini adalah peta radiometrik per-pita, sehingga satu kamera M3M menghasilkan radian float32 yang dikalibrasi (W/m²/sr/nm) dan reflektansi uint16 (`32768` = ρ 1,0) persis seperti yang dilakukan pita M3C. Bingkai mono membawa matriks respons sensor**identitas** — tidak diperlukan atau diterapkan pemisahan 3×3.
* **Satu kamera mono tidak dapat menghasilkan indeks vegetasi.** NDVI, NDRE, dan sejenisnya membutuhkan setidaknya dua band. Untuk menghitung indeks dari perangkat keras mono, Anda menggabungkan beberapa kamera M3M — lihat di bawah.
* Kamera M3M mengalirkan data **Mono12** (12-bit: 2 byte/piksel pada saluran, atau 1,5 byte/piksel saat sebuah array menggunakan [transmisi 12-bit terkompresi](arrays.md#packed-12-bit-transport)), yang berpengaruh pada [pengalokasian bandwidth array](arrays.md#bandwidth-how-many-cameras-fit).

## Apa yang dilewati Chloros untuk mode mono — dan bagaimana ia memberitahukannya

Tahapan pipa warna sama sekali tidak berlaku untuk sensor single-band. Chloros **melewati tahap-tahap tersebut dengan pesan satu baris** alih-alih menampilkan kesalahan, dan tetap menjalankannya secara normal untuk kamera M3C (Bayer) mana pun dalam sesi yang sama:

| Tahap | Perilaku Mono (M3M) | Perilaku M3C |
| --- | --- | --- |
| Demosaic / debayer | Dilewati — tingkat ekspor `debayered` adalah gambar skala abu-abu 1-saluran. | Demosaic 3-saluran. |
| Keseimbangan putih (`lattice white-balance`) | Dilewati dengan pesan satu baris. | Berjalan normal. |
| Profil warna (`lattice color-profile`) | Dilewati dengan pesan satu baris. | Berjalan normal. |
| Saturasi/kontras (`lattice color`) | Dilewati dengan pesan satu baris. | Berjalan normal. |
| Pemisahan crosstalk spektral | Identitas (tanpa matriks 3×3). | Matriks 3×3 per kamera diterapkan. |
| Radian / reflektansi | **Berjalan** — per-band, sepenuhnya dikalibrasi. | Berjalan per band. |

GUI menerapkan gating yang sama: untuk kamera mono, panel pengaturan per kamera menyembunyikan baris-baris khusus RGB (Keseimbangan Putih, Gamma, Profil Warna, Saturasi, Kontras, pemisahan saluran), dan histogram langsung dikunci pada satu jejak **MONO**. Pembeda di seluruh tumpukan adalah token `M3M` dalam string model, yang ditampilkan ke GUI/SDK sebagai `is_mono`.

## Indeks memerlukan ≥ 2 band: selaraskan → tumpuk → indeks

Alur kerja indeks mono selalu terdiri dari tiga langkah yang sama:

1. **Penjajaran** — arahkan beberapa kamera M3M pada panjang gelombang yang berbeda (misalnya F650 "Red" dan F850 "NIR"), hubungkan sebagai [susunan multi-kamera](arrays.md), dan biarkan Chloros menghitung transformasi co-registration antar kamera.
2. **Tumpuk** — bingkai yang telah diselaraskan menjadi satu gambar multi-pita (setiap kamera menyumbang satu pita yang diberi nama).
3. **Indeks** — mengevaluasi rumus indeks pada pita-pita tumpukan tersebut, dengan opsi untuk merendernya melalui LUT.

Di antarmuka pengguna (GUI), seluruh rangkaian ini merupakan mode tampilan susunan **Kamera Gabungan**: komposisi langsung sudah disejajarkan, dan Kalkulator Indeks susunan (di bawah) menentukan rumus yang digunakan untuk merendernya. Ekspor yang telah direkam dapat disesuaikan ke penyelarasan yang sama dengan opsi perekaman**Aligned**.

## Kalkulator Indeks

Kalkulator Indeks membuat ekspresi indeks yang digunakan oleh tampilan langsung dan ekspor indeks per-kamera. Ini adalah satu antarmuka bersama, yang dapat dibuka dari dua tempat di bilah samping tab Cameras:

* **Per-kamera**— Pratinjau Langsung → ikon roda gigi**Indeks** (hanya untuk kamera Bayer RGN/OCN/NGB; kamera mono tunggal tidak memiliki kontrol indeks karena satu band tidak dapat membentuk indeks).
* **Per-array**— pengaturan array → Pratinjau Langsung → ikon roda gigi**Indeks**. Ini adalah jalur mono: daftar pita mencakup**semua kamera anggota**, sehingga sepasang kamera mono menyumbangkan kedua pitanya di sini.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Kontrolnya, dari atas ke bawah:

* **Chip pita** ("Pita — klik untuk menambahkan ke ekspresi") — satu tombol per pita yang tersedia, diberi label nama warna + panjang gelombang dalam nm (nama warna yang sama dibedakan, misalnya "Warna 850"). Mengklik akan menyisipkan token pita di posisi kursor. Pita dari kamera yang tidak dapat menghasilkan radian per pita (RGB/FRGB) akan disaring.
* **Tombol operator dan fungsi** — `+ - * / ( ) ^ ,` ditambah `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Kotak teks ekspresi** — rumus yang dapat diketik bebas; placeholder menampilkan bentuk klasik NDVI, yaitu `(NIR - Red) / (NIR + Red)`. Pratinjau yang hanya dapat dibaca dan telah ditokenisasi di atasnya menampilkan chip band, angka, dan bendera untuk token yang tidak dikenal.
* **Banner validitas**— abu-abu "Kosong — tidak ada indeks yang akan diterapkan"; hijau "Ekspresi valid"; merah dengan kesalahan parsing spesifik (band tidak dikenal, band ambigu akibat penggunaan beberapa kamera, tanda kurung hilang, …); atau kuning saat ekspresi valid tetapi**konstanta** (misalnya `X/X`, atau penyebut NDVI yang diketik sebagai `−` alih-alih `+`) — konstanta memetakan seluruh bingkai ke satu warna.
* Peringatan kuning terpisah muncul jika ekspresi yang diterapkan tidak bermasalah tetapi **bingkai langsungnya seragam** (adegan datar atau jenuh) — keruntuhan histogram terdeteksi untuk Anda.
* **Terapkan LUT**(aktif secara default; nonaktif = peregangan skala abu-abu),**Level**2/3/5/7-stop (default 7-stop), dan input**Min / Max**yang mengapit bilah gradien. Nilai Min secara default adalah**0,2**— ini memperbesar rentang gradasi warna ke rentang yang relevan dengan vegetasi, sedangkan nilai di bawahnya akan ditampilkan sebagai skala abu-abu; atur Min ke −1 untuk rentang indeks penuh (tombol**Reset** mengembalikan nilai ke −1…+1). Nilai Max secara default adalah 1.
* **Histogram langsung** dari distribusi indeks — batang berskala akar kuadrat, garis persentil p2/p98 berwarna kuning, garis median putih, dan pembacaan ekor di luar rentang ("◀ N% < lo" / "hi < N% ▶") yang berubah menjadi kuning di atas 1% sebagai petunjuk untuk memperluas jendela Min/Max.
* **Terapkan** akan menerapkan ekspresi ke aliran langsung; penyesuaian LUT berlaku secara langsung tanpa perlu menekan Terapkan. Jika proyek terbuka, ekspresi dan pengaturan LUT akan disimpan bersama proyek dan dipulihkan saat Anda membukanya kembali; tanpa proyek, pengaturan tersebut hanya berlaku selama sesi tersebut.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Jalur CLI



Rantai align → stack → index yang sama, dapat diotomatisasi dari awal hingga akhir:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` menggunakan array terhubung yang menampung kameranya, atau membukanya sendiri untuk satu kali eksekusi. `index --live` selalu membuka kamera-kamera tersebut sendiri (sebuah [array sementara](../reference/cli-reference.md#multi-camera-transient-sessions)) dan tidak memiliki `--serials`: ia membuka setiap kamera di jaringan. Jadi, lepaskan setiap kamera yang disimpan oleh Chloros sebelum — GUI, `cam-connect`, `array-connect` (`chloros-cli lattice array-disconnect --all`): membuka kamera yang sedang ditahan untuk kedua kalinya dapat menghentikan aliran pemegangnya atau membuat backend crash. Kamera yang tidak mengirimkan frame apa pun akan menyebabkan proses gagal, dan tidak ada tumpukan atau indeks yang ditulis.

`--channel` memetakan simbol preset ke nama band pada tumpukan. Dua aturan berikut dapat mencegah kegagalan proses:

* **Simbol peka huruf besar/kecil** dan harus persis sama dengan nama saluran preset — preset menggunakan huruf kecil (preset NDVI adalah `red`,`nir`; lihat `--list-presets`). `--channel red=Red_660` berhasil; `--channel RED=660` gagal dengan kesalahan `channel_map missing entries`.
* Sisi band harus menyebutkan nama band dalam tumpukan yang telah disejajarkan (`lattice align-info --profile align.json` mencantumkannya). Mode offline juga menerima indeks band berbasis 0, misalnya `--channel red=0 --channel nir=1`.

`lattice index` juga dapat dijalankan secara offline terhadap berkas TIFF multi-band yang telah diselaraskan dan disimpan. Setiap formulir berjalan di backend, sehingga backend harus aktif dan terdaftar pada tingkatan Chloros+ berbayar; `--input` atau `--output` relatif akan mengacu pada folder tempat Anda menjalankan perintah:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Preset indeks

`lattice index --preset` (dan [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) di tab Gambar, yang menggunakan mesin yang sama) menyertakan **22 preset** berikut:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Jalankan `chloros-cli lattice index --list-presets` untuk melihat rumus dan simbol kanal setiap preset, serta `--list-gradients` untuk melihat gradien warna yang tersedia. Rumus kustom menggunakan `--formula EXPR` dengan sintaks yang sama seperti Kalkulator Indeks. Perhatikan bahwa daftar preset ini khusus untuk mesin indeks LATTICE — daftar yang muncul pada menu dropdown "Pengolahan" di Pengaturan Proyek untuk citra yang diimpor adalah daftar yang berbeda (lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md)).

Kumpulan bendera lengkap (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, kenop penyelarasan warp untuk `--live`, dan lainnya) didokumentasikan dalam [Referensi CLI § Matematika Indeks / Vegetasi](../reference/cli-reference.md#index--vegetation-maths); setara SDK terdapat di [Referensi SDK](../reference/sdk-reference.md).

## Mengambil produk indeks dari array mono

Dengan array yang terhubung dan ekspresi indeks yang diterapkan, `array-capture` (atau opsi GUI **Capture All**) akan menyimpan tingkat ekspor per kamera *dan* hasil render indeks — `--index`/`--no-index` mengaktifkan atau menonaktifkannya di CLI, dan pengambilan data secara default mencakup setiap tingkat yang berlaku. Kontribusi kamera mono ke setiap grup penangkapan adalah satu pita pada tingkat mentah/debayered (skala abu-abu)/radian/reflektansi, ditambah komposit indeks gabungan bersama saat array berjalan dalam mode gabungan. Lihat [Susunan Multi-Kamera § Pengambilan Gambar](arrays.md#capturing-monitoring-vs-analysis).

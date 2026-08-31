# Kamera Mono &amp; Indeks Vegetasi

## Satu kamera = satu pita

Kamera **M3M**adalah versi mono dari kamera Bayer**M3C**: sensor monokrom IMX265 yang dilengkapi dengan satu filter interferensi pita sempit. String model tersebut menamai pita spektralnya — `M3M-<lens>-F<wavelength>`, misalnya `M3M-L87-F685` (ditampilkan dalam Chloros sebagai `LATT-M3M-L87-F685`). Sensor ini menghasilkan**satu pita skala abu-abu** tanpa mosaik Bayer: tidak ada yang perlu didemosaik, tidak ada crosstalk antar-saluran yang perlu dipisahkan, dan tidak ada white balance yang perlu diatur.

Hal-hal yang perlu diketahui sebelum Anda merencanakan sistem monokrom:

* **Radian dan reflektansi didefinisikan sepenuhnya per pita.**Keduanya merupakan peta radiometrik per pita, sehingga satu kamera M3M menghasilkan radian float32 yang dikalibrasi (W/m²/sr/nm) dan reflektansi uint16 (`32768` = ρ 1,0) persis seperti yang dilakukan pita M3C. Frame mono membawa matriks respons sensor**identitas** — tidak diperlukan atau diterapkan pemisahan 3×3.
* **Satu kamera mono tidak dapat menghasilkan indeks vegetasi.** NDVI, NDRE, dan sejenisnya memerlukan setidaknya dua band. Untuk menghitung indeks dari perangkat mono, Anda menggabungkan beberapa kamera M3M — lihat di bawah.
* Kamera M3M mengalirkan data **Mono12** (12-bit, 2 byte/piksel pada kabel), yang penting untuk [penganggaran bandwidth array](arrays.md#bandwidth-the-rules-of-thumb).

## Apa yang dilewati oleh Chloros untuk mono — dan bagaimana cara memberitahu Anda

Tahapan pipa warna sama sekali tidak berlaku untuk sensor satu band. Chloros **melewati tahap-tahap tersebut dengan pesan satu baris** alih-alih menampilkan kesalahan, dan tetap menjalankannya secara normal untuk kamera M3C (Bayer) mana pun dalam sesi yang sama:

| Tahap | Perilaku Mono (M3M) | Perilaku M3C |
| --- | --- | --- |
| Demosaic / debayer | Dilewati — tingkat ekspor `debayered` adalah gambar skala abu-abu 1-saluran. | Demosaic 3-saluran. |
| Keseimbangan putih (`lattice white-balance`) | Dilewati dengan pesan satu baris. | Berjalan normal. |
| Profil warna (`lattice color-profile`) | Dilewati dengan pesan satu baris. | Berjalan normal. |
| Saturasi/kontras (`lattice color`) | Dilewati dengan pesan satu baris. | Berjalan normal. |
| Pemisahan crosstalk spektral | Identitas (tanpa matriks 3×3). | Matriks 3×3 per kamera diterapkan. |
| Radian / reflektansi | **Berjalan** — per-band, dikalibrasi sepenuhnya. | Berjalan per band. |

GUI menerapkan pengaturan gating yang sama: untuk kamera mono, panel pengaturan per-kamera menyembunyikan baris-baris khusus RGB (Keseimbangan Putih, Gamma, Profil Warna, Saturasi, Kontras, pemisahan saluran), dan histogram langsung dikunci pada satu jejak **MONO**. Pembeda di seluruh tumpukan adalah token `M3M` dalam string model, yang ditampilkan ke GUI/SDK sebagai `is_mono`.

## Indeks memerlukan ≥ 2 pita: selaraskan → tumpuk → indeks

Alur kerja indeks mono selalu terdiri dari tiga langkah yang sama:

1. **Penjajaran** — arahkan beberapa kamera M3M pada panjang gelombang yang berbeda (misalnya, F650 &quot;Red&quot; dan F850 &quot;NIR&quot;), hubungkan sebagai [susunan multi-kamera](arrays.md), dan biarkan Chloros menghitung transformasi registrasi bersama antar kamera.
2. **Stack** — bingkai-bingkai yang telah diselaraskan menjadi satu gambar multi-band (setiap kamera menyumbangkan satu band yang diberi nama).
3. **Index** — mengevaluasi rumus indeks pada band-band dalam stack, secara opsional merendernya melalui LUT.

Di antarmuka pengguna (GUI), seluruh rangkaian ini merupakan mode tampilan susunan **Combined Cameras**: komposit langsung sudah diselaraskan, dan Kalkulator Indeks susunan (di bawah) menentukan rumus yang ditampilkannya. Ekspor yang direkam dapat disesuaikan ke penyelarasan yang sama dengan opsi perekaman**Aligned**.

## Kalkulator Indeks

Kalkulator Indeks menyusun ekspresi indeks yang digunakan oleh tampilan langsung dan ekspor indeks per-kamera. Ini merupakan satu permukaan bersama, yang dapat dibuka dari dua tempat di bilah samping tab Cameras:

* **Per-kamera**— Pratinjau Langsung → ikon roda gigi**Indeks** (hanya untuk kamera Bayer RGN/OCN/NGB; kamera mono tunggal tidak memiliki kontrol indeks karena satu pita tidak dapat membentuk indeks).
* **Per-array**— pengaturan array → Pratinjau Langsung → ikon roda gigi**Indeks**. Ini adalah jalur mono: daftar pita mencakup**semua kamera anggota**, sehingga pasangan mono menyumbangkan kedua pitanya di sini.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Kontrolnya, dari atas ke bawah:

* **Chip pita** (&quot;Pita — klik untuk menambahkan ke ekspresi&quot;) — satu tombol per pita yang tersedia, berlabel nama warna + panjang gelombang dalam nm (nama warna yang sama dibedakan, misalnya &quot;Warna 850&quot;). Mengklik akan menyisipkan token pita di kursor. Pita dari kamera yang tidak dapat menghasilkan radian per pita (RGB/FRGB) akan disaring.
* **Tombol operator dan fungsi** — `+ - * / ( ) ^ ,` ditambah `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Area teks ekspresi** — rumus yang dapat diketik bebas; placeholder menampilkan bentuk klasik NDVI, yaitu `(NIR - Red) / (NIR + Red)`. Pratinjau yang hanya dapat dibaca dan telah ditokenisasi di atasnya menampilkan chip band, angka, dan bendera sebagai token yang tidak dikenal.
* **Banner validitas**— abu-abu &quot;Kosong — tidak ada indeks yang akan diterapkan&quot;; hijau &quot;Ekspresi valid&quot;; merah dengan kesalahan penguraian spesifik (pita tidak dikenal, pita ambigu yang terekam oleh beberapa kamera, tanda kurung hilang, …); atau kuning saat ekspresi valid tetapi**konstan** (misalnya `X/X`, atau penyebut NDVI yang diketik dengan `−` alih-alih `+`) — konstanta memetakan seluruh bingkai ke satu warna.
* Peringatan kuning terpisah akan muncul jika ekspresi yang diterapkan sudah benar tetapi **bingkai langsungnya seragam** (adegan datar atau jenuh) — keruntuhan histogram terdeteksi secara otomatis.
* **Terapkan LUT**(aktif secara default; nonaktif = peregangan skala abu-abu),**Level**2/3/5/7-stop (default 7-stop), dan masukan**Min / Max**yang mengapit bilah gradien. Nilai Min secara default adalah**0,2**— ini memperbesar rentang gradasi warna ke rentang yang relevan dengan vegetasi, sementara nilai di bawahnya diteruskan sebagai skala abu-abu; atur Min ke −1 untuk rentang indeks penuh (tombol**Reset** mengembalikan ke rentang −1…+1). Nilai Max secara default adalah 1.
* **Histogram langsung** dari distribusi indeks — batang berskala akar kuadrat, garis persentil p2/p98 berwarna kuning, garis median putih, dan pembacaan ekor di luar rentang (&quot;◀ N% &lt; lo&quot; / &quot;hi &lt; N% ▶&quot;) yang berubah menjadi kuning di atas 1% sebagai petunjuk untuk memperluas jendela Min/Max.
* **Terapkan**akan menerapkan ekspresi ke aliran langsung; penyesuaian LUT berlaku secara langsung tanpa perlu menekan Terapkan. Ekspresi sengaja dibuat**hanya untuk sesi** — ekspresi tersebut tidak disimpan antar sesi.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Jalur CLI

Rantai align → stack → index yang sama, dapat diprogram dari awal hingga akhir:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` memetakan simbol preset ke nama band di stack. Dua aturan berikut akan menyelamatkan Anda dari kegagalan eksekusi:

* **Simbol peka huruf besar-kecil** dan harus persis sama dengan nama saluran preset — preset menggunakan huruf kecil (NDVI adalah `red`,`nir`; periksa `--list-presets`). `--channel red=Red_660` berhasil; `--channel RED=660` gagal dengan kesalahan `channel_map missing entries`.
* Sisi band harus menyebutkan nama band dalam tumpukan yang telah disejajarkan (`lattice align-info --profile align.json` mencantumkannya). Mode offline juga menerima indeks band berbasis 0, misalnya `--channel red=0 --channel nir=1`.

`lattice index` juga berjalan sepenuhnya dalam mode offline terhadap file multi-band yang disimpan dan disejajarkan TIFF:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Preset indeks

`lattice index --preset` (dan [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) di tab Gambar, yang menggunakan mesin yang sama) menyertakan **22 preset** berikut:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Jalankan `chloros-cli lattice index --list-presets` untuk rumus dan simbol kanal setiap preset, serta `--list-gradients` untuk gradien warna yang tersedia. Rumus kustom menggunakan `--formula EXPR` dengan sintaks yang sama seperti Kalkulator Indeks. Perhatikan bahwa daftar preset ini khusus untuk mesin indeks LATTICE — daftar pada menu tarik-turun &quot;Pengolahan&quot; di Pengaturan Proyek untuk citra yang diimpor adalah daftar yang berbeda (lihat [Rumus Indeks Multispektral](../project-settings/multispectral-index-formulas.md)).

Kumpulan bendera lengkap (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, kenop penyelarasan dan distorsi untuk `--live`, dan lainnya) didokumentasikan dalam [Referensi CLI § Indeks / Perhitungan Vegetasi](../reference/cli-reference.md#index--vegetation-maths); setara dengan SDK terdapat di [Referensi SDK](../reference/sdk-reference.md).

## Menangkap produk indeks dari array mono

Dengan array yang terhubung dan ekspresi indeks yang diterapkan, `array-capture` (atau opsi **Capture All** di antarmuka pengguna) menyimpan tingkat ekspor per kamera *dan* hasil render indeks — `--index`/`--no-index` mengaktifkan atau menonaktifkannya pada CLI, dan penangkapan secara default mencakup setiap tingkat yang berlaku. Kontribusi kamera mono ke setiap grup perekaman adalah satu pita pada tingkat mentah/debayered (skala abu-abu)/radian/reflektansi, ditambah komposit indeks gabungan bersama saat susunan kamera berjalan dalam mode gabungan. Lihat [Susunan Multi-Kamera § Pengambilan Gambar](arrays.md#capturing-monitoring-vs-analysis).

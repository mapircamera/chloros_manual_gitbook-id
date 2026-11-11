# Memilih Gambar Target

Menandai gambar mana saja yang berisi target kalibrasi memberi tahu Chloros lokasi tepat di mana gambar-gambar tersebut harus dicari. Jika setidaknya satu gambar dicentang di kolom Target, Chloros hanya akan memindai **gambar-gambar yang dicentang** — sehingga menandai target tidak hanya mempercepat proses pemrosesan, tetapi juga mencegah citra survei disalahartikan sebagai target.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Mengapa Menandai Gambar Target?

### Penandaan Mengontrol Proses Pemindaian

Saat Anda mencentang kolom Target untuk gambar tertentu:

* Chloros hanya memindai gambar yang dicentang untuk mencari target
* Deteksi target selesai jauh lebih cepat
* Gambar survei tidak dapat menghasilkan deteksi target palsu

Jika **tidak ada** gambar yang dicentang, Chloros akan kembali memindai setiap gambar dalam proyek:

* Algoritma deteksi target dijalankan pada setiap gambar
* Ratusan atau ribuan gambar diperiksa tanpa perlu
* Pemrosesan memakan waktu jauh lebih lama, terutama untuk dataset besar

{% hint style="success" %}
**Peningkatan Kecepatan**: Menandai 2–3 gambar target dalam dataset berisi 500 gambar dapat mengurangi waktu deteksi target dari lebih dari 30 menit menjadi kurang dari 1 menit.
{% endhint %}

***

## Cara Menandai Gambar Target

### Langkah 1: Identifikasi Gambar Target Anda

Telusuri gambar yang telah diimpor di File Browser dan identifikasi gambar mana yang berisi target kalibrasi.

**Skenario umum:*** **Target pra-pengambilan**: Diambil sebelum memulai sesi
* **Target pasca-pengambilan**: Diambil setelah sesi selesai
* **Target di lapangan**: Target yang ditempatkan di dalam area pengambilan
* **Beberapa target**: 2–3 gambar target per sesi (disarankan)

### Langkah 2: Periksa Kolom Target <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Untuk setiap gambar yang berisi target kalibrasi:

1. Temukan gambar tersebut di tabel File Browser
2. Temukan kolom **Target** (kolom paling kanan)
3. Klik kotak centang di kolom Target untuk gambar tersebut
4. Ulangi untuk semua gambar yang berisi target

### Langkah 3: Verifikasi Pilihan Anda

Sebelum pemrosesan, periksa kembali:

* [ ] Semua gambar dengan target kalibrasi telah dicentang
* [ ] Tidak ada gambar non-target yang tercentang secara tidak sengaja
* [ ] Target terlihat jelas pada gambar yang telah dicentang

***

## LATTICE: Target Bersifat Opsional Saat DAQ Sedang Merekam

Untuk kamera multispektral LATTICE, target kalibrasi dalam bingkai merupakan **salah satu dari dua** referensi reflektansi yang mungkin:

* **Target dalam bingkai**: ketika gambar target yang ditandai lolos dari pemeriksaan kualitas (QA) Chloros, target tersebut menjadi**referensi reflektansi absolut** untuk citra di sekitarnya.
* **Sinar turun DAQ**: jika tidak ada target (atau gagal QA), Chloros menghitung reflektansi dari irradiansi sinar turun sensor cahaya DAQ sebagai gantinya (ρ = π·L/E). Jika perekaman `.daq` atau DAQ-M `.csv` mencakup tangkapan Anda, Anda akan mendapatkan reflektansi yang dikalibrasi**tanpa gambar target sama sekali**.

Perilaku otomatis ini adalah pengaturan default. Dalam CLI / SDK, hal ini sesuai dengan `--reflectance-source auto`; Anda juga dapat memaksa penggunaan `target` (ketat — tanpa penggantian oleh DAQ) atau `daq` (mengutamakan DAQ). Lihat [Referensi CLI](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometri target LATTICE**: selain deteksi panel klasik yang digunakan untuk Survey3, pemrosesan LATTICE mendukung**target bertanda ArUco**,**target ROI tetap**, dan**target strip**, yang dikonfigurasi per proyek. Pemindaian reflektansi target**yang diukur** per unit dapat disediakan berdasarkan nomor seri (CLI: `--target-reflectance-dir`, satu `<serial>.csv` per unit target), dengan spektrum T3/T4P nominal sebagai cadangan.

{% hint style="info" %}
**Modul F988**: Reflektansi F988 dikalibrasi menggunakan panel reflektansi di lokasi: pita tersebut berada di luar rentang kalibrasi sensor cahaya DAQ, sehingga Chloros menerapkan penangkapan panel terbaru Anda dan mempertahankannya di antara pengamatan panel. Jika modul F988 diproses hanya dengan DAQ, Chloros menolak reflektansi berbasis DAQ untuk pita tersebut (alasan penolakan `dls-uncalibrated-band-988`) — alur kerja panel adalah jalur yang didukung.
{% endhint %}

***

## Praktik Terbaik untuk Gambar Target

### Pedoman Pengambilan Gambar Target

**Waktu:**

* Ambil gambar target segera sebelum dan selama sesi pengambilan gambar Anda
* Dalam kondisi pencahayaan yang sama dengan sensor cahaya DAQ Anda
* Idealnya, ambil gambar target sesering mungkin untuk hasil terbaik. Jika tidak, data sensor cahaya akan digunakan untuk menyesuaikan kalibrasi seiring waktu.

**Posisi Kamera:**

* Pegang kamera di atas target sedemikian rupa sehingga target berada di tengah dan mengisi sekitar 40–60% bagian tengah gambar.
* Jaga agar kamera sejajar/nadir dengan permukaan target

**Pencahayaan:**

* Pencahayaan sekitar yang sama dengan sensor cahaya DAQ Anda
* Hindari bayangan pada permukaan target
* Jangan menghalangi sumber cahaya dengan tubuh, kendaraan, atau vegetasi
* Kondisi mendung memberikan hasil yang paling konsisten

**Kondisi Target:**

* Jaga panel target tetap bersih dan kering
* Semua panel target Anda (misalnya, keempat panel pada T4) harus terlihat jelas dan tidak terhalang
* Jika memungkinkan, posisikan target tegak lurus/nadir terhadap sumber cahaya

### Berapa Banyak Gambar Target?

**Minimum:**1 gambar target per sesi.**Disarankan:** 3–5 gambar target per sesi.**Jadwal praktik terbaik:**

* Ambil 3–5 gambar segera setelah sensor cahaya mulai merekam
* Putar posisi kamera di antara pengambilan gambar untuk hasil terbaik
* Opsional: secara berkala di tengah sesi jika kondisi pencahayaan terus berubah

***

## Bekerja dengan Beberapa Kamera

### Pengaturan Dua Kamera

Jika menggunakan dua kamera MAPIR secara bersamaan (misalnya, Survey3W RGN + Survey3N OCN):

1. Ambil gambar target dengan **kedua kamera** secara bersamaan
2. Gunakan **target fisik yang sama** untuk kedua kamera
3. Tandai gambar target untuk **kedua jenis kamera** di File Browser
4. Chloros akan menggunakan target yang sesuai untuk kalibrasi masing-masing kamera

### Kolom Model Kamera

Kolom **Model Kamera** membantu mengidentifikasi gambar mana yang berasal dari kamera mana:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* dll.

Gunakan kolom ini untuk memverifikasi bahwa Anda telah menandai target untuk setiap jenis kamera dalam proyek Anda.

***

## Pengaturan Deteksi Target

### Menyesuaikan Sensitivitas Deteksi

Jika Chloros tidak mendeteksi target Anda dengan benar, sesuaikan pengaturan berikut di [Pengaturan Proyek](adjusting-project-settings.md):**Luas sampel kalibrasi minimum (px):*** **Default**: 25 piksel
* **Tingkatkan** jika terjadi deteksi palsu pada artefak kecil
* **Kurangi** jika target tidak terdeteksi**Pengelompokan Target Minimum (0-100):*** **Default**: 60
* **Tingkatkan** jika target terpecah menjadi beberapa deteksi
* **Kurangi** jika target dengan variasi warna tidak terdeteksi sepenuhnya

{% hint style="info" %}
**Tips CLI**: `chloros-cli process` mendukung parameter yang sama (`--min-target-size`, `--target-clustering`), dan bendera `--target`/`--targets`-nya menandai seluruh folder masukan sebagai khusus panel target. Lihat [Referensi CLI](../reference/cli-reference.md).
{% endhint %}

***

## Masalah Umum pada Gambar Target

### Masalah: Tidak Ada Target yang Terdeteksi

**Penyebab yang mungkin:**

* Gambar target tidak ditandai di File Browser
* Target terlalu kecil dalam bingkai (< 30% dari gambar)
* Pencahayaan buruk (bayangan, silau)
* Pengaturan deteksi target terlalu ketat

**Solusi:**

1. Pastikan kolom Target dicentang untuk gambar yang benar
2. Periksa kualitas gambar target pada tampilan pratinjau
3. Ambil ulang gambar target jika kualitasnya buruk
4. Sesuaikan pengaturan deteksi target jika diperlukan

### Masalah: Deteksi Target Palsu

**Penyebab yang mungkin:**

* Bangunan, kendaraan, atau penutup tanah berwarna putih yang disalahartikan sebagai target
* Bintik-bintik terang pada vegetasi
* Sensitivitas deteksi terlalu rendah

**Solusi:**

1. Tandai hanya gambar target yang sebenarnya — hanya gambar yang dicentang yang akan dipindai
2. Tingkatkan luas sampel kalibrasi minimum
3. Tingkatkan nilai pengelompokan target minimum
4. Pastikan gambar target hanya menampilkan target (gangguan latar belakang minimal)

***

## Daftar Periksa Verifikasi

Sebelum memulai pemrosesan, verifikasi pemilihan gambar target Anda:

* [ ] Setidaknya 1 gambar target ditandai per sesi (atau, untuk LATTICE, rekaman `.daq`/`.csv` yang mencakup sesi tersebut)
* [ ] Kotak centang kolom target telah dicentang untuk semua gambar target
* [ ] Gambar target diambil dalam rentang waktu yang sama dengan survei
* [ ] Target terlihat jelas di pratinjau saat diklik
* [ ] Semua panel kalibrasi terlihat di setiap gambar target
* [ ] Tidak ada bayangan atau penghalang pada target
* [ ] Untuk kamera ganda: Target ditandai untuk kedua jenis kamera

***

## Pemrosesan Tanpa Target

### LATTICE: Dengan Rekaman DAQ

Jika sensor cahaya DAQ merekam irradiansi turun selama pengambilan gambar LATTICE Anda, tidak diperlukan target:

1. Impor berkas `.daq` (atau DAQ-M `.csv`) beserta citranya
2. Biarkan kolom Target tidak dicentang
3. Reflektansi dihitung secara otomatis dari referensi irradiance yang mengarah ke bawah (downwelling) DAQ
4. Radian tidak pernah memerlukan target atau DAQ — nilainya diperoleh semata-mata dari kalibrasi radiometrik pabrik kamera

### Pemrosesan Tanpa Referensi Apa Pun

Anda juga dapat melakukan pemrosesan tanpa target dan tanpa DAQ:

1. Biarkan semua kotak centang kolom Target tidak dicentang
2. **Nonaktifkan** "Kalibrasi reflektansi / white balance" di Pengaturan Proyek — deteksi target kemudian dilewati sepenuhnya
3. Koreksi vignette tetap akan diterapkan
4. Hasil keluaran tidak akan dikalibrasi untuk reflektansi absolut (LATTICE multispektral tetap mengekspor produk debayered, pratinjau, dan radian)

{% hint style="warning" %}
**Tidak Direkomendasikan untuk pekerjaan ilmiah Survey3**: Tanpa kalibrasi reflektansi, nilai piksel Survey3 hanya mewakili kecerahan relatif, bukan pengukuran reflektansi ilmiah. Gunakan target kalibrasi (atau, untuk LATTICE, sensor cahaya DAQ) untuk hasil yang akurat dan dapat diulang.
{% endhint %}

***

## Langkah Selanjutnya

Setelah Anda menandai gambar target Anda:

1. **Periksa pengaturan Anda** - Lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md)
2. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
3. **Pantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)

Untuk informasi lebih lanjut mengenai target kalibrasi itu sendiri, lihat [Target Kalibrasi](../calibration-targets.md).

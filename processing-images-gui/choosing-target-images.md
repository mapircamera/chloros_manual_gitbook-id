# Memilih Gambar Target

Menandai gambar yang mengandung target kalibrasi adalah langkah penting yang secara signifikan mempercepat alur kerja pemrosesan Chloros. Dengan memilih gambar target secara pra-seleksi, Anda menghilangkan kebutuhan Chloros untuk memindai setiap gambar dalam dataset Anda untuk mencari target kalibrasi.

## Mengapa Menandai Gambar Target?

### Kecepatan Pemrosesan

Tanpa menandai gambar target, Chloros harus:

* Memindai setiap gambar dalam proyek Anda
* Menjalankan algoritma deteksi target pada setiap gambar
* Memeriksa ratusan atau ribuan gambar secara tidak perlu

**Hasil**: Pemrosesan dapat memakan waktu jauh lebih lama, terutama untuk dataset besar.

### Dengan Gambar Target yang Ditandai

Ketika Anda menandai kolom Target untuk gambar tertentu:

* Chloros hanya memindai gambar yang ditandai untuk target
* Deteksi target selesai jauh lebih cepat
* Waktu pemrosesan keseluruhan berkurang secara signifikan

{% hint style=&quot;success&quot; %}
**Peningkatan Kecepatan**: Menandai 2-3 gambar target dalam dataset 500 gambar dapat mengurangi waktu deteksi target dari lebih dari 30 menit menjadi kurang dari 1 menit.
{% endhint %}

***

## Cara Menandai Gambar Target

### Langkah 1: Identifikasi Gambar Target Anda

Lihat gambar yang diimpor di File Browser dan identifikasi gambar mana yang mengandung target kalibrasi.

**Skenario umum:**

* **Target pra-perekaman**: Diambil sebelum memulai sesi
* **Target pasca-perekaman**: Diambil setelah menyelesaikan sesi
* **Target di area perekaman**: Target yang ditempatkan di area perekaman
* **Target ganda**: 2-3 gambar target per sesi (disarankan)

### Langkah 2: Periksa Kolom Target

Untuk setiap gambar yang mengandung target kalibrasi:

1. Temukan gambar di tabel File Browser
2. Temukan kolom **Target** (kolom paling kanan)
3. Klik kotak centang di kolom Target untuk gambar tersebut
4. Ulangi untuk semua gambar yang mengandung target

### Langkah 3: Verifikasi Pilihan Anda

Sebelum pemrosesan, periksa kembali:

* [ ] Semua gambar dengan target kalibrasi telah dicentang
* [ ] Tidak ada gambar non-target yang tercentang secara tidak sengaja
* [ ] Target terlihat jelas pada gambar yang dicentang

***

## Praktik Terbaik untuk Gambar Target

### Pedoman Pengambilan Gambar Target

**Waktu:**

* Ambil gambar target segera sebelum dan selama sesi pengambilan gambar
* Dalam kondisi pencahayaan yang sama dengan sensor cahaya DAQ
* Idealnya, ambil gambar target sesering mungkin untuk hasil terbaik. Jika tidak, data sensor cahaya akan digunakan untuk menyesuaikan kalibrasi seiring waktu.

**Posisi Kamera:**

* Tahan kamera di atas target sehingga berada di tengah dan mengisi sekitar 40-60% pusat gambar.
* Jaga kamera sejajar/nadir dengan permukaan target

**Pencahayaan:**

* Pencahayaan ambient yang sama dengan sensor cahaya DAQ Anda
* Hindari bayangan pada permukaan target
* Jangan blokir sumber cahaya dengan tubuh, kendaraan, atau vegetasi
* Kondisi mendung memberikan hasil yang paling konsisten

**Kondisi Target:**

* Jaga panel target tetap bersih dan kering
* Keempat panel harus terlihat jelas dan tidak terhalang
* Target tegak lurus/nadir terhadap sumber cahaya jika memungkinkan

### Berapa Banyak Gambar Target?

**Minimum:** 1 gambar target per sesi. **Disarankan:** 3-5 gambar target per sesi.

**Jadwal terbaik:**

* 3-5 gambar diambil segera setelah sensor cahaya mulai merekam
* Putar kamera antara pengambilan gambar untuk hasil terbaik
* Opsional: secara berkala di tengah sesi jika kondisi pencahayaan berubah secara konstan

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
* Survey3W\_RGB
* dll.

Gunakan kolom ini untuk memverifikasi bahwa Anda telah menandai target untuk setiap jenis kamera dalam proyek Anda.

***

## Pengaturan Deteksi Target

### Menyesuaikan Sensitivitas Deteksi

Jika Chloros tidak mendeteksi target Anda dengan benar, sesuaikan pengaturan ini di [Pengaturan Proyek](adjusting-project-settings.md):

**Area sampel kalibrasi minimum:**

* **Default**: 25 piksel
* **Tingkatkan** jika mendeteksi objek palsu pada artefak kecil
* **Kurangi** jika target tidak terdeteksi

**Pengelompokan target minimum:**

* **Default**: 60
* **Tingkatkan** jika target terbagi menjadi deteksi ganda
* **Kurangi** jika target dengan variasi warna tidak terdeteksi sepenuhnya

***

## Masalah Umum pada Gambar Target

### Masalah: Tidak Ada Target yang Terdeteksi

**Penyebab kemungkinan:**

* Gambar target tidak ditandai di File Browser
* Target terlalu kecil dalam bingkai (&lt; 30% dari gambar)
* Pencahayaan buruk (bayangan, pantulan cahaya)
* Pengaturan deteksi target terlalu ketat

**Solusi:**

1. Pastikan kolom Target dicentang untuk gambar yang benar
2. Periksa kualitas gambar target di pratinjau
3. Ambil ulang target jika kualitas buruk
4. Sesuaikan pengaturan deteksi target jika diperlukan

### Masalah: Deteksi Target Palsu

**Penyebab kemungkinan:**

* Bangunan putih, kendaraan, atau penutup tanah yang disalahartikan sebagai target
* Bintik-bintik terang di vegetasi
* Sensitivitas deteksi terlalu rendah

**Solusi:**

1. Tandai hanya gambar target sebenarnya untuk membatasi ruang lingkup deteksi
2. Tingkatkan area sampel kalibrasi minimum
3. Tingkatkan nilai klustering target minimum
4. Pastikan gambar target hanya menampilkan target (minimal gangguan latar belakang)

***

## Daftar Periksa Verifikasi

Sebelum memulai pemrosesan, verifikasi pemilihan gambar target Anda:

* [ ] Setidaknya 1 gambar target ditandai per sesi
* [ ] Kotak centang kolom target dicentang untuk semua gambar target
* [ ] Gambar target diambil dalam rentang waktu yang sama dengan survei
* [ ] Target terlihat jelas dalam pratinjau saat diklik
* [ ] Semua 4 panel kalibrasi terlihat dalam setiap gambar target
* [ ] Tidak ada bayangan atau penghalang pada target
* [ ] Untuk kamera ganda: Target ditandai untuk kedua jenis kamera

***

## Pemrosesan Tanpa Target

### Pemrosesan Tanpa Target Kalibrasi

Meskipun tidak direkomendasikan untuk pekerjaan ilmiah, Anda dapat memproses tanpa target:

1. Biarkan semua kotak centang kolom Target tidak dicentang
2. **Nonaktifkan** &quot;Kalibrasi reflektansi&quot; di Pengaturan Proyek
3. Koreksi vignette tetap diterapkan
4. Output tidak akan dikalibrasi untuk reflektansi absolut

{% hint style=&quot;warning&quot; %}
**Tidak Direkomendasikan**: Tanpa kalibrasi reflektansi, nilai piksel hanya mewakili kecerahan relatif, bukan pengukuran reflektansi ilmiah. Gunakan target kalibrasi untuk hasil yang akurat dan dapat diulang.
{% endhint %}

***

## Langkah Selanjutnya

Setelah Anda menandai gambar target Anda:

1. **Periksa pengaturan Anda** - Lihat [Menyesuaikan Pengaturan Proyek](adjusting-project-settings.md)
2. **Mulai pemrosesan** - Lihat [Memulai Pemrosesan](starting-the-processing.md)
3. **Pantau kemajuan** - Lihat [Memantau Pemrosesan](monitoring-the-processing.md)

Untuk informasi lebih lanjut tentang target kalibrasi itu sendiri, lihat [Target Kalibrasi](../calibration-targets.md).

# Jalur Pemrosesan

Chloros 1.3.0 menggunakan jalur pemrosesan 4-thread yang bekerja seperti jalur perakitan bertahap. Setiap thread menangani fase alur kerja yang berbeda, sehingga beberapa gambar dapat berada dalam proses pada tahap yang berbeda secara bersamaan.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arsitektur Jalur Pemrosesan

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Setiap gambar mengalir melalui keempat thread secara berurutan. Dengan pemrosesan multi-thread Chloros+, beberapa gambar dapat ditangani oleh thread yang berbeda secara bersamaan — saat Thread 3 memproses satu gambar, Thread 1 dapat mendeteksi gambar berikutnya, Thread 2 mengkalibrasi gambar lain, dan Thread 4 menyimpan gambar yang sudah selesai ke disk.

Kemajuan dilaporkan per thread dan dikirim melalui Server-Sent Events (backend menerbitkannya di `/api/events`). Pada tampilan kemajuan langsung di CLI, keempat tahap tersebut diberi label **Deteksi, Analisis, Pemrosesan, Ekspor**.***

## Detail Thread

### Thread 1: Deteksi

**Tujuan**: Memuat gambar dan mendeteksi target kalibrasi.

* Membaca berkas gambar dari disk — pasangan Survey3 `.raw`+`.jpg`, pengambilan gambar LATTICE `.tif`/`.tiff`, dan `.dng`
* Mengekstrak metadata EXIF (GPS, model kamera, cap waktu, eksposur)
* Mendeteksi target kalibrasi: geometri target bertanda ArUco untuk tangkapan LATTICE, dan panel detektor klasik untuk foto target kalibrasi Survey3
* Keluaran: data gambar + metadata + hasil deteksi target

Secara utama merupakan utas yang dibatasi oleh I/O dan CPU.

### Thread 2: Kalibrasi

**Tujuan**: Menghitung parameter kalibrasi dari target yang terdeteksi.

* Menghitung koefisien kalibrasi reflektansi dari gambar target
* Menghitung parameter koreksi vignette
* Menentukan kurva kalibrasi per-band
* Keluaran: parameter kalibrasi untuk setiap gambar

Benang komputasi yang dibatasi oleh CPU. Pada eksekusi Survey3 atau campuran, Benang 3 menunggu benang ini saat kalibrasi reflektansi diaktifkan, sehingga koefisiennya siap sebelum gambar mana pun diproses. Pada eksekusi LATTICE saja, Benang 3 berjalan selama deteksi: setiap bingkai dimulai setelah jangkar targetnya final. Sebuah bingkai mengambil target terdekat dalam waktu di kedua sisinya, sehingga menunggu hingga setiap bingkai kamera yang waktunya lebih dekat daripada target terdekat saat ini telah dipindai. Hasilnya sama, hanya saja lebih cepat.

### Thread 3: Pemrosesan (GPU)

**Tujuan**: Menerapkan koreksi dan menghitung indeks vegetasi.**Ini adalah thread yang paling intensif dalam hal komputasi.*** **Debayering**: mengubah data RAW Bayer menjadi gambar multi-saluran
  * Standar (Cepat, Kualitas Sedang) — default, `--debayer standard`
  * Sensitif Tekstur (Lambat, Kualitas Tertinggi) — hanya untuk Chloros+, `--debayer texture-aware`, menggunakan model penghilang noise berbasis AI/ML
  * Pengambilan gambar LATTICE mono (M3M) bersifat single-band: langkah demosaic dan white-balance dilewati untuk gambar-gambar tersebut (dengan pesan log satu baris), sementara gambar M3C/Bayer dalam proses yang sama tetap menjalani langkah-langkah tersebut
* **Koreksi vignette**: menerapkan koreksi vignette lensa ke seluruh gambar
* **Kalibrasi reflektansi**: menerapkan koefisien kalibrasi untuk mengonversi ke nilai reflektansi
* **Perhitungan indeks**: menghitung indeks vegetasi (NDVI, NDRE, GNDVI, …)
* Hasil: data gambar yang telah diproses dan siap diekspor

Thread ini paling diuntungkan dari akselerasi GPU, dan inilah thread yang disesuaikan oleh [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Thread 4: Ekspor

**Tujuan**: Menyimpan gambar yang telah diproses ke disk.

* Menyimpan berkas keluaran dalam format yang dipilih — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Menyematkan metadata ke dalam berkas keluaran (GPS, cap waktu, parameter pemrosesan) — termasuk tag pemrosesan `Chloros:*`, yang mulai versi 1.3.0 ditulis dalam setiap mode pemrosesan, baik untuk Survey3 maupun LATTICE
* Mengatur hasil pemrosesan di bawah folder proyek sebagai `<camera>/<format>/<Product>_Images/` — misalnya `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **File yang diekspor tetap menggunakan nama file sumber; folder tersebut mengidentifikasi produknya.**
* Untuk pengambilan data LATTICE, satu bingkai sumber dapat diolah menjadi beberapa produk (Debayered, Preview, Radiance, Reflectance, Index), masing-masing dalam folder produknya sendiri. Produk Index dihitung dari produk reflectance yang dihasilkan oleh proses tersebut (jika tidak ada, maka dari produk lain yang dihasilkan oleh proses tersebut, seperti radiance). Produk radiance dan reflectance dari kamera multispektral M3C juga dapat memiliki masker pembatas per piksel di `<format>/clamp_masks/` (lihat [Format Gambar Keluaran](../output-image-formats.md#clamp-masks))
* Keluaran: berkas akhir di disk

Secara utama merupakan utas yang dibatasi oleh I/O — penyimpanan SSD meningkatkan kinerjanya secara signifikan.

***

## Di Balik Layar: Eksekutor

Di dalam Utas 3, pekerjaan per-gambar diparalelkan sebagai berikut:

* **Strategi GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) menggunakan kumpulan**proses pekerja**, masing-masing merupakan proses OS terpisah dengan konteks CUDA-nya sendiri. Chloros memulai setiap pekerja dengan menjalankan kembali program backend yang telah diinstal — bukan melalui `multiprocessing` Python, karena proses memulai pekerja tidak berfungsi di dalam aplikasi yang dikemas
* **`CPU_PARALLEL`** menggunakan kumpulan thread (`ThreadPoolExecutor` dari Python) — NumPy dan OpenCV melepaskan GIL, sehingga thread sudah cukup
* Perangkat Jetson dengan RAM bersama 8GB atau kurang melewati executor sepenuhnya dan memproses secara in-process, secara berurutan
* Texture Aware pada GPU dengan VRAM di bawah 7GB juga berjalan secara berurutan — model denoiser tidak dapat dimuat lebih dari sekali

Chloros tidak menggunakan kerangka kerja terdistribusi pihak ketiga apa pun (seperti Ray). Lihat [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) untuk mengetahui bagaimana strategi dan jumlah pekerja dipilih.

***

## Pemrosesan Sekuensial vs. Berpipa

### Mode Bebas (Sekuensial)

Dalam versi gratis Chloros, gambar diproses **satu per satu**, secara berurutan melalui keempat tahap:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI menampilkan bilah kemajuan yang disederhanakan dalam mode gratis; fase-fase serialnya dilaporkan sebagai **Deteksi Target**dan kemudian**Pemrosesan**.

### Mode Chloros+ (Pipelined)

Dengan lisensi Chloros+, keempat utas beroperasi **secara bersamaan** pada gambar yang berbeda:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Bilah kemajuan GUI menampilkan keempat tahap tersebut; arahkan kursor ke atasnya untuk melihat kemajuan per-thread. Di CLI, keempat tahap yang sama ditampilkan secara langsung sebagai **Deteksi, Analisis, Pemrosesan, Ekspor**.

{% hint style="info" %}
**Satu label, dua nama.** CLI menyebut tahap 3 sebagai _Processing_. Umpan kemajuan mode premium di backend — yang ditampilkan oleh bilah kemajuan GUI — memberi label tahap yang sama sebagai _Calibrating_. Keduanya merupakan utas yang sama yang melakukan pekerjaan yang sama (Utas 3: debayer, koreksi, indeks).
{% endhint %}

{% hint style="success" %}
**Pemrosesan pipelined dengan Chloros+** dapat 3–5 kali lebih cepat daripada pemrosesan berurutan, tergantung pada perangkat keras dan ukuran dataset Anda. Peningkatan kecepatan paling signifikan terjadi pada sistem dengan GPU dan SSD yang cepat.
{% endhint %}

***

## Kemajuan Ekspor Thread 4

Thread ekspor memiliki pelacakan kemajuan tersendiri, yang dapat Anda periksa secara terpisah:**Antarmuka Baris Perintah (CLI):**

```bash
chloros-cli export-status
```

**Kit Pengembangan Perangkat Lunak (SDK):**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Pemrosesan selesai ketika Thread 4 mencapai 100%.

{% hint style="info" %}
**Proses yang tidak menulis gambar apa pun dianggap gagal.**Jika berhasil, `chloros-cli process` melaporkan jumlah produk gambar yang ditulis (`Image products written: N`). Jika produk diminta dan**tidak ada**yang ditulis — hanya `project.json` dan `calibration_data.json` — CLI mencetak `Processing finished but wrote no image products.` dan**keluar dengan nilai bukan nol**, dengan menyebutkan nama folder proyek dan penyebab umumnya (folder input tidak dikenali sebagai penangkapan — periksa tata letak dan `--input-level` — atau setiap produk yang diminta tidak berlaku untuk kamera-kamera tersebut). Skrip dapat mengandalkan kode keluar tersebut.
{% endhint %}

***

## Hubungan dengan Adaptasi Komputasi Dinamis

[Adaptasi Komputasi Dinamis](dynamic-compute-adaptation.md) terutama memengaruhi **Thread 3 (Pemrosesan)**:

* **`GPU_PARALLEL`**: Thread 3 memproses beberapa gambar melalui GPU secara bersamaan menggunakan pipa `fused_gpu`
* **`GPU_SINGLE`**: Thread 3 menserialisasikan akses GPU dengan semafor sementara proses pekerja melakukan I/O secara tumpang tindih, menggunakan pipa `fused_gpu` atau pipa `tiled_gpu` yang efisien dalam penggunaan memori
* **`CPU_PARALLEL`**: Thread 3 menggunakan pemrosesan berbasis CPU dengan paralelisme multi-thread

Alokasi memori GPU Thread 3 juga bertambah seiring selesainya Thread 1 dan 2 — lihat [Alokasi Memori GPU Dinamis](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Langkah Selanjutnya

* [Adaptasi Komputasi Dinamis](dynamic-compute-adaptation.md) — Cara Chloros memilih strategi optimal untuk perangkat keras Anda
* [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Perilaku pipa yang spesifik platform pada Jetson
* [Pemantauan Pemrosesan](../processing-images-gui/monitoring-the-processing.md) — Pemantauan kemajuan melalui antarmuka grafis (GUI)
* [Referensi CLI](../reference/cli-reference.md) — `process`, `export-status`, kode keluar, dan tata letak keluaran

# Jalur Pemrosesan

Chloros1.2.0 menggunakan jalur pemrosesan 4-thread yang bekerja seperti jalur perakitan bertahap. Setiap thread menangani fase alur kerja yang berbeda, sehingga beberapa gambar dapat berada dalam proses pada tahap yang berbeda secara bersamaan.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arsitektur Jalur Pemrosesan

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Setiap gambar mengalir melalui keempat utas tersebut secara berurutan. Dengan pemrosesan multi-utas Chloros+, beberapa gambar ditangani oleh utas yang berbeda secara bersamaan — saat Utas 3 memproses satu gambar, Utas 1 dapat mendeteksi gambar berikutnya, Utas 2 mengkalibrasi gambar lain, dan Utas 4 menyimpan gambar yang sudah selesai ke disk.

Kemajuan dilaporkan per thread dan dikirim melalui Server-Sent Events (backend menerbitkannya di `/api/events`). Pada tampilan kemajuan langsung di CLI, keempat tahap tersebut diberi label **Deteksi, Analisis, Pemrosesan, Ekspor**.***

## Detail Thread

### Thread 1: Deteksi

**Tujuan**: Memuat gambar dan mendeteksi target kalibrasi.

* Membaca berkas gambar dari disk — pasangan Survey3 `.raw`+`.jpg`, tangkapan LATTICE `.tif`/`.tiff`, dan `.dng`
* Mengekstrak metadata EXIF (GPS, model kamera, cap waktu, eksposur)
* Mendeteksi target kalibrasi: geometri target bertanda ArUco untuk hasil pengambilan gambar LATTICE, dan panel detektor klasik untuk foto target kalibrasSurvey3
* Keluaran: data gambar + metadata + hasil deteksi target

Secara utama merupakan utas yang terikat pada I/O dan CPU.

### Thread 2: Kalibrasi

**Tujuan**: Menghitung parameter kalibrasi dari target yang terdeteksi.

* Menghitung koefisien kalibrasi reflektansi dari gambar target
* Menghitung parameter koreksi vignette
* Menentukan kurva kalibrasi per-band
* Keluaran: parameter kalibrasi untuk setiap gambar

Sebuah thread komputasi yang dibatasi oleh CPU. Thread 3 akan menunggu thread ini ketika kalibrasi reflektansi diaktifkan, sehingga koefisiennya sudah siap sebelum gambar mana pun diproses.

### Thread 3: Pemrosesan (GPU)

**Tujuan**: Menerapkan koreksi dan menghitung indeks vegetasi.**Ini adalah thread yang paling intensif komputasi.*** **Debayering**: mengonversi data RAW Bayer menjadi gambar multi-saluran
  * Standar (Cepat, Kualitas Sedang) — default, `--debayer standard`
  * Sensitif Tekstur (Lambat, Kualitas Tertinggi) — hanya untuk Chloros+, `--debayer texture-aware`, menggunakan model penghilang noise berbasis AI/ML
  * Pengambilan gambar LATTICE mono (M3M) bersifat single-band: langkah demosaic dan white-balance dilewati untuk gambar-gambar tersebut (disertai pesan log satu baris), sementara gambar M3C/Bayer dalam proses pengambilan yang sama tetap menjalani langkah-langkah tersebut
* **Koreksi vignette**: menerapkan koreksi vignette lensa ke seluruh gambar
* **Kalibrasi reflektansi**: menerapkan koefisien kalibrasi untuk mengonversi ke nilai reflektansi
* **Perhitungan indeks**: menghitung indeks vegetasi (NDVI, NDRE, GNDVI, …)
* Keluaran: data gambar yang telah diproses dan siap diekspor

Thread ini paling diuntungkan dari akselerasi GPU, dan inilah thread yang disesuaikan oleh [Dynamic Compute Adaptation](dynamic-compute-adaptation.md).

### Thread 4: Ekspor

**Tujuan**: Menyimpan gambar yang telah diproses ke disk.

* Menulis berkas keluaran dalam format yang dipilih — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Menyematkan metadata ke dalam berkas keluaran (GPS, cap waktu, parameter pemrosesan)
* Mengatur berkas keluaran di bawah folder proyek sebagai `<camera>/<format>/<Product>_Images/` — misalnya `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **File yang diekspor mempertahankan nama file sumber; folder mengidentifikasi produk.**
* Untuk pengambilan data LATTICE, satu bingkai sumber dapat diproses menjadi beberapa produk (Debayered, Preview, Radiance, Reflectance, Index), masing-masing berada di folder produknya sendiri
* Hasil: file akhir di disk

Secara utama merupakan utas yang dibatasi oleh I/O — penyimpanan SSD meningkatkan kinerjanya secara signifikan.

***

## Di Balik Layar: Eksekutor

Di dalam Utas 3, pekerjaan per gambar diparalelkan dengan `concurrent.futures` standar dari Python:

* **Strategi GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) menggunakan `ProcessPoolExecutor` dengan metode**spawn** — setiap pekerja merupakan proses terpisah dengan konteks CUDA-nya sendiri (`fork` akan mewarisi status CUDA yang telah diinisialisasi dari induknya dan merusak anak-anaknya)
* **`CPU_PARALLEL`** menggunakan `ThreadPoolExecutor` — NumPy dan OpenCV melepaskan GIL, sehingga thread sudah cukup
* Perangkat Jetson dengan RAM bersama 8GB atau kurang melewati eksekutor sepenuhnya dan memproses secara in-process, secara berurutan
* Texture Aware pada GPU dengan VRAM di bawah 7GB juga berjalan secara berurutan — model denoiser tidak dapat dimuat lebih dari sekali

Chlorostidak menggunakan kerangka kerja terdistribusi pihak ketiga (seperti Ray). Lihat [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) untuk mengetahui bagaimana strategi dan jumlah worker dipilih.

***

## Pemrosesan Sekuensial vs. Berpipa

### Mode Bebas (Sekuensial)

Dalam versi gratis Chloros, gambar diproses **satu per satu**, secara berurutan melalui keempat tahap:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

GUI menampilkan bilah kemajuan yang disederhanakan dalam mode gratis; fase-fase serialnya dilaporkan sebagai **Deteksi Target**dan kemudian**Pemrosesan**.

### Mode Chloros+ (Berurutan)

Dengan lisensi Chloros+, keempat utas (thread) beroperasi **secara bersamaan** pada gambar yang berbeda:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Bilah kemajuan GUI menampilkan empat tahap; arahkan kursor ke atasnya untuk melihat kemajuan per-thread. Di CLI, keempat tahap yang sama ditampilkan secara langsung sebagai **Deteksi, Analisis, Pemrosesan, Ekspor**.

{% hint style="info" %}
**Satu label, dua nama.** CLI menyebut tahap 3 sebagai _Processing_. Umpan kemajuan mode premium di backend — yang ditampilkan oleh bilah kemajuan GUI — memberi label tahap yang sama sebagai _Calibrating_. Keduanya merupakan utas yang sama yang melakukan pekerjaan yang sama (Utas 3: debayer, koreksi, indeks).
{% endhint %}

{% hint style="success" %}
**Pemrosesan pipelined dengan Chloros+** dapat 3–5 kali lebih cepat daripada pemrosesan sekuensial, tergantung pada perangkat keras dan ukuran dataset Anda. Peningkatan kecepatan ini paling signifikan pada sistem dengan GPU dan SSD yang cepat.
{% endhint %}

***

## Kemajuan Ekspor Thread 4

Thread ekspor memiliki pelacakan kemajuan tersendiri, yang dapat Anda periksa secara terpisah:**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Pemrosesan selesai ketika Thread 4 mencapai 100%.

{% hint style="info" %}
**Proses yang tidak menulis gambar apa pun dianggap gagal.**Jika berhasil, `chloros-cli process` melaporkan berapa banyak produk gambar yang telah ditulis (`Image products written: N`). Jika produk diminta tetapi**tidak ada**yang ditulis — hanya `project.json` dan `calibration_data.json` — CLI mencetak `Processing finished but wrote no image products.` dan**keluar dengan nilai bukan nol**, dengan menyebutkan nama folder proyek dan penyebab umumnya (folder masukan tidak dikenali sebagai penangkapan — periksa tata letak dan `--input-level` — atau setiap produk yang diminta tidak berlaku untuk kamera-kamera tersebut). Skrip dapat mengandalkan kode keluar tersebut.
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
* [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Perilaku pipa yang spesifik untuk platform pada Jetson
* [Pemantauan Pemrosesan](../processing-images-gui/monitoring-the-processing.md) — Pemantauan kemajuan melalui antarmuka pengguna grafis (GUI)
* [Referensi CLI](../reference/cli-reference.md) — `process`, `export-status`, kode keluar, dan tata letak keluaran

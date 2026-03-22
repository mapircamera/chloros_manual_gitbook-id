# Jalur Pemrosesan

Chloros 1.1.0 menggunakan jalur pemrosesan 4-thread yang beroperasi layaknya jalur perakitan bertahap. Setiap thread menangani fase yang berbeda dalam alur kerja pemrosesan, sehingga memungkinkan beberapa gambar diproses secara bersamaan pada tahap yang berbeda.

***

## Arsitektur Jalur

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Setiap gambar mengalir melalui keempat thread secara berurutan. Dengan pemrosesan multi-thread Chloros+, beberapa gambar dapat berada di thread yang berbeda secara bersamaan — saat Thread 3 memproses satu gambar, Thread 1 dapat mendeteksi gambar berikutnya, Thread 2 dapat mengkalibrasi gambar lain, dan Thread 4 dapat menulis gambar yang telah diproses sebelumnya ke disk.

***

## Detail Thread

### Thread 1: Deteksi

**Tujuan**: Memuat gambar dan mendeteksi target kalibrasi.

* Membaca berkas gambar dari disk (RAW, JPG)
* Mengekstrak metadata EXIF (GPS, model kamera, cap waktu, eksposur)
* Mendeteksi target kalibrasi ArUco pada gambar target yang ditandai
* Output: data gambar + metadata + hasil deteksi target

Ini terutama merupakan thread yang dibatasi oleh I/O dan CPU.

### Thread 2: Kalibrasi

**Tujuan**: Menghitung parameter kalibrasi dari target yang terdeteksi.

* Menghitung koefisien kalibrasi reflektansi dari gambar target
* Menghitung parameter koreksi vignette
* Menentukan kurva kalibrasi per-band
* Output: parameter kalibrasi untuk setiap gambar

Ini adalah thread komputasi yang bergantung pada CPU.

### Thread 3: Pemrosesan (GPU)

**Tujuan**: Menerapkan koreksi dan menghitung indeks vegetasi.**Ini adalah thread yang paling intensif komputasi.*** **Debayering**: Mengonversi data pola Bayer RAW menjadi gambar multi-saluran
  * Standar (Cepat, Kualitas Sedang) — default
  * Texture Aware (Lambat, Kualitas Tertinggi) — hanya Chloros+, menggunakan denoising AI/ML
* **Koreksi vignette**: Menerapkan koreksi vignette lensa di seluruh gambar
* **Kalibrasi reflektansi**: Menerapkan koefisien kalibrasi untuk mengonversi ke nilai reflektansi
* **Perhitungan indeks**: Menghitung indeks vegetasi (NDVI, NDRE, GNDVI, dll.)
* Keluaran: data gambar yang telah diproses dan siap diekspor

Thread ini paling diuntungkan dari akselerasi GPU. Sistem [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) terutama mengoptimalkan perilaku thread ini.

### Thread 4: Ekspor

**Tujuan**: Menulis gambar yang telah diproses ke disk.

* Menulis file keluaran dalam format yang dipilih (TIFF 16-bit, TIFF 32-bit %, PNG, JPG)
* Menyematkan metadata EXIF ke dalam berkas keluaran (GPS, cap waktu, parameter pemrosesan)
* Mengorganisir keluaran ke dalam subfolder berdasarkan model kamera
* Keluaran: berkas akhir di disk

Ini terutama merupakan thread yang dibatasi oleh I/O. Penyimpanan SSD secara signifikan meningkatkan kinerja Thread 4.

***

## Pemrosesan Berurutan vs. Berpipa

### Mode Gratis (Berurutan)

Pada versi gratis Chloros, gambar diproses **satu per satu**, secara berurutan melalui keempat tahap:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

Bilah kemajuan antarmuka pengguna menampilkan 2 tahap: Deteksi Target dan Pemrosesan.

### Mode Chloros+ (Pipa)

Dengan lisensi Chloros+, keempat thread beroperasi **secara bersamaan** pada gambar yang berbeda:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

Bilah kemajuan antarmuka pengguna (GUI) menampilkan 4 tahap: Deteksi, Analisis, Kalibrasi, Ekspor. Arahkan kursor ke bilah kemajuan untuk melihat kemajuan per thread.

{% hint style="success" %}
**Pemrosesan pipelined dengan Chloros+** dapat 3-5 kali lebih cepat daripada pemrosesan berurutan, tergantung pada perangkat keras dan ukuran dataset Anda. Peningkatan kecepatan paling signifikan terjadi pada sistem dengan GPU dan SSD yang cepat.
{% endhint %}

***

## Kemajuan Ekspor Thread 4

Pada Chloros 1.1.0, thread ekspor (Thread 4) memiliki pelacakan kemajuan tersendiri. Anda dapat memantau kemajuan ekspor secara terpisah:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

Pemrosesan selesai ketika Thread 4 mencapai 100%.

***

## Hubungan dengan Adaptasi Komputasi Dinamis

Sistem [Adaptasi Komputasi Dinamis](dynamic-compute-adaptation.md) terutama memengaruhi **Thread 3 (Pemrosesan)**:

* Strategi **`GPU_PARALLEL`**: Thread 3 memproses beberapa gambar secara bersamaan melalui GPU menggunakan pipa `fused_gpu`
* Strategi **`GPU_SINGLE`**: Thread 3 memproses satu gambar pada satu waktu menggunakan pipa `tiled_gpu` yang efisien memori
* Strategi **`CPU_PARALLEL`**: Thread 3 menggunakan pemrosesan berbasis CPU dengan paralelisme multi-thread

Alokasi memori GPU Thread 3 juga berubah secara dinamis saat Thread 1 dan 2 selesai — lihat [Alokasi Memori GPU Dinamis](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Langkah Selanjutnya

* [Adaptasi Komputasi Dinamis](dynamic-compute-adaptation.md) — Bagaimana Chloros memilih strategi optimal untuk perangkat keras Anda
* [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Perilaku pipa yang spesifik platform pada Jetson
* [Pemantauan Pemrosesan](../processing-images-gui/monitoring-the-processing.md) — Pemantauan kemajuan melalui antarmuka pengguna grafis

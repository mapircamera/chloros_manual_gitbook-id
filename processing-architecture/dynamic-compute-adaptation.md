# Adaptasi Komputasi Dinamis

Chloros 1.1.0 menghadirkan deteksi perangkat keras cerdas dan pemilihan strategi pemrosesan otomatis. Mesin pemrosesan menyesuaikan diri dengan perangkat keras Anda — mulai dari Jetson Nano hingga workstation multi-GPU — tanpa perlu konfigurasi manual.

***

## Cara Kerjanya

Saat Chloros dimulai, sistem Anda akan diprofilkan secara otomatis:

1. **Mendeteksi sistem operasi** — Windows atau Linux
2. **Mengidentifikasi inti CPU dan total RAM**

3.**Mendeteksi keberadaan GPU** — kemampuan NVIDIA CUDA, VRAM, model
4. **Mengidentifikasi model Jetson** (jika berlaku) — melalui `/proc/device-tree/model`
5. **Memeriksa sensor suhu** (Jetson) — untuk pemrosesan yang sensitif terhadap suhu
6. **Memilih strategi komputasi optimal** — berdasarkan semua perangkat keras yang terdeteksi
7. **Mengonfigurasi jumlah pekerja, jenis pipa, dan alokasi memori** secara otomatis

Hasilnya disimpan dalam cache sehingga eksekusi berikutnya dimulai lebih cepat. Jika perangkat keras berubah (misalnya, GPU ditambahkan), Chloros akan melakukan profil ulang pada peluncuran berikutnya.

***

## Strategi Komputasi

Chloros memilih salah satu dari tiga strategi komputasi berdasarkan perangkat keras Anda:

| Strategi | GPU Diperlukan | Pekerja | Pipeline | Cocok Untuk |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Ya (12GB+ VRAM atau 16GB+ bersama) | 3-4 | `fused_gpu` | GPU desktop dengan 12GB+, Jetson Orin NX 16GB, AGX Orin |
| **`GPU_SINGLE`** | Ya (&lt; 12GB VRAM) | 1-3 | `tiled_gpu` | GPU entry-level, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Tidak | inti - 1 | `cpu_fallback` | Sistem tanpa GPU NVIDIA |

### Jenis Pipeline

* **`fused_gpu`** — Jalur pemrosesan GPU penuh. Semua operasi debayer, koreksi, dan indeks dijalankan di GPU dalam satu lintasan terpadu. Throughput tertinggi tetapi membutuhkan lebih banyak VRAM.
* **`tiled_gpu`** — Jalur GPU yang efisien memori. Memproses gambar dalam bentuk ubin agar sesuai dengan memori GPU yang terbatas. Throughput lebih rendah tetapi berfungsi pada perangkat dengan keterbatasan memori.
* **`cpu_fallback`** — Pemrosesan hanya menggunakan CPU dengan paralelisme multi-thread. Digunakan jika tidak ada GPU NVIDIA yang tersedia.***

## Perilaku Khusus Platform

| Platform | Strategi | Pekerja | Jalur | Catatan |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serial) | Mode efisien memori, memproses satu gambar sekaligus |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (bersamaan) | Perangkat tepi yang direkomendasikan — pemrosesan GPU paralel sejati |
| **Jetson AGX Orin 64GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (bersamaan) | Kinerja tepi maksimum |
| **Desktop dengan GPU 8GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Kinerja desktop yang baik dengan tile yang efisien memori |
| **Desktop dengan GPU 12GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Kinerja desktop optimal |
| **Sistem hanya CPU** | `CPU_PARALLEL` | inti - 1 | `cpu_fallback` | Tidak memerlukan GPU, menggunakan ThreadPool |

{% hint style="info" %}
**Memori terpadu Jetson**: Perangkat Jetson berbagi memori GPU dan CPU. Sebuah Jetson Orin NX 16GB melaporkan ~15,3GB VRAM, namun ini adalah RAM fisik yang sama yang digunakan oleh sistem operasi dan proses CPU. Chloros memperhitungkan hal ini saat menetapkan ambang batas alokasi memori.
{% endhint %}

***

## Alokasi Memori GPU Dinamis

Chloros menggunakan [pipeline pemrosesan 4-thread](processing-pipeline.md):

* **Thread 1** (Deteksi) — Pemuatan gambar, parsing EXIF, deteksi target
* **Thread 2** (Kalibrasi) — Perhitungan kalibrasi reflektansi
* **Thread 3** (Pemrosesan) — Debayer GPU, koreksi vignette, perhitungan indeks
* **Thread 4** (Ekspor) — Penulisan file, penyisipan metadata

Saat thread pipa sebelumnya menyelesaikan tugasnya (misalnya, semua gambar telah terdeteksi), alokasi memori GPU mereka dilepaskan dan **didistribusikan ulang ke thread aktif yang tersisa**. Ini berarti Thread 3 (tahap yang intensif GPU) mendapatkan memori yang semakin banyak seiring berjalannya pipeline, sehingga meningkatkan throughput untuk pekerjaan yang paling intensif komputasi.

### Tahap Alokasi

| Tahap | Thread Aktif | Distribusi Memori GPU |
| --- | --- | --- |
| **Awal** | 1, 2, 3, 4 | Dibagi ke semua thread |
| **Awal-Tengah** | 2, 3, 4 | Memori Thread 1 didistribusikan ulang |
| **Tengah-Akhir** | 3, 4 | Memori Thread 1+2 dialihkan ke 3+4 |
| **Akhir** | 3 atau 4 | Memori maksimum untuk thread yang tersisa |***

## Pemrosesan Texture Aware

Metode debayer Texture Aware (hanya Chloros+) menggunakan memori GPU yang jauh lebih besar daripada metode Standar karena model denoising AI/ML:

* Sistem dengan **&lt; 7GB VRAM** dipaksa masuk ke dalam loop pemrosesan sinkron untuk mode Texture Aware (satu gambar pada satu waktu)
* Sistem dengan **7GB+ VRAM** dapat memproses Texture Aware secara bersamaan, meskipun dengan jumlah worker yang lebih sedikit dibandingkan dengan metode Standard***

## Manajemen Termal (Jetson)

Perangkat Jetson memiliki batasan termal, terutama pada deployment tertutup atau di udara. Chloros memantau suhu GPU dan CPU serta secara otomatis menyesuaikan pemrosesan:

| Suhu | Tanggapan |
| --- | --- |
| **&lt; 70°C** | Operasi normal — kecepatan penuh |
| **70°C** (Peringatan) | Kurangi ukuran batch |
| **80°C** (Kritikal) | Pembatasan agresif — kurangi koncurrency dan jumlah pekerja |
| **90°C** (Matikan) | Hentikan pemrosesan GPU sepenuhnya |

Pemantauan suhu menggunakan `tegrastats` pada platform Jetson. Pada sistem desktop dengan pendinginan yang memadai, pembatasan termal jarang terpicu.

***

## Penanganan Tekanan Memori

Chloros memantau tekanan memori sistem selama pemrosesan:

* **Ambang batas memori**: Penggunaan 85% memicu perilaku konservatif
* **Pengurangan OOM**: Jika terjadi peristiwa kehabisan memori, alokasi dikurangi sebesar 25% (pengali 0,75x)
* **Pergantian pipa**: Di bawah tekanan memori yang parah, pipa pemrosesan beralih dari `fused_gpu` ke `tiled_gpu` secara otomatis
* **Rekomendasi swap**: Pada Jetson, Chloros memberi peringatan jika ruang swap tidak mencukupi untuk ukuran dataset Anda***

## Pemantauan Adaptasi Komputasi

### Output Status CLI

Saat pemrosesan dimulai, CLI menampilkan profil perangkat keras yang terdeteksi:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnostik Sistem

Jalankan `chloros-cli selftest` untuk melihat profil perangkat keras lengkap dan memverifikasi kemampuan komputasi:

```bash
chloros-cli selftest
```

Ini memeriksa ketersediaan CUDA, memori GPU, model denoiser, dan konektivitas backend.

***

## Langkah Selanjutnya

* [Pipa Pemrosesan](processing-pipeline.md) — Memahami arsitektur pipa 4-thread
* [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Deploy dan optimasi khusus Jetson
* [CLI : Baris Perintah](../CLI.md) — Referensi lengkap CLI

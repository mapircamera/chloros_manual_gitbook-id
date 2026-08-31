# Adaptasi Komputasi Dinamis

Chloros 1.2.0 menggunakan deteksi perangkat keras dan pemilihan strategi pemrosesan otomatis. Mesin pemrosesan ini menyesuaikan diri dengan perangkat keras Anda — mulai dari Jetson Orin Nano hingga workstation multi-GPU — tanpa perlu konfigurasi manual.

***

## Cara Kerjanya

Saat Chloros dimulai, aplikasi ini akan menganalisis sistem Anda:

1. **Mendeteksi sistem operasi** — Windows atau Linux
2. **Mengidentifikasi inti CPU dan total RAM**

3.**Mendeteksi keberadaan GPU** — Kemampuan NVIDIA CUDA, VRAM, model
4. **Mengidentifikasi model Jetson** (jika ada) — melalui `/proc/device-tree/model`
5. **Memeriksa sensor termal** (Jetson) — untuk pemrosesan yang mempertimbangkan suhu
6. **Memilih strategi komputasi** — berdasarkan semua perangkat keras yang terdeteksi
7. **Mengonfigurasi jumlah pekerja, jenis pipa, dan alokasi memori** secara otomatis

Profil yang terdeteksi disimpan dalam cache untuk sesi tersebut di memori dan di disk, sehingga eksekusi berikutnya dapat dimulai lebih cepat:

| Platform | Profil yang disimpan dalam cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (mengutamakan `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Hapus file tersebut untuk memaksa deteksi ulang — berguna setelah menambahkan GPU atau RAM tambahan. Chloros juga akan mendeteksi ulang secara otomatis jika cache ditulis oleh versi lama yang tidak kompatibel.

***

## Strategi Komputasi

Chloros memilih salah satu dari tiga strategi komputasi berdasarkan perangkat keras Anda:

| Strategi | Dipilih Saat | Pekerja | Pelaksana | Pipa |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA yang melaporkan**VRAM 12 GB+**(pada memori terpadu Jetson, juga memerlukan total RAM bersama 12 GB+) | `min(4, VRAM ÷ 4GB)`, minimal 2 —**dibatasi hingga 2 pada Jetson** | `ProcessPoolExecutor` (spawn) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA dengan**VRAM 2–12GB**| 3 (tumpang tindih I/O; akses GPU diserialisasikan oleh semafor).**1 (sekuensial) pada Jetson dengan RAM di bawah 12 GB** | `ProcessPoolExecutor` (spawn); sekuensial dalam proses pada Jetson dengan RAM rendah | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Tanpa GPU CUDA, atau VRAM di bawah 2 GB | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Contoh penerapan rumus pekerja `GPU_PARALLEL`: VRAM 12 GB → 3 pekerja, 16 GB+ → 4 pekerja, Jetson apa pun → 2 pekerja.

Paralelisme diimplementasikan dengan Python standar `concurrent.futures`: strategi GPU menggunakan `ProcessPoolExecutor` dengan metode **spawn** (setiap pekerja adalah proses terpisah dengan konteks CUDA sendiri — `fork` akan menyalin status CUDA yang sudah diinisialisasi dan merusak proses anak), sedangkan strategi CPU menggunakan `ThreadPoolExecutor`. Chloros tidak menggunakan kerangka kerja terdistribusi pihak ketiga apa pun (seperti Ray).

### Jenis Pipeline

* **`fused_gpu`** — Jalur pemrosesan GPU penuh. Operasi debayer, koreksi, dan pengindeksan dijalankan di GPU dalam satu lintasan terpadu. Throughput tertinggi, membutuhkan VRAM paling banyak.
* **`tiled_gpu`** — Jalur GPU yang efisien memori. Memproses gambar dalam bentuk ubin agar sesuai dengan memori GPU yang terbatas. Throughput lebih rendah tetapi berfungsi pada perangkat dengan keterbatasan memori.
* **`cpu_fallback`** — Pemrosesan hanya menggunakan CPU dengan paralelisme multi-thread. Digunakan jika tidak tersedia GPU NVIDIA, dan sebagai opsi cadangan terakhir jika kedua jalur GPU gagal.

Rantai cadangan runtime selalu `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Penggantian Strategi Secara Manual

Atur variabel lingkungan `CHLOROS_STRATEGY` untuk memaksa penggunaan strategi tertentu — solusi darurat bagi ahli jika deteksi otomatis memilih opsi yang tidak sesuai dengan situasi Anda (misalnya, untuk menjaga GPU tetap bebas untuk pekerjaan lain):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Variabel ini dicocokkan tanpa membedakan huruf besar-kecil; apa pun yang bukan salah satu dari tiga nama tersebut akan diabaikan dan deteksi otomatis akan berjalan seperti biasa. Saat pengaturan manual diterapkan, Chloros tetap akan menentukan jumlah pekerja untuk Anda:

| Pengaturan Manual | Jumlah pekerja yang digunakan |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Disarankan untuk mengaturnya per perintah daripada secara permanen, sehingga proses normal tetap beradaptasi secara otomatis.

***

## Perilaku Khusus Platform

| Platform | Strategi | Pekerja | Pipa | Catatan |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekuensial) | Mode hemat memori, satu gambar pada satu waktu |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekuensial) | RAM bersama di bawah 12GB memaksa pemrosesan sekuensial |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (bersamaan) | Perangkat tepi yang direkomendasikan — dibatasi oleh Jetson hingga 2 pekerja |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (bersamaan) | Kinerja tepi maksimum (juga dibatasi oleh Jetson hingga 2 pekerja) |
| **Komputer desktop dengan GPU 8GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 pekerja saling tumpang tindih dalam I/O sementara semafor menserialisasikan akses GPU |
| **Desktop dengan GPU 12GB+** | `GPU_PARALLEL` | 3-4 | `fused_gpu` (bersamaan) | Kinerja desktop optimal: 12GB → 3 pekerja, 16GB+ → 4 |
| **Sistem hanya CPU** | `CPU_PARALLEL` | inti fisik − 1 (min 2) | `cpu_fallback` | Tidak memerlukan GPU, menggunakan kumpulan utas |

{% hint style="info" %}
**Memori terpadu Jetson**: Perangkat Jetson berbagi memori GPU dan CPU. Sebuah Jetson Orin NX 16GB melaporkan ~15,3GB VRAM, tetapi itu adalah RAM fisik yang sama yang digunakan oleh sistem operasi dan proses CPU. Itulah mengapa Jetson 16GB+ memenuhi syarat untuk `GPU_PARALLEL` seperti GPU desktop 12GB+, namun dibatasi hingga 2 pekerja — GPU, proses pekerja, dan konteks CUDA per pekerja semuanya mengambil alokasi dari kumpulan memori bersama yang sama.
{% endhint %}

### Anggaran GPU berdasarkan VRAM (GPU diskrit)

Pada host x86_64 dengan GPU NVIDIA diskrit, VRAM yang terdeteksi juga menentukan seberapa besar kapasitas pemrosesan kartu yang dapat digunakan dan seberapa besar ukuran batch dapat bertambah:

| VRAM yang Terdeteksi | Batas atas anggaran GPU | Pengali ukuran batch |
| --- | --- | --- |
| **8GB+** | 90% | ×2,0 |
| **6–8GB** | 85% | ×1,75 |
| **3,5–6GB** | 80% | ×1,5 |
| **2–3,5GB** | 75% | ×1,25 |
| **Di bawah 2GB** | 70% | ×1,0 |

GPU terpisah hanya mengalokasikan 0,5GB untuk sistem, karena tidak berbagi RAM sistem. Profil Jetson mengalokasikan jauh lebih banyak dan membatasi lebih rendah — lihat [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Alokasi Memori GPU Dinamis

Chloros menggunakan [jalur pemrosesan 4-thread](processing-pipeline.md):

* **Thread 1** (Deteksi) — Pemuatan gambar, penguraian EXIF, deteksi target
* **Thread 2** (Kalibrasi) — Perhitungan kalibrasi reflektansi
* **Thread 3** (Pemrosesan) — Debayer GPU, koreksi vignette, perhitungan indeks
* **Thread 4** (Ekspor) — Penulisan file, penyisipan metadata

Thread 1, 2, dan 4 tidak banyak menggunakan GPU; sedangkan Thread 3 adalah yang paling banyak menggunakan GPU. Saat thread pipa sebelumnya selesai, alokasi GPU-nya **didistribusikan ulang ke thread aktif yang tersisa**, sehingga Thread 3 mendapatkan memori yang semakin banyak seiring berjalannya proses.

### Tahapan Alokasi

| Tahap | Thread Aktif | Distribusi Memori GPU |
| --- | --- | --- |
| **Awal** | 1, 2, 3, 4 | Dibagi ke semua thread, sebagian besar dialokasikan ke Thread 3 |
| **Awal-Tengah** | 2, 3, 4 | Bagian Thread 1 didistribusikan ulang |
| **Tengah-Akhir** | 3, 4 | Bagian Thread 1 dan 2 dialihkan ke Thread 3 dan 4 |
| **Akhir** | 3 atau 4 | Thread aktif terakhir mendapatkan alokasi maksimumnya |

Ada dua aturan yang mengatur angka-angka ini:

* Sebuah thread yang merupakan **satu-satunya** thread aktif akan diberikan alokasi maksimum sesuai profilnya.
* Ketika lebih dari satu tugas GPU *berat* aktif, alokasi dasar setiap tugas berat dibagi di antara mereka (tidak pernah di bawah batas minimum yang dikonfigurasi).

Nilai yang sebenarnya digunakan saat runtime adalah nilai **terendah** antara alokasi profil platform dan rekomendasi real-time dari pemantau memori GPU, sehingga kartu yang sibuk selalu diutamakan daripada profil yang terlalu optimis.***

## Pemrosesan yang Memperhatikan Tekstur

Debayer Texture Aware (**hanya Chloros+** — `--debayer texture-aware`) menjalankan model penghilang noise berbasis AI/ML yang membutuhkan sekitar 1,75 GB VRAM dalam FP16 per salinan, sehingga menggunakan memori GPU jauh lebih banyak daripada metode Standar:

* Sistem dengan **VRAM di bawah 7 GB**memproses Texture Aware dalam**loop sinkron, satu gambar pada satu waktu** — beberapa salinan model tidak dapat dimuat, dan kumpulan pekerja hanya akan menambah persaingan
* Sistem dengan **VRAM 7 GB atau lebih** dapat memproses Texture Aware secara bersamaan, meskipun dengan jumlah pekerja yang lebih sedikit dibandingkan dengan metode Standard
* Pada **Jetson**, Texture Aware selalu diikat ke satu pekerja, dan pada model berdaya rendah (Nano, Orin Nano), fitur ini juga secara otomatis menerapkan batas frekuensi GPU — lihat [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)***

## Manajemen Termal (Jetson)

Perangkat Jetson memiliki batasan termal, terutama pada penerapan di lingkungan tertutup atau di udara. Chloros memantau sensor suhu onboard Jetson dan secara otomatis menyesuaikan ukuran batch:

| Suhu | Tanggapan |
| --- | --- |
| **&lt; 70°C** | Operasi normal — kecepatan penuh |
| **70°C** (Peringatan) | Ukuran batch dikurangi secara bertahap (100% → 50% antara 70°C dan 80°C) |
| **80°C** (Kritikal) | Pembatasan kecepatan agresif (50% → 0% antara 80°C dan 90°C) |
| **90°C** (Mati) | Menghentikan pemrosesan GPU sepenuhnya |

Pada sistem desktop dengan pendinginan yang memadai, pembatasan termal jarang terpicu.

***

## Penanganan Tekanan Memori

Chloros terus memantau memori GPU selama pemrosesan dan bereaksi pada tiga tingkatan.

**Ukuran batch.** Sebuah batch dimulai dengan 8 gambar dikalikan dengan faktor pengali platform dari tabel di atas. Chloros kemudian memeriksa VRAM yang tersedia, menyisihkan 20% darinya untuk overhead PyTorch sendiri, dan mengasumsikan sekitar 100MB memori GPU per gambar 12MP — ukuran batch adalah yang lebih kecil di antara batas yang ditentukan berdasarkan memori atau nilai dasar platform. Ukuran batch tidak pernah turun di bawah 1.**Pengurangan preventif.**Di atas**85% pemanfaatan VRAM**, ukuran batch dikurangi sebelum terjadi kegagalan.**Pemangkasan alokasi per-thread.** Seiring meningkatnya pemanfaatan secara real-time, alokasi memori GPU setiap thread dikurangi: ×0,75 di atas 80% pemanfaatan, ×0,5 di atas 90%. Batas pemantauan adalah 70% (konservatif), 85% (batas operasi normal), dan 95% (risiko OOM).**Penundaan dan pemulihan OOM.** Jika kejadian kehabisan memori tetap terjadi:

* ukuran batch akan **dipotong setengahnya**, dan dipotong setengahnya lagi pada setiap kejadian OOM berturut-turut — setiap batch berikutnya yang berhasil akan mengurangi penalti tersebut satu langkah
* alokasi thread aktif dipotong menjadi 70% dari nilai saat ini dan alokator beralih ke strategi konservatifnya, kemudian kembali melonggarkan setelah serangkaian alokasi yang berhasil
* di bawah tekanan berat, pipa alur beralih dari `fused_gpu` ke `tiled_gpu`, dan ke `cpu_fallback` sebagai upaya terakhir

**RAM Host (Jetson).** Sebelum pemrosesan, CLI memperkirakan memori host puncak berdasarkan jumlah gambar dan mode debayer Anda, serta memberikan peringatan jika RAM ditambah swap berbasis file kemungkinan tidak mencukupi, sekaligus mencetak perintah tepat untuk menambahkan swap — lihat [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).***

## Pemantauan Adaptasi Komputasi

### Diagnostik Sistem

`chloros-cli selftest` adalah cara tercepat untuk memastikan apa yang dilihat oleh lapisan komputasi:

```bash
chloros-cli selftest
```

Tujuh pemeriksaan ini mencakup versi, ketersediaan port, proses startup backend, `/api/test`, informasi sistem, keberadaan model denoiser, dan kesiapan CUDA serta denoiser. Pemeriksaan ke-5 menampilkan baris spesifikasi perangkat keras secara langsung:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Pemeriksaan ke-7 menampilkan `CUDA: <bool>, Denoiser: <bool>` — keduanya harus bernilai benar agar Texture Aware dapat digunakan sama sekali.

### Log Backend

Strategi dan jumlah pekerja dipilih di dalam backend pada awal setiap eksekusi — tidak ada banner CLI yang mengumumkannya. Ketika terjadi perilaku yang tidak terduga (jalur GPU yang beralih ke cadangan, OOM, atau denoiser yang tidak dapat dimuat), log backend untuk sesi tersebutlah yang menampilkannya:

| Platform | Lokasi log |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (satu berkas per peluncuran) |
| **Linux, CLI-backend yang dimulai** | juga `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Kemajuan Real-Time

Selama proses berjalan, CLI menampilkan kemajuan real-time per-thread (Deteksi, Analisis, Pemrosesan, Ekspor) yang disiarkan melalui Server-Sent Events — ini menjadi indikator praktis apakah Thread 3 merupakan titik leher botol. Lihat [Pipa Pemrosesan](processing-pipeline.md).

***

## Langkah Selanjutnya

* [Pipa Pemrosesan](processing-pipeline.md) — Memahami arsitektur pipa 4-thread
* [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Penerapan dan optimasi khusus Jetson
* [CLI : Baris Perintah](../CLI.md) — Panduan CLI
* [Referensi CLI](../reference/cli-reference.md) — Daftar perintah lengkap untuk versi 1.2.0

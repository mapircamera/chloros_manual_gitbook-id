# Adaptasi Komputasi Dinamis

Chloros 1.3.0 menggunakan deteksi perangkat keras dan pemilihan strategi pemrosesan otomatis. Mesin pemrosesan menyesuaikan diri dengan perangkat keras Anda — mulai dari Jetson Orin Nano hingga workstation multi-GPU — tanpa perlu konfigurasi manual.

***

## Cara Kerjanya

Saat Chloros dimulai, aplikasi ini akan membuat profil sistem Anda:

1. **Mendeteksi sistem operasi** — Windows atau Linux
2. **Mengidentifikasi inti CPU dan total RAM**

3.**Mendeteksi keberadaan GPU** — kemampuan NVIDIA CUDA, VRAM, model
4. **Mengidentifikasi model Jetson** (jika ada) — melalui `/proc/device-tree/model`
5. **Memeriksa sensor termal** (Jetson) — untuk pemrosesan yang mempertimbangkan suhu
6. **Memilih strategi komputasi** — berdasarkan semua perangkat keras yang terdeteksi
7. **Mengonfigurasi jumlah pekerja, jenis pipa, dan alokasi memori** secara otomatis

Profil yang terdeteksi disimpan dalam cache untuk sesi tersebut di memori dan di disk, sehingga eksekusi berikutnya dapat dimulai lebih cepat:

| Platform | Profil yang disimpan dalam cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (mengutamakan `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Hapus berkas tersebut untuk memaksa deteksi ulang — berguna setelah menambahkan GPU atau RAM tambahan. Chloros juga akan mendeteksi ulang secara otomatis jika cache ditulis oleh versi lama yang tidak kompatibel.

***

## Strategi Komputasi

Chloros memilih salah satu dari tiga strategi komputasi berdasarkan perangkat keras Anda:

| Strategi | Dipilih Saat | Pekerja | Pelaksana | Jalur |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA yang melaporkan**VRAM 12GB+**(pada memori terpadu Jetson, juga memerlukan total RAM bersama 12GB+) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimal 2 —**dibatasi hingga 2 pada Jetson** | Proses pekerja (konteks CUDA terpisah) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA dengan**2–12GB VRAM**| 3 (tumpang tindih I/O; akses GPU diserialisasikan oleh semafor).**1 (sekuensial) pada Jetson dengan RAM di bawah 12 GB** | Proses pekerja (konteks CUDA terpisah); sekuensial dalam proses pada Jetson dengan RAM rendah | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Tanpa GPU CUDA, atau VRAM di bawah 2GB | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Contoh perhitungan rumus pekerja `GPU_PARALLEL`: VRAM 12 GB → 2 pekerja, 16 GB → 3 pekerja, 20 GB+ → 4 pekerja. Setiap worker dialokasikan 3,5GB, sesuai kebutuhan yang diukur untuk tugas Texture Aware, dalam batas 90% dari sisa VRAM setelah cadangan 0,5GB.

{% hint style="info" %}
**Strategi GPU memerlukan PyTorch yang dapat menggunakan CUDA.** Mesin yang PyTorch-nya tidak memiliki perangkat CUDA yang dapat digunakan akan mendapatkan `CPU_PARALLEL`, apa pun GPU-nya. Paket Jetson 1.3.0 menggunakan PyTorch versi CUDA dari JetPack 6, sehingga Jetson memenuhi syarat setelah pustaka CUDA dan cuDNN dari JetPack diinstal. Jetson dengan RAM kurang dari 12GB selalu memproses satu gambar pada satu waktu, dalam proses (tercatat sebagai `GPU_SINGLE`).
{% endhint %}

Strategi GPU menjalankan kumpulan proses pekerja, masing-masing merupakan proses OS terpisah dengan konteks CUDA-nya sendiri, yang dimulai oleh Chloros dengan meluncurkan kembali program backend yang terinstal miliknya (bukan melalui `multiprocessing` Python, yang proses awal pekerjaannya tidak berfungsi di dalam aplikasi yang dikemas). Strategi CPU menggunakan kumpulan utas (`ThreadPoolExecutor` dari Python). Chloros tidak menggunakan kerangka kerja terdistribusi pihak ketiga apa pun (seperti Ray).

### Jenis Pipeline

* **`fused_gpu`** — Jalur pemrosesan GPU penuh. Operasi debayer, koreksi, dan pengindeksan dijalankan di GPU dalam satu tahap terpadu. Memiliki throughput tertinggi, namun membutuhkan VRAM paling banyak.
* **`tiled_gpu`** — Jalur GPU yang efisien dalam penggunaan memori. Memproses gambar dalam bentuk ubin agar sesuai dengan memori GPU yang terbatas. Throughput lebih rendah tetapi berfungsi pada perangkat dengan keterbatasan memori.
* **`cpu_fallback`** — Pemrosesan hanya menggunakan CPU dengan paralelisme multi-thread. Digunakan saat tidak ada GPU NVIDIA yang tersedia, dan sebagai opsi cadangan terakhir jika kedua jalur GPU gagal.

Rantai cadangan saat runtime selalu `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Penggantian Strategi Secara Manual

Atur variabel lingkungan `CHLOROS_STRATEGY` untuk memaksa penggunaan strategi tertentu — jalan keluar khusus bagi ahli ketika deteksi otomatis memilih sesuatu yang tidak sesuai dengan situasi Anda (misalnya, menjaga GPU tetap bebas untuk pekerjaan lain):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

Variabel ini dicocokkan tanpa membedakan huruf besar-kecil; apa pun yang bukan salah satu dari tiga nama tersebut akan diabaikan dan deteksi otomatis akan berjalan seperti biasa. Saat pengaturan ini diterapkan, Chloros tetap menentukan jumlah pekerja untuk Anda:

| Pengaturan | Jumlah pekerja yang digunakan |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Penggantian dengan `GPU_SINGLE` atau `GPU_PARALLEL` pada mesin yang tidak memiliki perangkat CUDA yang dapat digunakan akan ditolak dengan munculnya baris `[STRATEGY]` dalam log, dan eksekusi tersebut menggunakan `CPU_PARALLEL`.

Disarankan untuk mengaturnya per-perintah daripada secara permanen, sehingga eksekusi normal tetap dapat beradaptasi secara otomatis.

***

## Perilaku Khusus Platform

| Platform | Strategi | Pekerja | Pipa | Catatan |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekuensial) | Mode hemat memori, satu gambar per kali |
| **Jetson Orin NX 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sekuensial) | RAM bersama di bawah 12GB memaksa pemrosesan sekuensial |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (bersamaan) | Perangkat tepi yang direkomendasikan — Jetson dibatasi hingga 2 pekerja |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (bersamaan) | Kinerja tepi maksimum (juga dibatasi oleh Jetson hingga 2 pekerja) |
| **Desktop dengan GPU 8GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 pekerja saling tumpang tindih dalam I/O sementara semafor menserialisasikan akses GPU |
| **Desktop dengan GPU 12GB+** | `GPU_PARALLEL` | 2-4 | `fused_gpu` (bersamaan) | Kinerja desktop optimal: 12GB → 2 pekerja, 16GB → 3, 20GB+ → 4 |
| **Sistem berbasis CPU saja** | `CPU_PARALLEL` | inti fisik − 1 (min. 2) | `cpu_fallback` | Tidak memerlukan GPU, menggunakan thread pool |

{% hint style="info" %}
**Memori terpadu Jetson**: Perangkat Jetson berbagi memori GPU dan CPU. Sebuah Jetson Orin NX 16GB melaporkan ~15,3GB VRAM, namun itu adalah RAM fisik yang sama yang digunakan oleh sistem operasi dan proses CPU. Itulah sebabnya Jetson 16GB+ memenuhi syarat untuk `GPU_PARALLEL` seperti GPU desktop 12GB+, namun dibatasi hingga 2 pekerja — GPU, proses pekerja, dan konteks CUDA per pekerja semuanya menggunakan kolam bersama yang sama.
{% endhint %}

### Anggaran GPU berdasarkan VRAM (GPU diskrit)

Pada host x86_64 dengan GPU NVIDIA diskrit, VRAM yang terdeteksi juga menentukan seberapa besar kapasitas pemrosesan kartu yang dapat digunakan serta seberapa besar ukuran batch yang dapat ditangani:

| VRAM yang Terdeteksi | Batas atas anggaran GPU | Pengali ukuran batch |
| --- | --- | --- |
| **8GB+** | 90% | ×2,0 |
| **6–8GB** | 85% | ×1,75 |
| **3,5–6GB** | 80% | ×1,5 |
| **2–3,5GB** | 75% | ×1,25 |
| **Di bawah 2GB** | 70% | ×1,0 |

GPU diskrit hanya mengalokasikan 0,5GB untuk sistem, karena tidak berbagi RAM sistem. Profil Jetson mengalokasikan jauh lebih banyak dan batasnya lebih rendah — lihat [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Alokasi Memori GPU Dinamis

Chloros menggunakan [pipa pemrosesan 4-thread](processing-pipeline.md):

* **Thread 1** (Deteksi) — Pemuatan gambar, penguraian EXIF, deteksi target
* **Thread 2** (Kalibrasi) — Perhitungan kalibrasi reflektansi
* **Thread 3** (Pemrosesan) — Debayer GPU, koreksi vignette, perhitungan indeks
* **Thread 4** (Ekspor) — Penulisan file, penyisipan metadata

Thread 1, 2, dan 4 merupakan pengguna GPU yang ringan; Thread 3 adalah yang paling memakan sumber daya. Saat thread-thread pada tahap awal selesai, alokasi memori GPU mereka **didistribusikan ulang ke thread-thread aktif yang tersisa**, sehingga Thread 3 mendapatkan memori yang semakin banyak seiring berjalannya proses.

### Tahapan Alokasi

| Tahap | Thread Aktif | Distribusi Memori GPU |
| --- | --- | --- |
| **Awal** | 1, 2, 3, 4 | Dibagi ke semua thread, sebagian besar dialokasikan ke Thread 3 |
| **Awal-Tengah** | 2, 3, 4 | Bagian Thread 1 didistribusikan ulang |
| **Tengah-Akhir** | 3, 4 | Bagian Thread 1 dan 2 dialihkan ke Thread 3 dan 4 |
| **Akhir** | 3 atau 4 | Thread aktif terakhir mendapatkan alokasi maksimumnya |

Ada dua aturan yang mengatur angka-angka ini:

* Sebuah thread yang merupakan **satu-satunya** yang aktif akan diberikan alokasi maksimum sesuai profilnya.
* Ketika lebih dari satu tugas GPU *berat* aktif, alokasi dasar setiap tugas berat tersebut dibagi di antara mereka (tidak pernah di bawah batas minimum yang dikonfigurasi).

Nilai yang sebenarnya digunakan saat runtime adalah nilai **terendah** antara alokasi profil platform dan rekomendasi real-time dari pemantau memori GPU, sehingga kartu yang sibuk selalu diutamakan daripada profil yang terlalu optimis.***

## Pemrosesan yang Mempertimbangkan Tekstur

Debayer Texture Aware (**hanya Chloros+** — `--debayer texture-aware`) menjalankan model penghilang noise AI/ML yang membutuhkan sekitar 1,75 GB VRAM dalam FP16 per salinan, sehingga menggunakan memori GPU jauh lebih banyak daripada metode Standar:

* Sistem dengan **VRAM di bawah 7 GB**memproses Texture Aware dalam**loop sinkron, satu gambar pada satu waktu** — beberapa salinan model tidak dapat dimuat, dan kumpulan pekerja hanya akan menambah persaingan
* Sistem dengan **VRAM 7 GB ke atas** dapat memproses Texture Aware secara bersamaan, meskipun dengan jumlah pekerja yang lebih sedikit dibandingkan dengan metode Standard
* Pada **Jetson**, Texture Aware selalu diikat ke satu pekerja
* Jika **tidak ada GPU CUDA yang dapat digunakan**, proses ini menggunakan debayer Standar, dan backend mencatat satu baris log `[DEBAYER] … needs a CUDA GPU …` yang menjelaskan alasannya
* Pada perangkat Jetson berdaya rendah (Nano, Orin Nano), antarmuka baris perintah (CLI) menerapkan batasan frekuensi GPU saat Anda meminta Texture Aware — lihat [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Manajemen Termal (Jetson)

Perangkat Jetson memiliki batasan termal, terutama pada pemasangan tertutup atau di udara. Chloros memantau sensor suhu onboard Jetson dan menyesuaikan ukuran batch secara otomatis:

| Suhu | Tanggapan |
| --- | --- |
| **< 70°C** | Operasi normal — kecepatan penuh |
| **70°C** (Peringatan) | Ukuran batch dikurangi secara bertahap (100% → 50% antara 70°C dan 80°C) |
| **80°C** (Kritis) | Pembatasan agresif (50% → 0% antara 80°C dan 90°C) |
| **90°C** (Mati) | Menghentikan pemrosesan GPU sepenuhnya |

Pada sistem desktop dengan pendinginan yang memadai, pembatasan termal jarang terpicu.

***

## Penanganan Tekanan Memori

Chloros memantau memori GPU secara terus-menerus selama pemrosesan dan bereaksi pada tiga tingkatan.

**Penentuan ukuran batch.** Sebuah batch dimulai dengan 8 gambar dikalikan dengan faktor pengali platform dari tabel di atas. Chloros kemudian memeriksa VRAM yang tersedia, menyisihkan 20% darinya untuk overhead PyTorch sendiri, dan mengasumsikan sekitar 100MB memori GPU per gambar 12MP — ukuran batch adalah yang lebih kecil di antara batas yang ditentukan berdasarkan memori atau nilai dasar platform. Ukuran batch tidak pernah turun di bawah 1.

**Pengurangan preventif.**Di atas**85% pemanfaatan VRAM**, ukuran batch dikurangi sebelum terjadi kegagalan.**Pemangkasan alokasi per-thread.** Seiring meningkatnya pemanfaatan real-time, alokasi memori GPU setiap thread dikurangi: ×0,75 di atas 80% pemanfaatan, ×0,5 di atas 90%. Batas pemantauan adalah 70% (konservatif), 85% (batas operasi normal), dan 95% (risiko OOM).**Penundaan dan pemulihan OOM.** Jika kejadian kehabisan memori tetap terjadi:

* ukuran batch **dipotong setengah**, dan dipotong setengah lagi pada setiap kejadian OOM berturut-turut — setiap batch berikutnya yang berhasil akan mengurangi penalti tersebut satu langkah
* alokasi thread aktif dikurangi menjadi 70% dari nilai saat ini dan alokator beralih ke strategi konservatifnya, kemudian kembali melonggarkan setelah serangkaian alokasi yang berhasil
* di bawah tekanan berat, pipa alur beralih dari `fused_gpu` ke `tiled_gpu`, dan ke `cpu_fallback` sebagai upaya terakhir

**RAM Host (Jetson).** Sebelum pemrosesan, CLI memperkirakan memori host puncak berdasarkan jumlah gambar dan mode debayer Anda, serta memberikan peringatan jika RAM ditambah swap berbasis file kemungkinan tidak mencukupi, sambil menampilkan perintah tepat untuk menambahkan swap — lihat [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Pemantauan Adaptasi Komputasi

### Diagnostik Sistem

`chloros-cli selftest` adalah cara tercepat untuk memastikan apa yang dilihat oleh lapisan komputasi:

```bash
chloros-cli selftest
```

7 pemeriksaan ini mencakup versi, ketersediaan port, proses startup backend, `/api/test`, informasi sistem, keberadaan model denoiser, dan kesiapan CUDA + denoiser. Pemeriksaan ke-5 menampilkan baris hardware secara langsung:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

Pemeriksaan ke-7 menampilkan `CUDA: <bool>, Denoiser: <bool>` — keduanya harus benar agar Texture Aware dapat digunakan sama sekali.

### Log Backend

Strategi dan jumlah pekerja dipilih di dalam backend pada awal setiap eksekusi — tidak ada banner CLI yang menampilkannya. Ketika terjadi perilaku yang tidak terduga (jalur GPU yang beralih ke cadangan, OOM, atau denoiser yang tidak dapat dimuat), log backend untuk sesi tersebutlah yang menampilkannya:

| Platform | Lokasi log |
| --- | --- |
| **Linux / Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (satu berkas per peluncuran) |
| **Linux, backend yang dijalankan melalui CLI** | juga `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Kemajuan Real-Time

Selama proses berjalan, antarmuka baris perintah (CLI) menampilkan kemajuan real-time per-thread (Deteksi, Analisis, Pemrosesan, Ekspor) yang disalurkan melalui Server-Sent Events — indikator praktis untuk mengetahui apakah Thread 3 menjadi titik leher botol. Lihat [Pipa Pemrosesan](processing-pipeline.md).

***

## Langkah Selanjutnya

* [Pipa Pemrosesan](processing-pipeline.md) — Memahami arsitektur pipa 4-thread
* [Panduan NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Penerapan dan optimasi khusus Jetson
* [CLI: Baris Perintah](../CLI.md) — Panduan CLI

* [Referensi CLI](../reference/cli-reference.md) — Daftar perintah lengkap untuk versi 1.3.0

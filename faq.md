---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Pertanyaan Umum

<details>

<summary>Apakah saya dapat memproses gambar dari kamera yang bukan merek MAPIR menggunakan Chloros?</summary>

Tidak, Chloros hanya mendukung pemrosesan gambar kamera MAPIR. Silakan lihat daftar [model kamera yang didukung](supported-cameras.md) untuk informasi lebih lanjut. Kami memang menawarkan pemrosesan kamera lain di MAPIR Cloud, lihat daftar lengkapnya [di sini](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Apakah saya dapat mengkalibrasi gambar saya untuk reflektansi tanpa target kalibrasi?</summary>

Tidak. Tanpa gambar target kalibrasi yang diambil bersamaan dengan gambar non-target, Anda tidak akan dapat menghubungkan nilai piksel gambar dengan persentase reflektansi yang diketahui. Jika Anda juga tidak menyertakan log dari sensor cahaya MAPIR, spektrum cahaya sekitar tidak akan diukur, dan hasil reflektansi tidak akan akurat.

</details>

<details>

<summary>Bisakah saya mengedit gambar saya sebelum diproses di Chloros?</summary>

Tidak. Chloros mengasumsikan data masukan belum dimodifikasi. Jangan ubah nama file.

</details>

<details>

<summary>Apakah saya dapat mengatur kamera MAPIR dan Survey3 ke mode eksposur otomatis dan memproses gambar di Chloros?</summary>

Tidak. Kumpulan data gambar Survey3 harus memiliki eksposur tetap/terkunci, jadi tidak boleh ada kecepatan rana otomatis atau ISO otomatis. Semua gambar dari model kamera yang sama harus memiliki kecepatan rana dan ISO (eksposur) yang identik.

</details>

<details>

<summary>Apakah Chloros dapat memproses atau menganalisis gambar ortomosaik?</summary>

Tidak. Hanya gambar kamera MAPIR individual yang didukung, bukan gambar yang digabungkan seperti peta ortomosaik.

</details>

<details>

<summary>Bagaimana cara mempercepat langkah deteksi target pada Chloros?</summary>

Di tabel penjelajah file, memilih gambar target terlebih dahulu di kolom kanan akan memberi tahu Chloros untuk hanya mencari target kalibrasi di gambar-gambar tersebut, sehingga sangat mempercepat pemrosesan.

</details>

<details>

<summary>Jika saya akan mengunggah gambar saya ke <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> apakah saya harus memprosesnya di Chloros sebelum mengunggahnya?</summary>

Jika Anda berencana mengunggah ke platform pemrosesan online kami [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), jangan mengedit gambar sebelum mengunggahnya. Cloud akan melakukan semua pemrosesan yang sama dan lebih banyak lagi.

</details>

<details>

<summary>Apakah MAPIR akan mendukung fitur X? Saya sangat berharap MAPIR menawarkan fitur X.</summary>

Kami selalu tertarik menerima masukan mengenai produk kami. Jika Anda menemukan masalah dengan produk kami, atau memiliki saran tentang cara kami dapat meningkatkan produk kami, silakan [HUBUNGI KAMI](https://www.mapir.camera/community/contact) untuk berbagi pemikiran Anda. Sebagian besar penelitian dan pengembangan (R&amp;D) kami didasarkan pada mendengarkan kebutuhan terbesar pelanggan kami.

</details>

<details>

<summary>Apakah Chloros tersedia untuk Linux?</summary>

Ya! Chloros 1.1.0 mendukung Linux amd64 (x86_64) dan arm64 (NVIDIA Jetson JetPack 6) melalui paket `.deb`. CLI dan Python SDK didukung sepenuhnya pada Linux. Tidak ada antarmuka grafis (GUI) untuk Linux — semua interaksi dilakukan melalui [CLI](CLI.md) atau [Python SDK](api-python-sdk.md). Lihat [Ikhtisar Linux](linux/linux-overview.md) untuk detailnya.

</details>

<details>

<summary>Apakah saya dapat menjalankan Chloros di NVIDIA Jetson?</summary>

Ya! Chloros 1.1.0 mendukung platform NVIDIA Jetson termasuk Jetson Nano, Orin Nano, Orin NX, dan AGX Orin yang menjalankan JetPack 6. Chloros secara otomatis mendeteksi model Jetson Anda dan mengoptimalkan strategi pemrosesannya. Lihat [Panduan NVIDIA Jetson](linux/nvidia-jetson-guide.md) untuk petunjuk pengaturan dan penerapan.

</details>

<details>

<summary>Apakah Chloros secara otomatis mengoptimalkan untuk perangkat keras saya?</summary>

Ya! Chloros 1.1.0 menyertakan [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) yang secara otomatis mendeteksi CPU, GPU, RAM, dan (pada Jetson) sensor termal Anda. Kemudian, fitur ini memilih strategi pemrosesan yang optimal — mulai dari `GPU_PARALLEL` pada sistem dengan memori tinggi hingga `GPU_SINGLE` pada perangkat dengan keterbatasan, serta `CPU_PARALLEL` pada sistem tanpa GPU NVIDIA. Tidak diperlukan konfigurasi manual.

</details>

<details>

<summary>Apa itu pipa pemrosesan 4-thread?</summary>

Chloros 1.1.0 menggunakan arsitektur pipelined 4-thread untuk pengguna Chloros+: Thread 1 (Deteksi) memuat gambar dan mendeteksi target kalibrasi, Thread 2 (Kalibrasi) menghitung kalibrasi reflektansi, Thread 3 (Pemrosesan) melakukan debayering yang dipercepat GPU dan perhitungan indeks, serta Thread 4 (Ekspor) menulis file output. Beberapa gambar dapat berada di thread yang berbeda secara bersamaan untuk throughput maksimum. Lihat [Pipa Pemrosesan](processing-architecture/processing-pipeline.md) untuk detailnya.

</details>

<details>

<summary>Bagaimana cara menjalankan diagnostik pada instalasi Chloros saya?</summary>

Gunakan perintah `selftest` untuk menjalankan 7 diagnostik sistem, termasuk pemeriksaan versi, ketersediaan port, startup backend, konektivitas API, informasi sistem, model denoiser, dan ketersediaan CUDA:

```bash
chloros-cli selftest
```

Ini sangat berguna pada sistem Linux/Jetson untuk memverifikasi pengaturan GPU dan CUDA.

</details>

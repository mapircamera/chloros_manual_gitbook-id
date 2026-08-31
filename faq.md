---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Pertanyaan Umum

<details>

<summary>Apakah saya dapat memproses gambar dari kamera yang bukan merek MAPIR menggunakan Chloros?</summary>

Tidak, Chloros hanya mendukung pemrosesan gambar dari kamera MAPIR — yaitu seri Survey3 dan LATTICE. Silakan lihat daftar [model kamera yang didukung](supported-cameras.md) untuk informasi lebih lanjut. Kami memang menawarkan pemrosesan kamera lain di MAPIR Cloud, lihat daftar lengkapnya [di sini](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Apakah Chloros mendukung kamera LATTICE?</summary>

Ya. Chloros 1.2.0 mendukung modul kamera LATTICE M3C dan M3M secara end-to-end: **kontrol langsung**— temukan, sambungkan, pratinjau, dan rekam dari tab &quot;Kamera&quot; di antarmuka pengguna (GUI), `chloros-cli lattice`, atau Python SDK, termasuk susunan multi-kamera yang disinkronkan dengan sinkronisasi waktu PTP — serta**pemrosesan radiometrik lengkap** dari hasil perekaman (mentah → debayered → radian → reflektansi → indeks). Lihat [Kamera yang Didukung](supported-cameras.md) dan [panduan LATTICE](lattice/README.md).

</details>

<details>

<summary>Bisakah saya mengkalibrasi gambar saya untuk reflektansi tanpa target kalibrasi?</summary>

**Survey3:** Tidak. Tanpa gambar target kalibrasi yang diambil bersamaan dengan pengambilan gambar non-target, Anda tidak akan dapat mengaitkan nilai piksel gambar dengan persentase reflektansi yang diketahui. Jika Anda juga tidak menyertakan log dari sensor cahaya MAPIR, maka spektrum cahaya sekitar tidak akan terukur, dan hasil reflektansi tidak akan akurat.**LATTICE:** Ya. Reflektansi dapat dirujuk ke irradiansi turun yang diukur oleh sensor cahaya DAQ, bukan panel (ρ = π·L/E). Ketika target dalam bingkai yang lulus QA *memang* ada, target tersebut secara default menjadi referensi absolut (`--reflectance-source auto`). Satu pengecualian: &quot;Reflektansi F988 dikalibrasi menggunakan panel reflektansi dalam adegan: pita tersebut berada di luar rentang kalibrasi sensor cahaya DAQ, sehingga Chloros menerapkan penangkapan panel terbaru Anda dan mempertahankannya di antara pengamatan panel.&quot; Lihat [Target Kalibrasi](calibration-targets.md).

</details>

<details>

<summary>Apakah saya memerlukan sensor cahaya DAQ?</summary>

Tidak untuk radian: Ekspor radian LATTICE berasal dari kalibrasi radiometrik pabrik masing-masing kamera dan tidak memerlukan sensor DAQ maupun target. Untuk **reflektansi**, Anda memerlukan referensi untuk cahaya sekitar — baik pengukuran cahaya yang turun dari sensor cahaya DAQ maupun target kalibrasi dalam bingkai. Sensor DAQ memungkinkan Anda menghasilkan reflektansi yang dikalibrasi**tanpa menempatkan panel apa pun di dalam adegan**. File `.daq` yang direkam akan dicocokkan secara otomatis dengan citra Anda berdasarkan cap waktu. Lihat [Target Kalibrasi](calibration-targets.md) dan [Referensi CLI](reference/cli-reference.md).

</details>

<details>

<summary>Apakah saya dapat menggunakan Chloros dengan asisten AI (Claude, ChatGPT, dll.)?</summary>

Ya — panduan ini serta berkas CLI/SDK dirancang untuk itu:

* Indeks lengkap panduan ini disajikan di `https://mapir.gitbook.io/chloros/llms.txt` agar asisten AI dapat menemukan setiap halaman.
* Kode markdown mentah setiap halaman tersedia di halaman huruf kecilnya, yaitu URL dengan tambahan `.md` (misalnya `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referensi CLI](reference/cli-reference.md) dan [Referensi SDK](reference/sdk-reference.md) ditulis untuk digunakan oleh LLM: bendera yang tepat, nilai default, semantik keluar, dan perintah yang dapat disalin-tempel.

Lihat [Asisten AI](ai-assistants.md) untuk mengetahui cara mengarahkan asisten Anda ke Chloros.

</details>

<details>

<summary>Ke mana file keluaran yang telah diproses akan disimpan?</summary>

Hasil diproses disimpan di bawah folder proyek, dikelompokkan berdasarkan kamera dan kemudian berdasarkan format file:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **folder-kamera** — `LATT-<sensor>-<lens>-F<filter>` untuk LATTICE, `<model>_<filter>` (misalnya `Survey3N_RGN`) untuk Survey3
* **folder-format** — `tiff16`, `tiff8`, `png8`, `jpg8`, atau `tiff32`
* **folder produk** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (selalu berada di bawah `tiff32`), `<INDEX>_Index_Images/`**File yang diekspor tetap menggunakan nama file aslinya — folderlah yang mengidentifikasi produk, bukan sufiks nama file.**Dengan CLI, folder proyek dibuat di samping folder input kecuali Anda memberikan argumen `-o`. Perhatikan bahwa eksekusi `chloros-cli process` yang meminta produk tetapi tidak menulis apa pun akan menampilkan pesan `Processing finished but wrote no image products.` dan**keluar dengan nilai non-nol**, sehingga skrip dapat mendeteksinya. Lihat [Format Gambar Keluaran](output-image-formats.md) dan [Referensi CLI](reference/cli-reference.md).

</details>

<details>

<summary>Apakah saya dapat mengedit gambar saya sebelum diproses di Chloros?</summary>

Tidak. Chloros mengasumsikan bahwa data masukan belum dimodifikasi. Jangan ubah nama file.

</details>

<details>

<summary>Apakah saya dapat mengatur kamera MAPIR dan Survey3 ke mode eksposur otomatis dan memproses gambar di Chloros?</summary>

Tidak. Kumpulan data gambar Survey3 harus memiliki eksposur yang tetap/terkunci, sehingga tidak boleh menggunakan kecepatan rana otomatis atau ISO otomatis. Semua gambar dari model kamera yang sama harus memiliki kecepatan rana dan ISO (eksposur) yang identik.

Kamera LATTICE tidak memiliki batasan ini: Chloros mengontrol eksposurnya secara langsung (Smart AE), dan setiap pengambilan gambar mencatat eksposur dan gain yang sebenarnya digunakan, yang diperhitungkan oleh alur kerja radiometrik.

</details>

<details>

<summary>Apakah Chloros dapat memproses atau menganalisis gambar ortomosaik?</summary>

Tidak. Hanya gambar kamera MAPIR individual yang didukung, bukan gambar yang disatukan seperti peta ortomosaik.

</details>

<details>

<summary>Bagaimana cara mempercepat langkah deteksi target di Chloros?</summary>

Di tabel penjelajah file, dengan memilih terlebih dahulu gambar target di kolom kanan, Chloros akan diperintahkan untuk hanya mencari target kalibrasi di gambar-gambar tersebut, sehingga prosesnya menjadi jauh lebih cepat.

</details>

<details>

<summary>Jika saya akan mengunggah gambar saya ke <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> apakah saya harus memprosesnya di Chloros terlebih dahulu sebelum mengunggahnya?</summary>

Jika Anda berencana mengunggah ke platform pemrosesan online kami [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), jangan mengedit gambar sebelum mengunggahnya. Cloud akan melakukan semua pemrosesan yang sama dan bahkan lebih.

</details>

<details>

<summary>Apakah MAPIR akan mendukung fitur X di masa mendatang? Saya sangat berharap MAPIR menyediakan fitur X.</summary>

Kami selalu tertarik menerima masukan mengenai produk kami. Jika Anda menemukan masalah dengan produk kami, atau memiliki saran tentang cara kami dapat meningkatkan produk kami, silakan [HUBUNGI KAMI](https://www.mapir.camera/community/contact) untuk berbagi pemikiran Anda. Sebagian besar kegiatan R&amp;D kami dipandu oleh mendengarkan kebutuhan terbesar pelanggan kami.

</details>

<details>

<summary>Apakah Chloros tersedia untuk Linux?</summary>

Ya! Chloros 1.2.0 mendukung Linux amd64 (x86_64) dan arm64 (NVIDIA Jetson JetPack 6) melalui paket `.deb`. CLI dan Python SDK didukung sepenuhnya pada Linux, termasuk kamera LATTICE langsung dan kontrol sensor DAQ. Tidak ada antarmuka pengguna grafis (GUI) untuk Linux — semua interaksi dilakukan melalui [CLI](CLI.md) atau [Python SDK](api-python-sdk.md). Lihat [Linux Ikhtisar](linux/linux-overview.md) untuk detailnya.

</details>

<details>

<summary>Apakah saya dapat menjalankan Chloros di NVIDIA Jetson?</summary>

Ya! Chloros mendukung platform NVIDIA Jetson, termasuk Jetson Nano, Orin Nano, Orin NX, dan AGX Orin yang menjalankan JetPack 6. Chloros secara otomatis mendeteksi model Jetson Anda dan mengoptimalkan strategi pemrosesannya. Lihat [Panduan NVIDIA Jetson](linux/nvidia-jetson-guide.md) untuk petunjuk pengaturan dan penerapan.

</details>

<details>

<summary>Apakah Chloros secara otomatis mengoptimalkan perangkat keras saya?</summary>

Ya! Chloros dilengkapi dengan [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) yang secara otomatis mendeteksi CPU, GPU, RAM, dan (pada Jetson) sensor suhu Anda. Kemudian, fitur ini memilih strategi pemrosesan yang optimal — mulai dari `GPU_PARALLEL` pada sistem dengan memori tinggi hingga `GPU_SINGLE` pada perangkat dengan keterbatasan, hingga `CPU_PARALLEL` pada sistem tanpa GPU NVIDIA. Tidak diperlukan konfigurasi manual.

</details>

<details>

<summary>Apa itu pipa pemrosesan 4-thread?</summary>

Chloros menggunakan arsitektur pipelined 4-thread untuk pengguna Chloros+: Thread 1 (Deteksi) memuat gambar dan mendeteksi target kalibrasi, Thread 2 (Kalibrasi) menghitung kalibrasi reflektansi, Thread 3 (Pemrosesan) melakukan debayering yang dipercepat GPU dan perhitungan indeks, serta Thread 4 (Ekspor) menulis file keluaran. Beberapa gambar dapat berada di thread yang berbeda secara bersamaan untuk throughput maksimum. Lihat [Pipa Pemrosesan](processing-architecture/processing-pipeline.md) untuk detailnya.

</details>

<details>

<summary>Bagaimana cara menjalankan diagnostik pada instalasi Chloros saya?</summary>

Gunakan perintah `selftest` untuk menjalankan uji coba 7 langkah: versi, ketersediaan port, startup backend, konektivitas API (`/api/test`), informasi sistem (`/api/system-info` — GPU/CUDA/PyTorch), keberadaan model denoiser, dan kesiapan CUDA + denoiser:

```bash
chloros-cli selftest
```

Hal ini sangat berguna pada sistem Linux/Jetson untuk memverifikasi pengaturan GPU dan CUDA.

</details>

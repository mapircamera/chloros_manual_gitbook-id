---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Pertanyaan yang Sering Diajukan

<details>

<summary>Apakah saya dapat memproses gambar dari kamera yang bukan merek MAPIR menggunakan Chloros?</summary>

Tidak, Chloros hanya mendukung pemrosesan gambar dari kamera MAPIR — seri Survey3 dan LATTICE. Silakan lihat daftar [model kamera yang didukung](supported-cameras.md) untuk informasi lebih lanjut. Kami memang menawarkan pemrosesan kamera lain di MAPIR Cloud, lihat daftar lengkapnya [di sini](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Apakah Chloros mendukung kamera LATTICE?</summary>

Ya. Chloros 1.3.0 mendukung modul kamera LATTICE M3C dan M3M secara menyeluruh: **kontrol langsung**— temukan, sambungkan, pratinjau, dan ambil gambar dari tab "Kamera" di antarmuka pengguna (GUI), `chloros-cli lattice`, atau SDK SDK, termasuk susunan multi-kamera yang disinkronkan dengan sinkronisasi waktu PTP — serta**pemrosesan radiometrik lengkap** dari hasil pengambilan gambar (mentah → debayered → radian → reflektansi → indeks). Lihat [Kamera yang Didukung](supported-cameras.md) dan [panduan LATTICE](lattice/README.md).

</details>

<details>

<summary>Bisakah saya mengkalibrasi gambar saya untuk reflektansi tanpa target kalibrasi?</summary>

**Survey3:** Tidak. Tanpa gambar target kalibrasi yang diambil bersamaan dengan gambar non-target, Anda tidak akan dapat mengaitkan nilai piksel gambar dengan persentase reflektansi yang diketahui. Jika Anda juga tidak menyertakan log dari sensor cahaya MAPIR, maka spektrum cahaya sekitar tidak akan terukur, dan hasil reflektansi tidak akan akurat.

**LATTICE:** Ya. Reflektansi dapat dirujuk ke irradiansi turun yang diukur oleh sensor cahaya DAQ, bukan panel (ρ = π·L/E). Ketika target dalam bingkai yang lulus QA *memang* ada, target tersebut secara default menjadi referensi absolut (`--reflectance-source auto`), kecuali jika target tersebut lulus dengan bendera, di mana target dan rantai DAQ bersaing berdasarkan skor yang diukur. Satu pengecualian: “Reflektansi F988 dikalibrasi menggunakan panel reflektansi dalam adegan: rentang band tersebut berada di luar rentang kalibrasi sensor cahaya DAQ, sehingga Chloros menerapkan penangkapan panel terbaru Anda dan mempertahankannya di antara pengamatan panel.” Lihat [Target Kalibrasi](calibration-targets.md).

</details>

<details>

<summary>Apakah saya memerlukan sensor cahaya DAQ?</summary>

Tidak untuk radian: Ekspor radian LATTICE berasal dari kalibrasi radiometrik pabrik masing-masing kamera dan tidak memerlukan sensor DAQ maupun target. Untuk **reflektansi**, Anda memerlukan referensi untuk cahaya ambient — baik pengukuran cahaya yang turun dari sensor cahaya DAQ maupun target kalibrasi dalam bingkai. Sensor DAQ memungkinkan Anda menghasilkan reflektansi yang dikalibrasi**tanpa menempatkan panel apa pun di dalam adegan**. File `.daq` yang direkam secara otomatis dicocokkan dengan citra Anda berdasarkan cap waktu. Lihat [Target Kalibrasi](calibration-targets.md) dan [Referensi CLI](reference/cli-reference.md).

</details>

<details>

<summary>Bisakah saya menggunakChloros dengan asisten AI (Claude, ChatGPT, dll.)?</summary>

Ya — panduan ini serta CLI/SDK dirancang untuk itu:

* Indeks lengkap panduan ini tersedia di `https://mapir.gitbook.io/chloros/llms.txt` agar asisten AI dapat menemukan setiap halaman.
* Markdown mentah setiap halaman tersedia di URL halaman huruf kecil dengan tambahan `.md` (misalnya `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* [Referensi CLI](reference/cli-reference.md) dan [Referensi SDK](reference/sdk-reference.md) ditulis agar dapat digunakan oleh LLM: bendera yang tepat, nilai default, semantik keluar, dan perintah yang dapat disalin dan ditempelkan.

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
* **folder produk** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (selalu berada di bawah `tiff32`), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — Khusus multispektral LATTICE: masker per piksel di samping folder produk, lihat di bawah**File yang diekspor mempertahankan nama file aslinya — folderlah yang mengidentifikasi produk, bukan sufiks nama file.**Dengan CLI, folder proyek dibuat di bawah folder proyek default Anda (`Documents/MAPIR/Chloros Projects` kecuali jika Anda mengubahnya) kecuali jika Anda memasukkan `-o`; `-o` relatif diambil berdasarkan folder tempat Anda menjalankan perintah tersebut. Perhatikan bahwa eksekusi `chloros-cli process` yang meminta produk tetapi tidak menulis apa pun akan menampilkan `Processing finished but wrote no image products.` dan**keluar dengan nilai non-nol**, sehingga skrip dapat mendeteksinya. Lihat [Format Gambar Keluaran](output-image-formats.md) dan [Referensi CLI](reference/cli-reference.md).

</details>

<details>

<summary>Apa itu folder clamp_masks dan berkas *_clampmask.tif?</summary>

Baru di versi 1.3.0. Produk radian dan reflektansi dari kamera multispektral LATTICE M3C menjalankan pemisahan band 3×3, dan jika menghasilkan nilai negatif pada suatu band, Chloros akan memotong nilai tersebut menjadi nol. Produk yang memiliki setidaknya satu piksel yang dipotong akan mendapatkan masker `uint8` satu-band yang menandai piksel dan band mana yang dipotong (bit 0 = kanal R produk, bit 1 = G, bit 2 = B). Topeng disimpan dalam folder `clamp_masks/` **di samping** folder produk — misalnya `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — dan tag XMP produk `Chloros:UnmixClampMask` mencantumkan nama maskernya (atau mencantumkan `none`).

Mask-mask ini bukanlah gambar dari adegan: proses impor Chloros sendiri melewatkannya, dan karena mask-mask ini tidak pernah berada di dalam folder produk, Anda dapat menyerahkan folder produk ke Pix4D atau Metashape apa adanya. Lihat [Mask Clamp](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Mengapa Chloros menampilkan pesan "Folder proyek terlalu dalam untuk komputer ini"?</summary>

Ini hanya berlaku untuk Windows. Windows secara default menonaktifkan jalur panjang, sehingga jalur file tidak boleh lebih panjang dari 260 karakter. Produk Chloros berada hingga sekitar 130 karakter di bawah folder proyek, sehingga Chloros menolak folder proyek yang lebih dalam sebelum pemrosesan — alih-alih berjalan dan tidak menulis apa pun, seperti yang mungkin terjadi sebelum versi 1.3.0. Pesan tersebut menunjukkan folder proyek terpanjang yang masih sesuai.

Perbaiki dengan salah satu cara berikut:

* Gunakan folder proyek yang lebih pendek (**Working Directory** di Pengaturan Proyek, atau `-o` / `chloros-cli set-project-folder` untuk antarmuka baris perintah/CLI).
* Aktifkan jalur panjang Windows dan mulai ulang Chloros. Opsi **Aktifkan jalur panjang Windows** pada penginstal Chloros (dicentang secara default) melakukan hal ini; jalankan kembali penginstal jika Anda telah menghapus centangnya.

Lihat [Memulai Pemrosesan](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Apakah saya dapat mengedit gambar saya sebelum diproses di Chloros?</summary>

Tidak. Chloros mengasumsikan bahwa data masukan belum dimodifikasi. Jangan ubah nama file.

</details>

<details>

<summary>Bisakah saya mengatur kamera MAPIR Survey3 ke mode eksposur otomatis dan memproses gambar di Chloros?</summary>

Tidak. Kumpulan data gambar Survey3 harus memiliki eksposur yang tetap/terkunci, jadi tidak boleh ada kecepatan rana otomatis atau ISO otomatis. Semua gambar dari model kamera yang sama harus memiliki kecepatan rana dan ISO (eksposur) yang identik.

Kamera LATTICE tidak memiliki batasan ini: Chloros mengendalikan eksposurnya secara langsung (Smart AE), dan setiap pengambilan gambar mencatat eksposur dan gain yang sebenarnya digunakan, yang diperhitungkan oleh pipeline radiometrik.

</details>

<details>

<summary>Apakah Chlorosdapat memproses atau menganalisis gambar ortomosaik?</summary>

Tidak. Hanya gambar kamera MAPIR individual yang didukung, bukan gambar yang disatukan seperti peta ortomosaik.

</details>

<details>

<summary>Bagaimana cara mempercepat langkah deteksi target padChloros?</summary>

Di tabel penjelajah file, dengan memilih terlebih dahulu gambar target di kolom kanan, Chloros akan hanya mencari target kalibrasi di gambar-gambar tersebut, sehingga mempercepat proses secara signifikan.

</details>

<details>

<summary>Jika saya akan mengunggah gambar saya ke <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> apakah saya harus memprosesnya di Chloros terlebih dahulu sebelum mengunggahnya?</summary>

Jika Anda berencana mengunggah ke platform pemrosesan online kami [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), jangan mengedit gambar sebelum mengunggahnya. Cloud akan melakukan semua pemrosesan yang sama dan bahkan lebih.

</details>

<details>

<summary>Apakah MAPIR akan mendukung fitur X? Saya sangat berharap MAPIR menawarkan fitur X.</summary>

Kami selalu tertarik menerima masukan mengenai produk kami. Jika Anda menemukan masalah dengan produk kami, atau memiliki saran tentang cara kami dapat meningkatkan produk kami, silakan [HUBUNGI KAMI](https://www.mapir.camera/community/contact) untuk berbagi pemikiran Anda. Sebagian besar penelitian dan pengembangan (R&D) kami dipandu oleh mendengarkan kebutuhan terbesar pelanggan kami.

</details>

<details>

<summary>Apakah Chloros tersedia untuk Linux?</summary>

Ya! Chloros 1.3.0 mendukung Linux amd64 (x86_64) dan arm64 (NVIDIA Jetson JetPack 6) melalui paket `.deb`. CLI dan Python SDK didukung sepenuhnya di Linux, termasuk kontrol langsung kamera LATTICE dan sensor DAQ. Tidak ada antarmuka grafis (GUI) untuk Linux — semua interaksi dilakukan melalui [CLI](CLI.md) atau [Python SDK](api-python-sdk.md). Lihat [Ikhtisar Linux](linux/linux-overview.md) untuk detailnya.

</details>

<details>

<summary>Apakah saya dapat menjalankan Chloros di NVIDIA Jetson?</summary>

Ya! Chloros mendukung **keluarga Orin** NVIDIA Jetson — Orin Nano, Orin NX, dan AGX Orin — yang menjalankan JetPack 6. Model yang lebih lama (Jetson Nano, TX2, Xavier NX) tidak dapat menjalankan JetPack 6 dan tidak didukung oleh paket saat ini. Chloros secara otomatis mendeteksi model Jetson Anda dan mengoptimalkan strategi pemrosesannya. Paket Jetson menggunakan build CUDA PyTorch dari JetPack 6, sehingga pemrosesan berjalan di GPU Jetson. Lihat [Panduan NVIDIA Jetson](linux/nvidia-jetson-guide.md) untuk petunjuk pengaturan dan penerapan.

</details>

<details>

<summary>Apakah Chloros secara otomatis mengoptimalkan untuk perangkat keras saya?</summary>

Ya! Chloros dilengkapi dengan [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) yang secara otomatis mendeteksi CPU, GPU, RAM, dan (pada Jetson) sensor suhu Anda. Selanjutnya, sistem akan memilih strategi pemrosesan yang optimal — mulai dari `GPU_PARALLEL` pada GPU dengan memori besar hingga `GPU_SINGLE` pada GPU yang lebih kecil, hingga `CPU_PARALLEL` pada sistem tanpa GPU CUDA yang dapat digunakan oleh PyTorch. Tidak diperlukan konfigurasi manual.

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

Ini sangat berguna pada sistem Linux untuk memverifikasi pengaturan GPU dan CUDA. Pada Jetson, `CUDA: False` biasanya berarti pustaka CUDA dan cuDNN JetPack 6 tidak ada: instal pustaka tersebut dengan `sudo apt install nvidia-jetpack`.

</details>

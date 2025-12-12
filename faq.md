---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# FAQ

<details>

<summary>Apakah saya dapat memproses gambar dari kamera yang bukan merek MAPIR menggunakan Chloros?</summary>

Tidak, Chloros hanya mendukung pemrosesan gambar kamera MAPIR. Silakan lihat daftar [model kamera yang didukung](supported-cameras.md) untuk informasi lebih lanjut. Kami juga menawarkan pemrosesan kamera lain di MAPIR Cloud, lihat daftar lengkap [di sini](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Bisakah saya mengkalibrasi gambar untuk reflektansi tanpa target kalibrasi?</summary>

Tidak. Tanpa gambar target kalibrasi yang diambil sekitar waktu gambar non-target diambil, Anda tidak dapat menghubungkan nilai piksel gambar dengan persentase reflektansi yang diketahui. Jika Anda juga tidak menyertakan log dari sensor cahaya MAPIR, spektrum cahaya ambient tidak akan diukur, dan hasil reflektansi tidak akan akurat.

</details>

<details>

<summary>Bisakah saya mengedit gambar saya sebelum diproses di Chloros?</summary>

Tidak. Chloros mengasumsikan data input belum dimodifikasi. Jangan ubah nama file.

</details>

<details>

<summary>Bisakah saya mengatur kamera MAPIR Survey3 ke mode eksposur otomatis dan memproses gambar di Chloros?</summary>

Tidak. Set data gambar Survey3 harus memiliki eksposur tetap/terkunci, jadi tidak boleh menggunakan kecepatan rana otomatis atau ISO otomatis. Semua gambar dari model kamera yang sama harus memiliki kecepatan rana dan ISO (eksposur) yang identik.

</details>

<details>

<summary>Apakah Chloros dapat memproses atau menganalisis gambar ortomosaik?</summary>

Tidak. Hanya gambar kamera MAPIR individu yang didukung, bukan gambar yang disatukan seperti peta orthomosaik.

</details>

<details>

<summary>Bagaimana cara mempercepat langkah deteksi target pada Chloros?</summary>

Dengan memilih gambar target secara pra-seleksi di kolom kanan pada tabel penjelajah file, Chloros akan hanya mencari target kalibrasi di gambar-gambar tersebut, sehingga mempercepat proses pengolahan.

</details>

<details>

<summary>Jika saya akan mengunggah gambar ke <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud,</a> apakah saya perlu memprosesnya di Chloros sebelum mengunggah?</summary>

Jika Anda berencana mengunggah ke platform pemrosesan online kami [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), jangan mengedit gambar sebelum mengunggahnya. Cloud akan melakukan semua pemrosesan yang sama dan lebih banyak lagi.

</details>

<details>

<summary>Apakah MAPIR akan pernah mendukung fitur X? Saya benar-benar berharap MAPIR menawarkan fitur X.</summary>

Kami selalu tertarik untuk menerima umpan balik tentang produk kami. Jika Anda menemukan masalah dengan produk kami, atau memiliki saran tentang cara kami dapat meningkatkan produk kami, silakan [HUBUNGI KAMI](https://www.mapir.camera/community/contact) untuk berbagi pemikiran Anda. Sebagian besar R&amp;D kami dipandu oleh mendengarkan kebutuhan terbesar pelanggan kami.

</details>

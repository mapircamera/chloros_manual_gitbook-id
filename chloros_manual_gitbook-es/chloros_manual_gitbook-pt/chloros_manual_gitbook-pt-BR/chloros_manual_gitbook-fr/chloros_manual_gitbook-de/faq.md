---
description: Pertanyaan yang Sering Diajukan
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Pertanyaan Umum

<details>

<summary>Dapatkah saya memproses gambar dari kamera selain merek MAPIR dengan Chloros?</summary>

Tidak, Chloros hanya mendukung pemrosesan gambar kamera MAPIR. Silakan lihat daftar [model kamera yang didukung](supported-cameras.md) untuk informasi lebih lanjut. Kami menawarkan pemrosesan kamera lain di MAPIR Cloud, lihat daftar lengkap [Di Sini](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Dapatkah saya mengkalibrasi gambar untuk reflektansi tanpa target kalibrasi?</summary>

Tidak. Tanpa gambar target kalibrasi yang ditangkap saat gambar non target diambil, Anda tidak akan dapat menghubungkan nilai piksel gambar dengan persen reflektansi yang diketahui. Jika Anda juga tidak menyertakan log dari sensor cahaya MAPIR maka spektrum cahaya sekitar tidak akan terukur, dan hasil reflektansi tidak akan akurat.

</details>

<details>

<summary>Dapatkah saya mengedit gambar saya sebelum diproses di Chloros?</summary>

No.Chloros mengasumsikan data masukan belum diubah. Jangan mengubah nama file.

</details>

<details>

<summary>Dapatkah saya mengatur kamera MAPIR Survey3 saya ke eksposur otomatis dan memproses gambar di Chloros?</summary>

Tidak. Kumpulan data gambar Survey3 harus memiliki eksposur tetap/terkunci, jadi tidak ada kecepatan rana otomatis atau ISO otomatis. Semua gambar dari model kamera yang sama harus memiliki kecepatan rana dan ISO (eksposur) yang sama.

</details>

<details>

<summary>Dapatkah Kloro memproses atau menganalisis gambar ortomosaik?</summary>

Tidak. Hanya gambar kamera MAPIR individual yang didukung, bukan gambar gabungan seperti peta ortomosaik.

</details>

<details>

<summary>Bagaimana cara mempercepat langkah deteksi target Kloros?</summary>

Dalam tabel browser file, memilih gambar target terlebih dahulu di kolom kanan akan memberi tahu Chloros untuk hanya mencari target kalibrasi di gambar tersebut, sehingga sangat mempercepat pemrosesan.

</details>

<details>

<summary>Jika saya akan mengunggah gambar saya ke <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a> haruskah saya memprosesnya di Chloros sebelum mengunggah?</summary>

Jika Anda berencana mengunggah ke platform pemrosesan online kami [Awan MAPIR](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription) jangan mengedit gambar sebelum mengunggah. Cloud akan melakukan semua pemrosesan yang sama dan banyak lagi.

</details>

<details>

<summary>Akankah MAPIR mendukung fitur X? Saya sangat berharap MAPIR menawarkan X.</summary>

Kami selalu tertarik untuk menerima umpan balik tentang produk kami. Jika Anda menemukan masalah dengan produk kami, atau memiliki saran tentang bagaimana kami dapat meningkatkan produk kami, silakan [HUBUNGI KAMI](https://www.mapir.camera/community/contact) untuk menyampaikan pendapat Anda. Sebagian besar penelitian dan pengembangan kami dipandu dengan mendengarkan kebutuhan terbesar pelanggan kami.

</details>

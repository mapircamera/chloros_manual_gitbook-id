# Susunan Multi-Kamera

Sebuah **susunan**LATTICE terdiri dari dua atau lebih kamera LATTICE yang dihubungkan sebagai satu unit yang tersinkronisasi. Satu kamera berperan sebagai**master**: kamera ini mengirimkan pulsa pemicu GPIO perangkat keras melalui jalur sinkronisasi bersama (secara default**Line2**), sehingga setiap anggota mengambil gambar pada saat yang sama. Chloros menambahkan sinkronisasi waktu PTP, pratinjau langsung (ubin per kamera atau komposit multi-band tunggal yang selaras), dan pengambilan gambar yang tersinkronisasi — setiap proses pengambilan menghasilkan satu **kelompok bingkai** di mana semua kamera berbagi cap waktu dan ID bingkai yang sama (dilaporkan sebagai `fid:N` pada output pengambilan gambar).

Array adalah cara kamera mono (M3M) menghasilkan indeks vegetasi — satu kamera menyumbangkan satu band, dan array tersebut menyelaraskan band-band tersebut menjadi tumpukan multi-band. Lihat [Kamera Mono & Indeks Vegetasi](mono-indices.md).

Ada tiga cara yang setara untuk menghubungkan sebuah array, dan semuanya menjalankan alur "smart-prep" yang sama:

| Antarmuka | Titik masuk |
| --- | --- |
| GUI | Tab Kamera → **Hubungkan Array** (tombol biru) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (nomor seri pertama = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (nomor seri pertama = master) |

Smart-prep melakukan: pemeriksaan kemampuan jaringan (ICMP DF ping + pemeriksaan GVSP), pemilihan tingkatan sinkronisasi, proyeksi laju bingkai pada ukuran bingkai yang diminta, keputusan pengiriman 12-bit terkompresi (lihat [Transportasi 12-bit Terkemas](#packed-12-bit-transport)), pengaktifan PTP, pemilihan otomatis format piksel per kamera, pengaturan awal eksposur otomatis dari status yang disimpan masing-masing kamera, dan konfigurasi pemicu GPIO pada Line2.

**Sensor penuh secara default (baru di versi 1.3.0).** Connect mempertahankan ukuran bingkai yang Anda minta — sensor penuh 2048×1536, tanpa penggabungan piksel dan tanpa pemotongan, kecuali jika Anda memilih sebaliknya — dan membiarkan laju bingkai menyesuaikan dengan yang disediakan oleh host dan jaringan. Connect tidak pernah melakukan binning atau pemotongan secara otomatis (versi 1.2.0 secara otomatis mengaktifkan binning jika laju bingkai di bawah 1,5 fps). Binning atau ROI yang lebih kecil adalah pilihan Anda: di dialog Connect, pilih salah satunya dan bagian "Projected Outcome" akan menampilkan laju bingkai yang dihasilkan; `array-connect` mencetak opsi yang lebih cepat beserta laju bingkai yang diproyeksikan.

{% hint style="info" %}
Kamera harus dapat dijangkau melalui koneksi sebelum semua ini berfungsi — lihat [Menghubungkan Kamera](connecting.md) untuk proses penemuan, penentuan alamat, dan pengunduhan kalibrasi pada koneksi pertama. Untuk rig multi-kamera, pengaturan receive-ring pada NIC host sama pentingnya dengan kecepatan koneksi; tabel lengkap gejala→penyelesaian terdapat di [Referensi CLI § Pengaturan & Penyesuaian NIC Host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialog Array Connect

Tab Kamera → **Connect Array**membuka wizard tiga langkah:**Pilih → Mode Tampilan → Pengaturan**.

### Langkah 1 — Pilih master dan slave

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



Kotak dialog ini akan memindai jaringan segera setelah dibuka ("Memindai jaringan..."), kemudian memeriksa kabel sinkronisasi ("Memeriksa kabel sinkronisasi...") untuk menemukan kamera mana saja yang terhubung ke jalur sinkronisasi M8. Anda memerlukan setidaknya **2 kamera** untuk membuat sebuah array.

Kabel sinkronisasi M8 bertemu di satu titik sambungan, sehingga **kamera mana pun yang terhubung dengan kabel dapat menjadi master**. Proses pemeriksaan akan mengisi peran secara otomatis — kamera RGB (`-FRGB`) sebagai master jika terhubung, jika tidak maka kamera pertama yang terhubung, dan kamera-kamera terhubung lainnya sebagai slave — serta menampilkan salah satu dari tiga banner berikut:

| Banner | Arti |
| --- | --- |
| "Kabel sinkronisasi ditemukan pada {n} kamera. Salah satunya dapat menjadi master; kamera RGB (jika tidak, yang pertama) telah dipilih sebelumnya." (hijau) | Probe menemukan kamera-kamera pada jalur sinkronisasi; kotak centang master dan slave sudah tercentang. |
| "Tidak terdeteksi master - periksa kabel GPIO" (oranye) | Tidak ada kamera yang mendeteksi pulsa pemicu; periksa pemasangan kabel sinkronisasi. Peran-peran tersebut masih terisi secara otomatis (kamera RGB, atau kamera pertama dalam daftar, sebagai master) dan Anda dapat mengubahnya. |
| "Tidak Ada Kabel Sinkronisasi: {serial}" (oranye) | Kamera-kamera yang terdaftar tidak terhubung dengan kabel sinkronisasi. |

Tabel kamera memiliki kolom **Kamera / Serial / IP / Master (radio) / Slave (kotak centang)**. Baris-baris terdaftar berdasarkan**urutan kamera** — kamera M3C (Bayer) sebelum M3M (mono); di antara M3C, urutannya adalah FRGB, FRGN, FOCN, lalu FNGB; M3M berdasarkan panjang gelombang filter, dimulai dari yang terendah (F450 sebelum F808). Daftar kamera di bilah samping dan tata letak default kisi menggunakan urutan yang sama.

* Pilih tepat **satu master**dan**satu atau lebih slave**. Mengklik kotak radio master saat ini sekali lagi akan menghapusnya. Memilih master yang berbeda hanya memerlukan satu klik: master baru akan meninggalkan slave-slave tersebut dan master sebelumnya menjadi slave (kecuali jika tidak memiliki kabel sinkronisasi).
* Kamera yang ditandai **"Tanpa Kabel Sinkronisasi"** tidak pernah dapat dipilih sebagai slave — slave tanpa kabel pemicu akan menunggu di jalur sinkronisasi selamanya dan menghasilkan umpan yang mati. Hubungkan kamera tersebut sebagai kamera mandiri saja.
* Kamera yang sudah terhubung secara mandiri *tidak* dinonaktifkan: koneksi array akan melepaskan sesi mandiri dan membuka kembali kamera di dalam array.

**Selanjutnya: Mode Tampilan →**akan aktif setelah master dan setidaknya satu slave dipilih.**Pindai Ulang** akan menjalankan kembali proses pencarian dan pemeriksaan kabel.

{% hint style="warning" %}
**Batal** dinonaktifkan saat pemindaian atau pemeriksaan sedang berlangsung — membatalkan di tengah pemeriksaan dapat menyebabkan crash pada SDK kamera pada firmware kamera LATTICE. Tunggu hingga indikator putar selesai.
{% endhint %}

### Langkah 2 — Mode Tampilan



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Mode | Hasil yang Didapatkan |
| --- | --- |
| **Kamera Terpisah** | Satu ubin langsung per kamera, semuanya dipicu bersamaan sehingga bingkai tetap sinkron. Setiap kamera mempertahankan warna dan pengaturannya sendiri. |
| **Kamera Gabungan** *(default)* | Satu ubin yang menampilkan komposit multi-band NDVI/indeks yang diselaraskan. Kamera-kamera berbagi warna array. |

Mode tampilan hanya mengubah tampilan pratinjau langsung — perilaku perekaman tetap sama di kedua mode.

### Langkah 3 — Pengaturan Array dan Hasil

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

yang Diperkirakan

Saat memasuki adegan ini, Chloros meminta **rekomendasi**dari backend dan menerapkannya:**sensor penuh — tanpa binning, tanpa pemotongan ROI — selama laju bingkai dapat menampungnya**, pada laju apa pun yang disediakan oleh host dan koneksi. Hanya jika tidak ada laju yang dapat menampung sensor penuh, rekomendasi akan memperkecil bingkai, dan kemudian lebih memilih binning daripada pemotongan ROI, karena binning mempertahankan bidang pandang penuh. Dari situ, Anda menukar resolusi dengan kecepatan bingkai. Setiap perubahan yang Anda lakukan akan menjalankan ulang analisis secara langsung dan memperbarui panel**Hasil yang Diperkirakan** di sebelah kanan.

Kolom kiri — pengaturan:

| Kontrol | Pilihan | Default | Catatan |
| --- | --- | --- | --- |
| **ROI (Bidang Pandang)** | Penuh (2048×1536) / Setengah (1024×768) / Seperempat (512×384) | Penuh | Pemotongan sensor: Pemotongan setengah/seperempat ke wilayah yang lebih kecil pada pitch piksel asli. |
| **Binning** | 1× / 2× (jumlah 2×2) / 4× (jumlah 4×4) | 1× | Binning perangkat keras: 2×2 = bidang pandang penuh dengan seperempat biaya transmisi; 4×4 = bidang pandang penuh dengan 1/16 biaya transmisi. Tersembunyi jika kamera tidak mendukung binning. |
| **Gambar sisi kabel** (pembacaan) | — | — | Lebar×tinggi pasca-binning yang sebenarnya dikirim melalui kabel, dibulatkan ke kelipatan 16 (minimum 64). |
| **Laju Pemicu** | 0,5–60 fps, langkah 0,1 | kosong = otomatis | Laju pemicu master. Biarkan kosong agar Chloros dapat menghitungnya secara otomatis. |
| **Anggaran Jaringan**| 20–2000 MB/s, langkah 10 | kosong = otomatis | Seberapa besar kapasitas yang sebenarnya dapat diserap oleh host, dalam MB/s —**satu-satunya angka yang menjadi dasar alokasi seluruh array.** Terdeteksi secara otomatis dari adaptor jaringan. Kurangi nilainya jika array melaporkan frame yang rusak: nilai yang terdeteksi terlalu tinggi untuk adaptor USB dan sakelar bersama. Mengubahnya akan menjalankan kembali proyeksi secara langsung. Kolom ini terbuka dengan anggaran yang digunakan saat array terakhir terhubung (disimpan bersama proyek), sehingga anggaran yang pernah Anda koreksi akan tetap disimpan. |

Kotak centang **Resolusi Pin** dari versi 1.2.0 telah dihapus: koneksi selalu mempertahankan ukuran frame yang Anda pilih dan menurunkan laju frame sebagai gantinya.

Kolom kanan — **Hasil Proyeksi**:

* **Tingkat sinkronisasi** — "Perekaman simultan" (hijau), "Perekaman simultan (emisi bergantian FTD)" (hijau), "Perekaman bergantian (pergeseran 100 ms)" (kuning), atau "Konfigurasi terlalu besar" (merah). Chloros memilih tingkatan: perekaman simultan setiap kali kabel dapat dibagi dengan kecepatan yang dapat disesuaikan dengan kamera (kecepatan bingkai menurun seiring berkurangnya porsi setiap kamera), dan perekaman bergantian hanya jika tidak memungkinkan — sehingga susunan terhubung pada jaringan apa pun, dengan kecepatan bingkai yang diizinkan oleh jaringan tersebut.
* **Proyeksi fps** — laju bingkai yang diharapkan dapat dijalankan oleh array. Pada pengambilan gambar simultan, nilainya berupa satu angka: laju frame ditentukan terlebih dahulu, dan eksposur otomatis mengisi rentang waktu yang ditentukan, lalu menaikkan gain, sehingga pemandangan yang redup tidak menurunkan laju frame (lihat [Bagaimana eksposur otomatis mengatur eksposur dan gain](camera-settings.md)). Pada perekaman bergiliran, nilainya ditampilkan sebagai rentang ("gelap → terang"), karena durasi eksposur setiap kamera memperpanjang selang waktu dalam adegan yang gelap.
* **NIC line** — kecepatan tautan dan throughput berkelanjutan ("NIC {mbps} Mbps · berkelanjutan {N} MB/s").
* **Pemeriksaan burst Sim-emit** — apakah cincin penerima NIC host dapat menampung satu burst simultan dari semua kamera ("Burst Sim-emit: X MB · Kapasitas cincin NIC: Y MB ✓/✗").
* **Pemeriksaan anggaran kabel** — permintaan agregat dalam kondisi stabil versus batas atas kabel yang aman dari tabrakan ("Anggaran kabel: {demand} MB/s yang diminta oleh {n} kamera · batas atas {ceiling} MB/s ✓/✗ kelebihan alokasi").
* **"Jumlah kamera maksimum pada kabel ini: {n} — ditentukan oleh batas bawah bandwidth per kamera, sehingga pengelompokan tidak akan menaikkannya."** — ditampilkan saat Anda mendekati (atau melebihi) batas jumlah kamera.
* **"FRAME AKAN TERLEWATKAN pada pengaturan ini."**— peringatan merah dengan alasan dari backend, ditambah daftar penghambat dan**saran perbaikan** berwarna biru ("Agar susunan ini muat di jaringan" / "Untuk mengaktifkan perekaman simultan").**Terapkan & Hubungkan** tidak dapat diakses sampai ada proyeksi, dan labelnya memberi tahu Anda alasan penolakannya:

| Label tombol | Arti | Apa yang sebenarnya membantu |
| --- | --- | --- |
| "Sedang menganalisis..." | Analisis masih berlangsung. | Tunggu. |
| **"Terlalu banyak kamera untuk jaringan ini"**| Jaringan tidak dapat menangani array ini bahkan satu kamera pun (pemeriksaan agregat gagal): jumlah kamera jauh melebihi kapasitasnya, atau ada kamera lain yang sedang streaming di jaringan tersebut. | Kurangi jumlah kamera, gunakan jumbo frames end-to-end, atau gunakan NIC yang lebih cepat.**Mengurangi ROI TIDAK AKAN membantu** — lihat di bawah. |
| **"Kurangi ROI untuk mengaktifkan"** | Frame akan terlewat pada pengaturan ini (pemeriksaan burst/ring gagal). | Kurangi ROI, tingkatkan binning, atau perbaiki ring penerimaan NIC. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Saat menghubungkan, **panel unduhan kalibrasi** berwarna hijau mungkin muncul dengan bilah kemajuan per serial: saat kamera dihubungkan ke mesin untuk pertama kalinya, Chloros mengambil paket kalibrasi pabrik berukuran ~3,8 MB dari kamera melalui GigE (sekitar 70 detik per kamera). Kamera yang sudah tersimpan dalam cache tidak akan pernah menampilkan panel ini; jika ada lebih dari enam kamera, baris-barisnya akan bergulir. Lihat [Menghubungkan Kamera](connecting.md).

### Pengiriman 12-bit yang dikemas

Format 12-bit (`BayerRG12`, `Mono12`) biasanya dikirim sebagai kata 16-bit. **Transportasi**terkompresi**mengirimkan 12 bit yang sama dalam 1,5 byte per piksel dan mendekompresinya di host, sehingga setiap frame membutuhkan waktu 25% lebih sedikit untuk dikirim — pada sensor penuh, kira-kira sepertiga lebih banyak frame per detik.**Data yang disimpan identik.** Array Connect menentukannya secara otomatis (`auto`): pengemasan hanya dilakukan jika bermanfaat dan host memiliki CPU yang cukup untuk proses pembongkaran. Antarmuka grafis (GUI) tidak memiliki opsi untuk ini; dari antarmuka baris perintah (CLI), gunakan `array-connect --packed {auto,on,off}` (default `auto`). Format 8-bit dan 16-bit tidak pernah dikompresi.

## Bandwidth: berapa banyak kamera yang dapat ditampung

Kapasitas yang dapat ditangani oleh sebuah array bergantung pada koneksi jaringan, bukan padChloros, sehingga angka-angka perencanaan terdapat dalam manual perangkat keras: **[Perencanaan Bandwidth Array](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Apa yang dilakukan Chloros: dialog koneksi menjalankan pemeriksaan jaringan, memperkirakan laju bingkai yang dapat dicapai, dan memilih tingkatan yang sesuai. Sebuah array dapat terhubung ke jaringan apa pun: pada koneksi lambat, setiap kamera disesuaikan dengan porsinya pada saluran dan laju bingkai akan menurun, alih-alih array ditolak atau bingkainya dipersempit. Hanya array yang koneksinya tidak mampu menampung bahkan satu kamera pun pada satu waktu yang akan ditolak (**Terlalu banyak kamera untuk jaringan ini**), bukan dengan membuang paket secara diam-diam — lihat panel hasil proyeksi yang dijelaskan di atas.

## Saat frame hilang

Sebuah kamera dapat tidak muncul dalam grup yang dipublikasikan karena dua alasan yang sama sekali berbeda,
dan keduanya memerlukan solusi yang berlawanan. Chloros menghitungnya secara terpisah alih-alih melaporkan satu
angka "tidak lengkap" yang tidak menyebutkan keduanya:

| Apa yang terjadi | Artinya | Di mana harus dicari |
| --- | --- | --- |
| **Rusak**— frame tiba namun secara struktural rusak | Kehilangan paket GVSP di jalur jaringan |**Kapasitas saluran**, cincin penerimaan NIC, frame jumbo, switch |
| **Tidak pernah tiba**— tidak ada frame yang datang sama sekali | Kamera tidak memicu, atau tidak ada yang keluar darinya |**Kabel sinkronisasi M8**, jalur sinkronisasi, apakah setiap anggota dalam status aktif |

Pembagian ini dievaluasi ulang setiap 10 detik selama array melakukan streaming. Di atas 5%, hal ini
dicatat dengan mencantumkan kedua angka tersebut, dan setiap buffer yang rusak dilaporkan saat pertama kali
terjadi per kamera, kemudian dirangkum setiap menit agar sesi yang panjang tetap dapat dibaca.

**Frame yang rusak dengan nilai nol untuk "tidak pernah tiba" berarti pemicu dan sinkronisasi kabel sempurna**dan setiap frame yang hilang terjadi di jalur jaringan. Solusinya adalah menurunkan**Wire Budget** dan
menyambungkan ulang.

{% hint style="warning" %}
**Menurunkan laju pemicu tidak membantu mengatasi frame yang rusak.** Pacing paket kamera
ditentukan sekali saja, saat koneksi terjalin. Menurunkan laju pemicu mengubah seberapa sering burst
terjadi, bukan seberapa cepat burst itu sendiri dikirim ke kabel. Pada rig 4 kamera yang diukur,
pengurangan laju pemicu sebesar 5× tidak mengubah apa pun, sementara menurunkan wire budget dari 240 menjadi
200 MB/s menurunkan tingkat kerusakan pada rig yang sama dari 10,4% menjadi nol.
{% endhint %}

Sebuah array yang sedang berjalan tidak dapat merencanakan ulang sendiri — putuskan sambungan dan sambungkan kembali agar pemilih waktu koneksi
dapat bekerja sesuai dengan anggaran baru.

### Adaptor jaringan USB memiliki batasan

Adaptor Ethernet USB mengiklankan kecepatan tautan *Ethernet*-nya, tetapi kecepatan yang sebenarnya
dapat dipertahankan dibatasi oleh bus USB dan drivernya. Sebuah dongle USB 10GbE dulunya diklaim
memiliki throughput sekitar 1.000 MB/s — angka yang belum pernah diukur sebelumnya — dan mengatur
kecepatan empat kamera berdasarkan ruang kosong fiktif tersebut merusak 6–18 % frame, sementara array
masih melaporkan laju frame target yang normal. Adaptor yang terhubung melalui USB kini dibatasi pada
**200 MB/s** di Windows dan NVIDIA Jetson, serta **400 MB/s** di Linux x86_64. Batas ini bersifat absolut, bukan persentase, karena batasnya terletak pada
bus: adaptor USB 1 GbE menghasilkan sekitar 80 MB/s dan tidak terpengaruh.

Jika host Anda benar-benar lebih cepat dari batas tersebut, naikkan **Wire Budget** untuk menyesuaikannya.

## Sinkronisasi Waktu PTP

*Sinkronisasi* bingkai berasal dari pemicu perangkat keras; **PTP** (IEEE 1588 PTPv2) menyediakan *cap timestamp* yang seragam di seluruh perangkat. Fitur ini diaktifkan secara default saat array terhubung:

* **Backend host Chloros menjalankan grandmaster PTP**. Kamera LATTICE dan sensor cahaya DAQ-E bertindak sebagai slave di domain 0, sehingga cap waktu gambar dan spektrum DAQ berada pada satu jam (~1 ms).
* `--no-ptp` (CLI) menonaktifkannya untuk pekerjaan di meja kerja — cap waktu antar-kamera kemudian **tidak** dapat dibandingkan.
* Periksa status sinkronisasi melalui CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Tab "Cameras" sendiri tidak memiliki indikator PTP; informasi sinkronisasi per kamera yang ditampilkan di sana adalah **Role**(Master/Slave) yang hanya dapat dibaca,**Sync Line**, dan tingkatan Capabilities dari array. Status PTP DAQ-E ditampilkan di detail sensor pada tab "Light Sensors".

## Tampilan

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

array langsung Area umpan utama menawarkan dua tata letak (beralih di bilah atas): **tampilan kisi**(setiap ubin adalah sel; seret untuk mengatur ulang urutan saat gembok kisi tidak terkunci) dan**tampilan daftar**(baris lebar penuh: array**gabungan**merupakan satu baris — ubin gabungannya — sedangkan array**terpisah**memberikan**baris tersendiri untuk setiap anggotanya**, dengan umpan langsung dan tabel Target Kalibrasi masing-masing; di bawah array, terdapat satu kamera mandiri — yang aktif). Penggeser**Zoom Umpan** mengatur ukuran ubin; jika lebar sel di bawah 200 piksel, overlay nama/fps akan tersembunyi secara otomatis. Kamera anggota muncul di bilah samping dan dalam tata letak default kisi sesuai urutan kamera (M3C sebelum M3M, FRGB terlebih dahulu, monokrom berdasarkan panjang gelombang); membuka kunci kisi dan menyeret tetap akan menggantikan urutan kisi.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**Mode terpisah** menampilkan satu ubin per kamera. Setiap ubin menampilkan:

* nama kamera (kiri atas),
* **tampilan fps** (kanan atas) — ini adalah *kecepatan pengambilan gambar sebenarnya* kamera yang dilaporkan oleh backend, bukan kecepatan polling pratinjau (pratinjau langsung dibatasi hingga 30 fps terlepas dari kecepatan pengambilan gambar),
* titik status — hijau (sedang streaming) / kuning (sedang dimuat) / merah (kesalahan),
* **spinner bingkai usang** ketika tidak ada bingkai baru yang tiba selama 2 detik — hal ini normal selama ~5 detik setelah koneksi terputus atau terhubung kembali sementara backend menyeimbangkan kembali alokasi bandwidth antar kamera.**Mode gabungan**menampilkan satu ubin komposit: backend melakukan debayering, penskalaan, penyelarasan, penghilangan noise, konversi ke radian per-band (ditambah reflektansi DLS jika sensor cahaya terikat), mengevaluasi ekspresi indeks array, menerapkan LUT, dan mengalirkan hasilnya sebagai MJPEG. Sampai bingkai pertama yang telah disejajarkan ditampilkan, ubin tersebut menjelaskan statusnya: "Mempersiapkan susunan…", "Mengkalibrasi penyelarasan…", "Menunggu bingkai pertama…", atau — jika upaya ulang penyelarasan otomatis ubin (sekitar satu menit) habis — "Diperlukan penyelarasan" dengan tombol**Kalibrasi penyelarasan**.

Tampilan fps ubin array juga melaporkan status streaming:**"Menerapkan pengaturan…"**saat perubahan pengaturan (format, binning, ROI) me-restart kamera,**"Terhenti"**ketika array masih aktif tetapi telah berhenti mengirimkan kelompok frame, dan**"Berhenti"**saat array telah berhenti melakukan streaming — dalam hal ini, sambungkan kembali array tersebut. Ketika pemisahan spektral harus membatasi (clamp) suatu band yang ditampilkan dalam komposit pada sebagian besar pikselnya (25% atau lebih), ubin akan menampilkan**"Data band dengan tingkat kepercayaan rendah · N%"**: nilai indeks yang dibangun berdasarkan pita tersebut diragukan keakuratannya (biasanya satu pita menerima cahaya yang terlalu sedikit).

Fakta berguna mengenai mode gabungan:

* Gambar gabungan disinkronkan dengan bingkai kamera **utama**. Penargetan AE-ROI dan pengukuran titik pada gambar gabungan bersifat tepat untuk kamera utama dan perkiraan untuk kamera bawahan; gunakan**Split View** (pengaturan array → "Tampilkan kamera anggota") untuk ubin per-kamera yang tepat hingga tingkat piksel tanpa membuka koneksi kamera tambahan.
* ****Lapisan Tampilan**(pengaturan array; default nonaktif) memungkinkan Anda memilih lapisan latar depan dan latar belakang — kamera anggota mana pun atau**Indeks**. Jika latar depan = Indeks, piksel di luar batas Min/Max LUT akan menampilkan lapisan latar belakang.
* **Resolusi render** (default 720p) mengatur tinggi siaran langsung *dan* ukuran ekspor komposit yang disimpan. Gambar per kamera selalu diekspor pada resolusi penuh.
* Penyelarasan dihitung per sesi dan tidak pernah disimpan. Chloros menyelaraskan **setiap array secara otomatis setelah terhubung, di kedua mode tampilan**, setelah eksposur otomatis anggota stabil (proses ini terus dicoba hingga 5 menit). Bagian**Penjajaran**pada panel pengaturan array menampilkan sisa RMS dan tombol**Kalibrasi Ulang Penjajaran** — juga untuk array terpisah yang tidak memiliki ubin komposit.

## Perekaman: pemantauan vs analisis

Permukaan perekaman array terbagi dengan jelas menjadi **tingkat pemantauan**(merekam apa yang Anda lihat) dan**tingkat analisis** (merekam data mentah, dikalibrasi nanti):

| Alur Kerja | Tingkat | Apa yang disimpan | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Perekaman**(gambar diam) | Analisis | Satu grup bingkai yang disinkronkan per lintasan; file per kamera pada setiap tingkat ekspor yang dipilih (mentah/debayered/radiance/reflectance/pratinjau/indeks) + file sidecar `.daq` | Tombol**Tombol **Capture All** + Pengaturan Capture | `lattice array-capture` |
| **Rekam video indeks** | Pemantauan | Komposit indeks gabungan langsung seperti yang ditampilkan — 8-bit, resolusi pratinjau, LUT sudah terintegrasi. Dari CLI tanpa tampilan GUI yang terbuka, backend merender NDVI (atau `--expr`) dari array setelah array tersebut disejajarkan; perekaman yang tidak menulis frame apa pun akan gagal | ● Rekam video indeks (array gabungan) | `lattice array-record` |
| **Burst mentah → buat video**| Analisis | Frame sensor mentah pada laju pengambilan penuh + manifest + `.daq`, kemudian rekonstruksi offline menjadi video radian / reflektansi / indeks yang dikalibrasi, disesuaikan waktunya dengan pembacaan DAQ | ⦿ Rekam burst mentah →**Buat video** | `lattice array-burst` → `lattice array-build-video` |

Aturan praktis: jika piksel akan digunakan untuk *pengukuran*, gunakan mode capture atau burst (tingkat analisis); jika Anda hanya perlu *menonton atau mendemonstrasikan* apa yang dilihat oleh array, rekam video indeks (tingkat pemantauan).

### Pengaturan Capture (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



Ikon roda gigi di sebelah **Capture All** membuka panel Pengaturan Capture (memerlukan proyek yang terbuka — hasil capture disimpan di dalamnya):

* **Mode Capture**:**Single**(satu kali) /**Continuous**(berurutan; dibatasi oleh jumlah capture, default 1, atau durasi, default 10 detik) /**Interval** (timelapse: N perekaman setiap interval X untuk total Y, default 1 setiap 5 detik selama 1 menit).
* **Jenis ekspor per kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — semua yang berlaku diaktifkan secara default. Radiance/Reflectance disembunyikan untuk kamera dengan filter RGB; **Reflectance hanya muncul jika kamera memiliki sensor cahaya DAQ** (miliknya sendiri atau diwarisi dari array); Index memerlukan ekspresi indeks yang telah dikonfigurasi.
* **Aligned**(per array, default**aktif**): menyesuaikan ekspor anggota dengan profil penyelarasan array sehingga ekspor terdaftar berdasarkan piksel. Raw selalu tetap tidak disesuaikan tetapi membawa transformasi dalam metadata. File anggota array disimpan dalam orientasi asli masing-masing kamera (pengaturan Mirror/Rotation apa pun dicatat dalam file, tidak diterapkan).
* **Fastest Capture** (toggle): hanya raw + pembacaan DAQ yang ditetapkan + komposit indeks gabungan gratis, melewati perhitungan kalibrasi pada saat pengambilan gambar untuk kecepatan maksimum — bangun kembali radiance/reflectance/indeks nanti dari `.daq` yang disimpan.
* Pilihan tetap berlaku pada proyek. Kamera yang disembunyikan atau dijeda dilewati.
* **Continuous** pada array berjalan di dalam backend Chloros (baru di 1.3.0): ketika setiap kamera dalam daftar termasuk dalam sebuah array, backend menjalankan loop pengambilan gambarnya sendiri, menulis satu grup sambil memilih grup berikutnya, tanpa permintaan per pengambilan gambar. Drive yang penuh akan menghentikan proses ("Berhenti: drive penuh. N disimpan, F gagal"). Lihat [Pengaturan & Mode Pengambilan](capture.md#capture-modes) untuk catatan Microsoft Defender mengenai laju pengambilan yang tinggi.

CLI yang setara (titik akhir backend yang sama, semantik yang sama):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

Kompresi TIFF untuk penangkapan adalah `deflate` (tanpa kehilangan data, default) atau `none` — tabel bendera lengkap, tata letak folder penangkapan, dan aturan pemrosesan ulang terdapat di [Referensi CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Memasangkan sensor cahaya DAQ

Pratinjau yang dikoreksi reflektansi dan iluminasi memerlukan data cahaya yang mengarah ke bawah dari sensor DAQ (yang terhubung di tab **Sensor Cahaya**):

* **Baris array**di bilah samping menampilkan**kotak "DAQ · on/off"** — *on* jika sensor cahaya tingkat array telah diatur **atau** salah satu kamera anggotanya memiliki sensor sendiri; tooltip-nya mencantumkan secara tepat sensor mana yang menyalurkan data ke kamera mana.
* Tetapkan pengaturan untuk seluruh array di menu pengaturan array → **Sensor Cahaya Sekitar**→ menu tarik-turun**Sensor Cahaya**. Pilihan ini tetap berlaku sepanjang proyek, diterapkan ke setiap kamera anggota, dan kamera individu masih dapat menggantikannya dengan sensornya sendiri.
* Baris status di bawahnya melaporkan status real-time: **Mati**→ "Menunggu spektrum pertama…" →**"Aktif — semua kamera dalam array telah dikoreksi pencahayaannya"** → atau, jika tidak ada spektrum baru yang diterima dalam 3 detik terakhir, muncul pemberitahuan bahwa data sudah kadaluwarsa — pembacaan terakhir tetap digunakan (pembacaan tidak pernah kadaluwarsa pada jalur pengambilan gambar).

Dengan sensor yang telah ditetapkan: jenis ekspor Reflectance menjadi tersedia, pratinjau langsung telah dikoreksi pencahayaannya, eksposur otomatis prediktif dapat menggunakan spektrum tersebut, dan setiap pengambilan gambar reflektansi menulis pembacaan DAQ yang sebenarnya digunakan sebagai **berkas `.daq`** di dalam folder `daq/` sehingga penangkapan tersebut dapat diproses ulang nanti. Di sampingnya, berkas `.daq.cameras.json` mencantumkan nama kamera-kamera yang bingkai gambarnya dijelaskan oleh pembacaan tersebut; saat diproses ulang, Chloros hanya mencocokkan pembacaan tersebut dengan gambar dari kamera-kamera tersebut.

## Opsi CLI

** `array-connect`

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--serials SN1,SN2,…` | temukan semua kamera LATTICE secara otomatis (diperlukan ≥2) | **Nomor seri pertama adalah MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Jalur sinkronisasi GPIO. |
| `--target-fps F` | auto | Kecepatan pemicu MASTER. |
| `--binning {1,2,4}` | auto | Pengelompokan perangkat keras. Opsi "auto" menggunakan sensor penuh (tanpa pengelompokan) setiap kali laju bingkai mendukungnya; opsi "connect never" tidak melakukan pengelompokan hanya untuk menjaga laju tetap tinggi. |
| `--packed {auto,on,off}` | `auto` | [Transportasi 12-bit terkompresi](#packed-12-bit-transport): `auto` hanya melakukan pengemasan jika hal itu membantu dan host mampu melakukan penguraian. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | otomatis | Penggantian pengaturan sync-tier oleh ahli. |
| `--wire-ceiling-mbps MB_PER_S` | terdeteksi otomatis | Anggaran bandwidth host dalam MB/s — bentuk CLI dari bidang **Wire Budget**. Kurangi nilainya jika array melaporkan frame yang rusak. Disimpan bersama proyek, sehingga saat terhubung kembali nanti, nilainya akan dipulihkan. |
| `--no-recommend` | nonaktif | Lewati langkah analisis jaringan. |
| `--no-ptp` | nonaktif | Nonaktifkan PTP (cap waktu antar-kamera tidak dapat dibandingkan). |

`lattice array-list`, `array-status`, dan `array-disconnect` mengelola sesi yang berkelanjutan. Referensi subperintah lengkap, termasuk penyelarasan (`align-calibrate` / `align-apply`) dan alat jaringan, terdapat di [Referensi CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); padanan dalam SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) terdapat di [Referensi SDK](../reference/sdk-reference.md). Dari Python, anggaran kabel adalah `connect_array(..., wire_ceiling_mbps=120)`, dan pembagian antara data yang rusak saat ditransmisikan dan data yang tidak pernah sampai terdapat di [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

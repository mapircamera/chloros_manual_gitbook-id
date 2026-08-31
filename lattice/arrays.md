# Susunan Multi-Kamera

**Susunan**LATTICE adalah dua atau lebih kamera LATTICE yang dihubungkan sebagai satu unit yang tersinkronisasi. Salah satu kamera berperan sebagai**master**: kamera ini mengirimkan pulsa pemicu GPIO perangkat keras melalui jalur sinkronisasi bersama (secara default**Line2**), sehingga setiap kamera menangkap momen yang sama. Chloros menambahkan sinkronisasi waktu PTP, pratinjau langsung (ubin per kamera atau komposit multi-band tunggal yang sejajar), dan pengambilan gambar yang tersinkronisasi — setiap proses pengambilan menghasilkan satu**kelompok bingkai** di mana semua kamera memiliki cap waktu dan ID bingkai yang sama (dilaporkan sebagai `fid:N` pada output pengambilan gambar).

Array adalah cara kamera mono (M3M) menghasilkan indeks vegetasi — satu kamera menyumbangkan satu band, dan array menyelaraskan band-band tersebut menjadi tumpukan multi-band. Lihat [Kamera Mono &amp; Indeks Vegetasi](mono-indices.md).

Terdapat tiga cara yang setara untuk menghubungkan sebuah array, dan semuanya menjalankan alur &quot;smart-prep&quot; yang sama:

| Permukaan | Titik masuk |
| --- | --- |
| GUI | Tab Kamera → **Hubungkan Array** (tombol biru) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (nomor seri pertama = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (nomor seri pertama = master) |

Smart-prep melakukan, secara berurutan: pengujian kemampuan jaringan (ICMP DF ping + pengujian GVSP), pemilihan tingkatan sinkronisasi, pengecilan otomatis ukuran frame agar sesuai dengan kabel, pengaktifan PTP, pemilihan otomatis format piksel per kamera, penetapan awal eksposur otomatis berdasarkan status yang disimpan dari masing-masing kamera, serta konfigurasi pemicu GPIO pada Line2.

{% hint style="info" %}
Kamera harus dapat dijangkau melalui tautan sebelum semua ini berfungsi — lihat [Menghubungkan Kamera](connecting.md) untuk penemuan, penentuan alamat, dan pengunduhan kalibrasi koneksi pertama. Untuk rig multi-kamera, pengaturan receive-ring NIC host sama pentingnya dengan kecepatan koneksi; tabel lengkap gejala→penyelesaian terdapat di [CLI Referensi § Pengaturan &amp; Penyesuaian NIC Host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## Dialog Array Connect

Tab Kamera → **Connect Array**akan membuka wizard tiga langkah:**Pilih → Mode Tampilan → Pengaturan**.

### Langkah 1 — Pilih master dan slave

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

Kotak dialog ini akan memindai jaringan segera setelah dibuka (&quot;Memindai jaringan...&quot;), lalu memeriksa kabel pemicu GPIO (&quot;Memeriksa kabel GPIO...&quot;). Anda memerlukan setidaknya **2 kamera** untuk membuat sebuah array.

Pemeriksaan kabel akan mengisi pilihan peran secara otomatis jika memungkinkan, dan menampilkan salah satu dari tiga pesan berikut:

| Pesan | Arti |
| --- | --- |
| &quot;Master GPIO terdeteksi — pilihan telah terisi otomatis&quot; (hijau) | Pemeriksaan menemukan topologi pemicu; kotak centang master radio dan slave sudah tercentang. |
| &quot;Tidak ada master yang terdeteksi - periksa kabel GPIO&quot; (oranye) | Tidak ada kamera yang mendeteksi pulsa pemicu; periksa kabel sinkronisasi. Anda masih dapat memilih peran secara manual. |
| &quot;Tidak Ada Kabel Sinkronisasi: {serial}&quot; (oranye) | Kamera-kamera yang terdaftar tidak terhubung dengan kabel sinkronisasi. |

Tabel kamera memiliki kolom **Kamera / Serial / IP / Master (radio) / Slave (kotak centang)**:

* Pilih tepat **satu master**dan**satu atau lebih slave**. Mengklik radio master saat ini sekali lagi akan menghapusnya.
* Kamera yang ditandai **&quot;Tanpa Kabel Sinkronisasi&quot;** tidak pernah dapat dipilih sebagai slave — slave tanpa kabel pemicu akan menunggu di jalur sinkronisasi selamanya dan menghasilkan siaran yang mati. Hubungkan kamera tersebut sebagai kamera mandiri saja.
* Kamera yang sudah terhubung secara mandiri *tidak* dinonaktifkan: koneksi array akan melepaskan sesi mandiri dan membuka kembali kamera di dalam array.

**Selanjutnya: Mode Tampilan →**diaktifkan setelah master dan setidaknya satu slave dipilih.**Pindai Ulang** menjalankan ulang proses pencarian dan pemeriksaan kabel.

{% hint style="warning" %}
**Batal** dinonaktifkan saat pemindaian atau pemeriksaan sedang berlangsung — membatalkan di tengah pemeriksaan dapat menyebabkan kamera SDK pada firmware kamera LATTICE mengalami crash. Tunggu hingga ikon putar selesai.
{% endhint %}

### Langkah 2 — Mode

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

Tampilan | Mode | Hasil yang Didapatkan |
| --- | --- |
| **Kamera Terpisah** | Satu ubin langsung per kamera, semuanya dipicu bersamaan sehingga bingkai tetap sinkron. Setiap kamera mempertahankan warna dan pengaturannya sendiri. |
| **Kamera Gabungan** *(default)* | Satu ubin yang menampilkan komposit multi-band NDVI/indeks yang telah diselaraskan. Kamera-kamera berbagi warna array. |

Mode tampilan hanya mengubah tampilan pratinjau langsung — perilaku perekaman tetap sama di kedua mode.

### Langkah 3 — Pengaturan Array dan Hasil

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

yang Diperkirakan Saat memasuki adegan ini, Chloros meminta **rekomendasi**dari backend dan secara otomatis menerapkan kombinasi ROI + binning yang sesuai dengan cincin penerima NIC Anda (sistem lebih memilih binning daripada pemotongan ROI, karena binning mempertahankan bidang pandang penuh). Setiap perubahan yang Anda lakukan akan menjalankan ulang analisis secara langsung dan memperbarui panel**Hasil yang Diproyeksikan** di sebelah kanan.

Kolom kiri — pengaturan:

| Kontrol | Pilihan | Default | Catatan |
| --- | --- | --- | --- |
| **ROI (Bidang Pandang)** | Penuh (2048×1536) / Setengah (1024×768) / Seperempat (512×384) | Penuh | Pemotongan sensor: Pemotongan setengah/seperempat ke wilayah yang lebih kecil pada pitch piksel asli. |
| **Binning** | 1× / 2× (jumlah 2×2) / 4× (jumlah 4×4) | 1× | Binning perangkat keras: 2×2 = bidang pandang penuh dengan seperempat biaya koneksi; 4×4 = FoV penuh dengan 1/16. Tersembunyi jika kamera tidak mendukung binning. |
| **Gambar sisi kabel** (pembacaan) | — | — | Lebar×tinggi pasca-binning yang sebenarnya dikirim melalui kabel, disesuaikan menjadi kelipatan 16 (minimum 64). |
| **Resolusi pin**| kotak centang | nonaktif | Chloros biasanya menaikkan pengelompokan secara otomatis saat terhubung ketika laju yang diproyeksikan turun di bawah**1,5 fps**. Penguncian pin mempertahankan ukuran bingkai yang Anda pilih dan menerima laju yang lebih rendah — serta mengubah konfigurasi yang kelebihan beban menjadi penolakan koneksi secara tegas, bukan penurunan laju otomatis. |
| **Laju Pemicu** | 0,5–60 fps, langkah 0,1 | kosong = otomatis | Laju pemicu master. Biarkan kosong agar Chloros menghitungnya sendiri. |
| **Anggaran Jaringan**| 20–2000 MB/s, langkah 10 | kosong = otomatis | Seberapa banyak yang dapat diserap oleh host, dalam MB/s —**satu-satunya angka yang menjadi acuan alokasi seluruh array.** Terdeteksi secara otomatis dari adaptor jaringan. Turunkan nilainya jika array melaporkan frame yang rusak: nilai yang terdeteksi terlalu tinggi untuk adaptor USB dan switch bersama. Mengubahnya akan menjalankan ulang proyeksi secara langsung. |

Kolom kanan — **Hasil Proyeksi**:

* **Tingkat sinkronisasi** — &quot;Perekaman simultan&quot; (hijau), &quot;Perekaman simultan (pengiriman berurutan FTD)&quot; (hijau), &quot;Perekaman berurutan (penundaan 100 ms)&quot; (kuning), atau &quot;Konfigurasi terlalu besar&quot; (merah).
* **proyeksi fps** — ditampilkan sebagai rentang (&quot;redup → terang&quot;), karena laju array yang disinkronkan dibatasi oleh waktu paparan kamera yang paling lambat.
* **baris NIC** — kecepatan tautan dan throughput berkelanjutan (&quot;NIC {mbps} Mbps · berkelanjutan {N} MB/s&quot;).
* **Pemeriksaan burst emisi simultan** — apakah cincin NIC host dapat menerima satu burst simultan dari semua kamera (&quot;Burst emisi simultan: X MB · kapasitas cincin NIC yang tersedia: Y MB ✓/✗&quot;).
* **Pemeriksaan anggaran kabel** — permintaan agregat dalam kondisi stabil versus batas atas kabel yang aman dari tabrakan (&quot;Anggaran kabel: {demand} MB/s yang diminta oleh {n} kamera · batas atas {ceiling} MB/s ✓/✗ kelebihan alokasi&quot;).
* **&quot;Jumlah kamera maksimum pada kabel ini: {n} — ditentukan oleh batas bawah bandwidth per kamera, sehingga pengelompokan tidak akan menaikkannya.&quot;** — ditampilkan saat Anda mendekati (atau melebihi) batas atas jumlah kamera.
* **&quot;FRAME AKAN TERLEWATKAN pada pengaturan ini.&quot;**— peringatan merah dengan alasan dari backend, ditambah daftar penghambat dan**saran perbaikan** berwarna biru (&quot;Agar susunan ini muat di jaringan&quot; / &quot;Untuk mengaktifkan perekaman simultan&quot;).**Terapkan &amp; Hubungkan** akan diblokir hingga ada proyeksi, dan labelnya memberi tahu Anda alasan penolakan:

| Label tombol | Arti | Solusi yang efektif |
| --- | --- | --- |
| &quot;Sedang dianalisis...&quot; | Analisis masih berlangsung. | Tunggu. |
| **&quot;Terlalu banyak kamera untuk jaringan ini&quot;**| Array tersebut membebani koneksi jaringan secara berlebihan (pemeriksaan agregat gagal). | Kurangi jumlah kamera, gunakan jumbo frame dari ujung ke ujung, atau gunakan NIC yang lebih cepat.**Mengurangi ROI TIDAK AKAN membantu** — lihat di bawah. |
| **&quot;Kurangi ROI untuk mengaktifkan&quot;** | Frame akan terlewat pada pengaturan ini (pemeriksaan burst/ring gagal). | Kurangi ROI, tingkatkan binning, atau perbaiki ring penerimaan NIC. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Saat terhubung, panel unduhan kalibrasi berwarna hijau mungkin muncul dengan bilah kemajuan per serial: saat kamera dihubungkan ke mesin untuk pertama kalinya, Chloros akan mengambil paket kalibrasi pabrik berukuran ~3,8 MB dari kamera melalui GigE (sekitar 70 detik per kamera). Kamera yang sudah tersimpan dalam cache tidak akan pernah menampilkan panel ini. Lihat [Menghubungkan Kamera](connecting.md).

## Bandwidth: berapa banyak kamera yang dapat ditampung

Kapasitas yang dapat ditangani oleh sebuah array bergantung pada kabel, bukan pada Chloros, sehingga angka perencanaan terdapat dalam manual perangkat keras: **[Perencanaan Bandwidth Array](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Apa yang dilakukan Chloros dengan data tersebut: dialog koneksi menjalankan pemeriksaan jaringan, memprediksi laju bingkai yang dapat dicapai, dan memilih tingkatan yang sesuai. Jika array melebihi kapasitas kabel, sistem akan menolak koneksi daripada membuang paket secara diam-diam — lihat panel hasil proyeksi yang dijelaskan di atas.

## Saat frame hilang

Sebuah kamera dapat tidak terdaftar dalam grup yang dipublikasikan karena dua alasan yang sama sekali berbeda,
dan keduanya memerlukan solusi yang berlawanan. Chloros menghitungnya secara terpisah alih-alih melaporkan satu
angka &quot;tidak lengkap&quot; yang tidak menyebutkan keduanya:

| Apa yang terjadi | Artinya | Di mana memeriksanya |
| --- | --- | --- |
| **Rusak**— frame tiba namun secara struktural rusak | Kehilangan paket GVSP pada jalur jaringan |**Kapasitas saluran**, cincin penerimaan NIC, frame jumbo, switch |
| **Tidak pernah tiba**— tidak ada frame yang datang sama sekali | Kamera tidak memicu, atau tidak ada data yang keluar darinya |**Kabel sinkronisasi M8**, jalur sinkronisasi, apakah setiap anggota dalam status aktif |

Pembagian ini dievaluasi ulang setiap 10 detik saat array melakukan streaming. Di atas 5%, hal ini
dicatat dengan mencantumkan kedua angka tersebut, dan setiap buffer yang rusak dilaporkan pada kali pertama
terjadi per kamera, kemudian dirangkum setiap menit agar sesi yang panjang tetap dapat dibaca.

**Frame yang rusak dengan nol frame yang tidak pernah diterima berarti pemicu dan sinkronisasi kabel sempurna**dan setiap frame yang hilang terjadi di jalur jaringan. Solusinya adalah menurunkan**Wire Budget** dan
menyambungkan ulang.

{% hint style="warning" %}
**Menurunkan laju pemicu tidak membantu mengatasi frame yang rusak.** Pengaturan kecepatan paket
kamera ditetapkan sekali, saat koneksi dibuat. Menurunkan laju pemicu hanya mengubah seberapa sering
burst terjadi, bukan seberapa cepat burst itu sendiri dikirim ke kabel. Pada rig 4 kamera yang diukur,
pengurangan laju pemicu sebesar 5× tidak mengubah apa pun, sementara menurunkan wire budget dari 240 menjadi
200 MB/s menurunkan tingkat kerusakan pada rig yang sama dari 10,4% menjadi nol.
{% endhint %}

Sebuah array yang sedang berjalan tidak dapat merencanakan ulang koneksinya sendiri — putuskan koneksi dan sambungkan kembali agar pemilih waktu koneksi
dapat bekerja sesuai dengan anggaran baru.

### Adaptor jaringan USB dibatasi hingga 200 MB/s

Adaptor Ethernet USB mengiklankan laju koneksi *Ethernet*-nya, tetapi apa yang sebenarnya
dapat dipertahankannya dibatasi oleh bus USB dan drivernya. Dongle USB 10GbE dulunya diklaim
memiliki throughput sekitar 1000 MB/s — angka yang belum pernah diukur sebelumnya — dan mengatur
kecepatan empat kamera berdasarkan ruang kosong yang tidak nyata tersebut menyebabkan 6–18 % frame rusak, sementara array
masih melaporkan laju frame target yang baik. Adaptor yang terhubung melalui USB kini dibatasi pada
**200 MB/s**. Batasan ini bersifat absolut, bukan persentase, karena batasnya terletak pada
bus: adaptor USB 1 GbE menghasilkan sekitar 80 MB/s dan tidak terpengaruh.

Jika host Anda benar-benar lebih cepat daripada batas tersebut, naikkan **Wire Budget** untuk menandakannya.

## Sinkronisasi waktu PTP

*Sinkronisasi* frame berasal dari pemicu perangkat keras; **PTP** (IEEE 1588 PTPv2) menyediakan *cap timestamp* yang seragam di seluruh perangkat. Fitur ini diaktifkan secara default saat array terhubung:

* Backend host **Chloros menjalankan grandmaster PTP**. Kamera LATTICE dan sensor cahaya DAQ-E mengikuti grandmaster tersebut di domain 0, sehingga cap waktu gambar dan spektrum DAQ berada pada satu jam (~1 ms).
* `--no-ptp` (CLI) menonaktifkannya untuk pekerjaan di meja kerja — cap waktu antar kamera kemudian **tidak** sebanding.
* Periksa status sinkronisasi dengan CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Tab &quot;Cameras&quot; sendiri tidak memiliki indikator PTP; permukaan sinkronisasi per-kamera yang ditampilkan di sana adalah **Peran**(Master/Slave) yang hanya dapat dibaca,**Garis Sinkronisasi**, dan tingkatan Kemampuan array. Status PTP DAQ-E ditampilkan di detail sensor pada tab &quot;Light Sensors&quot;.

## Tampilan

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

array langsung Area umpan utama menawarkan dua tata letak (beralih di bilah atas): **tampilan kisi**(setiap ubin adalah sel; seret untuk mengatur ulang urutan saat gembok kisi tidak terkunci) dan**tampilan daftar**(array dengan lebar penuh di bagian atas, satu kamera aktif di bawahnya). Penggeser**Zoom Umpan** mengatur ukuran ubin; di bawah lebar sel 200 px, overlay nama/fps akan tersembunyi secara otomatis.**Mode terpisah** menampilkan satu ubin per kamera. Setiap ubin menampilkan:

* nama kamera (kiri atas),
* **tampilan fps** (kanan atas) — ini adalah *kecepatan pengambilan sebenarnya* kamera yang dilaporkan oleh backend, bukan kecepatan polling pratinjau (pratinjau langsung dibatasi hingga 30 fps terlepas dari kecepatan pengambilan),
* titik status — hijau (sedang streaming) / kuning (sedang dimuat) / merah (kesalahan),
* **spinner frame usang** ketika tidak ada frame baru yang tiba selama 2 detik — hal ini normal selama ~5 detik setelah koneksi terputus atau terhubung kembali, sementara backend menyeimbangkan kembali alokasi bandwidth antar kamera.**Mode gabungan**menampilkan satu ubin komposit: backend melakukan debayering, penskalaan, penyelarasan, penghilangan noise, konversi ke radian per-band (ditambah reflektansi DLS jika sensor cahaya terikat), mengevaluasi ekspresi indeks array, menerapkan LUT, dan mengalirkan hasilnya sebagai MJPEG. Hingga bingkai yang disejajarkan pertama kali ditampilkan, ubin tersebut menjelaskan statusnya: &quot;Mempersiapkan array…&quot;, &quot;Mengkalibrasi penyelarasan…&quot;, &quot;Menunggu bingkai pertama…&quot;, atau — jika anggaran percobaan penyelarasan otomatis (~30 detik) habis — &quot;Diperlukan penyelarasan&quot; dengan tombol**Kalibrasi penyelarasan**.

Fakta berguna tentang mode gabungan:

* Komposit disinkronkan dengan bingkai kamera **utama**. Penargetan AE-ROI dan pengukuran titik pada komposit bersifat tepat untuk kamera utama dan perkiraan untuk kamera bawahan; gunakan**Split View** (pengaturan array → &quot;Tampilkan kamera anggota&quot;) untuk ubin per-kamera yang tepat hingga piksel tanpa membuka koneksi kamera tambahan.
* **Tampilan Lapisan**(pengaturan array; default nonaktif) memungkinkan Anda memilih lapisan latar depan dan latar belakang — kamera anggota mana pun atau**Indeks**. Jika latar depan = Indeks, piksel di luar batas Min/Max LUT akan menampilkan lapisan latar belakang.
* **Resolusi Render** (default 720p) mengatur tinggi siaran langsung *dan* ukuran ekspor komposit yang disimpan. Gambar per kamera selalu diekspor pada resolusi penuh.
* Penjajaran dihitung per sesi dan tidak pernah disimpan — lihat bagian penjajaran di panel pengaturan array untuk sisa RMS dan tombol **Recalibrate**.

## Perekaman: pemantauan vs analisis

Permukaan perekaman array terbagi dengan jelas menjadi **tingkat pemantauan**(merekam apa yang Anda lihat) dan**tingkat analisis** (merekam data mentah, dikalibrasi nanti):

| Alur Kerja | Tingkat | Apa yang disimpan | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Perekaman**(gambar diam) | Analisis | Satu grup bingkai yang disinkronkan per lintasan; file per kamera pada setiap tingkat ekspor yang dipilih (mentah/debayered/radiance/reflectance/pratinjau/indeks) + `.daq` sidecar | Tombol**Capture All** + Pengaturan Capture | `lattice array-capture` |
| **Rekam video indeks** | Pemantauan | Komposisi indeks gabungan langsung seperti yang ditampilkan — 8-bit, resolusi pratinjau, LUT sudah terintegrasi; memerlukan aliran langsung yang terbuka | ● Rekam video indeks (array gabungan) | `lattice array-record` |
| **Raw burst → buat video**| Analisis | Frame sensor mentah pada laju pengambilan penuh + manifest + `.daq`, kemudian direkonstruksi secara offline menjadi video radian / reflektansi / indeks yang dikalibrasi, disinkronkan waktunya dengan pembacaan DAQ | ⦿ Rekam burst mentah →**Buat video** | `lattice array-burst` → `lattice array-build-video` |

Aturan praktis: jika piksel akan digunakan untuk *pengukuran*, gunakan mode penangkapan atau burst (tingkat analisis); jika Anda hanya perlu *menonton atau mendemonstrasikan* apa yang dilihat oleh array, rekam video indeks (tingkat pemantauan).

### Pengaturan Capture (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

Ikon roda gigi di sebelah **Capture All** membuka panel Pengaturan Capture (memerlukan proyek yang terbuka — hasil capture disimpan ke dalamnya):

* **Mode capture**:**Single**(satu kali proses) /**Continuous**(berurutan; dibatasi oleh jumlah penangkapan, default 1, atau durasi, default 10 detik) /**Interval** (timelapse: N penangkapan setiap interval X sebanyak Y total, default 1 setiap 5 detik selama 1 menit).
* **Jenis ekspor per kamera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — semua yang berlaku diaktifkan secara default. Radiance/Reflectance disembunyikan untuk kamera dengan filter RGB;**Reflectance hanya muncul jika kamera memiliki sensor cahaya DAQ** (miliknya sendiri atau diwarisi dari array); Index memerlukan ekspresi indeks yang telah dikonfigurasi.
* **Aligned**(per array, default**aktif**): menyesuaikan ekspor anggota dengan profil penyelarasan array sehingga ekspor terdaftar piksel demi piksel. Raw selalu tetap tidak disesuaikan tetapi membawa transformasi dalam metadata.
* **Perekaman Tercepat** (toggle): hanya data mentah + pembacaan DAQ yang ditetapkan + komposit indeks gabungan gratis, melewati perhitungan kalibrasi pada saat perekaman untuk kecepatan maksimum — bangun kembali radian/reflektansi/indeks nanti dari `.daq` yang disimpan.
* Pilihan tetap tersimpan bersama proyek. Kamera yang disembunyikan atau dijeda akan dilewati.

Setara dengan CLI (titik akhir backend yang sama, semantik yang sama):

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

Kompresi TIFF untuk pengambilan gambar adalah `deflate` (tanpa kehilangan data, default) atau `none` — tabel bendera lengkap, tata letak folder penangkapan, dan aturan pemrosesan ulang terdapat di [Referensi CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Memasangkan sensor cahaya DAQ

Tampilan pratinjau yang dikoreksi reflektansi dan iluminasi memerlukan data cahaya yang mengarah ke bawah dari sensor DAQ (yang terhubung di tab **Sensor Cahaya**):

* **Baris array**di bilah samping menampilkan tombol**&quot;DAQ · on/off&quot;** — *on* jika sensor cahaya tingkat array telah diatur **atau** salah satu kamera anggotanya memiliki sensor sendiri; tooltip-nya mencantumkan dengan tepat sensor mana yang menyuplai kamera mana.
* Tetapkan untuk seluruh array di pengaturan array → menu tarik-turun **Sensor Cahaya Sekitar**→**Sensor Cahaya**. Pilihan ini tetap berlaku untuk proyek, diterapkan ke setiap kamera anggota, dan masing-masing kamera masih dapat menggantikannya dengan sensornya sendiri.
* Baris status di bawahnya melaporkan status saat ini: **Off**→ &quot;Menunggu spektrum pertama…&quot; →**&quot;Aktif — semua kamera dalam array telah dikoreksi pencahayaannya&quot;** → atau, jika tidak ada spektrum baru yang diterima dalam 3 detik terakhir, akan muncul pemberitahuan bahwa pembacaan terakhir masih digunakan (pembacaan tidak pernah kedaluwarsa pada jalur pengambilan gambar).

Dengan sensor yang telah ditetapkan: jenis ekspor Reflectance menjadi tersedia, pratinjau langsung telah dikoreksi pencahayaannya, eksposur otomatis prediktif dapat menggunakan spektrum tersebut, dan setiap pengambilan data reflektansi menulis pembacaan DAQ yang sebenarnya digunakan sebagai **`.daq` sidecar** di samping citra sehingga pengambilan tersebut dapat diproses ulang nanti.

## Opsi `array-connect` CLI

| Bendera | Default | Deskripsi |
| --- | --- | --- |
| `--serials SN1,SN2,…` | mendeteksi otomatis semua kamera LATTICE (membutuhkan ≥2) | **Kamera serial pertama adalah MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Jalur sinkronisasi GPIO. |
| `--target-fps F` | otomatis | Kecepatan pemicu MASTER. |
| `--binning {1,2,4}` | otomatis | Pengelompokan perangkat keras. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | otomatis | Penggantian pengaturan pemilih tingkat sinkronisasi oleh ahli. |
| `--wire-ceiling-mbps MB_PER_S` | terdeteksi otomatis | Anggaran kabel host dalam MB/s — bentuk CLI dari bidang **Anggaran Jaringan**. Kurangi nilainya jika array melaporkan frame yang rusak. Disimpan bersama proyek, sehingga koneksi ulang di kemudian hari akan mengembalikannya. |
| `--no-recommend` | mati | Lewati langkah analisis jaringan. |
| `--no-ptp` | mati | Nonaktifkan PTP (cap waktu antar-kamera tidak dapat dibandingkan). |

`lattice array-list`, `array-status`, dan `array-disconnect` mengelola sesi yang berkelanjutan. Referensi subperintah lengkap, termasuk penyelarasan (`align-calibrate` / `align-apply`) dan alat jaringan, terdapat di [Referensi CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); padanan dari SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) terdapat di [Referensi SDK](../reference/sdk-reference.md). Mulai dari Python, alokasi kabel adalah `connect_array(..., wire_ceiling_mbps=120)`, dan pemisahan antara data yang rusak saat siaran langsung dan yang tidak pernah diterima terdapat pada [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).

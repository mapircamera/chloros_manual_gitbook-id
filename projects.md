# GUI: Proyek

Chloros memungkinkan Anda membuat proyek yang dapat dibuka kembali di masa mendatang. Sebuah proyek adalah folder biasa (di dalam Folder Proyek Anda) yang berisi:

* `project.json` — pengaturan proyek, daftar file, dan preferensi tampilan
* `cameras.json` — kamera dan array yang terhubung saat proyek terbuka, beserta pengaturannya
* `sensors.json` — sensor cahaya DAQ yang terhubung saat proyek terbuka, beserta pengikatan kamera↔sensor
* tangkapan Anda, rekaman `.daq`, dan folder keluaran yang telah diproses

Tidak ada format berkas proyek khusus — folder dan berkas JSON-nya merupakan proyek itu sendiri, yang juga memudahkan untuk menyalin, mengarsipkan, dan menjalankan proyek dari [CLI](CLI.md) atau [Python SDK](api-python-sdk.md).

Kecuali jika Anda mengubahnya di [Pengaturan Proyek](project-settings/project-settings.md), Folder Proyek berada di `Documents\MAPIR\Chloros Projects` dalam folder Dokumen Anda (`~/Documents/MAPIR/Chloros Projects` di Linux).

{% hint style="warning" %}
**Windows: jaga agar folder proyek tetap pendek kecuali jika opsi jalur panjang diaktifkan.**Windows secara default menonaktifkan opsi jalur panjang, sehingga tidak ada jalur file yang boleh lebih panjang dari 260 karakter. Produk yang telah diproses disimpan hingga sekitar 130 karakter di bawah folder proyek, sehingga mulai versi 1.3.0, Chloros akan menolak folder proyek yang terlalu dalam — baik saat Anda membuatnya, membukanya, maupun saat memulai pemrosesan — alih-alih berjalan tanpa menghasilkan apa pun. Pesan kesalahan akan menunjukkan folder proyek terpanjang yang masih sesuai. Gunakan Folder Proyek yang lebih pendek, atau aktifkan jalur panjang Windows (opsi**Aktifkan jalur panjang Windows** pada penginstal, yang dicentang secara default) dan mulai ulang Chloros. Lihat [Memulai Pemrosesan](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Proyek Baru

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Pilih "Proyek Baru" dari menu utama dan masukkan nama unik untuk proyek Anda.

Jika Anda telah menyimpan templat proyek apa pun, menu tarik-turun **Pilih Templat** akan muncul di bawah bidang nama — memilih salah satunya akan memulai proyek baru berdasarkan pengaturan templat tersebut. Templat disimpan dari [Pengaturan Proyek](project-settings/project-settings.md): masukkan nama di kolom "Nama Templat Proyek yang Disimpan" dan klik ikon simpan.

## Buka Proyek

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>"Buka Proyek" menampilkan daftar semua proyek di folder proyek Anda, dengan opsi <strong>Buka Folder Proyek</strong> di bagian bawah</p></figcaption></figure>

Pilih "Buka Proyek" untuk melihat daftar proyek yang ada di Folder Proyek. Jika tidak ada proyek, menu samping sekunder tidak akan terbuka. Anda dapat melihat beberapa proyek yang dibuat melalui antarmuka pengguna (t1, t2, t3) yang tercantum pada foto di atas. Proyek-proyek DATE\_TIME dibuat oleh CLI menggunakan skema penamaan proyek default. Mengklik nama proyek mana pun akan membukanya.

Mengklik tombol "Buka Folder Proyek" akan membuka penjelajah file komputer Anda di jalur proyek. Anda dapat menyesuaikan jalur proyek di [Pengaturan Proyek](project-settings/project-settings.md).

Jika ada file gambar sumber proyek yang telah dipindahkan atau dihapus sejak terakhir kali dibuka, Chloros akan menampilkan kotak dialog yang mencantumkan file mana saja yang hilang, alih-alih membuka kisi kosong.

## Duplikat Proyek

Fitur ini tersedia setelah proyek dibuka. Pilih "Duplikat Proyek" untuk menyalin proyek saat ini dengan nama baru — Chloros akan menyarankan nama kosong berikutnya (misalnya "MyProject (2)") — dan salinan tersebut akan langsung terbuka.

## Tambahkan File

Setelah proyek dibuka, pilih "Tambahkan File" dari menu utama untuk menambahkan file gambar satu per satu ke proyek saat ini. Fitur ini mirip dengan fungsi "Tambahkan" di penjelajah file, tetapi dapat diakses langsung dari menu utama untuk kemudahan.

## Tambahkan Folder

Setelah proyek dibuka, pilih "Tambahkan Folder" dari menu utama untuk menambahkan folder berisi gambar ke proyek saat ini. Anda dapat memilih beberapa folder sekaligus. File duplikat akan diabaikan.

## Mulai / Hentikan Pemrosesan

Setelah berkas ditambahkan ke proyek, opsi "Mulai Pemrosesan" akan tersedia di menu utama. Tindakan ini sama dengan mengklik tombol Putar/Mulai di bilah atas. Selama pemrosesan, opsi menu berubah menjadi "Hentikan Pemrosesan" agar Anda dapat menghentikan alur kerja.

## Hubungkan ke Kamera / Hubungkan ke Sensor Cahaya

Bagian bawah menu utama memiliki dua pintasan perangkat keras, yang tersedia baik saat proyek terbuka maupun tidak:

* **Hubungkan ke Kamera** — membuka [tab Kamera](lattice/) untuk menghubungkan kamera atau array LATTICE.
* **Hubungkan ke Sensor Cahaya** — membuka [tab Sensor Cahaya](daq/) untuk menghubungkan sensor cahaya DAQ.

Menghubungkan perangkat keras saat proyek terbuka akan menyimpannya ke dalam proyek (lihat di bawah). Tanpa proyek, koneksi hanya berlaku untuk sesi tersebut.

{% hint style="info" %}
Item menu **Tambahkan File**,**Tambahkan Folder**, dan**Mulai/Hentikan Pemrosesan** hanya terlihat atau diaktifkan saat proyek terbuka dan file telah ditambahkan. Item-item ini menyediakan akses cepat ke tindakan yang juga tersedia melalui bilah samping Penjelajah File dan tombol-tombol di bagian atas.
{% endhint %}

## Proyek mengingat perangkat keras Anda

Baru di versi 1.2.0: proyek menyimpan perangkat keras yang Anda sambungkan selama proyek tersebut terbuka. Kamera dan susunan (beserta pengaturan per kamera, nama, warna, dan tata letak kisi) disimpan dalam file `cameras.json`, sedangkan sensor cahaya (beserta nama, warna, dan pengikatan kamera) disimpan dalam file `sensors.json` — secara otomatis, saat Anda bekerja.

Saat Anda **membuka kembali** sebuah proyek, Chloros tidak langsung mengakses perangkat keras apa pun. Setiap bagian akan terhubung kembali saat pertama kali Anda membuka tab yang mengelolanya:

* Membuka tab **Kamera** akan menghubungkan kembali kamera dan array yang tersimpan serta menerapkan kembali pengaturan yang tersimpan.
* Membuka tab **Sensor Cahaya**akan menghubungkan kembali sensor DAQ yang tersimpan dan menerapkan kembali pengaturan yang tersimpan, termasuk penutupnya. Sensor yang disetel ke**None (sensor telanjang)** di menu Cap-nya tetap dalam keadaan telanjang. Proyek yang disimpan oleh Chloros versi sebelumnya mencatat sensor telanjang sebagai tidak ada pilihan, sehingga sensor tersebut kembali dengan koreksi kosinus sinar matahari; atur kembali ke None dan simpan proyek.

Dengan cara ini, membuka proyek hanya untuk menelusuri atau mengekspor gambar tidak akan pernah mengaktifkan kamera ke mode streaming. Jika perangkat yang disimpan tidak dapat ditemukan saat tabnya dibuka, sebuah dialog akan memberi tahu Anda perangkat mana saja yang tidak tersedia sehingga Anda dapat menghubungkannya kembali atau menghapusnya.

## Rekaman DAQ dan berkas .daq dalam sebuah proyek

* Rekaman `.daq` yang dibuat saat proyek terbuka (dari tab Light Sensors atau selama proses perekaman) akan **secara otomatis ditambahkan ke proyek**.
* Berkas `.daq` yang diimpor, serta semua rekaman proyek, tercantum di bagian **Sensor Cahaya DAQ** pada [Pengaturan Proyek](project-settings/project-settings.md), masing-masing dengan profil koreksi cap-nya.
* Selama pemrosesan, berkas `.daq` proyek menyediakan iluminasi ke bawah untuk produk reflektansi — lihat [Format Gambar Keluaran](output-image-formats.md).

## Menjalankan proyek yang disimpan tanpa antarmuka grafis

Proyek yang disimpan dapat dijalankan tanpa antarmuka grafis:

* **CLI**: `chloros-cli project open / connect / capture / sensor / align / run` beroperasi berdasarkan jalur folder proyek — lihat [Referensi CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` mengembalikan handle proyek; `connect_all()` mengaktifkan setiap kamera, array, dan sensor cahaya yang telah disimpan beserta pengaturannya (sensor cahaya melalui backend Chloros yang sedang berjalan) — lihat [Referensi SDK](reference/sdk-reference.md).

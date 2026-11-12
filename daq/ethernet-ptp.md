# Jaringan & Sinkronisasi Waktu DAQ-E

> Pengaturan jaringan fisik untuk sensor — pemasangan kabel, PoE, penetapan alamat IP, dan pengaturan jaringan perangkat itu sendiri — dijelaskan dalam **[panduan pengguna DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Halaman ini membahas sisi Chloros: koneksi, sinkronisasi waktu, dan langkah-langkah yang harus diambil jika proses pencarian perangkat tidak menemukan apa pun.

DAQ-E adalah anggota keluarga DAQ yang menggunakan Ethernet: diberi daya melalui PoE, terdeteksi melalui mDNS (layanan `_daq-e._tcp`), dan dapat dialamatkan dengan nama host yang berasal dari ID sensornya — `daq-e-<6 hex>.local`, misalnya `daq-e-def330.local`. Halaman ini membahas cara perangkat ini mentransfer data di jaringan dan cara perangkat ini berpartisipasi dalam sinkronisasi waktu PTP.

## Mode transmisi

| Mode | Titik akhir | Pengguna | Catatan |
| --- | --- | --- | --- |
| **Multicast** (default) | UDP `239.10.10.10:5002` | Berapa pun jumlah perangkat di LAN yang sama akan menerima aliran yang sama | Setiap datagram diverifikasi menggunakan CRC-16/CCITT |
| **Raw** | Port TCP `5000` | Tepat satu klien (eksklusif) | Kompatibel byte-per-byte dengan DAQ-U |

Chloros menggunakan multicast secara default, yang memungkinkan GUI, CLI, dan SDK untuk memantau satu sensor secara bersamaan.

## Persyaratan jaringan

* **Domain siaran yang sama.** Mesin yang menjalankan Chloros harus berada di segmen jaringan L2 yang sama dengan sensor — penemuan mDNS tidak menembus router.
* **Firewall.** Di Windows, penginstal menambahkan aturan firewall **Chloros Backend (UDP-In)**untuk backend, pada setiap profil jaringan, yang mencakup data DAQ-E (UDP 5002), mDNS (UDP 5353), dan PTP (UDP 319/320), serta aturan**Chloros Backend (TCP-In)**, yang dibatasi pada subnet lokal, untuk koneksi yang dibuat DAQ-E kembali ke komputer selama pembaruan firmware. Di Linux, `.deb` membuka port UDP di `ufw` atau `firewalld` ketika firewall tersebut aktif pada saat instalasi. Firewall pihak ketiga, atau yang diaktifkan kemudian, memerlukan izin untuk port yang sama.
* **Daya PoE, tanpa LED status.** DAQ-E tidak memiliki LED sendiri — periksa daya melalui indikator link/PoE pada switch atau port injektor, dan tunggu beberapa detik setelah dinyalakan agar perangkat dapat boot dan bergabung dengan jaringan.

## Menghubungkan

**GUI:** Tab Sensor Cahaya → Hubungkan Sensor → Tipe Perangkat "DAQ-E (Ethernet)". Proses pencarian hanya berjalan selama dialog koneksi muncul di layar (penelusuran mDNS ditambah pemindaian ARP di Windows), berulang setiap 15 detik; tombol Refresh akan melakukan pemindaian ulang secara langsung. Sensor yang terdeteksi akan muncul di menu tarik-turun; sensor pertama yang terdeteksi akan dipilih secara otomatis.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend sedang berjalan):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

DAQ-E yang sudah ada di pool dikenali baik melalui nama host maupun alamat IP-nya: menghubungkannya kembali menggunakan alamat lain akan mengembalikan entri pool yang sudah ada, bukan membuka unit tersebut untuk kedua kalinya.

### Host dengan beberapa NIC dan koneksi pertama setelah boot

Pada host dengan lebih dari satu antarmuka jaringan aktif, **koneksi pertama** `pool-connect --eth` setelah boot bisa saja kosong meskipun sensor dalam kondisi baik — proses pencarian penemuan mungkin melewatkan antarmuka tempat sensor berada saat cache ARP masih belum terisi. Solusi yang andal adalah melewati proses pencarian dan memberikan alamat secara eksplisit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` menerima nama host mDNS atau alamat IP, selalu menargetkan sensor yang tepat, dan merupakan bentuk yang direkomendasikan untuk skrip dan instalasi tanpa antarmuka pengguna. Di antarmuka grafis (GUI), gunakan tombol Refresh pada dialog koneksi dan biarkan proses pemindaian ulang berlangsung.

## Pengaturan perangkat dan firmware

Sensor itu sendiri menyimpan pengaturan jaringannya: DHCP (pengaturan pabrik, yang akan beralih ke alamat link-local 169.254.x.x jika tidak ada server DHCP yang merespons) atau alamat statis. Sensor juga menyimpan pengaturan apakah akan mulai melakukan streaming saat dinyalakan. Ubah pengaturan tersebut di antarmuka grafis (ikon roda gigi pengaturan sensor, **Jaringan**:**Ubah…**,**Streaming saat dinyalakan**,**Mulai ulang sensor**; lihat [Tab DAQ](gui.md)), atau melalui CLI dan SDK dengan perintah `pool-set-network`, `pool-set-auto-stream` dan `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; lihat [Referensi CLI](../reference/cli-reference.md#daq-e-device-settings)). Sensor akan restart untuk menerapkan perubahan jaringan, dan Chloros akan menghubungkannya kembali. Alamat statis harus berada di jaringan komputer yang menjalankan Chloros: Chloros akan menolak alamat yang tidak dapat dijangkau setelah restart. ID sensor dan nama host `daq-e-<id>.local`-nya tidak pernah berubah. Baru di versi 1.3.0.

**Pembaruan firmware terintegrasi ke dalam antarmuka pengguna grafis (GUI).** Ketika DAQ-E yang terhubung menjalankan firmware yang lebih lama daripada gambar firmware yang disertakan dalam build Chloros Anda, baris sensornya akan menampilkan ikon **Pembaruan Tersedia** berwarna kuning, dan jendela pengaturan roda gigi menawarkan tombol "Perbarui ke \<version\>". Pembaruan akan dikirim melalui jaringan dalam waktu sekitar 30 detik; sensor akan restart dan terhubung kembali secara otomatis, dan jika transfer terputus, firmware saat ini akan tetap utuh. Dari CLI atau SDK, serta di Linux yang tidak memiliki antarmuka grafis (GUI), jalankan perintah `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sinkronisasi waktu PTP

Firmware DAQ-E v1.2.0+ berpartisipasi dalam IEEE 1588 PTPv2 sebagai jam biasa (hanya slave). **Backend host Chloros adalah grandmaster PTP** — setiap DAQ-E dan setiap kamera LATTICE di jaringan LAN menjadi slave terhadapnya di domain 0, menjaga agar semua cap waktu perangkat tetap dalam toleransi ~1 ms. Jam bersama itulah yang membuat pembacaan DAQ dapat disesuaikan cap waktunya dengan eksposur kamera (lihat [Perekaman & Format .daq](recording.md)).

Periksa sinkronisasi dari CLI:

| Perintah | Menampilkan |
| --- | --- |
| `chloros-cli time-sync status` | Status grandmaster host, prioritas BMCA, identitas jam, dan `health`: `OK`, atau `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` dengan `health_detail` yang menjelaskan mengapa paketnya tidak sampai ke jaringan |
| `chloros-cli time-sync peers` | Semua slave terdeteksi (sensor DAQ-E + kamera LATTICE) |
| `chloros-cli time-sync cameras` | Status kesehatan PTP per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Mulai ulang proses grandmaster |

Di antarmuka pengguna grafis (GUI), jendela pengaturan DAQ-E menampilkan baris **PTP Sync** secara langsung yang menunjukkan status PTP sensor saat ini.**DAQ-E dengan PTP yang dimatikan akan tetap dalam keadaan mati.** Unit menyimpan pengaturan PTP (aktif/nonaktif) di memori internalnya, sehingga unit yang PTP-nya dimatikan akan tetap dalam keadaan mati setelah koneksi ulang atau siklus daya, dan cap waktunya tidak disinkronkan dengan PTP. `chloros-cli daq pool-list` menandai unit tersebut (`!! PTP is switched OFF on this unit …`) dan mencetak perintah yang memperbaikinya:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Chloros tidak pernah mengaktifkan kembali unit secara otomatis, karena status mati bisa jadi disengaja.

Rincian untuk pengguna yang memerlukan penyelarasan ketat:

* Setiap datagram yang dikirimkan membawa bidang bendera; **bit ke-2 disetel pada frame yang cap waktunya disinkronkan dengan PTP**. Jalur pemrosesan yang memerlukan penyelarasan ketat antara kamera dan DAQ harus menggunakan bit tersebut sebagai kondisi penyaringan.
* Sebelum melakukan pengambilan data yang disinkronkan, pastikan sensor tersebut muncul di `chloros-cli time-sync peers` dan bahwa `chloros-cli daq pool-list` tidak menandai PTP-nya sebagai dimatikan. (Perangkat lunak internal MAPIR yang terintegrasi langsung dengan perangkat keras juga dapat membatasi perekaman berdasarkan kunci PTP dengan bendera `--wait-ptp` yang menunggu hingga 15 detik hingga sensor mencapai status SLAVE; perangkat lunak tersebut tidak termasuk dalam CLI yang disertakan.)
* Saat PTP sedang aktif sebagai slave, sensor menolak pengiriman clock manual ("PTP menyediakan clock"). Hal ini sesuai desain — percayalah pada PTP.

## Catatan Linux

* **Port PTP tanpa hak akses root.** PTP mengikat port UDP 319/320. `.deb` menulis ke `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), sehingga backend mengikat port-port tersebut saat berjalan sebagai pengguna Anda. Mulai versi 1.3.0, paket ini tidak menetapkan kemampuan berkas (`setcap`) pada backend dan tidak memerlukan `libcap2-bin`; jangan tambahkan kemampuan tersebut sendiri, karena backend yang telah diekstrak tidak akan berjalan. Jika PTP tidak dapat mengikat port 319, lihat [Instalasi Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson tanpa layar / host Linux:** paket ini menginstal layanan backend tetapi tidak mengaktifkan satupun. Untuk PTP yang selalu aktif (dan ketersediaan DAQ) tanpa perlu menjalankan perintah, aktifkan layanan tersebut sebagai pengguna Anda:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Tanpa itu, PTP hanya berjalan selama ada yang telah memulai layanan backend — perintah `chloros-cli` atau skrip SDK. Lihat [Instalasi Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) untuk akun root `chloros-backend.service` dan cara beralih darinya.

## Pemecahan Masalah: "Tidak ditemukan perangkat DAQ-E"

| Periksa | Detail |
| --- | --- |
| Daya | Tidak ada LED yang menyala pada sensor — periksa indikator PoE dan link pada port switch/injector; tunggu beberapa detik setelah dinyalakan |
| Domain siaran | Host dan sensor berada di segmen L2 yang sama; mDNS tidak merutekan |
| Firewall | Windows: aturan **Chloros Backend (UDP-In)** pada penginstal mengizinkan aliran DAQ-E (UDP 5002), mDNS (5353), dan PTP (319/320) masuk; firewall pihak ketiga memerlukan pengaturan yang sama. Linux: `.deb` membuka port-port ini di `ufw` / `firewalld` hanya jika firewall aktif saat instalasi — izinkan sendiri jika firewall diaktifkan kemudian |
| Host dengan beberapa NIC | Penemuan menjelajahi mDNS dengan cadangan ARP; `--eth-host <ip-or-hostname>` melewati proses penemuan dan terhubung langsung |
| Pemindaian ulang GUI | Penemuan hanya berjalan saat dialog koneksi terbuka; gunakan Refresh |

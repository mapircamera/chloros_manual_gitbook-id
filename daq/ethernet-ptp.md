# Jaringan &amp; Sinkronisasi Waktu DAQ-E

> Pengaturan jaringan fisik untuk sensor — pemasangan kabel, PoE, penetapan alamat IP, dan pengaturan jaringan perangkat itu sendiri — dijelaskan dalam **[panduan pengguna DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Halaman ini membahas sisi Chloros: koneksi, sinkronisasi waktu, dan langkah-langkah yang harus diambil jika proses pencarian perangkat tidak menemukan apa pun.

DAQ-E adalah anggota keluarga DAQ yang menggunakan Ethernet: diberi daya melalui PoE, terdeteksi melalui mDNS (layanan `_daq-e._tcp`), dan dapat dialamatkan dengan nama host yang berasal dari ID sensornya — `daq-e-<6 hex>.local`, misalnya `daq-e-def330.local`. Halaman ini membahas cara perangkat ini mentransfer data di jaringan dan cara perangkat ini berpartisipasi dalam sinkronisasi waktu PTP.

## Mode transmisi

| Mode | Titik akhir | Pengguna | Catatan |
| --- | --- | --- | --- |
| **Multicast** (default) | UDP `239.10.10.10:5002` | Berapa pun jumlah perangkat di LAN yang sama akan menerima aliran yang sama | Setiap datagram divalidasi dengan CRC-16/CCITT |
| **Raw** | Port TCP `5000` | Tepat satu klien (eksklusif) | Kompatibel byte-per-byte dengan DAQ-U |

Chloros menggunakan multicast secara default, yang memungkinkan GUI, CLI, dan SDK memantau satu sensor secara bersamaan.

## Persyaratan jaringan

* **Domain siaran yang sama.** Mesin yang menjalankan Chloros harus berada di segmen jaringan L2 yang sama dengan sensor — penemuan mDNS tidak melewati router.
* **Peringatan firewall Windows: terima saja.** Saat pertama kali Chloros mengikat soket multicast, Windows Defender akan menanyakan izin sekali. Mengizinkannya mencakup data DAQ-E (UDP 5002), mDNS (UDP 5353), dan PTP (UDP 319/320). Pada Linux, proses ini berlangsung tanpa pemberitahuan.
* **Daya PoE, tanpa LED status.** DAQ-E tidak memiliki LED sendiri — periksa daya melalui indikator link/PoE pada switch atau port injektor, dan tunggu beberapa detik setelah dinyalakan agar perangkat dapat melakukan booting dan bergabung dengan jaringan.

## Menghubungkan

**GUI:** Tab Sensor Cahaya → Hubungkan Sensor → Jenis Perangkat &quot;DAQ-E (Ethernet)&quot;. Proses pencarian hanya berjalan saat dialog koneksi muncul di layar (penelusuran mDNS ditambah pemindaian ARP pada Windows), berulang setiap 15 detik; tombol Refresh akan melakukan pemindaian ulang secara langsung. Sensor yang terdeteksi akan muncul di menu dropdown; sensor pertama yang terdeteksi akan dipilih secara otomatis.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend sedang berjalan):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Host dengan beberapa NIC dan koneksi pertama setelah boot

Pada host dengan lebih dari satu antarmuka jaringan aktif, **koneksi pertama** `pool-connect --eth` setelah boot bisa saja kosong meskipun sensor dalam kondisi sehat — proses pencarian mungkin melewatkan antarmuka tempat sensor berada saat cache ARP masih belum terisi. Solusi yang dapat diandalkan adalah melewati proses penelusuran dan memberikan alamat secara eksplisit:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` menerima nama host mDNS atau alamat IP, selalu menargetkan sensor yang tepat, dan merupakan bentuk yang direkomendasikan untuk skrip dan instalasi tanpa antarmuka grafis. Di antarmuka GUI, gunakan tombol Refresh pada dialog koneksi dan biarkan siklus pemindaian ulang berlangsung.

## Pengaturan perangkat dan firmware

Sensor itu sendiri menyimpan pengaturan jaringan — IP statis vs. DHCP + alamat link-local, nama perangkat, auto-stream saat boot, serta kata sandi OTA. Pengaturan di sisi perangkat ini tidak ditampilkan sebagai perintah di CLI yang dikirimkan; pengaturan tersebut dikelola melalui antarmuka pengguna (GUI) Chloros di mana pengaturan tersebut ditampilkan, atau dengan dukungan MAPIR.

**Pembaruan firmware terintegrasi ke dalam antarmuka pengguna (GUI).**Ketika DAQ-E yang terhubung menjalankan firmware yang lebih lama daripada gambar yang disertakan dalam build Chloros Anda, baris sensornya akan menampilkan ikon**Pembaruan Tersedia** berwarna kuning, dan jendela pengaturan roda gigi menawarkan<version>

tombol</version> &quot;Perbarui ke<version>

&quot;. Pembaruan dikirim melalui jaringan dalam waktu sekitar 30 detik; sensor akan restart dan terhubung kembali secara otomatis, dan jika transfer terputus, firmware saat ini akan tetap utuh.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sinkronisasi waktu PTP

Firmware DAQ-E v1.2.0+ berpartisipasi dalam IEEE 1588 PTPv2 sebagai jam biasa (hanya slave). **Backend host Chloros bertindak sebagai grandmaster PTP** — setiap DAQ-E dan setiap kamera LATTICE di jaringan LAN mengikuti jam tersebut di domain 0, sehingga menjaga semua cap waktu perangkat tetap dalam toleransi ~1 ms. Jam bersama inilah yang membuat pembacaan DAQ dapat disesuaikan cap waktunya dengan eksposur kamera (lihat [Perekaman &amp; Format .daq](recording.md)).

Periksa sinkronisasi dari CLI:

| Perintah | Menampilkan |
| --- | --- |
| `chloros-cli time-sync status` | Status grandmaster host, prioritas BMCA, identitas jam |
| `chloros-cli time-sync peers` | Semua slave yang terdeteksi (sensor DAQ-E + kamera LATTICE) |
| `chloros-cli time-sync cameras` | Status kesehatan PTP per kamera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Mulai ulang proses grandmaster |

Di antarmuka pengguna grafis (GUI), jendela pengaturan DAQ-E menampilkan baris **PTP Sync** secara langsung yang menunjukkan status PTP sensor saat ini.

Rincian untuk pengguna yang memerlukan penyelarasan ketat:

* Setiap datagram yang dikirimkan membawa bidang bendera; **bit ke-2 disetel pada frame yang cap waktunya disinkronkan dengan PTP**. Pipa yang memerlukan penyelarasan kamera/DAQ yang ketat harus menggunakan bit tersebut sebagai pemicu.
* Sebelum melakukan perekaman yang disinkronkan, pastikan sensor tersebut muncul di `chloros-cli time-sync peers`. (Alat perangkat keras langsung internal MAPIR juga dapat membatasi perekaman pada kunci PTP dengan bendera `--wait-ptp` yang menunggu hingga 15 detik hingga sensor mencapai status SLAVE; alat tersebut tidak termasuk dalam CLI yang dikirimkan.)
* Saat PTP sedang aktif sebagai slave, sensor menolak dorongan clock manual (&quot;PTP menyediakan clock&quot;). Hal ini sesuai desain — percayalah pada PTP.

## Catatan Linux

* **PTP memerlukan `libcap2-bin` pada saat instalasi.** Proses postinst `.deb` memberikan izin kepada `cap_net_bind_service=+ep` pada `/usr/lib/chloros/chloros-backend` sehingga dapat mengikat port PTP 319/320 tanpa hak akses root. Jika `libcap2-bin` tidak ada, langkah tersebut akan dilewati dan PTP akan gagal dimulai. Perbaikan:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi tanpa antarmuka grafis:** pada instalasi pertama, unit systemd `chloros-backend.service` dihasilkan tetapi tidak diaktifkan. Untuk PTP yang selalu aktif (dan ketersediaan DAQ) tanpa antarmuka grafis:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Tanpa unit tersebut, PTP hanya berjalan selama antarmuka grafis (GUI) Chloros terbuka.

## Pemecahan Masalah: &quot;Tidak ditemukan perangkat DAQ-E&quot;

| Periksa | Detail |
| --- | --- |
| Daya | Tidak ada LED yang menyala pada sensor — periksa indikator PoE dan link pada port switch/injector; tunggu beberapa detik setelah dinyalakan |
| Domain siaran | Host dan sensor berada di segmen L2 yang sama; mDNS tidak merutekan |
| Firewall Windows | Terima permintaan Defender saat pertama kali dijalankan (UDP 5002, 5353, 319/320) |
| Host multi-NIC | Pencarian pertama setelah boot mungkin tidak mendeteksi sensor — sambungkan dengan `--eth-host <ip-or-hostname>` |
| Pemindaian ulang GUI | Pencarian hanya berjalan saat dialog koneksi terbuka; gunakan Refresh |</version>

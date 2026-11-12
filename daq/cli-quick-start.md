# Panduan Cepat CLI (pool-*)

Drive `chloros-cli` yang disertakan mengendalikan sensor DAQ melalui rangkaian perintah **`daq pool-*`** — klien HTTP ringan yang mengoperasikan sensor melalui kumpulan sensor persisten di backend Chloros. Backend mengelola transportasi data, sehingga antarmuka grafis (GUI), antarmuka baris perintah (CLI), dan skrip SDK semuanya berbagi satu handle aktif, bukan berebut port. Semua yang dibutuhkan pelanggan dapat diakses melalui `pool-*`: terhubung, melakukan streaming, merekam berkas `.daq` yang telah dikalibrasi, dan mengganti profil kap.

`pool-*` adalah antarmuka DAQ yang didukung dalam rilis yang telah diterbitkan. `chloros-cli daq --help` mencantumkan subperintah `pool-*`, dan `daq discover` / `daq list` menjalankan pemindaian yang sama seperti `pool-discover`. Jika subperintah DAQ perangkat keras langsung tidak tersedia, program akan keluar dengan pesan kesalahan eksplisit yang menyebutkan paket yang hilang dan mengarahkan Anda kembali ke `pool-*` — tidak ada yang gagal tanpa pemberitahuan. (`pip install chloros-sdk` juga tidak menyediakan perintah perangkat keras langsung.)

***

## Persyaratan

* **Backend Chloros harus sedang berjalan** — perintah `pool-*` adalah klien HTTP, bukan driver perangkat keras. Di Windows, jalankan aplikasi desktop Chloros (aplikasi ini akan meluncurkan backend). Pada Linux/Jetson tanpa antarmuka grafis, aktifkan layanan sebagai pengguna Anda: `sudo systemctl enable --now chloros-backend@$USER.service` (lihat [Instalasi Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Masuk ke Chloros+ (tingkatan berbayar)**: jalankan `chloros-cli login` terlebih dahulu. Penegakan dilakukan di sisi server — tanpa login, perintah akan gagal dengan `401 AUTH_REQUIRED`; pada tingkatan gratis (Iron), perintah akan gagal dengan `403 PLAN_UPGRADE_REQUIRED`.
* Perintah-perintah ini secara default menargetkan `http://127.0.0.1:5000`; keluarga perintah `daq pool-*` akan mengikuti variabel lingkungan `CHLOROS_BACKEND_URL` jika backend Anda berjalan di tempat lain.

***

## Sesi lima menit

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — membuka sensor di kolam

| Varian | Arti |
| --- | --- |
| `daq pool-connect` | Deteksi cerdas: temukan DAQ apa pun di mesin ini. |
| `daq pool-connect --port PORT` | DAQ-U pada port serial tertentu (misalnya `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M melalui BLE, MAC dipindai secara otomatis. |
| `daq pool-connect --mac MAC` | DAQ-M pada MAC BLE yang diketahui (menyiratkan `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E pada nama host atau IP yang diketahui — **jalur yang andal**. |
| `daq pool-connect --eth` | DAQ-E dengan penemuan otomatis (mDNS, dengan cadangan ARP). Lihat peringatan di bawah ini. |

Bendera penyetelan, semuanya opsional:

| Bendera | Arti |
| --- | --- |
| `--integration-time MS` / `-t MS` | Waktu integrasi manual dalam milidetik. |
| `--frame-avg N` / `-f N` | Jumlah frame yang dirata-ratakan per spektrum yang dilaporkan. |
| `--no-ae` | Menonaktifkan eksposur otomatis (AE aktif secara default). |
| `--no-stream` | Terhubung tanpa memulai streaming (lanjutkan nanti dengan `pool-stream --start`). |
| `--cap-id CAP` | Profil koreksi batas; default backend adalah `sunshine_cosine`. Lihat [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Menghubungkan DAQ-E yang sudah ada di pool dengan alamat lain — nama host `daq-e-<id>.local` atau IP-nya — akan mengembalikan entri pool yang sudah ada, bukan membuka unit tersebut untuk kedua kalinya.

{% hint style="warning" %}
**Peringatan penemuan otomatis `--eth`.** Pada host multi-homed (memiliki lebih dari satu antarmuka jaringan aktif), *`pool-connect --eth`* pertama setelah boot dapat tidak menemukan apa pun meskipun sensor dalam kondisi sehat — proses penelusuran penemuan dapat melewatkan antarmuka sensor saat cache ARP masih kosong. Jika `--eth` tidak menemukan apa pun, coba lagi, atau lewati proses penemuan sepenuhnya dengan menggunakan `--eth-host <ip-or-hostname>`, yang merupakan jalur yang lebih andal pada mesin multi-homed. Nama host DAQ-E adalah `daq-e-<id>.local` (misalnya `daq-e-def330.local`); alamat IP-nya yang biasa juga bisa digunakan.
{% endhint %}

## `pool-discover` — pemindaian tanpa koneksi

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Memindai sensor yang dapat Anda sambungkan dan menampilkan alamat untuk disambungkan, tanpa benar-benar menyambung. `--only` mengambil subset yang dipisahkan dengan koma dari `usb,ble,eth`; `--timeout SEC` mengatur waktu pemindaian per transportasi (default 5 detik, dibatasi antara 1–20 detik). Sensor yang sudah terbuka di pool tidak tercantum (DAQ-M yang terhubung berhenti mengiklankan) — gunakan `pool-list` untuk sensor tersebut. Pada build yang telah dirilis, `chloros-cli daq discover` dan `daq list` menjalankan pemindaian yang sama ini.

## `pool-list` — lihat apa yang terhubung

Menampilkan setiap sensor di kolam backend, termasuk `sensor_id` yang dibutuhkan oleh setiap perintah lainnya:

| Model | Bentuk `sensor_id` | Contoh |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktet dengan tanda hubung | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Sebuah DAQ-E yang PTP-nya telah dimatikan (pengaturan disimpan di unit) ditandai dalam `pool-list` dengan `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` dan perintah untuk mengaktifkannya kembali — lihat [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — membaca bingkai spektrum

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Mengembalikan bingkai terbaru, atau bingkai-bingkai terbaru `--recent N`; `--json` menghasilkan keluaran yang dapat dibaca mesin untuk keperluan skrip. Bingkai-bingkai tersebut merupakan irradiansi spektral (W/m²/nm) yang telah dikalibrasi secara radiometrik pada kisi 135 titik, 340–1010 nm, dengan profil penutup sensor yang telah diterapkan. Untuk nilai irradiansi kuantitatif, hitung rata-rata setidaknya 15 detik frame — ini merupakan karakteristik instrumen, bukan cacat.

## `pool-stream` — jeda atau lanjutkan streaming

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — merekam berkas `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Bendera | Default | Arti |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Durasi perekaman dalam detik; `0` berarti jalankan hingga Anda menjalankan perintah `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Direktori keluaran. Dengan backend di komputer ini, jalur relatif diselesaikan berdasarkan folder tempat Anda menjalankan perintah; dengan backend jarak jauh, jalur tersebut diselesaikan **di mesin yang menjalankan backend**. |
| `--device-name NAME` | — | Label yang disimpan bersama rekaman. |
| `--stop` | — | Hentikan rekaman yang sedang berlangsung. |
| `--confirm-after SEC` | `20` | Setelah dimulai, tunggu hingga selama ini agar baris pertama ditulis, dan berikan peringatan keras (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) jika tidak ada yang masuk. `0` menonaktifkan pemeriksaan tersebut. |
| `--heartbeat SEC` | `30` | Dengan `--duration`, cetak jumlah baris setiap SEC detik dan berikan peringatan jika jumlahnya berhenti bertambah. `0` menonaktifkannya. |

{% hint style="info" %}
Perekaman terjadi di backend, sehingga berkas `.daq` disimpan di sistem berkas **mesin backend** — secara default di `~/Documents/DAQ Live View/` di sana, tidak harus di tempat Anda menjalankan CLI. Nama berkas mencakup ID sensor dan cap waktu.
{% endhint %}

## `pool-set-cap` — tentukan tutup yang terpasang

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID penutup menentukan profil koreksi yang diukur di pabrik dan diterapkan pada setiap spektrum, dan **harus sesuai dengan penutup yang terpasang secara fisik pada sensor** — baik sensor maupun perangkat lunak tidak dapat mendeteksi penutup tersebut secara otomatis, dan pemilihan ini dicatat dalam setiap berkas `.daq`. Nilai default di mana pun adalah `sunshine_cosine` (setiap DAQ dikirimkan dengan penutup korektor kosinus Sunshine terpasang, dengan atenuasi ~12× sesuai desain — perubahan penutup yang tidak dinyatakan akan menyebabkan koreksi spektrum yang salah dengan faktor sekitar itu).

| `--cap-id` | Tersedia pada |
| --- | --- |
| `sunshine_cosine` (default) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Hanya DAQ-U |
| `none` | Hanya DAQ-E — lihat catatan |

ID penutup di luar set sensor akan ditolak saat koneksi dengan pesan kesalahan yang jelas. `none` (DAQ-E) berarti penutup telah dilepas secara fisik — profil geometri pabrik tetap diterapkan untuk difuser kaca yang tersembunyi pada DAQ-E, sehingga ini bukan operasi kosong (no-op), dan DAQ-E tanpa penutup merupakan konfigurasi meja kerja, bukan konfigurasi lapangan yang didukung. (DAQ-U tanpa penutup adalah benar-benar tanpa penutup dan tidak memerlukan profil koreksi sama sekali; DAQ-M digunakan bersama penutup Sunshine-nya.)

## `pool-disconnect` — melepaskan sensor

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` melepaskan sensor siapa pun yang membukanya, sehingga juga membebaskan sensor yang ditinggalkan oleh skrip yang crash di dalam pool (perekaman pada sensor tersebut harus selesai terlebih dahulu). Sensor yang tidak ada di pool tidak memiliki apa pun untuk dilepaskan: perintah ini mencetak `<sensor-id>: already released (not in the pool)` dan keluar dengan pesan `0`. Pelepasan yang ditolak atau tidak dikonfirmasi oleh backend akan keluar dengan pesan `1`.

## `pool-enable-ptp` — mengaktifkan kembali PTP pada DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Sebuah DAQ-E menyimpan pengaturan PTP (on/off) di memorinya sendiri, sehingga unit yang PTP-nya telah dimatikan akan tetap mati — dan cap waktunya tidak disinkronkan dengan PTP — hingga diaktifkan kembali. `pool-list` menandai unit tersebut dan menampilkan perintah ini untuknya. `Chloros` tidak pernah mengaktifkan kembali unit secara otomatis, karena status "mati" bisa jadi disengaja. Baru di versi 1.3.0.

## Pengaturan perangkat DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` menampilkan firmware sensor (dan apakah rilis Chloros ini memiliki versi yang lebih baru), alamatnya dan cara mendapatkannya, pengaturan jaringan yang tersimpan jika berbeda dari yang sedang berjalan, streaming saat dinyalakan, dan PTP. `pool-set-network` me-restart sensor untuk menerapkan perubahan dan menghubungkannya kembali dengan pengaturan yang sebelumnya; `--no-reboot` hanya menyimpannya. Alamat statis harus ada di jaringan komputer ini, atau Chloros akan menolaknya; `--force` mengesampingkan hal tersebut, hanya untuk komputer yang akan diberi alamat baru atau dialihkan ke sana. Restart ditolak saat sensor sedang merekam. `pool-update-firmware` adalah **Update to \<version\>** </version\>pada antarmuka grafis (GUI) <version\>untuk baris perintah, serta cara memperbarui DAQ-E di Linux yang tidak memiliki antarmuka grafis. Baru di versi 1.3.0.

***

## Ringkasan perintah

| Perintah | Tujuan |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Membuka sensor di kolam backend. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Memindai sensor tanpa terhubung (mencari alamat MAC BLE DAQ-M). |
| `daq pool-list` | Menampilkan semua sensor di pool beserta `sensor_id`-nya. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N frame spektrum terkalibrasi terbaru. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Lanjutkan / jeda streaming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Mulai / hentikan perekaman `.daq` (di sisi backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Mengganti profil koreksi batas saat berjalan. |
| `daq pool-disconnect --sensor-id ID [--all]` | Melepaskan satu sensor, atau semuanya. |
| `daq pool-enable-ptp --sensor-id ID` | Mengaktifkan kembali PTP untuk DAQ-E yang PTP-nya dalam keadaan mati. |
| `daq pool-info --sensor-id ID [--json]` | Firmware, alamat, pengaturan jaringan yang disimpan, streaming saat dinyalakan, dan PTP dari sebuah DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP atau alamat statis; me-restart dan menghubungkan ulang sensor. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Streaming saat dinyalakan. |
| `daq pool-reboot --sensor-id ID` | Memulai ulang DAQ-E dan menghubungkannya kembali. |
| `daq pool-update-firmware --sensor-id ID` | Mem-flash firmware DAQ-E versi ini ke sensor yang lebih lama. |

***

## Pemecahan Masalah Koneksi DAQ-E Pertama Kali

1. DAQ-E tidak memiliki LED status — pastikan daya menyala melalui indikator PoE/link pada port switch atau injektor, dan tunggu beberapa detik setelah dinyalakan agar perangkat dapat melakukan booting dan bergabung dengan jaringan.
2. Mesin backend harus berada di **domain siaran yang sama** dengan sensor — mDNS tidak dapat menembus router.
3. Firewall harus mengizinkan masuknya mDNS (UDP 5353), data DAQ-E (UDP 5002), dan PTP (UDP 319/320). Penginstal Windows menambahkan aturan untuk ini, sedangkan penginstal Linux `.deb` membukanya melalui aturan yang sudah aktif pada saat instalasi; firewall pihak ketiga, atau firewall yang diaktifkan kemudian, perlu mengizinkan port-port tersebut.
4. Masih belum ada respons dari `--eth`? Gunakan `--eth-host` dengan nama host perangkat (`daq-e-<id>.local`) atau alamat IP-nya, yang akan melewati proses penemuan.

***
{% hint style="info" %}
**Tips untuk asisten AI.** Setiap halaman manual ini disajikan dalam format Markdown mentah — tambahkan `.md` ke bagian akhir slug URL huruf kecil dari halaman tersebut (halaman ini: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); indeks yang dapat dibaca mesin adalah `https://mapir.gitbook.io/chloros/llms.txt`. Untuk dokumentasi lengkap tingkat opsi dari `chloros-cli daq` dan setiap keluarga perintah lainnya, unduh [Referensi CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); jalur Python-nya adalah `chloros_sdk.connect_daq_sensor()` di [Referensi SDK](../reference/sdk-reference.md).
{% endhint %}
</version\>

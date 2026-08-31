# CLI Panduan Cepat (pool-*)

Drive `chloros-cli` yang disertakan mengendalikan sensor DAQ melalui keluarga perintah **`daq pool-*`** — klien HTTP ringan yang mengoperasikan sensor melalui kumpulan sensor persisten backend Chloros. Backend mengendalikan transportasi, sehingga GUI, skrip CLI, dan SDK semuanya berbagi satu handle aktif alih-alih berebut port. Semua yang dibutuhkan pelanggan dapat diakses melalui `pool-*`: terhubung, melakukan streaming, merekam berkas `.daq` yang telah dikalibrasi, dan mengganti profil kap.

`pool-*` juga merupakan **satu-satunya** permukaan DAQ pada rilis yang telah diterbitkan. `chloros-cli daq --help` mencantumkan subperintah `pool-*`, dan menjalankan subperintah DAQ langsung ke perangkat keras pada build yang telah dirilis akan berakhir dengan pesan kesalahan eksplisit yang menyebutkan paket yang hilang dan mengarahkan Anda kembali ke `pool-*` — tidak ada yang gagal tanpa pemberitahuan. (Perintah perangkat keras langsung hanya dapat dijalankan dari checkout sumber MAPIR; `pip install chloros-sdk` juga tidak menyediakannya.)

***

## Persyaratan

* **Backend Chloros harus sedang berjalan** — perintah `pool-*` adalah klien HTTP, bukan driver perangkat keras. Pada Windows, jalankan aplikasi desktop Chloros (aplikasi ini akan meluncurkan backend). Pada Linux/Jetson tanpa antarmuka grafis, aktifkan layanan: `sudo systemctl enable --now chloros-backend.service`.
* **Masuk ke Chloros+ (tingkatan berbayar)**: jalankan `chloros-cli login` terlebih dahulu. Penegakan dilakukan di sisi server — tanpa login, perintah akan gagal dengan kode kesalahan `401 AUTH_REQUIRED`; pada tingkatan gratis (Iron), perintah akan gagal dengan kode kesalahan `403 PLAN_UPGRADE_REQUIRED`.
* Perintah-perintah ini secara default menargetkan `http://127.0.0.1:5000`; keluarga `daq pool-*` akan mengikuti variabel lingkungan `CHLOROS_BACKEND_URL` jika backend Anda berjalan di tempat lain.

***

## Sesi lima menit

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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

## `pool-connect` — membuka sensor di dalam pool

| Varian | Arti |
| --- | --- |
| `daq pool-connect` | Deteksi cerdas: temukan DAQ apa pun di mesin ini. |
| `daq pool-connect --port PORT` | DAQ-U pada port serial tertentu (misalnya `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M melalui BLE, MAC dipindai secara otomatis. |
| `daq pool-connect --mac MAC` | DAQ-M pada MAC BLE yang diketahui (menyiratkan `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E pada nama host atau IP yang diketahui — **jalur yang andal**. |
| `daq pool-connect --eth` | DAQ-E dengan penemuan otomatis (mDNS, dengan cadangan ARP). Lihat peringatan di bawah ini. |

Bendera penyesuaian, semuanya opsional:

| Bendera | Arti |
| --- | --- |
| `--integration-time MS` / `-t MS` | Waktu integrasi manual dalam milidetik. |
| `--frame-avg N` / `-f N` | Rata-rata bingkai per spektrum yang dilaporkan. |
| `--no-ae` | Nonaktifkan pengaturan eksposur otomatis (AE aktif secara default). |
| `--no-stream` | Terhubung tanpa memulai streaming (lanjutkan nanti dengan `pool-stream --start`). |
| `--cap-id CAP` | Profil koreksi batas; default backend adalah `sunshine_cosine`. Lihat [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

{% hint style="warning" %}
**Peringatan penemuan otomatis `--eth`.** Pada host multi-homed (memiliki lebih dari satu antarmuka jaringan aktif), `pool-connect --eth` *pertama* setelah boot mungkin tidak menghasilkan apa-apa meskipun sensor dalam kondisi sehat — proses penelusuran penemuan dapat melewatkan antarmuka sensor saat cache ARP masih kosong. Jika `--eth` tidak menemukan apa pun, coba lagi, atau lewati proses penemuan sepenuhnya dengan menggunakan `--eth-host <ip-or-hostname>`, yang merupakan jalur yang andal pada mesin multi-homed. Nama host DAQ-E adalah `daq-e-<id>.local` (mis. `daq-e-def330.local`); alamat IP biasa juga dapat digunakan.
{% endhint %}

## `pool-list` — lihat apa yang terhubung

Menampilkan setiap sensor di kumpulan backend, termasuk `sensor_id` yang diperlukan oleh setiap perintah lainnya:

| Model | Bentuk `sensor_id` | Contoh |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-oktet dengan tanda hubung | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — membaca bingkai spektrum

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Mengembalikan bingkai terbaru, atau bingkai-bingkai terbaru `--recent N`; `--json` menghasilkan keluaran yang dapat dibaca mesin untuk keperluan skrip. Bingkai-bingkai tersebut merupakan irradiansi spektral (W/m²/nm) yang telah dikalibrasi secara radiometrik pada kisi 135 titik dengan rentang 340–1010 nm, dengan profil penutup sensor yang telah diterapkan. Untuk nilai irradiance kuantitatif, hitung rata-rata setidaknya 15 detik frame — ini merupakan karakteristik instrumen, bukan cacat.

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Direktori keluaran, ditentukan **pada mesin yang menjalankan backend**. |
| `--device-name NAME` | — | Label yang disimpan bersama rekaman. |
| `--stop` | — | Menghentikan rekaman yang sedang berlangsung. |

{% hint style="info" %}
Perekaman terjadi di backend, sehingga berkas `.daq` tersimpan di sistem berkas **mesin backend** — secara default di `~/Documents/DAQ Live View/` di sana, tidak harus di tempat Anda menjalankan CLI. Nama berkas mencakup ID sensor dan cap waktu.
{% endhint %}

## `pool-set-cap` — tentukan penutup yang terpasang

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

ID penutup ini menentukan profil koreksi yang diukur di pabrik dan diterapkan pada setiap spektrum, serta **harus sesuai dengan penutup yang terpasang secara fisik pada sensor** — baik sensor maupun perangkat lunak tidak dapat mendeteksi penutup secara otomatis, dan pemilihan ini dicantumkan dalam setiap berkas `.daq`. Nilai default di mana pun adalah `sunshine_cosine` (setiap DAQ dikirimkan dengan penutup korektor kosinus Sunshine terpasang, dengan atenuasi ~12× sesuai desain — perubahan penutup yang tidak dinyatakan akan mengoreksi spektrum secara salah dengan faktor sekitar itu).

| `--cap-id` | Tersedia pada |
| --- | --- |
| `sunshine_cosine` (default) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Hanya DAQ-U |
| `none` | Hanya DAQ-E — lihat catatan |

ID penutup di luar set sensor akan ditolak saat koneksi dengan pesan kesalahan yang jelas. `none` (DAQ-E) berarti penutup telah dilepas secara fisik — profil geometri pabrik tetap diterapkan untuk difuser kaca yang tersembunyi pada DAQ-E, sehingga ini bukan operasi kosong (no-op), dan DAQ-E tanpa penutup merupakan konfigurasi meja uji, bukan konfigurasi lapangan yang didukung. (DAQ-U tanpa penutup adalah konfigurasi benar-benar tanpa penutup dan tidak memerlukan profil koreksi sama sekali; DAQ-M digunakan bersama penutup Sunshine-nya.)

## `pool-disconnect` — melepaskan sensor

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Ringkasan perintah

| Perintah | Tujuan |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Membuka sensor di kumpulan backend. |
| `daq pool-list` | Menampilkan setiap sensor dalam pool beserta `sensor_id`-nya. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N bingkai spektrum terkalibrasi terbaru. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Lanjutkan / jeda streaming. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Mulai / hentikan perekaman `.daq` (di sisi backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Mengganti profil koreksi batas selama runtime. |
| `daq pool-disconnect --sensor-id ID [--all]` | Melepaskan satu sensor, atau semuanya. |

***

## Pemecahan Masalah Koneksi DAQ-E Pertama Kali

1. DAQ-E tidak memiliki LED status — pastikan daya menyala melalui indikator PoE/link pada port switch atau injektor, dan tunggu beberapa detik setelah dinyalakan agar perangkat dapat melakukan booting dan bergabung dengan jaringan.
2. Mesin backend harus berada di **domain siaran yang sama** dengan sensor — mDNS tidak menembus router.
3. Pada Windows, terima prompt firewall Defender pada pengoperasian pertama (mDNS UDP 5353, data DAQ-E UDP 5002, PTP UDP 319/320).
4. Masih belum ada respons dari `--eth`? Gunakan `--eth-host` dengan nama host unit (`daq-e-<id>.local`) atau alamat IP — ini merupakan jalur yang lebih andal, terutama pada host dengan multiple koneksi.

***{% hint style="info" %}**Tips untuk asisten AI.** Setiap halaman manual ini disajikan dalam format Markdown mentah — tambahkan `.md` ke slug huruf kecil halaman tersebut (halaman ini: URL); indeks yang dapat dibaca mesin adalah `https://mapir.gitbook.io/chloros/llms.txt`. Untuk dokumentasi lengkap tingkat flag dari `chloros-cli daq` dan setiap kelompok perintah lainnya, unduh [Referensi CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); jalur Python adalah `chloros_sdk.connect_daq_sensor()` dalam [Referensi SDK](../reference/sdk-reference.md).
{% endhint %}

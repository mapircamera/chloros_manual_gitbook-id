# Menggunakan Chloros dengan Asisten AI

Panduan ini ditulis untuk dua kelompok pembaca: manusia, dan asisten AI yang semakin sering digunakan manusia dalam bekerja. Setiap halaman memuat nilai-nilai yang tepat, pengaturan default, dan perintah yang dapat disalin dan ditempelkan sehingga asisten (Claude, ChatGPT, Copilot, agen pemrograman, …) dapat menulis otomatisasi Chloros yang berfungsi pada percobaan pertama.

Versi Chloros: **

1.2.0**. Platform CLI/SDK: Windows 10/11 x64 dan Linux (x86_64 / Jetson aarch64).

## Apa yang harus Anda serahkan kepada asisten Anda

| Sumber Daya | URL | Kegunaannya |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Indeks yang dapat dibaca mesin untuk setiap halaman dalam panduan ini. |
| **CLI Referensi** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Permukaan perintah `chloros-cli` yang lengkap: setiap perintah, bendera, nilai default, kode keluar, dan aturan folder keluaran. Ditulis untuk digunakan oleh LLM. |
| **Referensi SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | `chloros_sdk` Python API secara lengkap: kelas, tanda tangan, pengecualian, dan contoh yang telah dikerjakan. Ditulis untuk konsumsi LLM. |
| **Halaman apa pun sebagai Markdown mentah** | tambahkan `.md` ke halaman URL | misalnya `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` menampilkan halaman dalam format Markdown mentah — ideal untuk disisipkan ke jendela konteks atau diambil dari agen. |

Tautan dalam manual: [Referensi CLI](reference/cli-reference.md) · [Referensi SDK](reference/sdk-reference.md).

{% hint style="info" %}
Kedua halaman referensi tersebut bersifat mandiri: seorang asisten yang telah membaca salah satunya tidak memerlukan bagian lain dari manual ini untuk menulis skrip yang benar.
{% endhint %}

## Resep perintah

Salin, isi bagian yang kosong pada `<placeholders>`, lalu tempelkan ke asisten Anda.

### 1. Memproses folder penerbangan ke dalam NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Memantau direktori tangkapan secara batch

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Menghubungkan susunan LATTICE dan menangkap

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Merekam spektrum sensor cahaya DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Penulisan skrip DAQ dari baris perintah selalu dilakukan melalui keluarga perintah `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Subperintah `daq` lain yang mungkin diciptakan oleh asisten Anda tidak tersedia dalam build yang dirilis dan akan berakhir dengan pesan kesalahan.
{% endhint %}

## Mengapa skrip yang ditulis AI bekerja dengan baik bersama Chloros

Masing-masing dari ini adalah perilaku nyata dan terverifikasi dari Chloros 1.2.0 — fitur-fitur ini menghilangkan mode kegagalan klasik pada otomatisasi yang ditulis oleh mesin:

* **Tidak ada proses pengaturan yang rumit.**Pembantu koneksi cerdas SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) dan titik masuk pemrosesan (`ChlorosLocal`, `process_folder`)**secara otomatis memulai backend lokal**. Skrip yang dihasilkan tidak memerlukan GUI yang terbuka atau server yang dijalankan secara manual — skrip tersebut hanya memerlukan paket desktop/CLI yang terinstal.
* **Seluruh alur kerja dilakukan dalam satu panggilan.** `chloros_sdk.process_folder("path", indices=["NDVI"])` menjalankan impor → kalibrasi → reflektansi → ekspor indeks secara end-to-end. Semakin sedikit area permukaan, semakin sedikit peluang terjadinya kesalahan pada skrip yang dihasilkan.
* **Proses tanpa hasil akan melakukan diagnosis mandiri.** Setelah `process()`, ringkasan proses dilampirkan pada hasil, dan setiap petunjuk pemrosesan (mis. *mengapa* suatu proses tidak menghasilkan keluaran) juga ditampilkan kembali sebagai Python `UserWarning` — sehingga bahkan skrip yang tidak pernah memeriksa hasil dict pun akan menampilkan diagnosis tersebut.
* **CLI gagal dengan jelas.**Eksekusi `chloros-cli process` yang meminta produk tetapi tidak menulis apa pun akan mencetak `Processing finished but wrote no image products.` dan**keluar dengan nilai non-nol**, sehingga skrip shell dan CI dapat mendeteksinya dengan pemeriksaan kode keluar biasa. Eksekusi yang berhasil melaporkan `Image products written: N`.

Satu ketidaksimetrisan yang harus diketahui oleh asisten: `process()` dari SDK secara sengaja **tidak** memicu kesalahan pada eksekusi tanpa hasil — melainkan melaporkannya melalui ringkasan/petunjuk. Jika pipa Python harus berhenti pada eksekusi yang kosong, periksa ringkasan (resep 2 melakukannya).

## Peringatan

* **Diperlukan login Chloros+.**CLI dan SDK memerlukan tingkatan**berbayar** Chloros+, yang diberlakukan di sisi server: permintaan akan gagal dengan kode kesalahan `401 AUTH_REQUIRED` jika belum login dan `403 PLAN_UPGRADE_REQUIRED` pada tingkatan gratis. Jalankan `chloros-cli login` sekali per mesin sebelum menjalankan skrip yang dihasilkan. Lihat [Chloros+ Login](chloros+-login.md).
* **Perintah Capture mengendalikan perangkat keras yang sebenarnya.** Perintah `lattice` / `daq` / `project` serta objek sesi SDK terhubung, melakukan streaming, dan memicu kamera serta sensor fisik. Periksa skrip yang dihasilkan sebelum dijalankan pertama kali, dan jalankan dengan pengawasan langsung terhadap perangkat keras.
* **Periksa secara acak hasilnya.** Verifikasi folder hasil dan beberapa nilai piksel sebelum mempublikasikan hasil. Secara khusus, berkas TIFF reflektansi diskalakan per sumber — baca tag XMP `Chloros:PixelScale` (LATTICE: 32768 = reflektansi 1,0; Survey3: 65535) alih-alih mengasumsikan pembagi. Kedua halaman referensi mendokumentasikan hal ini di bawah &quot;Membaca piksel reflektansi&quot;.
* **Hal-hal kecil yang dapat mengganggu kode yang dihasilkan:**`pool-record` menulis ke sistem berkas**host backend** (default `~/Documents/DAQ Live View/`); pada mesin dengan beberapa antarmuka jaringan, gunakan `daq pool-connect --eth-host <ip-or-hostname>` daripada penemuan otomatis; dan gunakan `http://127.0.0.1:5000` (bukan `localhost`) di mana pun backend URL muncul.

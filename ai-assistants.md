# Menggunakan Chloros Bersama Asisten AI

Panduan ini ditulis untuk dua kelompok pembaca: manusia, dan asisten AI yang semakin sering digunakan oleh manusia. Setiap halaman memuat nilai-nilai pasti, pengaturan default, dan perintah yang dapat disalin dan ditempelkan sehingga asisten (Claude, ChatGPT, Copilot, agen pemrograman, …) dapat menulis otomatisasi Chloros yang berfungsi dengan baik pada percobaan pertama.

Versi Chloros: **

1.3.0**. Platform CLI/SDK: Windows 10/11 x64 dan Linux (x86\_64 / Jetson aarch64).

## Apa yang Harus Anda Berikan kepada Asisten Anda

| Sumber Daya | URL | Kegunaannya |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Indeks yang dapat dibaca mesin untuk setiap halaman dalam panduan ini. |
| **Referensi CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Permukaan perintah `chloros-cli` yang lengkap: setiap perintah, bendera, nilai default, kode keluar, dan aturan folder keluaran. Ditulis untuk konsumsi LLM. |
| **Referensi SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Python API `chloros_sdk` yang lengkap: kelas, tanda tangan, pengecualian, dan contoh penerapan. Ditulis untuk digunakan oleh LLM. |
| **Halaman apa pun dalam format Markdown mentah** | tambahkan `.md` ke URL halaman | misalnya, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` mengembalikan halaman dalam format Markdown mentah — ideal untuk disalin ke jendela konteks atau diambil dari agen. |

Tautan dalam manual: [Referensi CLI](reference/cli-reference.md) · [Referensi SDK](reference/sdk-reference.md).

{% hint style="info" %}
Kedua halaman referensi tersebut bersifat mandiri: asisten yang telah membaca salah satunya tidak memerlukan bagian manual lainnya untuk menulis skrip yang benar.
{% endhint %}

## Biarkan asisten Anda menjalankan Chloros secara langsung: `chloros-mcp`

Semua penjelasan di atas mengharuskan asisten Anda *menulis* skrip atau perintah. Dengan `chloros-mcp`, asisten yang mendukung [Model Context Protocol](https://modelcontextprotocol.io) (MCP), seperti Claude Desktop, Claude Code, atau Cursor, mengendalikan Chloros sendiri. Anda bertanya dalam bahasa sehari-hari dan asisten tersebut akan memanggil alat-alat di server:

* **Memproses citra:** membuat proyek, mengimpor folder tangkapan, memilih pengaturan, memproses, dan melaporkan hasil yang diekspor dari proses tersebut.
* **Mengoperasikan perangkat keras:** menghubungkan kamera LATTICE, array, dan sensor cahaya DAQ, melihat pratinjau, mengatur eksposur dan warna, mengambil gambar, merekam, serta memperbarui firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Alat-alat server ini mencakup fungsi yang dilakukan oleh [Python SDK](api-python-sdk.md). Alat-alat ini terinstal bersama SDK, sehingga biasanya tidak perlu menginstal apa pun lagi: di Windows, penginstal Chloros akan menempatkan `chloros-mcp.exe` di folder `Scripts` Python Anda, sedangkan di Linux, `.deb` akan menempatkan `chloros-mcp` di dalam `/usr/bin`.

**Kode Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**tambahkan ini ke `claude_desktop_config.json` (buka melalui**Pengaturan &gt; Pengembang &gt; Edit Konfigurasi**), lalu mulai ulang Claude Desktop.**Cursor** menggunakan entri yang sama di `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Jika klien melaporkan bahwa server gagal dimulai, kemungkinan `chloros-mcp` tidak tercantum pada `PATH` yang terlihat oleh klien: berikan jalur lengkapnya (`where chloros-mcp` di Windows, `which chloros-mcp` di Linux).

Anda perlu menginstal Chloros di komputer yang sama dan masuk ke akun di sana, baik melalui aplikasi maupun dengan `chloros-cli login`, pada paket Chloros+ berbayar apa pun (Copper atau yang lebih tinggi). Server tidak dapat masuk atas nama Anda.

{% hint style="info" %}
Pengaturan klien untuk setiap aplikasi, opsi, setiap alat, dan cara kerja proses: [MCP Server (Asisten AI)](reference/sdk-reference.md#mcp-server-ai-assistants) dalam Referensi SDK.
{% endhint %}

## Resep prompt

Salin, isi bagian `<placeholders>`, lalu tempelkan ke asisten Anda.

### 1. Mengolah folder penerbangan menjadi NDVI

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

### 3. Menghubungkan susunan LATTICE dan melakukan pengambilan data

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
Penulisan skrip DAQ dari baris perintah selalu dilakukan melalui keluarga perintah `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Subperintah `daq` lain yang mungkin diciptakan oleh asisten Anda tidak tersedia dalam versi yang dirilis dan akan berakhir dengan pesan kesalahan.
{% endhint %}

## Mengapa skrip yang ditulis AI berfungsi dengan baik bersama Chloros

Masing-masing dari hal ini merupakan perilaku nyata dan terverifikasi dari Chloros 1.3.0 — hal ini menghilangkan mode kegagalan klasik pada otomatisasi yang ditulis oleh mesin:

* **Tidak perlu proses penyiapan yang rumit.**Fitur bantuan koneksi cerdas SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) serta titik masuk pemrosesan (`ChlorosLocal`, `process_folder`)**secara otomatis menjalankan backend lokal**. Skrip yang dihasilkan tidak memerlukan antarmuka pengguna (GUI) yang terbuka atau server yang dijalankan secara manual — cukup dengan menginstal paket desktop atau CLI.
* **Seluruh alur kerja dilakukan dalam satu panggilan.** `chloros_sdk.process_folder("path", indices=["NDVI"])` menjalankan impor → kalibrasi → reflektansi → ekspor indeks secara end-to-end. Semakin sedikit area permukaan, semakin sedikit kemungkinan skrip yang dihasilkan mengalami kesalahan.
* **Proses tanpa hasil akan melakukan diagnosis mandiri.** Setelah `process()`, ringkasan proses dilampirkan pada hasilnya, dan setiap petunjuk pemrosesan (mis. *mengapa* suatu proses tidak menghasilkan keluaran) juga ditampilkan kembali sebagai skrip Python `UserWarning` — sehingga bahkan skrip yang tidak pernah memeriksa kamus hasil pun tetap menampilkan diagnosis tersebut.
* **Antarmuka Baris Perintah (CLI) akan memberikan peringatan yang jelas.**Eksekusi `chloros-cli process` yang meminta produk tetapi tidak menghasilkan apa pun akan menampilkan pesan `Processing finished but wrote no image products.` dan**keluar dengan kode keluar non-nol**, sehingga skrip shell dan CI dapat mendeteksinya melalui pemeriksaan kode keluar sederhana. Eksekusi yang berhasil melaporkan `Image products written: N`, ditambah `(plus M clamp masks)` jika masker ditulis. Perintah penangkapan juga mencantumkan setiap file yang tidak dapat ditulis dan keluar dengan kode `1`.
* **Jalur relatif berarti apa yang diharapkan oleh skrip.** Dengan backend lokal, jalur keluaran relatif (`-o out/` di CLI, `output_dir="out/"` di SDK) diselesaikan berdasarkan folder tempat perintah atau skrip dijalankan.

Satu ketidaksimetrisan yang harus diketahui oleh asisten: `process()` di SDK secara sengaja **tidak** memicu pengecualian pada eksekusi tanpa hasil — melainkan melaporkannya melalui ringkasan/petunjuk. Jika pipa Python harus berhenti saat eksekusi kosong, periksa ringkasan (resep 2 melakukannya).

## Peringatan

* **Diperlukan login Chloros+.**CLI dan SDK memerlukan tingkatan Chloros+**berbayar**, yang diberlakukan di sisi server: permintaan akan gagal dengan kode kesalahan `401 AUTH_REQUIRED` jika belum masuk, dan `403 PLAN_UPGRADE_REQUIRED` pada tingkatan gratis. Jalankan `chloros-cli login` sekali per mesin sebelum menjalankan skrip yang dihasilkan. Lihat [Masuk Chloros+](chloros+-login.md).
* **Perintah penangkapan mengendalikan perangkat keras fisik.** Perintah `lattice` / `daq` / `project`, objek sesi SDK, dan alat-alat `chloros-mcp` terhubung, melakukan streaming, dan memicu kamera serta sensor fisik. Periksa skrip yang dihasilkan sebelum dijalankan pertama kali, dan jalankan skrip tersebut dengan perangkat keras yang diawasi.
* **Periksa hasil keluaran secara acak.** Verifikasi folder hasil dan beberapa nilai piksel sebelum mempublikasikan hasil. Secara khusus, file TIFF reflektansi diskalakan per sumber — baca tag XMP `Chloros:PixelScale` (file LATTICE menyertakannya: 32768 = 1,0 reflektansi; file Survey3 tidak memilikinya, dan menggunakan 65535 = 1,0) alih-alih mengasumsikan pembagi. Kedua halaman referensi tersebut mendokumentasikan hal ini di bawah &quot;Membaca piksel reflektansi&quot;.
* **Hal-hal kecil yang dapat mengganggu kode yang dihasilkan:**`pool-record` menulis ke sistem berkas**host backend** (default `~/Documents/DAQ Live View/`); pada mesin dengan beberapa antarmuka jaringan, gunakan `daq pool-connect --eth-host <ip-or-hostname>` daripada penemuan otomatis; gunakan `http://127.0.0.1:5000` (bukan `localhost`) di mana pun URL backend muncul; skrip yang menjelajahi pohon output proyek harus melewati folder `clamp_masks/` (file `*_clampmask.tif` di dalamnya adalah masker, bukan gambar — lihat [Format Gambar Keluaran](output-image-formats.md#clamp-masks)); dan pada Windows dengan opsi jalur panjang dinonaktifkan, jaga agar nama folder proyek tetap pendek — folder proyek yang kedalamannya melebihi sekitar 130 karakter akan ditolak sebelum pemrosesan dimulai.

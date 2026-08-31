# Chloros+ Masuk

## Masuk melalui Antarmuka Pengguna Grafis (GUI)

Menu bilah samping <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> memungkinkan Anda masuk ke akun Chloros+ Anda dan mengakses fitur-fitur tambahan.

**Anda hanya perlu masuk sekali per perangkat.** Antarmuka GUI, CLI, dan Python serta SDK menggunakan sesi cache yang sama — masuk melalui antarmuka pengguna desktop juga akan mengaktifkan CLI dan SDK pada perangkat tersebut (dan sebaliknya melalui `chloros-cli login`).

Setelah masuk, detail akun Anda akan ditampilkan:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Tingkat Paket

| Paket | `plan_id` | Jenis |
| --- | --- | --- |
| Iron | `0` | Gratis |
| Copper | `1` | Berbayar (Chloros+) |
| Bronze | `2` | Berbayar (Chloros+) |
| Perak | `3` | Berbayar (Chloros+) |
| Emas | `4` | Berbayar (Chloros+) |

Lihat [paket dan harga](https://cloud.mapir.camera/pricing) untuk mengetahui apa saja yang termasuk dalam setiap tingkatan berbayar.

### Akses ke CLI / SDK memerlukan tingkatan berbayar

Akses ke CLI, Python, dan SDK memerlukan **tingkatan berbayar Chloros+ apa pun (Copper atau lebih tinggi)**. Hal ini diberlakukan**di sisi server** — setiap permintaan CLI/SDK harus disertai dengan sesi aktif dan paket berbayar:

| Status HTTP | `error_code` | Arti | Solusi |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Belum masuk ke sistem di mesin ini | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Telah masuk, tetapi tingkatan paket terlalu rendah (tingkatan Iron gratis) | Tingkatkan ke paket berbayar Chloros+ apa pun |

`chloros-cli status` tetap dapat diakses pada tingkatan gratis, sehingga Anda selalu dapat melihat paket Anda saat ini dan alasan penolakan akses.

### Batasan perangkat terhubung per paket

Setiap paket membatasi jumlah kamera LATTICE dan sensor cahaya DAQ yang dapat terhubung secara langsung sekaligus:

| Paket | Kamera LATTICE | Sensor Cahaya DAQ |
| --- | --- | --- |
| Iron (gratis / belum masuk) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Masuk ke CLI

Masuk menggunakan kredensial Chloros+ Anda untuk mengaktifkan pemrosesan CLI. Pada Linux (tanpa antarmuka grafis), ini adalah satu-satunya cara untuk mengaktifkan lisensi Anda.

**Sintaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Pengguna SDK**: Python hingga SDK juga menyediakan metode pemrograman `logout()` untuk menghapus kredensial yang tersimpan dalam cache. Lihat [Referensi SDK](reference/sdk-reference.md) untuk detailnya.
{% endhint %}

**Contoh:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Karakter Khusus**: Gunakan tanda kutip tunggal di sekitar kata sandi yang mengandung karakter seperti `$`, `!`, atau spasi.
{% endhint %}

**Hasil:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Penyimpanan Kredensial

Kredensial dan konfigurasi yang disimpan dalam cache disimpan di folder `.chloros` di direktori home pengguna Anda pada **semua platform**:

| Platform | Jalur Cache Kredensial |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Kedaluwarsa Paket dan Masa Tenggang Offline

Tanggal kedaluwarsa paket yang ditampilkan di antarmuka pengguna (GUI) menunjukkan kapan lisensi Anda akan menjadi tidak valid. Untuk langganan bulanan berulang, kedaluwarsa terjadi pada akhir bulan; sedangkan untuk langganan tahunan, kedaluwarsa terjadi satu tahun setelah Anda memulai langganan.

Chloros memvalidasi lisensi Anda secara online, tetapi penggunaan offline didukung selama masa tenggang:

* Validasi server yang berhasil disimpan dalam cache selama **5 menit**, sehingga penggunaan normal hanya memerlukan sedikit panggilan lisensi.
* Cache lisensi yang ditandatangani dan terikat pada perangkat mencakup periode offline yang lebih lama: **30 hari untuk paket bulanan**, dan**hingga tanggal kedaluwarsa langganan Anda (maksimal 365 hari) untuk paket tahunan**.
* Ketika masa tenggang berakhir, paket akan beralih ke tingkatan Iron gratis hingga perangkat tersebut dapat terhubung ke server lisensi sekali; akses akan dilanjutkan pada pemeriksaan berikutnya yang berhasil.

### Batas Perangkat

Setiap paket Chloros+ menawarkan jumlah perangkat terdaftar yang berbeda. Setiap perangkat yang Anda gunakan untuk masuk dengan akun Chloros+ akan dihitung dalam jumlah perangkat terdaftar Anda. Anda dapat mengganti nama dan menghapus perangkat di halaman akun MAPIR Cloud Anda.

<table><thead><tr><th width="168.5999755859375" align="right">Paket Chloros+</th><th align="center">TEMBAGA</th><th align="center">BRONZE</th><th align="center">PERAK</th><th align="center">EMAS</th></tr></thead><tbody><tr><td align="right">Perangkat yang Didukung</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>Jumlah perangkat yang diizinkan untuk akun Anda ditampilkan di halaman akun MAPIR Cloud Anda. Keluar dari perangkat akan secara pasti membebaskan slotnya, dan perangkat yang sudah terdaftar selalu dapat masuk kembali meskipun akun telah mencapai batas jumlah perangkat.

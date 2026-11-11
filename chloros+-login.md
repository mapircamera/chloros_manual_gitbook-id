# Masuk ke Chloros+

## Masuk melalui Antarmuka Pengguna Grafis (GUI)

Menu bilah samping pengguna <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> memungkinkan Anda masuk ke akun Chloros+ dan mengakses fitur-fitur tambahan.

**Anda hanya perlu masuk sekali per perangkat.** GUI, CLI, dan Python SDK menggunakan sesi cache yang sama — masuk melalui GUI desktop juga akan mengaktifkan CLI dan SDK di perangkat tersebut (dan sebaliknya melalui `chloros-cli login`).

Setelah masuk, detail akun Anda akan ditampilkan:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Tingkat Paket

| Paket | `plan_id` | Jenis |
| --- | --- | --- |
| Iron | `0` | Gratis |
| Copper | `1` | Berbayar (Chloros+) |
| Bronze | `2` | Berbayar (Chloros+) |
| Silver | `3` | Berbayar (Chloros+) |
| Emas | `4` | Berbayar (Chloros+) |

Lihat [paket dan harga](https://cloud.mapir.camera/pricing) untuk mengetahui apa saja yang termasuk dalam setiap tingkatan berbayar.

### Akses CLI/SDK memerlukan tingkatan berbayar

Akses CLI dan Python SDK memerlukan **tingkatan Chloros+ berbayar apa pun (Copper atau lebih tinggi)**. Hal ini diterapkan**di sisi server** — setiap permintaan CLI/SDK harus menyertakan sesi aktif dan paket berbayar:

| Status HTTP | `error_code` | Arti | Solusi |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Belum masuk (login) di mesin ini | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Telah masuk, tetapi tingkatan paket terlalu rendah (tingkatan Iron gratis) | Tingkatkan ke paket Chloros+ berbayar apa pun |

`chloros-cli status` tetap dapat diakses pada tingkatan gratis, sehingga Anda selalu dapat melihat paket Anda saat ini dan alasan mengapa akses ditolak.

### Batasan perangkat yang terhubung per paket

Setiap paket membatasi jumlah kamera LATTICE dan sensor cahaya DAQ yang dapat terhubung secara langsung sekaligus:

| Paket | Kamera LATTICE | Sensor cahaya DAQ |
| --- | --- | --- |
| Iron (gratis / belum masuk) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Masuk ke CLI



Masuk menggunakan kredensial Chloros+ Anda untuk mengaktifkan pemrosesan CLI. Di Linux (tanpa GUI), ini adalah satu-satunya cara untuk mengaktifkan lisensi Anda.

**Sintaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Pengguna SDK**: Python SDK juga menyediakan metode `logout()` untuk menghapus kredensial yang tersimpan dalam cache. Lihat [Referensi SDK](reference/sdk-reference.md) untuk detailnya.
{% endhint %}

**Contoh:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Karakter Khusus**: Gunakan tanda kutip tunggal di sekitar kata sandi yang mengandung karakter seperti `$`, `!`, atau spasi.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Penyimpanan Kredensial

Kredensial dan konfigurasi yang disimpan dalam cache disimpan di folder `.chloros` pada direktori home pengguna Anda di **semua platform**:

| Platform | Jalur Cache Kredensial |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

Menginstal versi yang lebih baru di atas versi yang sudah ada akan mempertahankan kredensial login Anda, baik di Windows (penginstal) maupun di Linux (folder `.deb`, termasuk `chloros-cli update`). Menghapus instalasi Chloros — atau menghapus paket Linux — akan menghapus data login yang tersimpan dalam cache, sehingga Anda harus masuk kembali setelah instalasi baru.

### Kedaluwarsa Paket dan Masa Tenggang Offline

Tanggal kedaluwarsa paket yang ditampilkan di antarmuka pengguna (GUI) menunjukkan kapan lisensi Anda akan menjadi tidak valid. Untuk langganan bulanan berulang, masa berlaku berakhir pada akhir bulan; sedangkan untuk langganan tahunan, masa berlaku berakhir satu tahun setelah Anda memulai langganan.

 Chloros memvalidasi lisensi Anda secara online, tetapi penggunaan offline didukung selama masa tenggang:

* Validasi server yang berhasil disimpan dalam cache selama **5 menit**, sehingga penggunaan normal hanya memerlukan sedikit panggilan lisensi.
* Cache lisensi yang ditandatangani dan terikat pada perangkat mencakup periode offline yang lebih lama: **30 hari untuk paket bulanan**, dan**hingga tanggal kedaluwarsa langganan Anda (maksimal 365 hari) untuk paket tahunan**.
* Saat masa tenggang berakhir, paket akan beralih ke tingkatan Iron gratis hingga perangkat dapat terhubung ke server lisensi sekali; akses akan dilanjutkan pada pemeriksaan berikutnya yang berhasil.

### Batas Perangkat

Setiap paket Chloros+ menawarkan jumlah perangkat terdaftar yang berbeda-beda. Setiap perangkat yang Anda gunakan untuk masuk ke akun Chloros+ akan dihitung sebagai bagian dari jumlah perangkat terdaftar Anda. Anda dapat mengganti nama dan menghapus perangkat di halaman akun MAPIR Cloud Anda.

<table><thead><tr><th width="168.5999755859375" align="right">Paket Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Perangkat yang Didukung</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

Jumlah perangkat yang diizinkan untuk akun Anda ditampilkan di halaman akun MAPIR Cloud Anda. Keluar dari perangkat akan membebaskan slotnya secara pasti, dan perangkat yang sudah terdaftar selalu dapat masuk kembali meskipun akun telah mencapai batas jumlah perangkat.

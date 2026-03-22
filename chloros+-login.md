# Chloros+ Masuk

## Chloros dan Chloros (Browser) Masuk

Menu <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> memungkinkan Anda masuk ke akun Chloros+ Anda dan membuka fitur tambahan.

Setelah masuk, detail akun Anda akan ditampilkan:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Masuk

Masuk dengan kredensial Chloros+ Anda untuk mengaktifkan pemrosesan CLI. Pada Linux (tanpa antarmuka grafis), ini adalah satu-satunya cara untuk mengaktifkan lisensi Anda.

**Sintaks:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Pengguna SDK**: Python SDK juga menyediakan metode programatik `logout()` untuk menghapus kredensial yang disimpan dalam cache. Lihat [dokumentasi Python SDK](api-python-sdk.md#logout) untuk detailnya.
{% endhint %}

**Contoh:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Karakter Khusus**: Gunakan tanda kutip tunggal di sekitar kata sandi yang mengandung karakter seperti `$`, `!`, atau spasi.
{% endhint %}

**Output:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Penyimpanan Kredensial

Kredensial yang disimpan dalam cache disimpan di lokasi yang spesifik untuk platform:

| Platform | Jalur Cache Kredensial |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Masa Berlaku Paket

Masa berlaku paket yang ditampilkan di antarmuka pengguna (GUI) menunjukkan kapan lisensi Anda akan kedaluwarsa. Untuk langganan bulanan berulang, masa berlaku berakhir pada akhir bulan. Untuk langganan tahunan, masa berlaku berakhir satu tahun setelah Anda memulai langganan. Pemeriksaan lisensi memerlukan koneksi internet bulanan untuk verifikasi, dengan masa tenggang 30 hari.

### Batas Perangkat

Setiap paket Chloros+ menawarkan jumlah perangkat terdaftar yang berbeda. Setiap perangkat yang Anda gunakan untuk masuk dengan akun Chloros+ akan dihitung sebagai bagian dari jumlah perangkat terdaftar Anda. Anda dapat mengganti nama dan menghapus perangkat di halaman akun MAPIR Cloud Anda.

<table><thead><tr><th width="168.5999755859375" align="right">Paket Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">PERAK</th><th align="center">EMAS</th></tr></thead><tbody><tr><td align="right">Perangkat yang Didukung</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

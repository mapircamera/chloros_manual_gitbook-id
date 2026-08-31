# Bahasa yang Didukung

Chloros menyediakan dukungan antarmuka lengkap dalam **38 bahasa di seluruh dunia**, sehingga dapat diakses oleh pengguna di seluruh dunia. Anda dapat mengganti bahasa secara instan baik di antarmuka pengguna desktop maupun di CLI.

Chloros mendukung bahasa-bahasa berikut:

| # | Bahasa | Nama Asli | Kode CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Bahasa Inggris | English | `en` |
| 2 | 🇪🇸 Bahasa Spanyol | Español | `es` |
| 3 | 🇵🇹 Portugis | Português | `pt` |
| 4 | 🇫🇷 Prancis | Français | `fr` |
| 5 | 🇩🇪 Jerman | Deutsch | `de` |
| 6 | 🇮🇹 Italia | Italiano | `it` |
| 7 | 🇯🇵 Bahasa Jepang | 日本語 | `ja` |
| 8 | 🇰🇷 Bahasa Korea | 한국어 | `ko` |
| 9 | 🇨🇳 Bahasa Mandarin (Sederhana) | 简体中文 | `zh` |
| 10 | 🇹🇼 Bahasa Tionghoa (Tradisional) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Bahasa Rusia | Русский | `ru` |
| 12 | 🇳🇱 Belanda | Nederlands | `nl` |
| 13 | 🇸🇦 Arab | العربية | `ar` |
| 14 | 🇵🇱 Bahasa Polandia | Polski | `pl` |
| 15 | 🇹🇷 Bahasa Turki | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Bahasa Indonesia | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Bahasa Vietnam | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Thailand | ไทย | `th` |
| 20 | 🇸🇪 Swedia | Svenska | `sv` |
| 21 | 🇩🇰 Denmark | Dansk | `da` |
| 22 | 🇳🇴 Norwegia | Norsk | `no` |
| 23 | 🇫🇮 Finlandia | Suomi | `fi` |
| 24 | 🇬🇷 Yunani | Ελληνικά | `el` |
| 25 | 🇨🇿 Ceko | Čeština | `cs` |
| 26 | 🇭🇺 Hongaria | Magyar | `hu` |
| 27 | 🇷🇴 Rumania | Română | `ro` |
| 28 | 🇺🇦 Ukraina | Українська | `uk` |
| 29 | 🇧🇷 Bahasa Portugis Brasil | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Bahasa Kanton | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Melayu | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Slovakia | Slovenčina | `sk` |
| 33 | 🇧🇬 Bulgaria | Български | `bg` |
| 34 | 🇭🇷 Kroasia | Hrvatski | `hr` |
| 35 | 🇱🇹 Bahasa Lituania | Lietuvių | `lt` |
| 36 | 🇱🇻 Bahasa Latvia | Latviešu | `lv` |
| 37 | 🇪🇪 Estonia | Eesti | `et` |
| 38 | 🇸🇮 Slovenia | Slovenščina | `sl` |

## Cara Mengubah Bahasa

### Di Chloros Desktop

1. Buka pengaturan aplikasi
2. Buka menu pemilihan bahasa
3. Pilih bahasa yang Anda inginkan dari daftar
4. Antarmuka akan diperbarui secara instan

### Di Chloros CLI

Gunakan perintah `language` untuk melihat atau mengubah bahasa antarmuka CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Untuk detail lebih lanjut, lihat [dokumentasi CLI](CLI.md).

## Cakupan

Semua 38 bahasa didukung sepenuhnya di:

* **Chloros Desktop** - Terjemahan antarmuka pengguna grafis (GUI) secara lengkap
* **Chloros CLI** - Antarmuka baris perintah dan pesan keluaran

Python SDK API serta [dokumentasi referensinya](reference/sdk-reference.md) disediakan dalam bahasa Inggris.

Dukungan bahasa memastikan bahwa pengguna di seluruh dunia dapat bekerja secara efisien dalam bahasa ibu mereka tanpa hambatan.

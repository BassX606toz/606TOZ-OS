# Indicator of Compromise (IOC)

Semua nilai berikut berasal dari observasi statis terhadap artefak yang dianalisis.

## Berkas

| Jenis | Nilai |
|---|---|
| Nama file | `gogorejeki.apk` |
| SHA-256 | `7f816e52c22361c83436775574d3f360db5f80521677a0ff503b12066ef91687` |
| Ukuran | 27.901.422 byte |
| Format | Android package (APK), Cocos2d-JS |

## Distribusi

| Jenis | Nilai |
|---|---|
| Situs | `https://www.use-gogorejeki.com/` |
| Endpoint generator URL | `https://www.use-gogorejeki.com/getapk.php` (parameter `vid`, `code`, `mb`) |
| Endpoint statistik klik | `https://www.use-gogorejeki.com/download_stat` |
| Skrip sidik jari | `https://www.use-gogorejeki.com/src/js/fingerprintjs.js` |
| CDN APK | `https://d25ez2bu9sf2hl.cloudfront.net/62/5708606/gogorejeki_CS3DCF8FWJY.apk` |
| CloudFront POP saat diakses | TPE53 (Taiwan) |

## Hot-update / konten game (HTTP, bukan HTTPS)

| Host | Kegunaan | Jumlah manifest |
|---|---|---|
| `ifs.vew729sh2isjo9o.com` | `GameX/1.1.3.9/<Game>` | 251 |
| `ifsa.vevoso09slcom.com` | paket tambahan | 2 |
| `ifs.fwui238929sh.com` | `GameX/1.1.1.3/ActiveMania` | 1 |

Daftar lengkap nama game dan `packageUrl`: `analysis/hotupdate_manifests.txt`.

## Identitas aplikasi

| Jenis | Nilai |
|---|---|
| Package | `com.gogo.rejeki.games` |
| Versi | `1.0.6` / `106` |
| Aktivitas | `org.cocos2dx.javascript.AppActivity` |
| Service | `org.cocos2dx.javascript.MyFirebaseMessagingService` |
| Provider | `org.cocos2dx.javascript.myFileProvider` |
| Skema tanda tangan | v1 + v2 (v3/v3.1 tidak ada) |
| Sertifikat SHA-256 (v1 & v2 sama) | `5cbb225fff2ab9db2ce018bcbadbd378151f7ed8fcbd3ec9dbeae6cc1ca8eeea` |

## Sosial / pendukung (dari halaman situs, bukan dari APK)

- `https://telegram.me/GOGOREJEKIOFFICIAL`
- `https://whatsapp.com/channel/0029VbDaDgN0bIdvPs2gEW12`
- `https://www.facebook.com/Gogorejeki`
- `https://www.instagram.com/gogo_rejekiofficial`
- `https://www.youtube.com/@GogoRejeki`
- `https://www.gogorejekihelp.com/chat/pc/v6/index.html?sysnum=…&channelid=10`

## Catatan

- Sertifikat di atas adalah identitas yang **dideklarasikan**, bukan hasil verifikasi
  keaslian atau penilaian kepercayaan.
- Endpoint URL APK bersifat dinamis (dihasilkan `getapk.php`); nilai CDN di atas adalah
  satu observasi pada waktu analisis.
- Tidak ada IOC yang berasal dari eksekusi aplikasi.

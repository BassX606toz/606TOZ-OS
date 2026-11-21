# Cara kerja aplikasi (observasi statis)

Semua pernyataan di bawah berasal dari analisis statis REA/JADX terhadap
`gogorejeki.apk` (SHA-256 `7f816e52…f91687`). APK tidak dijalankan. Rincian dan
Evidence ID ada di `evidence/`.

## 1. Distribusi dari situs

Halaman `https://www.use-gogorejeki.com/` memuat skrip yang:

1. Memuat `/src/js/fingerprintjs.js` dan menghitung `visitorId` (browser fingerprint).
2. Memanggil `GET /getapk.php?vid=<visitorId>&code=<referral>&mb=<...>`.
3. Jika respons diawali `http`, URL tersebut dijadikan `href` tombol unduh (`#down_apk_2`),
   lalu `/record_domain.js` mencatat klik ke `/download_stat` sebelum mengalihkan.

Dengan `vid` dummy, endpoint mengembalikan:

```
https://d25ez2bu9sf2hl.cloudfront.net/62/5708606/gogorejeki_CS3DCF8FWJY.apk
```

Ini berarti URL APK tidak statis di HTML; ia dihasilkan server berdasarkan sidik jari
browser/kode referral. (Observasi dari respons HTTP langsung, bukan bukti runtime.)

## 2. Identitas APK

Dari `inspect_android_package` (Evidence `ev_c9380d27…de460bc5c`):

| Field | Nilai |
|---|---|
| Package | `com.gogo.rejeki.games` |
| Versi | `1.0.6` (versionCode 106) |
| minSdk / targetSdk | 21 / 35 |
| Kelas DEX | 1.528 |
| Aktivitas utama | `org.cocos2dx.javascript.AppActivity` (exported, launcher + deep link BROWSABLE) |
| Service | `org.cocos2dx.javascript.MyFirebaseMessagingService` (FCM) |
| Provider | `org.cocos2dx.javascript.myFileProvider`, Firebase init provider |

Izin yang dideklarasikan: `INTERNET`, `ACCESS_NETWORK_STATE`, `VIBRATE`, `WAKE_LOCK`,
`POST_NOTIFICATIONS`, `READ_GSERVICES`, `RECEIVE` (C2DM), `AD_ID`,
`BIND_GET_INSTALL_REFERRER_SERVICE`.

Signature (Evidence `ev_e39ccd61…a5b9006503b6498b`): skema v1 + v2, satu signer,
sertifikat SHA-256 `5cbb225fff2ab9db2ce018bcbadbd378151f7ed8fcbd3ec9dbeae6cc1ca8eeea`.
Ini identitas yang dideklarasikan, bukan verifikasi keaslian.

## 3. Arsitektur aplikasi

APK adalah shell **Cocos2d-JS / Cocos Creator** (native `libcocos2djs.so` untuk
arm64-v8a dan armeabi-v7a, ~22 MB / ~17,5 MB). Logika game berada di JavaScript:

- `assets/main.js` — bootstrap Cocos, hot-update `HotUpdateSearchPaths`, `window.boot()`.
- `assets/project.json` — `jsList`: `src/resource.js`, `src/app.js`.
- `assets/src/*.jsc` — skrip terenkripsi XXTEA (termasuk `project.jsc` 620 KB dan
  `settings.jsc`). Kunci belum diperoleh; isi logika game tidak dibaca.
- `assets/jsb-adapter/jsb-builtin.js`, `jsb-engine.js` — adapter engine.

## 4. Alur startup (dari dekompilasi)

`org.cocos2dx.javascript.AppActivity.onCreate(Bundle)` (Evidence `ev_d18ddf54…da71dd31c9df0747fb1d2fdf2a99e635b691d32464821121ac7a8ff9`):

```
super.onCreate(bundle)
if (isTaskRoot()) {
  sCocos2dxActivity = this
  getWindow().addFlags(128)      // FLAG_KEEP_SCREEN_ON
  showSplash()
  SDKWrapper.getInstance().init(this)
  b = g.b(Cocos2dxActivity.getContext())
  PlatformAndroidApi.channelstr = b.a()
  PlatformAndroidApi.channelex = ProjUtil.map2json(b.b()).toString()
  onOpenAppByUrl(getIntent())    // deep link
  PlatformAndroidApi.googleService = onCheckGooglePlayService()
  createNotificationChannel()
  PlatformAndroidApi.reGetFMCToken()
}
```

`onNewIntent(Intent)` meneruskan intent ke `SDKWrapper` dan `onOpenAppByUrl`.
Aktivitas juga mendeklarasikan intent-filter `VIEW` + `BROWSABLE` dengan scheme/host
dari `@string/deep_link_scheme` / `@string/deep_link_host`.

## 5. Jembatan Java ↔ JavaScript

`JsTool` (Evidence `cls_JsTool.json`) memiliki konstanta:

- `objPath = "cc.vv.PlatformApiMgr"`
- `fName = "trigerCallback"`

artinya hasil dari API native dikirim balik ke objek JS `cc.vv.PlatformApiMgr`
melalui `callJsFunc(...)` / `sendToPlatformApiCbFunc(...)`.

`PlatformAndroidApi` (Evidence `cls_PlatformAndroidApi.json`) mengekspos ke JS,
antara lain:

- Identitas perangkat: `getDeviceId`, `getAnroidId` (di `ProjUtil`), `getDeviceBrand`,
  `getDeviceModel`, `getGSFId` / `getGSFID`, `getSimOperator`, `getSimacardid`,
  `getSingure`, `getContracts`, `getAPPBundleId`, `getAppVersion`,
  `getApplicationLanguage`, `getChannelstr`, `getChannelExStr`.
- Notifikasi: `getFMCToken`, `reGetFMCToken`, `subscribeFCMTopic`,
  `unsubscribeFCMTopic`.
- Clipboard: `getTxtFromClipboard`, `setTxtToClipboard`.
- Anti-cloner / deteksi lingkungan: `isCloner`, `isInstallFB`, `isOpenGPS`,
  `checksignture` (di `ProjUtil`), `googleService`.
- Pembayaran / toko: `SdkPay`, `queryAllSKU`, `gpCheckOwned`, `googleInstallReff`.
- Sosial / iklan: `fbSdkLogin`, `fbSdkLoginOut`, `fbSdkShare`, `FbFriendsInApp`,
  `OpenFB`, `googleLogin`, `loadAdMobRewardAd`, `loadTradPlusRewardedVideo`,
  `showTradPlusRewardedVideo`, `ADTrackEvent`, `KoTrackEvent`.
- Lainnya: `callPhone`, `openURL`, `openRating`, `loadReviewComment`, `sendMail`,
  `takePhoto`, `SaveToAlumb`, `SaveUrlToAlumb`, `SystemImageShare`,
  `joinVoiceChannel`, `setRemoteMute`, `setLocalMute`, `phoneShock`.

`ProjUtil` juga memuat `readApk`, `getPackagePath`, `isAppInstall`, `getHtmlStream`,
`bitmapToBase64`, `callJS`. `ImageUtils` menyediakan `uriToBase64` /
`getBase64WithHeader`.

Ini adalah **kemampuan yang dideklarasikan**, bukan bukti bahwa semua dipanggil saat
runtime. Refleksi, pemuatan dinamis, dan pemanggilan yang belum terselesaikan tetap
tidak diketahui.

## 6. Hot-update / katalog game

Aset `assets/res/raw-assets/**/*.manifest` berisi **254 manifest** untuk 253 sub-game
(slot, kartu, tembak, dll.). Contoh:

```json
{"version":"0.0.1","name":"Zeppelin","zhName":"Zeppelin",
 "packageUrl":"http://ifs.vew729sh2isjo9o.com/GameX/1.1.3.9/Zeppelin",
 "remoteManifestUrl":"http://ifs.vew729sh2isjo9o.com/GameX/Zeppelin/project.manifest"}
```

Distribusi host (dari `analysis/hotupdate_manifests.txt`):

| Host | Jumlah manifest |
|---|---|
| `http://ifs.vew729sh2isjo9o.com` | 251 |
| `http://ifsa.vevoso09slcom.com` | 2 |
| `http://ifs.fwui238929sh.com` | 1 (versi `1.1.1.3`) |

Pola: shell APK mengunduh paket game tambahan dari domain `ifs.*` pada versi
`GameX/1.1.3.9`. Ini menjelaskan klaim situs "70+ game": katalog sebenarnya jauh
lebih besar dan dimuat dari server eksternal. Konten server tersebut **tidak**
diunduh atau dianalisis di sini.

## 7. Yang tidak diketahui / batas

- Isi `project.jsc` dan `settings.jsc` (kunci XXTEA belum ditemukan), sehingga alur
  login, API backend, taruhan, dan logika kemenangan tidak dapat dipastikan.
- Tidak ada pengamatan runtime: tidak diketahui kapan API-native dipanggil, data apa
  yang dikirim, atau ke mana FCM token dikirim (`sendRegistrationToServer` ada,
  tetapi isinya tidak dibaca).
- Signature tidak diverifikasi secara kriptografis; ini hanya identitas yang
  dideklarasikan.
- Manifest dan dekompilasi adalah observasi statis, bukan pembuktian perilaku atau
  penegakan kebijakan.

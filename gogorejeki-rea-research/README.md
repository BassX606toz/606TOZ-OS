# Gogo Rejeki — Analisis Statis REA (riset keamanan)

Repo ini berisi **artefak analisis statis** dari aplikasi Android `com.gogo.rejeki.games`
(nama lokal `gogorejeki.apk`) yang diperoleh dari `https://www.use-gogorejeki.com/`.

## Batasan ruang lingkup (penting)

- Ini **bukan** repositori source yang bisa di-build, bukan fork, dan bukan paket
  distribusi aplikasi. Tidak ada APK, tidak ada aset game, dan tidak ada kode yang
  diubah atau dibuat ulang di sini.
- Tidak ada modifikasi GUI, tidak ada penggantian nama merek, dan tidak ada
  rekayasa hasil/winrate. Permintaan-permintaan tersebut ditolak.
- Tujuan repo ini hanya dokumentasi riset keamanan dan indicator of compromise (IOC)
  untuk keperluan pelaporan/takedown.

## Isi

- `FINDINGS.md` — ringkasan cara kerja aplikasi berdasarkan bukti statis REA/JADX.
- `IOC.md` — hash, domain, package, signer, endpoint distribusi.
- `analysis/hotupdate_manifests.txt` — inventaris 254 manifest hot-update game.
- `evidence/` — catatan Evidence JSON hasil REA CLI 6.10.1 (JADX headless 0.7.1).
  - `apk_evidence.json`, `apk_full.json` — identitas & manifest APK.
  - `sig_evidence.json` — skema dan identitas penandatanganan.
  - `classes_all.json` — inventaris 1.528 kelas DEX.
  - `cls_*.json` — inventaris member kelas utama.
  - `meth_*.json` — dekompilasi method terpilih.

## Reproduksi (analisis, bukan build)

Analisis dilakukan dengan REA CLI tanpa menjalankan APK:

```bash
export REA_JADX_MCP_JAR=/abs/path/jadx-headless-mcp-0.7.1-all.jar   # SHA-256 6e5eacf500b64292bfb73c49797c1958f6ee44646e43e868039ae7feb573ff75
npx -y rea-agents@6.10.1 inspect-android-package <apk> --detail summary --evidence_output out.json
npx -y rea-agents@6.10.1 search-android-classes <apk> "" --format json
npx -y rea-agents@6.10.1 inspect-android-method <apk> org.cocos2dx.javascript.AppActivity onCreate
npx -y rea-agents@6.10.1 inspect-apk-signature <apk> --detail summary
```

APK tidak pernah dieksekusi; tidak ada emulator, perangkat, atau runtime target yang
digunakan. Semua kesimpulan adalah observasi statis; perilaku runtime tidak dibuktikan.

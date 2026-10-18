# AvenHome for Android

The AvenHome family finance ledger, packaged as an installable Android app with Capacitor.
The app is fully offline: the web files, fonts and the PDF/Excel tools are bundled inside the package.

## What is in this folder

| Path | What it is |
|---|---|
| `www/` | The app (also a working installable web app / PWA: manifest, service worker, icons). |
| `android/` | The Android Studio / Gradle project (package `com.avenhome.app`, minSdk 24, targetSdk 36). |
| `capacitor.config.json` | Capacitor settings. Change `appId` here **and** in `android/app/build.gradle` if you want a different package name. It cannot be changed after the first Play upload. |
| `.github/workflows/android.yml` | Builds the APK and the signed AAB on GitHub for you. No Android Studio needed. |
| `make_keystore.sh` | Creates your Play upload key. |
| `store/` | Feature graphic, phone screenshots and the Play listing text. |
| `www/privacy.html` | Privacy policy page (Play requires a public URL for it). |

## Easiest route: let GitHub build it

1. Create a private GitHub repository and push this folder to it.
2. Open the **Actions** tab, choose **Build AvenHome Android**, and press **Run workflow**.
3. When it finishes, download **AvenHome-debug-apk** from the run page. Copy the `.apk` to your phone and open it (allow "install unknown apps" once). This is the file to test with.
4. For Google Play you need a signed bundle:
   1. Run `./make_keystore.sh` (needs Java). Keep `avenhome-release.jks` and the passwords safe. Do not commit them.
   2. In the repository: Settings, Secrets and variables, Actions, and add `AVEN_KEYSTORE_B64` (output of `base64 -w0 avenhome-release.jks`), `AVEN_STORE_PASSWORD`, `AVEN_KEY_ALIAS` (`avenhome`) and `AVEN_KEY_PASSWORD`.
   3. Run the workflow again. It also produces **AvenHome-release-aab**, the file you upload to Play.

## Local build (optional)

Needs Node 22, Java 21 and the Android SDK (Android Studio installs it).

```
npm ci
npx cap sync android
cd android
./gradlew assembleDebug      # android/app/build/outputs/apk/debug/app-debug.apk
./gradlew bundleRelease      # needs keystore.properties or AVEN_* env vars
```

`keystore.properties` format: `storeFile=../avenhome-release.jks`, `storePassword=...`, `keyAlias=avenhome`, `keyPassword=...`.

To release an update: raise `AVEN_VERSION_CODE` (the workflow uses the run number) and rebuild.

## How exports work on Android

PDF, Excel, CSV and backup files are written to the app cache, then the Android share sheet opens so the person can save to Files or Drive, or send by email or chat. The print-ready web pages are hidden on Android (PDF replaces them).
Data lives in the app's private storage. Clearing app storage or uninstalling deletes it, so users should make a backup from Export.

## Before submitting to Google Play

- Add your contact email to `www/privacy.html` and host that page at a public address (GitHub Pages works). Put the URL in Play Console, App content, Privacy policy.
- Play Console: create the app, choose **Play App Signing**, upload the `.aab` to an internal test track first.
- New personal developer accounts must run a closed test with at least 12 testers for 14 days before production access. Check Google's current rule for your account type.
- Fill in Data safety (see `store/listing.md`), content rating and target audience.
- The app contains two NIV Bible verses supplied by the owner, with the Biblica notice shown. Check that your use, especially if you charge for the app, is permitted by the NIV licence.
- Monetization: the app has no payment code. A paid app, in-app purchases or subscriptions on Google Play must use Google Play Billing, which is a separate piece of work.

## Not yet tested

This package was prepared in an environment that could not download the Android SDK, so the APK/AAB has **not** been compiled or run on a device or emulator. The web app inside was tested in Chromium (offline, install, exports, simulated Android save/share). First things to check on a real phone: install and launch, the status bar and notch spacing, the Back button, Export then Share to Files, backup and restore, and rotating the screen.

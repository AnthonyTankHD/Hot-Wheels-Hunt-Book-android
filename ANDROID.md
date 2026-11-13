# Building the Android APK

This turns the Hunt Book into a standalone Android app. Nothing runs on a
server: the same actions from `server/src/actions.ts` run inside the app,
on SQLite (WebAssembly) saved to the phone, with photos/videos/music kept
in the app's own storage.

## Easiest: build it in the cloud (no Android Studio)

1. Put this folder in a GitHub repo (private is fine).
2. Open the repo's **Actions** tab, pick **Build Android APK**, press **Run workflow**.
3. When it finishes (about 5–10 minutes), open the run and download the
   **hot-wheels-hunt-book-apk** artifact. Unzip it to get `app-debug.apk`.
4. Copy the APK to your phone and open it. Allow "install unknown apps"
   for your file manager or browser when Android asks.

## Or build on your own computer

Needs Bun, Node 22+, JDK 21 and the Android SDK (Android Studio installs it).

```bash
bun install
bun run build:android          # bundle -> client/dist-android/
node scripts/prepare-android.mjs   # adds android/, camera permission, icon
cd android && ./gradlew assembleDebug
# APK: android/app/build/outputs/apk/debug/app-debug.apk
```

## What is different from the desktop version

- Data lives in the app (IndexedDB). **Uninstalling the app deletes it** —
  use Backup & Settings → Download everything now and then. Backups and
  CSV/ZIP exports open the Android share sheet so you can save them to Files or Drive.
- The two "supplied backup / songs" import buttons report "not found"
  (they read files from a computer). Restoring a backup `.json` from the
  file picker works as before.
- Chat uses its built-in fallback replies (no AI model), same as the
  standalone build.
- `bun run start` (the desktop server) still works unchanged.

## Notes

- This is a **debug** APK, fine for your own phone. A Play Store release
  needs a signing key and `./gradlew bundleRelease`.
- The app id is `com.huntbook.hotwheels` (`capacitor.config.json`);
  changing it later makes Android treat it as a different app.
- `assets/icon.png` was upscaled from the 256px `icon.webp`; a bigger
  source image gives a sharper launcher icon.

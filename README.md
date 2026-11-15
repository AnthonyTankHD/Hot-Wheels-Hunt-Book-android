# Hot Wheels Hunt Book — Android

The Android version of the Hot Wheels Hunt Book: your whole collection, hunts, music, and now online Chat Rooms, in one app on your phone.

## What it does

- **Collection** — 5,188-car catalog. Mark cars owned, Treasure Hunt / Super Treasure Hunt, add photos, track duplicates and notes. Your data lives on your phone (IndexedDB), so back it up — uninstalling the app deletes it.
- **Chat Rooms** — sign up with an email + password and talk to other collectors in the app. Preset rooms: General, Trading, Show Off, Treasure Hunts — plus anyone can create their own room. Powered by Firebase (Auth + Firestore). The Firebase config lives in `client/src/chat/firebase-config.ts`; the Firestore rules are in `firestore.rules`.
- **Music Hub** — scans the music on your phone (songs, albums, artists, playlists) and plays it in the app, with background playback. Deezer search, charts, and 30-second previews, plus cover art matched to your songs.
- **Import castings** — scan allhotwheels.com from Settings and add new castings to the book (about 1.5 seconds per page, pause and resume whenever).
- **Games** — complete rosters for Hot Wheels Unleashed, Unleashed 2 Turbocharged, and Infinite Rush, including DLC.
- **Updates** — Settings → Check GitHub for updates. Every cloud build publishes a release here, and the app can pull the new version itself.

## Get the app

1. Go to **Actions** → **Build Android APK** → **Run workflow**.
2. When it finishes, download the **hot-wheels-hunt-book-apk** artifact, unzip it, and install `app-debug.apk` on your phone.
3. It is a debug build, so Android will ask about installing from unknown sources — that is normal. It installs over the previous version; do not uninstall first or you will lose your on-device data.

## Build it yourself

The ZIP at the repo root is the whole project. The workflow unzips it, installs dependencies with Bun, builds the web app, prepares the Capacitor Android project, and builds the debug APK with Gradle. To build locally you need Bun, Node 22, JDK 21, and the Android SDK (platform 36, build-tools 36.0.0).

## Make it yours (no code)

The `customize/` folder: `app.json` (name, ID, colors), `icon.png`, `my-cars.csv` (your extra cars), `cars.csv` (full list), `lists.json` (tags and filters). Run `scripts/apply-customize.mjs` to apply changes.

## Current build

v60-2026-11-14-chat-rooms-live — Chat Rooms with live Firebase backend.

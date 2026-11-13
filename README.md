# Hot Wheels Hunt Book

A Hot Wheels collection tracker — MIT licensed, open source.

Track your collection against a **5,188-car catalog**, mark cars owned,
Treasure Hunts and Super Treasure Hunts, add photos, plan hunts with
targets and a budget, earn badges, streaks, points and Rewards Shop
unlocks, keep a music playlist, and browse a video feed — all in the
original blue/flame Hot Wheels style with the gold Ultimate Edition layer.

This is the standalone open-source build (Build v51 source, November 2026).
It runs on your own computer with **Bun + React + Drizzle + SQLite** — no
Muse account, no hosted service, no third-party API needed. Your data
stays on your machine, in `data/hunt-book.db` and `data/blobs/`.

> **Origin:** this started as a Muse fullstack app built for collector
> Anthony Tankersley, from his own original Hunt Book HTML app. The
> catalog data is his own file, expanded with castings from
> allhotwheels.com and the Hot Wheels game rosters. The standalone
> version here is the same source with the Muse-only runtime replaced
> by a plain local server — see "How this differs from the Muse version"
> below.

## Quickstart

Requirements: **Bun 1.3+** (install from the Bun website).
Node is not needed. A fresh clone starts with an **empty collection** —
only the catalog is seeded.

```bash
git clone <your-repo-url> hot-wheels-hunt-book
cd hot-wheels-hunt-book
bun install
bun run build
bun run start
```

Then open **http://localhost:3000**.

| Command | What it does |
|---|---|
| `bun install` | Installs dependencies from npm (React, Drizzle, Tailwind, Zod…) |
| `bun run build` | Builds the React client into `client/dist/` |
| `bun run start` | Applies database migrations, then serves the app on port 3000 |
| `bun run dev` | Same as `start` (the server serves the built client) |
| `bun run typecheck` | Type-checks client and server |

### Options

```bash
PORT=8080 bun run start              # listen on another port
HOST=127.0.0.1 bun run start         # localhost only (default: 0.0.0.0)
HUNT_BOOK_DB=/path/to/book.db bun run start    # database file location
HUNT_BOOK_BLOBS=/path/to/blobs bun run start  # uploaded files location
```

Defaults: database `data/hunt-book.db`, blobs `data/blobs/`, port `3000`.

## Features

- **Catalog & garage** — 5,188 cars (mainline, Fantasy, Treasure Hunts,
  Super Treasure Hunts, Unleashed 1 / Unleashed 2 / Infinite Rush game
  rosters incl. DLC). Mark owned / Treasure Hunt / Super Treasure Hunt
  (photo required, as in the original), duplicates, notes, favorites,
  custom cars, A–Z jump, search, Targets-only and layout controls.
- **Photos & videos** — add photos (with thumbnails) and videos per car
  or to the gallery; photo gallery with layout controls.
- **Hunt Planner** — per-month hunt days, hunt targets, budget/spent,
  and a hunt-prep checklist.
- **Badges, streaks, points & Rewards Shop** — 17 badges, daily-visit
  streak, points (+10 new car, +2 extra copy, +50 TH, +100 STH,
  +5 photo, +25 target found, +5 daily visit), levels up to Hot Wheels
  Legend, and a shop for titles, frames and badges (points only).
- **Music** — pick MP3s from your computer into a playlist with cover
  art, drag reorder, sleep timer, crossfade, play speed, skip silence.
- **Video Feed** — your own videos plus a YouTube list (opens on
  YouTube; embedded playback depends on YouTube's embed rules).
- **Chat** — see "Chat in the standalone build" below.
- **Diagnostics** — live counts, background & performance scan, and a
  Performance Test.
- **Backup & Settings** — export/import the whole collection as JSON
  (the original app's backup shape), download progress as CSV, photos
  as a ZIP, light/dark mode.

## Your data

Everything is local:

- `data/hunt-book.db` — SQLite database (Drizzle schema in
  `server/src/schema.ts`, migrations in `drizzle/0001`–`0011`).
- `data/blobs/` — uploaded photos, videos, music and covers, one file
  per item.
- `data/` is git-ignored — it is **not** part of the source and is
  never committed. Back it up by copying the folder, or use
  **Backup & Settings → Download everything** in the app.

### Restoring a Hunt Book backup

If you have a backup JSON from the original app or the Muse app
(Backup & Settings → Download everything / copy-paste backup):

1. In the app, go to **Backup & Settings → Restore from a backup file**
   and pick the `.json` file (or paste it into Copy/paste backup).
2. Restore **merges** — it never deletes what is already in the book.
   Owned / TH / STH flags, counts, notes, custom cars, planner, targets
   and embedded photos are unioned in.

### The supplied backup & songs (optional)

The Muse version shipped with two one-off imports that read files from
Anthony's computer: the full backup from 2026-11-04
(`importSuppliedBackup`) and 7 songs from 2026-11-04
(`importSuppliedSongs`). Those personal files are **not** in this repo.
To use those two buttons in the standalone build, drop the files into a
`supplied/` folder in the project root (or set
`HUNT_BOOK_SUPPLIED_DIR`):

- `supplied/hunt-book-backup-2026-11-04_12_llek.json`
- `supplied/2-20_Adrenaline_0_dztz.flac` (converted to MP3 on import;
  needs `ffmpeg` on PATH)
- `supplied/All_Elite_Wrestling_-_Battle_Cry__Kenny_Omega_Theme__1_iaql.mp3`
- `supplied/It_Lives__It_Breathes_-_Elevated_2_roxn.mp3`
- `supplied/Kottonmouth_Kings__Tech_N9ne_-_Party_Monsters_3_r567.mp3`
- `supplied/Virtual_Riot_-_Pray_For_Riddim_4_8xmt.mp3`
- `supplied/TV-FM_-_TEKU-ISING__Shirako_Takamoto_Theme___Hot_Wheels_Acceleracers__5_l11h.mp3`
- `supplied/Excision___The_Frim_-_X_Up_feat_Messinian_6_mwlc.mp3`

Without them the buttons report, honestly, that the files were not
found. `supplied/` is git-ignored.

## Chat in the standalone build

In the Muse app, the in-app chat (Hot Wheels Manager — answers from
the live collection, can mark a car owned / Treasure Hunt or add a
target when clearly asked; Hot Wheels Guru — fun chat only, never
changes data) was powered by Muse's model.

**There is no model bundled with this source and no API key
configured**, so the standalone chat cannot generate AI replies. What
it does instead, by design:

- Your message is still saved to the chat history.
- You get a grounded fallback reply built from the live collection
  (owned/total/TH counts) that says plainly it could not think the
  message through — it never invents a car, price, or answer.
- **Nothing is marked, added, or changed** by a fallback reply.
- Hot Wheels Guru's fallback is the same shape, fun-toned, and also
  changes nothing.

To wire a real model in, implement `inference.complete()` in
`server/src/server.ts` — it receives the full prompt (collection
snapshot + recent history) and a Zod schema, and must return an
object matching that schema. Keep the existing guardrails: answer
only from the snapshot, never invent cars/prices/stock, and let the
server (not the model) apply any collection change.

## How this differs from the Muse version

| | Muse version | This standalone version |
|---|---|---|
| Runs on | Muse's hosted runtime | Your computer (`bun run start`) |
| Data | Muse-hosted database + blob storage | Local SQLite `data/hunt-book.db` + `data/blobs/` |
| Server actions | Muse `/actions` RPC, `ctx.db` / `ctx.blobs` | Plain Bun server (`server/src/server.ts`), same action names over `POST /actions` |
| Auth | Owner-only guard via Muse viewer | Single-user local app, no login (don't expose it to the internet) |
| Chat AI | Muse model (`ctx.inference`) | Grounded fallback replies only (see above) |
| Supplied backup/songs | Read from the host by a privileged bundle | Read from `./supplied/` in-process, if you provide the files |
| Build | Muse artifact pipeline | `bun run build` (plain `Bun.build`, Tailwind plugin) |
| Your collection | Anthony's live data | Empty to start — restore a backup to load a collection |

The UI, catalog, styling, points rules, badge thresholds, backup
format and action names are unchanged from Build v51. The code that
changed to make it standalone lives in:

- `server/src/sdk.ts` — local replacement for the Muse SDK server
  surface (`defineAction`, `z`, `Ctx`, `SpaceDb`, privileged contracts)
- `server/src/server.ts` — the Bun server: SQLite + migrations,
  local blob files, `POST /actions`, `GET /blobs/*`, static client
- `client/src/sdk-client.ts` — local replacement for the Muse SDK
  client surface (`createActionClient`, `spaceQueryClient`,
  `SafeAreaTopScrim`, file/base64 helpers)
- `client/build.mjs` — plain `Bun.build` client bundle (was the SDK's
  `buildClient()`, which refuses hand builds)
- `server/src/privileged.ts` — supplied-file reads now look in
  `./supplied/` instead of a Muse host path

## Project layout

```
hot-wheels-hunt-book/
  client/            React app (App.tsx is the whole UI, 8,768 lines)
    src/App.tsx      Home, All Cars, Gallery, Planner, Badges, Points/
                     Rewards, Music, Video Feed, Chat, Diagnostics,
                     Backup & Settings, Press Start splash + jingle
    src/catalog.ts   5,188-car catalog (client copy)
    src/theme.css    Blue/flame styling + gold Ultimate Edition layer
    src/api.ts       Typed RPC client (POST ./actions)
    src/sdk-client.ts  Local SDK-client replacement
    src/assets/      Jingle, logos, splash wallpaper, game logos,
                     YouTube thumbnails, phone-copy.htm
  server/
    src/actions.ts   All actions: getState, setCarState, photos, music,
                     planner, points, backup, chat, reset…
    src/schema.ts    Drizzle table definitions (18 tables)
    src/catalog.ts   5,188-car catalog (server copy)
    src/privileged.ts  Supplied backup/song readers
    src/sdk.ts       Local SDK-server replacement
    src/server.ts    Bun server + SQLite + local blobs
  drizzle/           Migrations 0001–0011 (+ journal)
  DATA-PLAN.md       Build-by-build history from the Muse version
  LICENSE            MIT — Copyright (c) 2026 Anthony Tankersley
```

## Contributing

Issues and pull requests are welcome. A few ground rules, carried
over from the original app:

- **Keep the original look.** The blue/flame style (plus the gold
  Ultimate Edition layer) is the app's identity — don't redesign it.
- **Don't drop the photo rule.** Marking a car owned or a Treasure
  Hunt requires a photo of that car.
- **Don't invent catalog data.** Cars come from real sources
  (the original app's catalog, allhotwheels.com, game rosters).
  Never invent a casting, a price, or store stock.
- **Backups merge, never delete.** Restore must union with existing
  rows, never wipe them.
- Run `bun run typecheck` and `bun run build` before sending a PR,
  and say what you tested on a real run (`bun run start` + browser).

## License

MIT — see LICENSE. Copyright (c) 2026 Anthony Tankersley.

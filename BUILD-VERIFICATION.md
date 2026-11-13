# Build verification — standalone open-source Hot Wheels Hunt Book

Verified 2026-11-12 on this VM (Linux x64) by building from the
packaged source. Source: Muse Build v51, converted to standalone
(see README.md "How this differs from the Muse version").

## Environment used

- Bun **1.4.4** (`bun --version`), Node 24.22.1 also present (not used)
- `package.json` asks for Bun 1.3.10; 1.4.4 worked
- `/opt/hatch` **does** exist on this VM, but nothing in this tree
  references it: `grep -rn "@hatch\|/opt/hatch" package.json bun.lock`
  returns nothing, and there is no `@hatch` package in `node_modules`
  after install. A normal computer without `/opt/hatch` installs the
  same way.

## Commands and results (in order)

```bash
cd hot-wheels-hunt-book-open-source
bun install
# Resolving dependencies / Resolved, downloaded and extracted [228]
# + 58 packages installed [14.45s]
# (one warning only: recharts@2.15.3 is deprecated, 1.x/2.x unmaintained)

bun run typecheck
# (no output — client and server both pass; exit 0)

bun run build
# client build ok: 21 outputs in ./client/dist
# client/dist is 5.6 MB (index.html + index-*.js/css + assets:
# jingle, logos, splash wallpaper, game logos, YouTube thumbnails,
# phone-copy.htm)

PORT=3000 HUNT_BOOK_DB=/tmp/hh-test.db HUNT_BOOK_BLOBS=/tmp/hh-blobs \
  bun run server/src/server.ts
# Hot Wheels Hunt Book (standalone) listening on http://0.0.0.0:3000
# Database: /tmp/hh-test.db
# Blobs:    /tmp/hh-blobs
```

HTTP checks against that running server (all with `curl`):

| Check | Result |
|---|---|
| `GET /` | 200, 752 bytes (built index.html, JS/CSS asset refs) |
| `GET /assets/index-*.js` | 200 |
| `POST /actions {"action":"getState","args":{}}` | 200, `total 5188, owned 0, photos 0, targets 0` (fresh DB: 5,188 catalog entries; the Muse app's 5,195 total included 7 custom cars, which are user data, not catalog) |
| `POST /actions setCarState {carKey:"boneshaker", owned:true, count:1}` | `{"ok":true,"points":{"awarded":10,"balance":10,"lifetime":10,"level":1,"levelName":"Rookie"}}` |
| `POST /actions addTarget {month:"2026-11", text:"Bone Shaker"}` | `{"added":true,"id":1}` |
| `POST /actions addPhoto` (1x1 PNG + thumbnail, base64) | `{"id":1,"points":{"awarded":5,"balance":15,"lifetime":15}}` |
| `GET /blobs/photos/<key>` | 200, `image/png`, 70 bytes |
| `POST /actions exportBackup` | JSON with `own: ["boneshaker"]`, plan month `2026-11` |
| `POST /actions recordVisit` | `{"current":1,"best":1,"points":{"awarded":5,"balance":20,"lifetime":20}}` |
| `POST /actions getState` (after writes) | `owned 1, copies 1, targets 1, photos 1, streak 1/1, points lifetime 20` |
| `POST /actions sendChatMessage` (manager, "How many cars do I own?") | Grounded fallback reply, no AI: "I couldn't think that one through just now, so I won't guess at an answer. Here's where your Hunt Book stands: Collection: 1 of 5188 owned, 5187 missing, 0 Treasure Hunts owned…" — nothing marked or changed |
| `POST /actions importSuppliedBackup` (no `supplied/` files present) | `{"ok":false,"error":"The supplied backup file (hunt-book-backup-2026-11-04_12_llek.json) was not found, so nothing was imported."}` |
| `POST /actions {"action":"nope"}` | 404 `{"error":"unknown action: nope"}` |
| `POST /actions resetAll` then `getState` | `owned 0, photos 0, total 5188` |
| Migrations on the test DB | 18 tables (`__drizzle_migrations` + 17 app tables), all 11 migrations applied |

Default-path run (no env vars, fresh `data/`):

```bash
rm -rf data client/dist
bun run build        # client build ok: 21 outputs
PORT=3001 bun run start
# listening on http://0.0.0.0:3001
# Database: <project>/data/hunt-book.db
# GET / -> 200
# getState -> total 5188, owned 0
```

Fresh-extraction check: the ZIP was unzipped to a clean directory
(`/tmp/fresh-hhb/hot-wheels-hunt-book-open-source`) and the full
sequence passed there: `bun install` (58 packages), `bun run
typecheck` (clean), `bun run build` (21 outputs), `bun run start`
on port 3002, `GET /` -> 200, `POST /actions getState` ->
`total 5188, owned 0`.

## What was NOT verified

- **Browser rendering.** No browser was driven against the running
  server in this check — only `curl` (page HTML, JS asset, and the
  action API). The client bundle is byte-comparable in size/content
  to the diagnostic bundle from the Muse source (5.6 MB, 21 outputs),
  and the React app mounts into the same root element, but visual /
  interaction testing in a real browser was not done here.
- **A machine without `/opt/hatch`.** The install was run on this VM
  where `/opt/hatch` exists, but the package manifest and lockfile
  contain no reference to it and no `@hatch` package is installed,
  so the install path is npm-only. It was not run on a second,
  hatch-free machine.
- **ffmpeg song import.** `importSuppliedSongs` was not exercised:
  the supplied song files are not in this repo. `ffmpeg 8.1.2` is
  present on this VM.
- **YouTube embeds.** The Video Feed's YouTube tab was already
  broken in the Muse app (Error 153 on the phone); nothing about
  that changed here.

## Known standalone limitations (by design)

- In-app chat has no model: replies are grounded fallbacks only
  (see README.md). Nothing is ever marked/changed by a fallback.
- Fresh installs start with an empty collection (catalog only).
  Restore a Hunt Book backup JSON to load a collection.
- Single-user, no login: do not expose the server to the internet.
- The supplied backup/songs buttons need files in `./supplied/`
  (not shipped — personal files).

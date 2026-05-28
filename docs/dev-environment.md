# Dev Environment — Moodle Mobile App (Windows)

## Machine: Windows 10 Pro (Asad, Rawalpindi)

Last updated: 2026-05-28

---

## Required Tools & Installed Versions

| Tool | Required | Installed | Status |
|------|----------|-----------|--------|
| Node.js | `>=22.17 <23` (from package.json engines) | **v24.14.1** | ⚠️ OUT OF RANGE |
| npm | any | 11.11.0 | ✅ |
| Git | any | 2.53.0.windows.2 | ✅ |
| Ionic CLI | 7.x | **7.2.1 (global)** | ✅ |
| Gulp CLI | any | **3.1.0 (global)** | ✅ |
| Jest | 30.x | 30.4.1 (npx) | ✅ |
| Chromium browser | required for dev | present | ✅ |

### Global installs (run once, already done)

```powershell
npm install -g @ionic/cli gulp-cli
```

### Node Version Warning

`package.json` specifies `"node": ">=v22.17 <23"` — meaning only Node 22.x LTS is officially supported.
You have **Node 24.14.1** which is outside this range.

**What this means:**
- `npm install` may show engine warnings
- Browser dev tasks work fine on Node 24
- If you hit weird build or dependency errors, downgrade to Node 22 LTS via nvm

**Fix if needed:**
```powershell
# Install nvm-windows from https://github.com/coreybutler/nvm-windows
nvm install 22
nvm use 22
node --version  # should show v22.x
```

---

## Current State (as of 2026-05-28)

- [x] Git cloned / project present at `F:\Asad\Freelancing\Samantha App\moodleapp`
- [x] `npm install` — DONE (1867 packages, 4 patches applied)
- [x] `gulp` — DONE (lang, env, icons all passed)
- [x] `npm start` — WORKING at https://localhost:8100
- [x] Windows bash fix applied to `package.json` (see `docs/windows-fixes.md`)

---

## How to Run (every session)

**Must use Git Bash** (not PowerShell/CMD — see Fix 2 in `windows-fixes.md`):

```bash
export NODE_OPTIONS=--max-old-space-size=8192
npm start
```

- Opens browser at **https://localhost:8100** automatically
- Accept the SSL cert warning (self-signed, expected)
- First build takes 1-3 minutes; subsequent rebuilds are fast (incremental)
- Stop with **Ctrl+C**

> Skipping the `export` line will cause a heap out-of-memory crash after a few minutes.

---

## First-Time Setup (already done — for reference only)

```powershell
npm install -g @ionic/cli gulp-cli   # install global tools
cd "F:\Asad\Freelancing\Samantha App\moodleapp"
npm install                          # install project deps + cordova-plugin-moodleapp
gulp                                 # preprocess lang/env/icons
npm start                            # start dev server
```

---

## Windows-Specific Notes

### bash fix in package.json (already applied)
`scripts/serve.sh` is a bash script. On Windows, `cross-env-shell` defaults to `cmd.exe` and can't run it.
Fix: `ionic:serve` in `package.json` now uses `bash ./scripts/serve.sh` explicitly.
See full details in [docs/windows-fixes.md](windows-fixes.md).

### Browser Dev (no extra setup needed)
`npm start` runs the app in Chromium. Works on Windows out of the box.

### SSL in Dev Server
`npm start` uses `--ssl` flag → HTTPS at `https://localhost:8100`.
Browser shows a self-signed cert warning — click through / add exception.

### Native Android Build (extra setup needed)
Only needed if building an actual APK:
```powershell
npm install --global --production windows-build-tools
# Warning: takes several hours on first run
```
Also needs Android Studio + SDK installed and `ANDROID_HOME` env var set.

### iOS Build
Not possible on Windows. Requires macOS.

---

## Env Config Override

```powershell
cp moodle.config.example.json moodle.config.local.json
# moodle.config.local.json is gitignored — edit freely
```

---

## Reference Links

- Setup guide: https://moodledev.io/general/app/development/setup
- Developer docs: https://moodledev.io/general/app
- Bug tracker: https://moodle.atlassian.net/browse/MOBILE
- nvm-windows: https://github.com/coreybutler/nvm-windows

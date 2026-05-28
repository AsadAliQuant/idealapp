# Fixes, Workarounds & Debug Sessions

Permanent fixes and detailed debug logs for this project. Future sessions don't need to redo the fixes — they're already in the codebase. The debug session logs are here as a reference if you hit the same errors again.

---

## Fix 1: ionic:serve — bash script not running on Windows

**Date:** 2026-05-28  
**File changed:** `package.json`

### Problem
`scripts/serve.sh` is a bash script. `cross-env-shell` on Windows defaults to `cmd.exe`, which can't execute `.sh` files. Running `npm start` would fail with:

```
npm has unexpectedly closed (exit code 0).
The Ionic CLI will exit. Please check any output above for error details.
```

### Root cause
The upstream `ionic:serve` script:
```json
"ionic:serve": "cross-env-shell ./scripts/serve.sh"
```
`cross-env-shell` picks the system shell. On Windows that's `cmd.exe`, not bash.

### Fix applied
```json
"ionic:serve": "cross-env-shell bash ./scripts/serve.sh"
```
Explicitly invokes `bash` (Git Bash at `/usr/bin/bash`) to run the script.

### Requirement
Git Bash must be installed (it is on this machine — `bash --version` confirms `5.2.37(1)-release`).

---

## Session 1 — Initial Setup (2026-05-28)

What was done to get the dev environment working from scratch:

| Step | Command | Result |
|------|---------|--------|
| Install global tools | `npm install -g @ionic/cli gulp-cli` | Ionic CLI 7.2.1, Gulp CLI 3.1.0 |
| Install project deps | `npm install` | 1867 packages, 4 patches applied |
| Preprocess assets | `gulp` | lang ✅ env ✅ icons ✅ |
| Apply Windows fix | Edit `package.json` ionic:serve | bash fix — see Fix 1 above |
| Start dev server | `npm start` | Running at https://localhost:8100 |

### npm install details
- 1867 packages installed
- 4 patches auto-applied via `patch-package`:
  - `@angular/compiler@20.3.18`
  - `@angular/router@20.3.18`
  - `@ionic/core@8.8.1`
  - `mp3-mediarecorder@4.0.5`
- `cordova-plugin-moodleapp` deps installed via postinstall hook
- 37 audit vulnerabilities (all in dev/build deps — do NOT run `npm audit fix --force`)

---

## Fix 2: Node.js heap out of memory during `npm start`

**Date:** 2026-05-28

### Problem
After a while, `npm start` crashes with:

```
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
```

localhost:8100 stops responding. Angular's incremental compiler eats through the default ~1.5 GB heap.

### Fix (not in package.json — must be set per session in Git Bash)
`NODE_OPTIONS=VAR=value` prefix in npm scripts doesn't work on Windows because npm uses cmd.exe to spawn scripts, even when you're inside Git Bash.

**Run these two commands before `npm start` each session:**

```bash
export NODE_OPTIONS=--max-old-space-size=8192
npm start
```

The `export` persists for the whole terminal session — you only need it once per terminal window.

---

## Future Sessions — Known Issues to Watch (Windows)

- **Node 24 vs required 22**: Works for browser dev, but if builds break unexpectedly, downgrade to Node 22 via nvm-windows.
- **SSL cert warning**: `https://localhost:8100` uses a self-signed cert — just click through in the browser.
- **First build slow**: Angular's first compile takes 1-3 min. Subsequent incremental builds are fast.

---

## Section 3 — WSL2 Ubuntu: First Successful APK Build (2026-05-28)

**Status: FULLY WORKING. APK tested and confirmed.**

This section is a complete debug log of everything that was attempted, every error hit, every fix applied, and the exact sequence that finally produced a working APK. Use this as a reference if you try to set up APK builds again on a fresh machine or hit similar errors.

### Environment at the start

- OS: WSL2 Ubuntu (running inside Windows via `wsl`)
- Project location: `~/projects/moodleapp` (inside WSL filesystem — NOT `/mnt/...`)
- Node: v22.22.3 (CLAUDE.md says 20 but 22 works fine)
- Java: OpenJDK 17.0.19
- ADB: 1.0.41 (Android SDK already set up from a previous session)
- `node_modules/` already existed from a prior install
- `package.json` had uncommitted changes (new deps added)

---

### Attempt 1 — Just run the build

**Command run:**
```bash
npm install   # package.json had changed, so sync first
gulp          # preprocess lang/env/icons
npm run prod:android
```

**`npm install` result:** Succeeded. 514 packages added (re-sync), 4 patches applied. No errors.

**`gulp` result:** Succeeded in 297ms. Lang ✅ Env ✅ Icons ✅

**`npm run prod:android` result — ERROR:**
```
sh: 1: cross-env: not found
```

The `prod:android` script is:
```
npm run prod --prefix cordova-plugin-moodleapp && cross-env NODE_ENV=production ionic cordova run android --prod
```

After the first half (`npm run prod --prefix cordova-plugin-moodleapp`) succeeds, the shell tries to run `cross-env` for the second half. `cross-env` should be in `node_modules/.bin/` but it was missing.

**Investigation:**
```bash
ls node_modules/.bin/cross-env   # → No such file or directory
```

Checked `package.json` — `cross-env: ^10.1.0` IS listed in `devDependencies`. It was in the package.json but npm had not placed it in `node_modules/.bin/`. Root cause suspected: the postinstall hook (`cd cordova-plugin-moodleapp && npm install`) runs after root install and interferes with npm's hoisting, leaving `cross-env` unlinked.

---

### Attempt 2 — Install cross-env manually

**Command run:**
```bash
npm install cross-env --save-dev
```

**Result:** "added 514 packages" (npm re-ran everything). Then checked:
```bash
ls node_modules/.bin/cross-env   # → EXISTS ✅
```

**Re-ran:** `npm run prod:android`

**Result — NEW ERROR, same symptom but different location:**
```
sh: 1: cross-env: not found
```

This time it was the `cordova-plugin-moodleapp` sub-package failing. The script `npm run prod --prefix cordova-plugin-moodleapp` runs npm in the `cordova-plugin-moodleapp/` subdirectory, which has its OWN `node_modules/`. That subdirectory also needs `cross-env`, and it wasn't installed there.

**Investigation:**
```bash
ls cordova-plugin-moodleapp/node_modules/.bin/cross-env   # → No such file or directory
```

Checked `cordova-plugin-moodleapp/package.json` — `cross-env: ^10.1.0` IS in its devDependencies.

---

### Attempt 3 — Install cross-env in both locations

**Commands run:**
```bash
npm install cross-env --save-dev --prefix cordova-plugin-moodleapp
```

**Result:** Added 76 packages in `cordova-plugin-moodleapp/node_modules/`. Checked:
```bash
ls /home/asad/projects/moodleapp/cordova-plugin-moodleapp/node_modules/.bin/cross-env   # EXISTS ✅
```

But then discovered root `cross-env` had vanished again:
```bash
ls node_modules/.bin/cross-env   # → No such file or directory (disappeared again!)
```

**Root cause identified:** Every time `npm install` runs (including via postinstall or `--prefix`), it either re-runs the postinstall hook or re-hoists packages, wiping the `cross-env` binary link from root `node_modules/.bin/`.

**Fix that finally stuck:**
```bash
npm install cross-env@10.1.0 --save-dev --ignore-scripts
```

The `--ignore-scripts` flag skips the postinstall hook (`cd cordova-plugin-moodleapp && npm install`). Without that hook interfering, `cross-env` stays properly linked.

Then separately (to ensure the cordova-plugin-moodleapp dir also has its deps):
```bash
cd cordova-plugin-moodleapp && npm install && cd ..
```

After this, both locations had `cross-env`:
```bash
ls node_modules/.bin/cross-env                                     # ✅
ls cordova-plugin-moodleapp/node_modules/.bin/cross-env            # ✅
```

**Re-ran:** `npm run prod:android > /tmp/build.log 2>&1`

---

### Attempt 4 — cross-env fixed, new error: Gradle not found

The build now got much further. The Angular bundle compiled fully (4.98 MB initial bundle, hundreds of lazy chunks). Then Cordova kicked in:

```
Could not find an installed version of Gradle either in Android Studio,
or on your system to install the gradle wrapper. Please include gradle
in your path, or install Android Studio
```

Exit code 1.

**Investigation:**
```bash
which gradle          # → nothing
ls platforms/android/gradlew   # → No such file or directory
ls ~/Android/Sdk/     # → has build-tools, platform-tools, platforms — but NO gradle
```

The Android SDK (installed via `sdkmanager`) does NOT include Gradle. Gradle is a separate tool. The `platforms/android/` directory had all the Gradle build files (`.gradle`, `build.gradle`, etc.) but was missing the `gradlew` wrapper script that normally auto-downloads Gradle.

**Attempts to install Gradle:**

1. `sudo apt-get install -y gradle` → `sudo: A terminal is required to authenticate` (no interactive sudo in Claude Code)
2. `sdk install gradle` (SDKMAN) → SDKMAN not installed
3. Searched `~/Android/Sdk` for any gradle binary → nothing found

**Solution — download Gradle manually (no sudo needed):**
```bash
mkdir -p ~/gradle-dist && cd ~/gradle-dist
wget -q https://services.gradle.org/distributions/gradle-8.14.2-bin.zip
unzip -q gradle-8.14.2-bin.zip
```

Then added to PATH before running the build:
```bash
export PATH="$HOME/gradle-dist/gradle-8.14.2/bin:$PATH"
gradle --version   # → Gradle 8.14.2 ✅
```

Why 8.14.2 specifically? Found in `platforms/android/cdv-gradle-config.json`:
```json
{ "GRADLE_VERSION": "8.14.2" }
```

**Re-ran:** `export PATH="..." && npm run prod:android > /tmp/build.log 2>&1`

---

### Attempt 5 — Gradle found, but cross-env vanished AGAIN

The build started, Cordova plugin built, then:
```
sh: 1: cross-env: not found
```

Cross-env had disappeared from root `node_modules/.bin/` again. This happened because the build process somewhere triggered an npm install that ran the postinstall hook, which wiped the cross-env link.

**Applied the `--ignore-scripts` fix again:**
```bash
npm install cross-env@10.1.0 --save-dev --ignore-scripts
ls node_modules/.bin/cross-env   # ✅
```

**Re-ran:** `export PATH="..." && npm run prod:android > /tmp/build.log 2>&1`

---

### Attempt 6 — Angular builds, Gradle builds, APK produced — but exit code 1

This was the most confusing point. The full build output showed:

```
✔ Browser application bundle generation complete.   ← Angular build succeeded
✔ Copying assets complete.
✔ Index html generation complete.

Initial chunk files | main.5fd027a2.js | 4.04 MB
...hundreds of lazy chunks...
Build at: 2026-05-28T08:49:51.144Z - Hash: b46c6ae3eb36be75 - Time: 89752ms

> cordova build android
Discovered plugin "cordova-plugin-moodleapp". Adding it to the project
Building cordova-plugin-moodleapp
cordova-plugin-moodleapp built

Failed to restore plugin "cordova-plugin-moodleapp". You might need to try adding it again.
Error: CordovaError: Failed to fetch plugin file:cordova-plugin-moodleapp via registry.
CordovaError: Error: Cannot find module 'cordova-plugin-moodleapp/package.json' from '/home/asad/projects/moodleapp'

cordova-plugin-androidx-adapter: Processed 111 source files in 2623ms
...Gradle tasks running...

BUILD SUCCESSFUL in 5m 28s
56 actionable tasks: 56 executed
Built the following apk(s):
    /home/asad/projects/moodleapp/platforms/android/app/build/outputs/apk/debug/app-debug.apk

[ERROR] An error occurred while running subprocess cordova.
        cordova build android exited with exit code 1.
```

**Exit code: 1. But an APK exists at 23 MB.**

What actually happened, step by step:
1. Angular bundle compiled successfully (4.98 MB, ~90 seconds)
2. Cordova started, tried to "restore" `cordova-plugin-moodleapp` into the Android platform
3. Plugin restore FAILED — Cordova's internal module resolution couldn't find `cordova-plugin-moodleapp/package.json` even though the symlink at `node_modules/cordova-plugin-moodleapp` was valid and `node -e "require('cordova-plugin-moodleapp/package.json')"` worked fine
4. Despite the restore failure, Cordova continued and ran the Gradle build
5. Gradle downloaded its own copy of Gradle 8.14.2 via a wrapper it generated (the manual Gradle install in `~/gradle-dist` wasn't even needed — Cordova handles this itself on first run)
6. Gradle build succeeded: `BUILD SUCCESSFUL in 5m 28s`
7. APK produced: `app-debug.apk` (23 MB)
8. But Cordova's exit code was 1 because the plugin restore at step 3 failed

**Why is `cordova-plugin-moodleapp` not in `platforms/android/android.json`?**

Checked:
```bash
python3 -c "import json; d=json.load(open('platforms/android/android.json')); print(list(d.get('installed_plugins',{}).keys()))"
```
Output showed 27 plugins — but `cordova-plugin-moodleapp` was NOT in the list. It's in `package.json` under `cordova.plugins` but was never added to the Android platform registry. Cordova detected it was "missing" and tried to add it at build time, but its module resolution for `file:` protocol plugins doesn't follow symlinks the way Node.js does.

**Also noticed:** The command `ionic cordova run android --prod` uses `run`, not `build`. The `run` command is meant to deploy to a connected Android device after building. In WSL2, there is no connected device. Even if the build worked perfectly, `run` would fail at the deploy step.

---

### Attempt 7 — Test with `--no-restore` flag

**Command tested:**
```bash
node_modules/.bin/cordova build android --prod --no-restore
```

`--no-restore` tells Cordova to skip the plugin restoration step entirely. Since all plugin native code is already compiled into `platforms/android/`, this is safe.

**Result:**
```
BUILD SUCCESSFUL in 28s
56 actionable tasks: 56 executed
Built the following apk(s):
    /home/asad/projects/moodleapp/platforms/android/app/build/outputs/apk/debug/app-debug.apk

Exit: 0
```

Exit code 0. 28 seconds (vs 5+ minutes first time — Gradle daemon was now warm).

**Then tested with Ionic CLI wrapper:**
```bash
node_modules/.bin/ionic cordova build android --prod -- --no-restore
```

The `--` separator passes `--no-restore` through to Cordova. Also exit code 0. ✅

---

### Final fix — update `prod:android` script permanently

**File changed:** `package.json`

**Before:**
```json
"prod:android": "npm run prod --prefix cordova-plugin-moodleapp && cross-env NODE_ENV=production ionic cordova run android --prod"
```

**After:**
```json
"prod:android": "npm run prod --prefix cordova-plugin-moodleapp && cross-env NODE_ENV=production ionic cordova build android --prod -- --no-restore"
```

Two changes:
1. `run` → `build` — don't try to deploy to a device, just produce the APK
2. `-- --no-restore` — skip the broken Cordova plugin restoration step

---

### Final working build sequence

```bash
# Step 1 — sync dependencies (only needed when package.json changes)
npm install cross-env@10.1.0 --save-dev --ignore-scripts
cd cordova-plugin-moodleapp && npm install && cd ..

# Step 2 — preprocess
gulp

# Step 3 — build APK
npm run prod:android
```

**Total time (with warm Gradle daemon):** ~2 minutes (Angular bundle) + 28 seconds (Gradle) = ~2.5 min  
**First ever build:** ~2 minutes (Angular) + 5–6 minutes (Gradle downloads 130 MB) = ~8 min  
**APK output:** `platforms/android/app/build/outputs/apk/debug/app-debug.apk` (23 MB)  
**APK type:** Debug-signed (usable for sideloading and testing; not for Play Store)  
**Tested:** Installed and ran successfully on a real Android device ✅

---

### Summary of all errors hit and their fixes

| # | Error message | What it meant | Fix |
|---|---|---|---|
| 1 | `sh: 1: cross-env: not found` (root) | `cross-env` binary missing from `node_modules/.bin/` despite being in `package.json` | `npm install cross-env@10.1.0 --save-dev --ignore-scripts` |
| 2 | `sh: 1: cross-env: not found` (cordova-plugin-moodleapp) | Same issue but in the subdirectory's node_modules | `cd cordova-plugin-moodleapp && npm install` |
| 3 | `cross-env` kept disappearing between runs | Postinstall hook (`cd cordova-plugin-moodleapp && npm install`) interfered with npm hoisting | Always use `--ignore-scripts` when installing cross-env |
| 4 | `Could not find an installed version of Gradle` | No Gradle binary on PATH, no `gradlew` wrapper in platform | Cordova auto-generates and downloads Gradle on first run — just needs internet. Manually downloading to `~/gradle-dist` also works. |
| 5 | `Cannot find module 'cordova-plugin-moodleapp/package.json'` → exit code 1 | Cordova's plugin restore uses its own module resolver (doesn't follow symlinks) — plugin wasn't in `android.json` | `--no-restore` flag on cordova build |
| 6 | `ionic cordova run` trying to deploy to device | `run` = build + deploy. WSL2 has no connected device | Changed script to `ionic cordova build` (build only, no deploy) |

### Key facts to remember for next time

- **Gradle is auto-downloaded** by Cordova's wrapper on first build (~130 MB, internet required). You do NOT need to install Gradle separately.
- **`cross-env` is fragile** in this project's npm setup. Always install with `--ignore-scripts`.
- **The APK is debug-signed** — fine for sideloading, not for Play Store release.
- **`--no-restore`** is now baked into the `prod:android` script in `package.json` — you don't need to pass it manually.
- **Build from WSL filesystem** (`~/projects/`), never from `/mnt/c/` or `/mnt/f/` — I/O over the /mnt bridge makes builds extremely slow.

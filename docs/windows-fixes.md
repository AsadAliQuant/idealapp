# Windows Fixes & Workarounds

Permanent fixes applied to make this project work on Windows. Future sessions don't need to redo these — they're already in the codebase.

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

## Future Sessions — Known Issues to Watch

- **Node 24 vs required 22**: Works for browser dev, but if builds break unexpectedly, downgrade to Node 22 via nvm-windows.
- **SSL cert warning**: `https://localhost:8100` uses a self-signed cert — just click through in the browser.
- **First build slow**: Angular's first compile takes 1-3 min. Subsequent incremental builds are fast.

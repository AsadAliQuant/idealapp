# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

The official Moodle Mobile App — an Angular + Ionic + Cordova hybrid app for iOS and Android (v5.2.0). It connects to any Moodle LMS via Web Services.

- **Source repo (Asad's fork)**: https://github.com/AsadAliQuant/idealapp.git — clone from here, not moodlehq
- Developer docs: https://moodledev.io/general/app
- Bug tracker: https://moodle.atlassian.net/browse/MOBILE

## Session Docs (start here for context)

- [docs/dev-environment.md](docs/dev-environment.md) — Windows setup status, installed tool versions, current state, how to run
- [docs/tech-stack.md](docs/tech-stack.md) — All dependency versions (Angular, Ionic, Cordova, plugins, testing tools) for debugging
- [docs/windows-fixes.md](docs/windows-fixes.md) — Permanent Windows fixes already applied (bash fix, session logs)
- [docs/android-build-setup.md](docs/android-build-setup.md) — JDK 17 + Android Studio setup, env vars, APK build commands
- [docs/browser-cors-fix.md](docs/browser-cors-fix.md) — Fix for `serverconnectionajax` / status 0 errors when using `npm start` in browser
- [docs/wsl2-android-setup.md](docs/wsl2-android-setup.md) — WSL2 Ubuntu setup for APK builds (Node 20, JDK 17, Android SDK CLI, Claude Code install)

## Commands

```bash
# Development
npm start                  # Serve in browser (Ionic dev server)
gulp                       # Preprocess: lang files, env config, icons (run before first build)
gulp watch                 # Watch lang/env files during development

# Build
npm run build              # Development build
npm run build:prod         # Production build

# Mobile
npm run dev:android        # Android emulator with live reload
npm run dev:ios            # iOS simulator with live reload
npm run prod:android       # Production Android APK
npm run prod:ios           # Production iOS build

# Test
npm test                   # Run all Jest tests (runs gulp first)
npm run test:watch         # Watch mode
npm run test:ci            # Sequential CI runner
npm run test:coverage      # With coverage report

# Lint
npm run lint               # ESLint
```

To run a single test file:
```bash
npx jest src/path/to/file.test.ts
```

## Architecture

### Two-tier module system

**`src/core/`** — Non-optional app infrastructure: authentication, routing, site management, shared services, shared components, directives, pipes, and core feature modules (login, courses, mainmenu, settings, etc.).

**`src/addons/`** — Optional feature addons mirroring Moodle's plugin system. Each addon maps to a Moodle plugin type:
- `addons/mod/` — Activity modules (assign, forum, quiz, lesson, scorm, h5pactivity, etc.)
- `addons/block/` — Block plugins (myoverview, recentlyaccessedcourses, etc.)
- `addons/messages/`, `addons/calendar/`, `addons/notifications/`, etc.
- `addons/qtype/`, `addons/qbehaviour/` — Question types and behaviours
- `addons/report/`, `addons/filter/` — Reports and filters

### Within each feature/addon

Each is self-contained with its own:
- `services/` — Angular services (API calls, business logic)
- `components/` — Angular components + templates
- `pages/` — Routed page components
- `lang/` — Translation strings (JSON, merged by Gulp into `src/assets/lang/`)
- `*.module.ts` — Angular module with lazy routing

### Key services to know

- `CoreSitesProvider` — Manages multiple Moodle site connections
- `CoreWSProvider` — Web Services HTTP layer
- `CoreFilepoolProvider` — Offline file caching
- `CoreNavigatorService` — App-wide navigation
- `CoreSyncBaseProvider` — Base for offline sync

### Gulp preprocessing (required before builds)

`gulp lang` scans all `lang/` folders across `src/core/features/` and `src/addons/` and merges them into `src/assets/lang/*.json`. Language strings don't exist at runtime until this runs. `npm test` runs it automatically; `npm start`/build does not — run `gulp` manually first.

### Configuration

`moodle.config.json` — Runtime app config (app ID, supported languages, cache frequencies, custom URL schemes, icon prefixes). Processed by `gulp env` into a typed environment file.

To override for local dev: create `moodle.config.local.json` (gitignored).

### Testing

- Framework: Jest + jest-preset-angular
- Test files: `**/*.test.ts` (co-located with source)
- Test utilities/mocks: `src/testing/`
- Module path aliases from `tsconfig.paths` are mapped in `jest.config.js`

## WSL2 Android Build (Preferred for APK builds)

Build APKs inside WSL2 Ubuntu — not Windows PowerShell. Faster node_modules, no path issues.

### One-time setup (run in Ubuntu WSL2)

```bash
# 1. Copy project into WSL filesystem (never build from /mnt/)
cp -r /mnt/f/Asad/Freelancing/Samantha\ App/moodleapp ~/projects/moodleapp
cd ~/projects/moodleapp

# 2. Install Node 20 + JDK 17
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs git openjdk-17-jdk unzip wget

# 3. Install Android SDK CLI tools
mkdir -p ~/android-sdk/cmdline-tools && cd ~/android-sdk/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
unzip commandlinetools-linux-*.zip && mv cmdline-tools latest

# 4. Set env vars (add to ~/.bashrc)
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export ANDROID_HOME=$HOME/android-sdk
export PATH=$PATH:$JAVA_HOME/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools

# 5. Accept licenses + install SDK
yes | sdkmanager --licenses
sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"

# 6. Install Claude Code in WSL2
npm install -g @anthropic-ai/claude-code
```

### Verify before building

```bash
java -version   # must show 17.x
adb --version
node -v         # 20.x
```

### Build commands (same as Windows)

```bash
npm install
gulp
npm run prod:android   # production APK
npm run dev:android    # debug APK with live reload
```

> Claude Code must be installed **inside WSL2** (not Windows) to run these builds. A Windows symlink does not work.

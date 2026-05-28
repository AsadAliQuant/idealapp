# WSL2 Android Build Setup for Moodle App

## Why WSL2 instead of Windows Git Bash

Ionic/Cordova/Android toolchain is built for Linux/macOS. On Windows you get:
- Partial npm installs (missing files like `@ionic/cli/lib/updates.js`)
- Cordova plugin resolution failures (`es6-promise-plugin` etc.)
- Bash script errors, CRLF line ending issues, long path failures

WSL2 runs a real Linux kernel — these errors don't happen.

---

## Step 0: Get your code into WSL2

You have two options. **Option B is recommended.**

### Option A: Access Windows files directly (quick but slow I/O)
From inside WSL2, your Windows F: drive is at `/mnt/f/`.
```bash
cd "/mnt/f/Asad/Freelancing/Samantha App/moodleapp"
```
Works but npm install and builds are slow over the /mnt bridge.

### Option B: Push to your own GitHub repo, clone inside WSL2 (recommended)

Your current remote points to the official moodlehq repo. You need your own repo.

**On Windows (Git Bash), do this once:**
```bash
cd "F:/Asad/Freelancing/Samantha App/moodleapp"

# Create a new repo on GitHub first (github.com → New repository → e.g. samantha-moodleapp)
# Then change the remote to your repo:
git remote set-url origin https://github.com/YOUR_USERNAME/samantha-moodleapp.git

# Commit your local changes first
git add config.xml moodle.config.json package.json package-lock.json
git add src/theme/globals.custom.scss src/theme/theme.custom.scss
git add CLAUDE.md docs/
git commit -m "Samantha app customizations"
git push -u origin main
```

**Then inside WSL2:**
```bash
cd ~
mkdir -p projects
cd projects
git clone https://github.com/YOUR_USERNAME/samantha-moodleapp.git
cd samantha-moodleapp
```

---

## Step 1: Install Node.js via nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

nvm install 20
nvm use 20
nvm alias default 20

node --version   # should show v20.x.x
npm --version
```

---

## Step 2: Install JDK 17

Cordova Android requires JDK 17 exactly (not 21, not 11).

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk

java -version   # should show openjdk 17
```

---

## Step 3: Install Android SDK (no Android Studio needed)

```bash
# Create SDK directory
mkdir -p ~/Android/Sdk/cmdline-tools

# Download command line tools
cd ~/Android/Sdk/cmdline-tools
wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
unzip commandlinetools-linux-11076708_latest.zip
mv cmdline-tools latest
rm commandlinetools-linux-11076708_latest.zip

# Install platform tools and build tools
~/Android/Sdk/cmdline-tools/latest/bin/sdkmanager --sdk_root=$HOME/Android/Sdk \
  "platform-tools" \
  "build-tools;36.0.0" \
  "platforms;android-36"

# Accept licenses
yes | ~/Android/Sdk/cmdline-tools/latest/bin/sdkmanager --sdk_root=$HOME/Android/Sdk --licenses
```

---

## Step 4: Set environment variables

Add these to the end of `~/.bashrc`:

```bash
cat >> ~/.bashrc << 'EOF'

# Java
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64

# Android SDK
export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$ANDROID_HOME
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/build-tools/36.0.0
EOF

source ~/.bashrc
```

Verify:
```bash
java -version
adb --version
sdkmanager --version
```

---

## Step 5: Install project dependencies

```bash
cd ~/projects/samantha-moodleapp   # or your project path

# Install global tools
npm install -g gulp-cli

# Install project dependencies
npm install

# Fix: cross-env may not land in node_modules/.bin after npm install due to
# postinstall ordering. Install it explicitly with --ignore-scripts to bypass that:
npm install cross-env@10.1.0 --save-dev --ignore-scripts

# Also ensure cordova-plugin-moodleapp has its own deps:
cd cordova-plugin-moodleapp && npm install && cd ..
```

> **Why the cross-env workaround?** The postinstall runs `cd cordova-plugin-moodleapp && npm install` after the root install. This sometimes causes npm's hoisting to leave `cross-env` out of the root `node_modules/.bin/`. The `--ignore-scripts` install adds it without re-triggering that race. If you ever see `sh: cross-env: not found` during a build, re-run the two lines above.

---

## Step 6: Connect Android device or emulator

### Physical device (easiest):
1. Enable Developer Options on Android phone
2. Enable USB Debugging
3. Connect via USB to your Windows PC
4. From WSL2, USB devices need usbipd to be forwarded:

```bash
# On Windows PowerShell (as admin) — one-time setup:
winget install usbipd

# Each time you connect a device, in PowerShell:
usbipd list                    # find your device's BUSID
usbipd bind --busid <BUSID>    # first time only
usbipd attach --wsl --busid <BUSID>

# Back in WSL2:
adb devices   # should show your device
```

### Emulator (alternative):
Run Android Studio on Windows side, start an emulator there, then from WSL2:
```bash
export ADB_SERVER_SOCKET=tcp:127.0.0.1:5037
adb connect 127.0.0.1:5555
adb devices
```

---

## Step 7: Build and run

```bash
cd ~/projects/samantha-moodleapp

# First time only — preprocess lang files and env config
gulp

# Build APK (production Angular bundle, debug APK signing)
npm run prod:android
```

**First build only:** Cordova auto-downloads Gradle 8.14.2 (~130 MB) from `services.gradle.org` via its wrapper. Needs internet. Takes ~5–6 minutes. Subsequent builds use the cached daemon and finish in ~20–30 seconds.

**APK output:** `platforms/android/app/build/outputs/apk/debug/app-debug.apk`

> **Note on APK type:** `--prod` sets Angular to production mode (minified bundle) but does **not** produce a signed release APK. The output is always a debug APK unless you add `--release` with a keystore configured. For sideloading and testing, the debug APK is fine.

**What the `prod:android` script does** (for reference):
1. Builds `cordova-plugin-moodleapp` native code
2. Runs `gulp` (lang + env preprocessing)
3. Runs `ng build` (Angular production bundle into `www/`)
4. Runs `cordova build android --no-restore` (compiles Java/Kotlin + packages APK)

---

## Step 8: Claude Code in WSL2

### Install Claude Code
```bash
npm install -g @anthropic-ai/claude-code
```

### Share your existing memories from Windows
Your Claude memories live at `C:\Users\Asad\.claude\` on Windows.
From WSL2 that path is `/mnt/c/Users/Asad/.claude/`.

To share them with WSL2 Claude Code (so it has all your context):
```bash
ln -s /mnt/c/Users/Asad/.claude ~/.claude
```

This symlinks your Windows Claude config into WSL2 — same memories, same project history, no duplication.

### Open the project in Claude Code
```bash
cd ~/projects/samantha-moodleapp
claude
```

The `CLAUDE.md` in the repo root will give Claude Code full project context automatically.

---

## Checklist

- [ ] WSL2 Ubuntu installed (`wsl --install -d Ubuntu` in Windows PowerShell as admin)
- [ ] Code pushed to your own GitHub repo
- [ ] Code cloned inside WSL2 `~/projects/`
- [ ] Node.js 20 via nvm
- [ ] JDK 17
- [ ] Android SDK + build-tools + platform
- [ ] Environment variables in `~/.bashrc`
- [ ] `npm install` completed without errors
- [ ] `cross-env` fix applied (Step 5 — `npm install cross-env@10.1.0 --save-dev --ignore-scripts`)
- [ ] `adb devices` shows your device (or skip if only building APK, not deploying)
- [ ] Claude Code installed + `~/.claude` symlinked from Windows
- [ ] First `npm run prod:android` run completes (downloads Gradle — needs internet)

## Known issues fixed (already applied to this repo)

| Issue | Symptom | Fix applied |
|---|---|---|
| `cross-env: not found` | Build fails immediately after plugin build | `npm install cross-env --save-dev --ignore-scripts` in root and cordova-plugin-moodleapp |
| Cordova plugin restore fails | `Cannot find module 'cordova-plugin-moodleapp/package.json'` — exit code 1 even though APK built | `prod:android` script changed to `ionic cordova build android -- --no-restore` |
| No `gradlew` in platform | First-time Gradle missing | Cordova wrapper auto-downloads Gradle 8.14.2 on first build (internet required) |

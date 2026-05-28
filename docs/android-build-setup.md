# Android Build Setup (Windows)

## 1. JDK 17
Download: https://adoptium.net/temurin/releases/?version=17

Installed path (this machine):
```
C:\Program Files\Eclipse Adoptium\jdk-17.0.19.10-hotspot
```

## 2. Android Studio
Download: https://developer.android.com/studio

During install, check:
- Android SDK
- Android SDK Platform (API 33 or 34)

After install, open **SDK Manager** (Tools → SDK Manager) and install:
- Android SDK Platform 33 or 34
- Android SDK Platform-Tools (this installs `adb.exe`)

## 3. Set Environment Variables (User-level, no admin needed)

Run in PowerShell (new terminal after setting):

```powershell
[System.Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Eclipse Adoptium\jdk-17.0.19.10-hotspot", "User")
[System.Environment]::SetEnvironmentVariable("ANDROID_HOME", "C:\Users\Asad\AppData\Local\Android\Sdk", "User")

$path = [System.Environment]::GetEnvironmentVariable("Path", "User")
$additions = ";C:\Program Files\Eclipse Adoptium\jdk-17.0.19.10-hotspot\bin;C:\Users\Asad\AppData\Local\Android\Sdk\platform-tools;C:\Users\Asad\AppData\Local\Android\Sdk\tools;C:\Users\Asad\AppData\Local\Android\Sdk\tools\bin"
[System.Environment]::SetEnvironmentVariable("Path", $path + $additions, "User")
```

> Status (2026-05-28): JAVA_HOME, ANDROID_HOME, and PATH all set. JDK bin added to PATH.

## 4. Verify (open a new terminal first)

```powershell
java -version   # must show 17.x — OpenJDK Temurin
adb --version   # Android Debug Bridge (needs platform-tools installed)
```

Expected java output:
```
openjdk version "17.0.19" 2026-04-21
OpenJDK Runtime Environment Temurin-17.0.19+10
```

## 5. Build

```powershell
cd "F:\Asad\Freelancing\Samantha App\moodleapp"
npm run dev:android    # debug APK with live reload (for device/emulator testing)
npm run prod:android   # production APK
```

> After setting env vars, always open a **new** terminal window before running any commands.

## Troubleshooting

- `java` not recognized → confirm `JAVA_HOME\bin` is in PATH (step 3), open new terminal
- `adb` not recognized → install **Android SDK Platform-Tools** via Android Studio SDK Manager
- Build fails with SDK error → confirm API 33/34 is installed in SDK Manager

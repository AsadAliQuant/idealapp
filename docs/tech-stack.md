# Tech Stack — Moodle Mobile App v5.2.0

Full dependency reference for debugging and version-pinning decisions.

---

## App Identity

| Field | Value |
|-------|-------|
| App version | 5.2.0 |
| Package name | moodlemobile |
| License | Apache-2.0 |
| Cordova platforms | android, ios |

---

## Core Framework Versions

| Technology | Version | Role |
|-----------|---------|------|
| Angular | 20.3.18 | UI framework |
| Angular CLI | 20.3.22 | Build tooling |
| Ionic Angular | 8.8.1 | Mobile UI components |
| Ionic CLI | 7.2.1 (devDep) | CLI wrapper |
| Cordova | 13.0.0 | Native bridge |
| cordova-android | 14.0.1 | Android platform |
| cordova-ios | 7.1.1 | iOS platform |
| TypeScript | 5.9.3 | Language |
| RxJS | 7.8.2 | Reactive streams |
| zone.js | 0.15.1 | Angular change detection |

---

## Build & Dev Tools

| Tool | Version | Role |
|------|---------|------|
| Gulp | 5.0.1 | Lang/env preprocessing |
| ESLint | 9.39.4 | Linting |
| angular-eslint | 20.7.0 | Angular-specific rules |
| typescript-eslint | 8.56.1 | TS-specific rules |
| cross-env | 10.1.0 | Cross-platform env vars |
| concurrently | 9.2.1 | Parallel npm scripts |
| patch-package | 8.0.1 | Post-install patches |

---

## Testing

| Tool | Version | Role |
|------|---------|------|
| Jest | 30.3.0 | Test runner |
| jest-preset-angular | 16.1.2 | Angular Jest integration |
| jest-environment-jsdom | 30.3.0 | DOM simulation |
| ts-jest | 29.4.9 | TS transform for Jest |
| @faker-js/faker | 9.9.0 | Test data generation |

---

## Key Runtime Dependencies

| Package | Version | Role |
|---------|---------|------|
| @ngx-translate/core | 17.0.0 | i18n translations |
| chart.js | 4.5.1 | Charts |
| dayjs | 1.11.20 | Date handling |
| jszip | 3.10.1 | ZIP file handling |
| mathjax | 3.2.2 | Math rendering |
| swiper | 12.1.3 | Carousels/sliders |
| video.js | 8.23.4 | Video playback |
| ngx-image-cropper | 9.1.6 | Image cropping |
| wa-sqlite | 1.0.0 | WebAssembly SQLite |
| ionicons | 8.0.13 | Icon set |
| core-js | 3.49.0 | JS polyfills |
| ogv | 1.9.0 | Ogg/WebM video (JS) |
| mp3-mediarecorder | 4.0.5 | Audio recording |
| ts-md5 | 2.0.1 | MD5 hashing |

---

## Cordova Plugins (Moodle-forked)

All `@moodlehq/cordova-plugin-*` are Moodle's maintained forks:

| Plugin | Version |
|--------|---------|
| cordova-plugin-advanced-http | 3.3.1-moodle.1 |
| cordova-plugin-camera | 7.0.0-moodle.1 |
| cordova-plugin-file-opener | 4.0.0-moodle.1 |
| cordova-plugin-file-transfer | 2.0.0-moodle.3 |
| cordova-plugin-inappbrowser | 6.0.0-moodle.1 |
| cordova-plugin-intent | 2.2.0-moodle.3 |
| cordova-plugin-ionic-keyboard | 2.2.0-moodle.3 |
| cordova-plugin-ionic-webview | 5.0.0-moodle.5 |
| cordova-plugin-media-capture | 5.0.0-moodle.1 |
| cordova-plugin-qrscanner | 3.0.1-moodle.6 |
| cordova-plugin-statusbar | 4.0.0-moodle.5 |
| cordova-plugin-zip | 3.1.0-moodle.1 |
| phonegap-plugin-push | 4.0.0-moodle.13 |

---

## Awesome Cordova Plugins (wrappers)

All `@awesome-cordova-plugins/*` at version **9.1.2** — Angular service wrappers over the native Cordova plugins above.

Includes: badge, camera, clipboard, core, device, file, file-opener, http, in-app-browser, ionic-webview, keyboard, local-notifications, media-capture, network, push, splash-screen, sqlite, status-bar, web-intent.

---

## Node Engine Requirement

```json
"engines": { "node": ">=v22.17 <23" }
```

Only **Node 22 LTS** is officially supported. See `docs/dev-environment.md` for current status and fix.

---

## Angular Build Memory

Builds require extra heap — enforced in all npm scripts:
```
NODE_OPTIONS=--max-old-space-size=8192
```
If you hit `JavaScript heap out of memory` errors, this is already handled by the npm scripts. Don't run `ng build` directly — always go through `npm run build`.

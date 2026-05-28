# Browser CORS Fix

When running `npm start`, the app is at `http://localhost:8100`. Connecting to a Moodle server on a different origin (e.g. `http://localhost/moodle`) triggers CORS errors in normal Chrome:

```
Error code: serverconnectionajax
Error connecting to the server: [Response status code: 0] Unknown error
```

Status 0 = browser blocked the request before it reached the server.

## Option 1 — Dev Chrome with flags (confirmed working ✓)

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --disable-web-security --user-data-dir="C:\chrome-dev-session" --allow-running-insecure-content
```

Then open `http://localhost:8100` in that Chrome window. This is the standard dev workflow for Moodle Mobile in browser.

## Option 2 — CORS headers on Moodle server (XAMPP)

Add to `.htaccess` in your Moodle root or Apache config:

```apache
Header set Access-Control-Allow-Origin "http://localhost:8100"
Header set Access-Control-Allow-Headers "Content-Type, Authorization"
Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"
```

## Option 3 — Ionic proxy

Add to `ionic.config.json`:

```json
"proxies": [{ "path": "/moodle", "proxyUrl": "http://localhost/moodle" }]
```

Then enter `/moodle` as the site URL in the app login screen.

# Research — Headless Chrome on Windows: what works for extension-style UI testing

**Tested:** 2026-10/11 · **Scope:** Chrome 137+ on Windows 11, PowerShell, LinkClipper dashboard + dev harness.

## The working incantation

```powershell
& "C:\Program Files\Google\Chrome\Application\chrome.exe" `
  --headless=new --disable-gpu --hide-scrollbars `
  --window-size=1280,800 `
  --screenshot="OUT.png" `
  --virtual-time-budget=9000 `
  "file:///C:/path/dev.html?state=reader"
```

- `--headless=new` (not `--headless` — old headless doesn't render everything we use).
- `--virtual-time-budget` advances timers/network deterministically; 9000–12000 ms is enough for our seeds + thumbnail loads.
- `--disable-gpu` avoids Brave/Chromium GPU-compositor artifacts we don't want to debug in screenshots.

## Traps we hit (do not rediscover these)

1. **Chrome 137+ blocks `--load-extension`.** There is no supported way to render real extension pages in headless. That's why `dashboard/dev.html` exists — it stubs `chrome.*` APIs and loads the *real* app source over `file://`. Test against the harness, not the extension.
2. **Sequential runs in a loop share the profile dir** and the second run fails or flaps. Either give each launch a distinct `--user-data-dir` (temp), or launch one-shot renders as separate process invocations with `Start-Process -Wait`. First solo invocation always works.
3. **`--screenshot` writes to stdout** (`NNN bytes written...`); stderr Chromium noise (registry loader errors, OS-integration warnings) is harmless and non-ANSI-garbled — do not treat as failure. Judge by the PNG size and by *looking*.
4. **`file://` + browser's own fetch of remote images works** (picsum thumbnails render), but only inside the virtual time budget — give network-dependent shots the larger budget.
5. **Reported PNG sizes are bytes written to disk**, never trust success by exit code ($LASTEXITCODE is 0 even on some aborts) — verify artifacts exist and their sizes changed.

## What this can't test

- Real `chrome.storage.local`/`chrome.permissions` behavior (harness stubs lie).
- Service worker lifecycle (`background.js`).
- Anything behind real site CSP/anti-automation.
Those need loaded-extension manual passes or Chrome automation with a real profile (outside store-safe headless).

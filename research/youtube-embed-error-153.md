# Research — YouTube embed rejection (`embedder.identity.missing.referrer`, Error 153)

**Tested:** 2026-10 · **Scope:** Chromium 137+ on Windows, extension dashboard page (`chrome-extension://…/dashboard/dashboard.html`), also popup and dev harness (`file://`).
**Conclusion:** YouTube enforces an embedder-identity check on iframe embeds that extension-origin pages cannot satisfy. This is not fixable from our side in-page. Not belief — matrix below.

## What was tried and rejected

All variants render YouTube's own YT_ErrorPage with `embedder.identity.missing.referrer` (Error 153), or a spinner that never resolves:

| Variant tried | Result |
|---|---|
| `https://www.youtube.com/embed/<id>` plain | ❌ 153 |
| `https://www.youtube.com/embed/<id>?origin=<chrome-extension origin>` | ❌ 153 |
| `?widget_referrer=` with extension origin | ❌ 153 |
| `https://www.youtube-nocookie.com/embed/<id>` | ❌ 153 |
| `watch_popup` iframed instead of embedded | ❌ 153 |
| plain embed with `referrerPolicy` attribute changed (`no-referrer`, `origin`, `unsafe-url`) | ❌ 153 |
| embed inside dashboard page as real window (top-level `chrome-extension` document, iframe within) | ❌ 153 |

## What works

- **`https://www.youtube.com/watch_popup?v=<id>`** opened as a top-level `window.open()` popup (not an iframe) — YouTube's minimal player chrome, plays normally.
- **`watch_popup?v=<id>&emb.short=1`** additionally yields portrait/scale-correct rendering suitable for Shorts (9:16 content).

## Why we believe it is unfixable in-page

The `embedder.identity.missing.referrer` message is consistent for every iframe variant regardless of referrer headers we set: the check reads the embedder origin context in ways a Chrome-extension page cannot forge (extension origins are reviewed differently by YouTube; per Chromium, `document.referrer` handling across extension pages historically transmissions as empty for cross-origin embed pulls). Given every URL-shape and attribute-level escape failed identically, the conclusion is vendor-enforced, not configuration.

## Third-party front-ends — abandoned

Probed 2026-10: Invidious and Piped public instances. yewtu.be serves a "Checking your browser" challenge; other instances are down or rate-limited into uselessness. Depending on a volunteer-driven proxy fleet is also a supply-chain liability for a store-listed extension. **Decision: never ship third-party YouTube players in LinkClipper.** (ADR-0004)

## Recheck cadence

Semi-annual quick probe (5 min): one plain embed + one nocookie from the dev harness. If either plays, reopen ADR-0004; the code change is one branch in `embedInfo`.

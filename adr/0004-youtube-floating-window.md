# ADR-0004 — YouTube is played in a floating window, not embedded

**Status:** accepted (2026-11) · **Supersedes:** —

## Context

Pinterest-style in-dashboard playback was verified against YouTube and failed **every** embed variant from extension origins (dashboard page, popup, dev harness at differing origins): `embedder.identity.missing.referrer` (Error 153). Third-party front-ends (Invidious/Piped instances) were probed in 2026-10 and are uniformly challenged or dead (yewtu.be behind a challenge wall; others down/amber).

Vimeo, Instagram, TikTok embed fine inline — this problem is YouTube-specific and is enforced by YouTube's embedder-identity check, which reads referrer state that extension pages can't supply in a way YouTube accepts.

Full investigation: `research/youtube-embed-error-153.md`.

## Decision

- Vimeo / Instagram / TikTok: inline `<iframe>` embeds in the dashboard (works today, keep).
- YouTube: on Play, open a small always-on-top-style **floating `watch_popup` window** docked right; Shorts render portrait via `emb.short`. The dashboard shows why (the button label "Play in floating window" educates the limitation).
- No YouTube third-party viewer/scraper code will be added; no referrer spoofing of any kind — policy + fragility.

## Consequences

- The floating window loses the dashboard's glass context (it's a real Chromium window). Accepted as vendor-inevitable UX loss, called out in docs so users know it's not a bug.
- If YouTube ever relaxes embedder identity, the change for us is one `embedInfo` branch and one function — cheap to revisit every few months.
- Shorts portrait sizing depends on `emb.short` continuing to exist; if it breaks, fallback is the same floating window with standard sizing.

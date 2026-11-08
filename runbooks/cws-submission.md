# Runbook — Chrome Web Store submission & future updates

**Owner's tasks (browser, signed into the publisher account).** Everything code-side is in the repo; this runbook is the manual layer.

## First submission (v0.3.0)

1. Zip: use the CI artifact, the tagged release asset `LinkClipper-0.3.0.zip` ([releases](https://github.com/linkclipper/linkclipper/releases)), or run `tools/pack.ps1` locally — never a hand-rolled zip (pack.ps1 excludes dev/tools/`LinkClipper-*.zip` leftovers).
2. [Developer dashboard](https://chrome.google.com/webstore/devconsole) → **+ New item** → upload the zip.
3. Meta:
   - Name: **LinkClipper**
   - Description: manifest's description (short + no keyword stuffing)
   - Category: Productivity → Tools
   - Language: English
4. Store listing: the 3 screenshots from `tools/store-assets/` (`webstore-1-grid.png` 635 KB, `webstore-2-reader.png` 422 KB, `webstore-3-edit.png` 412 KB). 1280×800, correct aspect. Copy matches what's on screen — screenshots come from the dev harness with seeded content, and their copy must not claim capabilities the real product lacks.
5. Privacy tab:
   - Single purpose: *"Save links you're browsing into tagged boards, replayable in place."*
   - Data usage matrix: declare **none** sold, **none** for unrelated purposes. Stored locally: page URLs, titles, user-entered notes/tags/thumbnails of saved pages. Transmitted: only if the user configures their own Supabase sync (then: same records to the user's own project endpoint).
   - Compliance status: user data is not used for any future purpose; not sold; not used for personalized advertising.
   - Privacy policy URL: `https://linkclipper.github.io/privacy.html` (live, site repo `privacy.html`).
6. Permission justifications (tab asks per permission) — paste from the table below.
7. Distribution: Public. Pricing: Free.
8. Submit; expect verification of the privacy URL. Deep review is *not* expected because there is no install-time broad host permission — per-site grants happen at runtime on user click. If the review stalls, check that the uploaded manifest matches the repo (version field, permissions arrays identical).

## Permission justification cheat sheet

| Permission | Paste this |
|---|---|
| `storage` | Saves boards, items, and settings locally on the user's device. No servers involved unless the user enables their own sync project. |
| `activeTab` | Saving the link/page the user is currently viewing, from the toolbar popup. |
| Declared host permissions (YouTube, Instagram, X, Reddit, TikTok, Vimeo) | Show the in-page save button and video preview on those specific well-known sites. |
| `optional_host_permissions` | The user may grant a specific site, with a click, so the extension can fetch that article for the offline reading view. Nothing installs with broad access. |

## Update cadence rules

- Version bump **must** accompany every resubmission (`manifest.json` version; changelog entry).
- Any new permission = explicit changelog line + updated justification + note in PR checklist (CONTRIBUTING).
- If review rejects a specific claim, fix the *claim* in the listing (not just the code) — CWS re-reviews the listing text.

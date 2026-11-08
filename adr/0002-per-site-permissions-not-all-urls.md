# ADR-0002 — Per-site permissions, never install-time all-URLs

**Status:** accepted (2026-11) · **Supersedes:** —

## Context

In-page save buttons require content scripts on social sites. The Reader feature must fetch article pages the user is reading — arbitrary hosts.

Options considered:

1. Install-time `<all_urls>` host permission → everything works immediately, but Chrome Web Store review escalates and users see an alarming "read all data on all sites" warning at install.
2. Install-time host permissions for the 6 known social sites, optional grants for the Reader.
3. `activeTab` + user-gesture injection.
4. `optional_host_permissions` + request-on-click per site.

## Decision

Option 4, with content scripts' declared `host_permissions` limited to the 6 supported social sites (YouTube, Instagram, X, Reddit, TikTok, Vimeo). Anything else — Reader fetching, previews — asks via `chrome.permissions.request()` at the moment the user clicks, one site at a time.

## Consequences

- Pros: CWS review stays shallow; install warnings stay honest; users see exactly which sites were granted (`chrome://extensions` is auditable).
- Cons: first Reader use costs a permission prompt per site (documented in the UI at the click itself, not after); every permission request must be inside a real user gesture — no speculative pre-requests, no requesting inside `system` events, or Chrome rejects the call.
- Engineering rule: **permissions are never widened by a release** without calling it out in the changelog. See `runbooks/cws-submission.md` for the reviewer-facing justifications written per permission.

# Design — The social backend (Phases B → C)

The social phase is the risk pivot of this project. This doc states what it looks like **before** any of it is built, so Phase B code doesn't paint us into a Phase C corner.

## Trust ladder (what changes when)

| Phase | Who controls data | Trust model |
|---|---|---|
| A (now) | The user, in `chrome.storage.local` | No server truth. User-owned Supabase mirror is fully optional. |
| B share-links | The user, in *their own* Supabase project | Anon visitors read published boards from the user's project. LinkClipper is a URL generator, not a host. |
| C discovery | LinkClipper-hosted Supabase project | Accounts. Public board index. Local store remains source of truth per device; server indexes published data. |

**Invariant:** the extension must keep working fully offline. The social layer is additive and per-feature opt-in. If Phase C auth ever imposes on a user, we have violated this repo's law (README: *requires an account* → redesign).

## Phase B: link-shares from a user-owned project

- `linkclipper.github.io/b/<slug>` = static page, JS reads the user's Supabase project directly with the anon key carried *in the URL fragment* (never logged server-side, cache-bypasses CDN problem, and the slug+fragment model means the page is a pure static file).
- Actually two candidate transports — decide at implementation:
  1. **URL fragment key**: `…/b/<slug>#k=<anonKey>` — works project-owned; cons: ugly, no link previews.
  2. **User registers the share URL in an `index_names` row of a tiny LinkClipper-hosted lookup** that only maps slug→project URL (no content at rest) — tradeoff: we hold a mapping table; user trust moves up a notch.
- Content shape shared: board + items, sanitized (no `readerText` unless author opts in).
- RLS draft: `linkclipper/backend/supabase/schema.sql` § share.

## Phase C: hosted discovery

- One managed Supabase project (or first a Postgres+Hobby Postgres fork). Schema extension, **not rewrite**: same `wc_*` tables + `owner_id` uuid → `profiles`.
- **No importer:** connecting an existing local library to Phase C is *always* an explicit export/import. Silent upload-on-login is banned.
- Public index table (`boardPublic`) holds denormalized published boards (slug, title, owner handle, item count, updated_at) refreshed by triggers/edge functions. Discovery queries are server-side ranked reads over an index — never raw table scans with client keys.
- Remix (`save to my board`) = fetch public item → write to *local* store → normal sync. Attribution: `remixedFromSlug` + owner handle live on the item record.
- Moderation: minimum viable = report button + blocking list + takedown SLA in privacy policy; then curated front-page. **Gate Phase C on this being written down first** (CWS policy pressure on user-generated content happens the moment discovery exists).

## Explicit non-goals

- Followers-after-we-have-content pitch: no invites/collabs until real usage data exists (ROADMAP Phase D).
- Own cloud-native stack (k8s etc.): Postgres + edge functions scale to large numbers of readers fine; revisit at 100k+ MAU.
- Building a moderation ML pipeline; human review before automation.

# Design — Sync protocol (current) and its evolution to the social backend

## Record format (v1)

Two tables, snake_case on the wire:

```
wc_items:  id (text pk) · type · title · url · canonical · thumbnail · description
           source · selection · note · board_id (fk) · tags (jsonb)
           created_at · updated_at · deleted_at
wc_boards: id (text pk) · name · color · description
           created_at · updated_at · deleted_at
```

- IDs locally-generated (WC.uid()) — no server-issued IDs ever; keeps local-first true.
- `deleted_at` tombstones make deletes/undeletes syncable; deletion = tombstone + `updatedAt` bump.
- `readerText` caches are **not** synced (bulk, private); a remote pull locally re-derives or lazy-fetches.

## Merge semantics

Per record LWW on `updatedAt`. Push-then-pull per table with `?updated_at=gt.<lastSyncAt>`; `Prefer: resolution=merge-duplicates` on upsert. `lastSyncAt` persisted after a **successful** full exchange. Client code: `lib/supabase.js` (the only thing that talks to the wire).

## Phase B addition — share links (no trust change)

`wc_boards` gains `visibility ('private'|'link')` + `share_id` in a migration; remote (read-only, anon) view:

```
GET /rest/v1/wc_boards?id=eq.<share_id>&visibility=eq.link
GET /rest/v1/wc_items?board_id=eq.<local board id>&deleted_at=is.null
```

RLS: anon reads **only** rows where `visibility='link'` matching a published share ticket (see `linkclipper/backend/supabase/schema.sql` § share). Share slugs are opaque; unauthenticated access is read-only and enforced server-side.

## Phase C change — server becomes arbiter (trust changes, shape doesn't)

1. Same tables get `owner_id` (uuid → profile). Migrating user-owned projects is *not* attempted; Phase C is a hosted opt-in project with account registration, and the migration path for existing users is **export/import JSON** (privacy-respecting; no silent upload).
2. LWW arbitration server-side on `updated_at`, same rule — the client merge code stays, only the "who wins on equality" tiebreak gets the server timestamp.
3. Everything public gets a separate surface (`share_id` → slug; `boardPublicStats`); private mirrors stay per-user and RLS'd to the owner.
4. Realtime (boards of followed users) arrives **after** read-model: it's Phase D; the record format is realtime-shaped already (records carry their own timestamps by design).

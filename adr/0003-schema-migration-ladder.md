# ADR-0003 — Schema changes go through the migration ladder, never in place

**Status:** accepted (2026-11) · **Supersedes:** —

## Context

From v0.4 the data model will evolve repeatedly (board share slugs, profiles, social fields). Data lives in users' `chrome.storage.local`; there is no server-side schema to alter safely. Any bug in an upgrade path destroys user collections silently and permanently.

## Decision

All stored shapes are versioned:

- `wc_meta.schemaVersion` records the current version (seeded 1 at install).
- `Store.MIGRATIONS` in `lib/store.js` is a ladder keyed "version upgraded **to**"; `Store.migrate()` runs every step between stored and current version at init, before any use.
- Steps are idempotent and crash-safe (a partially-applied migration should re-run cleanly).
- Neither *read* nor *write* paths may branch on "old shapes"; migrate first.
- Bumping the schema is a changelog-visible event, same bar as widening permissions.

Then-current decisions with schema impact stay recorded in the corresponding migration's commit message, and get a short note in `design/` if they affect sync.

## Consequences

- Pros: sync record format changes ride the same ladder for old clients pulling new shapes; user data upgrades in place forever.
- Cons: discipline cost — small things like adding a default field still require a step (e.g., migration N denied boards a `visibility` default; instead `undefined === private` was documented to avoid one). Mildly awkward "future-proofing" fields occasionally appear earlier than their features. Accepted.

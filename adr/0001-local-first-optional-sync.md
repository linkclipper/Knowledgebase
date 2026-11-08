# ADR-0001 — Local-first storage with optional user-owned sync

**Status:** accepted (2026-11) · **Supersedes:** —

## Context

LinkClipper clips web content people intend to revisit — the promise must survive extension updates, renderer crashes, and account changes like any bookmark manager does. Chrome extension options were: `chrome.storage.local` (large quota, local), `chrome.storage.sync` (tiny quota, UA-managed), or remote-from-day-one.

Store policy pressure makes "must have an account" a churn/review risk, and the product philosophy (see README) is that a server is a *feature*, not a *dependency*.

## Decision

- Primary store: `chrome.storage.local`, versioned keys `wc_*`, owned by `lib/store.js` alone.
- Cloud sync is **optional and user-owned**: Connectivity goes only to a Supabase project *the user created and configured* (URL + anon key pasted into settings). We hold no credentials, run no infrastructure for Phase A.
- Sync semantics: last-write-wins per record on `updatedAt`, deletes are tombstones (`deletedAt` + `updatedAt`), schema changes go through the migration ladder (`wc_meta.schemaVersion`, `Store.MIGRATIONS`).

## Consequences

- Pros: works fully offline; zero privacy exposure; no server costs at zero users; sync breakage degrades to "local still fine".
- Cons: UI must treat every server as untrusted/error-prone; merge is per-record LWW — field-level conflicts resolve bluntly; **multiple devices per account are fine, multiple users per device are not** (acceptable for a browser-extension context).
- Future note: for the social phase, the record format and LWW rule do not change — the server graduates from mirror to arbiter/transport. See `design/sync-protocol.md`.

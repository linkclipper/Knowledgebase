# LinkClipper Knowledgebase

The long-lived engineering memory of the org: decisions (ADRs), empirical research, and runbooks. Product code lives in [`linkclipper`](https://github.com/linkclipper/linkclipper); this repo holds *why*.

## Map

| Folder | Contents |
|---|---|
| `adr/` | Architecture decision records — high-level, numbered, immutable once merged (supersede, don't edit) |
| `research/` | Empirical findings with evidence — reports on platform behavior, tooling quirks |
| `runbooks/` | Step-by-step operational guides (store review, releases, incident response) |
| `design/` | Specifications for future architecture before it ships (sync protocol, social backend) |

## Conventions

- **ADRs**: `NNNN-kebab-title.md`, status (proposed/accepted/superseded), context → decision → consequences. Solo decision-making is fine; the record is for the future (including future-you).
- **Research**: state what was *tried and observed*, not what is believed. Include the repro when possible.
- **Runbooks**: assume a stressed operator. Exact commands, exact URLs, expected failure modes.

## Start here

- [ADR-0001 — Local-first storage with optional user-owned sync](adr/0001-local-first-optional-sync.md)
- [ADR-0002 — Per-site permissions, never install-time all-URLs](adr/0002-per-site-permissions-not-all-urls.md)
- [The store submission runbook](runbooks/cws-submission.md)

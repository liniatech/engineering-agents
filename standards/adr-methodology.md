# ADR & PRD Methodology

Read by `li-architect`, `li-database-engineer`, and `li-qa` during a project kickoff.
This is the numbering and layout every architecture document conforms to. The
orchestrator owns the numbers and file names; you write the content.

## The tree, rooted in intent

```
docs/scoping/                    raw material — the discovery interview, notes
  {SLUG}-discovery.md              what & why, in the user's words

docs/ADRs/                       one flat series, one slug per repo
  PRD-{SLUG}-001-product-requirements.md   the PRD — what & why, formalized
  ADR-{SLUG}-001-system-design.md          the SDD (C4 L1→L2→L3)
  ADR-{SLUG}-002-auth.md                   backbone …
  ADR-{SLUG}-009-testing.md                … backbone
  ADR-{SLUG}-010-<topic>.md                features and later decisions

docs/plan/                       the bridge to building
  {SLUG}-build-plan.md             the architecture broken into tasks
```

## Service-slug + fixed-topic numbering

Every document has an **identifier** `ADR-{SLUG}-NNN` (or `PRD-{SLUG}-NNN`) and a
**file name** that appends a topic slug: `ADR-{SLUG}-NNN-<topic>.md`.

- **`{SLUG}`** — a short **UPPERCASE** code (2–5 letters) derived from the
  service name (Roomie → `ROM`, desk-booking → `DESK`). **One slug per repo**,
  fixed for its life. No component prefixes. One flat `docs/ADRs/` series.
- **`NNN`** — the fixed-topic number (below).
- **`<topic>`** — a lowercase kebab-case word or two naming *what the file is*.
  It lives **only in the file name**, so a reader scanning the directory knows
  each file at a glance.

| Number | Topic | File name |
|---|---|---|
| `000` | reserved — SDLC / meta | — |
| `001` | the SDD (system design document) | `ADR-{SLUG}-001-system-design.md` |
| `002` | auth | `ADR-{SLUG}-002-auth.md` |
| `003` | database | `ADR-{SLUG}-003-database.md` |
| `004` | api | `ADR-{SLUG}-004-api.md` |
| `005` | observability | `ADR-{SLUG}-005-observability.md` |
| `006` | infrastructure | `ADR-{SLUG}-006-infrastructure.md` |
| `007` | ci/cd | `ADR-{SLUG}-007-ci-cd.md` |
| `008` | security | `ADR-{SLUG}-008-security.md` |
| `009` | testing | `ADR-{SLUG}-009-testing.md` |
| `010+` | features and subsequent decisions | `ADR-{SLUG}-0NN-<topic>.md` |

The PRD is `PRD-{SLUG}-001-product-requirements.md` — its own type, its own `001`.

## Identifier vs. file name

**The identifier carries no topic; the file name does.** Inside documents — the
title line and every cross-reference — use the bare identifier `ADR-{SLUG}-NNN`.
Only the file on disk gets the `-<topic>` suffix.

```
Title line:   # ADR-ROM-003 — Database
In prose:     "...the invariant is enforced in ADR-ROM-003."
File on disk: ADR-ROM-003-database.md
```

This keeps cross-references stable and short even if a topic word is later
reworded, while the directory listing stays self-describing.

## Splitting an oversized topic

If one backbone topic is too large for a single document, split it into an
index plus parts — the part number sits between the number and the topic:

```
ADR-{SLUG}-003-00-database.md   index — the map of the parts
ADR-{SLUG}-003-01-schema.md     part one
ADR-{SLUG}-003-02-indexing.md   part two
```

It is still one topic at one number.

## Rules

- **The slug is fixed for the life of the repo.** Confirm it once, at kickoff.
- **A number means a topic, always.** `003` is the database everywhere. Do not
  reuse a backbone number for a feature; features start at `010`.
- **Topic words in file names are lowercase kebab-case**, language-neutral even
  when the document body is written in another language.
- **Every document carries a last-verified date.** An ADR nobody has checked
  against the code is a hypothesis. See `standards/documentation.md`.
- **The SDD is the root.** `ADR-{SLUG}-001` holds the C4 view (L1 context →
  L2 containers → L3 components) and links down to the backbone ADRs that
  decide each piece. Read it first; it is the map.

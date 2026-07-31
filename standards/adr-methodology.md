# ADR & PRD Methodology

Read by `architect`, `database-engineer`, and `qa` during a project kickoff.
This is the numbering and layout every architecture document conforms to. The
orchestrator owns the numbers; you write the content.

## The tree, rooted in intent

```
docs/scoping/          raw material — the discovery interview, notes, links
  {SLUG}-discovery.md    what & why, in the user's words

docs/ADRs/             one flat series, one slug per repo
  PRD-{SLUG}-001.md      the PRD — what & why, formalized
  ADR-{SLUG}-001.md      the SDD — the system design (C4 L1→L2→L3)
  ADR-{SLUG}-002.md      backbone …
  ADR-{SLUG}-009.md      … backbone
  ADR-{SLUG}-010.md      features and later decisions

docs/plan/             the bridge to building
  {SLUG}-build-plan.md   the architecture broken into tasks
```

## Service-slug + fixed-topic numbering

Every document is `ADR-{SLUG}-NNN` or `PRD-{SLUG}-NNN`. **One slug per repo**,
kebab-case, derived from the service name. No component prefixes. One flat
`docs/ADRs/` series.

| Number | Topic |
|---|---|
| `000` | reserved — SDLC / meta |
| `001` | the SDD (system design document) |
| `002` | auth |
| `003` | database |
| `004` | api |
| `005` | observability |
| `006` | infrastructure |
| `007` | ci/cd |
| `008` | security |
| `009` | testing |
| `010+` | features and subsequent decisions |

`PRD-{SLUG}-001` is the product requirements doc — its own type, its own `001`.

## Splitting an oversized topic

If one backbone topic is too large for a single document, split it into an
index plus parts:

```
ADR-{SLUG}-003-00.md   index — the map of the parts
ADR-{SLUG}-003-01.md   part one
ADR-{SLUG}-003-02.md   part two
```

The index lists the parts and the decision each one records. Nothing else
changes — it is still one topic at one number.

## Rules

- **The slug is fixed for the life of the repo.** Confirm it once, at kickoff.
- **A number means a topic, always.** `003` is the database everywhere. Do not
  reuse a backbone number for a feature; features start at `010`.
- **Every document carries a last-verified date.** An ADR nobody has checked
  against the code is a hypothesis. See `standards/documentation.md`.
- **The SDD is the root.** `ADR-{SLUG}-001` holds the C4 view (L1 context →
  L2 containers → L3 components) and links down to the backbone ADRs that
  decide each piece. Read it first; it is the map.

---
description: Smoke-test li-project-kickoff by running it end-to-end on a brief (default: the Roomie brief) and regenerating the full example
argument-hint: [path to a brief file; defaults to examples/roomie/brief.md]
---

Run the `li-project-kickoff` skill as a **non-interactive self-test / demo**.

Brief: read the file at $ARGUMENTS if given, otherwise `examples/roomie/brief.md`.

This regenerates a complete kickoff example from the brief so we can verify the
skill still produces correct, convention-following output. Because it is a
demo, do **not** interview me — instead:

- Derive the discovery scoping from the brief, and mark every genuine unknown as
  `[NEEDS INPUT]` / `[PRECISA DE INPUT]` rather than inventing an answer.
- **Auto-approve the gates** (scoping, PRD, architecture) and say so in the
  report — a real `/li-kickoff` stops at each for a human.

Produce the documents in the **brief's language** (the Roomie brief is
Brazilian Portuguese) following the current conventions in
`standards/adr-methodology.md` and the skill's Output-style rules:

- slug = short UPPERCASE code; file names `ADR-{SLUG}-NNN-<topic>.md`, identifier
  short in titles/cross-refs
- concise and diagram-first: one plain `flowchart` per ADR; the SDD's three C4
  levels as captioned flowcharts (not the `C4Context` dialect); a task-dependency
  graph in the build plan

Spawn `li-product-manager`, `li-architect`, `li-database-engineer`, and `li-qa` exactly as
the skill defines; each returns document text and **you** (the orchestrator)
save it to its numbered path. Write into the brief's own directory (e.g.
`examples/roomie/docs/...`) and **create the full set**, nothing skipped:

- `docs/scoping/{SLUG}-discovery.md`
- `docs/ADRs/PRD-{SLUG}-001-product-requirements.md`
- `docs/ADRs/ADR-{SLUG}-001-system-design.md` (SDD)
- `docs/ADRs/ADR-{SLUG}-002-auth.md` … `ADR-{SLUG}-009-testing.md` (backbone)
- `docs/plan/{SLUG}-build-plan.md`

State up front that this **overwrites** the existing example, and do **not**
commit — leave that to me.

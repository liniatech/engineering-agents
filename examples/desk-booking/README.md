# Example: desk-booking kickoff output

This directory is a **worked example** — the complete document tree the
`project-kickoff` skill produces for one greenfield project. Nothing here is
production code; it exists to show what a kickoff *ends with*.

The sample project: an internal, SSO-gated web tool for ~120 hybrid employees
to reserve one of 70 office desks per day, with a hard "zero double-bookings"
guarantee.

## What's here, in the order kickoff produced it

```
docs/scoping/desk-booking-discovery.md     Phase 0 — the discovery interview notes
docs/ADRs/PRD-desk-booking-001.md          Phase 1 — the PRD (product-manager)
docs/ADRs/ADR-desk-booking-001.md          Phase 2 — the SDD, C4 L1→L2→L3 (architect)
docs/ADRs/ADR-desk-booking-002.md            backbone: auth        (architect)
docs/ADRs/ADR-desk-booking-003.md            backbone: database    (database-engineer)
docs/ADRs/ADR-desk-booking-004.md            backbone: api         (architect)
docs/ADRs/ADR-desk-booking-005.md            backbone: observability (architect)
docs/ADRs/ADR-desk-booking-006.md            backbone: infrastructure (architect)
docs/ADRs/ADR-desk-booking-007.md            backbone: ci/cd       (architect)
docs/ADRs/ADR-desk-booking-008.md            backbone: security    (architect)
docs/ADRs/ADR-desk-booking-009.md            backbone: testing     (qa)
docs/plan/desk-booking-build-plan.md       Phase 3 — the task breakdown  ← kickoff ends here
```

## Reading it

- Start with **`ADR-desk-booking-001`** (the SDD) — it holds the C4 diagrams and
  links down to every backbone decision.
- The thread to follow is the **zero-double-booking invariant**: stated in the
  SDD, physically decided in `-003` (a partial unique index), made observable in
  `-004` (409-vs-201), and proven in `-009` (the AC-5 concurrency merge gate).
- Note the `[NEEDS INPUT]` markers — kickoff surfaces unknowns rather than
  inventing answers, and the build plan maps each one to the task it blocks.

## How it was generated

Run `/kickoff` and answer the discovery interview. See
[`../../GUIDE.md`](../../GUIDE.md#start-a-greenfield-project--kickoff) and the
numbering convention in
[`../../standards/adr-methodology.md`](../../standards/adr-methodology.md).

> Generated as a dry-run of the `project-kickoff` skill. The interview answers
> for this sample were simulated; a real run collects them from you.

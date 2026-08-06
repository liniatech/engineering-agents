---
name: li-architecture-review
description: Reviews a design, ADR, RFC, or existing system against the architecture principles — boundaries, coupling, contracts, failure modes, observability. Use to review a proposed design before build, or to audit an existing system's structure. Design-level, not line-level.
---

# Architecture review

This reviews *structure*, not code. If the question is "is this diff good",
use `li-code-review` instead.

## Deliverable — a file, always

This skill produces **`docs/reviews/<date>-<slug>-architecture.md`**, using
`templates/design-review.md`.

Read `standards/deliverables.md`. The review is written to disk with the
`Write` tool **before** anything is printed in chat; the chat output is the
verdict, the top concerns, and the path. Architecture findings outlive the
conversation that produced them — they have to be readable next quarter.

## Steps

### 1. Establish what is under review

One of: a proposed ADR/RFC, an existing subsystem, or a change that crosses
service boundaries. Get the artifact or map the code.

### 2. Reconstruct the current state

Before judging a proposal, know what is actually there. Use Glob and Grep to
find service boundaries, data ownership, synchronous call chains, and shared
databases. Cite files. A review based on the diagram rather than the code
reviews a fiction.

### 3. Review against the principles

Read `standards/architecture-principles.md` and `standards/api-guidelines.md`,
then work through each lens:

**Boundaries** — Does each service own its data? Any shared-database
coupling? Is the boundary drawn along a real seam, or along team lines?

**Coupling** — What breaks when this changes? Trace the synchronous call
chain — how deep is it, and what is the compounded availability?

**Contracts** — Are they explicit and versioned? Is there a migration path
for breaking changes? Who is allowed to break whom?

**Failure modes** — For each external call: what happens on timeout, on
error, on slow-but-successful? Where is the retry, and is it idempotent?
Where is the backpressure? Unbounded queues and unbounded retries are the
two most common omissions.

**State and consistency** — Where is the source of truth? What is eventually
consistent, and does the product actually tolerate that?

**Observability** — Can you tell, from outside, that this is working? What
is the one metric that goes bad first?

**Reversibility** — How hard is this to undo in six months? This is the most
underweighted question in most designs.

### 4. Judge the alternatives honestly

Did the proposal consider real alternatives, or one option and two straw men?
If the tradeoff was not named, it was not made. Ask what is being given up.

### 5. Check for the two failure directions

- **Over-engineering** — abstraction for a requirement nobody has, a service
  boundary where a function would do, configurability nobody asked for.
- **Under-engineering** — no failure handling, no migration path, a decision
  that is cheap now and unreversible later.

Both are findings. Reviewers reliably catch the second and miss the first.

### 6. Write the review file — before reporting

1. `mkdir -p docs/reviews`
2. `date +%F` for the date — never guess it.
3. **Call the `Write` tool** to create
   `docs/reviews/<date>-<slug>-architecture.md` from
   `templates/design-review.md`, filling every section:

- **Verdict** — sound | concerns | reconsider
- **What this is** — your reconstruction of the design in your own words. A
  mismatch with the author's intent is itself the first finding.
- **Scope reviewed** — and what was explicitly not.
- **Boundaries, contracts, failure modes** — the lens tables from step 3.
- **Concerns** — ordered by cost-to-fix-later, not severity-today. Each: the
  concern, the scenario that makes it bite, the earliest warning signal, the
  cost now vs. later.
- **Unstated tradeoffs** — what is being given up that the document does not
  admit.
- **Over- / under-engineering** — both directions, from step 5.
- **Reversibility** — each major decision: cheap / expensive / one-way.
- **Alternatives** — real ones, or one option and two straw men?
- **Questions for the author** and **Action items** with owners.

### 7. Report — the pointer, not the review

Only after the file exists:

```
Verdict:     sound | concerns | reconsider
Design:      <what was reviewed>
Concerns:    N — <the top two, one line each>
One-way:     <decisions that are expensive or impossible to undo>
Questions:   N open for the author
Review:      docs/reviews/<date>-<slug>-architecture.md
```

## Never

- Never review a design without reading the code it will live in.
- Never propose a rewrite as a review finding. If that is your conclusion,
  say it plainly as its own recommendation with a cost estimate.
- Never approve a design whose failure modes are undefined.
- Never print the review in chat instead of writing the file. A verdict with
  no document behind it cannot be revisited when the design changes.

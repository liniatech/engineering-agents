---
name: project-kickoff
description: Use when starting a brand-new project or service from scratch — turning "what I want to build" into a documented architecture. Runs a discovery interview, then a PRD, SDD, and ADR backbone, and ends with a ready-to-build task list. Greenfield only; for a feature inside an existing codebase use new-feature.
---

# Project kickoff pipeline

You are the orchestrator. You run the discovery interview yourself, then spawn
agents to produce the documents, hold the gates, and stop at the task list.
You do not design or build — the agents do that, and building is a later step.

## What this produces

A greenfield service's architecture, documented and review-ready:

```
docs/scoping/{SLUG}-discovery.md     your interview notes
docs/ADRs/PRD-{SLUG}-001.md          the PRD
docs/ADRs/ADR-{SLUG}-001.md          the SDD (C4 L1→L2→L3)
docs/ADRs/ADR-{SLUG}-002…009.md      the backbone
docs/plan/{SLUG}-build-plan.md       the task breakdown  ← final deliverable
```

It **ends at the task list.** It does not build. Each task is built afterward,
separately (e.g. `/feature` per task).

## The convention you own

You own the numbering; the agents write the content. Read
`standards/adr-methodology.md` — it defines the tree, the slug rule, and the
fixed `002–009` backbone mapping (auth · db · api · observability · infra ·
cicd · security · testing). When you spawn an agent, pass it the *exact* output
path and the template to fill; agents never invent the numbering.

## Artifact rule

Subagents share no context. Anything crossing a handoff is written to a file
and its **path** passed to the next agent. Agents return the document text;
**you** save it to the numbered path.

## Steps

### 0. Discovery interview — you, not an agent

This is the part only you do. Ask the user about the business and the problem,
**one question at a time**, following `templates/scoping.md`: the problem, who
has it, why now, today's alternative, what success looks like, constraints,
non-goals. Do not ask the next question until the last is answered. Do not
invent answers — an unanswered question stays open.

Propose a `{SLUG}` (kebab-case, from the service name) and get the user to
confirm it. It is fixed for the life of the repo.

Write the notes to `docs/scoping/{SLUG}-discovery.md`.

**GATE — human.** Show the scoping summary and the open questions. Get a yes
before spending an agent on a PRD built on the wrong problem.

### 1. PRD

Spawn `product-manager` with the discovery-doc path. It returns a PRD; save it
to `docs/ADRs/PRD-{SLUG}-001.md`.

**GATE — human.** Show the Summary and Open questions. Do not proceed on
unanswered `[NEEDS INPUT]` markers that affect scope.

### 2. Architecture — drawn, then decided

Spawn `architect` with the PRD path. Tell it to produce, using
`templates/sdd.md` and `templates/adr.md`:

- `ADR-{SLUG}-001` — the SDD, with C4 as mermaid: L1 context → L2 containers →
  L3 components, and the decisions table pointing at the backbone ADRs.
- The backbone ADRs it owns: `002` auth, `004` api, `005` observability,
  `006` infra, `007` cicd, `008` security.

Then spawn the specialists for their backbone topics, each with the PRD and SDD
paths:

- `database-engineer` → `ADR-{SLUG}-003` (database). Schema is **designed and
  documented here — no migration is written.** This is a decision doc.
- `qa` → `ADR-{SLUG}-009` (testing). The test strategy the acceptance criteria
  will lean on.

Save each returned document to its numbered path. If a backbone topic is
genuinely too large for one document, split it (`-00` index + `-01/-02`) per
`standards/adr-methodology.md`.

**GATE — human.** Show the SDD's Recommendation/Risks and the backbone map.
Architecture is the expensive thing to undo; get a human yes.

### 3. Task breakdown — you

Read the backbone ADRs and the PRD and decompose them into build tasks, using
`templates/build-plan.md`. Every task cites the ADR it implements and carries
acceptance criteria drawn from the PRD and `ADR-{SLUG}-009`. Order the tasks by
dependency. Write to `docs/plan/{SLUG}-build-plan.md`.

**GATE — human.** Show the task list. This is where kickoff ends.

### 4. Report and stop

Summarize:

- The `{SLUG}`, and the paths to every document produced.
- The task list, and the recommended first task.
- Open questions still unresolved, and any gate you skipped and why.

State plainly that the next step is building — one task at a time, via a
separate run. **Do not start building. Do not commit or push.** That is the
user's call.

## Never

- Never skip the discovery interview and jump to the PRD. The interview is the
  point — it is what makes the PRD about the real problem.
- Never let an agent choose its own document number. You own the numbering.
- Never let `database-engineer` write a migration here. Kickoff documents the
  schema; migrations belong to `database-change` at build time.
- Never continue into building. Kickoff ends at the task list, even if the user
  seems to want more — offer the next step, don't take it.

## Failure honesty

If an agent returned BLOCKED, say so and route it (spec gap → back to the PRD;
architecture conflict → back to the architect). If you skipped a gate, say you
skipped it. A kickoff report that hides a skipped gate is worse than none.

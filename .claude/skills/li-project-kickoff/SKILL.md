---
name: li-project-kickoff
description: Use when starting a brand-new project or service from scratch — turning "what I want to build" into a documented architecture. Runs a discovery interview, then a PRD, SDD, and ADR backbone, and ends with a ready-to-build task list. Greenfield only; for a feature inside an existing codebase use li-new-feature.
---

# Project kickoff pipeline

You are the orchestrator. You run the discovery interview yourself, then spawn
agents to produce the documents, hold the gates, and stop at the task list.
You do not design or build — the agents do that, and building is a later step.

## What this produces

A greenfield service's architecture, documented and review-ready:

```
docs/scoping/{SLUG}-discovery.md                  your interview notes
docs/ADRs/PRD-{SLUG}-001-product-requirements.md  the PRD
docs/ADRs/ADR-{SLUG}-001-system-design.md         the SDD (C4 L1→L2→L3)
docs/ADRs/ADR-{SLUG}-002-auth.md … 009-testing.md the backbone
docs/plan/{SLUG}-build-plan.md                    the task breakdown  ← final deliverable
```

It **ends at the task list.** It does not build. Each task is built afterward,
separately (e.g. `/li-feature` per task).

## The convention you own

You own the numbering **and the file names**; the agents write the content.
Read `standards/adr-methodology.md` — it defines the tree, the slug rule, and
the fixed `002–009` backbone mapping. In short:

- **`{SLUG}`** is a short **UPPERCASE** code (2–5 letters) from the service name
  (Roomie → `ROM`), fixed for the repo. Confirm it in Phase 0.
- **File names carry a topic suffix**: `ADR-{SLUG}-NNN-<topic>.md`. Canonical
  topics: `001-system-design · 002-auth · 003-database · 004-api ·
  005-observability · 006-infrastructure · 007-ci-cd · 008-security ·
  009-testing`; the PRD is `-product-requirements`; features (`010+`) get an
  author-chosen kebab topic.
- **The identifier stays short.** Titles and in-prose cross-references use the
  bare `ADR-{SLUG}-NNN` — the `-<topic>` suffix lives only in the file name.

When you spawn an agent, pass it the *exact* output file name and the template
to fill; agents never invent the numbering or the file name.

## Artifact rule

Subagents share no context. Anything crossing a handoff is written to a file
and its **path** passed to the next agent. Agents return the document text;
**you** save it to the numbered path with the `Write` tool, before the gate for
that step.

Read `standards/deliverables.md`. Showing a document in chat is not saving it.
Every path in *What this produces* must exist on disk by the time you report.

## Output style — diagram-first and tight

Kickoff docs are read by busy people; optimise for scanning, not completeness.
Pass these limits to every agent you spawn.

- **One diagram per ADR, always.** Each backbone ADR carries exactly one
  mermaid diagram that shows the decision at a glance. The SDD carries three
  **`flowchart`** diagrams — the C4 levels (context → containers → components),
  each with a one-line caption; not the `C4Context` dialect, which renders
  poorly. The build plan carries a task-dependency graph.
  Suggested diagram per topic:

  | ADR | Topic | Diagram |
  |---|---|---|
  | 002 | auth | `sequenceDiagram` of the login flow |
  | 003 | database | UML class diagram (`classDiagram`) of the tables |
  | 004 | api | `sequenceDiagram` of the success-vs-conflict path |
  | 005 | observability | `flowchart` of the signal → alert path |
  | 006 | infrastructure | `flowchart`/C4 deployment view |
  | 007 | ci/cd | `flowchart` of the pipeline stages |
  | 008 | security | `flowchart` of the trust boundary |
  | 009 | testing | the test pyramid as a `flowchart` |

- **Tight prose.** Each backbone ADR ≤ ~350 words outside its diagram/tables;
  the SDD ≤ ~500. Lead with the Decision. Do not restate the PRD or context the
  reader already has. Prefer a table or list over a paragraph.
- **Delete, don't pad.** One decision per ADR; if a section adds nothing,
  remove it rather than fill it.

## Language

Kickoff produces every document in one language: **English (`en`)** or
**Brazilian Portuguese (`pt-BR`)**. Settle it before Phase 1:

- If the user passed one (e.g. `/li-kickoff --lang pt-BR …`), use it.
- Otherwise ask once during Phase 0; default to English if they have no
  preference.

Record the choice in the discovery doc and **pass it to every agent** — they
write all prose (and may localise section headings) in that language. Keep
language-neutral: document IDs (`ADR-{SLUG}-NNN`), file names, the slug, code,
SQL, and mermaid keywords.

## Stack defaults — from the brief

The kickoff may start from a `brief.md` (see `templates/brief.md`), whose
**Parameters** block sets `Language`, `Infra`, and `Programming language`. Read
them in Phase 0 and carry them through the pipeline. A value in the brief is
**decided** — do not re-ask it in discovery or mark it `[NEEDS INPUT]`.

- **Infra defaults to Terraform + AWS.** Unless the brief's `Infra` field
  overrides it, the architecture targets **AWS provisioned with Terraform** — pass
  this to the `li-architect` so `ADR-{SLUG}-006` (infrastructure) and `ADR-{SLUG}-007`
  (ci/cd) decide on that basis (IaC = Terraform, cloud = AWS). If the brief names a
  different target, that override wins; record it and its reason.
- **Programming language** — pass the brief's value to every agent that writes
  code-shaped decisions. If blank, it stays `[NEEDS INPUT]`; settle it in discovery
  rather than inventing a stack.
- **Language** (docs) — as in the Language section above; the brief's value, if
  set, skips the Phase 0 question.

When a brief sets none of these (or there is no brief), the defaults apply:
`Language = en`, `Infra = Terraform + AWS`, `Programming language = [NEEDS INPUT]`.

## Steps

### 0. Discovery interview — you, not an agent

This is the part only you do. If a `brief.md` exists, read it first — take its
**Parameters** (`Language`, `Infra`, `Programming language`) as decided (see
*Stack defaults*) and don't re-ask them. Then ask the user about the business and
the problem, **one question at a time**, following `templates/scoping.md`: the
problem, who has it, why now, today's alternative, what success looks like,
constraints, non-goals. Do not ask the next question until the last is answered.
Do not invent answers — an unanswered question stays open.

Propose a `{SLUG}` — a short UPPERCASE code (2–5 letters) from the service name
(Roomie → `ROM`) — and get the user to confirm it. It is fixed for the life of
the repo.

Settle the output **language** here (see the Language section) if it was not
passed on the command line.

Write the notes to `docs/scoping/{SLUG}-discovery.md`, and record the chosen
language in that file.

**GATE — human.** Show the scoping summary and the open questions. Get a yes
before spending an agent on a PRD built on the wrong problem.

### 1. PRD

Spawn `li-product-manager` with the discovery-doc path, the chosen language, and
the Output-style limits (tight prose, no restated context). It returns a PRD;
save it to `docs/ADRs/PRD-{SLUG}-001-product-requirements.md`. Its title line
and cross-refs use the bare identifier `PRD-{SLUG}-001`.

**GATE — human.** Show the Summary and Open questions. Do not proceed on
unanswered `[NEEDS INPUT]` markers that affect scope.

### 2. Architecture — drawn, then decided

Spawn `li-architect` with the PRD path, the chosen language, the **stack defaults**
(infra = the brief's `Infra` or **Terraform + AWS**; programming language = the
brief's value or `[NEEDS INPUT]`), and the Output-style limits (one mermaid
diagram per ADR from the topic table; each backbone ADR ≤ ~350 words, the SDD
≤ ~500; decision-first). Tell it that `ADR-{SLUG}-006` (infrastructure) targets
AWS with Terraform as the IaC unless the brief overrode it. Tell it to produce,
using `templates/sdd.md` and `templates/adr.md`:

- `ADR-{SLUG}-001-system-design.md` — the SDD. Draw the three C4 levels as
  plain `flowchart` diagrams (context → containers → components), each opened by
  a one-line "what this shows" caption; add the decisions table pointing at the
  backbone ADRs.
- The backbone ADRs it owns: `002-auth`, `004-api`, `005-observability`,
  `006-infrastructure`, `007-ci-cd`, `008-security`.

Then spawn the specialists for their backbone topics, each with the PRD and SDD
paths, the chosen language, and the same Output-style limits (one diagram,
≤ ~350 words):

- `li-database-engineer` → `ADR-{SLUG}-003-database.md`. Schema is **designed and
  documented here — no migration is written.** This is a decision doc. Draw the
  tables as a **UML class diagram** (mermaid `classDiagram`) — one class per table,
  each column written **`name: type`** followed by its key marker (`PK`/`FK`/`UK`)
  where it applies, plus the relationships between tables.
  Follow the diagram with an **explanation of each entity** — one line per table
  saying what it is and **why it exists in business terms**, tied to the PRD
  requirement it serves. A reader who doesn't know the schema must be able to see
  why every table is there and how it maps to the product.
- `li-qa` → `ADR-{SLUG}-009-testing.md`. The test strategy the acceptance criteria
  will lean on.

Save each returned document to its file name from the topic table (identifier +
`-<topic>.md`); the document's own title uses the bare `ADR-{SLUG}-NNN`. If a
backbone topic is genuinely too large for one document, split it
(`-00`/`-01`/`-02` part number before the topic) per
`standards/adr-methodology.md`.

**GATE — human.** Show the SDD's Recommendation/Risks and the backbone map.
Architecture is the expensive thing to undo; get a human yes.

### 3. Task breakdown — you

Read the backbone ADRs and the PRD and decompose them into build tasks, using
`templates/build-plan.md`. Every task cites the ADR it implements and carries
acceptance criteria drawn from the PRD and `ADR-{SLUG}-009`. Order the tasks by
dependency and include a **mermaid task-dependency graph**. Keep each task to a
line or two. Write it in the chosen language to `docs/plan/{SLUG}-build-plan.md`.

**Cover the whole path to a deployed project — not just application code.** The
plan must contain explicit tasks for **all the infrastructure** in
`ADR-{SLUG}-006` (IaC/state, networking, data & storage, compute, TLS/CDN — each
provisioned) and for **standing up the CI/CD delivery pipeline** in
`ADR-{SLUG}-007` (build → registry → IaC apply → migrations → deploy → health
check), through to a working deployable environment. Do **not** fold infra and
delivery into a single "foundation" task; break them out so each is picked up,
built, and verified on its own. Application-feature tasks depend on that
foundation.

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
- Never let an agent choose its own document number or file name. You own the
  numbering and the `-<topic>` file-name suffix.
- Never let `li-database-engineer` write a migration here. Kickoff documents the
  schema; migrations belong to `li-database-change` at build time.
- Never continue into building. Kickoff ends at the task list, even if the user
  seems to want more — offer the next step, don't take it.

## Failure honesty

If an agent returned BLOCKED, say so and route it (spec gap → back to the PRD;
architecture conflict → back to the architect). If you skipped a gate, say you
skipped it. A kickoff report that hides a skipped gate is worse than none.

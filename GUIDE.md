# Using the engineering-agents plugin

A practical, task-oriented guide. For *why* the plugin is built the way it is,
read the [README](README.md). This document is about *what to type* and *what
to expect back*.

## Before you start

Install the plugin (details in the [README](README.md#install-as-a-plugin)):

```
# test in one session, nothing registered
claude --plugin-dir /path/to/engineering-agents

# or install persistently
/plugin marketplace add liniatech/engineering-agents
/plugin install engineering-agents@liniatech
```

Once installed, confirm it loaded:

```
/help          # the commands (/li-kickoff, /li-feature, /li-bugfix, /li-diagnose, /li-review, /li-spec, /li-demo) appear
/agents        # the six specialists appear
```

There is nothing to configure. The agents read `standards/` and `templates/`
from the repo they run in — so if you want them to follow *your* conventions,
keep those two directories in your project (or edit the copies shipped here).

## The fastest way to pick an entry point

| You want to… | Use | It will… |
|---|---|---|
| Start a greenfield project | `/li-kickoff <what>` | interview you, then PRD → SDD + ADR backbone → task list (docs only, no code) |
| Build something new | `/li-feature <what>` | spec → architecture → schema → build → review, with approval gates |
| Fix a defect | `/li-bugfix <the bug>` | diagnose (gated) → failing test → ticket → fix → verify |
| Diagnose without fixing | `/li-diagnose <the bug>` | deep root cause + repro test + bug ticket, then stop |
| Check code before merge | `/li-review [PR/branch]` | run two review lenses, verify findings, report (no auto-fix) |
| Turn an idea into a PRD | `/li-spec <idea>` | write the PRD only — no design, no code |

You don't *have* to use the commands. You can describe the task in plain
language and Claude will match it to the right skill by its description. The
commands exist for when you want to be explicit and deterministic.

## Walkthroughs

### Start a greenfield project — `/li-kickoff`

```
/li-kickoff a service that ingests bank webhooks and reconciles them nightly
/li-kickoff --lang pt-BR um serviço de reserva de mesas para o escritório
```

Use this at the very beginning of a **new** project — before there's code to
put a feature into. Documents come out **tight and diagram-first** (one mermaid
diagram per ADR, C4 in the SDD, a task-dependency graph in the plan), in the
**language you choose** — English or Brazilian Portuguese (`--lang en|pt-BR`, or
it asks). What happens, in order:

1. **Discovery interview** — Claude (the orchestrator, not an agent) asks you
   about the business and the problem, **one question at a time**: who has the
   problem, why now, what success looks like, constraints. It also proposes a
   `{SLUG}` — a short uppercase code like `ROM` — and asks you to confirm it.
   Notes land in `docs/scoping/{SLUG}-discovery.md`. **You approve the scoping.**
2. **PRD** — `li-product-manager` writes `PRD-{SLUG}-001` to the file
   `PRD-{SLUG}-001-product-requirements.md`. **You approve it.**
3. **Architecture drawn** — `li-architect` writes the SDD (`ADR-{SLUG}-001`, file
   `…-001-system-design.md`) with C4 context→container→component diagrams, plus
   the backbone ADRs it owns; `li-database-engineer` writes the database ADR
   (`…-003-database.md`, schema *designed*, not migrated) and `li-qa` writes the
   testing ADR (`…-009-testing.md`). File names carry a topic suffix; the
   in-doc identifier stays `ADR-{SLUG}-NNN`. **You approve the architecture.**
4. **Task breakdown** — the documented architecture is decomposed into a
   dependency-ordered task list in `docs/plan/{SLUG}-build-plan.md`, each task
   citing its ADR and acceptance criteria.

Then it **stops.** Kickoff produces docs only — no code. Build the tasks
afterward, one at a time, with `/li-feature`. See
[the numbering methodology](standards/adr-methodology.md) for the `ADR-{SLUG}-NNN`
convention.

### Build a feature — `/li-feature`

```
/li-feature add rate limiting to the public API, 100 req/min per API key
```

What happens, in order:

1. **Spec** — the `li-product-manager` agent writes a PRD (user stories,
   acceptance criteria, edge cases). **You approve it before anything else
   runs.** This gate is deliberate — the spec is cheap to change now and
   expensive to change after code exists.
2. **Architecture** — the `li-architect` agent produces an ADR with tradeoffs
   (where the counter lives, what happens on a store outage, the contract
   change). **You approve this gate too.**
3. **Schema** — if the change touches the database, the `li-database-engineer`
   writes a reversible migration. *Agents never run migrations* — you do.
4. **Build** — the `li-backend-engineer` implements it with tests.
5. **Review** — a *fresh* `li-reviewer` and `li-qa` agent check the diff. The
   review→fix loop is capped at three rounds, then it escalates to you.

Artifacts land in `docs/features/<slug>/` so each handoff is a real file, not
a fact trapped in one agent's memory.

**Skip a gate on purpose?** The command tells the orchestrator it must announce
any skipped gate and why — it won't silently bypass the spec or architecture
approval.

### Fix a bug — `/li-bugfix`

```
/li-bugfix users with a + in their email get a 500 on password reset
```

The pipeline is root-cause-first and will **refuse to patch a symptom**. It runs
in two phases with a gate between them.

**Phase 1 — diagnose:**

1. Reproduce it. If it can't, it stops and tells you what it needs.
2. Root cause **by elimination** — at least two hypotheses, the evidence that
   refuted each, and the survivor with `file:line`. A single untested theory is
   not accepted as a cause.
3. Map the **blast radius** — every other site sharing the same cause, each with
   a verdict. Done here, before anything is patched, not after.
4. Write a **failing test** that captures the bug, run it, confirm it fails for
   the right reason.
5. Write the ticket to `docs/bugs/<slug>.md` with a task title you can paste
   straight into a tracker.

**── gate ──** you see the title, cause, what was ruled out, and the blast
radius, and approve before any production code is touched.

**Phase 2 — fix:** fix the cause only → verify (suite + original repro) →
fresh-spawn review → close the ticket.

Give it repro steps if you have them; it works far better with them. A bare
error message is a fine starting point though — it will investigate before
asking you for more.

### Diagnose without fixing — `/li-diagnose`

```
/li-diagnose KeyError: 'user_id' in webhooks/handlers.py, ~40x/day in prod
```

Same phase 1, then it **stops**. You get the deep diagnosis, the reproducing
test, and the written ticket — no fix, and it won't offer one. Use it when you
want to triage and file, and let someone else (or a later session) do the fix:

```
/li-bugfix docs/bugs/webhook-drops-events-missing-user-id.md
```

Passing a ticket path to `/li-bugfix` skips phase 1 — it re-confirms the repro
and goes straight to fixing.

### Review code — `/li-review`

```
/li-review 482            # a PR number
/li-review my-branch      # a branch
/li-review                # blank → the working tree, or the branch vs its base
```

Two lenses run in parallel: `li-reviewer` (correctness, security, readability,
performance) and `li-qa` (does the test suite actually prove the spec). Findings
are **verified before they're reported**, and nothing is fixed without asking
you first.

### Spec only — `/li-spec`

```
/li-spec a digest email that summarizes a user's activity for the past week
```

Produces a PRD and nothing else — no architecture, no code. Every open
question is surfaced rather than answered by assumption. Use this when the
requirements are vague or contested and you want them nailed down before
committing engineering time.

## All ten skills

The five commands cover the most common paths, but the plugin ships **ten**
workflow skills. The others have no slash command — you invoke them by
describing the task, and Claude matches the skill by its description. (You can
also name one explicitly: *"use the incident skill"*.)

| Skill | Command | Invoke by saying… | Core discipline |
|---|---|---|---|
| `li-project-kickoff` | `/li-kickoff` | "new project / start building X from scratch" | discovery interview → PRD → SDD + ADR backbone → task list; docs only |
| `li-new-feature` | `/li-feature` | "build / add …" | human gates after spec and architecture |
| `li-bug-fix` | `/li-bugfix`, `/li-diagnose` | "fix … / this is broken" / "diagnose this error" | hypothesis elimination → blast radius → **failing test** → ticket → gate → fix |
| `li-code-review` | `/li-review` | "review this PR / is this ready to merge" | two lenses, findings verified before reported |
| `li-architecture-review` | — | "review this design / ADR / RFC" | boundaries & failure modes; catches over- *and* under-engineering |
| `li-database-change` | — | "add a column / write this migration / add an index" | expand/contract, lock analysis; **humans run migrations** |
| `li-dependency-upgrade` | — | "bump … / patch this CVE / upgrade the framework" | read every intermediate changelog, apply incrementally |
| `li-incident` | — | "prod is down / this alert is firing / outage" | **stabilize first**, diagnose second, then postmortem |
| `li-performance-review` | — | "this is slow / latency regressed / optimize …" | **no optimizing without a measurement** |
| `li-release` | — | "cut a release / prep the deploy / tag a version" | migration ordering + written abort criteria; humans deploy |

A few worth calling out because their guardrails change how they behave:

- **`li-incident`** optimizes for *restoring service*, not elegance. It will
  mitigate first (a revert, a flag flip) and only then diagnose. Reach for it
  when something is broken *right now*.
- **`li-database-change`** and **`li-release`** both stop short of the irreversible
  step: the migration gets written and the deploy gets verified, but a human
  runs them. That boundary is intentional.
- **`li-performance-review`** refuses to optimize without a profile, and
  **`li-dependency-upgrade`** refuses to skip intermediate changelogs — same
  root-cause-first spirit as `li-bug-fix`.

Examples:

```
review the design in docs/adr/0007-event-bus.md against our architecture principles
add a nullable `deleted_at` column to orders for soft deletes
upgrade Django from 4.2 to 5.0
checkout is timing out for carts with 50+ items — find out why
we're cutting the 2.4 release tomorrow, prep the checklist
prod payments are returning 500s, help
```

## Conventions the skills follow

What the pipelines produce and how — the rules an outside reader (or a
teammate) needs to know when using the skills. Kickoff is the fullest example;
most of these hold across the doc-producing skills.

**Document naming** (see [`standards/adr-methodology.md`](standards/adr-methodology.md))
- **Slug**: a short UPPERCASE code per repo (Roomie → `ROM`), fixed for its life.
- **File name**: identifier + topic suffix — `ADR-{SLUG}-NNN-<topic>.md`
  (e.g. `ADR-ROM-003-database.md`). The PRD is `PRD-{SLUG}-001-product-requirements.md`.
- **Identifier** in titles and cross-references stays short: `ADR-ROM-003` — the
  topic lives only in the file name.
- **Fixed backbone numbers**: `001` system-design (SDD) · `002` auth · `003`
  database · `004` api · `005` observability · `006` infrastructure · `007`
  ci/cd · `008` security · `009` testing · `010+` features.

**Output style**
- **Concise and diagram-first**: each ADR ≤ ~350 words with **exactly one**
  mermaid diagram; the SDD ≤ ~500 with its three C4 levels.
- **Diagrams are plain `flowchart`s with a one-line caption** — *not* the
  `C4Context` mermaid dialect, which renders poorly. The build plan carries a
  task-dependency graph.
- **Language**: docs are produced in English or Brazilian Portuguese
  (`/li-kickoff --lang en|pt-BR`, or it asks). Prose and headings localize; IDs,
  file names, slug, code, SQL, and mermaid keywords stay neutral.

**Who writes what**
- The **orchestrator** (Claude, following the skill) runs the discovery
  interview, owns the numbering and file names, holds the gates, and saves every
  document. It does not write the content.
- **Agents** write the content and **return it as text** — they do not write
  files (the artifact rule). `li-product-manager` → PRD; `li-architect` → SDD +
  `002/004/005/006/007/008`; `li-database-engineer` → `003` (schema *designed*, no
  migration); `li-qa` → `009`.

**Gates** — kickoff stops for human approval after scoping, after the PRD, and
after the architecture, then ends at the task list. It never builds or commits.

## Running agents by hand

The commands orchestrate agents for you, but you can drive a single specialist
directly when you want tight control or you're learning where each one is weak:

```
Use the li-architect agent to review the data flow in docs/features/rate-limit/
```

The six agents and what each owns (and refuses):

| Agent | Owns | Will not |
|---|---|---|
| `li-product-manager` | PRD, acceptance criteria, edge cases | design or code |
| `li-architect` | boundaries, contracts, ADRs | write code or design schema |
| `li-database-engineer` | schema, indexes, migrations | write app code or run migrations |
| `li-backend-engineer` | implementation + tests | change schema, infra, or requirements |
| `li-reviewer` | code quality, security, correctness | write code (it has no Edit/Write tool) |
| `li-qa` | spec↔code↔test coverage gaps | change production code |

These limits are structural, not polite requests — `li-reviewer` literally has no
write tools, so it cannot rubber-stamp by fixing.

## How agents communicate

They don't talk to each other directly — and that's the point. **No agent can
spawn or call another** (`li-backend-engineer.md`: *"You cannot spawn other
agents."*). All coordination goes through two things: a structured handoff
line each agent returns, and the orchestrating skill that reads it and routes.

**Each agent ends with a fixed `## Handoff` line — a vocabulary, not prose:**

| Agent | Returns |
|---|---|
| `li-product-manager` | `NEXT: li-architect \| BLOCKED: <reason>` |
| `li-architect` | `NEXT: li-database-engineer \| li-backend-engineer \| BLOCKED: <reason>` |
| `li-database-engineer` | `NEXT: li-backend-engineer \| BLOCKED: <reason>` |
| `li-backend-engineer` | `NEXT: li-reviewer \| BLOCKED: <reason>` |
| `li-qa` | `NEXT: li-backend-engineer \| done \| BLOCKED: <reason>` |
| `li-reviewer` | its structured findings; `BLOCKED: self-review` if it wrote the diff |

The orchestrating skill reads that line and decides what to spawn next. An
agent that hits a wall doesn't call a peer — it *stops* and returns a
`BLOCKED:` line explaining why, and the orchestrator routes it. That's how
`li-backend-engineer` asks for a schema change (`BLOCKED: needs schema change`),
and how `li-qa` refuses to edit code (`BLOCKED: implementation defect`).

**Why the indirection?** A subagent starts with an empty context window — it
cannot see your conversation, the orchestrator's reasoning, or what a previous
agent produced. So anything that must survive a handoff has to be
*materialized* (`li-new-feature/SKILL.md`):

> "Everything that must survive a handoff has to be either **written to a
> file** or **pasted into the next agent's prompt**."

That's why the pipeline writes real artifacts and passes their *paths* forward:

```
docs/features/<slug>/prd.md        ← li-product-manager writes it
docs/features/<slug>/adr.md        ← li-architect reads prd.md, writes this
docs/features/<slug>/migration.md  ← li-database-engineer
```

The shape of it:

```
         ┌────────────── orchestrator (the skill) ───────────────┐
         │  spawns each agent · reads its Handoff · routes next   │
         └────┬──────────┬───────────┬───────────┬───────────────┘
  spawn+paths │          │           │           │
              ▼          ▼           ▼           ▼
       product-mgr → architect → database-eng → backend-eng → reviewer/qa
              │          │           │           │              │
   returns: NEXT/      NEXT/       NEXT/     NEXT or         BLOCKED:
            BLOCKED    BLOCKED     BLOCKED   BLOCKED         self-review
              └──────────┴───────────┴───────────┴──────────────┘
                (agents return UP to the orchestrator — never sideways)
```

Three rules this enforces, all structural:

- **No sideways calls.** An agent returns *up* to the orchestrator, never
  *across* to a peer.
- **`BLOCKED:` is the refusal channel.** It's how an agent declines
  out-of-scope work instead of overstepping.
- **Handoffs are files, not memory.** If it isn't written to disk or pasted
  into the next prompt, the next agent never learns it.

## Making it follow your conventions

The agents are told to read specific files at the start of each run. To change
what they enforce, edit the source — no code, all Markdown:

- `standards/` — coding, API, database, security, testing, git conventions
- `templates/` — the output shapes (PRD, ADR, RFC, migration, postmortem, …)
- `.claude/agents/<name>.md` — an agent's system prompt, tools, and its
  `## Never` section (the highest-value part to tune)
- `.claude/skills/<name>/SKILL.md` — a workflow's step sequence and gates

After editing anything under `.claude/` while a session is open, run
`/reload-plugins` to pick up the changes.

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| Commands don't appear in `/help` | Plugin not loaded — check the install step, then `/reload-plugins` |
| Agents ignore your standards | `standards/` isn't in the project being worked on, or the agent's "Before you start" section was edited out |
| `/li-feature` blew past a gate | It should announce skips; if it didn't, tighten the gate wording in `.claude/skills/li-new-feature/SKILL.md` |
| A migration was written but not applied | Correct — agents never run migrations; apply it yourself |
| Review keeps looping | The convergence cap (three rounds) escalates to you by design; decide the disagreement |

## Where to go next

- [README](README.md) — the design rationale and the three orchestration
  mechanisms
- `standards/` — the rules the agents enforce
- `templates/` — the exact output shapes they produce

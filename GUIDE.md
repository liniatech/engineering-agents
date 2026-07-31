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
/help          # the five commands (/kickoff, /feature, /bugfix, /review, /spec) appear
/agents        # the six specialists appear
```

There is nothing to configure. The agents read `standards/` and `templates/`
from the repo they run in — so if you want them to follow *your* conventions,
keep those two directories in your project (or edit the copies shipped here).

## The fastest way to pick an entry point

| You want to… | Use | It will… |
|---|---|---|
| Start a greenfield project | `/kickoff <what>` | interview you, then PRD → SDD + ADR backbone → task list (docs only, no code) |
| Build something new | `/feature <what>` | spec → architecture → schema → build → review, with approval gates |
| Fix a defect | `/bugfix <the bug>` | reproduce → root cause → failing test → fix → verify |
| Check code before merge | `/review [PR/branch]` | run two review lenses, verify findings, report (no auto-fix) |
| Turn an idea into a PRD | `/spec <idea>` | write the PRD only — no design, no code |

You don't *have* to use the commands. You can describe the task in plain
language and Claude will match it to the right skill by its description. The
commands exist for when you want to be explicit and deterministic.

## Walkthroughs

### Start a greenfield project — `/kickoff`

```
/kickoff a service that ingests bank webhooks and reconciles them nightly
```

Use this at the very beginning of a **new** project — before there's code to
put a feature into. What happens, in order:

1. **Discovery interview** — Claude (the orchestrator, not an agent) asks you
   about the business and the problem, **one question at a time**: who has the
   problem, why now, what success looks like, constraints. It also proposes a
   `{SLUG}` for the repo and asks you to confirm it. Notes land in
   `docs/scoping/{SLUG}-discovery.md`. **You approve the scoping.**
2. **PRD** — `product-manager` turns the interview into `PRD-{SLUG}-001`.
   **You approve it.**
3. **Architecture drawn** — `architect` writes `ADR-{SLUG}-001` (the SDD, with
   C4 context→container→component diagrams) plus the backbone ADRs it owns;
   `database-engineer` writes the database ADR (schema *designed*, not
   migrated) and `qa` writes the testing ADR. **You approve the architecture.**
4. **Task breakdown** — the documented architecture is decomposed into a
   dependency-ordered task list in `docs/plan/{SLUG}-build-plan.md`, each task
   citing its ADR and acceptance criteria.

Then it **stops.** Kickoff produces docs only — no code. Build the tasks
afterward, one at a time, with `/feature`. See
[the numbering methodology](standards/adr-methodology.md) for the `ADR-{SLUG}-NNN`
convention.

### Build a feature — `/feature`

```
/feature add rate limiting to the public API, 100 req/min per API key
```

What happens, in order:

1. **Spec** — the `product-manager` agent writes a PRD (user stories,
   acceptance criteria, edge cases). **You approve it before anything else
   runs.** This gate is deliberate — the spec is cheap to change now and
   expensive to change after code exists.
2. **Architecture** — the `architect` agent produces an ADR with tradeoffs
   (where the counter lives, what happens on a store outage, the contract
   change). **You approve this gate too.**
3. **Schema** — if the change touches the database, the `database-engineer`
   writes a reversible migration. *Agents never run migrations* — you do.
4. **Build** — the `backend-engineer` implements it with tests.
5. **Review** — a *fresh* `reviewer` and `qa` agent check the diff. The
   review→fix loop is capped at three rounds, then it escalates to you.

Artifacts land in `docs/features/<slug>/` so each handoff is a real file, not
a fact trapped in one agent's memory.

**Skip a gate on purpose?** The command tells the orchestrator it must announce
any skipped gate and why — it won't silently bypass the spec or architecture
approval.

### Fix a bug — `/bugfix`

```
/bugfix users with a + in their email get a 500 on password reset
```

The pipeline is root-cause-first and will **refuse to patch a symptom**:

1. Reproduce it. If it can't, it stops and tells you what it needs.
2. Show you the root cause — before writing any fix.
3. Write a **failing test** that captures the bug.
4. Fix it until that test passes.
5. Verify and hand to review.

Give it repro steps if you have them; it works far better with them.

### Review code — `/review`

```
/review 482            # a PR number
/review my-branch      # a branch
/review                # blank → the working tree, or the branch vs its base
```

Two lenses run in parallel: `reviewer` (correctness, security, readability,
performance) and `qa` (does the test suite actually prove the spec). Findings
are **verified before they're reported**, and nothing is fixed without asking
you first.

### Spec only — `/spec`

```
/spec a digest email that summarizes a user's activity for the past week
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
| `project-kickoff` | `/kickoff` | "new project / start building X from scratch" | discovery interview → PRD → SDD + ADR backbone → task list; docs only |
| `new-feature` | `/feature` | "build / add …" | human gates after spec and architecture |
| `bug-fix` | `/bugfix` | "fix … / this is broken" | reproduce → root cause → **failing test** → fix |
| `code-review` | `/review` | "review this PR / is this ready to merge" | two lenses, findings verified before reported |
| `architecture-review` | — | "review this design / ADR / RFC" | boundaries & failure modes; catches over- *and* under-engineering |
| `database-change` | — | "add a column / write this migration / add an index" | expand/contract, lock analysis; **humans run migrations** |
| `dependency-upgrade` | — | "bump … / patch this CVE / upgrade the framework" | read every intermediate changelog, apply incrementally |
| `incident` | — | "prod is down / this alert is firing / outage" | **stabilize first**, diagnose second, then postmortem |
| `performance-review` | — | "this is slow / latency regressed / optimize …" | **no optimizing without a measurement** |
| `release` | — | "cut a release / prep the deploy / tag a version" | migration ordering + written abort criteria; humans deploy |

A few worth calling out because their guardrails change how they behave:

- **`incident`** optimizes for *restoring service*, not elegance. It will
  mitigate first (a revert, a flag flip) and only then diagnose. Reach for it
  when something is broken *right now*.
- **`database-change`** and **`release`** both stop short of the irreversible
  step: the migration gets written and the deploy gets verified, but a human
  runs them. That boundary is intentional.
- **`performance-review`** refuses to optimize without a profile, and
  **`dependency-upgrade`** refuses to skip intermediate changelogs — same
  root-cause-first spirit as `bug-fix`.

Examples:

```
review the design in docs/adr/0007-event-bus.md against our architecture principles
add a nullable `deleted_at` column to orders for soft deletes
upgrade Django from 4.2 to 5.0
checkout is timing out for carts with 50+ items — find out why
we're cutting the 2.4 release tomorrow, prep the checklist
prod payments are returning 500s, help
```

## Running agents by hand

The commands orchestrate agents for you, but you can drive a single specialist
directly when you want tight control or you're learning where each one is weak:

```
Use the architect agent to review the data flow in docs/features/rate-limit/
```

The six agents and what each owns (and refuses):

| Agent | Owns | Will not |
|---|---|---|
| `product-manager` | PRD, acceptance criteria, edge cases | design or code |
| `architect` | boundaries, contracts, ADRs | write code or design schema |
| `database-engineer` | schema, indexes, migrations | write app code or run migrations |
| `backend-engineer` | implementation + tests | change schema, infra, or requirements |
| `reviewer` | code quality, security, correctness | write code (it has no Edit/Write tool) |
| `qa` | spec↔code↔test coverage gaps | change production code |

These limits are structural, not polite requests — `reviewer` literally has no
write tools, so it cannot rubber-stamp by fixing.

## How agents communicate

They don't talk to each other directly — and that's the point. **No agent can
spawn or call another** (`backend-engineer.md`: *"You cannot spawn other
agents."*). All coordination goes through two things: a structured handoff
line each agent returns, and the orchestrating skill that reads it and routes.

**Each agent ends with a fixed `## Handoff` line — a vocabulary, not prose:**

| Agent | Returns |
|---|---|
| `product-manager` | `NEXT: architect \| BLOCKED: <reason>` |
| `architect` | `NEXT: database-engineer \| backend-engineer \| BLOCKED: <reason>` |
| `database-engineer` | `NEXT: backend-engineer \| BLOCKED: <reason>` |
| `backend-engineer` | `NEXT: reviewer \| BLOCKED: <reason>` |
| `qa` | `NEXT: backend-engineer \| done \| BLOCKED: <reason>` |
| `reviewer` | its structured findings; `BLOCKED: self-review` if it wrote the diff |

The orchestrating skill reads that line and decides what to spawn next. An
agent that hits a wall doesn't call a peer — it *stops* and returns a
`BLOCKED:` line explaining why, and the orchestrator routes it. That's how
`backend-engineer` asks for a schema change (`BLOCKED: needs schema change`),
and how `qa` refuses to edit code (`BLOCKED: implementation defect`).

**Why the indirection?** A subagent starts with an empty context window — it
cannot see your conversation, the orchestrator's reasoning, or what a previous
agent produced. So anything that must survive a handoff has to be
*materialized* (`new-feature/SKILL.md`):

> "Everything that must survive a handoff has to be either **written to a
> file** or **pasted into the next agent's prompt**."

That's why the pipeline writes real artifacts and passes their *paths* forward:

```
docs/features/<slug>/prd.md        ← product-manager writes it
docs/features/<slug>/adr.md        ← architect reads prd.md, writes this
docs/features/<slug>/migration.md  ← database-engineer
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
| `/feature` blew past a gate | It should announce skips; if it didn't, tighten the gate wording in `.claude/skills/new-feature/SKILL.md` |
| A migration was written but not applied | Correct — agents never run migrations; apply it yourself |
| Review keeps looping | The convergence cap (three rounds) escalates to you by design; decide the disagreement |

## Where to go next

- [README](README.md) — the design rationale and the three orchestration
  mechanisms
- `standards/` — the rules the agents enforce
- `templates/` — the exact output shapes they produce

# engineering-agents

A reusable engineering team for Claude Code: six specialist agents, ten
workflow pipelines, and the standards and templates they read.

New here and just want to use it? See the **[usage guide](GUIDE.md)**. This
README covers the design and rationale.

## What this repo is

Two layers, deliberately separated:

```
.claude/            ← the machine layer. Claude Code reads this.
  agents/           six specialists, each with its own fresh context
  skills/           nine orchestration procedures
  commands/         slash-command entry points

standards/          ← the human layer. People read it; agents are TOLD to.
templates/          output shapes both humans and agents conform to
```

`.claude/` is only live for work done **inside this repo**. To use the team
in another project, copy all three directories into it:

```
cp -r .claude standards templates /path/to/your-project/
```

Everything is plain Markdown. There is no build step and no runtime.

## Install as a plugin

Instead of copying directories, you can install this repo as a Claude Code
plugin. The manifest (`.claude-plugin/plugin.json`) points at the existing
`.claude/` directories, so the agents, skills, and commands load without any
files moving.

**Try it in one session (no install).** From the repo root:

```
claude --plugin-dir ./
```

This is single-session only — nothing is registered. Use it to test changes.

**Install it persistently** (for yourself or your team). This uses the
marketplace defined in `.claude-plugin/marketplace.json`:

```
/plugin marketplace add liniatech/engineering-agents
/plugin install engineering-agents@liniatech
```

`liniatech` is the marketplace name; `engineering-agents` is the plugin. After
installing, the six agents, nine skills, and four commands are available in any
project — no per-repo copy required.

## The three mechanisms

| | Subagent | Skill | Slash command |
|---|---|---|---|
| Lives in | `.claude/agents/x.md` | `.claude/skills/x/SKILL.md` | `.claude/commands/x.md` |
| Is a | worker with a **fresh context** | procedure loaded into the current context | prompt macro |
| Answers | *who* does this | *how* it is done | *start* it now |
| Sees your conversation | **No** | Yes | Yes |
| Invoked by | the orchestrator | Claude, when the description matches | you, typing `/x` |

## The rule that explains everything else

**A subagent starts with an empty context window.** It cannot see your
conversation, the orchestrator's reasoning, or what a previous agent learned.
It receives exactly three things:

1. Its system prompt — the body of its `.md` file
2. The task prompt it was spawned with
3. Whatever it reads from disk, **if told to**

Three consequences that drive the whole design:

- **Standards are not ambient.** Writing `standards/security.md` does nothing
  on its own. Every agent here has a `## Before you start` section naming the
  exact files to read. Remove that section and the standards go inert.
- **Handoffs must be materialized.** Anything crossing an agent boundary is
  written to a file or pasted into the next prompt. The pipelines write
  artifacts to `docs/features/<slug>/` for precisely this reason.
- **Agents cannot call each other.** They return a `BLOCKED:` line; the
  orchestrating skill routes it. That is why every agent has a "Handoff
  signals" section instead of "delegate to X".

## The agents

| Agent | Owns | Cannot |
|---|---|---|
| `product-manager` | PRD, acceptance criteria, edge cases | design or code |
| `architect` | boundaries, contracts, ADRs, tradeoffs | write code, design schema |
| `database-engineer` | schema, indexes, migrations | write app code, run migrations |
| `backend-engineer` | implementation and tests | schema, infra, requirements |
| `reviewer` | code quality, security, correctness | write code *(no Edit/Write tool)* |
| `qa` | spec↔code↔test coverage gaps | change production code |

The `tools:` field enforces these. `reviewer` has no `Edit` or `Write`, so
"never writes code" is structural, not a request it might rationalize past.

`reviewer` and `qa` are genuinely different lenses: `reviewer` judges the
code, `qa` judges whether the tests prove the spec.

## The workflows

| Skill | Use for | Core discipline |
|---|---|---|
| `project-kickoff` | starting a greenfield project | discovery interview → PRD → SDD + ADR backbone; ends at a task list |
| `new-feature` | building something new | human gates after spec and architecture |
| `bug-fix` | a defect | reproduce → root cause → **failing test** → fix |
| `code-review` | a diff or PR | two parallel lenses, findings verified before reported |
| `database-change` | schema work | expand/contract; humans run migrations |
| `architecture-review` | a design or ADR | catches over- *and* under-engineering |
| `dependency-upgrade` | version bumps | read every intermediate changelog |
| `incident` | production is down | **mitigate first**, diagnose second |
| `performance-review` | something is slow | no optimizing without a measurement |
| `release` | cutting a release | deploy ordering + written abort criteria |

## Commands

- `/kickoff <what to build>` — greenfield project: interview → PRD → ADRs → tasks
- `/feature <what to build>` — full pipeline with approval gates
- `/bugfix <the bug>` — root-cause-first fix
- `/review [PR or branch]` — multi-lens review
- `/spec <idea>` — PRD only, no code

You can also just describe the task; Claude matches the skill `description`
and runs it. The commands are for when you want to be explicit.

## Orchestration patterns, cheapest to most controlled

1. **Model-driven** — write good `description` fields and let Claude pick.
   Flexible, non-deterministic; it may skip a step you wanted.
2. **Skill-as-procedure** — what the ten skills here are. You write the
   sequence in prose; Claude executes it. Deterministic enough, editable,
   no code. **Start here.**
3. **Workflow scripts** — real JS with parallel fan-out, for fifty items.
4. **You** — run agents one at a time and read each output. Slowest, and by
   far the best way to learn where each agent is weak.

## Three rules worth internalizing

**Never self-review.** An agent reviewing its own output approves it. Every
review step here is a fresh spawn given only the diff.

**Cap the convergence loop.** `new-feature` stops at three review→fix
iterations and escalates. Without a cap, two agents will politely disagree
forever.

**Gate what is expensive to undo.** Spec and architecture have human
approval gates. Implementation does not — it is cheap to redo. Migrations
and deploys are never executed by an agent at all.

## Conventions when editing

- The `description` field is how the orchestrator *selects* an agent or
  skill. Write it for selection: when to use it, and when not to.
- Every agent needs an explicit output contract. Its final text is its entire
  return value — unspecified shape means the orchestrator gets an essay.
- The `## Never` sections are the highest-value part of an agent file. Keep
  them specific and short enough to be read.

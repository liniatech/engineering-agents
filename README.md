# engineering-agents

A reusable engineering team for Claude Code: six specialist agents, eleven
workflow pipelines, and the standards and templates they read.

New here and just want to use it? See the **[usage guide](GUIDE.md)**. This
README covers the design and rationale.

## What this repo is

Two layers, deliberately separated:

```
.claude/            ← the machine layer. Claude Code reads this.
  agents/           six specialists, each with its own fresh context
  skills/           eleven orchestration procedures
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

**Try it in one session (no install).** From the repo root:[GID-Identidade-Auth-Documentacao.md](../../../Library/Application%20Support/Claude/local-agent-mode-sessions/8aee9f55-93e5-48f9-b471-6ad5d435bfe6/b80d5763-6a16-4fd3-b18a-f19e99ee9960/local_3f6eee5d-7aae-41f2-93c6-eaf659810bbe/outputs/GID-Identidade-Auth-Documentacao.md)

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
installing, the six agents, eleven skills, and eight commands are available in any
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
- **Every deliverable is a file.** Reports, analyses, tickets, and reviews are
  written to `docs/` *before* being summarized in chat — the summary is a
  pointer, never the deliverable. `standards/deliverables.md` defines the paths
  and the rule; every pipeline names its own output file.
- **Agents cannot call each other.** They return a `BLOCKED:` line; the
  orchestrating skill routes it. That is why every agent has a "Handoff
  signals" section instead of "delegate to X".

## The agents

| Agent | Owns | Cannot |
|---|---|---|
| `li-product-manager` | PRD, acceptance criteria, edge cases | design or code |
| `li-architect` | boundaries, contracts, ADRs, tradeoffs | write code, design schema |
| `li-database-engineer` | schema, indexes, migrations | write app code, run migrations |
| `li-backend-engineer` | implementation and tests | schema, infra, requirements |
| `li-reviewer` | code quality, security, correctness | write code *(no Edit/Write tool)* |
| `li-qa` | spec↔code↔test coverage gaps | change production code |

The `tools:` field enforces these. `li-reviewer` has no `Edit` or `Write`, so
"never writes code" is structural, not a request it might rationalize past.

`li-reviewer` and `li-qa` are genuinely different lenses: `li-reviewer` judges the
code, `li-qa` judges whether the tests prove the spec.

## The workflows

| Skill | Use for | Core discipline |
|---|---|---|
| `li-project-kickoff` | starting a greenfield project | discovery interview → PRD → SDD + ADR backbone; ends at a task list |
| `li-new-feature` | building something new | human gates after spec and architecture |
| `li-bug-fix` | a defect | hypothesis elimination → blast radius → **failing test** → ticket → gate → fix |
| `li-test-repair` | a red suite | verdict per failure; **only test files are edited** |
| `li-code-review` | a diff or PR | two parallel lenses, findings verified before reported |
| `li-database-change` | schema work | expand/contract; humans run migrations |
| `li-architecture-review` | a design or ADR | catches over- *and* under-engineering |
| `li-dependency-upgrade` | version bumps | read every intermediate changelog |
| `li-incident` | production is down | **mitigate first**, diagnose second |
| `li-performance-review` | something is slow | no optimizing without a measurement |
| `li-release` | cutting a release | deploy ordering + written abort criteria |

## Commands

- `/li-kickoff <what to build>` — greenfield project: interview → PRD → ADRs → tasks
- `/li-feature <what to build>` — full pipeline with approval gates
- `/li-bugfix <the bug>` — root-cause-first fix, diagnosis gated
- `/li-diagnose <the bug>` — deep diagnosis + repro test + bug ticket, no fix
- `/li-fixtests [filter]` — repair a red suite; stale tests only, never source
- `/li-review [PR or branch]` — multi-lens review
- `/li-spec <idea>` — PRD only, no code
- `/li-demo [brief]` — self-test: run kickoff on a brief (default the Roomie one) and regenerate the example

You can also just describe the task; Claude matches the skill `description`
and runs it. The commands are for when you want to be explicit.

## Orchestration patterns, cheapest to most controlled

1. **Model-driven** — write good `description` fields and let Claude pick.
   Flexible, non-deterministic; it may skip a step you wanted.
2. **Skill-as-procedure** — what the eleven skills here are. You write the
   sequence in prose; Claude executes it. Deterministic enough, editable,
   no code. **Start here.**
3. **Workflow scripts** — real JS with parallel fan-out, for fifty items.
4. **You** — run agents one at a time and read each output. Slowest, and by
   far the best way to learn where each agent is weak.

## Three rules worth internalizing

**Never self-review.** An agent reviewing its own output approves it. Every
review step here is a fresh spawn given only the diff.

**Cap the convergence loop.** `li-new-feature` stops at three review→fix
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

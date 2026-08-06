---
name: li-new-feature
description: Full pipeline for shipping a new feature — spec, architecture, schema, implementation, review, QA. Use when the user asks to build a feature, add functionality, or turn an idea into shipped code. Not for bug fixes (use li-bug-fix) or pure refactors.
---

# New feature pipeline

You are the orchestrator. You spawn agents, carry artifacts between them, and
hold the gates. You do not do their work yourself.

## Artifact rule

Subagents share no context. Everything that must survive a handoff has to be
either **written to a file** or **pasted into the next agent's prompt**.
Prefer files for anything longer than a paragraph:

```
docs/features/<slug>/prd.md
docs/features/<slug>/adr.md
docs/features/<slug>/migration.md
docs/features/<slug>/review.md      ← step 6's findings
```

Pass the *paths* to downstream agents and tell them to read them.

Read `standards/deliverables.md`. Every one of these is created with the
`Write` tool at the step that produces it, before the gate for that step is
shown. An artifact that only appeared in chat cannot be read by the next
agent — which is the entire reason this rule exists here.

## Steps

### 1. Scope check

If the request is a one-line change with obvious intent, say so and skip to
step 5. Running six agents on a copy tweak is theatre. Tell the user you are
skipping the pipeline and why.

### 2. Specification

Spawn `li-product-manager` with the raw request plus any context the user gave.

Write the returned PRD to `docs/features/<slug>/prd.md`.

**GATE — human.** Show the user the Summary and the Open questions. Ask them
to approve or correct. Do not proceed on unanswered `[NEEDS INPUT]` markers
that affect scope.

### 3. Architecture

Skip this step only if the change is confined to one existing module, adds no
dependency, and changes no contract. Say so if you skip it.

Spawn `li-architect` with the PRD path. Write the ADR to
`docs/features/<slug>/adr.md`.

**GATE — human.** Show the Recommendation and Risks. Architecture is the
expensive thing to undo; get a human yes.

### 4. Schema

If the ADR implies any schema change, spawn `li-database-engineer` with the PRD
and ADR paths. Write the migration doc to `docs/features/<slug>/migration.md`.

Never let `li-backend-engineer` write a migration. If it returns
`BLOCKED: needs schema change`, come back here.

### 5. Implementation

Spawn `li-backend-engineer` with the paths to every artifact produced so far and
a clear statement of the acceptance criteria it must satisfy.

If it returns BLOCKED, route by the reason:
- `needs schema change` → step 4, then re-spawn
- `spec ambiguity` / `spec conflict` → step 2, or ask the user directly
- `architecture conflict` → step 3
- `needs infrastructure` → stop and escalate to the user

### 6. Review — two lenses, fresh spawns

Spawn `li-reviewer` and `li-qa` **in parallel**, each as a new agent. Never send
this to the agent that wrote the code; a self-review is worthless.

Give each only what it needs: the base branch to diff against, and the PRD
path for the acceptance criteria.

Write their combined findings to `docs/features/<slug>/review.md` using
`templates/code-review.md` — before you act on them. Step 7 loops against that
file, and it is the record of what was accepted rather than fixed.

### 7. Converge

If `li-reviewer` returns Critical findings or `li-qa` returns gaps:

- Spawn a **fresh** `li-backend-engineer` with the specific findings. Do not
  reuse the earlier one — it will defend its choices.
- Re-review with a fresh `li-reviewer`.
- **Maximum 3 iterations.** If findings survive three rounds, stop and
  escalate to the user with the disagreement stated plainly. Looping past
  three means the agents are stuck, not converging.

Suggestions do not block. Report them; let the human decide.

### 8. Report

Summarize for the user:

- What shipped, and the paths to the artifacts
- Test results — the actual output, not "tests pass"
- Findings that were accepted rather than fixed, and why
- Anything you skipped, and why

Do not commit or push. That is the user's call.

## Failure honesty

If a step failed, say it failed. If you skipped a gate, say so. A pipeline
report that hides a skipped step is worse than no report.

If you skipped an artifact — no ADR because the change was confined to one
module, no migration because the schema did not move — say which file does not
exist and why. Silence reads as "it's there".

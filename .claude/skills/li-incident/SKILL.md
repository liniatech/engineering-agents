---
name: li-incident
description: Live production incident response — stabilize first, diagnose second, then postmortem. Use when something is broken in production, an alert is firing, or the user says there is an outage. Optimizes for restoring service, not for elegance.
---

# Incident response

**Mitigate first. Understand later.** The normal engineering instinct — find
the root cause before changing anything — is wrong during an incident. Users
are affected now.

Everything below assumes a human is in the loop and taking the actions. You
are assisting, not operating.

## Deliverable — a file, always

This skill produces **`docs/incidents/<date>-<slug>.md`**, using
`templates/postmortem.md`.

Read `standards/deliverables.md`. One exception to the write-first rule lives
here: **during phase 1, mitigation comes before documentation.** Keep a running
timeline as you go, and write the file the moment service is stable — the
timeline is unreconstructable an hour later.

Phases 2 and 3 follow the normal rule: file first, chat summary second.

## Phase 1 — Stabilize

### 1. Establish impact

- What is broken, from a user's point of view?
- How many users, which ones?
- When did it start? (This is the highest-value question — it points at the
  change that caused it.)
- Is it getting worse?

### 2. Look at the recent changes

Most incidents are caused by a change. Before theorizing:

```
git log --oneline --since="24 hours ago"
```

Recent deploys, migrations, config changes, feature-flag flips. Also check
whether a *dependency* changed — an upstream provider, a certificate, a quota.

### 3. Mitigate

Fastest safe path back to working. In rough order of preference:

1. Roll back the suspect change
2. Flip the feature flag off
3. Scale up / shed load / raise the limit
4. Targeted hotfix — only when 1–3 genuinely do not apply

A rollback that you are 70% sure about beats a diagnosis you are 100% sure
about an hour from now.

**Never take a mitigating action yourself.** State exactly what you recommend
and what it will do; a human executes it.

### 4. Confirm recovery

Verify with a signal, not a hope. Which metric, which log line, which
request — name it and check it. State plainly if the recovery is partial.

**Now write the file.** `mkdir -p docs/incidents`, `date +%F`, then use the
`Write` tool to create
`docs/incidents/<date>-<slug>.md` from `templates/postmortem.md` with impact,
the timeline so far (real timestamps, including detection and mitigation), and
the mitigating action taken. `status: draft`. Phases 2 and 3 fill the rest.

## Phase 2 — Diagnose

Only once service is restored.

Now do it properly: reproduce in a safe environment, find the root cause,
write the failing test. Use the `li-bug-fix` skill from here — the discipline is
the same, the urgency is gone.

If the mitigation was a rollback, the underlying bug is still there. Say so
explicitly; a rolled-back incident is not a closed incident.

## Phase 3 — Postmortem

Finish `docs/incidents/<date>-<slug>.md` while memory is fresh — the same file
started in phase 1, now completed against `templates/postmortem.md` and set to
`status: resolved`. Report the path in chat; do not paste the postmortem there
instead of writing it.

The rules that make a postmortem worth writing:

- **Blameless.** "The deploy did not have a canary stage" — not "X deployed
  without checking." Every human error is a system that permitted it.
- **Timeline with real timestamps** — including detection time and
  mitigation time. Time-to-detect is usually the most actionable number.
- **Root cause, not proximate cause.** "The query was slow" is proximate.
  "There is no query plan review, so an unindexed query reached production"
  is root.
- **Action items with owners.** Unowned action items do not happen.
- **What went well.** Not decoration — it tells you which investments paid
  off and are worth repeating.

## During an incident, never

- Never take a production action yourself — recommend, and let a human act.
- Never make multiple changes at once; you will not know which one worked.
- Never skip writing down what you did, even under pressure. The timeline
  is unreconstructable afterwards.
- Never let the search for a root cause delay the mitigation.
- Never close an incident without `docs/incidents/<date>-<slug>.md` on disk —
  including one mitigated by a rollback, which is not a closed incident.

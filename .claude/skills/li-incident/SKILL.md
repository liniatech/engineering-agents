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

## Phase 2 — Diagnose

Only once service is restored.

Now do it properly: reproduce in a safe environment, find the root cause,
write the failing test. Use the `li-bug-fix` skill from here — the discipline is
the same, the urgency is gone.

If the mitigation was a rollback, the underlying bug is still there. Say so
explicitly; a rolled-back incident is not a closed incident.

## Phase 3 — Postmortem

Write it while memory is fresh. Use `templates/postmortem.md`.

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

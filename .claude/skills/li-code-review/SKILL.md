---
name: li-code-review
description: Multi-lens review of a diff, PR, or working tree — code quality plus test coverage, with findings verified before they are reported. Use when the user asks to review code, check a PR, or asks whether a change is ready to merge.
---

# Code review

Two lenses, run in parallel, then verified. The verification step exists
because a confident review agent will produce plausible findings that are
simply wrong, and an unverified false finding costs more trust than a missed
real one.

## Steps

### 1. Establish the diff

Determine what is under review and get it concretely:

```
git diff main...HEAD          # branch vs base
git diff                      # uncommitted working tree
gh pr diff <n>                # a specific PR
```

If the diff is empty, stop and say so.

Note the size. Over ~800 changed lines, review in coherent chunks and say
that you did — a single pass over a huge diff degrades badly.

### 2. Find the intent

Look for the PRD, the ADR, the linked issue, or the PR description. A review
without knowing what the change was *supposed* to do can only catch local
defects, not wrong ones. If you cannot find intent, say so in the report.

### 3. Two lenses, parallel, fresh

Spawn both at once, each as a new agent with no prior context:

- `li-reviewer` — correctness, security, performance, maintainability
- `li-qa` — spec-to-code-to-test coverage gaps

Neither may be the agent that wrote the code.

### 4. Verify the Critical findings

For each Critical finding, before reporting it, confirm it yourself:

- Read the cited `file:line`. Does the code actually say that?
- Does the described failure input genuinely produce the described result?
- Is there existing handling elsewhere that the finding missed?

Drop findings that do not survive. Say how many you dropped — that number is
useful signal about the review's precision.

For a deep audit, spawn independent skeptics instead: for each Critical
finding, spawn 2–3 agents prompted to **refute** it, and keep the finding
only if a majority fail to refute. Use this when the stakes justify the cost.

### 5. Report

```
## Verdict
approve | request-changes | block

## Scope
What was reviewed (diff range, N files, N lines). What was NOT reviewed.

## Critical      (blocks merge)
file:line — defect, trigger, consequence.

## Warnings      (should fix)
## Suggestions   (optional)
## Coverage gaps (from li-qa)

## Verification
N findings raised, M dropped as unconfirmed.
```

### 6. Offer, do not apply

Ask before changing anything. Review and fix are separate acts, and the
author gets to decide.

## Rules

- Every finding needs `file:line` and a concrete failure scenario. No
  scenario, no finding.
- "Approve" is a real outcome. Do not invent findings to justify the review.
- Never re-litigate an accepted ADR. Implementation-contradicts-ADR is a
  finding; disagreeing-with-the-ADR is a separate conversation.

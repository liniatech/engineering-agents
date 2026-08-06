---
title: Code review — <subject>
type: code-review
date: <YYYY-MM-DD>
status: approved | changes-requested | blocked
scope: <diff range, N files, N lines>
---

# Code Review — <subject>

## Verdict

approve | request-changes | block — one sentence saying why.

## Scope

- **Diff:** `<command that produced it>` — N files, N lines
- **Intent:** the PRD / ADR / issue this change was supposed to satisfy, with
  a path. If none was found, say so — a review without intent catches local
  defects, not wrong ones.
- **Not reviewed:** what was deliberately left out, and why.

## Critical — blocks merge

Each finding: `file:line` — the defect, the input or state that triggers it,
and the consequence. No concrete failure scenario, no finding.

### 1. <finding>

- **Where:** `file:line`
- **Trigger:**
- **Consequence:**
- **Suggested direction:**

## Warnings — should fix

| # | Where | Finding | Why it matters |
|---|---|---|---|

## Suggestions — optional

| # | Where | Suggestion |
|---|---|---|

## Coverage gaps

From the QA lens: behavior in the diff that no test exercises, and documented
behavior that is not asserted anywhere.

| Behavior | Covered? | Missing case |
|---|---|---|

## Verification

N findings raised, M dropped as unconfirmed after checking the cited code.
That ratio is signal about this review's precision — record it honestly.

## Not applied

Review and fix are separate acts. Nothing here has been changed; the author
decides.

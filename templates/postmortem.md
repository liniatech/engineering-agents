# Postmortem

**Incident:**
**Date:**
**Duration:** detection → mitigation
**Severity:**
**Author:**

> This document is **blameless**. Describe systems that permitted an error,
> not people who made one. "The deploy pipeline had no canary stage" — not
> "X deployed without checking."

## Summary

Three sentences. What broke, who was affected, how it was resolved.

## Impact

- Users affected: how many, which segment
- Duration of user-visible impact:
- Data loss or corruption: yes / no — detail
- Revenue or SLA impact:

## Timeline

All times in one timezone; state which.

| Time | Event |
|---|---|
| | Change deployed / condition began |
| | First user impact |
| | **Detected** — by what: alert, user report, manual |
| | Investigation started |
| | Cause identified |
| | **Mitigation applied** |
| | Recovery confirmed |
| | Incident closed |

**Time to detect:**
**Time to mitigate:**

Time-to-detect is usually the most actionable number in this document.

## Root cause

Not the proximate cause. Keep asking "why" until you reach something
systemic.

- **Proximate:** what failed technically
- **Root:** the system, process, or absent guardrail that allowed it

## Contributing factors

Conditions that made it worse or slower to resolve.

## Detection

How was it found? Should an alert have caught it sooner? If a human found it
first, that is a detection gap — record it as an action item.

## Resolution

What was done to mitigate. Whether the underlying defect is fixed or still
present behind a rollback.

**Underlying bug fixed:** yes / no — if no, link the tracking issue.

## What went well

Not decoration. This tells you which past investments paid off.

## What went poorly

## Where we got lucky

The near-misses. These are your next incident if left unaddressed.

## Action items

Every item has an owner and a date. Unowned items do not happen.

| # | Action | Type | Owner | Due | Ticket |
|---|---|---|---|---|---|
| 1 | | prevent / detect / mitigate | | | |

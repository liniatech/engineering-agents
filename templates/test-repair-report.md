---
title: Test repair — <suite or subject>
type: test-repair
date: <YYYY-MM-DD>
status: draft | resolved
scope: <test command, commit>
---

# Test Repair — <suite or subject>

## Baseline

| | |
|---|---|
| Command | `<the project's real test command>` |
| Commit | `<short-sha>` |
| Result | N passed, M failed |
| Last known green | `<commit or tag>`, or "not established" |

Raw output:

```
```

## Failure groups

Grouped by apparent shared cause — same exception, module, fixture, or
assertion shape. Twelve failures are usually two causes.

| Test(s) | Verdict | Likely commit | Action |
|---|---|---|---|
| | STALE / REAL BUG / FLAKE | likely `<sha>` — "<subject>" | |

**Attribution is correlation, not proof.** `git log` shows a commit touched
the code, never that it broke the test. Write "likely".

## What the culprit changed

In prose: what the likely commit did, and why it landed the tests where it
did.

## Evidence per verdict

`STALE` requires *positive* proof the behavior change was intended — the
commit message, the diff, an ADR/PRD, or the user saying so. Unsure is
`REAL BUG`.

| Group | Verdict | Evidence |
|---|---|---|

## Repaired — `STALE` only

| Test | Old contract asserted | New contract asserted |
|---|---|---|

Each repaired test must still fail against the old behavior. If it does not,
it was defused, not repaired.

## Added

Complementary coverage for the new behavior — edge cases, error paths,
boundaries that no test now exercises.

## Still red

| Test | Verdict | Handed to |
|---|---|---|
| | REAL BUG | `/li-bugfix` |
| | FLAKE | quarantined — reason |

A `REAL BUG` row is out of scope for a test repair. It stays red.

## After

```
```

| | Before | After |
|---|---|---|
| Passed | | |
| Failed | | |

Every test that changed state, in either direction. **A test that went from
passing to failing is a stop.**

## Not run

Anything skipped, and why.

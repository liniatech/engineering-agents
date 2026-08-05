---
name: li-test-repair
description: Repair a red test suite — run everything, group the failures, attribute each group to the likely commit, then fix only the tests that are genuinely stale. Use when the suite went red after a merge, rebase, or dependency bump, or when the user asks to fix failing tests. Only ever edits test files; a real regression is reported and handed to li-bug-fix, never made green.
---

# Test suite repair

A red test means one of two things: **the code broke**, or **the test is
stale.** These need opposite fixes, and getting the direction wrong is how a
regression ships behind a green suite.

So this skill has one hard boundary: **it edits test files and nothing else.**
If the fix belongs in source, it says so and stops.

## Before you start

Read `standards/testing-standards.md` — repaired and added tests conform to it.

Find the project's real test command; do not guess it. Look in the `Makefile`,
`package.json` scripts, `pyproject.toml`, `tox.ini`, or the CI workflow. If you
cannot find one, ask.

## Steps

### 1. Run the whole suite

Run it. Paste the real output — counts, names, and the actual failure text.

Record the command you ran and the commit you ran it on (`git rev-parse --short
HEAD`). Everything downstream is compared against this baseline.

If the suite **does not run at all** — import error, missing dependency,
collection failure — that is not a test failure. Stop and report it as an
environment problem. Do not start editing tests to get collection working.

### 2. Inventory and group

List every failure. Then group them by **apparent shared cause** — the same
exception, the same module, the same fixture, the same assertion shape.

Twelve failures are usually two causes. Group first so attribution and verdicts
run once per group, not twelve times.

### 3. Attribute — `git log`, and say so honestly

Find the last commit where the suite was known green, if that is cheap to
establish (CI history, a release tag, the user telling you). If it is not
cheap, skip it — do not go hunting.

```
git log --oneline -20 -- <paths under test>   # who touched this code recently
git log -S'<symbol>' --oneline                # when this symbol's usage changed
git blame <file> -L <start>,<end>             # who last touched the failing line
```

**This is correlation, not proof.** `git log` tells you a commit touched the
code, never that it broke the test. Write it that way:

- Good: `likely a1b2c3d — "normalize emails before lookup", touched auth/reset.py`
- Bad: `a1b2c3d broke test_password_reset`

If you want proof, `git bisect run <test cmd>` gives it — but it checks out old
commits, so **ask the user before running it** and only when the correlation is
genuinely ambiguous.

### 4. Verdict per group — evidence required

Classify every group as exactly one of:

| Verdict | Means | Evidence needed |
|---|---|---|
| `STALE TEST` | Behavior changed **on purpose**; the test still asserts the old contract | Positive proof the change was intended — the commit message, the diff, an ADR/PRD, or the user saying so |
| `REAL BUG` | The code's new behavior contradicts documented or intended behavior | The spec, ADR, or the test's own stated intent |
| `FLAKE / ENV` | Fails nondeterministically, or for a reason unrelated to the code — ordering, clock, timezone, network, a missing env var | Passes on re-run, or the failure names an external cause |

Two rules on this table:

- **Unsure means `REAL BUG`.** Never default to `STALE`. Being wrong toward
  `REAL BUG` costs a conversation; being wrong toward `STALE` ships a
  regression behind a green suite.
- `STALE` needs *positive* evidence the change was deliberate. "The commit
  changed this behavior" is not evidence it *meant* to — that sentence
  describes a regression just as well.

### 5. Report and ask — before touching anything

**GATE — human.** Nothing has been edited yet. Show the table:

```
Suite:    <command> on <short-sha> — N passed, M failed
Baseline: <last known-green commit, or "not established">

| Test(s)              | Verdict   | Likely commit                    | Proposed action        |
|----------------------|-----------|----------------------------------|------------------------|
| test_a, test_b       | STALE     | likely a1b2c3d — "<subject>"     | update to new contract |
| test_c               | REAL BUG  | likely d4e5f6a — "<subject>"     | hand to li-bug-fix     |
| test_d               | FLAKE     | —                                | quarantine, report     |
```

State plainly, in prose, what the likely culprit commit changed and why it
landed the tests where it did. Then ask which groups to repair.

**Do not proceed on the `REAL BUG` rows even if the user says "fix everything."**
Those are out of scope here — say so and point at `/li-bugfix`.

### 6. Repair — `STALE` only

For each approved `STALE` group, update the test to assert the **new intended
behavior**, precisely. The repaired test must still fail against the old
behavior — otherwise you have not repaired it, you have defused it.

Forbidden, without exception:

- Deleting a test, or the assertion that fails
- `skip`, `xfail`, `pytest.mark.skip`, `it.skip`, commenting it out
- Loosening an assertion to something the old *and* new behavior both satisfy
  — `assertTrue(x)`, a regex widened to `.*`, an exact match turned into `in`
- Editing any file that is not a test

If a `STALE` repair seems to require a source change, it was not `STALE`.
Re-classify it and go back to the gate.

### 7. Complementary coverage

The behavior that changed is often now under-tested — the old test covered the
old contract, and the repaired one covers the happy path of the new one.

Spawn `li-qa` with the repaired test paths and the culprit commit's diff. Ask it
for the **gap**: edge cases, error paths, and boundaries of the new behavior
that no test now exercises. Have it add those cases.

This is additive only. `li-qa` may add tests; it may not relax the ones you just
repaired.

### 8. Verify

- Run the full suite again. Paste the real output.
- Compare against the step 1 baseline: counts before and after, and every test
  that changed state in either direction.
- **A test that went from passing to failing is a stop.** You broke something.
- `REAL BUG` and quarantined `FLAKE` tests are still red. That is correct — do
  not present the suite as green.

### 9. Report

```
Suite before: N passed, M failed  (<command> on <short-sha>)
Suite after:  N passed, M failed

Repaired:     <tests, and the contract they now assert>
Added:        <complementary tests from li-qa>
Still red:    <REAL BUG tests + the likely commit, handed to li-bug-fix>
Quarantined:  <FLAKE tests, and why>
Not run:      <anything skipped, and why>
```

Do not commit or push. That is the user's call.

## Never

- **Never edit a source file.** If the fix belongs in source, stop and hand it
  to `li-bug-fix`. That is the whole boundary of this skill.
- Never delete, skip, or `xfail` a test to make the suite green.
- Never weaken an assertion so the new behavior slips under it.
- Never classify a group `STALE` without positive evidence the change was
  intended. Unsure is `REAL BUG`.
- Never write that a commit "broke" a test when all you ran was `git log`.
  Write "likely".
- Never repair before the gate.
- Never report the suite as green while `REAL BUG` failures are outstanding.

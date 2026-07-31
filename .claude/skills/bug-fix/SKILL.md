---
name: bug-fix
description: Root-cause-first pipeline for fixing a defect — reproduce, diagnose, failing test, fix, verify, review. Use when the user reports a bug, a test is failing, or behavior is wrong. Refuses to patch symptoms without a root cause.
---

# Bug fix pipeline

The discipline here is simple and almost always skipped: **you do not write a
fix until you can reproduce the bug and name its cause.**

## Steps

### 1. Reproduce — before anything else

Establish the exact conditions that trigger it. Write them down:

- Input / state that triggers it
- Expected behavior (cite the spec or test if one exists)
- Actual behavior, with the real error output

If you cannot reproduce it, **stop**. Report what you tried and ask the user
for what is missing. A fix for a bug you never saw is a guess with a commit
message.

### 2. Root cause

Investigate before theorizing. Read the code path. Check recent changes to
the involved files (`git log -p --follow <file>`).

State the cause as a single sentence: *"X happens because Y, at file:line."*

If you have two competing theories, find the evidence that eliminates one.
Do not fix both.

**GATE — human.** For anything non-trivial, tell the user the root cause
before you fix it. Root cause is where they most often know something you
cannot see from the code.

### 3. Failing test first

Write a test that fails **because of this bug** and would pass once fixed.
Run it. Confirm it fails, and fails for the right reason.

This is the step that proves you understood the bug. If you cannot write a
test that fails, you have not found the cause yet — go back to step 2.

If the bug is genuinely untestable (a UI rendering artifact, a
race that will not reproduce deterministically), say so explicitly and
describe how you will verify manually instead.

### 4. Fix

Spawn `backend-engineer` with the root cause and the failing test.

Instruct it: **fix the cause, not the symptom, and change nothing else.** A
bug fix diff should be small. If it is large, the cause was probably
misdiagnosed or the fix is a refactor in disguise.

### 5. Verify

- The new test passes.
- The full suite still passes — run it, paste the real output.
- The original reproduction no longer reproduces.

### 6. Review

Spawn a fresh `reviewer` with the diff. For a bug fix specifically, ask it
to check: does the fix address the stated root cause, and could the same
cause produce other bugs elsewhere in the codebase?

### 7. Report

```
Root cause:   <one sentence, with file:line>
Fix:          <what changed>
Test:         <the test that now guards it>
Verification: <actual command output>
Related risk: <other places the same cause may lurk, or "none found">
```

## Never

- Never fix a symptom because the cause is hard to find. Say it is hard.
- Never widen the fix into a refactor. Note the refactor; do not do it.
- Never remove or weaken a failing test to make the suite green.
- Never report "fixed" without having re-run the reproduction.

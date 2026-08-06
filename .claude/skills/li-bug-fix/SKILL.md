---
name: li-bug-fix
description: Root-cause-first pipeline for a defect in behavior — reproduce, diagnose with competing hypotheses, map the blast radius, write a failing test, emit a bug ticket, then fix and verify. Use when the user reports a bug, pastes an error message, or behavior is wrong. Diagnosis is gated and can be the whole deliverable. For a suite that went red after a merge or upgrade, where the tests themselves may be stale, use li-test-repair. Refuses to patch symptoms without a root cause.
---

# Bug fix pipeline

The discipline here is simple and almost always skipped: **you do not write a
fix until you can reproduce the bug and name its cause.**

Two phases, with a human gate between them. Phase 1 produces a written
diagnosis that stands on its own — it is a deliverable, not a warm-up.

## Deliverable — a file, always

This skill produces **`docs/bugs/<slug>.md`**. That file is the deliverable.

Read `standards/deliverables.md`. In short: the ticket is written to disk with
the `Write` tool **before** anything is printed in chat, and the chat output is
a short pointer to it. A ticket that only appeared in the terminal did not
happen.

## Before you start

Read `templates/bug-report.md` — phase 1 writes that shape. Read
`standards/testing-standards.md` before writing the failing test.

## Input modes

| You were given | Do this |
|---|---|
| A description, or just an error message | Both phases |
| `/li-diagnose` was used | Phase 1 only — stop at the gate |
| A path to an existing bug ticket | Re-confirm the repro, then phase 2 |

A short error message is a normal starting point, not a reason to ask for more
before you begin. Investigate first; ask only for what investigation cannot
recover.

---

# Phase 1 — Diagnose

### 1. Reproduce — before anything else

Establish the exact conditions that trigger it. Write them down:

- Input / state that triggers it
- Expected behavior (cite the spec or test if one exists)
- Actual behavior, with the real error output

If you cannot reproduce it, **stop**. Report what you tried and ask the user
for what is missing. A fix for a bug you never saw is a guess with a commit
message.

If it reproduces only sometimes, say how often and under what conditions. An
intermittent repro is still a repro; a *guessed* repro is not.

### 2. Root cause — by elimination, not by hunch

Investigate before theorizing. Read the whole code path, not just the line in
the stack trace. Find when the behavior changed:

```
git log -p --follow <file>      # history of the file
git log -S'<symbol>' --oneline  # when this symbol's usage changed
```

Then work by elimination:

- **Write down at least two candidate hypotheses.** One hypothesis is a hunch,
  not a diagnosis. If only one occurs to you, you have not read enough of the
  path yet.
- For each, name the **observation that would refute it** — then go get that
  observation. Read the code, add a probe, check the data.
- **Kill hypotheses with evidence, never with plausibility.** Record the
  refuting evidence for each one. It goes in the ticket, and it is what stops a
  dead theory being resurrected next week.
- If two survive, you are not done. Find the observation that separates them.
  Do not fix both.

State the survivor as a single sentence: *"X happens because Y, at file:line."*

If you cannot cite a `file:line`, you have a symptom, not a cause. Say so
plainly rather than dressing up the best guess.

### 3. Blast radius

The same cause usually has siblings. Find them **now**, while the cause is
fresh and nothing has been patched:

- Other call sites of the faulty function or branch
- Other places the same assumption goes unchecked
- Other consumers of the same data shape or contract

List each as `file:line` with a verdict: **affected**, **not affected (why)**,
or **needs checking**. "None found" is a valid answer — but only after looking,
and say what you searched for.

This is the cheapest five minutes in the pipeline. After the fix ships, the
same search costs an incident.

### 4. Failing test first

Write a test that fails **because of this bug** and would pass once fixed. Run
it. Confirm it fails, and fails for the right reason — a test that fails
because of a typo in the test proves nothing.

This is the step that proves you understood the bug. If you cannot write a test
that fails, you have not found the cause yet — go back to step 2.

Add a case for each **affected** entry from step 3 that is genuinely a distinct
failure, not a rerun of the same one.

If the bug is genuinely untestable (a UI rendering artifact, a race that will
not reproduce deterministically), say so explicitly and describe how you will
verify manually instead.

### 5. Write the ticket — to a file, with the Write tool

**This step is a file operation, not a piece of prose.** Do it now, before you
say anything to the user:

1. `mkdir -p docs/bugs` — create it if it does not exist.
2. `date +%F` — for the `date:` field. Never guess it.
3. **Call the `Write` tool** to create `docs/bugs/<slug>.md`, filled from
   `templates/bug-report.md`, with the frontmatter from
   `standards/deliverables.md` (`type: bug`, `status: open`).

`<slug>` is short, lowercase, hyphenated, and names the failure:
`webhook-drops-events-missing-user-id`.

Printing the ticket in chat does **not** satisfy this step. If the `Write` tool
has not run, the ticket does not exist and phase 1 is not done.

**Task title** — one line, imperative, naming the observable failure and the
surface it happens on. It has to survive being pasted into a tracker with no
other context.

- Good: `Webhook handler drops events when the payload omits user_id`
- Bad: `KeyError in webhooks` — that is the symptom, not the task
- Bad: `Fix webhook bug` — says nothing

Fill every section you have evidence for. Mark what you do not know
`[UNKNOWN]`; never fill a gap with a plausible guess. Include the refuted
hypotheses from step 2 and the blast radius from step 3 — a ticket that shows
what the bug *isn't* is worth more than one that only asserts what it is.

**GATE — human.** The file exists by now. Show the user, and nothing more:

```
Title:        <the task title>
Root cause:   <one sentence, with file:line>
Ruled out:    <each refuted hypothesis + the evidence that killed it>
Blast radius: <file:line list with verdicts, or "none found (searched: …)">
Failing test: <path, and the confirmed failure output>
Ticket:       docs/bugs/<slug>.md
```

Root cause is where the user most often knows something you cannot see from the
code. Get their yes before touching production code.

**If phase 1 was the request (`/li-diagnose`), stop here.** Report the ticket
path. Do not offer to fix it and do not start fixing it.

---

# Phase 2 — Fix

### 6. Fix

Spawn `li-backend-engineer` with the ticket path, the root cause, and the
failing test.

Instruct it: **fix the cause, not the symptom, and change nothing else.** A bug
fix diff should be small. If it is large, the cause was probably misdiagnosed
or the fix is a refactor in disguise.

Blast-radius siblings marked **affected** are in scope. Anything else is a
separate ticket — write it down, do not fold it in.

### 7. Verify

- The new test passes.
- The full suite still passes — run it, paste the real output.
- The original reproduction no longer reproduces. Re-run it; do not assume.

### 8. Review

Spawn a fresh `li-reviewer` with the diff and the ticket path. Ask it
specifically: does the fix address the stated root cause, and does it hold for
every blast-radius site the ticket lists as affected?

### 9. Close the ticket — edit the file

`Edit` the same `docs/bugs/<slug>.md`: set `status: resolved`, confirm the Root
cause section, fill in Fix with what changed and the guarding test. If the fix
proved the diagnosis wrong, **correct the ticket** — a ticket left asserting a
refuted cause is a trap for the next person.

Do not write a second file for the fix. One bug, one ticket, updated in place.

### 10. Report

```
Title:        <the task title>
Root cause:   <one sentence, with file:line>
Fix:          <what changed>
Test:         <the test that now guards it>
Verification: <actual command output>
Blast radius: <sites fixed / sites deferred to their own ticket>
Ticket:       docs/bugs/<slug>.md
```

Do not commit or push. That is the user's call.

## Never

- Never fix a symptom because the cause is hard to find. Say it is hard.
- Never present a single untested hypothesis as a root cause.
- Never skip the blast radius because the fix looks local.
- Never widen the fix into a refactor. Note the refactor; do not do it.
- Never remove or weaken a failing test to make the suite green.
- Never report "fixed" without having re-run the reproduction.
- Never leave the ticket asserting a cause the fix disproved.
- Never print the ticket in chat instead of writing `docs/bugs/<slug>.md`. The
  file is the deliverable; the chat block is a pointer to it.

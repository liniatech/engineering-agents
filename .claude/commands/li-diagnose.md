---
description: Deep bug diagnosis only — root cause by elimination, blast radius, failing test, bug ticket. No fix.
argument-hint: <the bug, or just the error message>
---

Run **phase 1 only** of the `li-bug-fix` skill for this:

$ARGUMENTS

A short error message is enough to start. Investigate before asking me for
more — ask only for what investigation cannot recover.

I want the diagnosis as the deliverable:

- Reproduce it, or stop and tell me what is missing.
- Root cause by **elimination** — at least two hypotheses, the evidence that
  refuted each, and the survivor with `file:line`.
- Blast radius — every other site sharing the same cause, with a verdict each.
- A test that fails **because of this bug**, run and confirmed failing for the
  right reason.
- The ticket **written to `docs/bugs/<slug>.md`** with the Write tool, with a
  task title I can paste straight into a tracker. The file is the deliverable —
  show me a summary and the path, not the ticket pasted into chat.

**Stop at the gate.** Do not fix anything, and do not offer to — I will run
`/li-bugfix docs/bugs/<slug>.md` when I want the fix.

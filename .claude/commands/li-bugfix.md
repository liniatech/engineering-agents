---
description: Root-cause-first bug fix — diagnose, ticket, failing test, fix, verify
argument-hint: <the bug, an error message, or a path to an existing bug ticket>
---

Run the `li-bug-fix` skill for this defect:

$ARGUMENTS

Both phases. Reproduce it before diagnosing. At the phase-1 gate show me the
task title, the root cause with `file:line`, what you ruled out and why, and the
blast radius — then wait for my yes before touching production code.

If you cannot reproduce it, stop and tell me what you need.

If the argument above is a path to an existing bug ticket, re-confirm the
reproduction still holds, then go straight to phase 2.

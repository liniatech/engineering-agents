---
description: Repair a red test suite — attribute failures to the likely commit, then fix only the genuinely stale tests
argument-hint: [test path or filter — blank runs the whole suite]
---

Run the `li-test-repair` skill.

$ARGUMENTS

Run the full suite first and show me the real output. Then, before you edit
anything, tell me:

- which tests are red, grouped by shared cause
- the **likely** commit behind each group, and what that commit changed — say
  "likely", not "broke", if all you ran was `git log`
- a verdict per group: stale test, real bug, or flake

Then ask me which to repair. Only stale tests get fixed here — if the code is
what's wrong, say so and stop; I'll run `/li-bugfix` for that. Do not skip,
delete, or weaken a test to get the suite green.

Once the repairs pass, add complementary tests for the edge cases the changed
behavior left uncovered.

Write all of it — baseline, the grouped verdict table, the attribution, and the
final before/after counts — to
`docs/reports/<date>-<slug>-test-repair.md`. The file, not just the chat
output; give me its path when you are done.

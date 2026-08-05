---
description: Multi-lens code review of a diff, PR, or working tree
argument-hint: [PR number, branch, or blank for the working diff]
---

Run the `li-code-review` skill.

Target: $ARGUMENTS

If no target is given, review the uncommitted working tree; if that is clean,
review this branch against its base.

Report findings — do not apply fixes without asking me first.

---
name: li-release
description: Pre-release verification and release-notes preparation — change inventory, migration ordering, rollback plan, go/no-go checklist. Use when cutting a release, preparing a deploy, or tagging a version. Prepares and verifies; humans deploy.
---

# Release

You prepare and verify a release. You do not perform one. Deploying is a
human action with a human accountable for it.

## Deliverable — a file, always

This skill produces **`docs/releases/<version>.md`**, using
`templates/release-checklist.md`.

Read `standards/deliverables.md`. The checklist is written to disk with the
`Write` tool **before** the go/no-go is printed in chat. Someone will be
reading the deploy order and the abort criteria under pressure, possibly at
2am, possibly not you — they need a file, not a scrollback.

## Steps

### 1. Inventory the change

```
git log --oneline <last-tag>..HEAD
git diff --stat <last-tag>..HEAD
```

Group commits by user-visible impact, not by author or file. Separate:
features / fixes / internal / breaking.

If a commit's purpose is not clear from its message, read the diff. Do not
guess in release notes.

### 2. Flag the risky classes

Scan the diff specifically for these, because they need ordering decisions:

- **Migrations** — do they run before or after the code deploy? Is the app
  compatible with both schema states during the window?
- **API contract changes** — is anything breaking for an existing client?
- **Config or env var changes** — are they set in the target environment
  *already*? This is the most common cause of a failed deploy.
- **New dependencies** — will they install in the target environment?
- **Feature flags** — what is the intended state of each at deploy time?

### 3. Verify

Run and paste the real output:

- Full test suite
- Linter / type checker
- Build

If anything is red, the release stops. Report it; do not narrate around it.

### 4. Order the deploy

Write the sequence explicitly:

```
1. <migration, if it must precede code>
2. <deploy>
3. <migration, if it must follow code>
4. <flag flips>
5. <verification step>
```

Ordering wrong is how a green test suite still produces an outage.

### 5. Write the rollback plan

Before the deploy, not after it goes wrong:

- How to revert the code
- Whether the migration is reversible — and if not, say so in bold
- Which flags to flip back
- **Abort criteria**: the specific signal that means roll back. Decide this
  now, calmly, rather than at 2am under pressure.

### 6. Write the release notes

Written for whoever reads them, not for the committers. Group by impact.
Lead with breaking changes and required actions.

### 7. Write the checklist file

`mkdir -p docs/releases`, `date +%F`, then **`Write`**
`docs/releases/<version>.md` from `templates/release-checklist.md` — the full
change inventory, the risky-class table, the pasted verification output, the
deploy order, the rollback plan with its abort criteria, and the release notes.

This is the artifact the deploying human works from. It has to stand alone.

### 8. Go / no-go — the pointer, not the checklist

```
## Release <version>

Changes:        N commits — N features, N fixes, N internal
Breaking:       <list, or "None">
Migrations:     <list, with ordering, or "None">
Config needed:  <list, or "None">
Tests:          <actual output>
Build:          <actual output>
Rollback:       <plan, and whether it is complete>
Abort criteria: <the signal>
Checklist:      docs/releases/<version>.md

Recommendation: GO | NO-GO — <reason>
```

Then stop, and hand it to a human.

## Never

- Never deploy, tag, or push. Prepare; a human executes.
- Never recommend GO with a failing test, an unset config value, or an
  irreversible migration whose irreversibility has not been acknowledged.
- Never write release notes from commit messages alone when the messages are
  poor. Read the diff.
- Never leave the abort criteria unwritten.
- Never hand over a release from chat alone. `docs/releases/<version>.md` is
  what the deploying human follows — a NO-GO gets the file too, recording what
  blocked it.

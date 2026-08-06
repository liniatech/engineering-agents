---
title: Release <version>
type: release
date: <YYYY-MM-DD>
status: go | no-go
scope: <last-tag>..HEAD
---

# Release <version>

## Recommendation

**GO | NO-GO** — one sentence saying why.

## Change inventory

`<last-tag>..HEAD` — N commits.

Grouped by user-visible impact, not by author or file. Where a commit message
was unclear, the diff was read — nothing here is guessed.

### Breaking

### Features

### Fixes

### Internal

## Risky classes

| Class | Present? | Detail |
|---|---|---|
| Migrations | | ordering: before / after code deploy |
| API contract changes | | breaking for which clients |
| Config / env vars | | **set in the target environment already?** |
| New dependencies | | will they install there |
| Feature flags | | intended state at deploy time |

Unset config in the target environment is the most common cause of a failed
deploy. Verify it; do not assume it.

## Verification

Real output, pasted — not "tests pass".

**Test suite:**

```
```

**Linter / type checker:**

```
```

**Build:**

```
```

Anything red stops the release.

## Deploy order

```
1. <migration, if it must precede code>
2. <deploy>
3. <migration, if it must follow code>
4. <flag flips>
5. <verification step>
```

Ordering wrong is how a green suite still produces an outage.

## Rollback plan

- **Revert the code:**
- **Migration reversible?** yes / **NO — forward-fix only, described here**
- **Flags to flip back:**
- **Abort criteria:** the specific signal that means roll back. Decided now,
  calmly, rather than at 2am.

## Release notes

Written for whoever reads them, not for the committers. Breaking changes and
required actions first.

## Handoff

Prepared, not performed. A human deploys, tags, and pushes.

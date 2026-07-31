---
name: dependency-upgrade
description: Safely upgrade a library, framework, or runtime — changelog review, breaking-change hunt, incremental application, verification. Use when bumping versions, patching a CVE, or resolving dependency drift.
---

# Dependency upgrade

The risk in an upgrade is never the version number; it is the behavior change
nobody read about.

## Steps

### 1. Inventory

Establish precisely:

- Current version → target version
- Why now (CVE, feature needed, drift, EOL)
- Direct or transitive
- How many places in the codebase touch its API

```
grep -rn "<package>" --include="*.py" --include="*.toml" --include="*.txt" .
```

### 2. Read the changelog — all of it

Not just the target version. **Every version between current and target.**
Breaking changes hide in intermediate releases, and a two-hop upgrade
inherits both.

Extract:
- Breaking changes
- Deprecations (today's warning is next upgrade's break)
- Behavior changes that are not flagged as breaking — the dangerous class:
  a default that changed, a timeout that shortened, a return that went from
  `None` to raising

If the changelog is thin, diff the release tags for the modules you use.

### 3. Assess blast radius

For each breaking change, find every call site. `file:line` each. If a
breaking change affects zero call sites, say so — that is what makes an
upgrade cheap and it is worth knowing early.

### 4. Choose the path

- **Patch bump, no breaking changes** — apply, run the suite, done.
- **Minor with deprecations** — apply, fix deprecation warnings now, not later.
- **Major** — go one major version at a time. `1.x → 3.x` in one jump means
  debugging two sets of breaking changes simultaneously with no green state
  in between.

### 5. Establish a baseline first

Run the full suite **before** upgrading and record the result. Without a
baseline you cannot tell which failures the upgrade caused. If the suite is
already red, fix or document that first.

### 6. Apply

Spawn `backend-engineer` with the version bump and the list of breaking
changes plus their call sites. Instruct it to fix breakage only — no
opportunistic refactoring inside an upgrade diff.

### 7. Verify beyond the test suite

The suite proves what it covers, which is never everything. Also check:

- Deprecation warnings in the test output — read them, do not suppress them
- The lockfile diff: what transitive dependencies moved, and did anything
  unexpected come along
- Startup: does the app actually boot
- Any behavior the changelog flagged that tests do not cover

### 8. Review and report

Spawn a fresh `reviewer`. Then report:

```
Package:     <name> <old> → <new>
Reason:      <why>
Breaking:    <each change, and the call sites fixed>
Transitive:  <what else moved in the lockfile>
Baseline:    <suite result before>
After:       <suite result after — actual output>
Unverified:  <behavior changes tests do not cover>
Rollback:    <how to revert>
```

## Never

- Never upgrade across multiple majors in one step.
- Never suppress a new deprecation warning instead of addressing it.
- Never bundle an upgrade with a feature change — when it breaks in
  production you cannot tell which half did it.
- Never skip the pre-upgrade baseline.

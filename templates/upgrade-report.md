---
title: Upgrade — <package> <old> → <new>
type: upgrade
date: <YYYY-MM-DD>
status: draft | resolved
scope: <package and version range>
---

# Upgrade — <package> `<old>` → `<new>`

## Why now

CVE / feature needed / drift / EOL — and the deadline, if there is one.

## Inventory

| | |
|---|---|
| Package | |
| Current → target | |
| Direct or transitive | |
| Call sites touching its API | N (`grep` used: `<command>`) |

## Changelog — every version between current and target

Not just the target release. Breaking changes hide in intermediate versions,
and a two-hop upgrade inherits both.

### Breaking changes

| Version | Change | Call sites | Fixed? |
|---|---|---|---|

### Deprecations

Today's warning is next upgrade's break.

| Version | Deprecated | Replacement | Addressed? |
|---|---|---|---|

### Unflagged behavior changes

The dangerous class — a default that moved, a timeout that shortened, a return
that went from `None` to raising.

| Version | Change | Does it affect us? |
|---|---|---|

## Path taken

Patch / minor / one major at a time — and why. Record each hop separately if
the upgrade crossed more than one major.

## Baseline — before the upgrade

```
```

If the suite was already red before touching anything, that is recorded here,
not discovered afterwards.

## After

```
```

## Beyond the suite

- **Deprecation warnings in the output:** read, not suppressed —
- **Lockfile diff:** what transitive dependencies moved, and anything
  unexpected that came along —
- **Startup:** does the app actually boot —
- **Changelog behavior the tests do not cover:** —

## Unverified

Behavior the changelog flagged that nothing in the suite exercises. This is
the honest residual risk of the upgrade.

## Rollback

Exactly how to revert — the version pin, the lockfile, and any code change
that has to go back with it.

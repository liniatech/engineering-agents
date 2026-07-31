# Design Review

**Design:**
**Author:**
**Reviewer:**
**Date:**

## Verdict

sound | concerns | reconsider

## What this design does

The reviewer's own restatement, in their words. If this does not match the
author's intent, that mismatch is the first finding.

## Scope reviewed

What was covered. What was explicitly **not** covered.

## Boundaries and ownership

Does each component own its data? Any shared-store coupling? Is the boundary
drawn along a real seam?

## Contracts

Explicit and versioned? Migration path for breaking changes?

## Failure modes

| Dependency | On timeout | On error | Retry safe? | Backpressure |
|---|---|---|---|---|

## Concerns

Ordered by cost-to-fix-later, not severity-today.

### 1. <concern>

- **Scenario that makes it bite:**
- **Earliest warning signal:**
- **Cost to address now vs. later:**

## Unstated tradeoffs

What is being given up that the document does not admit to.

## Over- / under-engineering

- **Over:** abstraction for a requirement nobody has
- **Under:** missing failure handling, no migration path, one-way doors

## Reversibility

| Decision | Cheap / Expensive / One-way |
|---|---|

## Alternatives

Were real alternatives considered, or one option and two straw men?

## Questions for the author

## Action items

| Item | Owner | Blocking? |
|---|---|---|

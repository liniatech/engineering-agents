---
title: Performance — <operation>
type: performance
date: <YYYY-MM-DD>
status: draft | resolved
scope: <endpoint, query, or job>
---

# Performance — <operation>

## Target

| | |
|---|---|
| Operation | <specific endpoint / query / job> |
| Metric | p50 / p95 / p99 / throughput / memory |
| Current | <measured number> |
| Target | <number, and where it comes from> |

## Method

How the measurement was taken — profiler, timing harness, APM trace, data
volume, warm or cold. **Both measurements must use this exact method**; a
before and after measured differently prove nothing.

## Baseline

The real number, with the raw output.

```
```

## Bottleneck

What it actually was, with evidence — the query plan, the profile frame, the
query count. Cite `file:line`.

**Share of total:** this component is N% of the measured operation. Fixing it
perfectly buys at most that much.

Candidates considered and eliminated:

| Suspected | Ruled out by |
|---|---|

## Change

One change. What was done, and where.

## After

Same method, same conditions, same data volume.

```
```

**Improvement:** <x% / Nms>. If it came in below expectation, say so — a
change that did not help should be reverted, not kept on faith.

## Guard

The test, benchmark, or alert that stops this regressing:

- Query-count assertion / benchmark threshold / metric alert
- Path:

## Remaining

The next bottleneck, now that this one is gone — and whether it is worth
chasing.

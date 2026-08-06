---
name: li-performance-review
description: Measure-first performance investigation — profile, find the real bottleneck, fix, prove the improvement. Use when something is slow, latency regressed, or the user asks to optimize. Refuses to optimize without a measurement.
---

# Performance review

**Measure before you change anything.** Intuition about performance is
reliably wrong, and an optimization without a before-number cannot be shown
to have worked.

## Deliverable — a file, always

This skill produces **`docs/performance/<date>-<slug>.md`**, using
`templates/performance-report.md`.

Read `standards/deliverables.md`. The report is written to disk with the
`Write` tool **before** anything is printed in chat. Numbers that live only in
a terminal are numbers nobody can compare against next time — and the whole
discipline here is comparison.

Write the file as soon as you have the baseline (step 2), then update it in
place at step 6 and step 8. That way the before-number survives even if the
optimization goes nowhere.

## Steps

### 1. Define the target

Vague goals produce vague work. Establish:

- Which operation (specific endpoint, query, job)
- Which metric (p50, p95, p99, throughput, memory) — p99 and p50 problems
  have different causes and different fixes
- Current value, measured
- Target value, and where it comes from

If the user cannot say what "fast enough" means, ask. Optimization with no
finish line does not have one.

### 2. Measure

Get a real number before touching code. Profile, time the query, read the
APM trace. Record the method so it can be repeated identically afterwards —
a before and after measured differently prove nothing.

**Create the file now.** `mkdir -p docs/performance`, `date +%F`, then `Write`
`docs/performance/<date>-<slug>.md` from `templates/performance-report.md` with
Target, Method, and Baseline filled in. The rest gets filled as you go.

### 3. Find the actual bottleneck

Follow the evidence, not the suspicion. Common real causes, roughly in order
of how often they turn out to be the answer:

- **N+1 queries** — the single most common, and usually invisible in code review
- **Missing index** — check the query plan; a sequential scan on a large
  table is the tell
- **Unbounded result set** — no pagination, no limit; fine until the table grows
- **Serial I/O that could be concurrent**
- **Work inside a loop that belongs outside it**
- **Over-fetching** — selecting columns or relations nobody reads
- **Missing cache on genuinely stable data**

Algorithmic complexity is the cause people look for first and find last.

### 4. Confirm it is the bottleneck

Before optimizing, establish that this component actually dominates. If the
slow query is 40ms of a 3-second request, fixing it perfectly buys 1%.

Say what percentage of the total you are addressing. This step prevents most
wasted optimization work.

### 5. Fix

Spawn `li-backend-engineer` (or `li-database-engineer` if the fix is an index or
query rewrite) with the measurement and the identified bottleneck.

One change at a time. Two simultaneous optimizations mean you cannot
attribute the improvement, and one of them may be making things worse.

### 6. Measure again — identically

Same method, same conditions, same data volume. Report the real numbers.

If the improvement is smaller than expected, say so. A change that did not
help should be reverted, not kept because it seems like it should help.

### 7. Guard it

A performance fix with no regression test decays. Add whichever fits:

- A test asserting query count (catches N+1 returning)
- A benchmark with a threshold
- An alert on the metric

### 8. Finish the file, then report

`Edit` `docs/performance/<date>-<slug>.md` to completion — Bottleneck with its
evidence and share-of-total, the eliminated candidates, Change, After with the
raw output, Guard, and Remaining. Set `status: resolved`.

Then, in chat:

```
Operation:   <what>
Metric:      <which>
Before:      <number, and how measured>
Bottleneck:  <what it actually was, with evidence>
Change:      <what was done>
After:       <number, same method>
Improvement: <x% / Nms>
Guard:       <the test or alert that prevents regression>
Remaining:   <the next bottleneck, now that this one is gone>
Report:      docs/performance/<date>-<slug>.md
```

## Never

- Never optimize without a before-measurement.
- Never make several optimizations at once.
- Never report an improvement measured differently from the baseline.
- Never trade correctness for speed without saying so explicitly.
- Never cache to hide an N+1. Fix the query.
- Never report numbers without writing the file. An investigation that found
  nothing worth changing still gets one — the baseline is the value.

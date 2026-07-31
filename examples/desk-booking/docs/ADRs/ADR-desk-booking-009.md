# ADR-desk-booking-009 — Testing

## Status

Proposed — 2026-07-31. Owned by QA. Defines the test strategy the acceptance criteria (AC-1..AC-10) and the correctness invariant (NFR-4) will lean on. Depends on ADR-001 (single-writer invariant), ADR-004 (409-vs-201 contract), ADR-007 (CI merge gates). No code exists yet; this ADR fixes the strategy so tests are written *with* the feature, not after.

## Context

The dominant product risk is trust (PRD Risks): one wrong seat collapses adoption. The single hard constraint is NFR-4 — zero double-bookings under concurrency, an invariant, not a target. AC-5 makes that invariant observable: two simultaneous bookings of the same desk/date must yield exactly one `201` and one clean `409 desk_unavailable` (ADR-004), never a second reservation and never a `200`-with-error.

ADR-001 fixes that the reservations data store is the sole owner and enforcer of the (desk, date) uniqueness invariant, with exactly one writer. ADR-004 makes the race outcome visible at the API as a distinguishable status. Those two decisions are what makes NFR-4 *testable at all*: the proof reduces to "the database rejects the second write atomically, and the API maps that rejection to 409."

The load-bearing requirement from ADR-001/004 and ADR-007: there must be an automated concurrency test proving AC-5, running as a required merge gate. A green suite without it is spending trust it has not earned.

Constraint on what can be tested today: several PRD open questions are `[NEEDS INPUT]` and gate acceptance criteria directly. Tests for those AC cannot be authored to a fixed assertion until the input lands, because the expected outcome is undefined. This ADR states which AC are blocked and why, so the gap is visible rather than silently uncovered.

## Decision

### Test levels and what belongs at each

Per `standards/testing-standards.md`: mostly fast and narrow, few slow and broad.

**Unit — pure behavior, no I/O.** Booking-window validation (once FR-10 resolves, clock injected); role derivation from SSO claims (ADR-002); error-envelope mapping (uniqueness → `desk_unavailable`, out-of-window → `422`); availability projection.

**Integration — real Postgres, real transactions, real constraints.** The tier that carries NFR-4. SQLite prohibited — the uniqueness constraint and isolation behavior only exist on the real engine. Covers: the (desk, date) uniqueness invariant (second insert rejected, no duplicate row); cancellation frees the slot (AC-6); manager-visibility query returns holder identity (AC-8); empty/full state (AC-9, AC-10). Each test creates and tears down its own data.

**Contract — API shape against what the SPA expects (ADR-004).** `POST /reservations` → `201`+`Location` / `409 desk_unavailable` / `422`, never `200`-with-error; `Idempotency-Key` replay of a success returns the original `201`; `GET /desks?date=` returns `status`, `400` on malformed date; authz `403`s.

**E2E — very few.** Two flows only: (1) SSO gate → availability grid (AC-1+AC-2); (2) book → confirmed to self, reserved to a second session (AC-3). Proves wiring, not coverage; not where the invariant is proven.

### The AC-5 concurrency test — load-bearing, and a required CI gate

One dedicated, deterministic integration test is the primary proof of NFR-4:

- Arrange: one desk, one date, clean DB, two prepared booking requests.
- Act: fire both writes so they contend, using a **synchronization barrier** (both transactions reach the write point, then release together) — **not** `sleep`. A timing-based race is flaky and will be rejected.
- Assert, as one behavior: exactly one `201`; the other `409 desk_unavailable`; and the table holds exactly **one** row for that (desk, date). The row-count assertion is the one that catches the real bug — a "clean 409" that still wrote a duplicate would pass a status-only check and violate NFR-4 silently.
- Catches: any check-then-insert race, any isolation level too weak, any code path that swallows the constraint violation.

Per ADR-007 this runs against real Postgres in the `test` stage as a **required merge gate**: red or absent → merge blocked. CI runs the suite in randomized order. A higher-concurrency variant (N writers → one `201`, N−1 `409`, one row) is recommended; the two-writer test is the gate.

### How NFR-4 is proven — by construction, three layers

1. **Database (integration):** the constraint rejects the duplicate atomically — the row-count-of-one assertion.
2. **API (contract):** the rejection surfaces as `409 desk_unavailable`, distinguishable from success.
3. **Concurrency (the AC-5 gate):** the two hold *under contention*, not just sequentially.

AC-4 (sequential) is necessary but not sufficient; AC-5 (simultaneous) is the real proof and the gate.

### Coverage matrix — acceptance criteria to test level

| AC | Behavior | Level(s) | Testable now? |
|---|---|---|---|
| AC-1 | SSO required, no data pre-auth | Contract + E2E | Yes |
| AC-2 | View availability for a date | Integration + E2E | Yes |
| AC-3 | Successful booking, visible to others | Contract (201) + E2E | Yes |
| AC-4 | Sequential double-booking rejected | Integration + Contract (409) | Yes |
| **AC-5** | **Concurrent — exactly one winner** | **Integration concurrency (required gate)** | **Yes — the priority** |
| AC-6 | Cancellation frees the desk | Integration + Contract (204) | Yes |
| AC-7 | Booking window enforced | Unit + Contract (422) | **Blocked** — FR-10 window rule is `[NEEDS INPUT]` |
| AC-8 | Manager sees all + holder identity | Integration + Contract (403 non-mgr) | Partial — depends on FR-13 |
| AC-9 | Empty state — all available | Integration | Yes |
| AC-10 | Fully booked — nothing offered | Integration | Yes |

### How open questions limit what can be tested

- **FR-10 booking window (AC-7).** No window rule → no boundary to assert. **Blocked.**
- **FR-9 one-desk-per-day.** The "already holds a desk that day" `409` branch has no defined behavior to test. **Blocked.**
- **FR-13 manager cancel/reassign (AC-8, `DELETE` 403).** Determines authz assertions. **Partially blocked.**
- **FR-15 desk attributes.** id + floor testable now; attribute tests wait.
- **FR-5 staleness (AC-3 "reflected to others").** No staleness bound → E2E proves eventual visibility on refresh, not a real-time SLA.
- **NFR-1 peak / NFR-2 p95 < 1s.** A load test cannot be given a target until confirmed. Correctness under concurrency (AC-5) is independent and tested now.

## Alternatives

- **Prove NFR-4 with unit tests and mocks.** Rejected: mocking the DB mocks away the exact mechanism that enforces the invariant.
- **`sleep`-based timing race.** Rejected: flaky, trains the team to ignore red. Use a barrier.
- **SQLite for integration speed.** Rejected by standards: differs from Postgres in constraint and isolation behavior — the ways that matter here.
- **Heavy E2E of every AC.** Rejected: slow, brittle, low diagnostic value. Push behavior down to integration/contract.
- **Defer the concurrency test until after launch.** Rejected: NFR-4 is the whole reason the product exists.

## Tradeoffs

Integration/concurrency tiers need a real Postgres in CI — slower, more infra; accepted as the only place NFR-4 is provable. The AC-5 gate can slow merges if it flakes — mitigated by barrier determinism and randomized ordering; a flaky gate is fixed, never retried into green. Several AC are blocked on open questions — the honest cost is visible uncovered criteria now, rather than tests written against guessed behavior.

## Consequences

- Merge is blocked by a red or missing AC-5 test (ADR-007), intentionally.
- The 409-vs-201 distinction is a tested contract; changing it breaks contract tests — the desired alarm.
- AC-7 and the FR-9/FR-13-dependent branches ship as *known, tracked* coverage gaps until inputs resolve — never silently skipped.
- Latency/load verification is out of this suite until targets are confirmed; correctness under concurrency is not deferred.

## Follow-up Actions

- Author the AC-5 concurrency integration test first and wire it as a required merge gate (ADR-007).
- Stand up real-Postgres integration + contract tiers with randomized ordering.
- On FR-10: add AC-7 window boundary unit tests + `422` contract test.
- On FR-9: add or delete the "already holds a desk that day" `409` branch test.
- On FR-13: fix the `DELETE`/manager authz assertions.
- On NFR-1/NFR-2: add a load test asserting p95 < 1s and correctness at peak.
- Track every blocked AC as an open coverage ticket.

## Last verified
2026-07-31

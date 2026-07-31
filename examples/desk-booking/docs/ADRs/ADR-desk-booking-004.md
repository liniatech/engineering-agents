# ADR-desk-booking-004 — API

## Status

Proposed — 2026-07-31.

## Context

The SPA needs to read availability (FR-4/FR-5), book (FR-6/FR-8), view and cancel own reservations (FR-11/FR-12), and let managers view all reservations (FR-14) and configure inventory (FR-15). The contract must make the concurrency guarantee (NFR-4, AC-4, AC-5) observable to the client as a clean, distinguishable outcome. `standards/api-guidelines.md` governs shape, status codes, errors, pagination, idempotency, versioning.

## Decision

**REST, versioned in the path (`/v1`), JSON, `snake_case` fields, ISO-8601 UTC timestamps.** The same-origin SPA authenticates with the session cookie from ADR-002 (not a bearer token — no third-party or mobile client exists). CSRF token required on all state-changing requests (ADR-008).

| Method + path | Purpose | Success | Notable failures |
|---|---|---|---|
| `GET /v1/desks?date=YYYY-MM-DD` | Availability for a date (FR-4) | 200 list with `status: available\|reserved` | 400 bad date |
| `POST /v1/reservations` | Book one desk/day (FR-6) | 201 + `Location` | **409** desk/date taken; 422 outside window; 409 employee already holds a desk that day (if FR-9 = one/day) |
| `GET /v1/reservations?mine=true` | My upcoming reservations (FR-11) | 200 paginated | — |
| `DELETE /v1/reservations/{id}` | Cancel own (FR-12) | 204, desk freed | 403 not owner (unless manager, FR-13); 404 |
| `GET /v1/reservations?date=YYYY-MM-DD` | Manager: all for a date (FR-14) | 200 paginated incl. holder | 403 non-manager |
| `GET/POST/PATCH/DELETE /v1/desks…` | Inventory config (FR-15) | per method | 403 non-manager |

- **Booking concurrency contract:** `POST /v1/reservations` is where AC-5 is made visible. The losing side of a race gets **409** with stable `code: "desk_unavailable"`; exactly one concurrent request gets 201. Never 200-with-error.
- **Idempotency:** `POST /v1/reservations` accepts an optional `Idempotency-Key`; a network retry of a successful booking returns the original 201, not a spurious 409.
- **Error shape:** the single standard envelope (`error.code`, `message`, `details[]`, `request_id`). Validation reports all failures at once.
- **Pagination:** every collection is paginated (cursor, default + hard-max limit).
- **Data ownership:** the Booking API is the sole owner and sole writer of desks and reservations.

## Alternatives

- **GraphQL.** Rejected: a small SPA with fixed views gains nothing and pays in caching/authz complexity.
- **gRPC.** Rejected: not browser-native.
- **Bearer/JWT instead of cookie session.** Rejected for v1: no cross-origin/mobile/third-party consumer; cookie session is less code and less token risk.

## Tradeoffs

Cookie-session auth couples the API to a browser client and requires CSRF defense. Path versioning means a breaking change costs a parallel `/v2` — the standard cost of a real contract. REST over GraphQL may over-fetch on some views; negligible at this data size.

## Consequences

- Breaking changes require `/v2` + expand-migrate-contract; additive changes do not. Clients must tolerate unknown fields.
- The 409-vs-201 distinction is load-bearing for the QA concurrency test (ADR-009).

## Follow-up Actions

- Finalize request/response schemas with backend-engineer once FR-9/FR-10/FR-13 resolve.
- Confirm booking-window rule (FR-10) to define the 422 case.

## Last verified
2026-07-31

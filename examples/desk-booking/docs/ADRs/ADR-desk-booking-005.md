# ADR-desk-booking-005 — Observability

## Status

Proposed — 2026-07-31.

## Context

`architecture-principles.md`: if you cannot tell from outside that it is working, it is not operable — structured logs with a correlation ID, RED metrics, health vs readiness. But this is a **single service** at tiny scale; distributed tracing has no boundaries to cross here. NFR-3 targets 99.5% during business hours; NFR-2 targets p95 < 1s.

## Decision

Right-size to a single-service monolith:

- **Structured JSON logs**, one event per request, with a **correlation/request ID** propagated from ingress and returned in the API `request_id` field. No PII beyond name/email in logs, never secrets (ADR-008).
- **RED metrics** (Rate, Errors, Duration) per endpoint and for the DB dependency call, to the platform's managed metrics backend — no self-hosted Prometheus/Grafana for 120 users.
- **Business events:** `reservation_created`, `reservation_cancelled`, `reservation_conflict` (the 409 path). Conflicts are expected and are an adoption/health signal.
- **The one metric that goes bad first:** p95 latency and 5xx rate on `POST /v1/reservations`. Alert on that user-visible symptom, plus an external uptime check on the availability endpoint for the 99.5% SLO.
- **Invariant alarm:** any DB uniqueness violation that surfaces as a 5xx (i.e. not cleanly translated to 409) is a defect in the correctness path — alert distinctly. A successful 409 is normal; a leaked constraint error is not.
- **Health vs readiness:** `/healthz` (process alive) and `/readyz` (DB reachable).
- **No distributed tracing** in v1 — single service, single DB hop.

## Alternatives

- **Full OpenTelemetry + tracing + self-hosted stack.** Rejected: over-engineering for one service and 120 users.
- **Logs only, no metrics/alerts.** Rejected: leaves the SLO and adoption-critical latency unmonitored — you'd learn of an outage from an office manager.

## Tradeoffs

We give up cross-service tracing and rich custom dashboards for launch speed and low operational load — acceptable with one service. If the monolith is split, this ADR must be superseded to add tracing.

## Consequences

- Managed logging/metrics ties us somewhat to the chosen platform (ADR-006); reversible.
- Alerts are symptom-based (latency, 5xx, uptime, leaked-invariant).

## Follow-up Actions

- Define alert thresholds and on-call recipient.
- Confirm business hours / time zones for the uptime window (PRD open question 11).

## Last verified
2026-07-31

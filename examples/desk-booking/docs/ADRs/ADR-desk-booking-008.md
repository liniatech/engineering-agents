# ADR-desk-booking-008 — Security

## Status

Proposed — 2026-07-31.

## Context

Internal tool, one Workspace domain, storing only name + work email (NFR-6) — but "internal" is not a security exemption (`security.md`: trusted/internal input still needs the full checklist). NFR-5 mandates non-public reachability. Dominant risks are **broken access control** (IDOR — cancelling someone else's reservation, or reading the manager view) and **injection**. FR-13 (who may cancel/reassign) is unresolved and directly affects authorization rules.

## Decision

Defense in depth, following `security.md`:

- **Network (NFR-5):** private ingress only (IAP/VPN, ADR-006).
- **Transport:** TLS everywhere, including app-to-Postgres.
- **Authentication:** Google OIDC only, library-based, no local credentials (ADR-002).
- **Authorization — default deny, object-level:** every endpoint checks *may this caller act on this specific object*, derived from the server session, never from a client-supplied id/role.
  - Cancel/view-own: caller must own the reservation (`DELETE /reservations/{id}` → 403 if not owner). Primary IDOR guard.
  - Manager views require the server-derived manager role. Manager cancel/reassign of others' bookings is gated on FR-13 — **until resolved, deny it**.
- **CSRF:** cookie session (ADR-004) → all state-changing requests carry a CSRF token; `SameSite=Lax` as a second layer.
- **Injection:** parameterized queries only; string-interpolated SQL is a Critical finding even internally. Filters/sorts server-side allowlisted.
- **Input validation** at the trust boundary: allowlist; bound every field's type/length/range; validate dates and desk ids server-side.
- **Rate limiting:** `POST`/`DELETE /reservations` per user and per IP to bound morning-peak hammering and retry storms; 429 + `Retry-After`.
- **Secrets:** OAuth client secret, DB credentials → secrets manager, injected at runtime; never in git, logs, or client bundle. `.env` gitignored.
- **Least privilege:** app connects as CRUD-only; migrations use a separate DDL credential; retention-purge job (NFR-8) gets its own credential.
- **Data minimization + encryption:** store only name + work email (NFR-6); managed encryption at rest; design for KMS key rotation.
- **Retention:** a scheduled purge enforces NFR-8 (proposed current + prior 90 days) once confirmed.
- **Error hygiene:** standard envelope leaks no internals; unauthorized reads of real vs nonexistent objects return the same status (no enumeration oracle).

## Alternatives

- **Rely on network isolation alone (skip object-level authz).** Rejected: a valid internal user could still cancel a colleague's booking — the exact friction the PRD wants gone. Internal ≠ trusted.
- **Route-level authz only (role check, no object check).** Rejected: this is precisely IDOR.

## Tradeoffs

CSRF handling and per-user/per-IP rate limiting add surface for a low-threat internal audience; accepted because the cost is small and the access-control failures prevented are the product's core promise. Object-level checks add a query condition per reservation operation — negligible, non-negotiable.

## Consequences

- FR-13 must resolve before manager cancel/reassign can be enabled; until then denied by default.
- Three DB credential scopes to provision (app CRUD, migration DDL, purge job).
- Any change to auth, authz, or PII handling triggers the `security.md` human-review escalation.

## Follow-up Actions

- Resolve FR-13 (manager cancel/reassign) and FR-9 (one desk/day) — both authorization rules.
- Confirm retention period (NFR-8, open question 13) to configure the purge job.
- Security review of the OIDC flow and object-level authz before launch.

## Last verified
2026-07-31

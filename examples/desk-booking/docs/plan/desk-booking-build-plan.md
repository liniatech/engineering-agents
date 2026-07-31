# desk-booking — Build Plan

The documented architecture, broken into tasks an engineer can pick up. This
is the kickoff's final deliverable. Each task is built separately afterward
(e.g. via `/feature`), not in the kickoff run.

## Ordering

Build tasks top to bottom. A task lists what must exist before it starts. T1–T3
are the spine; the booking core (T5) is the reason the product exists and the
riskiest task — do not reorder it earlier than its dependencies.

## Blocking open questions

These `[NEEDS INPUT]` items gate specific tasks and should be answered by
product before that task starts. They do not block T1–T5.

- FR-10 booking window → T6 (AC-7)
- FR-9 one desk/day → T5 (extra unique index + 409 branch)
- FR-13 manager cancel/reassign → T7, T8 (authz)
- FR-15 desk attributes → T4 (columns)
- NFR-8 retention period → T10

## Tasks

### T1 — Repo, CI skeleton, and infra baseline

- **Implements:** `ADR-desk-booking-006`, `ADR-desk-booking-007`
- **Depends on:** none
- **Scope:** Stand up the stateless service skeleton + responsive SPA shell, a managed Postgres (staging), and the CI pipeline (build → test → migrate-with-`migrator`-cred → deploy-to-staging, manual gate to prod). Provision the three DB roles (`app_rw`, `migrator`, `purge_job`). Secrets via secrets manager; `.env.example` only.
- **Acceptance criteria:** a trivial change flows through CI to staging behind private ingress; the app connects as `app_rw` (no DDL rights); migrations run only as `migrator`.

### T2 — Schema: initial create migration

- **Implements:** `ADR-desk-booking-003`
- **Depends on:** T1
- **Scope:** Write the initial create migration (human-run) for `users`, `desks`, `reservations`, including the partial unique index `reservations_active_desk_date_uidx` and `users_email_lower_uidx`. Reversible `down`. No FR-9/FR-13 columns yet (gated).
- **Acceptance criteria:** migration applies and rolls back cleanly on an empty DB; the partial unique index exists and rejects a duplicate active `(desk, date)` at the DB level.

### T3 — Auth: Google SSO + server session + role derivation

- **Implements:** `ADR-desk-booking-002`, `ADR-desk-booking-008` (session/CSRF)
- **Depends on:** T1
- **Scope:** OIDC auth-code + PKCE against Google, `hd` + allowlist domain check, server-side session (httpOnly/Secure/SameSite=Lax cookie, stored in DB), server-derived role (employee default; manager via allowlist/group), CSRF on state-changing requests.
- **Acceptance criteria:** **AC-1** — an unauthenticated user sees no desk/reservation data and is prompted to sign in; a signed-in user gets a session; a client-supplied role claim is ignored.

### T4 — Desk inventory (admin)

- **Implements:** `ADR-desk-booking-004` (`/v1/desks` config), `ADR-desk-booking-003`
- **Depends on:** T2, T3
- **Scope:** Manager-only CRUD to configure the 70 desks across two floors (`retired_at` for removal, not delete). `[NEEDS INPUT FR-15]` for attributes beyond code + floor.
- **Acceptance criteria:** a manager can add/retire desks; a non-manager gets 403; retired desks stop being bookable but keep historical reservations valid.

### T5 — Booking core (the AC-5 task)

- **Implements:** `ADR-desk-booking-003`, `ADR-desk-booking-004`, `NFR-4`
- **Depends on:** T2, T3
- **Scope:** `POST /v1/reservations` as a single `INSERT` with **no** check-then-act; map `SQLSTATE 23505` on the partial index → **409 `desk_unavailable`**; `201` + `Location` on success; optional `Idempotency-Key` replay returns the original 201. `GET /v1/desks?date=` availability (`cancelled_at IS NULL`). `[NEEDS INPUT FR-9]` adds the per-user/day index + branch.
- **Acceptance criteria:** **AC-2, AC-3, AC-4, AC-5** — especially AC-5: two simultaneous bookings of the same desk/date → exactly one 201, one clean 409, and exactly one row (the barrier-synchronized concurrency test from `ADR-desk-booking-009`, wired as a required merge gate).

### T6 — My reservations + booking window

- **Implements:** `ADR-desk-booking-004`
- **Depends on:** T5
- **Scope:** `GET /v1/reservations?mine=true` (paginated); enforce the booking window → `422` outside it. `[NEEDS INPUT FR-10]` defines the window rule.
- **Acceptance criteria:** **AC-7** (once FR-10 lands); an employee sees only their upcoming reservations.

### T7 — Cancellation

- **Implements:** `ADR-desk-booking-004`, `ADR-desk-booking-008` (object-level authz)
- **Depends on:** T5
- **Scope:** `DELETE /v1/reservations/{id}` as an `UPDATE cancelled_at = now()` (not a delete); owner-only by default; `[NEEDS INPUT FR-13]` decides manager cancel/reassign.
- **Acceptance criteria:** **AC-6** — cancel frees the desk/date for re-booking; a non-owner gets 403 (until FR-13 says otherwise).

### T8 — Manager view

- **Implements:** `ADR-desk-booking-004` (`/v1/reservations?date=`), `ADR-desk-booking-008`
- **Depends on:** T5
- **Scope:** Manager-only list of all reservations for a date, including holder identity, across both floors; paginated.
- **Acceptance criteria:** **AC-8** — a manager sees every reserved desk + holder for a date; a non-manager gets 403.

### T9 — Observability + alerts

- **Implements:** `ADR-desk-booking-005`
- **Depends on:** T5
- **Scope:** Structured JSON logs with correlation/request ID; RED metrics per endpoint; business events (`reservation_created/cancelled/conflict`); `/healthz` + `/readyz`; alerts on `POST /reservations` p95 + 5xx, an external uptime check, and a distinct alarm for a leaked (non-409) constraint violation.
- **Acceptance criteria:** a slow/erroring booking path fires an alert; a leaked uniqueness violation (5xx) alarms distinctly from a normal 409.

### T10 — Retention purge

- **Implements:** `ADR-desk-booking-008`, `NFR-8`
- **Depends on:** T2
- **Scope:** Scheduled purge job running as `purge_job` (SELECT/DELETE on reservations only). `[NEEDS INPUT NFR-8]` retention period (proposed current + prior 90 days).
- **Acceptance criteria:** reservations older than the retention window are purged on schedule; the job has no access beyond `reservations`.

## Not in this plan

- Meeting rooms, floor-plan editor, native mobile app, partial-day booking (PRD Out of Scope).
- Any fairness/allocation policy for the 70-desk/120-person scarcity (`[NEEDS INPUT]`, PRD Risk) — would be a feature ADR (`≥010`), not backbone.
- No-show release/expiry (`[NEEDS INPUT]`, PRD Risk).
- Load/performance test asserting p95 < 1s — deferred until NFR-1/NFR-2 targets are confirmed (`ADR-009`).

## Last verified
2026-07-31

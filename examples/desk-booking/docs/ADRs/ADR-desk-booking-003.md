# ADR-desk-booking-003 — Database

## Status

Proposed — 2026-07-31. Decision document only (kickoff). Physical migrations are written at build time, not here. Implements the data-store obligations assigned by `ADR-desk-booking-001` (single relational DB, sole owner and single writer of the no-double-booking invariant). Depends on `ADR-desk-booking-002` (auth/role), `ADR-desk-booking-004` (API error mapping), `ADR-desk-booking-006` (infra/roles), `ADR-desk-booking-007` (expand-contract CI).

## Context

`ADR-desk-booking-001` fixed the one decision the system is built around: a single managed PostgreSQL is the **sole owner and single enforcer** of the invariant that a given desk is held by at most one employee per date `(desk, date)`. It deferred the *physical mechanism* to this ADR and forbade any application-level check-then-act (option B, rejected as a race by construction).

This ADR must make `POST /reservations` (AC-3, AC-5, NFR-4) resolve so that under simultaneous attempts for the same desk/date, **exactly one commits (201) and every other attempt fails cleanly (409)** — with the database, not application logic, deciding the winner.

Scale is tiny and stable: ~120 users, ≤120 reservations/day, ~40k reservation rows/year, peak ≤120 requests / 30 min. The correctness mechanism must be exact; performance headroom is not the concern; over-engineering is the larger risk than throughput.

Greenfield — no existing schema/migrations/ORM. Conventions follow `standards/database-standards.md`: `snake_case`, plural tables / singular columns, `uuid` PKs via `gen_random_uuid()`, `timestamptz` never `timestamp`, `NOT NULL` by default, `text` over `varchar(n)`, no nullable booleans, no stored derived data, one logical change per migration, reversible migrations.

Four PRD open questions gate columns/constraints and are carried inline as **[NEEDS INPUT]**: FR-9 (one desk per employee per day), FR-10 (booking window), FR-13 (manager cancel/reassign), FR-15 (desk attributes / inventory ownership).

## Decision

### 1. Concurrency mechanism — the load-bearing decision

Enforce the invariant with a **partial `UNIQUE` index on `(reservation_date, desk_id)` filtered to non-cancelled rows**, and a write path that is a **single `INSERT` with no preceding `SELECT`**.

```sql
-- illustrative target schema, NOT a migration to run
CREATE UNIQUE INDEX reservations_active_desk_date_uidx
  ON reservations (reservation_date, desk_id)
  WHERE cancelled_at IS NULL;
```

Under contention, two concurrent inserts for the same `(desk, date)` attempt the same index key; Postgres serializes at the index level — first inserter holds the key, the second blocks then fails with `SQLSTATE 23505` (`unique_violation`). The backend maps `23505` on this index to a clean **409** (owned by `ADR-004`). Holds at default **`READ COMMITTED`** — the unique index enforces physically regardless of isolation, so no `SERIALIZABLE` and no retry loop.

Rules on the single writer (Booking API), for `ADR-004`:
- **No check-then-act.** Insert directly; let the database rule.
- **Do not swallow the conflict.** Plain `INSERT`, not `ON CONFLICT DO NOTHING`; the raised `23505` is the signal.
- The database is the sole judge of the winner.

The index is **partial** (`WHERE cancelled_at IS NULL`) so cancellation (AC-6) frees the desk without destroying history (NFR-8): cancelling sets `cancelled_at`, removing the row from the index; the same `(desk, date)` can be re-booked while cancelled rows are retained.

`(reservation_date, desk_id)` leads with the date so the same index also serves the dominant read — availability for a date across all desks (FR-4, AC-2, FR-14).

### 2. Logical schema (illustrative target DDL, not a migration)

```sql
CREATE TABLE users (
  id            uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
  email         text        NOT NULL,
  display_name  text        NOT NULL,
  created_at    timestamptz NOT NULL DEFAULT now(),
  updated_at    timestamptz NOT NULL DEFAULT now()
);
CREATE UNIQUE INDEX users_email_lower_uidx ON users (lower(email));

CREATE TABLE desks (
  id          uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
  code        text        NOT NULL,
  floor       smallint    NOT NULL,
  retired_at  timestamptz,
  created_at  timestamptz NOT NULL DEFAULT now(),
  updated_at  timestamptz NOT NULL DEFAULT now()
  -- [NEEDS INPUT FR-15] zone / accessibility / equipment?
);
CREATE UNIQUE INDEX desks_code_uidx ON desks (code);

CREATE TABLE reservations (
  id                uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
  desk_id           uuid        NOT NULL REFERENCES desks (id),
  user_id           uuid        NOT NULL REFERENCES users (id),
  reservation_date  date        NOT NULL,
  created_at        timestamptz NOT NULL DEFAULT now(),
  cancelled_at      timestamptz
  -- [NEEDS INPUT FR-13] cancelled_by_user_id uuid REFERENCES users(id) if managers can cancel others
);
```

- **UUIDv4 PKs** — v7's fragmentation benefit is moot at ≤120 inserts/day; native `gen_random_uuid()` needs no extra extension.
- **`cancelled_at timestamptz`** instead of a status boolean/enum — a nullable event timestamp: `IS NULL` = active, and it records *when*. A `status` column would be stored derived data (forbidden). This is the one deliberately nullable column; its meaning is fixed by the partial-index predicate.
- **`desks.retired_at`** (nullable) rather than `is_active boolean` — same reasoning; desks are retired, never hard-deleted, so reservation FKs stay valid.
- **`users.email` unique on `lower(email)`** — SSO emails must not duplicate by case.
- **No role/PII columns on `users`** — role is derived server-side (`ADR-002`); NFR-6 caps stored PII at name + email. A persisted admin flag depends on FR-13 — **[NEEDS INPUT FR-13]**.
- **`reservations` has `created_at` but no `updated_at`** — rows are immutable except the cancellation event. Revisit if FR-13 introduces reassignment.

### 3. Columns/constraints gated by open questions — [NEEDS INPUT]

- **FR-9 (one desk per employee per day, assumed no).** If confirmed, add a second partial unique index, no table change:
  `CREATE UNIQUE INDEX reservations_active_user_date_uidx ON reservations (user_id, reservation_date) WHERE cancelled_at IS NULL;` — **[NEEDS INPUT FR-9]**.
- **FR-10 (booking window).** A rolling "N days ahead" window depends on `now()` (not immutable) so it cannot be a `CHECK`; app-enforced (AC-7), likely config not a column — **[NEEDS INPUT FR-10]**.
- **FR-13 (manager cancel/reassign).** May require `reservations.cancelled_by_user_id` and/or a persisted admin flag on `users`, plus the write role's UPDATE scope — **[NEEDS INPUT FR-13]**.
- **FR-15 (desk attributes; inventory ownership).** `floor` confirmed; zone/accessibility/equipment and self-service audit columns open — **[NEEDS INPUT FR-15]**.
- **FR-7 / partial-day (PRD open q #2).** Full-day uses a `date` column + btree unique index. If partial-day is confirmed later, `reservation_date date` becomes a `tstzrange` and the index becomes `EXCLUDE USING gist (desk_id WITH =, during WITH &&) WHERE (cancelled_at IS NULL)` — a type-changing migration, flagged so the assumption is not silently baked in. Out of scope for v1.

### 4. Credential scopes — three roles, least privilege

Provisioned by infra (`ADR-006`), used by CI (`ADR-007`). All `NOSUPERUSER`, `NOCREATEDB`, `NOCREATEROLE`, TLS-only.

1. **`app_rw`** — `SELECT/INSERT/UPDATE/DELETE` on the three tables only; no DDL. The single writer of `ADR-001`.
2. **`migrator`** — DDL; owns schema objects; used only by the CI `migrate` step, never the running app, never an agent.
3. **`purge_job`** — `SELECT/DELETE` on `reservations` only, for the retention purge (NFR-8; proposed keep current + prior 90 days) — **[NEEDS INPUT: retention period, NFR-8 / open q #13]**.

Separating `app_rw` from `migrator` is what makes expand-contract safe: the app physically cannot alter schema.

### 5. Migrations: backward-compatible expand-contract (per ADR-007)

Expand (additive, nullable or `NOT NULL` with default; indexes `CONCURRENTLY` on live tables) → migrate/backfill (separate migration, batched) → contract (drop old only after all code stops referencing it). One logical change per migration; `down` written with `up`; immutable once merged; **a human runs them, never an agent**. The initial create runs against an empty DB, so plain `CREATE INDEX` in a transaction is correct there; `CONCURRENTLY` is only for indexes added later (e.g. the FR-9 index).

## Alternatives

- **App check-then-insert (ADR-001 option B).** Forbidden — race by construction.
- **Queue + single worker (option C).** Rejected — infra/latency/duplicate handling for what the index solves free.
- **`SERIALIZABLE` instead of a unique index.** Rejected — pushes `40001` retries on every writer for no benefit.
- **Advisory locks on `(desk, date)`.** Rejected — a hand-rolled lock manager duplicating the index, easy to leak.
- **Full (non-partial) unique index + hard-delete on cancel.** Rejected — destroys history, conflicts with retention.
- **`status` enum column.** Rejected — derivable from `cancelled_at`, i.e. stored derived data.
- **`EXCLUDE USING gist` on a range now.** Overkill for full-day equality; reserved as the partial-day migration path.

## Tradeoffs

- Single writer, no write scaling — accepted (ADR-001), irrelevant at this volume.
- The active/cancelled distinction lives in one predicate (`cancelled_at IS NULL`) that every read and both partial indexes must apply — mitigated by encoding it in the index predicate and centralizing it in the persistence layer.
- Plain `INSERT` + error mapping (not `ON CONFLICT`) keeps the DB authoritative; `ADR-004` should key off the index name, not a bare error code.
- UUIDv4 over v7 — negligible fragmentation at ≤40k rows/year.
- Soft-cancel retains rows — requires the `purge_job` role + retention policy to bound growth.

## Consequences

For **backend-engineer** (`ADR-004`):
1. No SELECT-then-INSERT on the booking path — insert directly.
2. Map `SQLSTATE 23505` on `reservations_active_desk_date_uidx` to HTTP 409 (AC-4, AC-5, NFR-4); do not retry or fall back to a read.
3. "Available/active" is `cancelled_at IS NULL` everywhere (FR-4, FR-11, FR-14, AC-9, AC-10).
4. Cancellation is an `UPDATE` setting `cancelled_at = now()`, not a `DELETE` (AC-6).
5. Active inventory is `desks WHERE retired_at IS NULL`.
6. Default `READ COMMITTED`; no special isolation/locking.

For **QA** (`ADR-009`): the AC-5 proof fires N simultaneous inserts at one free `(desk, date)` and asserts exactly one 201 and N−1 clean 409s.

For **infra/CI** (`ADR-006`/`ADR-007`): provision `app_rw`, `migrator`, `purge_job`; wire `migrate` to `migrator` only; schedule the purge under `purge_job`.

## Follow-up Actions

- Resolve gating open questions before the build finalizes schema: FR-9, FR-10, FR-13, FR-15.
- Confirm the NFR-8 retention period to finalize `purge_job`.
- Confirm the deployment target for concrete role provisioning (ADR-006).
- At build time, write the initial create migration + rollback per `templates/migration.md`; each open-question resolution as a separate expand-contract migration. Not done here.
- Flag to product/architect if FR-7 partial-day reopens — it changes the invariant to a gist `EXCLUDE` on ranges.

## Last verified
2026-07-31

# ADR-desk-booking-007 — CI/CD

## Status

Proposed — 2026-07-31.

## Context

Six weeks to a hard date (NFR-7) with adoption riding on correctness — regressions in the booking path are the worst possible failure. We need repeatable, low-ceremony delivery with a safe database-migration path (`architecture-principles.md`: expand then contract; deploys must tolerate mixed versions). `security.md`: migrations and background jobs get their own credentials.

## Decision

A **git-based pipeline** (assumption: GitHub Actions; equivalent CI is fine) with short-lived branches off `main`:

1. **On PR:** build, lint, unit + integration tests, including the **AC-5 concurrency test** (ADR-009) as a required gate.
2. **On merge to `main`:** build an immutable container image, deploy automatically to **staging**.
3. **To production:** a **manual approval gate**, then deploy. Rolling/replace deploy on the managed platform (ADR-006).
4. **Migrations run as a separate pipeline step** using a **dedicated migration credential** (DDL), never the app's request-path credential (CRUD only) — least privilege (ADR-008).
5. **Migrations are expand-contract and backward-compatible**, so a rolling deploy with old + new versions concurrent is always safe.
6. **Rollback:** redeploy the previous image. Backward-compatible migrations mean a code rollback needs no DB rollback.
7. **Secrets** injected from the secrets manager at deploy/runtime, never in repo or image; `.env.example` only in git.

## Alternatives

- **Auto-deploy to prod on merge (no gate).** Rejected for v1: a human gate before prod is cheap insurance in the fragile launch window; relax later.
- **Manual/scripted deploys.** Rejected: not repeatable, no automated concurrency-regression gate.
- **Coupled code+migration with the app credential.** Rejected: violates least privilege and makes rollback unsafe.

## Tradeoffs

The manual prod gate trades a little velocity for release safety during the high-risk launch window. Expand-contract adds a small per-change discipline cost for always-safe rolling deploys and code-only rollback.

## Consequences

- Two DB credentials to provision (migration DDL vs app CRUD) — coordinate with database-engineer (ADR-003) and security (ADR-008).
- The concurrency test as a merge gate makes NFR-4 a standing automated guarantee.

## Follow-up Actions

- Confirm CI platform; set up staging/prod pipelines and approval gate.
- Agree migration tooling and expand-contract checklist with database-engineer.

## Last verified
2026-07-31

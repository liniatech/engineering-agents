# ADR-desk-booking-006 — Infrastructure

## Status

Proposed — 2026-07-31.

## Context

NFR-5: internal-only, not reachable from the public internet. NFR-3: 99.5% business-hours uptime, overnight maintenance acceptable. NFR-7: ~6 weeks to production. Tiny scale (NFR-1). The org already runs Google Workspace (FR-1), so Google Cloud is the lowest-friction identity and hosting fit — stated as an assumption, not a confirmed fact.

## Decision

**Single-region, fully managed, private-ingress deployment:**

- **Compute:** a managed container service running the stateless Booking API (assumption: **GCP Cloud Run**; portable to AWS App Runner/ECS or Azure Container Apps). Stateless — any instance serves any request; sessions live in the DB, not memory.
- **Database:** a **managed PostgreSQL** (assumption: Cloud SQL) — automated backups, PITR, encryption at rest (ADR-008). Single primary; sole writer and owner of the correctness invariant.
- **Static SPA:** static assets from managed object storage/CDN behind the same private entry point.
- **Internal-only (NFR-5):** private ingress via **Identity-Aware Proxy / corporate VPN** — not on a public address. Network-layer defense; application auth (ADR-002) is defense-in-depth on top.
- **Single region.** 99.5% monthly ≈ 3.6h/month allowed downtime; a single-region managed stack with overnight maintenance meets this. No multi-region, no read replicas at launch.
- **Environments:** staging + production, same shape, separate credentials and data.

## Alternatives

- **Multi-region / HA cluster.** Rejected: engineering for an availability requirement nobody stated; threatens the 6-week timeline.
- **On-prem / self-hosted VMs.** Rejected: more ops burden, slower to stand up securely; managed services give backups, patching, TLS out of the box.
- **Public ingress + app auth only.** Rejected: NFR-5 requires not being publicly reachable.

## Tradeoffs

We give up multi-region resilience and vendor neutrality for speed, low ops cost, and simplicity. A regional outage means downtime until recovery — acceptable against a business-hours 99.5% SLO for an internal tool with a manual fallback (managers can read current bookings). Managed platform couples us to vendor primitives; re-homing is a project, a deliberate named cost.

## Consequences

- Confirm-cloud is a launch dependency (default GCP).
- Single primary DB → maintenance/upgrade implies a brief window (permitted overnight by NFR-3).
- DB failure blast radius: total outage, bounded by managed PITR/backups; RPO/RTO to be set.

## Follow-up Actions

- Confirm target cloud and that IAP/VPN internal-ingress is available.
- Set backup RPO/RTO and maintenance window against NFR-3 and business hours (open question 11).
- Confirm exact go-live date (open question 12).

## Last verified
2026-07-31

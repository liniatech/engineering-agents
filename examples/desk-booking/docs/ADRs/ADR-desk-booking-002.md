# ADR-desk-booking-002 — Authentication (Google SSO)

## Status

Proposed — 2026-07-31.

## Context

FR-1/FR-2 require sign-in exclusively through the company Google SSO; no local credentials, and unauthenticated users must see no data. FR-3 requires recording each reservation's holder identity (name + work email). NFR-6 caps stored PII at name + email. The user population is one Google Workspace domain, ~120 people. `standards/security.md`: never hand-roll session/token handling; use the framework/library primitive.

Two identities matter: **who you are** (Google) and **what you may do** (employee vs office manager, FR-14). Authentication answers only the first.

## Decision

Use **OpenID Connect Authorization Code flow with PKCE** against Google as the IdP, via a maintained OIDC library (not a hand-rolled flow).

- Restrict to the company Workspace domain: verify the `hd` (hosted-domain) claim **and** validate against a server-side allowlist. Always verify token signature and `aud`/`iss`/`exp` first — `hd` alone is spoofable otherwise.
- On successful login, mint a **server-side session** referenced by an `httpOnly`, `Secure`, `SameSite=Lax` cookie. Do not put Google tokens in the browser.
- Store only name + work email from the ID token (NFR-6). No Google refresh token persisted — no offline-access requirement today.
- **Authorization role is derived server-side**, never read from a client claim: a small allowlist (or Google Group membership) marks office managers. Default role is employee; default deny for manager-only views.
- Session lifetime bounded (proposed 12h) with idle timeout; document the value.

## Alternatives

- **Google Identity Services (client-side one-tap) only.** Rejected: no server session or object-level authorization; FR-2 and the authz model need a server session.
- **Third-party IdP/SaaS (Auth0/Okta/Cognito) fronting Google.** Rejected: added cost/vendor/time for 120 internal users when Google is already the sole IdP.
- **SAML.** Rejected: heavier than OIDC for a browser SPA + JSON API.

## Tradeoffs

We give up IdP portability — deliberately coupled to Google Workspace, which FR-1 mandates. Re-homing would be an OIDC-config change plus an ADR, not a rewrite. Server-side session state must live in a shared store (the DB), not process memory, to keep the API stateless.

## Consequences

- Every API request carries the session cookie; Auth & Session validates it and attaches server-derived identity + role. Cookie sessions require CSRF protection (ADR-008).
- No password storage/reset/rotation to build — large scope and risk reduction for a 6-week build.
- Manager membership source must be decided (allowlist vs Google Group) — depends on FR-13/manager count.

## Follow-up Actions

- Register an OAuth client in the company Google Cloud project; secret → secrets manager (ADR-008).
- Confirm office-manager membership source and count (PRD open questions 4, 5).
- Confirm session lifetime with security.

## Last verified
2026-07-31

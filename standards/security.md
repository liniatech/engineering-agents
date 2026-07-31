# Security Standards

Read by `architect`, `backend-engineer`, and `reviewer`. Every rule here is
checkable — if you cannot tell whether a diff complies, the rule is badly
written; fix the rule.

## OWASP

Treat the OWASP Top 10 as a review checklist, not background reading. For any
change touching a request path, check at minimum:

- **Injection** — SQL, command, template, LDAP. Parameterized queries only.
  String-interpolated SQL is a Critical finding, with no exceptions for
  "internal" endpoints or "trusted" input.
- **Broken access control** — the most common real vulnerability. Every new
  endpoint needs an explicit authorization check. See below.
- **Cryptographic failures** — no home-rolled crypto, no MD5/SHA1 for
  anything security-bearing, no ECB mode.
- **Insecure design** — the mitigation belongs in the design, not bolted on.
- **SSRF** — any server-side fetch of a user-supplied URL must be
  allowlisted. Blocklists do not work here.
- **Deserialization** — never `pickle.loads` untrusted data. Never
  `yaml.load` without `SafeLoader`.

## Authentication

- Never implement your own session, token, or password handling when the
  framework provides one.
- Passwords: bcrypt or Argon2 only. Never MD5, SHA-*, or unsalted anything.
- Tokens must expire. A token with no expiry is a permanent credential.
- Compare secrets with a constant-time comparison (`hmac.compare_digest`),
  never `==`.
- Authentication failures must be generic: "invalid credentials", never
  "no such user" — the difference is a user-enumeration oracle.
- Rate-limit login, password reset, and token refresh. Always.

## Authorization

**Every endpoint answers two questions: who are you, and may you do this to
*this specific object*?** Authentication answers the first. It never answers
the second.

- Default deny. Access is granted explicitly, never assumed from the absence
  of a check.
- Check authorization on the **object**, not just the route. `GET /orders/42`
  requires that this user owns order 42 — a valid session is not enough.
  This is IDOR, and it is the single most common finding in real reviews.
- Enforce on the server. A hidden UI element is not an access control.
- Re-check on every request. Never trust a client-supplied role, tenant ID,
  or user ID — derive it from the session server-side.
- In a multi-tenant system, tenant scoping belongs in the query, not in a
  post-fetch filter.

## Secrets

- Never in source. Never in a commit, including one you plan to amend — once
  pushed, treat it as compromised and **rotate it**, do not just remove it.
- Never in logs, error messages, stack traces, or analytics payloads.
- Never in a URL — query strings land in access logs and referrer headers.
- Load from environment or a secrets manager at runtime.
- Never in a client bundle, mobile app, or anything a user can read.
- `.env` files stay gitignored. Ship a `.env.example` with the keys and no
  values.

## Encryption

- TLS for everything, including internal service-to-service calls.
- Encrypt at rest for PII, credentials, and financial data.
- Use the platform primitive (AWS KMS, `cryptography`), never a hand-rolled
  scheme.
- Never invent a cipher, a key derivation, or a signature scheme.
- Key rotation must be possible without downtime — design for it up front,
  because retrofitting it means re-encrypting everything.

## Least privilege

- Database users get only the grants they need. The application should not
  connect as a superuser.
- IAM roles are scoped per service, not shared.
- Grants are additive and permanent in practice — nobody revokes. Start
  narrow.
- Background jobs and migrations get their own credentials, distinct from
  the request path.

## Rate limiting

- Every public endpoint has a limit.
- Stricter limits on: login, registration, password reset, search, and
  anything that sends an email or SMS, triggers a webhook, or costs money
  per call.
- Limit per user *and* per IP. Either alone is trivially evaded.
- Return `429` with `Retry-After`.
- Expensive endpoints need a limit even when authenticated.

## Input validation

- Validate at the trust boundary, on arrival. Never deep inside the call
  stack, where it may be bypassed by another caller.
- Allowlist, do not blocklist. Enumerate what is permitted; reject the rest.
- Validate type, range, length, and format. An unbounded string field is a
  memory and storage risk.
- Bound every collection — a request that accepts a list must cap its length.
- Validate on the server even when the client already did.
- Encode on output, contextually: HTML-escape for HTML, parameterize for SQL,
  quote for shell. Validation on input and encoding on output are different
  defenses; you need both.

## Review triggers

Escalate to a human security reviewer for any change that touches:
authentication, authorization logic, cryptography, payment flows, PII
handling, file upload, or deserialization of external data.

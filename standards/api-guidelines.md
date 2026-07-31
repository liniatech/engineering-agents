# API Guidelines

Read by `architect`, `backend-engineer`, and `reviewer`.

An API is a contract you cannot unilaterally break. Everything here follows
from that.

## REST first

Default to REST. Deviate only with a stated reason in an ADR.

- **Resources are nouns, plural.** `/orders`, `/users/42/addresses`.
  Never `/getOrder`, `/createUser` — the verb is the HTTP method.
- **Methods mean what they mean:**

  | Method | Semantics | Safe | Idempotent |
  |---|---|---|---|
  | GET | read | yes | yes |
  | POST | create / non-idempotent action | no | no |
  | PUT | full replace | no | yes |
  | PATCH | partial update | no | no |
  | DELETE | remove | no | yes |

  A `GET` that mutates state is a defect, not a shortcut. Crawlers,
  prefetchers, and retries will find it.

- **Status codes carry meaning.** `200` ok, `201` created (with `Location`),
  `202` accepted-async, `204` no content, `400` malformed, `401`
  unauthenticated, `403` unauthorized, `404` not found, `409` conflict,
  `422` semantically invalid, `429` rate limited, `5xx` our fault.
  Never return `200` with an error in the body.
- **Nest at most one level.** `/users/42/orders` is fine;
  `/users/42/orders/7/items/3` is not — use `/order-items/3`.
- Use GraphQL or gRPC where they genuinely fit; record the decision.

## Consistent error responses

One shape for every error across every endpoint. Clients write error handling
once.

```json
{
  "error": {
    "code": "insufficient_funds",
    "message": "Account balance is below the transfer amount.",
    "details": [
      { "field": "amount", "issue": "exceeds available balance" }
    ],
    "request_id": "req_01H8X..."
  }
}
```

- `code` is a stable machine-readable string. Clients branch on it, so
  changing it is a breaking change.
- `message` is for humans, and must never leak internals — no stack traces,
  no SQL, no file paths, no upstream hostnames.
- `request_id` on every error. Without it, support tickets are unanswerable.
- Validation errors report **all** failures at once, not the first.
- Never leak existence through error differences: an unauthorized read of a
  real object and of a nonexistent one should both be `404`, or both `403`.
  Consistently.

## Pagination

**Every collection endpoint is paginated.** No exceptions — the endpoint that
returns "just a few" rows today returns fifty thousand next year.

Prefer cursor pagination:

```
GET /orders?limit=50&cursor=eyJpZCI6...
{ "data": [...], "next_cursor": "eyJpZCI6...", "has_more": true }
```

- Cursor over offset: offset drifts when rows are inserted mid-scan, and gets
  slower the deeper you page.
- Default limit if unspecified. **Hard maximum**, enforced server-side.
- Offset pagination is acceptable only for small, static, ordered sets.
- Total counts are expensive at scale — make them optional, not default.

## Filtering

- Query parameters, explicitly allowlisted: `?status=pending&created_after=…`
- **Never** accept raw query fragments or arbitrary field names from the
  client — that is injection with extra steps.
- Every filterable field must be indexed, or it is a denial-of-service vector.
- Sorting: `?sort=-created_at` (leading `-` for descending). Allowlist the
  sortable fields too.
- Document exactly which fields are filterable. Undocumented filters become
  load-bearing.

## Versioning

- Version in the URL path: `/v1/orders`. Visible, cacheable, unambiguous.
- **Additive changes do not need a version bump**: new optional field, new
  endpoint, new optional parameter.
- **Breaking changes always do**: removing or renaming a field, changing a
  type, making an optional field required, changing an error code, changing
  default behavior.
- Clients must tolerate unknown fields. Say so in the docs — then adding a
  field stays non-breaking.
- Running two versions is expensive. Announce deprecation with a date, emit
  a `Deprecation` header, monitor which clients still call the old version,
  and do not remove it until that number is zero.

## Authentication

- Every endpoint is authenticated unless deliberately public — and public
  endpoints are listed explicitly somewhere a reviewer can check.
- `Authorization: Bearer <token>`. Never a token in a query string; those
  land in access logs and referrer headers.
- Authentication is not authorization. See `standards/security.md` — check
  the caller may act on *this object*, not merely that they are logged in.
- `401` means "who are you"; `403` means "not allowed". They are different
  and clients act on them differently.
- Tokens expire. Document the lifetime and the refresh flow.

## Idempotency

Networks retry. Assume every request may arrive twice.

- `GET`, `PUT`, `DELETE` must be naturally idempotent.
- **`POST` that creates or costs money requires an idempotency key:**

  ```
  POST /payments
  Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
  ```

  Store the key with the result. A replay returns the original response
  rather than performing the action twice.

- Retain keys long enough to outlive any client retry window — 24 hours is a
  common floor.
- A duplicate key with a *different* payload is a client bug: return `409`.
- Consumers of async events must be idempotent too; at-least-once delivery
  means duplicates are guaranteed, not hypothetical.

## Also

- **Timestamps** ISO 8601, UTC, with offset: `2026-07-31T14:22:00Z`.
- **Field naming** consistent — pick `snake_case` or `camelCase` and never
  mix within an API.
- **Nulls** distinguish "absent" from "explicitly null" in PATCH semantics,
  and document which you mean.
- **Long operations** return `202` with a status URL, not a held connection.
- **Rate limits** returned as headers so clients can self-throttle.

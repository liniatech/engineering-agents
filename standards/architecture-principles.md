# Architecture Principles

Read by `li-architect` and `li-reviewer`.

These are defaults, not laws. Each may be overridden — in an ADR, with the
reason and the tradeoff written down. An unrecorded deviation is the thing
these principles exist to prevent.

## Prefer modularity

A module is a boundary with a purpose, not a folder.

- Each module has one reason to change. If two unrelated requirements both
  force edits here, it is two modules.
- The public surface is small and deliberate. Everything else is internal.
- Depend on interfaces, not implementations, **where the implementation is
  genuinely likely to change.** An interface with exactly one implementation
  and no prospect of a second is indirection, not abstraction.
- Modules within a service are cheap and reversible. Services are neither.
  **Reach for a module first.** A distributed system built to avoid a
  refactor is the most expensive mistake in this document.

## Prefer stateless services

- No in-memory state that a request depends on. Any instance must be able to
  serve any request.
- Session state goes to a shared store, not to process memory.
- Never rely on sticky sessions for correctness.
- Stateless services scale horizontally, restart safely, and deploy without
  drain logic. State belongs in the database, the cache, or the queue —
  places designed for it.
- When state must be local (a connection pool, a warm cache), it must be
  reconstructable and never authoritative.

## Minimize coupling

Coupling is what makes a change in one place break another. Ranked from
least to most damaging:

1. **Data coupling** — passing arguments. Fine.
2. **Contract coupling** — calling a versioned API. Manageable.
3. **Temporal coupling** — A must be up for B to work. Costly; every
   synchronous hop multiplies your failure probability.
4. **Shared-database coupling** — two services writing the same table. This
   is the worst kind: invisible in the code, unversioned, and it turns two
   services into one system that cannot be deployed independently. Avoid it.

Rules:

- One writer per table. Others read via an API or an event.
- Prefer asynchronous messaging where the caller does not need the answer now.
- Trace your synchronous chains. Five services at 99.9% each, called in
  series, is 99.5% — and that is your ceiling, before any bug.
- Never share internal models across a service boundary. Publish a contract
  and map to it.

## Maximize cohesion

- Things that change together live together. This is the strongest signal
  for where a boundary belongs.
- Organize by domain, not by technical layer. `orders/` containing its
  handler, logic, and persistence beats `controllers/`, `services/`, and
  `repositories/` each containing a slice of everything.
- If a single feature change requires edits in five directories, your
  boundaries are drawn along the wrong axis.
- Beware boundaries that mirror the org chart rather than the domain — they
  outlive the org chart.

## Design for observability

If you cannot tell from outside that it is working, it is not operable.

- **Structured logs** — key-value, not prose. Include a correlation ID on
  every entry so a request can be reconstructed across services.
- **Metrics** — at minimum rate, errors, and duration for every endpoint and
  every dependency call.
- **Traces** across service boundaries. In a distributed system this is the
  only way to answer "where did the time go".
- Name the **one metric that goes bad first** when this component fails, and
  alert on it. Alert on user-visible symptoms, not on causes.
- Health checks distinguish "alive" from "ready to serve".
- Emit business events, not just technical ones — "payment failed" is more
  actionable than "500 on POST /payments".

## APIs are contracts

- Once published, it is not yours to change unilaterally.
- Additive changes are safe; removals and semantic changes are not. See
  `standards/api-guidelines.md`.
- The contract includes behavior, not just shape: error codes, ordering,
  nullability, idempotency, and timing guarantees are all part of it.
- Consumers depend on what you actually do, not what you documented. Undefined
  behavior becomes a contract the moment someone relies on it.
- Contract tests at the boundary. Unit tests on both sides will happily agree
  with each other and be jointly wrong.

## Favor backward compatibility

- Expand, then contract. Add the new thing, migrate consumers, remove the old
  thing — three deploys, never one.
- Deploy must tolerate mixed versions running concurrently. During any rolling
  deploy, both versions are live simultaneously.
- Deprecate with a date, a migration path, and monitoring of remaining callers.
- A breaking change is sometimes correct. Make it deliberately, announce it,
  and own the migration — do not let it happen by accident.

## Use ADRs for significant decisions

Write an ADR when a decision is **expensive to reverse**: choosing a
datastore, defining a service boundary, adopting a framework, selecting a
protocol, deciding data ownership.

Do not write one for a decision a single PR could undo.

- Use `templates/adr.md`.
- Record the alternatives honestly. Two straw men and a favorite is not a
  record of a decision; it is a justification written afterwards.
- Record what you **gave up**. Every real architectural decision trades
  something away. If you cannot name it, you have not made a decision — you
  have expressed a preference.
- ADRs are immutable. Superseding one writes a new ADR that links back; it
  never edits the original. The wrong decisions are the valuable record.

## Two failure directions

Reviewers reliably catch one and miss the other.

- **Under-engineering** — no failure handling, no migration path, a one-way
  door walked through casually. Easy to spot.
- **Over-engineering** — abstraction for a requirement nobody has, a service
  where a module would do, configurability nobody asked for, a plugin system
  with one plugin. Harder to spot, because it looks like diligence.

The question that separates them: *what specific, named requirement does this
serve?* "Flexibility" is not an answer.

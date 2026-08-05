# Coding Standards

Read by `li-backend-engineer`, `li-reviewer`, and `li-qa`. Python-flavored; the
principles are language-independent.

**The rule above all others:** match the code around you. A change written in
your preferred style inside a codebase written in another is harder to review
and harder to maintain than a change that is locally consistent but not your
favorite. Consistency beats preference, always.

## General

### Readability over cleverness

Code is read far more often than it is written, usually by someone with less
context than the author had — including the author, later.

- If a line takes a second read to parse, rewrite it.
- A nested comprehension that needs a comment should be a loop.
- Being able to name the intermediate step is worth the extra line.

```python
# Clever
r = [f(x) for x in (y for y in items if y.ok) if f(x) is not None]

# Readable
valid = [item for item in items if item.ok]
results = [transform(item) for item in valid]
return [r for r in results if r is not None]
```

### Explicit is better than implicit

- No magic. No implicit global state, no action-at-a-distance, no behavior
  that depends on import order.
- Pass dependencies in; do not reach out for them. It makes the coupling
  visible and the code testable.
- Name your constants. A bare `86400` in a conditional is a question; a
  `SESSION_TTL_SECONDS` is an answer.
- Avoid boolean parameters at call sites — `send(user, True)` is unreadable.
  Use a keyword argument or an enum.

### Prefer composition over inheritance

- Inheritance couples you to a parent's implementation forever, and the
  coupling is invisible at the call site.
- Depth beyond two levels is almost always a mistake.
- Inherit for genuine "is-a" substitutability. Compose for "has-a" or
  "uses-a" — which is nearly always what you actually mean.
- Mixins that reach into the host class's attributes are inheritance with
  extra steps and worse discoverability.

### Keep functions under 40 lines when practical

Length is a symptom, not the disease. A long function usually means several
responsibilities sharing a scope.

- If you need a comment to mark a section, that section is a function.
- Watch the indentation depth more than the line count. Three levels of
  nesting is a warning; four is a defect.
- Use early returns to flatten. Guard clauses at the top, the real work
  unindented below.

```python
# Nested
def process(order):
    if order:
        if order.is_valid():
            if order.items:
                return charge(order)

# Flat
def process(order):
    if not order:
        return None
    if not order.is_valid():
        raise InvalidOrder(order.id)
    if not order.items:
        return None
    return charge(order)
```

### One responsibility per class

- One reason to change. If two unrelated requirements both force edits here,
  it is two classes.
- A class whose name contains "Manager", "Handler", "Processor", or "Utils"
  is often several classes that have not been named yet.
- Prefer a module of functions over a class with no state. A class with one
  method and no attributes is a function.

## Naming

### Variables should explain intent

- The name says what the value *means*, not what type it is.
  `pending_orders`, not `order_list`.
- Scope earns length: a two-line comprehension can use `x`; a
  fifty-line function cannot.
- Booleans read as assertions: `is_active`, `has_permission`,
  `should_retry`.
- Avoid negated names. `not is_disabled` is a puzzle; `is_enabled` is not.

### Avoid abbreviations

- `user_repository`, not `usr_repo`. Typing is cheap; misreading is not.
- Universally understood ones are fine: `id`, `url`, `http`, `db`, `api`.
- Never invent an abbreviation. If it is not already common in this
  codebase, spell it out.

### Functions should be verbs

- `calculate_total()`, `send_invoice()`, `validate_address()`.
- Predicates read as questions: `is_expired()`, `has_access()`.
- The name states the **whole** effect. A function called `get_user` that
  also writes an audit row is misnamed, and the surprise will cause a bug.
- Same concept, same word, everywhere. Do not mix `fetch`, `get`, `load`,
  and `retrieve` for the same operation.

### Classes should be nouns

- `Invoice`, `PaymentGateway`, `OrderValidator`.
- Domain language, not technical language. Name it what the business calls
  it — that is the vocabulary the requirements arrive in.

## Error Handling

### Never silently ignore exceptions

```python
# Never
try:
    charge(order)
except Exception:
    pass
```

- Never a bare `except:` — it catches `KeyboardInterrupt` and `SystemExit`.
- Catch the narrowest exception that can actually occur.
- If you genuinely intend to continue, log it and write down why.
- Do not catch what you cannot handle. Letting it propagate to a boundary
  that logs it with context beats swallowing it here.

### Raise domain-specific exceptions

- Define an exception hierarchy in domain terms:
  `InsufficientFunds(PaymentError)`. Callers can then handle the category or
  the specific case.
- Never raise bare `Exception` or `ValueError` for a domain condition —
  callers cannot distinguish it from a bug.
- Translate at boundaries: a database error should not reach the HTTP layer
  as a database error.
- Preserve the cause: `raise PaymentError(...) from exc`.

### Include actionable error messages

- Say what failed, with which values, and what to do about it.
- `"Transfer of 500.00 exceeds available balance of 320.50 on account
  acc_8812"` — not `"invalid amount"`.
- Never leak internals to a user-facing message: no stack traces, no SQL,
  no file paths, no hostnames. Log those; return a `request_id`.

## Logging

### Log meaningful business events

- Log what you would want during an incident at 2am, not what was convenient
  to log while developing.
- Structured, key-value, machine-parseable — not interpolated prose.
- Every log line in a request carries the correlation ID.
- Levels mean something: `ERROR` needs a human, `WARNING` is degraded but
  handled, `INFO` is a business event, `DEBUG` is off in production.
- Do not log inside tight loops. Log the aggregate.

```python
logger.info("payment_captured", extra={
    "order_id": order.id, "amount": amount, "gateway": "stripe",
    "request_id": ctx.request_id,
})
```

### Never log secrets

- No passwords, tokens, API keys, card numbers, or session IDs.
- No full request or response bodies on authenticated endpoints — they
  contain PII by default.
- Log identifiers, not payloads. `user_id`, not the user object.
- Redact at the logger, not at each call site. Relying on every developer to
  remember will fail exactly once, permanently, in production.

## Comments

Comments explain **why**, not what.

- The code says what it does. A comment restating it is noise that drifts out
  of sync and then actively misleads.
- Comment the non-obvious: a workaround and the bug it works around, a
  deliberate deviation from the standard, a constraint that is not visible
  locally, a reason the simple approach fails.

```python
# Bad
# increment the counter
counter += 1

# Good
# Stripe's API returns 200 with an empty body on a duplicate charge rather
# than 409, so an empty response here means the idempotency key already
# fired. See PROJ-1180.
if not response.content:
    return previously_recorded_result(key)
```

- Delete commented-out code. Git remembers it.
- A `TODO` without a ticket reference is a wish. `# TODO(PROJ-482): ...`
- Docstrings on public functions: purpose, parameters, return, and what it
  raises. Skip them on obvious private helpers — a docstring restating the
  name is noise.

## Testing

Every feature should include tests. See `standards/testing-standards.md` for
what and how.

The minimum bar for any change:

- The new behavior has a test that would fail without the change.
- Every new conditional branch is exercised.
- A bug fix has a test that fails on the old code — that is the proof you
  found the actual cause.
- The suite was **run**, and the output read. Not assumed.

## Type hints

- Type hints on all public functions and all module-level functions.
- Run the type checker in CI. A type hint nobody verifies is a comment that
  looks authoritative.
- Prefer precise types: `list[Order]` over `list`, `Decimal` over `float` for
  money.
- `Any` is an escape hatch that needs a reason.

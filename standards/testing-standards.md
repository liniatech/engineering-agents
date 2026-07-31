# Testing Standards

Read by `backend-engineer` and `qa`.

A test suite exists to let you change code with confidence. A suite that
passes while the system is broken is worse than none, because it spends
trust it has not earned.

## Levels

Most tests should be fast and narrow; a few should be slow and broad.

### Unit

- One unit of behavior, no I/O — no database, no network, no filesystem.
- Milliseconds. Hundreds of them run in seconds.
- Test the **public behavior** of the unit, not its internals. A test that
  breaks on every refactor while the behavior is unchanged is measuring the
  wrong thing.
- Mock only what crosses a boundary you own. Mocking everything produces a
  test that verifies your mocks agree with each other.

### Integration

- Real database, real queries, real transactions.
- This is where ORM mistakes, migration errors, constraint violations, and
  N+1 queries actually surface. Unit tests cannot see any of them.
- Use a real Postgres, not SQLite. They differ in the ways that matter.
- Each test sets up and tears down its own data. No shared fixtures that
  tests mutate — that is how ordering dependencies are born.

### Contract

- Verifies that a service's API matches what its consumers expect.
- Necessary because unit tests on both sides of a boundary will happily agree
  with each other and be jointly wrong.
- Test the contract, not the implementation: shape, status codes, error
  codes, nullability.
- Run against every published version still in support.

### E2E

- A real user path through the real stack.
- Slow, brittle, expensive. **Keep very few.** Cover the two or three flows
  where failure is unacceptable — signup, checkout, login.
- When an E2E test fails, it tells you *something* is broken but rarely
  *what*. That is the cost. Do not use it for coverage.

## Coverage

### Critical business logic must be tested

Not "everything must be tested". Rank by consequence of failure:

1. **Must** — money, auth, permissions, data integrity, anything
   irreversible
2. **Should** — core user flows, complex conditionals, error handling
3. **Optional** — glue code, simple accessors, framework boilerplate

Coverage percentage is a weak proxy. An untested branch on the payment path
matters more than fifty untested getters, and both count the same in the
number.

**Test the edges, not just the happy path:**

- Empty, zero, one, many, maximum
- Boundaries — and one either side of each
- Null and absent, which are different
- Duplicate submission and replay
- Concurrency — two writers, same row
- Partial failure — the downstream call that times out mid-transaction
- Permission denied

The happy path is what you thought of. Bugs live in what you did not.

### Tests should be deterministic

A flaky test is worse than a missing one: it trains everyone to ignore red.

- **No wall-clock dependence.** Inject the clock. A test that fails at
  midnight, or in another timezone, or on Feb 29, is a real defect in the
  test.
- **No real network.** Ever.
- **No ordering dependence.** Each test passes alone and in any order.
  Randomize the order in CI to enforce it.
- **No shared mutable state** between tests.
- **No `sleep`.** Poll for the condition, or inject the scheduler.
- **No randomness** without a fixed seed.

A flaky test is fixed or deleted. It is never retried into green — a retry
wrapper is a way of not knowing whether your system works.

## Writing tests

- **Know what bug the assertion catches before you write it.** A test that
  cannot fail proves nothing and costs maintenance forever.
- Name the behavior, not the function:
  `test_transfer_rejects_amount_above_balance`, not `test_transfer_2`.
- Arrange / Act / Assert, in that order, visibly.
- One behavior per test. A test with six assertions about five things
  reports one failure and hides the rest.
- Assert on the outcome, not the mechanism. `assert account.balance == 50`,
  not `assert mock_db.update.called_once()`.
- Prefer a real object over a mock when it is cheap to construct.

## Test-first, where it fits

- For a **bug fix** it is mandatory: write the test that fails because of
  the bug, watch it fail for the right reason, then fix. If you cannot write
  a failing test, you have not found the root cause.
- For a **feature**, write the test first where the behavior is clear. Where
  you are still exploring the shape, write it immediately after — not
  "later", which means never.

## Never

- Never weaken an assertion to make a test pass.
- Never delete or `skip` a failing test to unblock a merge. A skipped test
  with no ticket is deleted coverage that still looks like coverage.
- Never change production code to satisfy a test without understanding which
  one is actually wrong.
- Never report the suite as passing without having run it and read the
  output.
- Never let a test depend on a test that ran before it.

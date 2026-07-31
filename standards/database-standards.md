# Database Standards

Read by `database-engineer`. PostgreSQL.

The schema outlives the application code. Decisions here are the most
expensive to reverse in the system.

## Use UUID primary keys unless justified

- Default to `uuid` with `gen_random_uuid()`.
- Why: IDs can be generated client-side and before insert, they do not leak
  volume ("we have 1,847 customers"), and they do not collide when merging
  data across environments or shards.
- **Use `uuid`, never a UUID stored as `text`** — 16 bytes versus 37, and the
  text form compares wrong.
- Prefer UUIDv7 where available: time-ordered, so it avoids the index
  fragmentation that random v4 causes on insert-heavy tables.
- **Justified exceptions:** very high-volume append-only tables where 8 bytes
  versus 16 matters at scale; tables whose natural key is genuinely stable
  and unique. Write the justification in the migration doc.

## Every FK should have an index

Postgres creates an index for the **primary** key, not for foreign keys
pointing at it. This is the most common performance defect in a schema.

- Index every foreign key column.
- Without it, deleting or updating the parent row scans the whole child
  table to check the constraint.
- Composite indexes: column order matters. `(user_id, created_at)` serves
  `WHERE user_id = ?` and `WHERE user_id = ? ORDER BY created_at` — the
  reverse order serves neither well.
- **State the query each index serves.** An index with no named query is a
  guess. Indexes are not free: they slow every write and consume storage.
- Drop unused indexes. `pg_stat_user_indexes` shows which have never been
  scanned.
- On a live table, always `CREATE INDEX CONCURRENTLY` — the plain form takes
  a write lock for the duration of the build.

## Prefer NOT NULL

- Default to `NOT NULL`. Add nullability deliberately, with a reason.
- A nullable column pushes a decision onto every reader, forever. Every query,
  every serializer, every consumer must decide what null means — and they
  will each decide differently.
- Give a default where a sensible one exists.
- Null means "unknown" or "not applicable". It does not mean zero, empty
  string, or false. If you find yourself writing `COALESCE(x, 0)` everywhere,
  the column should have been `NOT NULL DEFAULT 0`.
- Adding `NOT NULL` to an existing column requires a backfill first, then
  the constraint added `NOT VALID` and validated separately — see the
  `database-change` skill.

## Avoid nullable booleans

A nullable boolean is a three-state field wearing a two-state type. Every
reader must handle `true`, `false`, and `null`, and they will disagree about
what `null` means.

- If there really are three states, use an enum: `pending | approved |
  rejected`. Now the third state has a name.
- If it is "has this happened", use a nullable timestamp instead:
  `approved_at`. It answers the boolean question *and* records when.
  Strictly more information for the same cost.
- If it is genuinely binary, make it `NOT NULL DEFAULT false`.

## Never store derived data unnecessarily

Derived data is a cache, and caches go stale.

- Compute it in the query unless you have measured that you cannot.
- Denormalize only with: a measured performance reason, a named refresh
  mechanism, and a way to detect drift.
- Prefer a materialized view or generated column over an application-managed
  duplicate — the database keeps them honest; your code will not.
- When you do denormalize, write down what recomputes it and how you would
  detect a mismatch.
- Immutable snapshots are **not** derived data: the price on a historical
  order line is a fact about that order, not a duplicate of the product price.
  Store it.

## Migrations must be reversible

- Write the `down` migration when you write the `up`. Not later.
- Test the rollback before shipping.
- Where a rollback genuinely loses data (dropping a populated column), say so
  explicitly in the migration doc and use expand/contract instead of a
  single-step change.
- **One logical change per migration.** Debugging a partially-applied
  multi-purpose migration at 2am is avoidable.
- Migrations are immutable once merged. Never edit one that has run anywhere;
  write a new one.
- Never mix schema changes and data backfills in one migration — they have
  different lock profiles and different failure modes.
- Migrations run by humans, never by agents.

## Lock awareness

The statements that cause outages, on a large table:

| Operation | Risk |
|---|---|
| `ADD COLUMN` nullable, no default | Safe — metadata only |
| `ADD COLUMN NOT NULL DEFAULT` | Safe on PG 11+; full rewrite before that |
| `ALTER COLUMN TYPE` | Full rewrite, `ACCESS EXCLUSIVE` — avoid |
| `ADD CONSTRAINT` | Scans the table — use `NOT VALID`, then `VALIDATE` |
| `ADD FOREIGN KEY` | Locks **both** tables |
| `CREATE INDEX` | Blocks writes — use `CONCURRENTLY` |
| `DROP COLUMN` | Fast, but breaks any code still selecting it |

Always know the table's realistic row count before choosing an approach. The
safe path for 10k rows and for 100M rows are different.

## Naming and types

- `snake_case`. Plural table names, singular column names.
- Foreign keys: `<referenced_table_singular>_id`.
- Timestamps: `created_at`, `updated_at`, and `<verb>_ed_at` for events.
  Always `timestamptz`, never `timestamp` — naive timestamps are a bug
  waiting for a deploy in another region.
- Money: `numeric`, never `float`. Floating point cannot represent 0.10.
- Prefer `text` over `varchar(n)` unless the limit is a real business rule.
  In Postgres there is no performance difference, and the length limit only
  becomes a painful migration later.
- Follow whatever convention the existing schema already uses. Consistency
  beats preference.

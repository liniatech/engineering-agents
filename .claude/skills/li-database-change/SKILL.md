---
name: li-database-change
description: Safe path for any schema change — design, migration, lock analysis, expand/contract sequencing, rollback. Use when adding or altering tables or columns, adding indexes, writing a migration, or backfilling data. Migrations run by humans, never by agents.
---

# Database change

Schema changes are the class of change most likely to cause an outage and
least likely to be reversible. The pipeline is built around that.

## Steps

### 1. Understand what exists

Find the current schema before designing anything: migration directory, ORM
models, `schema.sql`. Identify the table's realistic row count — the safe
approach for 10k rows and 100M rows are not the same, and you cannot reason
about lock risk without it. If you do not know, ask.

### 2. Design

Spawn `li-database-engineer` with the requirement and the current schema.

It owns this decision. `li-backend-engineer` may not write migrations.

### 3. Classify the risk

Sort the change honestly:

**Safe online** — new table; new nullable column; `CREATE INDEX CONCURRENTLY`;
new constraint added `NOT VALID` then validated separately.

**Needs care** — new `NOT NULL` column with default (version-dependent
rewrite); type change; adding a foreign key (locks both tables); any
backfill over a large table.

**Breaking** — dropping or renaming a column or table still referenced by
running code. **These require expand/contract.** Never do them in one step.

### 4. Expand/contract, when breaking

Never rename. Never drop in the same deploy as the code change. Sequence:

1. **Expand** — add the new column/table. Deploy.
2. **Dual-write** — application writes both old and new. Deploy. Let it run.
3. **Backfill** — migrate historical rows in batches. Verify counts match.
4. **Switch reads** — application reads the new. Deploy. Watch.
5. **Contract** — drop the old, only after the previous deploy has been
   stable and no rollback to it is plausible.

Each numbered step is a separate deploy. Compressing them is exactly how
schema changes cause outages.

### 5. Rollback

Write it. Test it. If a rollback is genuinely impossible, state that in
writing and describe the forward-fix instead — "we cannot roll this back" is
an acceptable answer only when it is said out loud in advance.

### 6. Application changes

Spawn `li-backend-engineer` with the migration doc. Application code must
tolerate both the pre- and post-migration schema during the deploy window.

### 7. Review

Spawn a fresh `li-reviewer`. Have it specifically check: lock duration, rollback
correctness, whether app code survives both schema states, and index
justification.

### 8. Hand to a human

Present the migration for a human to run. Include:

- The exact statements, in order
- Expected duration and lock behavior per statement
- The rollback
- What to watch during and after
- The abort criteria — the signal that means stop

## Never

- Never run a migration against any database. Agents write; humans run.
- Never combine a breaking schema change with an application change in one
  deploy.
- Never `CREATE INDEX` without `CONCURRENTLY` on a live table.
- Never backfill in a single unbatched `UPDATE` on a large table.

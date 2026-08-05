# Git Standards

Read by `li-reviewer`. The goal of every rule here is the same: make the history
answer "why does this line exist?" a year from now.

## Branch naming

```
<type>/<short-description>
<type>/<ticket-id>-<short-description>
```

Types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `perf`, `hotfix`.

- Lowercase, hyphen-separated.
- Short enough to type, specific enough to identify.
- One concern per branch. If the name needs "and", split the branch.

```
feat/rate-limit-public-api
fix/PROJ-482-duplicate-webhook-delivery
hotfix/expired-signing-cert
```

Not `alini-work`, `fixes`, `new-branch`.

## Commit style

Conventional Commits:

```
<type>(<scope>): <subject>

<body — why, not what>

<footer — refs, breaking changes>
```

**Subject**

- Imperative mood: "add", not "added" or "adds". It completes the sentence
  "if applied, this commit will…"
- 72 characters or less, no trailing period.
- Say what changed in user or system terms, not file terms. `fix(auth):
  reject expired refresh tokens` — not `fix: update auth.py`.

**Body**

- Explain **why**. The diff already shows what.
- Note anything non-obvious: a constraint you worked around, an alternative
  you rejected, a reason the simple approach does not work.
- Wrap at 72.

**Footer**

- `Refs: PROJ-482`
- `BREAKING CHANGE: <what breaks and what callers must do>`

**Rules**

- One logical change per commit. A commit that needs "and" should be two.
- Never commit commented-out code, debug prints, or `.env` files.
- Never commit a secret. If you do, rotate it — removing it from history is
  not sufficient, because it was pushed.
- Never mix a refactor with a behavior change. Reviewers cannot see the
  behavior change inside a large reformat, so it ships unreviewed.

## PR checklist

Before requesting review:

- [ ] Branch rebased on the current base
- [ ] Full test suite passes locally — output seen, not assumed
- [ ] Linter and type checker pass
- [ ] No debug code, no `TODO` without a ticket reference
- [ ] Migrations reversible, or irreversibility documented
- [ ] No secrets in the diff
- [ ] Self-reviewed the diff as if someone else wrote it

**PR description**

- What changed, and **why**
- Link to the ticket, PRD, or ADR
- How it was tested
- Deploy notes: migrations, config, feature flags, required ordering
- Rollback plan, if not simply "revert"
- Screenshots for anything user-visible

**Size**

Under 400 changed lines where possible. Review quality falls off a cliff
above that, and an approval on a 2000-line PR mostly means nobody read it.
Split by: refactor first, then behavior. Mechanical change first, then the
interesting part.

## Merge strategy

- **Squash merge** for feature branches — one clean commit on the base
  branch. The messy work-in-progress history stays in the PR.
- **Rebase** to update a branch, not merge. Keeps history linear and the
  diff readable.
- **Merge commits** only for release branches, where the topology matters.
- Never force-push a shared branch. Never force-push after review has
  started — it destroys the reviewer's diff. `--force-with-lease` if you
  must force-push your own branch.
- Base branch stays green. A red base blocks everyone.
- Delete the branch after merge.

## Release tagging

Semantic versioning: `vMAJOR.MINOR.PATCH`

- **MAJOR** — breaking change to a public contract
- **MINOR** — backward-compatible functionality
- **PATCH** — backward-compatible fix

- Annotated tags only (`git tag -a`) — they carry author, date, and message.
- Tag the exact commit that was deployed, not a nearby one.
- Tag message contains the release summary, or links to the release notes.
- Never move or delete a published tag. Cut a new one.
- Prereleases: `v2.1.0-rc.1`.

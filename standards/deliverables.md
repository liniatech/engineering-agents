# Deliverable Standards

Read by every skill that produces a report, an analysis, a ticket, or a
review. This standard exists because a report that only ever appeared in a
terminal did not happen — the next person cannot read it, diff it, or link
to it.

## The rule

**Every report, analysis, task detail, or review is written to a `.md` file
before it is shown in chat.**

Not "written down" as prose. Written to disk, with the `Write` tool, at the
path the skill names. Then — and only then — print the short summary in chat,
and end it with the file path.

Chat output is a **pointer to the deliverable, never the deliverable.** If the
file was not created, the step is not done, regardless of how complete the
chat output looks.

## When it applies

Any time the work produces a durable finding:

- a bug ticket or a diagnosis
- a code review, architecture review, or design review
- a performance measurement or analysis
- a test-suite repair report
- a dependency upgrade report
- a release checklist or release notes
- a postmortem
- a task breakdown or task detail
- a PRD, ADR, SDD, or migration doc

It does **not** apply to conversational answers, one-line status replies, or a
question the user asked in passing. If in doubt: the user asked for a *report*,
an *analysis*, a *review*, or a *ticket* → write the file.

## Where it goes

Each skill names its own path. The tree:

```
docs/bugs/<slug>.md                             bug tickets and diagnoses
docs/reviews/<date>-<slug>-code-review.md       code reviews
docs/reviews/<date>-<slug>-architecture.md      architecture / design reviews
docs/performance/<date>-<slug>.md               performance analyses
docs/reports/<date>-<slug>-test-repair.md       test-suite repairs
docs/reports/<date>-<slug>-upgrade.md           dependency upgrades
docs/releases/<version>.md                      release checklist + notes
docs/incidents/<date>-<slug>.md                 postmortems
docs/database/<slug>.md                         schema changes and migrations
docs/features/<slug>/{prd,adr,migration}.md     feature pipeline artifacts
docs/scoping/ · docs/ADRs/ · docs/plan/         project kickoff artifacts
```

`<date>` is today, `YYYY-MM-DD` — get it from `date +%F`, never from memory.

`<slug>` is short, lowercase, hyphenated, and names the *subject*, not the
activity: `webhook-drops-events`, not `review-of-my-changes`.

Create the directory if it does not exist (`mkdir -p`). A missing directory is
not a reason to skip the file.

## Every deliverable opens with

```markdown
---
title: <one line, the same title used in chat>
type: bug | code-review | architecture-review | performance | test-repair | upgrade | release | postmortem | schema
date: <YYYY-MM-DD>
status: draft | open | resolved | approved | changes-requested
scope: <what was examined — diff range, service, endpoint, version>
---
```

Then the body, in the shape the skill defines, or the matching
`templates/*.md` where one exists.

## Writing it

- **Use the template if one exists.** `templates/` holds the durable shapes:
  `bug-report.md`, `code-review.md`, `design-review.md`,
  `performance-report.md`, `test-repair-report.md`, `upgrade-report.md`,
  `release-checklist.md`, `postmortem.md`, `migration.md`, `prd.md`, `adr.md`.
- **The file is the long form.** Full evidence, full command output, every
  `file:line`, every hypothesis ruled out. The chat summary is the five lines
  that make someone open it.
- **Mark gaps `[UNKNOWN]`.** Never fill a section with a plausible guess to
  make the document look finished.
- **Re-running a skill on the same subject updates the same file** rather than
  creating a near-duplicate — unless it is a genuinely new run on a new date,
  in which case the date prefix already separates them.

## In chat, after writing

```
<the 4–8 line summary the skill defines>

Written to: docs/<path>.md
```

## Never

- Never print a report in chat without having written the file first.
- Never say a document was "created" without the `Write` tool having run.
- Never skip the file because the finding seems small — a one-paragraph
  review is still a file.
- Never invent the date. Run `date +%F`.
- Never put a report anywhere but under `docs/`.

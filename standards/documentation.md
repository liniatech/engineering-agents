# Documentation Standards

Read by `li-product-manager`. Documentation exists for the person who arrives
after you leave, and for you in six months, who will remember nothing.

## The rule that matters most

**Documentation that is wrong is worse than documentation that is missing.**
Missing docs make people read the code. Wrong docs make them trust something
false. If you change behavior, update the doc in the same PR — a doc update
deferred to "later" does not happen.

Every document carries a **last-verified date**. A runbook nobody has run is
a hypothesis, not a runbook.

## Every project must contain

### README

The entry point. A new engineer should get the service running from this
alone, without asking anyone.

- What this service does — two sentences, no jargon
- Who uses it and why
- Quickstart: clone → install → configure → run → verify it worked
- How to run the tests
- Where the deeper docs live
- Who owns it

The quickstart must actually work. Run it on a clean checkout before
claiming it does — a broken quickstart is the most common and most
expensive documentation defect.

### Architecture

- What problem the design solves
- Component diagram: what talks to what, and in which direction
- Data ownership: which component owns which data
- Key decisions, and the tradeoffs accepted — link to the ADRs
- Failure modes: what happens when each dependency is unavailable

Link to `templates/adr.md` records rather than restating them. The ADRs are
the durable form; the architecture doc is the map.

### Deployment

- Environments and how they differ
- How a deploy is triggered, and by whom
- Deploy ordering when migrations are involved
- **How to roll back** — the section people read under pressure. Make it
  executable without thinking.
- What to watch after a deploy, and the abort signal

### Configuration

Every variable in a table:

| Variable | Purpose | Required | Default | Safe to change live? |

- Say which are secrets — and never show their values
- Note which require a restart
- Keep `.env.example` in sync with this table; drift here causes deploy
  failures more often than anything else

### Runbook

Use `templates/runbook.md`. Written for someone paged at 2am who has never
touched this service. Assume no context and no patience.

### Troubleshooting

Symptom → likely cause → fix. Ordered by frequency, not by severity.

The best source is your own incident history: every postmortem should
contribute an entry here. If it happened once, it will happen again, and the
next person deserves the shortcut.

## Writing rules

- Lead with the answer. Context after, if needed.
- Write for someone with no context. "Obvious" is the most expensive word in
  documentation.
- Concrete over abstract: real commands, real paths, real values.
- Short sentences. Short sections. Headings people can scan.
- State the "why" for anything non-obvious — the constraint, the rejected
  alternative, the reason the simple approach fails.
- Mark uncertainty explicitly. "I believe X, unverified" beats a confident
  wrong claim.
- Never document aspirations as if they exist. If it is planned, label it
  planned.

## In code

- Comments explain **why**, not what. A comment restating the code is noise
  that will drift out of sync.
- Docstrings on public functions: purpose, parameters, return, and what it
  raises.
- Comment the non-obvious: a workaround, a magic number, a deliberate
  deviation from the standard. Include the reason and a link if there is one.
- Delete commented-out code. Git remembers it; you will not need it.

## Keeping it true

- Doc changes ship in the same PR as the behavior change.
- Broken quickstart or runbook = a bug, filed as a bug.
- A doc nobody has verified in a year is a hypothesis. Either verify it or
  mark it stale — an honest "unverified since <date>" is more useful than
  false confidence.

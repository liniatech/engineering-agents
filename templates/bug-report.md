# Bug Report

**Task title:** One line, imperative, naming the observable failure and the
surface it happens on. Has to stand alone when pasted into a tracker — not
the symptom ("KeyError in webhooks"), not a placeholder ("fix webhook bug").

## Summary

One sentence. What is broken, for whom.

## Severity

Critical (data loss / outage / security) | High | Medium | Low

## Environment

- Environment: production / staging / local
- Version or commit:
- First observed:
- Still occurring: yes / no

## Reproduction

Deterministic steps. If it is intermittent, say how often.

1.
2.
3.

**Reproduces:** always / intermittently (N in M) / once, not since

## Expected behavior

What should happen, and the source — spec, ticket, or documented behavior.

## Actual behavior

What happens instead. Paste the real error output, not a paraphrase.

```
```

## Impact

Who is affected, how many, and whether there is a workaround.

## Evidence

Logs, trace IDs, screenshots, failing test, relevant metrics.

## Ruled out

Every hypothesis investigated and refuted, with the evidence that killed it.
Recording a dead theory is what stops it being resurrected next week.

| Hypothesis | Refuted by |
|---|---|
| | |

## Root cause

One sentence, with `file:line`. If no `file:line` can be cited, this is still a
symptom — say so rather than promoting the best guess.

## Blast radius

Other sites sharing the same cause — other call sites, the same unchecked
assumption, other consumers of the same contract. Each as `file:line` with a
verdict: **affected** / **not affected (why)** / **needs checking**.

"None found" is a valid answer, but only after looking — record what was
searched for.

## Fix

Link to PR. The test that now guards against regression. If the fix disproved
the Root cause above, correct it here rather than leaving both standing.

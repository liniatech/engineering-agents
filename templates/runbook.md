# Runbook: <service or procedure>

**Owner:**
**Last verified:** <date — a runbook nobody has run is a hypothesis>
**Escalation:**

## What this service does

Two sentences, written for someone paged at 2am who has never touched it.

## Architecture at a glance

Upstream dependencies → this service → downstream consumers.
What breaks if each dependency is down.

## Health

| Check | Where | Healthy looks like |
|---|---|---|
| Liveness | | |
| Key metric | | |
| Dashboard | | |
| Logs | | |

## Common alerts

For each alert: what it means and what to do.

### <ALERT_NAME>

- **Means:**
- **Urgency:** page / ticket
- **First check:**
- **Likely causes:** ordered by frequency
- **Mitigation:**
- **Escalate if:**

## Operations

### Deploy

### Rollback

The most important section. Should be executable without thinking.

### Restart

### Scale up / down

## Configuration

| Variable | Purpose | Default | Safe to change live? |
|---|---|---|---|

## Failure modes

| Symptom | Likely cause | Mitigation |
|---|---|---|

## Things that will bite you

The non-obvious knowledge that usually lives only in one person's head.
Write it down here.

## What NOT to do

Actions that seem reasonable and are not. Explain why.

## Escalation

| Situation | Contact |
|---|---|

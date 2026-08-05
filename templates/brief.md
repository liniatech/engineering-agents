# {Project} — Brief

The raw ask, in your own words — the seed the kickoff grows from. `li-project-kickoff`
reads this first, then runs the discovery interview. Write the idea freely; set the
parameters to steer the defaults the kickoff obeys.

## Idea

What you want to build, in plain language. A paragraph or two is enough — the
discovery interview draws out the rest. Anything left vague becomes a
`[NEEDS INPUT]` marker downstream, not an invented answer.

## Parameters

Set what you know; leave the rest blank to take the default. A value here is
treated as **decided** — it overrides the default and is not re-asked in discovery.

- **Language:** `<en | pt-BR>` — the language every kickoff document is written in.
  Default: **`en`**. (Language-neutral regardless: document IDs, file names, the
  slug, code, SQL, and mermaid keywords.)
- **Infra:** `<e.g. Terraform + AWS | GCP | Vercel | on-prem …>` — the
  infrastructure the architecture targets. Default: **Terraform + AWS**. State an
  override here (and why) if the project must run elsewhere.
- **Programming language:** `<e.g. Python/FastAPI | TypeScript/Node | Go …>` — the
  implementation language/stack. Default: **`[NEEDS INPUT]`**, settled in discovery.

<!-- Example
Language: pt-BR
Infra: Terraform + AWS
Programming language: Python (FastAPI)
-->

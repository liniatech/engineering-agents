---
description: Kick off a greenfield project — discovery interview, PRD, SDD + ADR backbone, ending at a build-ready task list
argument-hint: [--lang en|pt-BR] <what you want to build>
---

Run the `li-project-kickoff` skill for this new project:

$ARGUMENTS

Start with the discovery interview — ask me about the business and the problem
one question at a time before spawning any agent. If I passed `--lang`, produce
all documents in that language (English or Brazilian Portuguese); otherwise ask
me once which language I want. Follow the skill's gates: stop for my approval
after scoping, after the PRD, and after the architecture.

Keep the generated docs tight and diagram-first per the skill's Output-style
rules. End at the task list in `docs/plan/`. Do not start building, and do not
commit.

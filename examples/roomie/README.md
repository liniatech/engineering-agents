# Exemplo: saída do kickoff do Roomie (pt-BR)

> **Example: Roomie kickoff output.** A complete, tight, diagram-first document
> tree produced by the `project-kickoff` skill for one greenfield project — in
> **Brazilian Portuguese**, to demonstrate the `--lang pt-BR` option. Interview
> answers for this sample were simulated from the brief.

Esta pasta é um **exemplo completo** — a árvore de documentos que a skill
`project-kickoff` produz para um projeto greenfield, em português, no estilo
enxuto e diagram-first (um diagrama mermaid por ADR).

Projeto de exemplo: **Roomie** — um catálogo web onde universitários publicam e
procuram quartos e colegas de quarto (fotos, descrição, valor), com reputação
por pessoa; a plataforma medeia o primeiro contato.

## O que há aqui, na ordem em que o kickoff produziu

```
brief.md                                 o pedido original (input do usuário)
docs/scoping/roomie-discovery.md         Fase 0 — notas da entrevista de descoberta
docs/ADRs/PRD-roomie-001.md              Fase 1 — o PRD (product-manager)
docs/ADRs/ADR-roomie-001.md              Fase 2 — o SDD, C4 L1→L2→L3 (architect)
docs/ADRs/ADR-roomie-002.md                backbone: auth          (architect)
docs/ADRs/ADR-roomie-003.md                backbone: banco de dados (database-engineer)
docs/ADRs/ADR-roomie-004.md                backbone: api           (architect)
docs/ADRs/ADR-roomie-005.md                backbone: observabilidade (architect)
docs/ADRs/ADR-roomie-006.md                backbone: infraestrutura (architect)
docs/ADRs/ADR-roomie-007.md                backbone: ci/cd         (architect)
docs/ADRs/ADR-roomie-008.md                backbone: segurança     (architect)
docs/ADRs/ADR-roomie-009.md                backbone: testes        (qa)
docs/plan/roomie-build-plan.md           Fase 3 — a quebra em tarefas  ← o kickoff termina aqui
```

## Como ler

- Comece pelo **`ADR-roomie-001`** (o SDD) — tem os diagramas C4 e a tabela que
  aponta para cada decisão de backbone.
- Cada ADR tem **exatamente um diagrama** que mostra a decisão num relance.
- Repare nos marcadores `[PRECISA DE INPUT]` — o kickoff expõe o que não sabe em
  vez de inventar, e o plano de construção mapeia cada lacuna à tarefa que ela
  trava (ex.: o modelo de reputação bloqueia a tarefa T8).

## Como foi gerado

Rode `/kickoff --lang pt-BR <a ideia>` e responda à entrevista de descoberta.
Veja [`../../GUIDE.md`](../../GUIDE.md#start-a-greenfield-project--kickoff) e a
convenção de numeração em
[`../../standards/adr-methodology.md`](../../standards/adr-methodology.md).

> Gerado como uma execução da skill `project-kickoff`. As respostas da entrevista
> para este exemplo foram simuladas a partir do brief; uma execução real coleta
> as respostas de você.

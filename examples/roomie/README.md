# Exemplo: saída do kickoff do Roomie (pt-BR)

> **Example: Roomie kickoff output.** A complete, tight, diagram-first document
> tree produced by the `li-project-kickoff` skill for one greenfield project — in
> **Brazilian Portuguese**, to demonstrate the `--lang pt-BR` option. Interview
> answers for this sample were simulated from the brief.

Esta pasta é um **exemplo completo** — a árvore de documentos que a skill
`li-project-kickoff` produz para um projeto greenfield, em português, no estilo
enxuto e diagram-first (um diagrama mermaid por ADR).

Projeto de exemplo: **Roomie** — um catálogo web onde universitários publicam e
procuram quartos e colegas de quarto (fotos, descrição, valor), com reputação
por pessoa; a plataforma medeia o primeiro contato.

## O que há aqui, na ordem em que o kickoff produziu

Slug do repo: `ROM`. Nome do arquivo = identificador + sufixo de tópico; o
identificador dentro do documento permanece curto (`ADR-ROM-003`).

```
brief.md                                          o pedido original (input do usuário)
docs/scoping/ROM-discovery.md                     Fase 0 — notas da entrevista de descoberta
docs/ADRs/PRD-ROM-001-product-requirements.md     Fase 1 — o PRD (li-product-manager)
docs/ADRs/ADR-ROM-001-system-design.md            Fase 2 — o SDD, C4 L1→L2→L3 (li-architect)
docs/ADRs/ADR-ROM-002-auth.md                       backbone: auth           (li-architect)
docs/ADRs/ADR-ROM-003-database.md                   backbone: banco de dados (li-database-engineer)
docs/ADRs/ADR-ROM-004-api.md                        backbone: api            (li-architect)
docs/ADRs/ADR-ROM-005-observability.md              backbone: observabilidade (li-architect)
docs/ADRs/ADR-ROM-006-infrastructure.md             backbone: infraestrutura (li-architect)
docs/ADRs/ADR-ROM-007-ci-cd.md                      backbone: ci/cd          (li-architect)
docs/ADRs/ADR-ROM-008-security.md                   backbone: segurança      (li-architect)
docs/ADRs/ADR-ROM-009-testing.md                    backbone: testes         (li-qa)
docs/plan/ROM-build-plan.md                       Fase 3 — a quebra em tarefas  ← o kickoff termina aqui
```

## Como ler

- Comece pelo **`ADR-ROM-001`** (o SDD, arquivo `ADR-ROM-001-system-design.md`)
  — tem os diagramas C4 e a tabela que aponta para cada decisão de backbone.
- Cada ADR tem **exatamente um diagrama** que mostra a decisão num relance.
- Repare nos marcadores `[PRECISA DE INPUT]` — o kickoff expõe o que não sabe em
  vez de inventar, e o plano de construção mapeia cada lacuna à tarefa que ela
  trava (ex.: o modelo de reputação bloqueia a tarefa T13).

## Como foi gerado

Rode `/li-kickoff --lang pt-BR <a ideia>` e responda à entrevista de descoberta.
Veja [`../../GUIDE.md`](../../GUIDE.md#start-a-greenfield-project--kickoff) e a
convenção de numeração em
[`../../standards/adr-methodology.md`](../../standards/adr-methodology.md).

> Gerado como uma execução da skill `li-project-kickoff`. As respostas da entrevista
> para este exemplo foram simuladas a partir do brief; uma execução real coleta
> as respostas de você.

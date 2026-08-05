# PRD-ROM-001 — Requisitos de Produto

> **Documento:** `PRD-ROM-001` · **Slug:** `ROM` · **Idioma:** pt-BR (IDs, código, SQL e mermaid em inglês)
> **Última verificação:** 2026-07-31 · **Fonte:** `docs/scoping/ROM-discovery.md`

## Resumo

Roomie é um catálogo web onde universitários publicam um quarto/vaga ou se anunciam como candidatos, cada pessoa carrega uma reputação, e a plataforma medeia o **primeiro contato** entre as duas pontas. O produto substitui a busca espalhada por WhatsApp, Facebook e murais — que falha em padronização, histórico e permanência — por um lugar único com confiança embutida. Sucesso é o par se formar dentro do Roomie: publicação → candidato confiável → conexão mútua → fechamento.

## Problema

A busca por quarto/colega de moradia acontece hoje fragmentada e sem confiança. As três falhas da alternativa atual:

| Falha | Efeito |
|---|---|
| Sem padronização | Impossível comparar valor e condições entre anúncios |
| Sem histórico | Não há como saber se a pessoa é confiável — aposta às cegas |
| Efêmero | O post some no scroll; a busca recomeça do zero |

A dor comum às duas pontas é **confiança**. Anúncio sem reputação é risco.

## Usuários

Dois lados do mesmo mercado, ambos universitários, público mobile-first.

| Persona | Contexto | Objetivo | Dor principal |
|---|---|---|---|
| **Anunciante** (tem quarto/vaga) | Já mora no imóvel, divide aluguel | Preencher a vaga com alguém confiável | Não sabe quem está do outro lado |
| **Candidato** (procura quarto/colega) | Cidade nova, início de semestre, com pressa e orçamento apertado | Achar quarto e colega confiáveis rápido | Anúncios sem histórico, comparação difícil |

Prioridade de atendimento: o **candidato** é o lado escasso de atenção (chega com urgência); o **anunciante** é quem alimenta o catálogo. Ambos são P0 — sem os dois não há marketplace.

## Objetivos

1. Padronizar a publicação de quartos e de candidatos (dados comparáveis).
2. Dar a cada usuário uma reputação legível antes do contato.
3. Mediar o primeiro contato apenas após interesse mútuo.
4. Concentrar a jornada busca→match→fechamento dentro da plataforma.

## Métricas de sucesso

| Métrica | Baseline | Alvo |
|---|---|---|
| **Norte:** conexões mútuas concretizadas / mês | 0 (greenfield) | `[PRECISA DE INPUT]` |
| Anúncios publicados / mês | 0 | `[PRECISA DE INPUT]` |
| % de conexões mútuas que resultam em fechamento | n/a | `[PRECISA DE INPUT]` |
| % de usuários com reputação preenchida antes do 1º contato | n/a | `[PRECISA DE INPUT]` |

## Requisitos funcionais

Numerados para citação em ADRs e no build plan.

**Cadastro e identidade**
- **RF-1** — O sistema deve permitir criar conta e autenticar.
- **RF-2** — O sistema deve registrar o vínculo estudantil do usuário. *Método de verificação (e-mail institucional / documento / nenhum): `[PRECISA DE INPUT]`.*
- **RF-3** — O sistema deve permitir que um usuário atue nas duas pontas (anunciar e procurar) com o mesmo cadastro.

**Anúncio de quarto (lado oferta)**
- **RF-4** — O usuário deve poder publicar um anúncio de quarto/vaga com fotos, descrição e valor.
- **RF-5** — O sistema deve impor um conjunto padronizado de campos obrigatórios para que anúncios sejam comparáveis.
- **RF-6** — O usuário deve poder editar, pausar e encerrar seu anúncio.

**Anúncio de candidato (lado demanda)**
- **RF-7** — O usuário deve poder publicar um perfil de candidato (o que procura, faixa de valor, preferências).
- **RF-8** — O sistema deve permitir buscar e filtrar anúncios de quarto por critérios comparáveis (ex.: valor, localização, características). *Critérios exatos dependem do escopo geográfico: `[PRECISA DE INPUT]`.*

**Reputação**
- **RF-9** — O sistema deve exibir uma reputação por usuário, visível a ambas as pontas antes do contato. *O que gera reputação (avaliação pós-conexão / verificação / nota / badges / histórico): `[PRECISA DE INPUT]` — coração do produto, indefinido.*

**Conexão / match**
- **RF-10** — Um usuário deve poder manifestar interesse em um anúncio da outra ponta.
- **RF-11** — O sistema deve liberar o contato somente após interesse mútuo (match). *Hipótese de trabalho: match mútuo → libera contato. Forma da mediação (chat interno vs. troca de contato após consentimento): `[PRECISA DE INPUT]`.*
- **RF-12** — O sistema deve registrar quando uma conexão mútua se concretiza (para a métrica-norte).

**Trust & safety**
- **RF-13** — O sistema deve permitir denunciar anúncios e usuários. *Fluxo de moderação: `[PRECISA DE INPUT]`.*

## Requisitos não-funcionais

- **RNF-1 — Plataforma:** aplicação web responsiva; deve funcionar bem em mobile-web (público jovem, mobile-first).
- **RNF-2 — Idioma:** interface em pt-BR.
- **RNF-3 — Desempenho:** tempo de resposta da busca `[PRECISA DE INPUT: p95 alvo]`.
- **RNF-4 — Escala:** usuários e anúncios simultâneos esperados `[PRECISA DE INPUT]`; a demanda concentra-se em picos sazonais de matrícula (hipótese).
- **RNF-5 — Disponibilidade:** SLA alvo `[PRECISA DE INPUT]`.
- **RNF-6 — Escopo geográfico:** uma cidade / um campus / nacional `[PRECISA DE INPUT]` — dimensiona busca e infraestrutura.
- **RNF-7 — Privacidade:** dados de contato só se tornam visíveis após match (RF-11); tratamento conforme LGPD `[PRECISA DE INPUT: requisitos específicos]`.

## Critérios de aceite

**Publicar quarto (RF-4, RF-5)**
- Dado um usuário autenticado, Quando ele preenche todos os campos obrigatórios padronizados e envia, Então o anúncio fica publicado e buscável.
- Dado um anúncio sem campo obrigatório, Quando o usuário tenta publicar, Então o sistema bloqueia e indica o campo faltante.

**Buscar (RF-8)**
- Dado um catálogo com anúncios, Quando o candidato aplica filtros comparáveis, Então o sistema retorna apenas anúncios que satisfazem todos os filtros.
- Dado um filtro sem resultados, Quando a busca é executada, Então o sistema exibe estado vazio explícito (sem erro).

**Reputação (RF-9)**
- Dado um perfil de usuário, Quando a outra ponta o visualiza antes do contato, Então a reputação é exibida. *Regra de composição pendente — `[PRECISA DE INPUT]`.*

**Conexão mútua (RF-10, RF-11, RF-12)**
- Dado que A manifestou interesse em B, Quando B ainda não correspondeu, Então o contato permanece bloqueado para ambos.
- Dado que A e B manifestaram interesse mútuo, Quando o match se forma, Então o sistema libera o contato e registra a conexão para a métrica-norte.
- Dado dois cliques de interesse do mesmo usuário no mesmo alvo, Quando o segundo é enviado, Então o sistema não cria interesse/match duplicado.

**Denúncia (RF-13)**
- Dado um anúncio ou usuário, Quando alguém o denuncia, Então o sistema registra a denúncia para triagem. *Ação subsequente pendente — `[PRECISA DE INPUT]`.*

## Riscos

| Risco | Impacto | Observação |
|---|---|---|
| Modelo de reputação indefinido | Alto | É o diferencial central (RF-9); sem ele o produto vira mais um mural |
| Cold start do marketplace | Alto | Sem oferta inicial, candidatos não voltam (e vice-versa) |
| Verificação de vínculo frágil | Médio | Afeta confiança e cadastro (RF-2) |
| Trust & safety subespecificado | Médio | Denúncia sem moderação real não protege usuários |
| Sazonalidade de matrícula | Médio | Picos previsíveis pressionam escala (RNF-4) |

## Fora de escopo

- Processamento de pagamento de aluguel e retenção de caução (a plataforma medeia contato, não transação). *Reabrir se a monetização exigir — `[PRECISA DE INPUT]`.*
- Funções de imobiliária: contrato de locação, vistoria, intermediação jurídica.
- Rede social genérica / feed — o objetivo é o par morador↔quarto.

## Perguntas em aberto

1. **Modelo de reputação** — o que gera reputação (avaliação pós-conexão, verificação de matrícula, nota 1–5, badges, histórico)? `[PRECISA DE INPUT]` (RF-9).
2. **Verificação de identidade / vínculo estudantil** — exigir e-mail institucional, documento, ou nada? `[PRECISA DE INPUT]` (RF-2).
3. **Monetização** e existência de fluxo de pagamento. `[PRECISA DE INPUT]` (Fora de escopo, RNF).
4. **Escopo geográfico** (cidade / campus / nacional) e **escala** esperada. `[PRECISA DE INPUT]` (RNF-4, RNF-6).
5. **Forma da mediação do contato** — chat interno ou troca de contato após consentimento mútuo? `[PRECISA DE INPUT]` (RF-11).
6. **Moderação/denúncia** — fluxo e ações após uma denúncia. `[PRECISA DE INPUT]` (RF-13).
7. **Números-alvo das métricas de sucesso** (norte e secundárias). `[PRECISA DE INPUT]` (Métricas).
8. **Gatilho de tempo / "por que agora"** — sazonalidade de matrícula é hipótese, não confirmada. `[PRECISA DE INPUT]`.

## Last verified
2026-07-31

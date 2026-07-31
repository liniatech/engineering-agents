---
**ID:** PRD-roomie-001
**Slug:** roomie
**Status:** Rascunho
**Autor:** product-manager
**Última verificação:** 2026-07-31
---

### Problema

Estudantes universitários enfrentam duas dores complementares:

- **Quem tem um quarto** para dividir não tem onde anunciá-lo com foto, descrição e valor de forma padronizada e visível para o público certo.
- **Quem procura** um quarto/colega precisa vasculhar canais dispersos (grupos de mensagem, murais físicos, indicação informal), sem filtro, sem preço comparável e sem qualquer sinal de confiabilidade da outra pessoa.

Nenhum canal atual oferece, ao mesmo tempo: catálogo com fotos e preço, reputação da pessoa e mediação da conexão. A consequência é busca lenta, risco de golpe/má experiência e conexões que não acontecem.

### Usuários

Dois lados, ambos universitários. Ordenados por prioridade no v1:

| # | Persona | Quem é | Problema específico |
|---|---------|--------|---------------------|
| 1 | **Anunciante** | Estudante com um quarto disponível para dividir | Precisa divulgar o quarto (fotos, descrição, valor) e receber contatos de pessoas confiáveis |
| 2 | **Buscador** | Estudante procurando quarto/colega | Precisa encontrar e comparar quartos, avaliar a confiabilidade do anunciante e iniciar contato |

- Uma mesma pessoa pode ser anunciante e buscadora em momentos diferentes.
- `[PRECISA DE INPUT: escopo geográfico — uma universidade/cidade específica ou aberto no v1?]`
- `[PRECISA DE INPUT: cadastro restrito a e-mail acadêmico (ex.: domínio .edu / institucional) ou cadastro aberto?]`

### Objetivos

1. Oferecer um catálogo único onde universitários publicam e encontram quartos/colegas.
2. Dar sinal de confiança por meio de reputação atrelada a cada pessoa.
3. Mediar o primeiro contato entre anunciante e buscador dentro da plataforma.

Não-objetivos (v1): intermediar pagamento, contrato de locação ou substituir a negociação entre as partes.

### Métricas de sucesso

| Métrica | Baseline | Meta | Como medir |
|---------|----------|------|------------|
| Conexões efetivadas / mês | 0 (produto novo) | `[PRECISA DE INPUT]` | Contagem de primeiros contatos iniciados via plataforma |
| % de anúncios que recebem ≥1 contato | 0 | `[PRECISA DE INPUT]` | Anúncios com contato / total publicados |
| Anúncios publicados / mês | 0 | `[PRECISA DE INPUT]` | Contagem de anúncios "publicado" |
| Tempo mediano até o 1º contato | n/a | `[PRECISA DE INPUT]` | Mediana entre publicação e primeiro contato |

Baseline 0 é confirmado por ser produto greenfield; metas dependem de input do negócio.

### Requisitos funcionais

Prioridade: **M** (must, v1) / **P** (poderia).

**Conta e identidade**
- RF-01 (M) Criar conta e fazer login.
- RF-02 (M) Manter perfil com, no mínimo, nome e foto.
- RF-03 (M) Validar cadastro segundo a regra de elegibilidade. `[PRECISA DE INPUT: e-mail acadêmico ou aberto?]`
- RF-04 (P) Editar e excluir o próprio perfil.

**Anúncios (publicar)**
- RF-05 (M) Criar anúncio com: uma ou mais fotos, descrição e valor.
- RF-06 (M) Exigir campos obrigatórios antes de publicar. `[PRECISA DE INPUT: lista definitiva de campos]`
- RF-07 (M) Editar, despublicar e excluir os próprios anúncios.
- RF-08 (M) Cada anúncio associado à pessoa que o publicou e exibindo sua reputação.
- RF-09 (P) Marcar anúncio como "indisponível/preenchido" sem excluir.

**Catálogo (procurar)**
- RF-10 (M) Listar anúncios publicados em um catálogo.
- RF-11 (M) Ver detalhe do anúncio (fotos, descrição, valor, reputação do anunciante).
- RF-12 (M) Filtrar por, no mínimo, faixa de valor. `[PRECISA DE INPUT: outros filtros — localização, gênero, disponibilidade?]`
- RF-13 (P) Ordenar (valor, reputação, mais recente).

**Reputação**
- RF-14 (M) Reputação visível no perfil e nos anúncios de cada pessoa.
- RF-15 (M) Gerar reputação a partir de interações. `[PRECISA DE INPUT: modelo — avaliação mútua? nota 1–5? quem avalia quem e quando?]`
- RF-16 (M) Impedir autoavaliação.
- RF-17 (P) Mitigar avaliações repetidas para inflar/rebaixar. `[PRECISA DE INPUT: regra antiabuso]`

**Conexão**
- RF-18 (M) Iniciar contato com o anunciante a partir de um anúncio.
- RF-19 (M) Notificar o anunciante quando alguém inicia contato. `[PRECISA DE INPUT: canal — e-mail, in-app?]`
- RF-20 (M) Registrar cada conexão iniciada (métrica). `[PRECISA DE INPUT: mecanismo do primeiro contato]`

**Moderação e segurança de conteúdo**
- RF-21 (M) Denunciar anúncio ou pessoa.
- RF-22 (M) Moderador remove anúncio/perfil que viole as regras. `[PRECISA DE INPUT: quem modera e quais regras?]`

### Requisitos não funcionais

| # | Categoria | Requisito |
|---|-----------|-----------|
| RNF-01 | Latência | Catálogo e detalhe carregam em ≤ 2 s no p95, em 4G. `[PRECISA DE INPUT: confirmar]` |
| RNF-02 | Escala | Suportar `[PRECISA DE INPUT: nº de usuários e anúncios no v1]` |
| RNF-03 | Disponibilidade | ≥ 99,5% mensal. `[PRECISA DE INPUT: confirmar]` |
| RNF-04 | Plataforma | Web responsiva; navegadores modernos; tela ≥ 360 px |
| RNF-05 | Conformidade | LGPD: base legal, consentimento, direito de exclusão e portabilidade |
| RNF-06 | Segurança | Autenticação para publicar, contatar e avaliar; credenciais nunca em texto claro |
| RNF-07 | Upload de mídia | Limitar tamanho/formato. `[PRECISA DE INPUT: nº máx. de fotos, tamanho, formatos]` |
| RNF-08 | Idioma | Interface e conteúdo em pt-BR |
| RNF-09 | Acessibilidade | `[PRECISA DE INPUT: nível-alvo, ex.: WCAG 2.1 AA?]` |

### Critérios de aceitação

**CA-01 — Publicar anúncio.** Dado anunciante autenticado e elegível, Quando preenche os campos obrigatórios (fotos, descrição, valor) e confirma, Então o anúncio aparece no catálogo associado ao seu perfil com sua reputação.

**CA-02 — Publicação bloqueada por campo faltante.** Dado que crio um anúncio, Quando tento publicar sem um campo obrigatório, Então é recusado indicando o campo faltante, sem descartar o preenchido.

**CA-03 — Encontrar e filtrar.** Dado buscador no catálogo, Quando filtro por faixa de valor, Então vejo apenas anúncios publicados nessa faixa.

**CA-04 — Ver reputação no anúncio.** Dado anúncio publicado, Quando abro o detalhe, Então vejo fotos, descrição, valor e a reputação atual do anunciante.

**CA-05 — Iniciar conexão.** Dado buscador autenticado vendo anúncio de outra pessoa, Quando inicio contato, Então o anunciante é notificado e a conexão é registrada.

**CA-06 — Reputação após interação.** Dado que participei de conexão elegível, Quando avalio a outra pessoa segundo as regras, Então a reputação dela é atualizada e não posso avaliar a mim mesmo. `[PRECISA DE INPUT: "conexão elegível" e regra de atualização]`

**CA-07 — Denúncia e remoção.** Dado anúncio/perfil que viola as regras, Quando alguém denuncia, Então a denúncia é registrada e um moderador pode removê-lo.

**CA-08 — Editar/excluir o próprio conteúdo.** Dado que sou dono, Quando edito ou excluo, Então reflete no catálogo e só eu (ou um moderador) posso alterá-lo.

### Casos de borda

| Cenário | Comportamento esperado |
|---------|------------------------|
| Estado vazio (catálogo/busca sem resultado) | Mensagem de estado vazio orientando o próximo passo |
| Escala máxima (muitos anúncios) | Resultados paginados; latência no RNF-01. `[PRECISA DE INPUT: tamanho de página]` |
| Acesso concorrente (2 contatos no mesmo anúncio) | Ambas as conexões registradas; nenhuma sobrescreve a outra |
| Falha parcial (upload de foto falha) | Anúncio não publicado pela metade; erro exibido; texto preservado |
| Permissão negada (editar anúncio alheio) | Ação bloqueada; nenhum dado alterado |
| Submissão duplicada (clique duplo) | Sem anúncio/conexão duplicados |
| Anúncio já preenchido | `[PRECISA DE INPUT: bloquear, avisar ou permitir?]` |
| Conta excluída | `[PRECISA DE INPUT: o que ocorre com anúncios ativos e reputação recebida?]` |

### Riscos

| Risco | Impacto | Observação |
|-------|---------|------------|
| Modelo de reputação mal definido permite abuso | Perda de confiança | Depende de RF-15/RF-17, hoje em aberto |
| Conteúdo impróprio ou golpes | Segurança do usuário, marca | Mitigar com moderação e denúncia |
| Massa crítica insuficiente no lançamento | Produto morre | Estratégia depende do escopo geográfico |
| Não conformidade LGPD (fotos, contato) | Legal e reputacional | Base legal e fluxo de exclusão desde o v1 |
| Cadastro aberto vs. restrito indefinido | Retrabalho | Bloqueia RF-03 |

### Fora de escopo (v1)

- Pagamento/checkout do aluguel na plataforma.
- Contrato/assinatura digital de locação.
- App mobile nativo (v1 é web responsiva).
- Chat em tempo real (v1 medeia o primeiro contato; conversa contínua fica fora).
- Verificação/scoring externo de crédito ou antecedentes.
- Matching algorítmico automático de colegas.

### Perguntas em aberto

1. Escopo geográfico: uma universidade/cidade ou aberto no v1?
2. Cadastro restrito a e-mail acadêmico ou aberto? (RF-03, RNF-06)
3. Modelo de reputação: quem avalia quem, quando, escala, verificação, antiabuso? (RF-15/16/17, CA-06)
4. Mecanismo do primeiro contato e canal de notificação? (RF-18/19/20)
5. Metas numéricas das métricas de sucesso.
6. Moderação: quem, quais regras, SLA de denúncia? (RF-22)
7. Lista definitiva de campos obrigatórios do anúncio, incluindo localização. (RF-06)
8. Filtros/ordenações mínimos além de valor. (RF-12/13)
9. Limites de upload: nº de fotos, tamanho, formatos. (RNF-07)
10. Alvos de latência, escala, disponibilidade, acessibilidade. (RNF-01/02/03/09)
11. Contato a anúncio já preenchido; exclusão de conta com anúncios/reputação.
12. Gatilho de negócio/prazo ("por que agora").

## Última verificação
2026-07-31

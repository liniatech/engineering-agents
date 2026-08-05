# ROM — Discovery

Matéria-prima da entrevista de kickoff. As palavras do usuário, ainda não
formalizadas. `li-product-manager` lê isto para escrever `PRD-ROM-001`.

> **Idioma do projeto:** pt-BR. Neutro por convenção: IDs (`ADR-ROM-NNN`), nomes
> de arquivo, slug, código, SQL e palavras-chave mermaid ficam em inglês.
> **Slug do repo:** `ROM` (fixo para a vida do repositório).

> **Nota do demo:** esta descoberta foi **derivada do brief**, não de uma
> entrevista real. Toda incógnita genuína está marcada como `[PRECISA DE INPUT]`
> em vez de inventada.

## O problema

Universitários que precisam dividir moradia não têm um lugar bom para se achar.
Hoje a busca por um quarto ou por um colega de quarto acontece espalhada por
grupos de WhatsApp, Facebook e murais de faculdade — sem fotos padronizadas, sem
valor claro e, sobretudo, sem nenhuma noção de com quem se está falando. Roomie é
um catálogo web onde o estudante publica um quarto (fotos, descrição, valor) ou
se anuncia como candidato, cada pessoa carrega uma reputação, e a plataforma
medeia o primeiro contato entre as duas pontas.

## Quem tem

O universitário, nas duas pontas do mesmo mercado:

1. **Quem tem um quarto/vaga** e quer preencher com alguém confiável — muitas
   vezes já morando no imóvel e dividindo o aluguel.
2. **Quem procura um quarto/colega** numa cidade nova, geralmente com pressa
   (início de semestre) e orçamento apertado.

A dor comum às duas: **confiança**. Anúncio sem reputação é aposta às cegas.

## Por que agora

`[PRECISA DE INPUT]` — o brief não diz o gatilho de tempo (início de ano letivo?
alta de aluguel? uma migração específica de campus?). Tratar como hipótese: a
sazonalidade de matrícula concentra demanda em picos previsíveis.

## Alternativa de hoje

Grupos de WhatsApp/Facebook e murais físicos ou digitais da faculdade. Falham em
três pontos: (a) sem padronização — cada post é diferente, difícil comparar
valor e condições; (b) sem histórico — não há como saber se a pessoa é confiável;
(c) efêmero — o post some no scroll e a busca recomeça do zero.

## Sucesso é

O par se forma **dentro do Roomie**: alguém publica um quarto, um candidato
confiável aparece, os dois se conectam pela plataforma e fecham. Métrica-norte
proposta: **nº de conexões mútuas concretizadas / mês**. Números-alvo:
`[PRECISA DE INPUT]`.

## Restrições

- **Plataforma:** aplicação web (o brief diz "página web"). Mobile-web deve
  funcionar bem — o público é jovem e mobile-first.
- **Idioma / mercado:** pt-BR; presumivelmente Brasil. Escopo geográfico
  (uma cidade? um campus? nacional?): `[PRECISA DE INPUT]`.
- **Orçamento, prazo, tamanho do time:** `[PRECISA DE INPUT]`.
- **Escala esperada (usuários, anúncios):** `[PRECISA DE INPUT]`.
- **Monetização** (grátis, comissão, assinatura, anúncios): `[PRECISA DE INPUT]`
  — afeta se a plataforma processa pagamento (ver não-objetivos).

## Não-objetivos

- **Não** processa o pagamento do aluguel nem retém caução no MVP — a plataforma
  medeia o **contato**, não a transação financeira. (A confirmar se a
  monetização exigir o contrário — `[PRECISA DE INPUT]`.)
- **Não** é imobiliária: sem contrato de locação, vistoria ou intermediação
  jurídica.
- **Não** é rede social genérica — o objetivo é o par morador↔quarto, não feed.

## Perguntas em aberto

Cada lacuna abaixo é rastreada até a tarefa que ela trava no plano de build.

- **Modelo de reputação.** O que gera reputação? Avaliação pós-conexão? Verificação
  de matrícula? Nota de 1–5, badges, ou histórico? É o coração do produto e está
  **indefinido**. `[PRECISA DE INPUT]` → trava a tarefa de reputação (T8).
- **Verificação de identidade / vínculo estudantil.** Exigir e-mail `.edu`/
  institucional? Documento? Afeta cadastro (T2/auth) e confiança. `[PRECISA DE INPUT]`
- **Monetização** e, por consequência, se há fluxo de pagamento. `[PRECISA DE INPUT]`
- **Escopo geográfico** e **escala** — dimensionam infraestrutura e busca. `[PRECISA DE INPUT]`
- **Mediação do contato** — chat interno, ou troca de contato após consentimento
  mútuo? Afeta a API de conexão (T7). Hipótese de trabalho: **match mútuo →
  libera contato**. `[PRECISA DE INPUT]` para confirmar.
- **Moderação/denúncia** de anúncios e usuários — necessária para trust & safety,
  não especificada. `[PRECISA DE INPUT]`

## Last verified
2026-07-31

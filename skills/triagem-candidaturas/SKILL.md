---
name: triagem-candidaturas
description: Faz a triagem de candidaturas e os convites de uma campanha na Conty via MCP. Lista interessados e pré-aprovados, aprova ou rejeita em lote (por lista ou por critério, ex. match abaixo de 50), convida creators para a campanha e cancela convites de prospect. Use quando o usuário pedir para ver, aprovar, rejeitar ou gerenciar candidaturas e interessados, ou para convidar creators para uma campanha.
---

# Triagem de candidaturas e convites

**Escopo necessário: escrita** para decidir e convidar. As listagens são
leitura.

## Fluxo: candidaturas

1. **Localizar a campanha.** `buscar_campanhas` com `q` e confirme qual.

2. **Tamanho da fila.** `contagens_campanha` com o `campaign_id`: candidaturas
   todas, pendentes, aprovadas, rejeitadas, expiradas e pré-aprovadas.

3. **Listar.** `listar_itens_campanha` com:
   - `view: "applications"`, `status: "pending"` (ou CSV, `"pending,approved"`)
   - `sortBy`: `match` (aderência), `rating`, `followers`, `recent`, `oldest`
   - `only_compatible: true` para só os compatíveis
   - com agência na cadeia de revisão, `application_stage`: `pre_approved`
     (aprovadas pelos lados anteriores, esperando a decisão final),
     `interested`, `declined_by_brand` ou `curation_rejected`
   - `limit` até 100; confira `total` para saber se a página cobre tudo

   Para olhar um candidato a fundo: `ver_perfil_creator`,
   `ver_social_creator` e `ver_analytics_creator` (skill `buscar-creators`).

4. **Decidir em LOTE, uma chamada só.** `decidir_candidatura` com
   `campaign_id`, `status` (`approved` ou `rejected`) e o alvo por UM destes:
   - `application_ids`: a lista que o usuário apontou (1 a 500)
   - `selecao`: critério apurado no servidor, ex.:
     `{ "status": "pending", "match_max": 50 }` para rejeitar quem tem match
     abaixo de 50

   Antes de chamar, pergunte o que a campanha exige:
   - **`delivery_date`** quando a campanha deixa o prazo com a marca
     (`delivery_deadline_by_brand` no `detalhar_campanha`);
   - **`motivo`** (mín. 20 caracteres, lido só pela agência) ao rejeitar com
     agência na cadeia de revisão;
   - `payment_amount` opcional para fixar o cachê.

   Aprovar **cria o convite**: o creator entra quando aceita. Rejeitar encerra.
   Os dois notificam o creator.

## Fluxo: convidar creators

1. Achar a campanha (`buscar_campanhas`) e os creators (skill
   `buscar-creators`; @handles em lista vão por `resolver_handles_creators`).
   Nunca invente `user_id`.
2. A campanha precisa estar **no ar** (`approved`).
3. **Acordo:** leia `detalhar_campanha`. Se `acordo.scope_options` tem
   escopos, pergunte qual o creator entrega (`scope_option_id`); se há mais de
   um em `acordo.briefings`, pergunte qual briefing (`briefing_id`). As duas
   perguntas numa mensagem só, e nunca escolha pela marca.
4. `convidar_creators_campanha` com `campaign_id` e **todos** os `user_ids`
   numa chamada. Opcionais: `message` (até 100 caracteres, texto da marca),
   `payment_amount`, `delivery_date`, `script_deadline`.
5. Leia a resposta:
   - `vacancies`: se `open` < `waiting`, diga quantas vagas há e que quem
     aceitar primeiro entra; com `open` 0, os convites ficam em espera;
   - `pre_approved` / `pre_approved_count`: o convite foi feito pela agência e
     virou **pré-aprovação**. Ninguém foi convidado ainda; a marca decide.
     Diga isso, sem falar em "convidados".

**Convite de prospect** (creator de fora da plataforma) ainda pendente:
`cancelar_convite` com o `invite_id`.

## Erros e conflitos

- **`delivery_date_required`** / **`rejection_reason_required`**: faltou data
  ou motivo; pergunte e repita.
- **Falha parcial** (orçamento, status, `no_items_left` em campanha com produto
  sem item sobrando): a resposta separa quem passou e quem não; reporte os dois.
- **Convite**: falha só por convite ou entrega já em andamento,
  `already_pre_approved`, `already_applied_to_group` (decida pela candidatura),
  `plan_limit_reached` ou `no_items_left`. Vaga e verba não recusam convite.
- **422 pedindo escopo ou briefing**: o acordo exige a escolha; pergunte.
- **Candidatura já decidida**: recarregue a lista antes de seguir.

## Limites

- `decidir_candidatura` decide candidatura de **campanha avulsa**. Candidatura
  de acordo se decide pelo painel.
- Decisões e convites notificam o creator na hora: nada de "pra testar".
- Esta skill não aprova vídeo nem paga ninguém.

---
name: acompanhar-missoes
description: Acompanha e revisa as entregas (missões) de uma campanha na Conty via MCP. Mostra os vídeos esperando revisão, aprova com nota e data de postagem, pede alteração ou recusa em lote, recupera vídeo recusado, revisa capa/legenda/roteiro e remove ou restaura creator. Use quando o usuário pedir para ver entregas, revisar, aprovar ou recusar vídeos, pedir ajustes, ou gerenciar os creators de uma campanha.
---

# Acompanhar missões

**Escopo necessário: escrita** para agir. As listagens são leitura.

## Fluxo

1. **Ver o que espera revisão.**
   - "O que tenho para revisar?", em uma ou em todas as campanhas:
     `listar_entregas` com `etapa: "revisao"` (e `campaign_id` para uma só).
     `etapa: "aprovadas"` traz os aprovados com métricas da publicação.
   - Recorte da operação de uma campanha: `listar_missoes` com `campaign_id`.
     Vem por aba (convidados, fazendo, enviados, em alteração, aprovados,
     recusados, excluídos). Filtre com `statuses` (CSV, ex.:
     `"em_analise,editado"`), `q` (creator) e ordene com `sortBy` (`value`,
     `predicted_send_at`, `created_at`, `rating`, `views`, `engagement`) +
     `order`. Ex.: aprovados com mais views é `statuses: "aprovado"`,
     `sortBy: "views"`, `order: "desc"`.

   Status de missão: `convidado`, `fazendo`, `em_analise` e `editado` (vídeo
   enviado esperando revisão), `recusado` (alteração pedida),
   `recusado_permanentemente`, `aprovado`, `expirado`.

2. **Aprovar** (uma missão por chamada): `aprovar_video` com `mission_id` e:
   - `rating` de 1 a 5, **obrigatório**: pergunte a nota se o usuário não deu;
   - `posted_at` quando a campanha deixa a data de postagem com a marca
     (`posting_schedule_type: "brand_decides"` no `detalhar_campanha`) ou paga
     na aprovação: `2026-10-22T15:00:00-03:00` ou só `2026-10-22`, nunca antes
     de hoje;
   - `approval_message` opcional, texto da marca.

   **APROVAR NÃO SE DESFAZ** e, quando a marca paga pela carteira, já reserva o
   repasse do creator. Confirme antes. Com capa, legenda ou roteiro obrigatório
   pendente, a nota fica guardada e a entrega só fecha quando o último item for
   aprovado no passo 4.

3. **Pedir alteração ou recusar, em LOTE.** Uma chamada para todos os vídeos
   que recebem o MESMO feedback, com `campaign_id` e o alvo por `mission_ids`
   (1 a 500) ou `selecao`, nunca os dois:
   - `pedir_alteracao_video`: o creator vê o `feedback` e reenvia.
     `change_deadline_at` (ISO com fuso) é o prazo da alteração: mínimo 24 h,
     até o fim da campanha; omitido vale 48 h, então avise se a campanha acaba
     antes.
   - `recusar_video`: **RECUSA DEFINITIVA**, sai da fila. Só volta por
     `recuperar_video` (uma missão, volta para `editado`, sem pagar).

   O `feedback` é o texto que o creator lê: peça ao usuário, nunca invente.
   Feedbacks diferentes por creator são chamadas separadas, uma por texto.

4. **Capa, legenda e roteiro** (campanhas com pré-aprovação):
   `revisar_asset_missao` com `mission_id`, `asset` (`cover`, `caption` ou
   `script`) e `decision` (`approved`/`rejected`). Recusa exige `reason`, que o
   creator vê. Em missão `fazendo` só o roteiro aceita decisão; capa e legenda
   depois da entrega. Se aprovar o último item fecha a entrega e a API pedir,
   mande `rating`/`posted_at`.

5. **Tirar ou devolver creator:**
   - `remover_creator_campanha`: missão `convidado`/`fazendo` vai para
     Excluídos.
   - `restaurar_missao`: devolve ao status anterior. Se o prazo do convite já
     venceu, exige `delivery_date` nova (e `script_deadline` onde há roteiro).

6. **Push manual**, só se o usuário pedir: `notificar_creator` com
   `mission_id` e `type` (`mission_approved`, `mission_rejected`,
   `mission_rejected_permanent`, `mission_paid`). Aprovar, recusar e pedir
   alteração **já notificam**: não duplique.

## Erros e conflitos

- **`approval_rating_required`** / **`approval_posted_at_required`**: falta nota
  ou data de postagem; pergunte e repita.
- **Falha parcial** no lote: reporte quem passou e quem não, e por quê.
- **`recuperar_video`/`restaurar_missao`**: 409 `limit_reached` (sem vaga),
  `budget_exceeded` (sem verba), `no_items_left` (produto esgotado), 422
  `plan_limit_reached`, `deadline_required`, `script_deadline_required`.
  A missão não muda quando a API recusa.
- **Transição inválida**: recarregue `listar_missoes`; o status pode ter mudado.

## Limites

- Pedir alteração, recusar e decidir são ações de revisão: resolvem-se com a
  tool da ação. Não abra conversa para isso.
- Prazo e creator parado: skill `reengajar-creators`.

---
name: acompanhar-missoes
description: Acompanha e revisa as entregas (missões) de uma campanha na Conty via MCP — lista vídeos em análise, pede alteração, recusa definitivamente, recupera, reconvida creator expirado e revisa capa/legenda/roteiro. Use quando o usuário pedir para ver entregas, revisar/aprovar/recusar vídeos, pedir ajustes, ou gerenciar creators em uma campanha.
---

# Acompanhar missões

**Escopo necessário: escrita** para agir sobre as missões. A listagem
(`listar_missoes`) é leitura.

Importante: **não existe tool de aprovação de vídeo no MCP.** Aprovar uma
entrega dispara o pagamento do creator e corre pelo fluxo financeiro, fora
deste MCP. O que esta skill faz é tudo em volta: triagem, recusa, alteração,
recuperação, reconvite e revisão de assets.

## Fluxo

1. **Localizar a campanha** — se preciso, `listar_campanhas` e confirme o id.

2. **Mapear as abas.** Chame `listar_missoes` com `campaign_id`. As missões
   vêm agrupadas por status: convidados, fazendo, enviados (em análise), em
   alteração, aprovados, recusados, excluídos. Para filtrar:
   - `statuses` — CSV com os status de missão para recortar (os status vêm
     na própria listagem; use os valores que ela retorna, não traduza)
   - `prospects`, `rating`, `delivery` — flags de recorte
   - `sortBy`: `value`, `rejected_at`, `predicted_send_at`, `created_at`
   - paginação `limit`/`offset`, busca por `q`

   Cada missão traz creator, status, link do vídeo, status de capa/legenda/
   roteiro, pagamento e métricas. Apresente a fila e pergunte o que revisar.

3. **Agir sobre uma entrega em análise** — sempre mostrando o feedback que o
   creator vai receber antes de confirmar:
   - **Pedir alteração**: `pedir_alteracao_video` com `mission_id` e `feedback`
     (1–1000 caracteres, obrigatório — é o texto que o creator vê). A missão
     vai para `recusado` e o creator pode reenviar.
   - **RECUSA DEFINITIVA**: `recusar_video` com `mission_id` e `feedback`.
     A missão vai para `recusado_permanentemente` e sai da fila. Confirme com
     o usuário antes — o nome da tool já anuncia a gravidade. Se um dia for
     preciso desfazer, `recuperar_video` devolve a missão à revisão (status
     `editado`) — **sem pagar o creator**, só recoloca na fila.

4. **Revisar capa, legenda e roteiro** (campanhas com pré-aprovação): chame
   `revisar_asset_missao` com `mission_id`, `asset` (`cover`, `caption` ou
   `script`) e `decision` (`approved`/`rejected`). Em rejeição, passe `reason`
   — o creator vê o motivo, é notificado e pode reenviar.

5. **Prazos e desistências:**
   - Missão **expirada**: `reconvidar_creator` com `mission_id` recoloca o
     creator como `convidado` com prazo limpo e o notifica. Só funciona em
     missão `expirado`.
   - **Remover creator da campanha**: `remover_creator_campanha` com
     `mission_id` — a missão sai da operação (aba Excluídos). Reversível com
     `restaurar_missao`, que devolve a missão ao status anterior.

6. **Notificar manualmente**, se o usuário pedir: `notificar_creator` com
   `mission_id` e `type` (`mission_approved`, `mission_rejected`,
   `mission_rejected_permanent`, `mission_paid`) e `message` opcional. Push
   direto ao creator — use com parcimônia.

## Erros e conflitos

- **Transição inválida (ex.: reconvidar missão não expirada)**: a API recusa —
  recarregue `listar_missoes` para ver o status atual antes de repetir.
- **Feedback ausente**: `recusar_video` e `pedir_alteracao_video` exigem
  `feedback`; peça ao usuário o motivo em vez de inventar um genérico — o
  creator lê esse texto.

## Limites

- Aprovação de vídeo e pagamento ficam fora do MCP (fluxo financeiro).
- Nada aqui é anônimo: recusas, alterações e rejeições de asset notificam o
  creator na hora.

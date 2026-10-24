---
name: reengajar-creators
description: Reengaja creators parados em uma campanha da Conty via MCP. Acha missões expiradas e creators travados no meio da produção, dá mais prazo a um creator ou a todos, reconvida quem expirou antes de aceitar e manda mensagem para quem sumiu. Use quando o usuário pedir para reengajar, cobrar, dar mais prazo, estender prazo, reativar ou dar um toque em creators que pararam ou deixaram o prazo expirar.
---

# Reengajar creators

**Escopo necessário: escrita** para agir. O mapeamento é leitura.

## Fluxo

1. **Localizar a campanha** com `buscar_campanhas` e confirme qual. Leia o fim
   da campanha no `detalhar_campanha`: nenhum prazo novo passa dele.

2. **Mapear quem está parado.** `listar_missoes` com o `campaign_id`:
   - **Expirados**: `statuses: "expirado"`.
   - **Travados**: `statuses: "fazendo"`, `sortBy: "predicted_send_at"`,
     `order: "asc"` (prazo mais antigo primeiro). Quanto tempo conta como
     travado quem define é o usuário; pergunte se ele não disser.

   Apresente creator, status e prazo, e confirme quem entra antes de agir.
   Sempre reliste antes de agir em lote: o creator pode ter entregue enquanto
   vocês decidiam.

3. **Escolher a ação pelo caso:**

   | Caso | Tool |
   |---|---|
   | Um creator em `fazendo` precisa de mais prazo | `estender_prazo_missao` |
   | Expirou por prazo **depois** de aceitar | `estender_prazo_missao` (volta para Fazendo, sem aceitar de novo) |
   | Expirou **antes** de aceitar o convite | `reconvidar_creator` |
   | Mais prazo para **todos** de uma vez | `renovar_prazos` (skill `editar-campanha`) |
   | Creator sumido, sem caso de prazo | conversa (passo 5) |

4. **Dar prazo ou reconvidar.**
   - `estender_prazo_missao` com `mission_id` e `delivery_date` (ISO com fuso,
     ex.: `2026-10-26T23:59:59-03:00`, vale o dia inteiro); `script_deadline`
     só onde há roteiro. "Mais 3 dias" é o `predicted_send_at` da missão mais
     3 dias. O creator é avisado por push e e-mail.
   - `reconvidar_creator` com `mission_id`: volta a `convidado` com prazo limpo
     e notifica. Não ocupa vaga nem verba; o teto é conferido no aceite.

   Essas ações já notificam: não precisa de mensagem manual.

5. **Cobrar quem sumiu.** Abra a conversa e envie o texto **da marca**:
   - `abrir_conversa` com `creator_id` + `enterprise_id` (DM com o creator) ou
     `campaign_id` (sala da campanha), exatamente um escopo;
   - `enviar_mensagem` com o `room_id` devolvido e o `content`.

   **Nunca improvise a cobrança.** Peça a mensagem ou mostre um rascunho para o
   usuário aprovar: tom de cobrança é decisão da marca.

6. **Resumir**: quantos ganharam prazo, quantos foram reconvidados, quantos
   contatados, e quem ficou de fora e por quê.

## Erros e conflitos

- **`reconvidar_creator` em missão não expirada**: a API recusa. Reliste.
- **409 `no_items_left`**: campanha com produto sem item sobrando.
- **Data depois do fim da campanha**: recusada. Estenda o fim antes com
  `renovar_prazos` (`end_date`) ou `editar_acordo`, com confirmação.
- **Tool de escrita ausente**: login sem opt-in de escrita; reconecte.

## Limites

- Desistência definitiva de quem não responde é `remover_creator_campanha`,
  na skill `acompanhar-missoes`.
- Pedir alteração ou recusar entrega não é cobrança: use a skill
  `acompanhar-missoes`, não o chat.

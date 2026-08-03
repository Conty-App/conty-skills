---
name: reengajar-creators
description: Reengaja creators parados em uma campanha da Conty via MCP — encontra missões expiradas e creators travados no meio da produção, reconvida quem expirou e manda mensagem pra quem sumiu. Use quando o usuário pedir para reengajar, cobrar, reativar ou dar um toque em creators que pararam ou deixaram o prazo expirar.
---

# Reengajar creators

**Escopo necessário: escrita** para agir (`reconvidar_creator`,
`abrir_conversa`, `enviar_mensagem`). O mapeamento inicial (`listar_missoes`)
é leitura.

## Fluxo

1. **Localizar a campanha** — se preciso, `listar_campanhas` (normalmente
   `status: "active"`) e confirme o id.

2. **Mapear quem está parado.** Chame `listar_missoes` com o `campaign_id` e
   separe dois grupos:
   - **Expirados**: missões com status `expirado` — creator perdeu o prazo.
   - **Travados**: missões em `fazendo` há muito tempo — ordene com
     `sortBy: "created_at"`, `order: "asc"` para ver as mais antigas primeiro.
     O que conta como "travado" (7 dias? 15?) quem define é o usuário —
     pergunte se ele não disser.

   Apresente a lista: creator, status, há quanto tempo está parada. Confirme
   quem entra no reengajamento antes de agir.

3. **Reconvidar os expirados.** Para cada missão expirada confirmada, chame
   `reconvidar_creator` com o `mission_id`. A missão volta para `convidado`
   com prazo limpo e o creator é notificado automaticamente. Não precisa de
   mensagem manual pra esse grupo — a notificação já faz o toque.

4. **Dar o toque nos travados.** Creator em `fazendo` não se reconvida — a
   abordagem é conversa:
   - `abrir_conversa` com `campaign_id` para usar a sala da campanha (para
     DM direta com um creator específico, `abrir_conversa` também aceita
     escopo por creator — a API valida o formato)
   - `enviar_mensagem` com o `room_id` retornado e o texto combinado com o
     usuário

   **Nunca improvise o texto da cobrança.** Peça ao usuário a mensagem ou um
   rascunho pra ele aprovar — tom de cobrança é decisão da marca, não do
   agente. A mensagem é persistida e notifica o creator offline.

5. **Resumir o resultado**: quantos reconvidados, quantos contatados, e quem
   ficou de fora (e por quê).

## Erros e conflitos

- **`reconvidar_creator` em missão não expirada**: a API recusa — só vale pra
  `expirado`. Se falhar, recarregue `listar_missoes`: o status pode ter mudado
  desde o mapeamento (o creator pode ter entregue enquanto vocês decidiam).
- **Missão que voltou a mover sozinha**: sempre reliste antes de agir em lote;
  agir sobre estado velho é o erro clássico desse fluxo.
- **Tool de escrita ausente na sessão**: o login foi feito sem opt-in de
  escrita — avise que é preciso reconectar autorizando escrita.

## Limites

- Reconvite zera o prazo, não estende — se a campanha em si está no fim
  (`duration_days` perto do limite), reconvidar pode não dar tempo de entrega;
  verifique com `detalhar_campanha` antes de prometer prazo ao creator.
- Nada aqui remove creator da campanha — desistência de quem não responde é
  `remover_creator_campanha`, no fluxo da skill `acompanhar-missoes`.

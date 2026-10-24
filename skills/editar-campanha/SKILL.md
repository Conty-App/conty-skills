---
name: editar-campanha
description: Altera uma campanha ou acordo que já existe na Conty via MCP. Muda briefing, verba, datas, capa, público e convidados, renova os prazos de entrega de todos os creators, e pausa, reativa, encerra ou arquiva. Use quando o usuário pedir para editar, mudar, ajustar, melhorar o briefing, aumentar a verba, trocar a capa, estender o prazo de todos, pausar, reativar, encerrar ou arquivar uma campanha.
---

# Editar campanha

**Escopo necessário: escrita.**

## Fluxo

1. **Localizar.** `buscar_campanhas` com `q` (nome como a marca chama). Pegue o
   `campaign_id` de um bloco em `campaigns[]` (num acordo, o da rede pedida ou
   qualquer um).

2. **Ler o estado atual.** `detalhar_campanha` sempre antes de propor mudança.
   Repare em:
   - `acordo.group_id`: se existe, é **acordo**;
   - `has_blocking_missions: true`: há creator produzindo, e só verba, datas e
     comunidade são editáveis;
   - campo de texto com `[CORTADO]`: não reescreva inteiro sem o texto completo
     do usuário.

3. **Escolher a tool pelo campo:**

   | O que muda | Tool |
   |---|---|
   | Acordo: nome, capa, início/fim, verba, `creators_limit`, quem vê (pública ou só convidados) | `editar_acordo` com `group_id` |
   | Avulsa (ou bloco de acordo): briefing, roteiro, regras, hashtags, público, formato, pagamento etc. | `editar_campanha` com `campaign_id` |
   | Prazo de entrega de **todos** os creators, com ou sem novo fim | `renovar_prazos` |
   | Prazo de **um** creator | `estender_prazo_missao` (skill `reengajar-creators`) |
   | Pausar, reativar, encerrar, arquivar | `mudar_status_campanha` |
   | Capa nova | `subir_capa_campanha` → `cover_url` no `editar_acordo` ou `editar_campanha` |

   Mande **só** os campos que o usuário pediu. Para apagar um campo opcional no
   `editar_campanha`, mande `null` nele. Se faltar informação, pergunte tudo
   numa mensagem só.

4. **Lista de convidados do acordo** (`editar_acordo` com `target_user_ids`): é
   a lista INTEIRA. Some os novos aos que já estão em `target_user_ids` do
   `detalhar_campanha`, senão você tira gente. Quem entra num acordo no ar é
   convidado na hora. `null` volta a campanha a pública.

5. **Renovar prazos.** `renovar_prazos` com o `campaign_id` e ao menos uma data:
   `delivery_date`, `script_delivery_date` (só onde há roteiro),
   `specific_posting_date` (ISO com fuso, ex.: `2026-11-05T23:59:59-03:00`, e
   depois de agora) e/ou `end_date` (`AAAA-MM-DD`). Entregas vencidas voltam ao
   fluxo e os creators são avisados. No acordo vale para todos os blocos.

6. **Ciclo de vida** (`mudar_status_campanha`), sempre com confirmação:
   - `paused`: pausa uma campanha no ar
   - `approved`: **reativa** uma pausada (é este o valor para "voltar a
     veicular"; nunca `active`)
   - `review`: envia um rascunho para publicação
   - `completed`: **ENCERRA DE VEZ**, não reativa depois
   - `archived`: **TIRA DO AR**
   - `active`: só para desarquivar campanha avulsa

## Erros e conflitos

- **409 `managed_by_group`** no `editar_campanha`: o campo é do acordo, use
  `editar_acordo`. Acordo arquivado também não desarquiva pelo status.
- **409 com `has_blocking_missions`** (`editar_campanha`) ou
  **`campaign_fields_locked`** (`editar_acordo`): campo travado por creator
  produzindo. Objetivo, pagamento, público e redes nunca mudam nesse estado; a
  saída é campanha nova.
- **409 em `paused`/`completed`/`archived`**: missão em andamento bloqueia.
  Mostre a fila com `contagens_campanha`. Encerrar à força com pendências é só
  pelo painel.
- **409 `budget_exceeded`** ao reativar: o gasto já bateu a verba; ofereça
  aumentar a verba antes. **409 `insufficient_balance`**: falta saldo.
- **422 em `review`**: perfil da marca incompleto ou rascunho que não fecha as
  regras; a mensagem diz o que falta, complete com `editar_campanha`.
- **400 `payment_type_unavailable`** / **422 `perpetual_term_unavailable`**:
  formato ou prazo que a conta não aceita em mudança.

## Limites

- Termos que o creator lê (briefing, regras, hashtags) mudados com creator
  produzindo podem gerar uma versão nova dos termos, que ele aceita ou desiste
  sem penalidade. Avise o usuário antes.
- Prazo de pagamento só diminui; vagas não ficam abaixo das ocupadas.
- Entregas, escopos, briefings e pagamento de acordo mudam só pelo painel.
- Datas diferentes por bloco ou por escopo, só pelo painel.

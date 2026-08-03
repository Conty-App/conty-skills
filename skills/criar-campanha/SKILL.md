---
name: criar-campanha
description: Cria e publica uma campanha na Conty via MCP, do briefing à ativação — define orçamento, formato, pagamento e leva a campanha pelos status de review/approved/active. Use quando o usuário pedir para criar, montar, lançar ou publicar uma campanha, ou para ativar/pausar/encerrar uma campanha existente.
---

# Criar campanha

**Escopo necessário: escrita.** Sem o opt-in de escrita no login do MCP,
`criar_campanha` e `mudar_status_campanha` não existem na sessão.

## Fluxo

1. **Coletar o briefing.** Antes de chamar qualquer tool, garanta com o usuário
   os obrigatórios de `criar_campanha`:
   - `name` (5–100 caracteres)
   - `description` — o que a campanha quer, visível pros creators
   - `budget` — orçamento total (≥ 0)
   - `audience` — segmentação do público
   - `video_format` — formato do vídeo esperado

   E o modelo de pagamento, que define os campos seguintes:
   - `payment_type: "per_view"` → informar `pay_per_view_min` e `pay_per_view_max`
   - `payment_type: "per_video"` → informar `pay_per_video_value` e `slots`

   Opcionais úteis: `duration_days` (prazo da campanha) e `briefing_message`
   (mensagem que o creator recebe ao entrar). Campos de configuração avançada
   (flags de pré-aprovação de capa/legenda/roteiro, plataformas, formatos
   extras) vão no objeto `extra` — só inclua o que o usuário pedir
   explicitamente; não invente defaults por conta própria.

2. **Criar.** Chame `criar_campanha` com os campos coletados. A campanha
   **nasce em `review`** — não está no ar ainda. Guarde o `campaign_id`
   retornado.

3. **Conferir.** Chame `detalhar_campanha` com o `campaign_id` e apresente o
   resumo ao usuário (status, orçamento, pagamento, prazo, `share_url`).
   Confirme antes de avançar o status.

4. **Levar ao ar** (só com confirmação explícita do usuário). Use
   `mudar_status_campanha` com `campaign_id` e o `status` alvo, respeitando a
   máquina de estados:
   - `review` → `approved` → `active` (no ar, creators podem participar)
   - `active` ↔ `paused` (pausar/reativar)
   - `active` → `completed` (encerrar)
   - qualquer um → `archived` — **ARQUIVAR TIRA A CAMPANHA DO AR** (soft-delete)

## Erros e conflitos

- **409 ao pausar ou concluir**: há missões em andamento que bloqueiam a
  transição. Chame `contagens_campanha` pra mostrar ao usuário quantas missões
  estão ativas e decidir o que fazer antes de tentar de novo.
- **Validação (400)**: o corpo do erro da API diz qual campo reprovou —
  corrija e repita a chamada, sem mudar outra coisa.
- **403**: a conta não tem permissão de escrita na empresa do login. Confirme
  com o usuário se ele entrou com a conta certa.

## Limites

- Esta skill não convida creators nem distribui em comunidade — depois da
  campanha ativa, os fluxos são `buscar-creators` (descoberta) e as tools de
  comunidade (`compartilhar_campanha_comunidade`).
- Não existe voltar para `draft` — `mudar_status_campanha` não aceita `draft`
  como alvo.

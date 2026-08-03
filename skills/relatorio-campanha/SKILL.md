---
name: relatorio-campanha
description: Consolida os números e o status de uma campanha da Conty via MCP — contagens por aba, missões por status, candidaturas, orçamento e métricas de entrega em um resumo executivo. Use quando o usuário pedir um relatório, resumo, status, balanço ou "como está" uma campanha.
---

# Relatório de campanha

**Escopo necessário: leitura.** Funciona em qualquer sessão.

## Fluxo

1. **Localizar a campanha.** Se o usuário não deu o id, `listar_campanhas`
   com `q` (nome) ou `status`, e confirme qual campanha reportar.

2. **Coletar os dados em paralelo** — três chamadas independentes com o
   `campaign_id`:
   - `detalhar_campanha` — status atual, orçamento, modelo de pagamento,
     prazo, plataformas, flags de aprovação, contagens gerais e `share_url`
   - `contagens_campanha` — missões por status, convites de prospect
     pendentes, candidaturas (todas/pendentes/aprovadas/rejeitadas/expiradas)
   - `listar_missoes` — o detalhe por creator: status, link do vídeo,
     pagamento e métricas de cada entrega (use `limit`/`offset` se a campanha
     for grande; para um recorte, filtre por `statuses`)

3. **Montar o resumo**, nesta ordem:
   - **Situação**: nome, status da campanha, dias restantes (de
     `duration_days` e datas), orçamento e modelo de pagamento
   - **Funil**: candidaturas pendentes → aprovadas; convites pendentes
   - **Produção**: missões por status (convidados, fazendo, em análise, em
     alteração, aprovados, recusados) — destaque o que trava a operação:
     vídeos parados em análise e missões expiradas
   - **Pendências da marca**: itens que esperam decisão do usuário
     (candidaturas sem resposta, vídeos em análise, assets a revisar)

4. **Apontar próximos passos**, não só números: se há vídeos em análise,
   sugira a skill `acompanhar-missoes`; se há candidaturas pendentes,
   `triagem-candidaturas`. O relatório termina em ação.

## Erros e conflitos

- **Campanha arquivada**: ainda é possível reportar — `listar_campanhas` com
  `archived: true` a encontra. Avise no resumo que ela está fora do ar.
- **Listas grandes**: não despeje centenas de missões na resposta; agregue por
  status e detalhe só o recorte que o usuário pedir.

## Limites

- Métricas de comunidade (quando a campanha roda em comunidade) não estão
  aqui — use `analytics_comunidade` com o `community_id` para esse recorte.
- Esta skill só lê: nenhum número apresentado aqui foi alterado pelo
  relatório.

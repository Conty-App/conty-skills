---
name: relatorio-campanha
description: Consolida os números e o status de uma campanha da Conty via MCP. Junta situação, verba, funil de candidaturas, produção por etapa, entregas e pendências da marca num resumo executivo ou numa planilha. Use quando o usuário pedir relatório, resumo, status, balanço, planilha ou "como está" uma campanha.
---

# Relatório de campanha

**Escopo necessário: leitura.**

## Fluxo

1. **Localizar.** `buscar_campanhas` com `q` (traz acordos e avulsas como na
   tela). Use o `campaign_id` de um bloco; num acordo, qualquer bloco lê o
   acordo inteiro.

2. **Coletar em paralelo** (chamadas independentes):
   - `detalhar_campanha`: status, verba e valores, datas, flags de aprovação,
     plataformas, `share_url`, e no acordo os escopos e briefings;
   - `contagens_campanha`: missões por aba, convites de prospect pendentes e
     candidaturas (pendentes, aprovadas, rejeitadas, expiradas, pré-aprovadas);
   - `listar_entregas` com `campaign_id` e `etapa: "revisao"`: o que espera a
     marca.

3. **Detalhe por creator**, quando o usuário quiser ou a campanha for pequena:
   `listar_itens_campanha` com `view: "missions"` (progresso e pendências),
   `"applications"` (candidaturas) ou `"videos"` (entregas), `limit: 100` e
   `offset` para paginar. Pedido de **planilha** ou Excel: mesma tool com
   `as_spreadsheet: true` (as datas da etapa entram por padrão; só mande
   `include_dates: false` se pedirem sem datas).

4. **Montar o resumo**, nesta ordem:
   - **Situação**: nome, status (`approved` = no ar), dias até o fim, verba e
     modelo de pagamento;
   - **Funil**: candidaturas pendentes → aprovadas; convites pendentes;
   - **Produção**: missões por etapa, destacando o que trava: vídeos parados em
     análise, alterações pendentes, expirados;
   - **Pendências da marca**: candidaturas sem resposta, vídeos e assets a
     revisar.

   No acordo, "Aguardando envio" é creator aprovado ou convidado que ainda não
   recebeu briefing ou prazo (a agência envia pelo painel); não confunda com
   "Aguardando conteúdo", em que o creator já recebeu e não entregou.

5. **Terminar em ação**: vídeos em análise → `acompanhar-missoes`; candidaturas
   pendentes → `triagem-candidaturas`; creators parados → `reengajar-creators`.

## Perguntas que não cabem nas tools acima

`consultar_dados` lê as views da marca (`campaigns`, `campaign_groups`,
`missions`, `applications` e outras) para cortes específicos, ex.: missões
atrasadas, nota média dos aprovados, quanto foi pago. Prefira `query`; `sql`
cru só para JOIN ou janela. Use só quando nenhuma tool de cima responde.

## Erros e limites

- Campanha arquivada ainda é reportável: `buscar_campanhas` com
  `archived: true`. Avise que está fora do ar.
- Listas grandes: agregue por etapa e detalhe só o recorte pedido.
- Marca atendida por agência não recebe campos de dinheiro: ausência não é
  zero, diga que o valor não está disponível.
- Métricas de comunidade: `analytics_comunidade` com o `community_id`.
- Não exponha nem reconstrua link de vídeo ou arquivo: a disponibilidade da
  entrega é da rota autenticada.

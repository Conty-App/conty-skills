---
name: comparar-campanhas
description: Compara o desempenho de várias campanhas da Conty via MCP — funil de candidaturas, produção de missões, orçamento e modelo de pagamento lado a lado para ver o que está performando melhor. Use quando o usuário pedir para comparar campanhas, ver qual campanha vai melhor, ou um panorama consolidado de um período.
---

# Comparar campanhas

**Escopo necessário: leitura.** Funciona em qualquer sessão.

## Fluxo

1. **Definir o recorte com o usuário.** Comparar o quê? Campanhas ativas, de
   um período, de um modelo de pagamento? Use `listar_campanhas` com os
   filtros correspondentes:
   - `status` — ex.: `active` para o que está no ar, `completed` para
     retrospectiva
   - `payment_type` — `per_view` vs. `per_video`
   - `q` — recorte por nome/tema
   - `archived: true` só se o usuário quiser incluir arquivadas

2. **Fechar a shortlist antes de coletar.** Se a listagem trouxer muitas
   campanhas, apresente os nomes e confirme quais entram na comparação — 3 a 6
   é o limite prático de uma tabela legível. Não dispare coleta detalhada de
   dezenas de campanhas.

3. **Coletar os dados de cada campanha da shortlist** — duas chamadas por
   campanha, independentes entre si (podem ir em paralelo):
   - `detalhar_campanha` — status, orçamento, modelo de pagamento, prazo,
     plataformas
   - `contagens_campanha` — candidaturas (pendentes/aprovadas/rejeitadas/
     expiradas), convites pendentes, missões por status

   Se o usuário quiser profundidade em UMA campanha específica da comparação,
   aí sim chame `listar_missoes` para ela — não para todas.

4. **Montar a tabela comparativa**, uma linha por campanha:
   - nome, status, dias restantes
   - orçamento e modelo de pagamento
   - **funil de entrada**: candidaturas totais → aprovadas (taxa de aprovação)
   - **funil de produção**: convidados → fazendo → em análise → aprovados
     (taxa de entrega e de aprovação de conteúdo)
   - **gargalo**: onde cada campanha está travando (fila de análise grande,
     muitas expiradas, poucas candidaturas)

5. **Ler a tabela, não só exibi-la.** Aponte o que os números dizem: qual
   campanha converte melhor candidatura em entrega, qual está com funil
   entupido e onde (entrada ou produção), qual modelo de pagamento está
   rendendo mais entregas aprovadas. Termine com ação sugerida por campanha
   (triagem pendente → `triagem-candidaturas`; fila de análise →
   `acompanhar-missoes`; detalhe de uma → `relatorio-campanha`).

## Erros e conflitos

- **Campanha com poucos dados**: campanha recém-criada distorce taxas — marque
  na tabela em vez de comparar percentuais de base 2 com base 200.
- **Recortes incomparáveis**: avise se o usuário misturar status muito
  diferentes (uma ativa há 2 dias vs. uma concluída) — compare fase com fase
  ou normalize por tempo no ar.

## Limites

- Comparação é entre campanhas da empresa do login; não há recorte
  cross-empresa.
- Métricas de comunidade ficam de fora — para elas, `analytics_comunidade` por
  comunidade.
- Só leitura: nada é alterado pela comparação.

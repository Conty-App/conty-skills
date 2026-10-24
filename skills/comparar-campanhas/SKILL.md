---
name: comparar-campanhas
description: Compara o desempenho de campanhas da Conty via MCP ou mostra a evolução da marca no tempo. Ranqueia campanhas por views, curtidas, creators, verba ou gasto, compara funil e produção lado a lado e traça a curva de entregas e views num período. Use quando o usuário pedir para comparar campanhas, ver qual vai melhor, ranking, panorama de um período, ou como a marca evoluiu.
---

# Comparar campanhas

**Escopo necessário: leitura.**

## Escolher a fonte pelo eixo da pergunta

Errar isso responde a pergunta que o usuário não fez.

| Eixo | Tool |
|---|---|
| **Campanha contra campanha** (qual rendeu mais, ranking, compare) | `listar_campanhas` |
| **Tempo** (como evoluiu, este mês vs. o passado, está crescendo) | `panorama_da_marca` |
| Corte que nenhuma das duas faz | `consultar_dados` |

## Fluxo: campanha contra campanha

1. **Listar com métricas.** `listar_campanhas` traz views, curtidas, creators e
   verba de cada. Parâmetros:
   - `visualization`: `performance` para ranking de uma métrica (`metric`:
     `views`, `likes`, `creators`, `budget` ou `spent`; `chart: "bar"` para
     ranking, `"donut"` para participação), `status` para distribuição por
     situação, `table` para lista;
   - `status` só se o usuário nomear um (`approved` = no ar, `completed` para
     retrospectiva); sem ele vêm as não arquivadas e `status_counts`;
   - `payment_type`, `q`, `order_by` (`created_at` ou `end_date`);
   - "a última campanha" é `limit: 1`. Portfólio inteiro: `limit: 100` e
     pagine até cobrir `total` antes de dizer que analisou tudo.

   `listar_campanhas` só traz **campanhas avulsas**. Se a marca trabalha com
   acordos, liste-os com `buscar_campanhas` e colete cada um pelo passo 2.

2. **Funil e produção** de uma shortlist de 3 a 6 campanhas, em paralelo por
   campanha: `detalhar_campanha` + `contagens_campanha`. Não dispare isso para
   dezenas de campanhas.

3. **Montar a tabela**, uma linha por campanha: status, dias no ar, verba e
   modelo de pagamento, candidaturas → aprovadas (taxa), convidados → fazendo
   → em análise → aprovados (taxa de entrega), views/curtidas, e o **gargalo**
   (fila de análise, expirados, poucas candidaturas).

4. **Ler a tabela.** Diga qual converte melhor candidatura em entrega, qual
   está entupida e onde, e termine com ação por campanha (`triagem-candidaturas`,
   `acompanhar-missoes`, `relatorio-campanha`).

## Fluxo: evolução no tempo

`panorama_da_marca` com `from` e `to` (`AAAA-MM-DD`) e, se pedirem,
`payment_type`. Traz entregues/aprovados/em revisão, views e curtidas do
Instagram e a curva diária; com acesso ao financeiro, valor previsto e pago.
Para "este mês vs. o passado", chame duas vezes, um período cada. Aqui os
números são da marca somada: nenhuma campanha aparece separada.

## Erros e limites

- **Permuta**: verba, saldo e gasto vêm 0 sem significar orçamento esgotado.
  Deixe de fora de ranking de dinheiro.
- **Campanha recém-criada** distorce taxa: marque em vez de comparar base 2 com
  base 200. Compare fase com fase.
- **Marca atendida por agência** não recebe campos de dinheiro: ausência não é
  zero.
- Comparação é dentro da marca ativa; outra marca só com `enterprise_id`
  quando o usuário a nomear.

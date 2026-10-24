---
name: buscar-creators
description: Encontra e avalia creators na base da Conty via MCP. Acha um creator pelo @handle, nome ou link do perfil, descobre creators por critério (nicho, seguidores, cidade, gênero, idade, engajamento, nota, brand safety) e abre perfil, números da rede e histórico na Conty. Use quando o usuário pedir para procurar, encontrar, pesquisar ou recomendar creators/influenciadores, ou ver o perfil e as métricas de um creator.
---

# Buscar creators

**Escopo necessário: leitura.** Nada aqui altera estado.

## Escolher o caminho pelo pedido

1. **Creator específico** ("perfil do @fulana", link do Instagram, nome):
   `buscar_creators` com `query` (o @ é opcional, maiúsculas e acentos não
   importam), `page: 1`. Lista longa de @handles completos vai por
   `resolver_handles_creators` (até 50 por chamada; acima disso, fatie). Handle
   que não resolveu ali passa por `buscar_creators` com `query` antes de
   concluir que não está na base. Só diga que não está depois disso, e peça o @
   exato, sem tentar variações.

2. **Descoberta por critério de perfil** (nicho, seguidores, cidade, UF,
   gênero, idade, engajamento, nota): `consultar_dados` com `query` na view
   `creators_catalog`, ordenada pela métrica pedida (`instagram_followers`,
   `engagement_rate` ou `average_score`), `limit: 12` por padrão. Ex.:

   ```json
   {
     "pergunta": "creators de culinária em São Paulo com mais de 20 mil seguidores",
     "query": {
       "view": "creators_catalog",
       "columns": ["creator_id", "name", "handle", "city", "instagram_followers", "engagement_rate"],
       "filters": [
         { "column": "business_category", "op": "ilike", "value": "gastronom" },
         { "column": "state", "op": "eq", "value": "SP" },
         { "column": "instagram_followers", "op": "gte", "value": 20000 }
       ],
       "orderBy": [{ "column": "instagram_followers", "direction": "desc" }],
       "limit": 12
     },
     "visualizacao": "tabela",
     "titulo": "Creators de culinária em SP",
     "finalidade": "resposta"
   }
   ```

   Nicho é rótulo armazenado (`moda`, `beleza`, `fitness`, `tecnologia`,
   `gastronomia`, `viagens`, `lifestyle`, `games`, `educacao`, `negocios`,
   `maternidade`, `pets`, `humor`, `casa`): use `ilike` no radical.

3. **Assunto fora dos rótulos** ("finanças", "skincare") ou **ranking de match
   para uma campanha**: `buscar_creators` com `niche` no termo como o usuário
   disse e os filtros que ele citou (`state`, `city`, `followersRange` como
   `"20000+"`, `ageMin`/`ageMax`, `minEngagementRate` em %, `brandSafety`,
   `audienceGender`). `sortBy: "rating-desc"` só se pedirem os mais bem
   avaliados. Com `hasMore`, avance `page` antes de concluir que viu tudo.

**Vazio é resposta.** Diga que não achou no recorte e pergunte se quer ampliar
(ex.: para a categoria). Nunca repita a busca com sinônimo por conta própria, e
nunca afirme o total da base.

## Avaliar um creator

Com o `user_id`:
- `ver_perfil_creator`: bio, nicho, redes e marcas com quem já trabalhou;
- `ver_social_creator` (`platform`, padrão `instagram`): seguidores e
  engajamento;
- `ver_analytics_creator`: histórico na Conty (entregas, prazo, preço,
  revisões, audiência estimada, similares).

Para casting, o histórico pesa mais que seguidores: prazo e revisões dizem como
é trabalhar com a pessoa.

## Próximo passo

A skill termina na shortlist. Para chamar os creators para uma campanha:
`convidar_creators_campanha`, na skill `triagem-candidaturas`.

## Erros e limites

- `consultar_dados` recusado: corrija uma vez pelo motivo; se recusar de novo,
  pare e explique. View inexistente: fonte indisponível.
- Marca atendida por agência pode não ter acesso à descoberta; nesse caso diga
  que o recurso não está disponível para a conta.
- Creator de fora da plataforma não está na base: entra por convite de
  prospect, pelo painel.

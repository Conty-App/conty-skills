---
name: buscar-creators
description: Descobre e avalia creators na base da Conty via MCP — busca por nicho, plataforma, seguidores, localização e abre o perfil completo com métricas de entrega e audiência. Use quando o usuário pedir para procurar, encontrar ou pesquisar creators/influenciadores, ou para ver o perfil e as métricas de um creator específico.
---

# Buscar creators

**Escopo necessário: leitura.** Funciona em qualquer sessão, inclusive sem
opt-in de escrita.

## Fluxo

1. **Buscar na base.** Chame `buscar_creators` com os filtros que o usuário
   mencionar — todos opcionais, combine livremente:
   - `query` — texto livre (nome, tema)
   - `niche` — nicho de conteúdo
   - `platform` — plataforma (ex.: instagram, tiktok, youtube)
   - `followersRange` — faixa de seguidores
   - `location`, `gender`, `profession`
   - `ageMin`/`ageMax` ou `ageRange` — faixa etária
   - `sortBy`: `relevance` (padrão) ou `rating-desc`
   - paginação por `page`/`limit`

   Se a busca vier vazia, afrouxe um filtro por vez (comece removendo os mais
   restritivos: localização e faixa de seguidores) em vez de repetir a mesma
   consulta.

2. **Apresentar a lista** de forma comparável: nome, nicho, plataforma,
   seguidores. Pergunte quais perfis o usuário quer abrir.

3. **Avaliar um perfil.** Chame `ver_perfil_creator` com o `user_id` do
   creator (e `platform` se o usuário quiser uma rede específica — o padrão é
   instagram). O retorno traz três blocos:
   - `public_profile` — bio, nicho, redes, marcas com quem já trabalhou
   - `social_profile` — seguidores e engajamento na plataforma
   - `profile_analytics` — histórico na Conty: entregas, cumprimento de prazo,
     preço médio, revisões, perfil da audiência e creators similares

   Para decisão de casting, o `profile_analytics` pesa mais que seguidores:
   prazo e revisões dizem como é trabalhar com a pessoa.

4. **Próximo passo.** Esta skill termina na shortlist. Convidar para uma
   campanha específica é outro fluxo — os creators encontrados aqui entram na
   campanha via candidatura ou convite (ver skill `triagem-candidaturas`).

## Erros e conflitos

- **Lista vazia não é erro** — é filtro demais. Afrouxe e tente de novo.
- **`user_id` inválido**: confirme que o id veio de uma listagem desta sessão;
  ids de conversas antigas podem estar errados.

## Limites

- A busca cobre a base Conty; creator de fora da plataforma entra por convite
  de prospect na campanha, não por aqui.
- Nada nesta skill altera estado — é seguro explorar à vontade.

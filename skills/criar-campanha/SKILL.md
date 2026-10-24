---
name: criar-campanha
description: Cria e publica uma campanha na Conty via MCP, do briefing ao ar. Monta o acordo (objetivo, entregas, escopos, briefings, verba e valor por conteúdo), sobe capa e anexo de briefing, salva rascunho ou publica. Também duplica uma campanha existente. Use quando o usuário pedir para criar, montar, lançar, abrir, publicar ou duplicar uma campanha.
---

# Criar campanha

**Escopo necessário: escrita.** Sem o opt-in de escrita no login do MCP, as
tools de criação não existem na sessão.

## Qual tool cria

- **`publicar_acordo`**: o caminho padrão. Cria a campanha como acordo, no
  formato do formulário do painel. Se ela está na sessão, use sempre ela.
- **`criar_campanha`**: trilho legado de campanha avulsa. Só aparece em conta
  sem acordos ligados; nesse caso `publicar_acordo` não existe.

Antes de criar, confira que a marca não está falando de uma campanha que já
existe (`buscar_campanhas` com `q`). Mudar briefing, verba, prazo ou formato de
campanha existente é a skill `editar-campanha`, nunca campanha nova.

## Fluxo com `publicar_acordo`

1. **Coletar o briefing, numa mensagem só.** Pergunte tudo o que falta de uma
   vez, nunca uma pergunta por turno:
   - `name` e `objective` (`vendas`, `views`, `branding` ou `criativos`)
   - `start_date` e `end_date` (`AAAA-MM-DD`)
   - `budget` (verba total em reais) e `pay_per_video_value` (valor por
     conteúdo, o que o creator recebe pelo escopo completo)
   - as **entregas** (`platforms[]`): rede (`platform`: `reels_collab`,
     `tiktok`, `youtube_longo`...), formato (`delivery_formats`), quantidade
     (`videos_count`) e prazos de roteiro/entrega/postagem
   - os **briefings** (`briefings[]`: `title` + `body`)
   - direito de uso (`usage_rights_duration`), impulsionamento
     (`boost_duration`) e exclusividade (`exclusivity_duration`): só `30_days`,
     `90_days` ou `1_year`. **Prazo perpétuo não existe mais.**

   Opcionais que só entram se o usuário pedir: `scope_options` (pacotes de
   entregas diferentes, um por escopo; também é como se faz teste A/B, um
   escopo e um briefing por gancho), `creator_niches`, faixa de seguidores,
   hashtags e termos proibidos, regras de participação (`participation`:
   aprovação de roteiro, capa, legenda, envio de produto) e campanha só para
   convidados (`target_user_ids`, com os ids de `buscar-creators`).

   **Nunca invente verba, valor por conteúdo nem data.**

2. **Permuta** (só onde a conta oferece): `payment_type: "permuta"` exige
   `permuta_item` (nome curto do produto), verba 0, e `creators_limit` é a
   quantidade de itens. Em `per_video`, o mesmo produto pode vir **além** do
   cachê. Produto que não é físico (assinatura, crédito, voucher) leva
   `ships_product: false`.

3. **Capa** (opcional): se o usuário mandou imagem, `subir_capa_campanha` com
   `source_url` (link https aberto) ou `content_base64`. Ela só devolve o
   `cover_url`: passe-o no `publicar_acordo`. Capa é horizontal 3:1, de
   1200 × 400 para cima.

4. **Rascunho ou publicação.** Mostre o resumo do acordo ao usuário e pergunte:
   - `status: "draft"`: só salva o rascunho; a marca termina e publica pelo
     painel.
   - `status: "review"` (padrão): **PUBLICA E COMPROMETE A VERBA.** Com saldo,
     o acordo já entra `approved` (no ar).

5. **Anexo do briefing** (opcional, depois de criado): `anexar_arquivo_briefing`
   com o `group_id` que o `publicar_acordo` devolveu, `file_name` e o arquivo
   por `source_url` (link do Google Drive/Docs aberto para "qualquer pessoa com o
   link" vira PDF) ou `content_base64`. PDF, JPG ou PNG até 10 MB. Com mais de
   um briefing, passe `briefing_id`.

6. **Conferir.** `detalhar_campanha` com o `campaign_id` de um bloco do acordo
   (`buscar_campanhas` traz os blocos em `campaigns[]`) e mostre status,
   verba, datas e `share_url`.

## Duplicar campanha

Leia a original com `detalhar_campanha` e mande os campos dela no
`publicar_acordo`, trocando o que o usuário pediu. Se a original tinha prazo
perpétuo, pergunte qual de `30_days`, `90_days` ou `1_year` usar. Reaproveite o
`cover_url` dela se o usuário não mandar outra capa.

## Fluxo com `criar_campanha` (conta sem acordos)

Obrigatórios: `name` (≥ 5 caracteres), `description`, `budget`, `audience`,
`video_format` e `end_date` ou `duration_days`. O pagamento define o resto:
- `per_video` → `pay_per_video_value` e `slots`
- `per_view` → `pay_per_view_min` (teto `pay_per_view_max` opcional); há contas
  que não oferecem
- `permuta` → `permuta_item`, `budget: 0`, `slots` = quantidade de itens

Nasce em `review` (fila de revisão), ou `status: "draft"` para rascunho.
`requires_ad_code` só com `video_format: "Reels_Collab"`.

## Erros e conflitos

- **400 `payment_type_unavailable`**: a conta não oferece esse formato
  (por view ou permuta). Explique e ofereça `per_video`, sem tentar de novo.
- **422 `perpetual_term_unavailable`**: prazo perpétuo; troque por
  `30_days`/`90_days`/`1_year` com o usuário.
- **422 `permuta_item_required`** / **`permuta_product_not_allowed`**: falta o
  nome do produto, ou produto em `per_view`.
- **422 `end_date_required`** (avulsa): campanha publicada precisa de fim.
- **409 `insufficient_balance`** ou recusa por perfil incompleto (logo e
  descrição da marca): a marca precisa depositar ou completar o perfil. Ofereça
  salvar como `draft` enquanto isso.
- **Validação (400/422)**: o corpo diz qual campo reprovou; corrija só ele.

## Limites

- Publicar compromete verba da marca: nunca publique sem confirmação explícita.
- Esta skill não convida creators: depois de no ar, `triagem-candidaturas`
  (convite e candidaturas) e `buscar-creators` (descoberta).
- Entregas, escopos e briefings de acordo já criado só mudam pelo painel.

---
name: triagem-candidaturas
description: Faz a triagem de candidaturas de uma campanha na Conty via MCP — lista candidatos, aprova (colocando o creator na campanha), rejeita e cancela convites pendentes. Use quando o usuário pedir para ver, aprovar, rejeitar ou gerenciar candidaturas, interessados ou convites de uma campanha.
---

# Triagem de candidaturas

**Escopo necessário: escrita** para decidir (`decidir_candidatura`,
`cancelar_convite`). A listagem (`listar_itens_campanha`) é leitura e funciona
em qualquer sessão.

## Fluxo

1. **Localizar a campanha.** Se o usuário não deu o id, chame
   `listar_campanhas` (filtre por `status` ou `q` com o nome) e confirme qual
   campanha antes de seguir.

2. **Ver o tamanho da fila.** Chame `contagens_campanha` com o `campaign_id`:
   ele devolve candidaturas totais, pendentes, aprovadas, rejeitadas e
   expiradas, além dos convites de prospect pendentes.

3. **Listar as candidaturas.** Chame `listar_itens_campanha` com:
   - `campaign_id`
   - `view: "applications"`
   - `status: "pending"` (ou um CSV como `"pending,approved"` para misturar)
   - ordenação por `sortBy`: `match` (aderência), `rating`, `followers`,
     `recent`, `oldest`
   - `only_compatible: true` se o usuário quiser só candidatos compatíveis
   - paginação com `limit`/`offset`

   Apresente a lista com nome, seguidores e o que mais ajudar a decisão. Para
   avaliar um candidato a fundo antes de decidir, use `ver_perfil_creator` com
   o `user_id` dele (bio, redes, engajamento, histórico de entregas e prazo).

4. **Decidir, um a um, com confirmação.** Para cada candidatura, chame
   `decidir_candidatura` com `campaign_id`, `application_id` e `status`:
   - `approved` — o creator **entra na campanha** (a missão dele nasce como
     `convidado`) e é notificado. Em campanhas `per_video`, passe
     `payment_amount` para fixar o valor pago; omitindo, vale o valor padrão da
     campanha.
   - `rejected` — a candidatura é encerrada e o creator é notificado.

5. **Convites pendentes** (creators de fora da plataforma): para desistir de um
   convite ainda não aceito, chame `cancelar_convite` com o `invite_id`.
   Convite já aceito não se cancela por aqui — vira gestão de missão
   (skill `acompanhar-missoes`).

## Erros e conflitos

- **Candidatura já decidida**: a API recusa a segunda decisão — recarregue a
  lista (passo 3) antes de continuar a triagem, o estado pode ter mudado.
- **409/validação**: leia o corpo do erro e corrija só o campo apontado.
- **Tool de escrita ausente na sessão**: o login foi feito sem opt-in de
  escrita. Avise o usuário que é preciso reconectar autorizando escrita.

## Limites

- Decisões notificam o creator imediatamente — não use "pra testar".
- Esta skill não aprova vídeos nem paga ninguém; aprovação de candidatura só
  coloca o creator na campanha.

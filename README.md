# conty-skills

Skills para **agentes de IA** (Claude Code, Kimi Code, Cursor etc.) operarem a
Conty via MCP. Este repositório é para quem quer tocar a operação da plataforma
(campanhas, acordos, entregas, creators, comunidades e chat) conversando com um
agente, sem abrir o painel.

Cada skill ensina o agente a executar uma **tarefa da marca** de ponta a ponta:
quais tools chamar, em que ordem, com quais parâmetros, e o que fazer quando a
API responde erro. Não há skill por tool: a unidade aqui é o fluxo de trabalho.

## Instalação

### Com o skills CLI (recomendado)

O caminho mais rápido é o [skills CLI](https://skills.sh) da Vercel, que
detecta as skills do repositório e instala direto no seu agente:

```bash
# instala as skills no(s) agente(s) detectado(s), com prompt pra escolher
npx skills add Conty-App/conty-skills

# instala TODAS as skills em TODOS os agentes, sem prompt
npx skills add Conty-App/conty-skills --all

# escolhe skill e agente específicos
npx skills add Conty-App/conty-skills --skill criar-campanha --agent claude-code

# instala globalmente (disponível em todos os seus projetos)
npx skills add Conty-App/conty-skills -g
```

Funciona com Claude Code, Kimi Code, Cursor, Codex e mais de 70 agentes.
Para atualizar as skills depois: `npx skills update`.

### Manual

Clone o repositório e copie os diretórios desejados para a pasta de skills do
projeto/usuário:

```bash
git clone https://github.com/Conty-App/conty-skills.git
```

- **Kimi Code**: copie `skills/<nome>` para `.agents/skills/` (projeto) ou
  `~/.kimi-code/skills/` (usuário).
- **Claude Code**: copie para `.claude/skills/` ou `~/.claude/skills/`.

Pré-requisito: o agente precisa estar conectado ao MCP da Conty
(`https://mcp.appconty.com/mcp`) e autenticado.

## Skills disponíveis

| Skill | Tarefa | Escopo |
|---|---|---|
| `criar-campanha` | Criar e publicar um acordo (ou campanha avulsa) do briefing ao ar | escrita |
| `editar-campanha` | Mudar campos, capa, prazos e ciclo de vida de campanha que já existe | escrita |
| `triagem-candidaturas` | Decidir candidaturas em lote e convidar creators | escrita |
| `acompanhar-missoes` | Revisar entregas: aprovar, pedir alteração, recusar, revisar capa/legenda/roteiro | escrita |
| `reengajar-creators` | Dar mais prazo, reconvidar expirados e cobrar quem parou | escrita |
| `buscar-creators` | Achar creators por @, por critério de perfil ou por match com a campanha | leitura |
| `relatorio-campanha` | Consolidar números, funil e pendências de uma campanha | leitura |
| `comparar-campanhas` | Comparar campanhas lado a lado ou ver a evolução da marca no tempo | leitura |

## Regras que valem para todas as skills

- **O catálogo é dinâmico.** As tools vêm da conty-api ao abrir a sessão e são
  filtradas pelo que a sua conta enxerga: escopo do login, cargo na marca e
  módulos do tenant. Tool que "não existe" é quase sempre um desses três, não
  bug:
  - **escrita** só aparece se você marcou o opt-in de escrita na tela de login
    do MCP (`campaigns:write`); sem ele, reconecte autorizando;
  - **cargo**: membro sem a permissão da ação não recebe a tool;
  - **módulo**: com acordos ligados (`campaign_groups`), `criar_campanha` some
    e quem cria é `publicar_acordo`; sem o módulo de comunidades, as tools de
    comunidade não aparecem.
- **Qual marca você opera.** Sem `enterprise_id`, toda tool opera a marca do
  login. Para operar outra marca da qual você é dono, membro ou agência, chame
  `listar_minhas_marcas` e passe o `id` dela em `enterprise_id` em **cada**
  chamada. Só troque de marca quando o usuário nomear a outra; não achar uma
  campanha na marca atual não autoriza procurar nas outras.
- **Status da campanha**: `draft`, `review`, `approved`, `paused`, `completed`,
  `archived`. **`approved` é NO AR** (aprovada, ativa, rodando: tudo a mesma
  coisa). Nunca diga que a marca não tem campanha ativa se ela tem uma em
  `approved`.
- **Acordo × campanha avulsa.** Acordo (campanha em grupo) é uma campanha só
  para a marca, com um bloco por rede (`campaigns[]` no `buscar_campanhas`).
  As tools de campanha recebem o `campaign_id` de um bloco; os campos do acordo
  (nome, capa, datas, verba, convidados) mudam pelo `editar_acordo`, com o
  `acordo.group_id` do `detalhar_campanha`.
- **Ação em lote é uma chamada só.** Decidir candidatura, pedir alteração,
  recusar vídeo e convidar creators recebem uma LISTA de alvos (até 500 nas
  decisões). Nunca repita a tool creator por creator.
- **Ação sobre várias campanhas de uma vez não existe**: trate uma por vez.
- **Ações destrutivas se confirmam.** O cliente MCP pede aprovação nas tools
  marcadas como destrutivas; mesmo assim, mostre ao usuário o que vai acontecer
  (quem é notificado, o que é irreversível) antes de chamar.
- **Erros da API vêm crus.** O MCP devolve o corpo de erro original com
  `HTTP <status>`. Leia o `code` e corrija a chamada; conflito de estado (409)
  é informação, não falha da tool. Um 422 de "falta escolher" (escopo,
  briefing, data) traz o que perguntar ao usuário.
- **Texto que o creator lê é da marca.** Feedback de recusa, motivo, recado e
  mensagem de chat: peça ao usuário ou mostre um rascunho para ele aprovar,
  nunca invente.

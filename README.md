# conty-skills

Skills para **agentes de IA** (Claude Code, Kimi Code, Cursor etc.) operarem a
Conty via MCP. Este repositório é para quem quer tocar a operação da plataforma
— campanhas, missões, creators, comunidades e chat — conversando com um agente,
sem abrir o painel.

Cada skill ensina o agente a executar uma **tarefa da marca** de ponta a ponta:
quais tools chamar, em que ordem, com quais parâmetros, e o que fazer quando a
API responde erro. Não há skill por tool — a unidade aqui é o fluxo de trabalho.

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

Pré-requisito: o agente precisa estar conectado ao MCP da Conty e autenticado.

## Skills disponíveis

| Skill | Tarefa | Escopo |
|---|---|---|
| `criar-campanha` | Criar e publicar uma campanha do briefing à ativação | escrita |
| `triagem-candidaturas` | Aprovar, rejeitar e gerir candidaturas e convites | escrita |
| `acompanhar-missoes` | Revisar entregas: pedir alteração, recusar, reconvidar | escrita |
| `reengajar-creators` | Reconvidar expirados e dar um toque em creators parados | escrita |
| `buscar-creators` | Descobrir creators na base e avaliar perfis | leitura |
| `relatorio-campanha` | Consolidar números e status de uma campanha | leitura |
| `comparar-campanhas` | Comparar funil e desempenho de várias campanhas lado a lado | leitura |

## Regras que valem para todas as skills

- **Escopo leitura vs. escrita**: tools de escrita só existem na sessão se a
  marca deu opt-in de escrita na tela de login do MCP. Se uma tool de escrita
  "não existe", o motivo é quase sempre esse — não é bug.
- **Empresa alvo**: o MCP não guarda "empresa atual" em sessão — quem você
  opera é sempre a empresa do login. Nunca assuma contexto de conversas
  anteriores; confirme a marca antes de qualquer escrita.
- **Erros da API vêm crus**: o MCP devolve o corpo de erro original. Leia a
  mensagem e corrija a chamada; conflitos de estado (ex.: 409 ao pausar
  campanha com missões em andamento) são informação, não falha da tool.

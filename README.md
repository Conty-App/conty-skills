# conty-skills

Skills para **agentes de IA** (Claude Code, Kimi Code, Cursor etc.) operarem a
Conty via MCP. Este repositório é para quem quer tocar a operação da plataforma
— campanhas, missões, creators, comunidades e chat — conversando com um agente,
sem abrir o painel.

Cada skill ensina o agente a executar uma **tarefa da marca** de ponta a ponta:
quais tools chamar, em que ordem, com quais parâmetros, e o que fazer quando a
API responde erro. Não há skill por tool — a unidade aqui é o fluxo de trabalho.

## Instalação

Clone o repositório e aponte a pasta de skills do seu agente para ele, ou
copie os diretórios desejados para a pasta de skills do projeto/usuário:

```bash
git clone <url-do-repo> conty-skills
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
| `buscar-creators` | Descobrir creators na base e avaliar perfis | leitura |
| `relatorio-campanha` | Consolidar números e status de uma campanha | leitura |

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

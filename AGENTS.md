# conty-skills — convenções

Repositório de skills para usuários do conty-mcp. Sem build, sem dependências:
só markdown.

## Estrutura

```
skills/<nome-da-skill>/SKILL.md
```

Um diretório por skill, um `SKILL.md` por diretório. Nome do diretório em
kebab-case, igual ao `name` do frontmatter.

## Formato do SKILL.md

Frontmatter YAML obrigatório:

```yaml
---
name: nome-da-skill
description: Uma frase que diz o que a skill faz E quando ativá-la, com as
  palavras que o usuário usa ("use quando pedir para..."). A description é o
  gatilho de ativação — é interface, não comentário.
---
```

Corpo, nesta ordem:

1. **Escopo necessário** — leitura ou escrita (escrita exige opt-in no login do
   MCP; sem ele as tools nem aparecem na sessão).
2. **Fluxo** — passos numerados: qual tool chamar, com quais parâmetros
   essenciais, em que ordem. Use os nomes reais das tools do MCP.
3. **Erros e conflitos** — o que fazer com as respostas de erro da API (que
   chegam cruas): 409 de máquina de estados, 403 de permissão, validação.
4. **Limites** — o que a skill não faz, o que é irreversível, o que depende de
   outra skill.

## Regras

- **Fidelidade ao MCP**: só cite tools, parâmetros e transições que existem no
  conty-mcp. Se a skill precisa de uma capacidade que não existe, o trabalho é
  na conty-api/conty-mcp, não na skill — nunca ensine o modelo a simular
  endpoint compondo chamadas.
- **Português**, do jeito que a marca fala da operação.
- **Ações destrutivas se anunciam**: se um passo é definitivo ou tira algo do
  ar, diga isso no próprio passo, em maiúsculas, como as próprias tools fazem.
- Exemplos de parâmetros devem ser realistas (enums e formatos reais do schema),
  nunca placeholders abstratos tipo `"valor"`.
- Nada de skill por tool. Se duas skills repetem um bloco de instrução,
  considere extrair a instrução comum pro README ou referenciar a outra skill.

## Antes de dar por pronto

- Conferir cada nome de tool e parâmetro citado contra `conty-mcp/src/*/*.tools.ts`
  e `*.schemas.ts`.
- Testar a skill de verdade: rodar o fluxo com o MCP conectado e ver se o
  modelo segue sem improvisar.

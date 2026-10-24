# conty-skills: convenções

Repositório de skills para usuários do conty-mcp. Sem build, sem dependências:
só markdown.

## De onde vêm as tools

O conty-mcp não declara tools. O catálogo mora na **conty-api**, em
`src/features/ai/tools/catalog/*.catalog.ts` (mais `consultar_dados`, do plugin
`insights`), e é o mesmo que o Copilot do painel usa. O MCP busca
`GET /v1/ai/tool-catalog` ao abrir a sessão. Tool nova entra lá; aqui só se
ensina o fluxo.

Consequência: **toda mudança no catálogo pode deixar estas skills velhas.**
Nome de tool, parâmetro, enum e código de erro citados aqui têm que bater com o
catálogo atual.

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
  gatilho de ativação: é interface, não comentário.
---
```

Corpo, nesta ordem:

1. **Escopo necessário**: leitura ou escrita (escrita exige opt-in no login do
   MCP; sem ele as tools nem aparecem na sessão).
2. **Fluxo**: passos numerados: qual tool chamar, com quais parâmetros
   essenciais, em que ordem. Use os nomes reais das tools.
3. **Erros e conflitos**: o que fazer com as respostas de erro da API (que
   chegam cruas): 409 de máquina de estados, 403 de permissão, 422 de validação,
   com o `code` real.
4. **Limites**: o que a skill não faz, o que é irreversível, o que depende de
   outra skill.

## Regras

- **Fidelidade ao catálogo**: só cite tools, parâmetros e transições que existem
  no catálogo da conty-api. Se a skill precisa de uma capacidade que não existe,
  o trabalho é na conty-api, não na skill: nunca ensine o modelo a simular
  endpoint compondo chamadas.
- **Não repita a descrição da tool.** O modelo já recebe a descrição e o schema
  de cada tool. A skill diz a ordem, a escolha entre tools parecidas e o que
  perguntar ao usuário.
- **Português**, do jeito que a marca fala da operação.
- **Ações destrutivas se anunciam**: se um passo é definitivo ou tira algo do
  ar, diga isso no próprio passo, em maiúsculas, como as próprias tools fazem.
- Exemplos de parâmetros devem ser realistas (enums e formatos reais do schema),
  nunca placeholders abstratos tipo `"valor"`.
- Nada de skill por tool. Regra que vale para todas vai no README.
- Sem travessão em texto: use vírgula, dois pontos ou parênteses.

## Antes de dar por pronto

- Conferir cada nome de tool, parâmetro e enum citado contra o catálogo. O jeito
  mais rápido é gerar o dump a partir do checkout da conty-api:

  ```ts
  // dump.ts, rodar na raiz da conty-api: npx tsx dump.ts > catalog.json
  import { z } from "zod";
  import { toolCatalog } from "./src/features/ai/tools/catalog/index.ts";
  console.log(JSON.stringify(toolCatalog().map((s) => ({
    name: s.name, access: s.access, description: s.description,
    input: z.toJSONSchema(s.input),
  })), null, 1));
  ```

- Testar a skill de verdade: rodar o fluxo com o MCP conectado e ver se o
  modelo segue sem improvisar.

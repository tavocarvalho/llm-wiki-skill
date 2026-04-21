# Workflows detalhados

Passo-a-passo completo dos 5 workflows. Leia a seção relevante antes de executar se não se lembra do detalhe.

## 1. Ingest — arquivar uma nova fonte

**Quando usar**: usuário colocou um arquivo em `raw/sources/` (ou pediu pra colocar) e quer que vire páginas de wiki.

### Passo 1 — Ler a fonte inteira

Use `Read`. Para PDFs grandes (>10 páginas), leia em faixas de 20 via `pages: "1-20"`, `pages: "21-40"`, etc. Não fake skimming — o wiki depende da qualidade dessa leitura.

Se a fonte tem imagens relevantes que o texto refere, abra cada imagem individualmente com `Read` após ler o texto.

### Passo 2 — Propor TL;DR e esperar confirmação

Apresente ao usuário:
- 3-5 bullets do que a fonte diz.
- Lista preliminar de entidades e conceitos que você detectou e vai criar páginas pra.

**Espere resposta explícita** ("pode seguir", "ok", "vai") antes de escrever qualquer arquivo. Este é o anti-padrão #1: ingerir sem TL;DR discutido.

Se o usuário corrigir sua leitura, incorpore antes de criar páginas.

### Passo 3 — Criar `wiki/sources/{slug}.md`

Slug: kebab-case sem acento, deriva do título da fonte. Ex: fonte "5W1H Event Semantic Extraction..." → `5w1h-extraction-llm-cao2024`.

Use o template de source (ver `reference/templates.md`). Frontmatter completo:

```yaml
---
title: "Título legível da fonte"
type: source
categoria: pesquisa   # ou outra
tags: [lista, de, tags, relevantes]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [../../raw/sources/NOME-DO-ARQUIVO.md]
status: stable
---
```

### Passo 4 — Derivar entidades e conceitos

Leia a fonte inteira mentalmente e liste:
- **Entidades novas**: pessoas, empresas, ferramentas, produtos que aparecem.
- **Conceitos novos**: ideias, padrões, frameworks, termos técnicos.
- **Páginas existentes** que a fonte atualiza (adiciona fato, confirma/contradiz).

Verifique no `index.md` quais já existem. Para as que existem, planeje **updates**; para as novas, planeje **stubs**.

Típico: uma fonte substancial toca 10-15 páginas (5 novas + 10 atualizadas).

Use o template apropriado pra cada tipo (`entity`, `concept`, `project` ...). Em stubs, tudo bem deixar seções incompletas — marque `status: stub`.

**Paralelize as escritas**. Se você vai criar 10 arquivos novos, faça 10 chamadas Write em paralelo na mesma mensagem — são independentes.

### Passo 5 — Atualizar `index.md`

Adicione todas as novas páginas nas seções corretas (Fontes / Conceitos / Entidades / Projetos / Sínteses / Queries). Agrupe por tema se fizer sentido (ex: se 8 páginas novas são todas do mesmo projeto, crie subseção `### {Projeto}`).

Atualize os contadores em "Visão geral": total de fontes, total de páginas.

### Passo 6 — Append em `log.md`

Entrada no final do arquivo (cronológico crescente):

```markdown
## [YYYY-MM-DD] ingest | {Título resumido da fonte}

- **Fonte**: [[wiki/sources/slug]]
- **Páginas criadas**: [[wiki/x/y]], [[wiki/a/b]], ...
- **Páginas atualizadas**: [[wiki/z/w]] (o que mudou)
- **Contradições detectadas**: nenhuma | descrição
- **Perguntas abertas**: que a fonte levanta mas não responde.
```

### Passo 7 — Reportar ao usuário

Resumo em 3 bullets:
- Quantas páginas criadas / atualizadas.
- Link pra fonte principal ([[wiki/sources/slug]]).
- Perguntas abertas mais interessantes.

### Edge cases

- **Fonte contradiz página existente**: NÃO sobrescreva. Adicione seção "Contradições ou debates" na página afetada, cite ambas as fontes, marque no log.
- **Fonte é híbrida (source + synthesis)**: crie 2 páginas. Source em `wiki/sources/`, synthesis em `wiki/synthesis/`. Source cita a synthesis e vice-versa.
- **Sessão Cowork / Claude Code**: não ingira transcrito cru. Peça pro agente da sessão gerar digest primeiro (ver workflow 4).

## 2. Query — responder pergunta usando o wiki

**Quando usar**: usuário faz pergunta ("o que o wiki diz sobre X?", "quais páginas cobrem Y?").

### Passo 1 — Leia `index.md`

Sempre. **Não faça busca por similaridade embedding** se o index cobre. O index é a interface principal de navegação.

### Passo 2 — Identifique páginas relevantes

Liste os candidatos (3-10). Se a pergunta é ampla, comece pelos mais específicos.

### Passo 3 — Leia as páginas

Em paralelo quando possível. Um Read por página.

### Passo 4 — Sintetize

Responda citando wikilinks pra toda página usada. Forma: "Segundo [[wiki/concepts/rag]], ..., e [[wiki/synthesis/llm-wiki-vs-rag]] adiciona que ..."

Se faltar info, seja honesto: "O wiki não cobre X ainda — posso fazer web search e ingerir nova fonte se quiser."

### Passo 5 — Ofereça arquivar

Se a resposta tem valor persistente (não é trivial, combinou múltiplas páginas, traz insight novo), ofereça:

> "Quer que eu arquive isso como `wiki/queries/YYYY-MM-DD-slug.md`?"

Se sim, crie a query com o template. Frontmatter:

```yaml
---
title: "Pergunta original"
type: query
categoria: pesquisa   # ou outra
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [../sources/a, ../sources/b]
status: draft
---
```

### Passo 6 — Log

```markdown
## [YYYY-MM-DD] query | {pergunta resumida}

- **Páginas consultadas**: [[a]], [[b]]
- **Arquivado em**: [[queries/YYYY-MM-DD-slug]] (ou: não arquivado)
```

## 3. Lint — health check

**Quando usar**: usuário pede "lint", "revisa o wiki", "health check", ou periodicamente (mensalmente).

### Checklist

Para cada item, rode o check e colete issues:

1. **Contradições entre páginas**
   - Duas fontes com números diferentes pro mesmo fato.
   - Duas páginas com definições incompatíveis do mesmo conceito.
   - Estratégia: grep temas comuns, compare.

2. **Claims stale**
   - Fontes antigas com "status quo" superado por fontes novas.
   - Heurística: data da fonte > 2 anos E há fonte mais nova sobre o mesmo tema.

3. **Órfãs** (páginas sem inbound links)
   - Ferramenta: `grep -r "\[\[{slug}" wiki/ | wc -l` pra cada página.
   - Se zero, a página não é descobrível — precisa de link de algum hub (project, entity principal, index).

4. **Conceitos mencionados sem página própria**
   - Se uma página menciona "XYZ" mas não tem `[[xyz]]` e não existe `wiki/concepts/xyz.md`, candidato a criar stub.
   - Ferramenta: extraia termos técnicos recorrentes, compare com lista de páginas.

5. **Cross-references faltando**
   - Página A menciona "B" em texto puro, B tem página própria, mas A não tem `[[B]]`.

6. **Gaps preenchíveis por web search**
   - Fatos marcados com `<!-- confirmar -->` ou `???`.
   - Sugira ao usuário web searches específicos.

7. **Frontmatter inconsistente**
   - `type` faltando, `categoria` fora do vocabulário, `updated` desatualizada.
   - Tags duplicadas/inconsistentes (ex: `dynamodb` vs `DynamoDB`).

8. **`status` calibrado corretamente**
   - Páginas `stable` que na verdade têm 2 linhas → rebaixar pra `stub` ou `draft`.
   - Páginas `stub` maduras há meses → promover.

### Output

Relatório em markdown:

```markdown
# Lint report — YYYY-MM-DD

## Contradições (N)
- ...

## Claims stale (N)
- ...

## Órfãs (N)
- [[slug]] — sugestão de link de hub

## Conceitos sem página (N)
- "XYZ" aparece em [[a]], [[b]]. Candidato a `wiki/concepts/xyz.md`.

## Cross-refs faltando (N)
- [[a]] menciona "B" em texto mas sem link.

## Gaps preenchíveis (N)
- [[a]] tem "<!-- confirmar data -->". Web search: "..."

## Frontmatter issues (N)
- [[a]]: `categoria` ausente.

## Sugestões de novas páginas/fontes
- ...
```

### Aplicar correções

**Espere aprovação do usuário.** Depois aplique correções item por item, mostrando diff pra mudanças substanciais.

### Log

```markdown
## [YYYY-MM-DD] lint

- N contradições achadas, M corrigidas
- N órfãs, ações: ...
- N gaps, ações: ...
- Sugestões de novas páginas/fontes: ...
```

## 4. Session-dump — curar digest de sessão

**Quando usar**: conversa atual foi trabalho real (código, debug, decisões) e vale preservar. **Invocado no projeto, não no wiki.**

### Passo 1 — Validar ambiente

```bash
echo "$LLM_WIKI"
```

Se vazio: avise o usuário que precisa exportar a var no shell (ex: `export LLM_WIKI="$HOME/projetos/llm-wiki"` em `~/.zshrc`) e **pare**.

### Passo 2 — Criar esqueleto via script

```bash
"$HOME/.claude/skills/llm-wiki/scripts/wiki-dump" $ARGUMENTS --no-edit
```

`$ARGUMENTS` vem do slash command (pode ser vazio). `--no-edit` é obrigatório — você (agente) vai preencher, não abrir editor.

O script imprime:
- Em stderr: `# auto-detectado: projeto=X slug=Y` quando deriva defaults.
- Em stdout: path absoluto do digest na última linha. **Capture esse path.**

### Passo 3 — Preencher o digest

Edite o arquivo criado com:

- **`event_5w1h.what`** — 1 frase com o objetivo real. NÃO "trabalhamos em X" — seja concreto: "refatoramos o parser de NF-e pra suportar XML 4.0".
- **`event_5w1h.why`** — ticket, issue, motivo. Se não souber, **pergunte ao usuário**, não invente. Se ele não souber/não quiser dizer, deixe `<!-- confirmar com Gustavo -->`.
- **`event_5w1h.how`** — abordagem (ex: "TDD via pytest, pair-dev com Claude Code, refactor guiado por testes").
- **`event_5w1h.where`** — a ferramenta (Cowork, Claude Code, Cursor).
- **`event_5w1h.who`** — `["Gustavo", "Agente LLM"]` tipicamente.
- **`event_5w1h.when`** — data + duração estimada.

Seções:
- **Contexto inicial** — estado do repo/problema no começo.
- **Decisões tomadas** — cada uma com alternativas descartadas e trade-off. Decisão sem alternativa é ruído.
- **Arquivos modificados** — use `git diff --name-only` se disponível.
- **Problemas resolvidos** — sintoma → root cause → fix. Não pule root cause.
- **Padrões descobertos** — coisas que viraram "ah, isso aqui é reusável" — candidatos a virar concept/snippet no wiki.
- **Follow-ups** — checklist acionável (`- [ ]`).
- **Trechos de transcrito relevantes** — só se agregar contexto que as seções estruturadas não cobrem. Omita por default.

### Passo 4 — Reportar

- Path absoluto do digest.
- 3-bullet resumo.
- Sugestão: "na próxima sessão aberta em `$LLM_WIKI`, peça ingestão desta fonte".

### Regras

- **Não invente.** Campo incerto → comentário HTML `<!-- ... -->`.
- **Não inclua transcrito bruto por default.** O digest é curadoria.
- **Se slug for genérico** (ex: `sess-1639`), sugira um melhor e ofereça renomear.

## 5. Import — arquivar markdown externo

**Quando usar**: usuário tem `.md` pronto (ata, post-mortem, memo) e quer arquivar como fonte bruta, sem reprocessar. **Invocado na pasta onde o arquivo está** (geralmente projeto, não wiki).

### Passo 1 — Validar ambiente

`echo "$LLM_WIKI"`. Se vazio, pare.

### Passo 2 — Rodar script

```bash
"$HOME/.claude/skills/llm-wiki/scripts/wiki-import" <arquivo> [<projeto>] [<slug>] [--categoria <cat>] [--move] [--no-date-prefix]
```

Defaults:
- projeto: `basename $PWD`.
- slug: nome do arquivo sanitizado.
- categoria: `projeto-bhub`.
- Operação: copy (use `--move` pra mover).
- Prefixo de data: `YYYY-MM-DD-` (use `--no-date-prefix` pra omitir).

O script:
- Se o arquivo já tem frontmatter YAML: preserva; reporta campos obrigatórios faltando.
- Se não tem: injeta template mínimo.
- Imprime path final.

### Passo 3 — Oferecer completar frontmatter

Se o script reportou campos faltando, ofereça:

> "O frontmatter tá faltando `X`, `Y`. Quer que eu preencha com base no conteúdo? [rascunho: ...]"

### Passo 4 — Explicar próximo passo

> "Arquivo arquivado em `raw/sources/`. A ingestão real (criar `wiki/sources/`, derivar páginas, atualizar index) é o workflow de Ingest — vamos fazer agora ou depois numa sessão no wiki?"

### Diferença vs wiki-dump

- **wiki-dump**: a fonte É a sessão em si (digest do que acabou de ser feito).
- **wiki-import**: a fonte JÁ EXISTE como arquivo (gerado por humano, exportado de outra ferramenta, baixado).

## Resumo de invariantes

Em qualquer workflow:

1. Sempre localizar o wiki e ler `CLAUDE.md` primeiro.
2. Sempre atualizar `index.md` + `log.md` no final de ingest e lint.
3. Nunca sobrescrever edição humana.
4. Nunca modificar `raw/`.
5. Discussão antes de criação em larga escala (TL;DR antes de ingest, relatório antes de aplicar lint).
6. Wikilinks em vez de Markdown links.
7. PT-BR em páginas do wiki.

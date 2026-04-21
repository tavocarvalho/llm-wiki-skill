---
description: Importa um .md pré-existente para raw/sources/ do LLM Wiki. Auto-detecta projeto e slug.
argument-hint: <arquivo> [<projeto>] [<slug>] [--categoria <cat>] [--move] [--no-date-prefix]
---

# /wiki-import — importa um .md existente para o LLM Wiki

Use quando **já tem um documento pronto** (ata de reunião, post-mortem, doc de arquitetura, memo, nota avulsa) e quer arquivá-lo no wiki como fonte bruta.

Diferença para `/wiki-dump`: `wiki-dump` **gera** um digest a partir da sessão atual; `wiki-import` **transfere** um arquivo já existente.

## Argumentos

- **arquivo** (obrigatório) — path do `.md` a importar.
- **projeto** (opcional) — default: `basename $PWD`.
- **slug** (opcional) — default: nome do arquivo sem `.md`, sanitizado.
- **--categoria** — default `projeto`. O vocabulário real vem do `CLAUDE.md` do wiki; se o default não bater, passe explícito (ex: `--categoria pesquisa`).
- **--move** — move em vez de copiar (default é copiar).
- **--no-date-prefix** — não prefixa `YYYY-MM-DD-` (útil se o arquivo já tem data no nome).

## Passos (executar em ordem)

1. **Validar ambiente.** Rode `echo "$LLM_WIKI"` via Bash. Se vazio, avise o usuário e aborte.

2. **Validar arquivo.** Confirme que `$ARGUMENTS` começa com um path existente (`ls <primeiro-arg>`). Se o usuário deu um path relativo, confirme que é relativo ao `$PWD` atual.

3. **Chamar o script:**
   ```
   "$HOME/.claude/skills/llm-wiki/scripts/wiki-import" $ARGUMENTS
   ```
   Capture stdout (path final) e stderr (avisos).

4. **Revisar o resultado:**
   - Se o script avisou que **injetou frontmatter**: abra o arquivo final, leia o topo e sugira ao usuário ajustes de `title`, `tags`, `status`. Se o doc é um evento (reunião, incidente, call), ofereça preencher `event_5w1h`.
   - Se o script avisou que **campos obrigatórios estão faltando**: liste quais e ofereça completar com base no conteúdo do arquivo.
   - Se o script avisou que **todos os campos estão OK**: apenas confirme o path e siga.

5. **Reportar ao usuário:**
   - Path final do arquivo em `raw/sources/`.
   - Se houve ajustes no frontmatter, um resumo do que mudou.
   - Próximo passo: "na próxima sessão aberta na pasta do wiki, peça para ingerir esta fonte e derivar páginas do wiki."

## Regras importantes

- **Nunca modifique o corpo do documento original** — só frontmatter. Se o usuário quiser reescrever o conteúdo, é tarefa separada.
- **Não sobrescreva arquivos existentes em `raw/sources/`.** O script já bloqueia isso (exit 6); se der erro, sugira slug diferente ou pergunte se é duplicata.
- **Se o arquivo tem frontmatter com convenções estranhas** (campos em inglês, vocabulário fora do controlado), mostre ao usuário e pergunte se quer traduzir/mapear antes de ingerir.
- **Default é copiar, não mover.** Só use `--move` se o usuário pedir explicitamente — preservar o original no local onde ele estava é mais seguro.

## Pré-requisito

- `$LLM_WIKI` exportado apontando para a raiz do wiki.
- Skill `llm-wiki` instalada em `~/.claude/skills/llm-wiki/` (o script vive em `scripts/wiki-import`).

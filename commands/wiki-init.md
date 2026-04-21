---
description: Inicializa um LLM Wiki novo em $LLM_WIKI com CLAUDE.md, AGENTS.md, README.md, index.md, log.md e estrutura de diretórios canônica.
argument-hint: [--force]
---

# /wiki-init — inicialização de um LLM Wiki novo

Cria o scaffold completo de um LLM Wiki na pasta apontada por `$LLM_WIKI`: schema (`CLAUDE.md`, `AGENTS.md`), orientação humana (`README.md`), catálogo (`index.md`), log histórico (`log.md`) e estrutura de diretórios (`raw/`, `wiki/`).

**Use uma vez**, na instalação inicial. Depois disso, não precisa rodar de novo — o wiki evolui via ingest/dump/import.

## Argumentos (opcionais)

- **`--force`** — sobrescreve `CLAUDE.md`/`AGENTS.md` existentes. Destrutivo — só use se tem certeza que quer recomeçar do zero.

## Passos (executar em ordem)

1. **Validar ambiente.** Rode `echo "$LLM_WIKI"` via Bash. Se vazio, avise o usuário e **pare** — o script exige a variável exportada. Sugira o comando:
   ```bash
   export LLM_WIKI="$HOME/caminho/do/wiki"
   ```
   E lembre de adicionar no `~/.zshrc` ou `~/.bashrc` pra persistir.

2. **Checar se a pasta já tem conteúdo.** Se `$LLM_WIKI` já tem `CLAUDE.md`, o wiki já está inicializado — **pergunte** antes de passar `--force`. Na maioria dos casos a resposta correta é "não, aborta — já tá inicializado".

3. **Invocar o script.** Via Bash:
   ```
   "$HOME/.claude/skills/llm-wiki/scripts/wiki-init" $ARGUMENTS
   ```
   O script cria diretórios, copia os 5 templates de `reference/bootstrap/` e imprime no stderr um resumo do que foi feito e os próximos passos.

4. **Reportar ao usuário** em 3 pontos:
   - Caminho do wiki inicializado.
   - Lista dos arquivos e diretórios criados.
   - Próximos passos sugeridos (revisar categorias no `CLAUDE.md`, opcional `git init`, primeira ingestão).

5. **Oferecer personalização das categorias.** O template default tem `pesquisa | projeto | engenharia | referencia | pessoal`. Pergunte ao usuário se ele quer ajustar antes de começar a usar — ex: se o trabalho dele é muito focado num projeto específico, pode querer renomear `projeto` pra `projeto-{nome}` ou adicionar uma categoria extra (`regra-de-negocio`, `cliente-X`, etc.). Se sim, edite a seção 3 do `CLAUDE.md` + a regra 7 do `AGENTS.md` pra sincronizar.

## Regras importantes

- **Nunca rode `--force` sem confirmação explícita do usuário.** Sobrescrever `CLAUDE.md` destrói personalizações acumuladas.
- **Se o usuário não tem `$LLM_WIKI` exportado, não tente adivinhar.** Pergunte onde ele quer criar o wiki e sugira o comando de export.
- **Não crie páginas de exemplo** no `wiki/`. O scaffold é vazio de propósito — o wiki cresce com ingest de fontes reais, não com placeholders.

## Pré-requisito de instalação

- A skill `llm-wiki` instalada em `~/.claude/skills/llm-wiki/` (o script vive dentro dela, em `scripts/wiki-init`, e os templates em `reference/bootstrap/`).
- `$LLM_WIKI` exportado (o script falha se vazio):
  ```bash
  export LLM_WIKI="$HOME/caminho/do/wiki"
  ```

## Ordem de instalação recomendada

1. Clonar `llm-wiki-skill` e symlinkar em `~/.claude/skills/llm-wiki/`.
2. Symlinkar os slash commands em `~/.claude/commands/` (incluindo este, `wiki-init.md`).
3. Decidir onde o wiki vai viver e exportar `$LLM_WIKI`.
4. Rodar `/wiki-init`.
5. Revisar e personalizar o `CLAUDE.md` criado.
6. (Opcional) `git init` no wiki pra versionar.
7. Primeira ingestão: `/wiki-import` um documento ou arraste algo pra `raw/sources/` e peça ingestão.

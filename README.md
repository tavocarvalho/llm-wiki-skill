# llm-wiki-skill

Skill do Claude Code / Cowork para manter um **LLM Wiki** — base de conhecimento em markdown interligado (wikilinks estilo Obsidian) com schema rígido. O humano traz fontes e faz perguntas; o agente cria/atualiza páginas, preserva consistência, mantém índices.

Este repositório é **apenas a ferramenta**. O wiki em si (conteúdo: `CLAUDE.md`, `raw/`, `wiki/`, `index.md`, `log.md`) vive num outro repositório apontado pela variável de ambiente `$LLM_WIKI`.

> **Créditos.** A ideia do LLM Wiki é de **Andrej Karpathy**, descrita originalmente no gist [karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — "uma pattern, não um produto": usar o LLM como compilador que lê fontes brutas e mantém uma base de conhecimento markdown interligada, em alternativa a RAG/embeddings na escala de uma base pessoal. Este repo é uma **implementação concreta** do pattern, opinativa em PT-BR, com schema fixo (`type`, `categoria`, `event_5w1h`), scripts pra session-digest e import, e um slash command pra bootstrap. Ver também [§Referências](#referências).

## Arquitetura

Dois repos separados por responsabilidade:

| Repo                          | Papel                  | Conteúdo                                           |
|-------------------------------|------------------------|----------------------------------------------------|
| `llm-wiki` (seu, privado)     | **conteúdo**           | `CLAUDE.md`, `raw/`, `wiki/`, `index.md`, `log.md` |
| `llm-wiki-skill` (este repo)  | **ferramenta**         | `SKILL.md`, `commands/`, `scripts/`, `reference/`  |

**Novo wiki?** Use `/wiki-init` (ou `scripts/wiki-init`) pra criar o scaffold canônico do `llm-wiki` a partir dos templates genéricos em `reference/bootstrap/`.

A skill nunca assume que os scripts estão em `$LLM_WIKI/scripts/` — ela sempre invoca via path absoluto `$HOME/.claude/skills/llm-wiki/scripts/<script>`. A única informação que a skill precisa do wiki em runtime é o valor de `$LLM_WIKI`.

## Estrutura

```
llm-wiki-skill/
├── SKILL.md               # descrição + triggers + workflows resumidos
├── README.md              # este arquivo
├── reference/
│   ├── schema.md          # frontmatter completo, vocabulários, estruturas por tipo
│   ├── workflows.md       # passo-a-passo dos 5 workflows
│   ├── templates.md       # templates copy-pastable
│   ├── troubleshooting.md # problemas comuns
│   └── bootstrap/         # templates canônicos pra inicializar wiki novo
│       ├── CLAUDE.md      # schema genérico (categorias genéricas, PT-BR)
│       ├── AGENTS.md      # espelho do CLAUDE.md
│       ├── README.md      # orientação humana
│       ├── index.md       # catálogo vazio
│       └── log.md         # log vazio
├── scripts/
│   ├── wiki-init          # inicializa scaffold de wiki novo
│   ├── wiki-dump          # gera esqueleto de session digest
│   └── wiki-import        # importa .md externo pra raw/sources/
├── commands/
│   ├── wiki-init.md       # slash command /wiki-init
│   ├── wiki-dump.md       # slash command /wiki-dump
│   ├── wiki-import.md     # slash command /wiki-import
│   └── wiki-ingest-session.md   # slash command /wiki-ingest-session
└── evals/
    └── evals.json         # casos de teste
```

## Instalação no host (Mac/Linux)

Três passos. Assumindo que você vai clonar este repo em `~/projetos/llm-wiki-skill/` e seu wiki vive em `~/projetos/llm-wiki/` (ajuste os paths se preferir outro layout):

### 1. Clonar este repo

```bash
git clone https://github.com/tavocarvalho/llm-wiki-skill.git ~/projetos/llm-wiki-skill
```

### 2. Exportar `$LLM_WIKI` no shell

Em `~/.zshrc` (ou `~/.bashrc`):

```bash
export LLM_WIKI="$HOME/projetos/llm-wiki"   # ajuste pro path do seu wiki
```

Re-source (`source ~/.zshrc`) e valide (`echo "$LLM_WIKI"`).

### 3. Symlinkar a skill e os slash commands

```bash
# Skill
mkdir -p ~/.claude/skills
ln -sfn "$HOME/projetos/llm-wiki-skill" ~/.claude/skills/llm-wiki

# Slash commands (um por arquivo)
mkdir -p ~/.claude/commands
ln -sf "$HOME/.claude/skills/llm-wiki/commands/wiki-init.md"           ~/.claude/commands/wiki-init.md
ln -sf "$HOME/.claude/skills/llm-wiki/commands/wiki-dump.md"           ~/.claude/commands/wiki-dump.md
ln -sf "$HOME/.claude/skills/llm-wiki/commands/wiki-import.md"         ~/.claude/commands/wiki-import.md
ln -sf "$HOME/.claude/skills/llm-wiki/commands/wiki-ingest-session.md" ~/.claude/commands/wiki-ingest-session.md
```

Verifique:

```bash
ls -la ~/.claude/skills/llm-wiki ~/.claude/commands/wiki-*
```

Deve mostrar os 5 symlinks apontando pro repo. Reinicie o Claude Code (feche e reabra a sessão — commands são carregados no startup). Digite `/wiki-` e o autocomplete deve listar os quatro.

### 4. Inicializar o wiki (só na primeira vez)

Se `$LLM_WIKI` aponta pra uma pasta vazia (ou inexistente), rode:

```bash
/wiki-init
```

Cria `CLAUDE.md`, `AGENTS.md`, `README.md`, `index.md`, `log.md` + estrutura de diretórios (`raw/sources/`, `raw/assets/sessions/`, `wiki/{entities,concepts,sources,projects,synthesis,queries}/`) a partir dos templates em `reference/bootstrap/`. Depois disso, revise o `CLAUDE.md` e ajuste as categorias se quiser.

Se `$LLM_WIKI` já tem um wiki populado, **pule este passo** — `/wiki-init` se recusa a sobrescrever sem `--force`.

### (opcional) Rodar scripts direto do shell

Se quiser invocar `wiki-dump` / `wiki-import` fora do Claude Code, adicione ao `$PATH`:

```bash
export PATH="$HOME/.claude/skills/llm-wiki/scripts:$PATH"
```

## Uso

A skill ativa automaticamente quando você menciona algo do domínio wiki — não precisa invocar por nome. Exemplos:

| Você diz                                                              | Operação        |
|-----------------------------------------------------------------------|-----------------|
| "bota esse paper no wiki"                                             | Ingest          |
| "o que o wiki já tem sobre X?"                                        | Query           |
| "faz um lint, tô achando que tem duplicata"                           | Lint            |
| "salva essa sessão" / `/wiki-dump`                                    | Session-dump    |
| "importa esse post-mortem" / `/wiki-import ./doc.md`                  | Import          |

Detalhes completos: leia `SKILL.md` e `reference/workflows.md`.

## Sincronia com o schema

A **fonte de verdade** do schema é o `CLAUDE.md` do repo `llm-wiki` (conteúdo) — a skill lê esse arquivo no começo de toda operação via `$LLM_WIKI/CLAUDE.md`. Se o schema evoluir (novo campo obrigatório, categoria nova), edite o `CLAUDE.md` do wiki primeiro; a skill vai pegar a mudança automaticamente.

Se mudar algo que afeta os scripts (ex: template de digest, validação de `--categoria`), lembre de atualizar neste repo:

1. `scripts/wiki-dump` (template `cat > "$OUT" <<EOF` e validação de `$CATEGORIA`).
2. `scripts/wiki-import` (validação de `$CATEGORIA`, frontmatter template injetado).
3. `commands/*.md` (as instruções que o agente segue).
4. `reference/templates.md`.

## Troubleshooting

Ver `reference/troubleshooting.md`. Sintomas mais comuns:

- **`LLM_WIKI nao existe`** — `$LLM_WIKI` unset ou aponta pra pasta errada. `echo "$LLM_WIKI"`.
- **Slash command não aparece no Claude Code** — confirme symlinks em `~/.claude/commands/`. Reinicie a sessão.
- **Skill não ativa** — confirme symlink em `~/.claude/skills/llm-wiki`. Descrição em `SKILL.md` precisa bater com a intenção do usuário.

## Distribuição futura (time)

Hoje é setup individual (clone + symlink). Pra distribuir pro time, o próximo passo é empacotar como **Claude Code plugin** — um bundle com skill + commands + settings que qualquer pessoa instala via marketplace. Deferido até a skill estabilizar em uso real.

## Referências

- **Andrej Karpathy, "llm-wiki" gist** — [gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Descrição original do pattern: base de conhecimento markdown mantida por LLM, alternativa a RAG pra escala pessoal. Arquitetura de 3 camadas (raw / wiki / schema), operações `ingest` / `query` / `lint`. Esta skill implementa essas ideias com opiniões concretas (PT-BR, schema fixo, scripts de session-digest/import/bootstrap).
- **MindStudio — "What Is Andrej Karpathy's LLM Wiki?"** — [mindstudio.ai/blog/andrej-karpathy-llm-wiki-knowledge-base-claude-code](https://www.mindstudio.ai/blog/andrej-karpathy-llm-wiki-knowledge-base-claude-code). Walkthrough explicativo do pattern.
- **VentureBeat — "Karpathy shares 'LLM Knowledge Base' architecture that bypasses RAG"** — [venturebeat.com/data/karpathy-shares-llm-knowledge-base-architecture-that-bypasses-rag-with-an](https://venturebeat.com/data/karpathy-shares-llm-knowledge-base-architecture-that-bypasses-rag-with-an). Cobertura da ideia e contraste explícito com RAG.
- **DAIR.AI Academy — "LLM Knowledge Bases"** — [academy.dair.ai/blog/llm-knowledge-bases-karpathy](https://academy.dair.ai/blog/llm-knowledge-bases-karpathy). Análise pedagógica do conceito.

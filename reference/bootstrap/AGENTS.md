# AGENTS.md — Schema do LLM Wiki (versão genérica para agentes)

> Equivalente do `CLAUDE.md` para agentes que seguem a convenção `AGENTS.md` (OpenAI Codex, OpenCode, Pi, etc.). As regras são idênticas — este arquivo existe para compatibilidade cruzada. Se editar convenções, atualize **ambos**.

---

## Resumo rápido (para quem abre este arquivo primeiro)

Este repositório é um **LLM Wiki** em Português (BR): um sistema onde o agente LLM mantém uma coleção persistente e interligada de páginas markdown, alimentada por fontes brutas em `raw/`. O agente é o maintainer; o humano é o curador.

**Para instruções completas, leia `CLAUDE.md`.** Este arquivo espelha o conteúdo principal para garantir que qualquer agente consiga operar o wiki.

---

## Estrutura

```
llm-wiki/                  # $LLM_WIKI aponta só para esta pasta (conteúdo)
├── CLAUDE.md / AGENTS.md  # schema (este arquivo e o CLAUDE.md)
├── README.md              # orientação humana
├── index.md               # catálogo (content-oriented)
├── log.md                 # histórico (chronological, append-only)
├── raw/                   # fontes imutáveis
│   ├── sources/
│   └── assets/
└── wiki/                  # páginas geradas pelo LLM
    ├── entities/
    ├── concepts/
    ├── sources/
    ├── projects/
    ├── synthesis/
    └── queries/

# Ferramenta vive em repo git separado (clonado pelo usuário, symlinkado):
~/.claude/skills/llm-wiki/
├── SKILL.md               # descrição + triggers
├── reference/             # schema, workflows, templates, troubleshooting, bootstrap
├── scripts/               # wiki-init, wiki-dump, wiki-import (bash)
├── commands/              # /wiki-init, /wiki-dump, /wiki-import, /wiki-ingest-session
└── evals/                 # casos de teste
```

## Regras fundamentais

1. **`raw/` é imutável.** O agente lê, nunca modifica.
2. **`wiki/` é do agente.** O agente escreve, mantém cross-references, atualiza em cada ingest.
3. **Idioma do wiki: PT-BR.** Fontes podem estar em qualquer idioma.
4. **Uma página = um conceito/entidade.** Muitas páginas pequenas, interligadas com `[[wikilinks]]`.
5. **Frontmatter YAML** obrigatório em toda página com: `title`, `type`, `categoria`, `tags`, `created`, `updated`, `status`, `sources` (exceto quando `type: source`). Campos opcionais: `domain` (só em `synthesis`), `event_5w1h` (só em `source` tipo evento), `repo` (URL HTTPS do repo git associado — auto-preenchido pelos scripts quando o PWD é git repo; use em sources de sessão e páginas de project).
6. **Nomes de arquivo** em `kebab-case-sem-acentos.md`.
7. **Vocabulário controlado de `categoria`:** `pesquisa | projeto | engenharia | referencia | pessoal`. Se precisar de outro, edite o schema antes de usar.
8. **Cada página tem exatamente um `type`.** Não misturar.

## Workflows (resumo)

### Ingest
1. Ler fonte inteira em `raw/sources/`.
2. Discutir TL;DR com o humano.
3. Criar `wiki/sources/{slug}.md`.
4. Atualizar/criar páginas de entidades e conceitos tocados (tipicamente 10-15).
5. Atualizar `index.md`.
6. Append em `log.md` com formato `## [YYYY-MM-DD] ingest | {título}`.

### Query
1. Ler `index.md` para localizar páginas.
2. Ler páginas relevantes.
3. Responder com wikilinks para todas as citações.
4. Oferecer arquivar como `wiki/queries/YYYY-MM-DD-slug.md`.
5. Append em `log.md`.

### Lint
Verificar: contradições, claims stale, órfãs, conceitos sem página, cross-refs faltando, gaps preenchíveis por web search, frontmatter inconsistente. Relatar, aguardar aprovação, aplicar.

### Ingest de sessão de LLM
**Transcrito cru não vai em `raw/sources/`.** Fontes de sessão são **digests curados** gerados pelo agente ao final da sessão. Template do digest inclui frontmatter com `event_5w1h` completo + seções Contexto, Decisões, Arquivos modificados, Problemas resolvidos, Padrões descobertos, Follow-ups. Transcritos brutos opcionais vão em `raw/assets/sessions/`.

**Kit operacional:**
- `scripts/wiki-init` — inicializa scaffold de wiki novo (só usa uma vez, na instalação).
- `scripts/wiki-dump` — cria esqueleto de session digest.
- `scripts/wiki-import` — importa `.md` pré-existente para `raw/sources/`.
- Slash commands `/wiki-*` equivalentes.

**Detalhes completos no `CLAUDE.md`.**

## Log format

Todas as entradas usam o prefixo:
```
## [YYYY-MM-DD] {tipo} | {resumo}
```
Tipos: `ingest`, `query`, `lint`, `note`, `refactor`.

Permite grep: `grep "^## \[" log.md | tail -20`.

## Anti-padrões

- Não modificar `raw/`.
- Não repetir informação — **link em vez de copiar**.
- Não ingerir sem discutir TL;DR.
- Não esquecer de atualizar `index.md` e `log.md`.
- Não usar inglês no corpo das páginas (exceto termos técnicos sem tradução consagrada).
- Não misturar tipos em uma única página. Cada página pertence a exatamente um `type`.
- Não despejar transcrito bruto de sessão em `raw/sources/`. Ingest de sessão é **digest curado** (ver `CLAUDE.md` 4.5).
- Não omitir `categoria:` no frontmatter.

---

**Fonte canônica:** veja `CLAUDE.md` para detalhes completos, templates por tipo de página e tips operacionais.

**Última atualização:** (data de criação) — scaffold inicial via `/wiki-init`.

# Schema completo — LLM Wiki

Referência operacional do schema. Toda página de wiki segue este contrato. A fonte de verdade canônica é `$WIKI_ROOT/CLAUDE.md` — este arquivo é um extrato focado em schema.

## Arquitetura em 3 camadas

```
llm-wiki/
├── CLAUDE.md            # constituição (schema, workflows)
├── AGENTS.md            # equivalente para agentes genéricos
├── README.md            # orientação humana
├── index.md             # catálogo navegável (content-oriented)
├── log.md               # histórico append-only (chronological)
│
├── raw/                 # CAMADA 1 — Fontes brutas (imutáveis)
│   ├── sources/         # artigos, papers, transcripts, docs
│   └── assets/          # imagens, PDFs, anexos, transcritos de sessão
│
├── wiki/                # CAMADA 2 — Páginas geradas pelo LLM
│   ├── entities/        # pessoas, empresas, ferramentas, produtos
│   ├── concepts/        # ideias, termos, padrões, frameworks
│   ├── sources/         # uma página por fonte ingerida
│   ├── projects/        # páginas de projeto (bhub ou pessoais)
│   ├── synthesis/       # análises transversais, comparações, teses
│   └── queries/         # respostas arquivadas a perguntas
│
├── scripts/             # CAMADA 3 — Tooling
│   ├── wiki-dump        # gera digest de sessão
│   ├── wiki-import      # importa md externo
│   └── claude-commands/ # slash commands equivalentes
│
└── skills/              # skills Claude Code locais
    └── llm-wiki/        # esta skill
```

Regras duras de camadas:
- `raw/` é **imutável**. LLM lê, nunca escreve. Se precisa corrigir fonte, cria nova versão.
- `wiki/` é **owned pelo LLM**. LLM cria, atualiza, mantém consistência.
- Arquivos de raiz (`index.md`, `log.md`, `CLAUDE.md`, `AGENTS.md`) são editados pelo LLM também.

## Frontmatter YAML

### Campos obrigatórios (toda página)

```yaml
---
title: "Nome legível da página"
type: entity | concept | source | project | synthesis | query
categoria: pesquisa | projeto-bhub | engenharia | regra-de-negocio | pessoal
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: stub | draft | stable
---
```

### Campos semi-obrigatórios

- **`sources`**: lista de wikilinks pra páginas de `wiki/sources/` que alimentam esta página. Obrigatório em toda página **exceto** `type: source` (que cita a si mesma via `../../raw/`).

### Campos opcionais

- **`domain`**: `bhub | pessoal | pesquisa`. Só em synthesis, pra marcar se a conclusão é domain-specific.
- **`event_5w1h`**: só em sources do tipo evento (sessão, reunião, incidente, post-mortem).
  ```yaml
  event_5w1h:
    what: "o que aconteceu"
    when: "YYYY-MM-DD (duração ~Nh)"
    where: "Cowork | Claude Code | Cursor | sala | sistema"
    who: [lista, de, participantes]
    why: "motivo / ticket originador"
    how: "abordagem / como foi conduzido"
  ```

### Vocabulário controlado

- **`type`** (fixo):
  - `entity` — pessoas, empresas, ferramentas, produtos, sistemas.
  - `concept` — ideias, termos, padrões, frameworks, abstrações.
  - `source` — uma por fonte ingerida (paper, artigo, spec, digest).
  - `project` — projeto em andamento (BHub ou pessoal).
  - `synthesis` — comparações, teses, análises transversais combinando várias fontes.
  - `query` — resposta arquivada a uma pergunta. Nome com data: `YYYY-MM-DD-slug.md`.

- **`categoria`** (fixo):
  - `pesquisa` — papers, conceitos acadêmicos, teoria.
  - `projeto-bhub` — tudo relacionado a produto/projeto BHub.
  - `engenharia` — patterns técnicos, ferramentas, stacks.
  - `regra-de-negocio` — conhecimento interno de domínio/processos.
  - `pessoal` — notas, modelos mentais, conclusões do Gustavo.

- **`status`** (fixo):
  - `stub` — esboço mínimo, esperando mais fontes.
  - `draft` — conteúdo razoável mas incompleto/em evolução.
  - `stable` — página madura, confiável.

Para adicionar valores ao vocabulário, edite o `CLAUDE.md` primeiro, depois use.

## Estrutura recomendada por tipo

### Entity (`wiki/entities/`)

```markdown
# {Nome}

## TL;DR
(1-2 frases do que é a entidade)

## Contexto
(origem, quando apareceu, onde encaixa)

## Relações
- [[../concepts/X]] — tipo de relação
- [[../entities/Y]] — tipo de relação

## Fatos datados
- **YYYY-MM-DD**: fato concreto ([[sources/fonte|citação]])

## Fontes
- [[../sources/nome-da-fonte]]
```

### Concept (`wiki/concepts/`)

```markdown
# {Conceito}

## Definição
(frase curta e precisa)

## Por que importa
(contexto de uso, valor)

## Mecânica / como funciona
(como o conceito opera, passo a passo quando aplicável)

## Exemplos
(exemplos concretos com wikilinks pra casos reais)

## Contraste com ideias próximas
- Vs [[../concepts/X]]: diferença chave.

## Contradições ou debates (se houver)
(posições conflitantes na literatura, escolhas abertas)

## Fontes
- [[../sources/...]]
```

### Source (`wiki/sources/`)

Uma por fonte ingerida.

```markdown
# {Título da fonte}

## Metadados
- **Autor(es)**: ...
- **Data**: ...
- **Link**: ...
- **Raw**: [[../../raw/sources/NOME]]

## TL;DR
- Bullet 1
- Bullet 2
- Bullet 3

## Takeaways principais
(detalhamento do TL;DR com citações)

## Citações notáveis
> "citação textual"

## Páginas do wiki afetadas por esta ingestão
- Criadas: [[...]], [[...]]
- Atualizadas: [[...]]

## Perguntas abertas que a fonte levanta
- ...
```

No frontmatter, o campo `sources` aponta pra raw:
```yaml
sources: [../../raw/sources/2026-04-20-arquivo.md]
```

### Project (`wiki/projects/`)

```markdown
# {Projeto}

## Objetivo
(uma frase)

## Status atual (YYYY-MM-DD)
(onde está, em que fase)

## Decisões
- [[../synthesis/adr-X|ADR-001 — X]]: resumo.

## Componentes
- [[../entities/Y]] — papel no projeto.

## Riscos
- Zona vermelha: ...
- Zona amarela: ...

## Próximos passos
- [ ] Item acionável

## Fontes
- [[../sources/...]]
```

### Synthesis (`wiki/synthesis/`)

Análises transversais combinando várias fontes. Formato livre, mas deve incluir:
- Pergunta ou tese central.
- Comparação ponto-a-ponto quando aplicável.
- Conclusão ou tradeoff explícito.
- Todas as fontes usadas listadas em `sources:` do frontmatter.

### Query (`wiki/queries/`)

Nome: `YYYY-MM-DD-pergunta-resumida.md`.

```markdown
# {Pergunta original}

## Contexto
(por que a pergunta foi feita, o que motivou)

## Resposta
(síntese com wikilinks pra toda página usada)

## Páginas consultadas
- [[../entities/X]]
- [[../concepts/Y]]

## Follow-ups
- Perguntas derivadas, novas páginas sugeridas.
```

## Nomenclatura

- **Arquivos**: `kebab-case-sem-acento.md`. Ex: `llm-wiki.md`, `vannevar-bush.md`, `nota-fiscal.md` (não `nota-fiscal.md`).
- **Uma página = um conceito/entidade.** Prefira muitas páginas pequenas a poucas gigantes.
- **Título no frontmatter pode ter acento** e maiúsculas. Só o nome de arquivo é normalizado.

## Wikilinks

- Mesmo diretório: `[[slug]]`.
- Diretório diferente: `[[../subdir/slug]]`.
- Texto customizado: `[[slug|texto customizado]]`.
- **Nunca use** caminhos Markdown relativos `./foo.md` — só wikilinks. Obsidian e qmd dependem disso.

## `index.md`

Catálogo content-oriented. Organizado por categoria/tipo. Atualizado em todo ingest.

Cabeçalho:
```markdown
# Index — Catálogo do LLM Wiki

> Catálogo content-oriented. Cada página listada com link e one-liner.
> **Ver também:** [`log.md`](log.md), [`CLAUDE.md`](CLAUDE.md).

## Visão geral
- **Total de fontes ingeridas:** N
- **Total de páginas no wiki:** N
- **Última atualização:** YYYY-MM-DD
```

Entrada típica:
```markdown
- [[wiki/concepts/rag]] — Retrieval-Augmented Generation; contraste com LLM Wiki. `(draft)`
```

Grupos são subseções `###` dentro das seções principais (Fontes / Conceitos / Entidades / Projetos / Sínteses / Queries).

## `log.md`

Append-only, **cronológico crescente** (entradas antigas no topo, novas no fim — permite `tail` para últimas).

Prefixo padrão:
```
## [YYYY-MM-DD] {tipo} | {resumo}
```

Tipos: `ingest | query | lint | note | refactor`.

Entrada de ingest típica:
```markdown
## [2026-04-20] ingest | Paper Cao 2024 — 5W1H extraction via LLMs

- **Fonte**: [[wiki/sources/5w1h-extraction-llm-cao2024]]
- **Páginas criadas**: [[wiki/concepts/5w1h-extraction]], [[wiki/entities/giveme5w1h]], ...
- **Páginas atualizadas**: [[wiki/concepts/llm-wiki]] (adicionado insight Y)
- **Contradições detectadas**: nenhuma
- **Perguntas abertas**: ...
```

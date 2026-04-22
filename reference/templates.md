# Templates prontos

Copie, adapte, não improvise. Os templates abaixo respeitam o schema completo.

## Entity

```markdown
---
title: "{Nome}"
type: entity
categoria: {pesquisa | projeto | engenharia | referencia | pessoal}
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - ../sources/slug-da-fonte.md
status: stub
---

# {Nome}

## TL;DR

(1-2 frases do que é a entidade, o essencial pra alguém ler em 5 segundos.)

## Contexto

(Origem, quando apareceu, onde encaixa no ecossistema/projeto.)

## Relações

- [[../concepts/X]] — tipo de relação.
- [[../entities/Y]] — tipo de relação.

## Fatos datados

- **YYYY-MM-DD** — fato concreto ([[../sources/fonte|citação]]).

## Fontes

- [[../sources/nome-da-fonte]].
```

## Concept

```markdown
---
title: "{Conceito}"
type: concept
categoria: {...}
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - ../sources/slug-da-fonte.md
status: stub
---

# {Conceito}

## Definição

(Uma frase precisa.)

## Por que importa

(Contexto de uso, valor prático.)

## Mecânica / como funciona

(Como opera. Passo a passo quando aplicável.)

## Exemplos

- Caso concreto: [[../projects/X]] usa assim...

## Contraste com ideias próximas

- **Vs [[../concepts/outro]]**: diferença chave.

## Contradições ou debates

(Posições conflitantes na literatura, escolhas abertas. Opcional.)

## Fontes

- [[../sources/...]]
```

## Source (genérico)

```markdown
---
title: "{Título da fonte}"
type: source
categoria: {...}
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - ../../raw/sources/NOME-DO-ARQUIVO.md
status: stable
---

# {Título da fonte}

## Metadados

- **Autor(es)**: ...
- **Data**: ...
- **Link original**: ...
- **Onde está em raw**: [[../../raw/sources/NOME]]

## TL;DR

- Bullet 1.
- Bullet 2.
- Bullet 3.

## Takeaways principais

(Detalhamento do TL;DR com citações textuais quando relevante.)

## Citações notáveis

> "Citação textual da fonte."

## Páginas do wiki afetadas por esta ingestão

- **Criadas**: [[../concepts/x]], [[../entities/y]].
- **Atualizadas**: [[../concepts/z]].

## Perguntas abertas que a fonte levanta

- Pergunta 1.
- Pergunta 2.
```

## Source (sessão / evento) — com `event_5w1h`

```markdown
---
title: "Sessão {projeto} — {feature ou problema}"
type: source
categoria: projeto
repo: https://github.com/org/repo   # opcional; auto-preenchido pelo wiki-dump se o PWD é um git repo
tags: [sessao, {projeto}]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - ../../raw/sources/YYYY-MM-DD-PROJETO-SLUG.md
status: stable
event_5w1h:
  what: "Objetivo concreto da sessão em uma frase."
  when: "YYYY-MM-DD (duração ~Nh)"
  where: "Cowork | Claude Code | Cursor | outro"
  who: ["<usuário>", "Agente LLM"]
  why: "Ticket/issue/motivo originador."
  how: "Abordagem (TDD, pair-dev, refactor guiado por testes)."
---

# Sessão {projeto} — {feature ou problema}

## Contexto inicial

(Estado do repo/problema no começo da sessão.)

## Decisões tomadas

### Decisão 1: {título}

- **Escolha**: ...
- **Alternativas descartadas**: ...
- **Trade-off**: ...

## Regras de negócio tocadas

<!-- Regra + antes/depois + link pra [[wiki/projects/X]] que deve ser atualizado no mesmo
     ingest. Deixar vazio se a sessão não alterou/esclareceu regra de negócio. -->

### Regra 1: {descrição curta}

- **Antes**: ...
- **Depois**: ...
- **Project page a atualizar**: [[wiki/projects/X]]

## Arquivos modificados

- `path/to/file.py` — o que mudou.

## Problemas resolvidos

### Problema 1: {sintoma}

- **Root cause**: ...
- **Fix**: ...

## Padrões descobertos

(Candidatos a virar concept/snippet no wiki.)

## Follow-ups

- [ ] Item acionável.

## Trechos de transcrito relevantes (opcional)

(Apenas se agregar contexto que seções acima não cobrem.)
```

## Project

```markdown
---
title: "{Projeto}"
type: project
categoria: {projeto | pessoal}
repo: https://github.com/org/repo   # opcional; URL canônica do projeto se versionado em git
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - ../sources/...
status: draft
---

# {Projeto}

## Objetivo

(Uma frase.)

## Status atual (YYYY-MM-DD)

(Onde está, fase, última entrega.)

## Decisões

- [[../synthesis/adr-X|ADR-001 — {título}]]: resumo de 1 linha.

## Componentes

### Workflows
- [[../entities/X]] — papel.

### Services
- [[../entities/Y]] — papel.

### Infra
- [[../entities/Z]] — papel.

## Riscos

- **Zona vermelha**: ...
- **Zona amarela**: ...
- **Zona verde**: ...

## Próximos passos

- [ ] Item acionável.

## Fontes

- [[../sources/...]]
```

## Synthesis

```markdown
---
title: "{Tese ou pergunta sintética}"
type: synthesis
categoria: {...}
domain: {profissional | pessoal | pesquisa}   # opcional; valor livre
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - ../sources/fonte-a
  - ../sources/fonte-b
status: draft
---

# {Tese ou pergunta sintética}

## Contexto

(Por que essa pergunta/tese importa agora.)

## Comparação / análise

(Ponto-a-ponto quando aplicável. Tabelas são bem-vindas.)

| Aspecto | Opção A | Opção B |
|---------|---------|---------|
| ... | ... | ... |

## Conclusão / tradeoff

(Posição clara ou tradeoff explícito.)

## Fontes usadas

- [[../sources/a]]
- [[../sources/b]]
```

## Query

Nome: `YYYY-MM-DD-pergunta-slug.md`.

```markdown
---
title: "{Pergunta original}"
type: query
categoria: {...}
tags: [...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - ../sources/...
status: draft
---

# {Pergunta original}

## Contexto

(Por que a pergunta foi feita, o que motivou.)

## Resposta

(Síntese citando wikilinks pra toda página usada.)

## Páginas consultadas

- [[../entities/X]]
- [[../concepts/Y]]

## Follow-ups

- Perguntas derivadas.
- Novas páginas sugeridas.
- Gaps que a pergunta revelou.
```

## Entrada de `log.md` — ingest

```markdown
## [YYYY-MM-DD] ingest | {Título resumido da fonte}

- **Fonte**: [[wiki/sources/slug]]
- **Páginas criadas**: [[wiki/x/a]], [[wiki/x/b]], [[wiki/y/c]]
- **Páginas atualizadas**: [[wiki/x/d]] (adicionado fato Z)
- **Contradições detectadas**: nenhuma | descrição
- **Perguntas abertas**: ...
```

## Entrada de `log.md` — query

```markdown
## [YYYY-MM-DD] query | {pergunta resumida}

- **Páginas consultadas**: [[wiki/a]], [[wiki/b]]
- **Arquivado em**: [[wiki/queries/YYYY-MM-DD-slug]] | não arquivado
```

## Entrada de `log.md` — lint

```markdown
## [YYYY-MM-DD] lint

- N contradições achadas, M corrigidas.
- N órfãs, ações: ...
- N gaps, ações: ...
- Novos sugeridos: [[x]], [[y]].
```

# CLAUDE.md — Schema do LLM Wiki

> Este arquivo é a **constituição** do wiki. Ele diz ao agente LLM como o wiki está organizado, quais convenções usar e quais workflows seguir. Atualize este arquivo quando as regras evoluírem — ele é co-evoluído entre humano e LLM.

---

## 1. Identidade e propósito

Este repositório é um **LLM Wiki** pessoal/profissional. O wiki serve a múltiplos domínios:

- **Pesquisa e aprofundamento** de temas ao longo de semanas/meses.
- **Documentação de projetos** (arquitetura, decisões, runbooks).
- **Código e engenharia** (padrões, snippets, post-mortems).
- **Referência consultiva** (procedimentos, specs, manuais).
- **Entendimentos pessoais** (sínteses, modelos mentais, conclusões).

O LLM é o maintainer. O humano é o curador: traz fontes, faz perguntas, revisa e dirige. **O humano raramente escreve páginas do wiki diretamente** — se o humano editar manualmente, o LLM respeita e preserva a edição.

**Idioma padrão:** Português (BR). Fontes podem estar em qualquer idioma; sínteses e páginas do wiki são **sempre em PT-BR**.

---

## 2. Arquitetura em 3 camadas

```
llm-wiki/
├── CLAUDE.md            # este arquivo (schema)
├── AGENTS.md            # schema equivalente para agentes genéricos
├── README.md            # orientação humana sobre o repo
├── index.md             # catálogo navegável (content-oriented)
├── log.md               # histórico append-only (chronological)
│
├── raw/                 # CAMADA 1 — Fontes brutas (imutáveis)
│   ├── sources/         # artigos, papers, transcripts, docs
│   └── assets/          # imagens, PDFs, anexos
│
└── wiki/                # CAMADA 2 — Páginas geradas pelo LLM
    ├── entities/        # pessoas, empresas, ferramentas, produtos
    ├── concepts/        # ideias, termos, padrões, frameworks
    ├── sources/         # uma página por fonte ingerida
    ├── projects/        # páginas de projeto
    ├── synthesis/       # comparações, teses, análises transversais
    └── queries/         # respostas arquivadas a perguntas feitas
```

**Regras duras:**
- `raw/` é **imutável**. O LLM lê, nunca modifica. Se uma fonte precisa ser corrigida, crie uma nova versão.
- `wiki/` é **owned pelo LLM**. O LLM cria, atualiza e mantém consistência.
- `index.md`, `log.md`, `CLAUDE.md`, `AGENTS.md` na raiz.

---

## 3. Convenções de páginas

### Nomeação
- `kebab-case-em-portugues.md` (ex: `llm-wiki.md`, `rag.md`, `vannevar-bush.md`).
- Sem acentos nos nomes de arquivo (só no conteúdo).
- Uma página = um conceito/entidade. Prefira **muitas páginas pequenas** a poucas gigantes.

### Frontmatter YAML (obrigatório em toda página de wiki)

```yaml
---
title: Nome legível da página
type: entity | concept | source | project | synthesis | query
categoria: pesquisa | projeto | engenharia | referencia | pessoal
tags: [tag1, tag2]
created: 2026-01-15
updated: 2026-01-15
sources: [../sources/nome-da-fonte.md]   # fontes que alimentam esta página
status: stub | draft | stable              # maturidade do conteúdo
# --- campos opcionais ---
domain: opcional                           # só em synthesis, marca se a conclusão é domain-specific
repo: https://github.com/org/repo         # URL HTTPS do repo git associado — auto-preenchido pelos scripts wiki-dump/wiki-import quando o PWD é git repo. Use em sources de sessão e páginas de project. Dimensão distinta de event_5w1h.where (ferramenta de trabalho).
event_5w1h:                                # só em sources do tipo evento (sessão, reunião, incidente, post-mortem)
  what: "o que aconteceu"
  when: "quando (data + hora + duração se couber)"
  where: "onde (sistema, sala, canal)"
  who: [lista, de, participantes]
  why: "motivo / ticket originador"
  how: "abordagem / como foi conduzido"
---
```

**Campos obrigatórios:** `title`, `type`, `categoria`, `tags`, `created`, `updated`, `status`.
**`sources`** é obrigatório exceto em páginas do tipo `source` (que citam a si mesmas em `../../raw/`).
**Vocabulário controlado de `categoria`:** fixo no início — se precisar de outro, edite esta lista primeiro.

Use YAML porque o Dataview (Obsidian) consegue consultar frontmatter e gerar tabelas dinâmicas. Com os campos `categoria` e `event_5w1h` estruturados, queries como `todas as sessões do projeto X` ou `todos os incidentes em 2026` viram uma linha de Dataview.

### Links internos
Use **wikilinks estilo Obsidian**: `[[nome-da-pagina]]` ou `[[nome-da-pagina|texto customizado]]`.
Se o link aponta para página em subdiretório diferente, use caminho relativo: `[[../concepts/rag]]`.

### Estrutura recomendada por tipo

**Entity** (`wiki/entities/`):
```
# {Nome}
## TL;DR (1-2 frases)
## Contexto
## Relações ([[links]] para outras entidades/conceitos)
## Fatos datados (quando possível, com fonte)
## Fontes
```

**Concept** (`wiki/concepts/`):
```
# {Conceito}
## Definição
## Por que importa
## Mecânica / como funciona
## Exemplos
## Contraste com ideias próximas
## Contradições ou debates (se houver)
## Fontes
```

**Source** (`wiki/sources/`): uma página por fonte ingerida.
```
# {Título da fonte}
## Metadados (autor, data, link, onde está em raw/)
## TL;DR (3-5 bullets)
## Takeaways principais
## Citações notáveis
## Páginas do wiki afetadas por esta ingestão
## Perguntas abertas que a fonte levanta
```

**Project** (`wiki/projects/`):
```
# {Projeto}
## Objetivo
## Status atual (data)
## Decisões (com links para ADRs)
## Componentes
## Riscos
## Próximos passos
```

**Synthesis** (`wiki/synthesis/`): análises transversais que combinam várias fontes.

**Query** (`wiki/queries/`): respostas arquivadas. Data no nome: `2026-01-15-como-funciona-x.md`.

---

## 4. Workflows

### 4.1 Ingest (ingestão de nova fonte)

Gatilho: usuário coloca algo em `raw/sources/` e pede para ingerir.

**Como o arquivo chega em `raw/sources/`:** três caminhos — (a) arrastar manualmente (PDFs, papers baixados, transcrições exportadas); (b) `/wiki-dump` que gera um digest de sessão (ver 4.5); (c) `/wiki-import` que transfere um `.md` pré-existente (ver 4.6).

Passos:
1. **Ler** a fonte inteira (use Read; para PDFs grandes, usar `pages:` em faixas).
2. **Discutir TL;DR** com o usuário (3-5 bullets, perguntar se captou o essencial).
3. **Criar/atualizar `wiki/sources/{slug}.md`** com frontmatter completo.
4. **Identificar entidades e conceitos** mencionados.
   - Para cada entidade/conceito: se já existe página, atualizar (adicionar fatos, citar fonte); se não, criar stub.
   - Tipicamente uma fonte toca **10-15 páginas**.
5. **Atualizar `index.md`** com novas páginas.
6. **Append em `log.md`** usando prefixo consistente:
   ```
   ## [2026-01-15] ingest | {Título da fonte}
   - Páginas criadas: [[x]], [[y]]
   - Páginas atualizadas: [[z]]
   - Contradições detectadas: nenhuma | descrição
   - Perguntas que ficaram: ...
   ```
7. **Reportar** ao usuário o resumo das mudanças.

### 4.2 Query (consulta)

Gatilho: usuário faz uma pergunta ao wiki.

Passos:
1. **Ler `index.md` primeiro** para localizar páginas relevantes.
2. **Ler as páginas** identificadas (não faça RAG por similaridade se o index cobre).
3. **Sintetizar** resposta com **citações wikilink** para todas as páginas usadas.
4. **Perguntar ao usuário** se a resposta deve ser arquivada como `wiki/queries/YYYY-MM-DD-slug.md`. Se sim, arquivar.
5. **Append em `log.md`**:
   ```
   ## [2026-01-15] query | {pergunta resumida}
   - Páginas consultadas: [[a]], [[b]]
   - Arquivado em: [[queries/2026-01-15-slug]]
   ```

### 4.3 Lint (health check)

Gatilho: usuário pede "lint" ou periodicamente.

Checklist:
- **Contradições** entre páginas (ex: duas fontes dando números diferentes para o mesmo fato).
- **Claims stale** — fontes antigas superadas por novas.
- **Órfãs** — páginas sem inbound links (usar `grep -r "\[\[{nome}" wiki/`).
- **Conceitos mencionados sem página própria** (candidatos a criar).
- **Cross-references faltando** (página A menciona tema de B em texto mas sem wikilink).
- **Gaps de dados** que uma web search poderia preencher (sugerir ao usuário).
- **Frontmatter inconsistente** (tipo faltando, data desatualizada).

Produzir relatório em markdown e, após aprovação do usuário, aplicar correções.

Log entry:
```
## [2026-01-15] lint
- N contradições achadas, M corrigidas
- N órfãs, ações: ...
- Sugestões de novas páginas/fontes: ...
```

### 4.4 Revise (revisão manual do humano)

Se o humano editar uma página manualmente, o LLM preserva a edição. Se detectar edição humana (commit diff, ou pelo contexto), **não sobrescreve** — integra a nova informação respeitando o que o humano escreveu.

### 4.5 Ingest de sessão de LLM (Claude Code, Cursor, Cowork, etc.)

Uma "sessão" é a conversa com um agente durante trabalho em projeto. O **transcrito cru é barulhento** e não deve entrar direto em `raw/sources/`. A unidade correta de ingest é um **digest curado** gerado **pelo próprio agente ao final da sessão**, com o transcrito completo opcionalmente preservado em `raw/assets/sessions/` como backup.

Use o slash command `/wiki-dump` (da skill `llm-wiki`) pra gerar o esqueleto do digest. Detalhes operacionais no `commands/wiki-dump.md` da skill.

### 4.6 Importar um `.md` pré-existente

Gatilho: você tem um arquivo markdown pronto (ata, post-mortem, memo, doc de arquitetura, notas avulsas) e quer arquivar no wiki como fonte bruta — sem gerar digest de sessão, sem reescrever.

Use `/wiki-import <arquivo>`. A ingestão real (criar `wiki/sources/`, derivar conceitos/entidades, atualizar `index.md`) continua sendo o workflow 4.1 — feito numa próxima sessão aberta na pasta do wiki.

**Quando usar wiki-import vs wiki-dump:**
- **wiki-dump** = a fonte **é** a sessão em si (o que o agente acabou de fazer).
- **wiki-import** = a fonte **já existe** como arquivo (gerada por humano, exportada de outra ferramenta, baixada).

---

## 5. Convenções do `index.md`

- Organizado por categoria: Entidades, Conceitos, Projetos, Fontes, Sínteses, Queries.
- Cada linha: `- [[slug]] — {one-liner}`
- Opcional: badge de status `(stub)` `(draft)` para páginas em construção.
- Atualizado em todo ingest.

## 6. Convenções do `log.md`

- **Append-only**. Entradas no fim (cronológico crescente) pra simplicidade de `tail`.
- Prefixo de entrada: `## [YYYY-MM-DD] {tipo} | {resumo}`
  - tipos: `ingest`, `query`, `lint`, `note`, `refactor`
- Permite `grep "^## \[" log.md | tail -20` para ver as últimas 20 entradas.

## 7. Tips operacionais

- **Imagens**: quando uma fonte tem imagens relevantes, o LLM lê o texto primeiro e depois abre imagens individualmente via Read para contexto adicional.
- **Diffs**: antes de aplicar mudanças grandes, mostrar diff proposto ao usuário.
- **Batch vs interativo**: por padrão **ingestão interativa** (uma fonte por vez, com discussão). Só fazer batch quando o usuário pedir.
- **Web search**: permitido para preencher gaps, mas sempre citar URL e arquivar como nova fonte em `raw/sources/` se material for substancial.
- **Git**: se o wiki for um repo git, o LLM pode sugerir commits; o humano aprova.
- **Dataview**: páginas com frontmatter rico permitem tabelas dinâmicas no Obsidian.

## 8. Anti-padrões (o que NÃO fazer)

- Não modificar `raw/`.
- Não criar páginas gigantes — quebre em múltiplas com wikilinks.
- Não repetir informação; **link em vez de copiar**.
- Não fazer RAG por similaridade embedding se `index.md` resolve.
- Não ingerir sem discutir TL;DR com o humano.
- Não esquecer de atualizar `index.md` e `log.md` após ingest.
- Não usar inglês em páginas do wiki (exceto termos técnicos sem tradução).
- **Não misturar tipos em uma única página.** Se a informação é híbrida (ex: fonte + síntese), crie duas páginas e linke. Cada página pertence a exatamente um `type`.
- **Não despejar transcrito bruto de sessão em `raw/sources/`.** Fontes de sessão são **digests curados** (ver 4.5). O transcrito cru, se preservado, vai para `raw/assets/sessions/`.
- **Não omitir `categoria:`** — sem ela, queries Dataview por domínio ficam furadas.

## 9. Evolução deste schema

Se durante o uso aparecer padrão que deveria ser regra, proponha editar este CLAUDE.md. O usuário aprova e aplicamos.

**Personalize as categorias conforme o uso real do seu wiki.** As categorias default (`pesquisa | projeto | engenharia | referencia | pessoal`) são um ponto de partida genérico — se você trabalha com um projeto específico (ex: `projeto-acme`) ou tem um domínio que merece categoria própria, edite a lista na seção 3 antes de começar a usar, não depois.

## 10. Ferramenta em repo separado

A ferramenta (skill + scripts + slash commands) vive num **repositório git separado** chamado `llm-wiki-skill`. Este repo aqui (`llm-wiki`) contém **apenas conteúdo** — schema, fontes brutas e páginas geradas.

Por que separar:
- **Ciclos de vida diferentes:** wiki commita diariamente (ingest de fontes, updates de páginas); skill muda raramente (é software versionado).
- **Privacidade diferente:** wiki contém conteúdo pessoal/profissional; skill é genérica e distribuível.

A skill lê este `CLAUDE.md` em runtime via `$LLM_WIKI/CLAUDE.md`. Mudanças no schema pegam na hora. Se a mudança afeta os scripts (template de digest, validação de flags), é no repo `llm-wiki-skill` que se edita.

---

**Última atualização:** (data de criação) — scaffold inicial via `/wiki-init`. Personalize as categorias e adicione histórico de evoluções conforme o wiki crescer.

---
name: llm-wiki
description: Mantém um LLM Wiki pessoal/profissional — uma base de conhecimento persistente em markdown interligado (wikilinks estilo Obsidian) com schema rígido (frontmatter YAML, tipos entity/concept/source/project/synthesis/query, categoria controlada). Use SEMPRE que o usuário mencionar qualquer uma destas intenções — mesmo sem citar "wiki" explicitamente — "ingerir fonte", "arquivar esse paper/artigo/doc", "bota isso no wiki", "salva essa sessão", "dump da sessão", "consulta o wiki", "o que o wiki diz sobre X", "quais páginas falam de Y", "lint do wiki", "health check do wiki", "importa esse markdown", "procura nas minhas notas", "adiciona essa fonte", "cria página pra esse conceito", "o que eu sei sobre X", "what does my wiki say", "ingest this source", "archive this document", "save this session digest", "query my notes", "import this markdown". A skill localiza o wiki (via `$LLM_WIKI` ou detecção), lê o schema canônico em `CLAUDE.md` do repo, e executa os 5 workflows (ingest / query / lint / session-dump / import) respeitando as convenções. Funciona de qualquer pasta — não precisa estar dentro do wiki.
---

# llm-wiki

Este é o sistema de manutenção de um **LLM Wiki** — uma base de conhecimento em markdown interligado que o LLM mantém para um humano. O humano traz fontes e faz perguntas; o LLM cria/atualiza páginas, preserva consistência, e mantém índices.

## O que é um LLM Wiki

Não é RAG. Não é embeddings. É um **repositório de markdown curado**, em oposição clara a um store efêmero de chunks:

- **Páginas pequenas e focadas** (uma página = um conceito/entidade) ligadas por `[[wikilinks]]`.
- **Schema declarado** em frontmatter YAML com vocabulário controlado.
- **Três camadas**: `raw/` (fontes imutáveis) + `wiki/` (páginas geradas pelo LLM) + schema (`CLAUDE.md`, `index.md`, `log.md`).
- **Idempotente e reconstrutível**: se um dia o LLM tiver que reprocessar tudo, o schema + as fontes cruas são suficientes.

O humano é curador (traz material, revisa). O LLM é maintainer (escreve páginas, detecta contradições, preserva invariantes).

## Passo 0 — localizar o wiki

`$LLM_WIKI` aponta **apenas para a pasta do wiki** (conteúdo: `CLAUDE.md`, `raw/`, `wiki/`, `index.md`, `log.md`). Os scripts e slash commands vivem separados, dentro desta skill, em `~/.claude/skills/llm-wiki/` — independentes do repo do wiki.

Antes de qualquer operação, descubra onde o wiki vive. Ordem de precedência:

1. **Variável `$LLM_WIKI`** — execute `echo "$LLM_WIKI"` via Bash. Se populada e o path existe e contém `CLAUDE.md`, use.
2. **Config file** — se existir `~/.config/llm-wiki/path`, leia.
3. **Pasta atual** — se o `pwd` atual contém `CLAUDE.md` com a frase "LLM Wiki" nas primeiras 50 linhas, use `pwd`.
4. **Perguntar** — se nenhuma detecção funcionou, pergunte ao usuário onde está o wiki e **sugira** exportar `$LLM_WIKI` no shell pra acelerar as próximas interações.

Guarde o path como `WIKI_ROOT` para o resto da conversa.

## Passo 1 — carregar o schema canônico

**Sempre** leia `$WIKI_ROOT/CLAUDE.md` antes de escrever qualquer página. O `CLAUDE.md` do repo é a **fonte de verdade** — esta skill é só um trigger + quick-reference. Se houver qualquer divergência entre o que esta skill diz e o `CLAUDE.md`, o `CLAUDE.md` vence.

Motivo: o schema evolui no repo (o humano atualiza o `CLAUDE.md` quando regras mudam); a skill pode ficar defasada. O contrato é: a skill te traz até a porta, o `CLAUDE.md` diz como se comportar dentro.

Se o `CLAUDE.md` não existir no `WIKI_ROOT`, o wiki ainda não foi inicializado. Pergunte se é isso mesmo — se sim, ofereça rodar `/wiki-init` (ou invoque `scripts/wiki-init` diretamente via Bash) que cria o scaffold a partir dos templates em `reference/bootstrap/`. Se o usuário esperava ter um wiki já populado, pare e ajude a diagnosticar (`$LLM_WIKI` apontando pra pasta errada, wiki em outro path, etc.). **Nunca adivinhe o schema** — sempre use os templates canônicos.

## As 5 operações (+ bootstrap)

> **Bootstrap (uma vez):** se `$LLM_WIKI` aponta pra pasta vazia ou sem `CLAUDE.md`, rode `/wiki-init` (ou invoque `scripts/wiki-init` via Bash). Cria scaffold canônico a partir de `reference/bootstrap/`. Não é uma das 5 operações — é pré-requisito antes de ingest/query/lint/session-dump/import funcionarem.


Cada operação tem workflow próprio. Resumos abaixo; detalhe completo em `reference/workflows.md` — leia esse arquivo antes de executar se a operação envolver mais de uma ou duas páginas, ou se você não se lembra de um detalhe.

### 1. Ingest — arquivar uma nova fonte no wiki

**Triggers**: "ingere essa fonte", "arquiva esse paper", "bota isso no wiki", "processa o PDF".

**Fluxo curto**:
1. Leia a fonte inteira (para PDFs grandes, use `pages:` em faixas de 20).
2. Proponha TL;DR em 3-5 bullets ao usuário e **espere confirmação** — nunca escreva antes.
3. Crie `wiki/sources/{slug}.md` com frontmatter completo (ver `reference/templates.md`).
4. Identifique entidades e conceitos mencionados; crie **stubs** pros que não existem, atualize os existentes citando a fonte. Uma fonte típica toca 10-15 páginas.
5. Atualize `index.md` com todas as páginas novas.
6. Append em `log.md`: `## [YYYY-MM-DD] ingest | {título}` listando páginas criadas/atualizadas e perguntas abertas.
7. Reporte ao usuário o que mudou.

### 2. Query — responder pergunta usando o wiki

**Triggers**: "o que o wiki diz sobre X", "quais páginas falam de Y", "consulta o wiki", "procura nas minhas notas".

**Fluxo curto**:
1. **Leia `index.md` primeiro** — não faça busca por similaridade se o índice cobre.
2. Leia as páginas relevantes.
3. Sintetize a resposta **citando wikilinks** pra toda página usada.
4. Ofereça arquivar como `wiki/queries/YYYY-MM-DD-{slug}.md` se a resposta tiver valor persistente.
5. Se arquivar, append em `log.md`: `## [YYYY-MM-DD] query | {pergunta}`.

### 3. Lint — health check do wiki

**Triggers**: "lint do wiki", "health check", "cleanup", "revisa o wiki".

**Checklist**: contradições entre páginas, claims stale, páginas órfãs (sem inbound links), conceitos mencionados sem página própria, cross-references faltando, gaps preenchíveis por web search, frontmatter inconsistente.

Produza relatório em markdown, **espere aprovação**, depois aplica correções. Append `## [YYYY-MM-DD] lint | {resumo}` em `log.md`.

### 4. Session-dump — curar digest de sessão de trabalho

**Triggers**: "salva essa sessão", "dump da sessão", "arquiva o que fizemos hoje", "vira isso em fonte do wiki".

Use quando a conversa atual é trabalho real em projeto (código, decisões, debug) e tu quer preservar o aprendizado no wiki — **não como transcrito cru**, mas como digest curado.

**Fluxo curto**:
1. Verifique `$LLM_WIKI` exportado. Se não, avise e pare.
2. Invoque `"$HOME/.claude/skills/llm-wiki/scripts/wiki-dump" [projeto] [slug] --no-edit` via Bash. Captura o path do digest esqueleto que o script imprime.
3. **Você (agente)** preenche o digest — não delegue ao humano. Seções: `event_5w1h` (what/when/where/who/why/how), Contexto inicial, Decisões (com alternativas descartadas), Arquivos modificados, Problemas resolvidos (sintoma → root cause → fix), Padrões descobertos, Follow-ups.
4. **Não invente.** Se `why` não ficou claro, deixe `<!-- confirmar com Gustavo -->`.
5. Reporte path do digest + 3-bullet resumo. Sugira ingerir na próxima sessão aberta no wiki.

Detalhes completos no slash command correspondente em `commands/wiki-dump.md` (dentro desta skill).

### 5. Import — trazer markdown pré-existente como fonte bruta

**Triggers**: "importa esse markdown", "bota esse doc no wiki", "arquiva essa ata/post-mortem/memo".

Uso: tu tem um `.md` pronto (ata, post-mortem, notas) e quer arquivar como fonte **sem** reprocessar.

**Fluxo curto**:
1. `"$HOME/.claude/skills/llm-wiki/scripts/wiki-import" <arquivo> [projeto] [slug] [--categoria X] [--move]` via Bash.
2. Script copia pra `$LLM_WIKI/raw/sources/YYYY-MM-DD-{projeto}-{slug}.md`.
3. Se o arquivo não tem frontmatter, o script injeta template. Ofereça completar os campos faltando.
4. Reporte path final. **A ingestão real** (criar `wiki/sources/`, derivar páginas, atualizar `index.md`) é o workflow de Ingest (1) e acontece depois.

## Regras fundamentais (hard rules)

Estas são não-negociáveis — derivadas das seções 2, 3, e 8 do `CLAUDE.md`:

1. **`raw/` é imutável.** LLM lê, nunca modifica. Correções de fonte → nova versão.
2. **Páginas do wiki sempre em PT-BR.** Fontes podem ser em qualquer idioma, mas páginas geradas são PT-BR.
3. **Frontmatter YAML obrigatório** em toda página de wiki — com campos `title`, `type`, `categoria`, `tags`, `created`, `updated`, `status`. Campos opcionais relevantes: `sources`, `domain` (só em synthesis), `event_5w1h` (só em source tipo evento), `repo` (URL HTTPS do repo git, em sources de sessão e páginas de project). Se o frontmatter já tem `repo:` (scripts auto-preenchem quando o PWD é git repo), **preserve** — é rastreabilidade de qual codebase originou a página.
4. **Vocabulário controlado de `categoria`**: `pesquisa | projeto-bhub | engenharia | regra-de-negocio | pessoal`. Se precisar de outro, **edite o `CLAUDE.md` primeiro**.
5. **Vocabulário controlado de `type`**: `entity | concept | source | project | synthesis | query`. Uma página = um tipo. Se o conteúdo é híbrido, crie duas páginas e linke.
6. **Links são wikilinks Obsidian**: `[[slug]]` ou `[[../subdir/slug]]`. Nunca URLs relativas `./foo.md`.
7. **Nomes de arquivo são kebab-case sem acento**: `llm-wiki.md`, `vannevar-bush.md`.
8. **`index.md` e `log.md` atualizados em todo ingest/lint.** Nunca pule.
9. **Discuta TL;DR com o humano antes de criar páginas**. Ingerir sem confirmação é anti-padrão.
10. **Preserve edições humanas.** Se o humano editou manualmente, respeite — integre novas infos sem sobrescrever o trabalho dele.

## Anti-padrões comuns

Estes erros derrubam a qualidade do wiki — evite ativamente:

- **Criar página gigante.** Quebre em múltiplas pequenas com wikilinks.
- **Repetir informação.** Link em vez de copiar.
- **Misturar tipos.** Uma página não pode ser "source + synthesis". Crie duas.
- **Despejar transcrito bruto em `raw/sources/`.** Sessões viram digests curados (ver operação 4). Transcrito cru vai pra `raw/assets/sessions/`.
- **Omitir `categoria`.** Sem ela, queries Dataview por domínio ficam furadas.
- **Usar inglês em página do wiki.** Só termos técnicos sem tradução (ex: "fan-out workflow").
- **Modificar `raw/`.** Nunca.

## Scripts disponíveis

Vivem **dentro desta skill**, em `~/.claude/skills/llm-wiki/scripts/` (não no repo do wiki):

- **`wiki-init`** — inicializa scaffold de wiki novo (CLAUDE.md, AGENTS.md, README.md, index.md, log.md + estrutura de diretórios) copiando os templates de `reference/bootstrap/`. Flags: `--force`. Rode uma vez na instalação.
- **`wiki-dump`** — gera esqueleto de session digest em `$LLM_WIKI/raw/sources/`. Flags: `--no-edit`, `--categoria <cat>`, `--transcript <path>`.
- **`wiki-import`** — copia/move markdown externo pra `$LLM_WIKI/raw/sources/` com frontmatter básico. Flags: `--categoria <cat>`, `--move`, `--no-date-prefix`.

Ambos exigem `$LLM_WIKI` exportado (pra saber onde escrever). Se não estiver, os scripts falham com mensagem útil — repasse ao usuário.

Invoque sempre pelo path absoluto `$HOME/.claude/skills/llm-wiki/scripts/<script>` — **não** dependa dos scripts estarem no `$PATH`.

Slash commands equivalentes (`/wiki-init`, `/wiki-dump`, `/wiki-import`, `/wiki-ingest-session`) vivem em `commands/` dentro desta skill; o usuário instala criando symlinks em `~/.claude/commands/` (ver README da skill).

## Quando recorrer a `reference/`

Esta skill é um quick-reference. Para detalhe operacional, leia:

- **`reference/schema.md`** — frontmatter completo, campos opcionais (`domain`, `event_5w1h`), estrutura recomendada por tipo.
- **`reference/workflows.md`** — passo-a-passo detalhado dos 5 workflows (ingest / query / lint / session-dump / import), incluindo edge cases.
- **`reference/templates.md`** — templates prontos pra copiar (entity, concept, source, project, synthesis, query).
- **`reference/troubleshooting.md`** — problemas comuns (wiki não encontrado, `$LLM_WIKI` unset, conflito de edição humana, contradições entre fontes).

Regra prática: se a operação toca mais de 3 páginas, ou se você não executou esse workflow específico na sessão atual, **leia o `reference/workflows.md`** antes de começar.

## Tips operacionais

- **Paralelismo.** Quando criar múltiplos stubs (ex: 10 entidades novas em um ingest), use múltiplas tool calls Write em paralelo — cada uma é independente.
- **Diff antes de grande mudança.** Se for modificar uma página madura (status `stable`), mostre o diff proposto e espere aprovação.
- **Web search permitido** pra preencher gaps, mas cite URL e arquive como nova fonte em `raw/sources/` se o material for substancial (>1 parágrafo de conteúdo reutilizável).
- **Git é do humano.** Sugira commits no final de operações grandes; não commite sem aprovação.
- **Batch vs interativo.** Default é interativo (uma fonte por vez, com discussão). Só batch quando o usuário pedir explicitamente.

## Exemplos de invocação

**Usuário**: "cara, acabei de baixar esse paper do Cao sobre extração 5W1H, bota no wiki"
→ Operação 1 (Ingest). Localize wiki, leia `CLAUDE.md`, abra o PDF, discuta TL;DR, crie source + conceitos + entidades, atualize index/log.

**Usuário**: "o que a gente já anotou sobre Restate?"
→ Operação 2 (Query). Leia `index.md`, encontre páginas que mencionam Restate, sintetize com wikilinks.

**Usuário**: "salva essa sessão, acabamos de resolver um bug chato"
→ Operação 4 (Session-dump). Rode `wiki-dump`, preencha digest com root cause do bug, reporte path.

**Usuário**: "importa esse memo aqui como fonte, depois a gente ingere direito"
→ Operação 5 (Import). Rode `wiki-import`, confirme path final, não ingere ainda.

**Usuário**: "faz um lint, tô achando que tem coisa duplicada"
→ Operação 3 (Lint). Rode checklist, produza relatório, espere aprovação, aplique correções.

# Troubleshooting

Problemas comuns e como resolver.

## `$LLM_WIKI` não está exportado

**Sintoma**: scripts `wiki-dump` / `wiki-import` falham com `LLM_WIKI is not set`. Skill não consegue localizar o wiki.

**Fix**:

1. Verifique: `echo "$LLM_WIKI"`.
2. Se vazio, peça ao usuário adicionar ao shell rc (`~/.zshrc` ou `~/.bashrc`):
   ```bash
   export LLM_WIKI="$HOME/projetos/llm-wiki"   # aponta só pra pasta do wiki
   ```
   `$LLM_WIKI` é só pra conteúdo (`CLAUDE.md`, `raw/`, `wiki/`). Os scripts vivem dentro da skill, em `~/.claude/skills/llm-wiki/scripts/` — não precisa de PATH extra.
3. Re-source: `source ~/.zshrc`.
4. Confirme: `echo "$LLM_WIKI"` deve imprimir o path.

**Paliativo temporário** (sessão atual apenas): exportar inline:
```bash
export LLM_WIKI=/path/to/llm-wiki && "$HOME/.claude/skills/llm-wiki/scripts/wiki-dump" --no-edit
```

## Wiki não encontrado pela skill

**Sintoma**: skill ativada mas não localiza o `CLAUDE.md`.

**Fallback chain**:
1. `$LLM_WIKI` env var populada e path existe.
2. `~/.config/llm-wiki/path` existe (arquivo de 1 linha com o path).
3. `pwd` atual tem `CLAUDE.md` com "LLM Wiki" nas primeiras 50 linhas.
4. Último recurso: **pergunte ao usuário**.

Se nada funciona, não chute. Avise: "Não consegui localizar o wiki. Qual o path absoluto?" e sugira exportar a env var pro futuro.

## Fonte já ingerida, usuário quer reingerir

**Sintoma**: `wiki/sources/slug.md` já existe pra essa fonte.

**Opções**:

1. **Update** — provavelmente o melhor. Leia a fonte de novo, compare com o que tem, adicione deltas à página source, atualize páginas derivadas se necessário.
2. **Recriar** — só se o usuário explicitar que quer apagar e começar do zero. Antes, salve a versão atual em `raw/assets/wiki-archive/YYYY-MM-DD-slug-v1.md` pra histórico.
3. **Nova versão** — se a fonte em si mudou (paper foi revisado), trate como fonte separada: `slug-v2` ou `slug-2026-04`.

**Log** qualquer caso como `## [YYYY-MM-DD] reingest | {título}`.

## Contradição entre fontes

**Sintoma**: fonte nova diz X, página existente (citando fonte antiga) diz Y.

**Fix**:

1. **Não sobrescreva** a página existente. O LLM wiki preserva histórico.
2. Adicione seção "Contradições ou debates" à página do conceito/entidade, cite ambas as fontes:
   ```markdown
   ## Contradições ou debates

   - [[../sources/fonte-antiga]] afirma X.
   - [[../sources/fonte-nova]] (mais recente) afirma Y.
   - Status: em aberto — confirmar com...
   ```
3. No log, marque: `- **Contradições detectadas**: fonte X vs Y sobre Z`.
4. Se a contradição é trivial (erro da fonte antiga, aceito como corrigido), apenas atualize, mas **mantenha nota de rodapé** citando a correção.

## Humano editou uma página manualmente

**Sintoma**: diff de commit mostra edição humana em `wiki/` que você não fez.

**Regra**: preserve. Integre novas infos **ao redor** do que o humano escreveu. Nunca apague conteúdo humano.

**Detecção**: se você está prestes a editar uma página e o conteúdo atual não bate com o que você escreveu na última sessão, assuma edição humana e proceda com cuidado.

## Frontmatter inválido (categoria fora do vocabulário)

**Sintoma**: alguém (humano ou agente novato) pôs `categoria: arquitetura` ou outra fora do vocabulário.

**Fix**:

1. O vocabulário real vem do `CLAUDE.md` do wiki específico. Leia ele pra saber as categorias válidas. Exemplo genérico do bootstrap: `pesquisa | projeto | engenharia | referencia | pessoal`.
2. Se o conceito pedido é legítimo e recorrente, proponha **editar o `CLAUDE.md`** pra adicionar ao vocabulário.
3. Se é só um engano, corrija pro valor correto mais próximo e cite no log: `- **Frontmatter**: corrigido [[slug]] (arquitetura → engenharia)`.

**Nunca introduza valores ad-hoc** — o vocabulário controlado é o que permite query Dataview consistente.

## Páginas gigantes

**Sintoma**: página ultrapassou 300 linhas, virou árvore de seções.

**Fix**: quebre.

1. Identifique as seções autônomas (ex: "Histórico", "API", "Exemplos").
2. Crie páginas separadas pra cada (tipicamente concept ou entity).
3. A página original vira "hub" — índice curto com wikilinks pras sub-páginas.
4. Log: `## [YYYY-MM-DD] refactor | split [[a]] em [[a]], [[b]], [[c]]`.

## Wikilink quebrado

**Sintoma**: `[[slug]]` aponta pra página inexistente.

**Fix**:

1. Rode verificação: grep `\[\[` em todas as páginas, extraia slugs, compare com arquivos existentes.
2. Pra cada quebrado:
   - Se o slug mudou: atualize o link.
   - Se a página nunca existiu: crie stub ou remova o link (dependendo da intenção).
3. Log como parte de lint.

Script Python rápido pra rodar manualmente:

```python
import re
from pathlib import Path

WIKI = Path("wiki")
links = set()
for md in WIKI.rglob("*.md"):
    text = md.read_text()
    for m in re.finditer(r'\[\[([^\]|]+)', text):
        slug = m.group(1).strip().split('/')[-1].removesuffix('.md')
        links.add(slug)

existing = {md.stem for md in WIKI.rglob("*.md")}
broken = links - existing
for b in sorted(broken):
    print(b)
```

## Slug genérico de session-dump

**Sintoma**: `wiki-dump` sem argumentos gerou `sess-1639.md`.

**Fix**: ofereça ao usuário um slug melhor baseado no conteúdo da sessão (ex: `parser-nfe-xml-4-0`). Se aprovado, renomeie:

```bash
mv "$LLM_WIKI/raw/sources/2026-04-20-projeto-sess-1639.md" \
   "$LLM_WIKI/raw/sources/2026-04-20-projeto-parser-nfe-xml-4-0.md"
```

Atualize o `title` no frontmatter também.

## Múltiplos agentes editando ao mesmo tempo

**Sintoma**: o wiki é compartilhado, alguém mais pode estar editando.

**Fix**: o wiki é **single-user**. Se for virar multi-user, precisa de lock/branching via git (cada um numa branch, merge review). Para já, assuma single-writer. Log sempre identifica quem — bom pra auditoria.

## Skill ativou quando não devia

**Sintoma**: usuário disse "minha wiki do Zelda" e a skill ativou.

**Fix**: refine a description da skill. Adicione contexto pra distinguir wiki pessoal/profissional de outras wikis. Pode ser necessário iterar.

## Ingest sem TL;DR (anti-padrão)

**Sintoma**: você criou páginas direto, sem discutir TL;DR primeiro.

**Fix agora**: avise o usuário, mostre o que criou, peça revisão. Na próxima, **sempre** TL;DR antes. Este é o anti-padrão #1 do wiki por um motivo — a qualidade cai drasticamente sem o check humano.

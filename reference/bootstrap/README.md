# LLM Wiki

Base de conhecimento pessoal/profissional em markdown interligado (wikilinks estilo Obsidian), mantida colaborativamente entre você (curador) e um agente LLM (maintainer).

## O que é

- **Páginas pequenas e focadas** (uma página = um conceito/entidade) ligadas por `[[wikilinks]]`.
- **Schema declarado** em frontmatter YAML com vocabulário controlado.
- **Três camadas:** `raw/` (fontes imutáveis) + `wiki/` (páginas geradas pelo LLM) + schema (`CLAUDE.md`, `index.md`, `log.md`).
- **Idempotente e reconstrutível:** se um dia o LLM tiver que reprocessar tudo, o schema + as fontes cruas são suficientes.

## Primeiros passos

1. **Leia o `CLAUDE.md`** — é a constituição do wiki. Entenda as 5 operações (ingest / query / lint / session-dump / import) e o vocabulário de `categoria` e `type`.
2. **Personalize as categorias** se necessário (seção 3 do `CLAUDE.md`). As defaults (`pesquisa | projeto | engenharia | referencia | pessoal`) são genéricas — ajuste antes de começar a usar.
3. **Exporte `$LLM_WIKI`** apontando pra esta pasta. No `~/.zshrc` ou `~/.bashrc`:
   ```bash
   export LLM_WIKI="$HOME/caminho/pra/este/wiki"
   ```
4. **Instale a skill `llm-wiki-skill`** se ainda não instalou — ver README do repo da skill.
5. **Ingira sua primeira fonte.** Arraste um PDF/markdown pra `raw/sources/` e diga ao agente: *"ingere essa fonte pro wiki"*.

## Estrutura

```
llm-wiki/
├── CLAUDE.md            # schema (constituição) — leia primeiro
├── AGENTS.md            # espelho para agentes AGENTS.md-compatible
├── README.md            # este arquivo
├── index.md             # catálogo de todas as páginas
├── log.md               # histórico cronológico de operações
│
├── raw/                 # fontes brutas (imutáveis)
│   ├── sources/
│   └── assets/
│
└── wiki/                # páginas geradas pelo LLM
    ├── entities/
    ├── concepts/
    ├── sources/
    ├── projects/
    ├── synthesis/
    └── queries/
```

## Versionamento

Opcional mas recomendado: `git init` nesta pasta e commita as mudanças. Assim você tem histórico humano-legível de como o wiki cresceu, e a skill consegue detectar edições manuais via `git diff`.

## Uso com Obsidian

Abra esta pasta como vault no Obsidian. O frontmatter YAML permite queries Dataview (ex: *listar todas as fontes de pesquisa ingeridas em 2026*), e os wikilinks `[[...]]` renderizam como links clicáveis.

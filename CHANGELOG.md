# Changelog

## [0.2.0] - 2026-04-21

### Changed
- **Breaking (organizacional):** skill extraída do repo `llm-wiki` pra repo próprio. Agora dois repos separados:
  - `llm-wiki` — conteúdo (schema + raw + wiki)
  - `llm-wiki-skill` — ferramenta (este repo)
- Install passa a ser **3 passos**: clone + export `$LLM_WIKI` + symlinks. Não depende mais de o skill viver dentro do wiki.
- `README.md` reescrito pra refletir a nova estrutura.

### Rationale
Skill e conteúdo têm ciclos de vida diferentes (skill muda raramente, wiki commita diariamente), privacidade diferente (wiki privado, skill open-source-able) e trajetórias diferentes (skill → plugin no futuro). Separar deixa os dois limpos.

## [0.1.0] - 2026-04-20

Versão inicial. Skill nasceu dentro do repo `llm-wiki` como `skills/llm-wiki/`. Inclui:
- `SKILL.md` com 5 workflows (Ingest, Query, Lint, Session-dump, Import)
- `scripts/wiki-dump`, `scripts/wiki-import`
- Slash commands `/wiki-dump`, `/wiki-import`, `/wiki-ingest-session`
- `reference/{schema,workflows,templates,troubleshooting}.md`
- `evals/evals.json` com 3 casos (ingest, query, import)
- Iteration-1 e iteration-2 do eval rodadas — skill validada com +9.6pp de pass rate vs baseline.

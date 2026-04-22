# Changelog

## [0.3.0] - 2026-04-22

### Added
- Seção **"Regras de negócio tocadas"** no template de digest (entre "Decisões tomadas" e "Arquivos modificados"). Presente em `scripts/wiki-dump`, `reference/templates.md` e listada em `SKILL.md` + `commands/wiki-dump.md`.
- **Check novo no lint** (`reference/workflows.md` §3 item 9): project pages desatualizadas — comparar `updated:` do project com o da source mais recente que o referencia; sinalizar staleness.
- **Orientação no workflow de ingest** (`reference/workflows.md` §1 Passo 4): se a fonte toca regras de negócio de `wiki/projects/X`, atualizar o project page **no mesmo ingest**, usando a seção "Regras de negócio tocadas" do digest como gatilho.

### Rationale
Pergunta levantada em 2026-04-22: qual a melhor fonte pra manter requisitos/regras de negócio atualizados no wiki? Diagnóstico: não tem single source — regras vazam por PRD/tickets/Slack/reunião/código. O padrão que funciona é tratar o project page como hub vivo alimentado por digests recorrentes. Sem cadeia fechada "digest declara regra tocada → ingest atualiza hub → lint detecta staleness", o project page apodrece em 2-3 meses. Essas 3 edições criam a cadeia. Schema canônico vive no `CLAUDE.md` do wiki (§4.1/§4.3/§4.5.1) — esta skill espelha.

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

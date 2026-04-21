# Log — Histórico do LLM Wiki

> Append-only, cronológico crescente. Entradas antigas no topo, novas no fim — `tail` mostra as mais recentes.
>
> Formato: `## [YYYY-MM-DD] {tipo} | {resumo}` onde tipo ∈ {`ingest`, `query`, `lint`, `note`, `refactor`}.
>
> Grep útil: `grep "^## \[" log.md | tail -20`.

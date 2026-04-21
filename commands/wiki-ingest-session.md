---
description: Ingere uma sessão antiga (Cowork/Claude Code) no LLM Wiki via MCP — lista, seleciona, gera digest e arquiva.
argument-hint: [projeto-ou-query] — opcional, filtra as sessões listadas
---

# /wiki-ingest-session — ingest retroativo de sessão

Ingere uma sessão **passada** no LLM Wiki sem sair da sessão atual. Usa MCP `list_sessions` + `read_transcript` (Cowork) ou arquivos locais `~/.claude/projects/.../sessions/*.jsonl` (Claude Code CLI).

## Passos

1. **Listar candidatos.**
   - **Cowork:** chame `mcp__session_info__list_sessions` passando `$ARGUMENTS` como filtro se fornecido.
   - **Claude Code CLI:** rode via Bash `ls -lt ~/.claude/projects/*/sessions/*.jsonl | head -20` e exiba as 20 mais recentes com título/data inferidos.

2. **Apresentar lista ao usuário** com número, título, data e duração aproximada. Pedir para escolher uma.

3. **Ler transcrito.**
   - **Cowork:** `mcp__session_info__read_transcript` com o id escolhido.
   - **Claude Code CLI:** `Read` no `.jsonl` (cuidado: arquivos grandes — usar offset/limit).

4. **Sumarizar** o transcrito em 5-8 bullets antes de ir adiante. Confirmar com o usuário que captou o essencial.

5. **Extrair metadados para `event_5w1h`:**
   - `what` — objetivo da sessão (deduzir do primeiro request + primeiros assistants).
   - `when` — timestamp da primeira mensagem (formato `YYYY-MM-DD` + hora de início, + duração se inferível).
   - `where` — ferramenta + caminho do projeto (`cwd` do Claude Code, ou nome do workspace no Cowork).
   - `who` — participantes (tipicamente `["<usuário>", "Claude"]` ou `["<usuário>", "Claude Code"]`; substitua pelo nome real se conhecer).
   - `why` — motivo/ticket (perguntar se não explícito).
   - `how` — abordagem (pair-dev, debug, refactor, research).

6. **Criar o digest** invocando via Bash:
   ```
   "$HOME/.claude/skills/llm-wiki/scripts/wiki-dump" <projeto> <slug> --no-edit
   ```
   - Derive `projeto` do path da sessão (nome da pasta).
   - Derive `slug` do objetivo (kebab-case, 2-4 palavras).
   - Capture o path retornado.

7. **Preencher todas as seções do digest** com base no sumário e no transcrito. Seguir as mesmas regras de `/wiki-dump` (sem invenção, decisões com alternativas, etc.).

8. **Preservar transcrito bruto (opcional).** Perguntar ao usuário se quer manter o `.jsonl` original em `$LLM_WIKI/raw/assets/sessions/`. Se sim, reinvoque o script com `--transcript <path>` (o script cuida da cópia).

9. **Reportar** ao usuário: path do digest, resumo, e próximo passo sugerido (abrir o wiki e ingerir).

## Notas

- **Não esqueça a categoria.** Default do script é `projeto`; para sessões de pesquisa pura, use `--categoria pesquisa`. Consulte o `CLAUDE.md` do wiki pra saber se há vocabulário customizado.
- **Privacidade.** Se o transcrito tocar em dados sensíveis (senhas, tokens, PII de clientes), avise e ofereça redação antes de arquivar.
- **Sessões longas.** Para sessões >100k tokens, leia em chunks e sumarize iterativamente antes de consolidar.

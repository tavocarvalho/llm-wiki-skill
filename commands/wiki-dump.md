---
description: Gera um session digest do trabalho atual e arquiva em raw/sources/ do LLM Wiki. Auto-detecta projeto e slug.
argument-hint: [<projeto>] [<slug>] [--categoria <cat>]
---

# /wiki-dump — session digest para o LLM Wiki

Gera um digest **curado** da sessão atual e arquiva em `$LLM_WIKI/raw/sources/`, pronto para ser ingerido como fonte.

## Argumentos (todos opcionais)

- **projeto** — default: `basename $PWD`.
- **slug** — default: `<branch>-<HHMM>` se em git repo não-main/master, senão `sess-<HHMM>`.
- **--categoria** — default: `projeto`. O vocabulário real vem do `CLAUDE.md` do wiki; se o default não bater, passe explícito (ex: `--categoria pesquisa`).

Basta digitar `/wiki-dump` sem argumentos na maioria dos casos. Só passe argumentos explícitos se quiser sobrescrever os defaults (ex: `/wiki-dump --categoria engenharia` quando o trabalho é tech-debt transversal).

**Antes de preencher a categoria, leia o `CLAUDE.md` do wiki** pra saber o vocabulário real — ele pode ter categorias customizadas (ex: `projeto-acme`, `regra-de-negocio`) que divergem do default genérico do script.

## Passos (executar em ordem)

1. **Validar ambiente.** Rode `echo "$LLM_WIKI"` via Bash. Se vazio, avise o usuário que precisa exportar `LLM_WIKI` no shell e aborte.

2. **Criar o esqueleto.** Invoque via Bash (passando `$ARGUMENTS` mesmo se vazio):
   ```
   "$HOME/.claude/skills/llm-wiki/scripts/wiki-dump" $ARGUMENTS --no-edit
   ```
   O `--no-edit` é obrigatório: você (agente) vai preencher, não o usuário. O script escreve em stderr uma linha `# auto-detectado: ...` quando deriva projeto/slug, e em stdout o path absoluto do digest (última linha). Capture o path.

3. **Preencher o digest.** Edite o arquivo criado preenchendo:
   - **`event_5w1h.what`** — 1 frase com o objetivo real do que foi feito.
   - **`event_5w1h.why`** — ticket, issue ou motivo originador. Se não souber, pergunte.
   - **`event_5w1h.how`** — abordagem (ex: "pair-dev com Claude Code, TDD, refactor guiado por testes").
   - **Contexto inicial** — estado do repo/problema no início da sessão.
   - **Decisões tomadas** — cada decisão com alternativas descartadas e trade-off.
   - **Arquivos modificados** — liste os que de fato foram tocados na sessão (use `git diff --name-only` se relevante).
   - **Problemas resolvidos** — sintoma → root cause → fix.
   - **Padrões descobertos** — candidatos a virar concept/snippet no wiki.
   - **Follow-ups** — checklist acionável.

4. **Reportar ao usuário.** Mostre:
   - Caminho absoluto do digest criado.
   - Resumo em 3 bullets do que foi capturado.
   - Sugestão de comando para ingerir: _"na próxima sessão aberta em `$LLM_WIKI`, peça ingestão desta fonte"_.

## Regras importantes

- **Não invente.** Se algo não ficou claro na sessão (ex: `why`), deixe em branco com `<!-- confirmar com o usuário -->` em vez de alucinar.
- **Não inclua transcrito bruto por padrão.** A seção "Trechos de transcrito relevantes" só deve ser preenchida se agregar contexto que as seções estruturadas não cobrem.
- **Preserve decisões com contexto.** Uma decisão sem alternativa descartada ou trade-off é ruído — descarte ou complete.
- **Se o slug auto-derivado for genérico** (ex: `sess-1639` em repo sem branch descritiva), considere sugerir ao usuário um slug melhor e perguntar se quer renomear o arquivo antes de fechar.

## Pré-requisito de instalação

- `$LLM_WIKI` exportado apontando para a raiz do wiki (pasta com `CLAUDE.md`, `raw/`, `wiki/`):

  ```bash
  export LLM_WIKI="$HOME/code/llm-wiki"   # ajuste para onde o wiki vive
  ```

- A skill `llm-wiki` instalada em `~/.claude/skills/llm-wiki/` (o script vive dentro dela, em `scripts/wiki-dump`).

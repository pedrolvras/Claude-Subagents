# Claude Subagents

Esteira de trabalho para o Claude Code: agentes, a skill `planejar` e o modelo de pasta `.claude/` de um projeto.

## Conteúdo

| Pasta | O que é |
|---|---|
| `agents/` | Subagentes (`dev-task-writer`, `dev-coder`, `dev-reviewer`, `dev-test-designer`, `dev-tester`, `dev-debugger`) |
| `skills/planejar/` | Skill que monta a proposta de uma mudança grande antes de codar |
| `.claude/` | Modelo do `CLAUDE.md` e da pasta `planejamento/` para copiar em cada projeto |

## Esteira

| Etapa | Quem | Modelo |
|---|---|---|
| 1. Planejar | sessão principal + skill `planejar` | Opus medium |
| 1b. Proposta aprovada -> tarefas no `plano.md` | `dev-task-writer` | Opus medium |
| 2. Codar | `dev-coder` | Sonnet medium |
| 3. Revisar | `dev-reviewer` | Opus medium |
| 4. Montar a bateria de testes | `dev-test-designer` | Haiku |
| 5. Aplicar em dev | humano ou sessão principal | — |
| 6. Testar na tela | `dev-tester` | Sonnet medium |
| 7. Falhou sem causa óbvia | `dev-debugger` | Opus high |

## Instalar

- Agentes: copiar `agents/*.md` para `~/.claude/agents/` (todos os projetos) ou para `.claude/agents/` do projeto.
- Skill: copiar `skills/planejar/` para `~/.claude/skills/planejar/`.
- Projeto: copiar `.claude/` para a raiz do projeto e preencher os `<...>` do `CLAUDE.md`.

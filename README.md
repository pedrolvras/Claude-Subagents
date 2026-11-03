# Agents-Orca

Esteira de trabalho para o Claude Code: agentes, a skill `planejar` e o modelo de pasta `.claude/` de um projeto.

## Conteúdo

| Pasta | O que é |
|---|---|
| `agents/` | Subagentes Claude (`dev-task-writer`, `dev-reviewer`, `dev-tester`, `dev-debugger`) |
| `skills/planejar/` | Skill que monta a proposta de uma mudança grande antes de codar |
| `skills/gemini/` | Skill que roda as etapas do Gemini pelo `agy` (papéis coder e test-designer em `papeis/`) |
| `skills/testar/` | Comando `/testar <etapa>`: chama o `dev-tester` |
| `.claude/` | Modelo do `CLAUDE.md` e da pasta `planejamento/` para copiar em cada projeto |

## Esteira

| Etapa | Quem | Executor | Modelo |
|---|---|---|---|
| 1. Planejar | sessão principal + skill `planejar` | Claude | Opus medium |
| 1b. Proposta aprovada -> tarefas no `plano.md` | `dev-task-writer` | Claude | Opus medium |
| 2. Codar (lint/testes antes do reviewer) | skill `gemini`, papel coder | Gemini (`agy`) | Gemini 4 Argon (Medium) |
| 3. Revisar | `dev-reviewer` | Claude | Opus medium |
| 4. Montar a bateria de testes | skill `gemini`, papel test-designer | Gemini (`agy`) | Gemini 3.8 Flash (High) |
| 5. Aplicar em dev | humano ou sessão principal | — | — |
| 6. Testar na tela | `/testar` (só o humano chama) | Claude | Sonnet medium |
| 7. Falhou sem causa óbvia | `dev-debugger` | Claude | Opus high |

Cada tarefa do Gemini roda numa worktree nova do Orca, com o `agy` num terminal visível: dá para acompanhar o trabalho ao vivo. Defeito do reviewer ou falha de lint/testes volta para a mesma sessão do Gemini, até 2 rodadas cada.

## Instalar

- Agentes: copiar `agents/*.md` para `~/.claude/agents/` (todos os projetos) ou para `.claude/agents/` do projeto.
- Skills: copiar `skills/planejar/`, `skills/gemini/` e `skills/testar/` para `~/.claude/skills/`.
- Gemini: instalar o Antigravity CLI (`agy`) e rodar `agy` uma vez no terminal para fazer o login do Google.
- Orca: a skill `gemini` usa o CLI `orca` para criar a worktree e abrir o terminal. Sem Orca, ela cai para `git worktree` e `agy -p`.
- Projeto: copiar `.claude/` para a raiz do projeto e preencher os `<...>` do `CLAUDE.md`.

---
name: "testar"
description: "Executa na tela os casos de teste de uma etapa (seção do .claude/planejamento/APLICAR.md), pelo agente dev-tester. Só pelo comando /testar <etapa>."
disable-model-invocation: true
context: fork
agent: dev-tester
argument-hint: "<etapa NN>"
---

Teste a etapa $ARGUMENTS seguindo os casos da seção dela em `.claude/planejamento/APLICAR.md`, no ambiente de dev do `.claude/CLAUDE.md`.

Sem etapa informada: usar a etapa mais recente com status "aplicada" na "Situação atual" do `.claude/planejamento/CONTEXTO.md` e dizer qual escolheu.

# Plano — tarefas em aberto

Só o que vai ser implementado agora: é a especificação que o `dev-coder` e o `dev-reviewer` leem. Não é registro nem fonte da verdade.

- **Quando a tarefa sai:** assim que o reviewer aprova (não espera aplicar nem testar). Tarefa desfeita ou substituída sai na hora.
- **Para onde vai, na saída:** aplicação e testes para a seção NN do `APLICAR.md` (a bateria completa quem monta é o `dev-test-designer`, partindo dos testes da tarefa); status para o `CONTEXTO.md`; o que foi feito para o `CHANGELOG.md` (os três nesta pasta).
- **Nunca aqui:** histórico, "foi desfeito", "situação em...". Mudou de ideia: reescrever a tarefa ou tirá-la.
- **Regra que vale para toda tarefa** (não só para uma) vai para o `.claude/CLAUDE.md`, não para cá.

## Formato de tarefa nova

Tarefa que vem de uma proposta aprovada (skill `planejar`) quem escreve é o `dev-task-writer`. Cada tarefa é autossuficiente: arquivos e camadas, tabelas, o que fazer, critérios de aceite e testes com o resultado exato esperado. O agente recebe só "implemente a tarefa X de .claude/planejamento/plano.md".

```markdown
## T1 — <título>

- **Arquivos:** `<caminho>` (o que muda em cada um)
- **Banco:** <migration nova / nenhuma>
- **O que fazer:** <passos objetivos>
- **Critérios de aceite:** <lista verificável>
- **Testes:** <caso -> resultado exato esperado, inclusive texto de mensagem>
```

---

Nenhuma tarefa em aberto.

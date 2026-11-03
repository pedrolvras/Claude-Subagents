---
name: "gemini"
description: "Executa as etapas da esteira que são do Gemini: \"2. Codar\" (papel coder) e \"4. Montar a bateria de testes\" (papel test-designer). Cada tarefa roda numa worktree nova do Orca, com o Antigravity CLI (agy) aberto num terminal visível do Orca. Inclui a checagem automática (lint/testes) antes do reviewer e a volta dos defeitos para a mesma sessão do Gemini. Use quando a esteira chegar nessas etapas ou quando pedirem \"implemente a tarefa T1\" ou \"monte os testes da etapa NN\". Não use para planejar, passar para tarefas, revisar, testar na tela nem depurar: essas etapas são do Claude."
---

# Etapas do Gemini

A sessão principal prepara uma worktree do Orca para a tarefa, abre o Gemini (`agy`) num terminal do Orca dentro dela e confere o resultado. O usuário acompanha o Gemini ao vivo no Orca. O Gemini não tem a conversa: só o texto do papel, o pedido e os arquivos da worktree. Trabalho dele só vale depois de conferido.

## Papéis

| Papel | Arquivo do papel | Modelo | Pode alterar |
|---|---|---|---|
| coder | `papeis/coder.md` | `Gemini 4 Argon (Medium)` | só os arquivos que a tarefa cita |
| test-designer | `papeis/test-designer.md` | `Gemini 3.8 Flash (High)` | só `.claude/planejamento/APLICAR.md` |

Os caminhos dos papéis são relativos à pasta desta skill. Se o `agy` recusar o modelo, rodar `agy models` e usar o mais próximo, avisando o usuário.

Abaixo, `<WT>` é o caminho da worktree, `<PAPEIS>` é a pasta `papeis/` desta skill e `<SAIDA>` é uma pasta temporária fora do projeto (ex.: `$TEMP/agy-orca`). Usar caminhos absolutos com `/`.

## 1. Worktree da tarefa (uma por tarefa)

Da raiz do projeto principal:

1. **Base certa.** O Orca cria a worktree a partir do branch remoto padrão (ex.: `origin/master`). Se `git status -sb` mostrar o branch local à frente (`ahead N`), sugerir ao usuário um push antes. Sem push, depois de criar a worktree, rodar `git merge --ff-only <branch local>` dentro dela (seguro: o branch é novo e vazio).
2. **Criar:** `orca worktree create --name tarefa-<T> --no-parent --setup skip --json`. Guardar o `path` (`<WT>`).
3. **`.claude/` na worktree.** A worktree só tem o que está commitado. Se a `.claude/` do projeto não está no git, sugerir ao usuário commitá-la. Sem commit, copiar a `.claude/` para `<WT>` e, ao fim da tarefa, copiar de volta para o projeto principal o que o fluxo mudou (`plano.md`, `APLICAR.md`).
4. **Estado inicial:** `git -C <WT> status --porcelain --untracked-files=all`, guardar a saída.
5. **Card no Orca:** `orca worktree set --worktree "path:<WT>" --comment "<T>: Gemini codando" --workspace-status in-progress --json`.

A tarefa inteira (Gemini, lint, testes, reviewer, test-designer) roda em `<WT>`. A sessão principal não edita nada em `<WT>` enquanto o Gemini trabalha.

## 2. Abrir o Gemini no Orca

```bash
CMD="agy --dangerously-skip-permissions --model \"Gemini 4 Argon (Medium)\" --add-dir \"<PAPEIS>\" --add-dir \"<SAIDA>\" -i \"Leia o arquivo <PAPEIS>/coder.md e siga essas instrucoes como o seu papel. A pasta do projeto e <WT>: trabalhe somente nela. Nao procure, nao entre (cd/Set-Location) e nao altere arquivos fora dela, mesmo que encontre outra copia do projeto. Pedido: implemente a tarefa T1 de <WT>/.claude/planejamento/plano.md. Nao faca commit nem deploy. Ao terminar, grave o resumo final (o Devolva do papel) no arquivo <SAIDA>/T1-coder.md.\""
orca terminal create --worktree "path:<WT>" --title "Gemini coder T1" --command "$CMD" --focus --json
```

- Guardar o `handle` do terminal: as correções vão para a mesma sessão.
- `-i` roda o pedido e mantém a sessão aberta. Evitar acentos e aspas dentro do pedido (vai pela linha de comando do shell do Orca).
- **Pasta confiável:** na primeira vez em cada pasta, o `agy` pergunta "Do you trust the contents of this project?". Conferir na tela (`orca terminal read --terminal <handle> --screen --json`) que a pasta é `<WT>` e só então confirmar com `orca terminal send --terminal <handle> --enter --json`.
- test-designer: mesmo comando com `papeis/test-designer.md`, o modelo da tabela, o título `Gemini test-designer T1`, o pedido `monte a bateria de testes da etapa NN a partir da tarefa T1 de .claude/planejamento/plano.md` e a saída `<SAIDA>/T1-test-designer.md`.
- O pedido cita só a tarefa e a etapa, sempre com o caminho absoluto de `<WT>`. Não recopiar o plano.
- Motivo do caminho absoluto: com `--dangerously-skip-permissions` o `agy` pode rodar comandos em qualquer pasta (`--add-dir` só acrescenta pastas, não restringe). Num teste, ele achou a cópia do projeto principal procurando o `plano.md` e trabalhou lá.

## 3. Esperar e conferir

1. Guardar `git status --porcelain` do projeto principal também. Esperar em segundo plano o arquivo de resumo: `until [ -s "<SAIDA>/T1-coder.md" ]; do sleep 15; done` (com timeout de ~25 min). Não usar `orca terminal wait --for tui-idle` com o `agy`: ele dá ociosidade enquanto o Gemini ainda está lendo arquivos. Avisar o usuário que ele pode acompanhar no terminal do Orca.
2. Ler o resumo em `<SAIDA>/T1-coder.md`. Passou do tempo sem arquivo: ler a tela (`terminal read --screen`) para ver se o Gemini parou numa pergunta ou num erro.
3. Antes de cada nova rodada (seção 5), apagar o arquivo de resumo, para a espera valer para a rodada nova.
3. Comparar `git -C <WT> status --porcelain --untracked-files=all` com o estado inicial. Arquivo alterado fora do "Pode alterar" -> parar e mostrar a lista ao usuário. Não reverter sozinho. Conferir também que o projeto principal não mudou; se mudou, o Gemini trabalhou no lugar errado: parar, mostrar ao usuário e não seguir a esteira.
4. Falha do `agy` (login, cota, modelo): mostrar a mensagem da tela ao usuário.

## 4. Coder: checagem automática antes do reviewer

Em `<WT>`, rodar os comandos "Lint/tipos" e "Testes" da seção "Stack e comandos" do `.claude/CLAUDE.md`. Comando ainda como `<...>` -> pular e dizer ao usuário.

- Passou -> delegar ao `dev-reviewer`, dizendo que o projeto está em `<WT>`.
- Falhou -> mandar as últimas ~40 linhas da saída para a mesma sessão do Gemini (ver "5. Devolver defeitos"), esperar e rodar de novo.
- Até 2 rodadas. Ainda falhando -> parar, mostrar o erro ao usuário e sugerir o `dev-debugger`.

O reviewer só recebe código que já passa no lint e nos testes.

## 5. Devolver defeitos (reviewer ou lint/testes)

Mandar só a lista de defeitos (arquivo, linha, o que está errado), numa linha, para o mesmo terminal:

```bash
orca terminal send --terminal <handle> --text "Corrija so estes defeitos, sem refazer o resto da tarefa, e atualize o resumo em <SAIDA>/T1-coder.md: <defeitos>" --enter --json
```

Depois: esperar `tui-idle`, checagem automática e reviewer de novo, focado nos defeitos apontados. Até 2 rodadas de revisão. Ainda com defeito grave -> o usuário decide (aceitar, corrigir à mão ou `dev-debugger`).

- Reviewer sem defeito -> etapa 4 (test-designer), em outro terminal na mesma `<WT>`.
- Card: `--comment "<T>: em revisão"` / `"<T>: aprovada"` e `--workspace-status in-review` / `completed`.

## 6. Ao fim da etapa

- `CHANGELOG.md` (entrada do dia), uma linha: `T1 codada pelo Gemini (Gemini 4 Argon Medium) na worktree tarefa-T1: 1 rodada de lint, 2 de revisão; arquivos: ...`.
- Depois que o reviewer aprovar: sugerir ao usuário o commit no branch da worktree, já com a mensagem pronta, e depois o merge no branch principal. Commit e merge só se ele pedir.
- Fechar o terminal do Gemini (`orca terminal close --terminal <handle> --json`) só quando a etapa acabar. Remover a worktree (`orca worktree rm`) só se o usuário pedir.
- Seguir a regra de saída do `plano.md` (a tarefa sai depois do test-designer).

## Sem Orca

Se `orca` não estiver disponível, usar `git worktree add ../<projeto>-<T> -b tarefa-<T>` e rodar o `agy` em modo print, em segundo plano, dentro dela:

```bash
agy -p "$(cat "<PAPEIS>/coder.md"; printf '\n\n---\n\nPedido: implemente a tarefa T1 de .claude/planejamento/plano.md. Não faça commit nem deploy.\n')" \
  --model "Gemini 4 Argon (Medium)" --print-timeout 20m --dangerously-skip-permissions </dev/null > "<SAIDA>/T1.log" 2>&1
```

Sempre `</dev/null` no fim (sem isso o `agy` trava). Correções com `agy --continue -p "..."` com as mesmas opções, só se não houve outra chamada ao `agy` depois.

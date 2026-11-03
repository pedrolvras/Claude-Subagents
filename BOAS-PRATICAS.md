# Boas práticas: gastar menos e evitar retrabalho

Lições do primeiro teste da esteira com o Gemini (03/11/2026), em que o uso do Claude passou de 20% para 54% em pouco tempo. As regras curtas que a sessão precisa seguir estão no `.claude/CLAUDE.md` (seção "Uso de tokens"); aqui está o porquê de cada uma.

## Por que o uso sobe rápido

- **Cada ação reenvia a conversa inteira.** Todo comando, leitura ou edição manda o histórico completo de novo ao modelo. O cache deixa mais barato, mas conta no limite. Conversa longa = cada passo mais caro.
- **O que entra no contexto fica até o fim da sessão.** Um arquivo de 30 KB lido uma vez é reenviado em todas as ações seguintes.
- **O modelo pesa.** Opus custa bem mais que Sonnet pelo mesmo trabalho.
- **Retrabalho multiplica.** Cada tentativa que falha soma várias ações, todas com o contexto cheio.

Esperar o Gemini não gasta token: quem espera é um comando de shell em segundo plano, e o Claude só acorda com o aviso de que terminou.

## 1. Sessão curta

- **Uma sessão por etapa ou por tarefa.** Acabou a tarefa -> `/clear` e sessão nova.
- Antes do `/clear`, o "Retomar daqui" do `CONTEXTO.md` tem que bastar para a sessão nova continuar sem a conversa.
- Conversa resumida automaticamente é sinal de que já passou da hora de abrir sessão nova.

## 2. Modelo certo para cada fase

| Fase | Modelo da sessão principal |
|---|---|
| Planejar (skill `planejar`), decidir, revisar proposta | Opus |
| Executar a esteira: chamar o Gemini, conferir diff, rodar lint/testes, commitar | Sonnet |
| Depurar falha sem causa óbvia | `dev-debugger` (Opus high) |

Trocar com `/model`. Os subagentes já têm o modelo deles no próprio arquivo; isto vale para a sessão principal.

## 3. Ler só o necessário

- Trecho, não arquivo inteiro: `grep`, `head`, `tail`, `sed -n 'a,bp'`, ou o Read com intervalo de linhas.
- Saída de comando sempre limitada (`| tail -40`, `| head -20`, `grep` do que importa).
- Guia ou documentação de ferramenta: buscar o comando que precisa (`--help` do subcomando), não carregar o guia inteiro.
- Não reler arquivo que já está no contexto.

## 4. Esperar sem consultar

- Trabalho longo (Gemini, testes demorados) roda em segundo plano, com **um** aviso no fim.
- Nada de consultar a tela do Gemini de tempos em tempos: cada consulta é uma ação com o contexto inteiro.
- Com o `agy`, o sinal de fim é o arquivo de resumo que ele grava (`until [ -s <arquivo> ]`). O `orca terminal wait --for tui-idle` dá falso "terminou".

## 5. Checklist antes de chamar o Gemini

Cada item abaixo custou retrabalho no primeiro teste:

- [ ] **Branch local enviado ao remoto** (`git status -sb` sem `ahead`). O Orca cria a worktree a partir do remoto; sem push, ela nasce sem os commits locais.
- [ ] **`.claude/` commitada.** A worktree só recebe o que está no git; sem isso, o Gemini não acha o `plano.md`.
- [ ] **Caminho absoluto da worktree no pedido**, com "trabalhe somente nesta pasta". Com `--dangerously-skip-permissions`, o `agy` pode entrar em qualquer pasta; no teste, ele achou a cópia do projeto principal procurando o `plano.md` e codou lá.
- [ ] **Estado guardado** (`git status`) da worktree e do projeto principal, para conferir depois.
- [ ] **Uma chamada só.** Conferir base, caminhos e pedido antes de abrir o Gemini; cada recomeço é retrabalho dos dois lados.

## 6. Delegar sem duplicar

- Cada etapa tem um dono (Claude ou Gemini). Dois modelos na mesma tarefa só para comparar é desperdício.
- Ao delegar, citar só a tarefa (`implemente a tarefa T1 de .claude/planejamento/plano.md`). Não recopiar o plano no pedido.
- Subagente (ex.: `dev-reviewer`) tem contexto próprio e paga a própria leitura: mandar só o que ele precisa (caminho do projeto, tarefa).
- Correção de defeito volta para a **mesma** sessão do Gemini: ele não relê papel, regras e tarefa.
- Correção pequena e óbvia: direto na sessão principal, sem esteira e sem Gemini (o custo de preparar e conferir é maior que o da correção).

## 7. Cota do Gemini é separada

O Gemini gasta a cota do Google AI Pro, não a do Claude (barra "Gemini" no rodapé do Orca). Por isso a esteira manda para ele as etapas que leem e escrevem muito código (coder, test-designer).

# <Nome do projeto>

<Uma linha: o que o sistema faz e para quem.>

## Onde está cada coisa

| Arquivo | Para quê |
|---|---|
| `.claude/planejamento/CONTEXTO.md` | O sistema, a **situação atual** (etapas, pendências, última sessão, retomar daqui), as decisões e o banco |
| `.claude/planejamento/CHANGELOG.md` | Log, mais novo no topo |
| `.claude/planejamento/APLICAR.md` | Como aplicar e testar cada etapa (sem status) |
| `.claude/planejamento/requisitos/` | Pedidos de mudança recebidos |
| `.claude/planejamento/plano.md` | Tarefas em aberto para os agentes |
| `<pasta do código>` | <ex.: `src/` backend, `web/` frontend> |
| `<schema/migrations>` | <ex.: `db/migrations/` ou `prisma/schema.prisma`: ler para entender o banco> |

Cada informação fica num lugar só: status das etapas e pendências só no `CONTEXTO.md`; regras de
código só aqui; tarefas só no `plano.md`.

## Stack e comandos

- Linguagem/versão: <ex.: PHP 8.3 | Node 22 + TypeScript 5 | Python 3.12>
- Framework: <ex.: Laravel 11 | Next.js 15 | FastAPI>
- Banco/versão: <ex.: PostgreSQL 16 | MySQL 8 | SQLite>
- Pacotes: <npm | pnpm | composer | uv>
- Subir em dev: `<comando>`
- Lint/tipos: `<comando>`
- Testes: `<comando>` (um arquivo só: `<comando>`)
- Build: `<comando>`

## Ambiente de teste

`<http://localhost:3000>` (sobe com `<comando>`). Login de teste: ver `<seed/fixture/.env.example>`.
Nunca testar em produção.

## Regras de código (valem para toda tarefa e todo agente)

- Seguir o estilo do arquivo vizinho (nomes, comentários, idioma). Preservar o final de linha de cada arquivo.
- Usar o que o projeto já tem (helpers, services, componentes) antes de criar; dependência nova só com motivo.
- SQL sempre parametrizado (prepared statement/ORM), nunca dado de usuário concatenado. Texto que vai ao
  HTML sempre escapado (o template engine já faz; `dangerouslySetInnerHTML`/`{!! !!}`/`|safe` só com motivo).
- Segredo só em variável de ambiente; `.env` fora do git, `.env.example` atualizado quando entrar variável nova.
- Nunca editar arquivo gerado (`dist/`, `build/`, `vendor/`, `node_modules/`, código de codegen).
- Toda mudança de banco vai numa migration nova (nunca editar migration já aplicada; nunca mudar o banco à
  mão sem deixar a migration), mais uma seção no `APLICAR.md` e uma linha na tabela de etapas do `CONTEXTO.md`.
- <regras específicas do projeto: padrão de resposta da API, mensagens de erro exatas, permissões...>

## Armadilhas conhecidas

<Erros que já aconteceram e a causa, para o debugger checar primeiro. Ex.: "erro X -> causa Y".>

## Situação atual e changelog

- Início de sessão: ler a seção "Situação atual" do `CONTEXTO.md` antes de qualquer outra coisa.
- Atualizar a "Situação atual" e o `CHANGELOG.md` (só a sessão principal, não os agentes) ao terminar
  cada passo da esteira, ao mudar o status de uma etapa, quando o usuário resolver uma pendência e antes
  de encerrar a sessão. Na "Situação atual": data, tabela de etapas, pendências, "Última sessão" (reescrita
  do zero a cada passo, nunca acumular) e "Retomar daqui". No `CHANGELOG.md`, só linhas novas na
  entrada do dia: o que mudou, em quais arquivos, o que ficou para o humano.
- Recomendar ao usuário encerrar e abrir sessão nova (`/clear`) quando: a tarefa atual acabou e a
  próxima é de outra etapa; a conversa foi resumida automaticamente; ou for preciso reler arquivos
  grandes já lidos nesta sessão. Antes, deixar o "Retomar daqui" suficiente para a sessão nova continuar.

## Uso de tokens

O porquê de cada regra está no `BOAS-PRATICAS.md` do repositório da esteira.

- Uma sessão por etapa ou tarefa: ao terminar, deixar o "Retomar daqui" pronto e recomendar `/clear`.
- Sessão principal em Opus só para planejar e decidir; para executar a esteira, recomendar `/model` Sonnet.
- Ler trecho, não arquivo inteiro; saída de comando sempre limitada (`tail`, `head`, `grep`).
- Trabalho longo em segundo plano com um aviso no fim; nunca consultar o Gemini de tempos em tempos.
- Antes de chamar o Gemini, passar pelo checklist da skill `gemini` (push, `.claude/` commitada, caminho
  absoluto da worktree): cada recomeço gasta o contexto inteiro de novo.

## Esteira de trabalho

| Etapa | Quem | Executor | Modelo |
|---|---|---|---|
| 1. Planejar | sessão principal (plan mode / skill `planejar`) | Claude | Opus medium |
| 1b. Passar a proposta aprovada para tarefas no `plano.md` | agente `dev-task-writer` | Claude | Opus medium |
| 2. Codar (com lint/testes antes do reviewer) | skill `gemini`, papel coder | Gemini (`agy`) | Gemini 4 Argon (Medium) |
| 3. Revisar | agente `dev-reviewer` | Claude | Opus medium |
| 4. Montar a bateria de testes (seção da etapa no `APLICAR.md`) | skill `gemini`, papel test-designer | Gemini (`agy`) | Gemini 3.8 Flash (High) |
| 5. Aplicar em dev (migration, env, subir) | humano ou sessão principal | — | — |
| 6. Testar na tela | comando `/testar` (agente `dev-tester`), só o humano chama | Claude | Sonnet medium |
| 7. Falhou sem causa óbvia | agente `dev-debugger` | Claude | Opus high |

Regras:
- Funcionalidade nova ou mudança em mais de um arquivo/camada -> passa pela esteira, com tarefa em
  `.claude/planejamento/plano.md` (formato na seção "Formato de tarefa nova" dele). Correção pequena e
  óbvia -> direto, sem esteira.
- O `plano.md` é só a especificação do que vai ser implementado, nunca log nem fonte da verdade.
  A tarefa sai de lá depois que o reviewer aprova e o test-designer monta a bateria; tarefa desfeita ou substituída sai na hora. Na saída:
  aplicação e testes vão para a seção da etapa no `APLICAR.md`, o status para o `CONTEXTO.md` e o que foi
  feito para o `CHANGELOG.md`. Regra que vale para toda tarefa fica aqui, não no plano.
- Cada tarefa precisa ser autossuficiente (arquivos, tabelas, critérios de aceite, testes com resultado esperado).
- Ao delegar, cite só a tarefa: "implemente a tarefa T1 de .claude/planejamento/plano.md". Não recopie o plano.
- Cada tarefa do Gemini roda numa worktree nova do Orca (`tarefa-<T>`), com o `agy` num terminal visível
  do Orca. Lint, testes, reviewer e test-designer rodam nessa mesma worktree.
- Reviewer com defeito ou lint/testes falhando -> volta para a mesma sessão do Gemini no terminal do Orca, até 2
  rodadas cada; depois disso, o humano decide ou chama o `dev-debugger`.
- Commitar a `.claude/` no git: a worktree só recebe o que está commitado.
- Teste no navegador só em ambiente de desenvolvimento e só pelo `/testar`. Nenhum agente faz deploy nem commit;
  a sessão principal sugere o commit ao humano quando o reviewer aprova a tarefa.
- Quando um roteiro de teste estiver estável, o `dev-tester` pode ir para `model: haiku`.

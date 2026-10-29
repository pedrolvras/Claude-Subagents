---
name: "planejar"
description: "Monta ou melhora a proposta de uma mudança grande antes de codar. Parte do rascunho do usuário quando houver, confere tudo com o código, registra decisões numeradas com a origem e escreve um documento por camadas (dados, banco, telas, fluxo, testes, riscos, tarefas). Use quando o usuário pedir para planejar, montar ou melhorar uma proposta ou um planejamento, analisar um pedido de mudança, uma reformulação ou uma funcionalidade nova que mexe em mais de uma tela ou tabela. Não use para correção pequena e óbvia."
---

# Planejar uma mudança

Nada é codado durante o planejamento: o resultado é um documento de proposta e, depois de aprovado, as tarefas.

## Processo (nesta ordem)

1. **Contexto do projeto.** Ler o CLAUDE.md, a situação atual, o plano em aberto e o schema do banco. Não propor nada antes disso. Se o projeto tem uma esteira (plano, changelog, guia de aplicação), seguir as regras dela.

2. **Estudar o código que a mudança toca.** Ler cada tela, arquivo e tabela afetada. Anotar como é hoje: nomes reais de colunas, mensagens exibidas, regras de acesso, fluxo de status, o que cada tela grava.

3. **Partir do rascunho do usuário, se houver.** Ele é a fonte das intenções: não reescrever o que o usuário decidiu.
   - Cada ponto do rascunho vira uma decisão com origem "Usuário".
   - Conferir cada ponto com o código. Se algo conflita com o que existe ou com as regras do CLAUDE.md, apontar o conflito e propor uma saída, sem trocar a decisão sozinho.
   - Completar o que o rascunho não cobre: banco, telas afetadas, o que sai, testes, riscos, tarefas. O que vier daí tem origem "Proposta".
   - Perguntar só sobre lacunas e conflitos.

   **Sem rascunho:** resumir o pedido em itens numerados e perguntar o que muda o desenho antes de escrever. Se o pedido estiver vago, oferecer o roteiro de references/roteiro-rascunho.md para o usuário escrever o rascunho primeiro.

   **Perguntas:** até 5 por rodada, cada uma com a opção recomendada e o porquê. Não perguntar o que dá para descobrir no código.

4. **Escrever o documento** no arquivo de proposta (no projeto, onde o CLAUDE.md indicar; sem indicação, modificacoes.md na pasta de planejamento), com o modelo abaixo. Decisão ainda sem resposta fica marcada como **A confirmar**.

5. **Incorporar as respostas.** Atualizar a decisão e levar a mudança a todas as seções que ela afeta (banco, telas, testes, tarefas). Registrar no changelog do projeto, se houver. Repetir até não sobrar nada "a confirmar".

6. **Só depois de aprovado:** passar as tarefas para o arquivo de plano do projeto, cada uma autossuficiente (telas e eventos, tabelas, o que fazer, critérios de aceite, testes com resultado esperado exato).

## Modelo do documento

Para o nível de detalhe e o tom, ver o exemplo real em references/exemplo-proposta.md. Usar só como referência de formato: o conteúdo dele é de outro projeto.

```markdown
# Modificações — etapa NN: <título>

<Uma linha de situação: proposta ou aprovada, data, o que substitui, o que falta confirmar.>

## 1. Pedido de origem (resumo)
<Itens numerados, fiéis ao pedido ou ao rascunho.>

## 2. Decisões
| # | Decisão | Origem |
|---|---|---|
| D1 | ... | Usuário |
| D2 | ... **A confirmar:** ... | Usuário + proposta |

## 3. Dados e arquivos
<Onde cada coisa fica, formato, nomes, caminhos.>

## 4. Banco
| Tabela | Situação |
|---|---|
| ... | Igual / Sai coluna X / Entra coluna Y / Nova / Apagada |
<Regra de acesso nova por extenso, se mudar.>

## 5. Telas
### 5.1 <Tela>
- **Entra:** ...
- **Sai:** ...
- Mensagens exatas entre crases.

## 6. Fluxo
<Diagrama em texto com os status e as transições.>

## 7. Fora desta etapa
<O que fica de fora de propósito.>

## 8. Testes
<Ordem de aplicação.>
| Id | Passo | Resultado esperado |
|---|---|---|
| NN.a | ... | ... `mensagem exata` |

## 9. Riscos
<Risco, como ele aparece, como corrigir.>

## 10. Tarefas
| Tarefa | Conteúdo | Depende de |
|---|---|---|
| R1 Banco | seção 4 | — |
```

Seções que não se aplicam saem; camadas que o projeto tem a mais (API, relatório, agendador...) entram com o mesmo formato.

## Regras de escrita

- Mensagem ao usuário sempre exata, entre crases.
- Nomes de tabelas, colunas, telas e arquivos conferidos no código, nunca inventados.
- Cada decisão fica num lugar só (a tabela). As outras seções citam o número dela ("ver D4").
- Dizer o que sai, não só o que entra.
- Cada teste pode ser verificado por quem não conhece o contexto.
- Tarefas com dependência explícita; o banco vem primeiro.
- Se a mudança altera uma regra do CLAUDE.md, listar a atualização no documento (e na tarefa que a faz).
- Datas absolutas (22/10/2026), nunca "ontem" ou "semana que vem".
- Frases curtas, sem jargão desnecessário: o documento é lido por quem vai codar, revisar e testar.

## Antes de entregar, conferir

- **Falha:** cada operação nova diz o que acontece quando dá errado (arquivo não grava, configuração ausente, status mudou no meio) e a mensagem exata. Nada pode ficar meio gravado: dizer a ordem segura (ex.: grava temporário, atualiza o banco, só então renomeia).
- **Banco:** dizer como o banco existente chega no novo (migração, ou reinício com trava contra rodar por engano) e que o script pode rodar mais de uma vez.
- **Segurança:** quem pode cada ação, conferido no servidor e não só escondendo botão; validação de entrada; escape de todo texto que vai ao HTML; caminho de arquivo sem `..`; link de arquivo nunca direto ao disco.
- **Testes negativos:** além do caminho feliz, sem permissão, outra pessoa, prazo vencido, texto com `<b>` e aspas, script rodado duas vezes, troca de configuração.
- **Exemplos concretos:** um caminho real, um valor de busca e o resultado esperado, uma combinação de dados (ex.: níveis 8, 8, 6 e 12 e como ficam).
- **Arquivos por tela:** cada tela cita os arquivos, eventos e fontes que mudam, e o que sai deles.
- **Ações do humano:** o que não é código (criar ou apagar aplicação na IDE, rodar script, remover tarefa agendada, dar permissão no banco) fica escrito, com quem faz.
- **Documentação:** listar cada arquivo de documentação do projeto que muda (regras, contexto, guia de aplicação) e o quê.



## Ao fim de cada rodada

Resposta curta no chat (o detalhe fica no documento):
- o que mudou no documento;
- conflitos encontrados com o código, se houver;
- as perguntas que faltam, com a opção recomendada;
- o próximo passo.

=== references/roteiro-rascunho.md ===
# Roteiro para escrever o rascunho

Para o usuário preencher antes do planejamento, com as próprias palavras, em tópicos. Não precisa saber nada de banco nem de código: o Claude completa essa parte.

1. **O que muda para quem usa?** Ex.: "o documento não vai mais para o outro setor".
2. **Como fica o fluxo, passo a passo?** De onde começa até onde termina, com os status (Rascunho, Concluído...).
3. **Quem pode fazer o quê?** Por perfil: chefe, colaborador, leitor... O que cada um vê, edita, aprova, exclui.
4. **O que deixa de existir?** Ex.: "acabam os e-mails", "sai o filtro Setor".
5. **Regras e exceções.** Ex.: "sigiloso: só o chefe vê", "só pode excluir em Rascunho".
6. **Mensagens ou telas que você já imagina.** Pode ser um texto de aviso, um botão novo, um campo.
7. **Dúvidas que você ainda tem.** Pode escrever "não sei se X ou Y": vira pergunta para decidir.

  
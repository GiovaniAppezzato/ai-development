---
name: my-git-pr-writer
description: "Especialista em pull requests. Use quando: escrever o título e a descrição de um PR a partir dos commits e das alterações pendentes da branch atual, inclusive antes ou junto dos commits; abrir um pull request no GitHub quando o desenvolvedor pedir (como rascunho, se solicitado); atualizar a descrição de um PR já existente. Preenche o template de PR do projeto (por exemplo, .github/PULL_REQUEST_TEMPLATE.md). Sempre apresenta a proposta ao desenvolvedor, confirma a branch de destino e só abre ou altera o PR após aprovação explícita. Não use para mensagens de commit, criação de branches, changelogs ou release notes."
---

# /my-git-pr-writer: Pull Requests Claros

## Objetivo

Escrever títulos e descrições de pull requests claros e específicos, a partir das alterações da branch e do contexto do desenvolvimento, seguindo o template do projeto. Quando o desenvolvedor pedir, abrir o PR no GitHub, pronto para revisão ou como rascunho, se ele solicitar.

A skill analisa e propõe; o desenvolvedor decide. **Nenhum PR é aberto ou alterado sem aprovação explícita da proposta e confirmação da branch de destino.**

## Modo de Operação

Interprete a solicitação do desenvolvedor, explícita ou pelo contexto da chamada, para decidir o modo:

- **Escrever** (padrão): apresente o título e a descrição para aprovação e entregue o texto final, sem abrir nada no GitHub.
- **Abrir** (quando o desenvolvedor pede para criar ou abrir o PR): apresente a proposta e, após a aprovação e a confirmação do destino, abra o PR pronto para revisão. Abra como rascunho apenas se o desenvolvedor pedir.
- **Atualizar** (quando já existe um PR aberto para a branch): apresente a nova descrição e, após a aprovação, atualize o PR existente.

Se o desenvolvedor pedir para abrir o PR depois de aprovar um texto no modo **Escrever**, passe para o modo **Abrir** usando a proposta já aprovada, sem pedir nova aprovação do texto. O destino continua precisando de confirmação, se ainda não tiver sido confirmado.

Em todos os modos, a proposta é sempre apresentada primeiro, mesmo que ainda faltem commits. O que for necessário para abrir o PR é resolvido depois da aprovação.

## Fluxo de Trabalho

1. **Inspecionar a branch**
   - Considere o contexto da conversa anterior à execução da skill (veja **Contexto da Conversa**).
   - Identifique a branch atual (a origem do PR) com `git branch --show-current`.
   - Atualize as referências do remoto com `git fetch --prune origin`. Se falhar, siga com as referências locais e avise na resposta.
   - Defina a branch de destino sugerida, conforme **Branch de Destino**.
   - Liste os commits da branch com `git log --oneline origin/<destino>..HEAD`.
   - Leia as alterações commitadas com `git diff --stat origin/<destino>...HEAD` e `git diff origin/<destino>...HEAD`.
   - Leia também as alterações pendentes, que ainda não foram commitadas: `git status --short --untracked-files=all`, `git diff --cached`, `git diff` e o conteúdo dos arquivos novos.
   - Considere as alterações commitadas e as pendentes em conjunto: o PR deve descrever o resultado final do desenvolvimento, mesmo que parte dele ainda não tenha sido commitada.
   - Se a branch já tiver PR (`gh pr view --json number,url,isDraft,state,baseRefName`), use o modo **Atualizar**. Nesse caso, o destino é o do PR existente.

2. **Escrever a proposta**
   - Localize o template de PR do projeto e preencha todas as seções, conforme **Descrição do PR**.
   - Escreva o título conforme **Título do PR**.
   - Identifique o que ainda falta para abrir o PR, conforme **Requisitos para Abrir o PR**, para informar na proposta.

3. **Apresentar a proposta e aguardar aprovação**
   - Apresente a proposta no formato descrito em **Ciclo de Revisão**.
   - Não execute push nem comandos do `gh` que criem ou alterem PRs nesta etapa.
   - Permaneça no ciclo de revisão até o desenvolvedor aprovar, encerrar a skill ou mudar claramente de contexto.

4. **Abrir ou atualizar o PR** (somente após a aprovação e, no modo **Abrir**, após a confirmação do destino)
   - Antes de abrir, confirme que todos os **Requisitos para Abrir o PR** foram atendidos. Se faltar algo, siga o que está descrito naquela seção, mantendo a proposta aprovada.
   - Se a branch ainda não estiver no remoto ou tiver commits não enviados, faça push com `git push -u origin <branch>`.
   - Para abrir: `gh pr create --base <destino> --title "<título>" --body-file <arquivo>`. Acrescente `--draft` somente se o desenvolvedor pediu rascunho.
   - Para atualizar: `gh pr edit <número> --title "<título>" --body-file <arquivo>`.
   - Use exatamente o título, a descrição e o destino aprovados, sem alterá-los.
   - Grave a descrição em um arquivo temporário fora do repositório para usar no `--body-file`.

## Branch de Destino

O destino do PR é sempre decidido pelo desenvolvedor. A skill apenas sugere um destino para analisar as alterações e montar a proposta:

- Se o desenvolvedor informou o destino, use o informado.
- Se não informou, descubra a branch de desenvolvimento do repositório, ou seja, a branch em que o trabalho do dia a dia é integrado antes de seguir para produção. O nome muda entre repositórios (por exemplo, `dev`, `develop`, `development`, `staging`):
  - Liste as branches do remoto com `git branch -r` e procure nomes que indiquem desenvolvimento.
  - Se o `gh` estiver disponível, veja para onde os PRs recentes foram mesclados com `gh pr list --state merged --limit 20 --json baseRefName --jq '.[].baseRefName'`. A branch mais frequente é um bom indício.
  - Se houver mais de uma candidata, sugira a mais provável e cite as outras na proposta.
- Se não for possível identificar uma branch de desenvolvimento, sugira a branch padrão do remoto (`git symbolic-ref --short refs/remotes/origin/HEAD`).
- Se o desenvolvedor trocar o destino, analise de novo as alterações em relação ao novo destino e reapresente a proposta, pois os commits e o diff do PR mudam.

No modo **Abrir**, o PR só é aberto depois que o destino for confirmado explicitamente. A aprovação do texto não confirma o destino por si só, a menos que a resposta deixe isso claro (por exemplo, "pode abrir para a dev").

## Ciclo de Revisão

Antes de abrir ou atualizar o PR, apresente a proposta em dois blocos separados, um para o título e outro para a descrição, para que cada um possa ser copiado individualmente. Nunca junte título e descrição em um único bloco.

**Título:**

```text
<identificador da tarefa> - <título>
```

Sem identificador de tarefa, o título é apenas `<título>`.

**Descrição:**

```markdown
<template preenchido>
```

Logo após a descrição, informe em uma linha a origem e o destino considerados na análise. No modo **Abrir**, peça a confirmação do destino junto com o pedido de aprovação e informe se será feito push. Informe também se o PR será aberto pronto para revisão ou como rascunho. Por exemplo: "Ao aprovar, faço push da branch 123456 e abro o PR. Confirma a dev como destino?"

Ao final da proposta, inclua também:

- O que ainda falta para abrir o PR (por exemplo, alterações sem commit), de forma curta.
- Informações que não puderam ser deduzidas.
- Um pedido de aprovação curto, em tom de conversa normal, sem listar opções ou sugestões de resposta.

Regras do ciclo:

- O desenvolvedor pode pedir qualquer ajuste no título, na descrição ou na branch de destino.
- A cada ajuste, reapresente a proposta completa atualizada e aguarde nova resposta. Repita quantas vezes for necessário.
- A aprovação vale apenas para a última proposta apresentada. Se o conteúdo das alterações mudar durante o ciclo, analise de novo e atualize a proposta.
- Commitar alterações que já estavam pendentes e foram consideradas na proposta não muda o conteúdo do PR: nesse caso, a aprovação continua valendo.
- Se o desenvolvedor encerrar a skill ou mudar claramente de contexto, saia do ciclo sem abrir nada e informe que nenhum PR foi aberto ou alterado.

## Estados da Interação

Acompanhe em que estado a interação está, para não repetir aprovações nem refazer etapas já concluídas:

| Estado | Significado | Próximo passo |
| --- | --- | --- |
| **Em revisão** | A proposta foi apresentada e ainda não foi aprovada. | Aplicar ajustes e reapresentar, ou receber a aprovação. |
| **Aguardando destino** | O texto foi aprovado no modo **Abrir**, mas o destino não foi confirmado. | Perguntar de forma curta se pode abrir para o destino sugerido. |
| **Pronta para abrir** | Texto aprovado e destino confirmado (ou modo **Atualizar**). | Verificar os requisitos e abrir ou atualizar o PR. |

Regras dos estados:

- Uma aprovação já dada não é pedida de novo. Ao sair de **Aguardando destino**, use exatamente a última proposta aprovada.
- Se o desenvolvedor pedir ajuste no texto em qualquer estado, volte para **Em revisão**. Um destino já confirmado é mantido e mostrado na proposta reapresentada.
- Se o destino mudar, volte para **Em revisão**, pois a análise e o texto podem mudar.

## Requisitos para Abrir o PR

Estes requisitos são verificados somente depois da aprovação da proposta, nunca antes de apresentá-la:

- **Alterações pendentes:** se houver alterações consideradas na proposta que ainda não foram commitadas, informe que elas precisam ser commitadas antes e sugira a skill my-git-commiter. Mantenha a proposta aprovada e, depois dos commits, abra o PR sem pedir nova aprovação.
- **Origem igual ao destino:** se a branch atual for a própria branch de destino, não é possível abrir o PR. Informe e sugira criar uma branch com a skill my-git-branch; depois, siga com a proposta aprovada.
- **Sem commits:** se não houver commits em relação à branch de destino, não há o que abrir; informe o que falta.
- **GitHub CLI:** verifique o `gh` com `gh auth status`. Se não estiver instalado ou autenticado, informe que é preciso configurá-lo (por exemplo, com `gh auth login`) e entregue o texto aprovado e o destino para abertura manual.

## Interpretação das Respostas

Interpretar a resposta do desenvolvedor é responsabilidade da skill. Ele pode responder do jeito que quiser, e não existe uma lista fixa de frases aceitas: os exemplos desta skill são apenas ilustrativos.

- Entenda a intenção da resposta, considerando o estado atual da interação e a última pergunta feita: aprovação do texto, confirmação ou troca do destino, pedido de ajuste ou encerramento.
- Uma mesma resposta pode conter mais de uma intenção (por exemplo, aprovar o texto e confirmar o destino). Trate todas.
- Se tiver certeza da intenção, siga sem pedir confirmação adicional.
- Se ficar em dúvida se a proposta foi aprovada ou se o destino foi confirmado, **não abra nem altere o PR**. Responda com uma pergunta de confirmação direta, que possa ser respondida com sim ou não, como: "Posso abrir o PR para a dev?"

## Idioma

Escreva o título e a descrição do PR em português, seguindo o template do repositório. Use outro idioma (geralmente inglês) somente quando o desenvolvedor pedir explicitamente. Mantenha os títulos das seções do template como estão no arquivo; o idioma vale para o conteúdo escrito pela skill.

## Contexto da Conversa

Se houver conversa anterior à execução da skill (por exemplo, o desenvolvimento da feature que está no PR), use o que foi discutido, planejado e implementado junto com a análise das alterações para escrever uma descrição mais completa e precisa. Sem conversa anterior, siga normalmente apenas com a análise dos commits e das alterações pendentes.

## Título do PR

Use o formato:

```text
<identificador da tarefa> - <título>
```

O identificador só entra quando for encontrado; sem ele, o título é apenas o texto, sem prefixo nem separador.

Para montar o título:

- **Identificador e título conhecidos:** use o identificador e o título da tarefa, encontrados no contexto ou informados pelo desenvolvedor. Exemplo: `123456 - Ajuste no fluxo de cadastro`.
- **Só o identificador conhecido:** use o identificador seguido de um texto gerado a partir das alterações. Exemplo: `PROJ-123 - Corrige a validação de e-mail no cadastro`.
- **Sem identificador:** omita o identificador e o separador, deixando apenas o texto descritivo gerado a partir das alterações. Exemplo: `Corrige a validação de e-mail no cadastro`.
- Se o desenvolvedor informar a tarefa depois (e, opcionalmente, o título dela), atualize o título com o identificador e reapresente a proposta atualizada para aprovação.

Para identificar a tarefa, siga a seção **Identificação da Tarefa** da skill my-git-branch (`../my-git-branch/SKILL.md`, relativo à pasta desta skill), com dois complementos desta skill:

- Procure o identificador também no nome da branch de origem do PR.
- Nunca invente o título da tarefa. Sem o título real, use o texto gerado a partir das alterações.

Se não for possível ler a skill my-git-branch, procure o identificador no contexto da sessão e no nome da branch, sem inventar, e pergunte se houver mais de um possível.

Regras do texto gerado:

- Escreva no idioma definido em **Idioma**, começando com letra maiúscula.
- Resuma o objetivo do PR como um todo, não o último commit.
- Não use títulos vagos, como "Ajustes", "Correções" ou "Update".
- Não use `WIP` no título; para indicar que está em andamento, abra o PR como rascunho.

## Descrição do PR

Sempre siga o template de PR do projeto. Procure-o em `.github/PULL_REQUEST_TEMPLATE.md`, `.github/pull_request_template.md`, na pasta `.github/PULL_REQUEST_TEMPLATE/`, na raiz do repositório ou em `docs/`. Não redefina o template nesta skill: preencha todas as seções do arquivo, substituindo os textos de exemplo pelo conteúdo real.

Se o projeto não tiver template, use as seções das orientações abaixo, na mesma ordem, e avise na proposta que o repositório não tem template de PR.

Orientações por tipo de seção (use a que corresponder a cada seção do template, mesmo que o nome seja diferente):

- **Problema ou necessidade** (por exemplo, "Descrição" ou "Problema"): o problema ou a necessidade que motivou o PR, do ponto de vista do negócio ou do usuário.
- **Solução** (por exemplo, "Solução" ou "Mudanças realizadas"): como foi resolvido, em alto nível. Destaque decisões importantes e não liste cada arquivo alterado.
- **Como testar:** pré-requisitos e passos concretos para o testador reproduzir e validar, incluindo telas, rotas ou comandos envolvidos.
- **Outras notas** (por exemplo, "Outras notas" ou "Informações adicionais"): riscos, impactos em outras áreas e breaking changes. Se não houver notas, responda "Não".
- **Alterações de banco** (por exemplo, "Banco de dados" ou "SQL"): registra as alterações de banco que precisam ser aplicadas.
  - **Projeto com sistema de migrations** (Laravel, Symfony ou PHP puro com migrations): liste os caminhos dos arquivos de migration adicionados no PR, por exemplo `database/migrations/2026_01_01_000000_create_examples_table.php`. Se não houver migrations novas, responda "Não".
  - **Projeto sem migrations**, em que os comandos são executados manualmente no banco: inclua os comandos SQL necessários. Se não houver, responda "Não".
- **Seções sem orientação aqui** (por exemplo, checklist, tarefa vinculada ou screenshots): preencha com o que puder ser deduzido das alterações e do contexto. Em checklists, marque apenas os itens que foram de fato cumpridos.

Regras da descrição:

- Seja direto ao ponto. O PR será revisado por uma pessoa, então a descrição deve ser rápida de ler: clara, curta e sem informações avulsas. Descrições longas ou complicadas atrapalham a revisão.
- Seja específico: prefira "Adiciona filtro por data na listagem de pedidos" a "Alterações na listagem".
- Nunca omita breaking changes.
- Não invente passos de teste, tarefas ou resultados. Se algo não puder ser deduzido, deixe o campo sinalizado e avise na proposta.
- Escreva no idioma definido em **Idioma**.
- Nunca inclua rodapé ou qualquer atribuição de autoria gerada por ferramenta (por exemplo, "Generated with Claude Code").

## Regras de Segurança

- Nunca abra, altere, feche ou faça merge de PRs sem aprovação explícita da proposta atual.
- Nunca abra um PR sem a confirmação explícita da branch de destino.
- Abra o PR como rascunho (`--draft`) somente quando o desenvolvedor pedir.
- Faça push apenas da branch de origem do PR e somente após a aprovação. Nunca faça push para a branch de destino nem use `--force` ou `--force-with-lease`.
- Nunca faça commit, rebase, reset ou merge.
- Não inclua segredos, tokens, senhas ou dados pessoais de clientes ou usuários (por exemplo, CPF, e-mail, telefone ou dados de saúde) na descrição do PR.

## Resposta Final

Responda de forma curta, informando:

- **Escrever:** o título e a descrição finais, prontos para copiar.
- **Abrir:** o link do PR, as branches de origem e destino, se o PR foi aberto como rascunho e se foi feito push.
- **Atualizar:** o link do PR e o que mudou.

Se o ciclo for encerrado sem aprovação, informe apenas que nenhum PR foi aberto ou alterado.

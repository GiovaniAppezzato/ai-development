---
name: my-git-commiter
description: "Especialista em commits Git. Use quando: criar um ou mais commits locais a partir da worktree atual; agrupar arquivos alterados por contexto coerente e unidade de história; fazer stage apenas dos arquivos ou hunks que pertencem ao mesmo grupo; escrever mensagens Conventional Commits estruturadas e explicativas, com tipo, escopo opcional, marcador opcional de breaking change, assunto e corpo. Não use para descrições de PR, changelogs, release notes ou para publicar/fazer push de branches. Sempre apresenta a proposta de commits ao desenvolvedor e só commita após aprovação explícita."
---

# /my-git-commiter: Commits Git Coerentes

## Objetivo

Criar commits Git locais pequenos, coerentes e fáceis de revisar. Agrupar as alterações por contexto e unidade de história, fazer stage apenas do que pertence a cada unidade e escrever mensagens Conventional Commits explicativas.

A skill analisa e propõe; o desenvolvedor decide. A análise e a organização dos commits são automáticas, mas o que será commitado, como será dividido e quais mensagens serão usadas é sempre decisão do desenvolvedor. **Nenhum commit é executado sem aprovação explícita.**

## Fluxo de Trabalho

1. **Inspecionar o estado do repositório**
   - Considere o contexto da conversa anterior à execução da skill (veja **Contexto da Conversa**).
   - Identifique a branch atual com `git branch --show-current`, mas não faça nenhum alerta sobre ela antes de apresentar a proposta.
   - Execute `git status --short --untracked-files=all`.
   - Inspecione as alterações em stage separadamente das que não estão em stage com `git diff --cached --stat` e `git diff --stat`.
   - Leia os diffs antes de commitar: `git diff --cached` para o conteúdo em stage e `git diff -- <path>` para arquivos fora do stage.
   - Preserve as alterações do usuário que não têm relação com o commit solicitado.

2. **Identificar as unidades de história**
   - Agrupe os arquivos por resultado visível ao usuário, correção de bug, refatoração, alteração de documentação, cobertura de testes ou alteração de infraestrutura.
   - Prefira um commit por unidade de história coerente.
   - Mantenha metadados gerados junto da alteração de código que os exige.
   - Mantenha os testes junto da implementação que eles validam, a menos que a alteração exclusiva de testes seja uma unidade própria.
   - Não combine módulos não relacionados, correções não relacionadas ou alterações apenas de formatação com alterações de comportamento.

3. **Planejar os commits**
   - Se houver apenas uma unidade óbvia, proponha um commit.
   - Se houver várias unidades independentes, proponha vários commits em ordem de dependência.
   - Se um arquivo contiver alterações não relacionadas que não possam ir para o stage inteiro com segurança, indique isso na proposta e pergunte se deve ser feito stage parcial.
   - Se o agrupamento for ambíguo, escolha a opção mais coerente e destaque a dúvida na proposta com uma pergunta objetiva.

4. **Apresentar a proposta e aguardar aprovação**
   - Apresente a proposta completa no formato descrito na seção **Ciclo de Revisão**.
   - Se a branch atual for protegida, inclua o alerta ao final da proposta, conforme a seção **Branches Protegidas**.
   - Não execute `git add`, `git commit` nem qualquer comando que altere o stage ou o histórico nesta etapa.
   - Permaneça no ciclo de revisão até o desenvolvedor aprovar, encerrar a skill ou mudar claramente de contexto.

5. **Fazer stage com precisão** (somente após a aprovação e, em branch protegida, após a definição do local do commit)
   - Faça stage exatamente dos arquivos e hunks da proposta aprovada.
   - Use caminhos explícitos: `git add -- path/to/file`.
   - Evite `git add .` e stage amplo, a menos que todos os arquivos alterados pertençam ao mesmo commit.
   - Verifique `git diff --cached --name-status` antes de cada commit.
   - Não faça stage de segredos, configurações locais, logs, artefatos de build, arquivos de editor ou arquivos gerados não relacionados.

6. **Validar antes de commitar**
   - Execute a validação leve pertinente aos arquivos alterados, quando disponível.
   - No mínimo, inspecione `git diff --cached --check`.
   - Se a validação não puder ser executada, explique o motivo na resposta final.
   - Se a validação falhar, não commite: informe o problema ao desenvolvedor e volte ao ciclo de revisão.

7. **Commitar**
   - Use `git commit` com exatamente as mensagens aprovadas, sem alterá-las.
   - Após cada commit, confirme com `git status --short` e `git log -1 --oneline`.
   - Continue até que todos os commits aprovados estejam feitos ou bloqueados.

## Ciclo de Revisão

Antes de qualquer commit, apresente a proposta neste formato:

```text
Commit 1 de N
Arquivos:
- caminho/do/arquivo1
- caminho/do/arquivo2 (stage parcial: apenas os trechos de <descrição>)

Mensagem:
tipo(escopo): assunto conciso

Contexto:
- ...

Alterações:
- ...
```

Quando a proposta tiver apenas um commit, omita o cabeçalho `Commit 1 de N` e a lista de `Arquivos`, apresentando somente a mensagem. A exceção é quando houver stage parcial: nesse caso, mantenha a lista de arquivos para indicar quais trechos entram no commit.

Ao final da proposta, inclua também:

- Arquivos alterados que ficarão de fora, com o motivo.
- Dúvidas pendentes (agrupamento ambíguo, stage parcial, alterações já em stage).
- Um pedido de aprovação curto, em tom de conversa normal, sem listar opções ou sugestões de resposta.

Regras do ciclo:

- O desenvolvedor pode pedir qualquer ajuste: juntar ou separar commits, mover arquivos entre commits, excluir arquivos, mudar tipo, escopo, assunto ou corpo.
- A cada ajuste, reapresente a proposta completa atualizada e aguarde nova resposta. Repita quantas vezes for necessário.
- Só considere aprovado com uma confirmação explícita da proposta atual, interpretada conforme a seção **Interpretação das Respostas**.
- A aprovação vale apenas para a última proposta apresentada. Se algo mudar depois dela, apresente de novo.
- Se o desenvolvedor aprovar com um pequeno ajuste (por exemplo, "pode commitar, mas troca o escopo para `api`"), aplique o ajuste e mostre a versão final antes de commitar.
- Se a worktree mudar durante o ciclo, inspecione de novo e atualize a proposta.
- Se o desenvolvedor encerrar a skill ou mudar claramente de contexto, saia do ciclo sem commitar nada e informe que nenhum commit foi feito.

## Interpretação das Respostas

Interpretar a resposta do desenvolvedor é responsabilidade da skill. Ele pode responder do jeito que quiser, e não existe uma lista fixa de frases aceitas: os exemplos desta skill são apenas ilustrativos.

- Entenda a intenção da resposta, considerando o estado atual da interação e a última pergunta feita: se é uma aprovação, um pedido de ajuste, uma definição de local do commit, um pedido para aguardar ou um encerramento.
- Uma mesma resposta pode conter mais de uma intenção (por exemplo, aprovar a proposta e definir a branch). Trate todas.
- Se tiver certeza da intenção, siga sem pedir confirmação adicional.
- Se ficar em dúvida se a proposta foi aprovada ou se o local do commit foi definido, **não commite**. Responda com uma pergunta de confirmação direta, que possa ser respondida com sim ou não, como: "Posso commitar a proposta acima?" ou "Confirma o commit diretamente na develop?"
- Na dúvida, prefira perguntar a commitar: uma pergunta a mais custa menos que um commit indevido.

## Estados da Interação

Acompanhe em que estado a interação está, para não repetir aprovações nem refazer etapas já concluídas:

| Estado                    | Significado                                                                 | Próximo passo                                                    |
| ------------------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Em revisão**            | A proposta foi apresentada e ainda não foi aprovada.                        | Aplicar ajustes e reapresentar, ou receber a aprovação.          |
| **Aguardando branch**     | A proposta foi aprovada, mas o local do commit em branch protegida não foi definido. | Fazer a pergunta curta sobre o local do commit.                  |
| **Aguardando branch manual** | A proposta foi aprovada e o desenvolvedor vai criar a branch manualmente. | Aguardar a confirmação do desenvolvedor para continuar.          |
| **Pronta para commitar**  | Proposta aprovada e local definido (ou branch não protegida).               | Fazer stage, validar e commitar.                                 |

Regras dos estados:

- Uma aprovação já dada não é pedida de novo. Ao sair de **Aguardando branch** ou **Aguardando branch manual**, use exatamente a última proposta aprovada, sem gerar novas mensagens.
- Se o desenvolvedor pedir ajuste na proposta em qualquer estado, volte para **Em revisão**. A decisão sobre o local do commit, se já tiver sido dada, é mantida e mostrada na proposta reapresentada.
- Se a worktree mudar depois da aprovação, volte para **Em revisão** e reapresente a proposta atualizada.

## Contexto da Conversa

Se houver conversa anterior à execução da skill (por exemplo, o desenvolvimento da feature que está sendo commitada), use o que foi discutido, planejado e implementado junto com a análise das alterações para sugerir mensagens mais completas e precisas. Sem conversa anterior, siga normalmente apenas com a análise das alterações.

## Branches Protegidas

As branches principais do repositório exigem atenção especial. Cada repositório tem o seu padrão (por exemplo, `master`, `main`, `dev`, `develop`, `staging`, `prd`); não assuma uma lista fixa. Identifique as do repositório atual:

1. Descubra a branch padrão do remoto com `git symbolic-ref --short refs/remotes/origin/HEAD`.
2. Liste as branches do remoto com `git branch -r` e considere principais as que seguem nomes de ambiente ou integração, como as do exemplo acima.
3. Se o `gh` estiver disponível e autenticado, confirme as branches protegidas com `gh api 'repos/{owner}/{repo}/branches?protected=true' --jq '.[].name'`. Se falhar, siga só com os passos anteriores.

Se não for possível identificar as branches principais, trate a branch atual como protegida quando o nome dela seguir um desses padrões.

A verificação acontece **somente depois** de apresentar a proposta, nunca antes: independentemente da branch, a skill sempre analisa as alterações, considera o contexto e apresenta a proposta primeiro.

Se a branch atual for protegida, use a skill my-git-branch no modo **Sugerir** para obter um nome de branch e termine a proposta com um alerta curto, em tom de conversa normal, sem listar opções ou sugestões de resposta. Por exemplo:

```text
⚠️ Você está na branch develop. Posso fazer os commits diretamente nela ou prefere criar a branch 123456?
```

Como interpretar a resposta do desenvolvedor (os exemplos são ilustrativos; aplique as regras de **Interpretação das Respostas**):

- **Aprovação com local** (por exemplo, "Pode mandar o commit direto na develop"): a proposta está aprovada e o local está definido. Siga para **Pronta para commitar**.
- **Aprovação sem local** (por exemplo, "Está boa a mensagem, pode fazer"): a proposta está aprovada, mas o local não. Vá para **Aguardando branch** e faça uma pergunta curta e objetiva, como: "A mensagem foi aprovada. Deseja realizar o commit diretamente na develop ou prefere criar a branch 123456?"
- **Commit na branch atual** (por exemplo, "Faça na develop"): commite na branch atual usando a última proposta aprovada.
- **Criar branch pela skill** (por exemplo, "Cria a branch sugerida" ou "Crie a branch feat/minha-alteracao e faça nela"): crie a branch sugerida ou a informada, mantendo as alterações da worktree, e commite nela usando a última proposta aprovada.
- **Criação manual** (por exemplo, "Espera, vou criar a branch"): vá para **Aguardando branch manual**, mantenha a proposta aprovada e aguarde. Quando o desenvolvedor confirmar, verifique a branch atual com `git branch --show-current`. Se ainda for protegida, pergunte de novo; caso contrário, commite.
- Se o desenvolvedor indicar uma branch existente diferente da atual, pergunte antes de trocar de branch.
- A decisão sobre o local vale para todos os commits da proposta.

Criação de branch:

- Use a skill my-git-branch no modo **Criar**, com o nome sugerido ou o informado pelo desenvolvedor. A branch é criada direto, sem nova confirmação, e o fluxo segue com a proposta já aprovada.
- Se a skill my-git-branch não estiver disponível, crie com `git switch -c <nome>`, validando antes o nome (`git check-ref-format --branch <nome>`) e se a branch já existe. Se houver problema, informe e pergunte como prosseguir.
- Nunca faça push da branch criada.

Nas demais branches, a aprovação explícita da proposta já é suficiente para commitar, sem confirmação adicional de local.

## Conventional Commits

Use este formato de assunto:

```text
tipo(escopo)!: assunto conciso no imperativo
```

Tipos comuns:

| Tipo       | Usar para                                                       |
| ---------- | --------------------------------------------------------------- |
| `feat`     | Nova funcionalidade para o usuário ou para o produto            |
| `fix`      | Correção de bug ou de comportamento                             |
| `refactor` | Reestruturação interna sem alteração de comportamento           |
| `docs`     | Alterações apenas de documentação                               |
| `test`     | Alterações apenas de testes                                     |
| `style`    | Alterações apenas de formatação                                 |
| `chore`    | Manutenção, ferramentas, metadados gerados ou tarefas gerais    |
| `build`    | Alterações no sistema de build ou em dependências               |
| `ci`       | Alterações em workflows de CI/CD                                |
| `perf`     | Melhoria de desempenho                                          |
| `revert`   | Reversão de um commit anterior                                  |

Regras do assunto:

- Use o tipo em letras minúsculas.
- Use um escopo curto quando ele deixar a mensagem mais clara: `feat(auth): ...`.
- Use `!` apenas para breaking changes.
- Mantenha o assunto conciso, de preferência com menos de 72 caracteres.
- Comece a descrição com um verbo que diga o que o commit faz: no presente em português (por exemplo: `corrige`, `adiciona`, `remove`, `ajusta`) ou no imperativo em inglês (por exemplo: `fix`, `add`, `remove`, `update`).
- Não termine o assunto com ponto final.

## Idioma das Mensagens

Siga o idioma que o repositório já usa nas mensagens de commit:

1. Leia os assuntos dos commits recentes com `git log -n 30 --no-merges --format=%s`, de preferência na branch padrão do remoto (`git log -n 30 --no-merges --format=%s origin/<branch-padrão>`).
2. Desconsidere commits automáticos (por exemplo, de bots de dependências ou de release) e avalie apenas a descrição, sem o tipo e o escopo.
3. Se a grande maioria estiver em inglês, escreva em inglês. Se estiver em português, escreva em português.
4. Se não houver commits suficientes, se o idioma não puder ser identificado ou se o histórico for misturado, escreva em português.
5. Se o desenvolvedor pedir outro idioma, use o idioma pedido.

Na proposta, informe em uma linha curta o idioma usado e o motivo (por exemplo, "Mensagens em inglês, seguindo o histórico do repositório.").

O idioma vale para a descrição do assunto e todo o corpo, incluindo os rótulos `Contexto:` e `Alterações:` (em inglês, `Context:` e `Changes:`). Permanecem como estão os elementos do padrão Conventional Commits (o tipo, como `feat` e `fix`, e o rodapé `BREAKING CHANGE:`), além de nomes de código, arquivos e comandos.

Exemplo em português:

```text
fix(cadastro): corrige validação do e-mail

Contexto:
- O e-mail era rejeitado quando tinha letras maiúsculas.

Alterações:
- Normaliza o e-mail antes da validação.
```

Exemplo em inglês:

```text
fix(signup): fix email validation

Context:
- The email was rejected when it had uppercase letters.

Changes:
- Normalize the email before validation.
```

## Corpo da Mensagem de Commit

Use um corpo quando o commit não for autoexplicativo. Prefira esta estrutura:

```text
tipo(escopo): assunto conciso

Contexto:
- Por que esta alteração existe.

Alterações:
- O que mudou nesta unidade de história.
- O que mudou nos testes/docs/configurações relacionados.
```

Regras do corpo:

- Explique o porquê e o quê, não cada linha de código.
- Mencione o impacto para o usuário quando for relevante.
- Mencione riscos ou notas de migração quando for relevante.
- Inclua um rodapé `BREAKING CHANGE:` para breaking changes.
- Não inclua referências de ticket na mensagem; o ticket já é identificado pelo nome da branch.
- Não invente resultados de validação.
- Nunca inclua um trailer `Co-Authored-By` nem qualquer atribuição de autoria na mensagem de commit.

## Regras de Agrupamento de Commits

- **Mesmo commit:** a implementação e seus arquivos complementares diretos de template/CSS/JS/PHP.
- **Mesmo commit:** a migration e as alterações de repository/service que dependem dela, se a funcionalidade não funcionar sem ambas.
- **Mesmo commit:** a documentação que explica exatamente a alteração sendo commitada.
- **Commit separado:** módulos não relacionados ou histórias de usuário distintas.
- **Commit separado:** formatação ampla, separada de alterações de comportamento.
- **Commit separado:** limpeza exclusiva de testes não relacionada à implementação.
- **Commit separado:** atualizações de lockfile/dependências geradas quando não fizerem parte da história principal.

## Regras de Segurança

- Nunca execute `git add` ou `git commit` sem a aprovação explícita da proposta atual.
- Nunca commite em uma branch protegida sem a confirmação explícita do local do commit.
- Nunca crie ou troque de branch sem que o desenvolvedor tenha solicitado.
- Nunca commite arquivos que não foram inspecionados.
- Nunca reescreva o histórico nem faça amend, reset, rebase, squash ou push, a menos que isso seja solicitado explicitamente.
- Nunca remova ou reverta alterações do usuário para facilitar o commit.
- Nunca esconda uma validação que falhou; inclua-a na resposta final.
- Nunca crie um commit a partir de uma área de stage suja sem verificar o que já está nela.
- Se houver alterações já em stage sem relação com o commit solicitado, pergunte antes de tirá-las do stage ou commitá-las.

## Resposta Final

Após commitar, informe:

- A branch em que os commits foram feitos (e se ela foi criada pela skill).
- O hash e o assunto de cada commit.
- Os arquivos ou a unidade de história incluídos em cada commit.
- A validação executada e o resultado.
- As alterações que continuam sem commit, se houver.

Se o ciclo for encerrado sem aprovação, informe apenas que nenhum commit foi feito.

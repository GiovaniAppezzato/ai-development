---
name: my-git-update
description: "Especialista em atualização de repositórios Git. Use quando: verificar se um repositório, um conjunto de repositórios ou um workspace inteiro (por exemplo, b2b2c, anne ou nutripass) têm atualizações no remoto; atualizar branches locais informadas pelo nome ou pelo ambiente (desenvolvimento ou produção), inclusive branches diferentes da atual; sincronizar o ambiente no início do dia. Analisa os repositórios e apresenta uma tabela com a situação de cada um antes de qualquer alteração, e só atualiza, sempre por fast-forward, após aprovação explícita. Não use para commits, push, merge, rebase, resolução de conflitos, criação de branches de trabalho ou PRs."
---

# /my-git-update: Atualização de Repositórios

## Objetivo

Verificar rapidamente quais repositórios têm atualizações no remoto e sincronizar as branches locais, sem precisar entrar em cada repositório. O cenário principal é o início do dia: uma solicitação curta, como "atualize o ambiente de desenvolvimento da Anne", gera uma tabela com a situação de cada repositório e as ações propostas.

A skill analisa e propõe; o desenvolvedor decide. A complexidade fica na lógica da skill, não na interação: ela identifica sozinha o que for possível e pergunta apenas quando realmente precisa. **Nenhuma branch é atualizada, criada ou trocada sem aprovação explícita.**

## Modos de Operação

Interprete a solicitação do desenvolvedor para decidir o modo:

- **Verificar** (quando o pedido é só olhar, por exemplo "veja se tem atualização nos repositórios da Anne" ou "minha branch está atualizada?"): faça a análise e apresente a tabela, sem coluna de ação e sem pergunta de execução.
- **Atualizar** (quando o pedido é atualizar, por exemplo "atualize a develop desse projeto"): faça a análise, apresente a proposta e aguarde a aprovação.

Se o desenvolvedor pedir para atualizar depois de uma verificação, passe para o modo **Atualizar** reaproveitando a análise já feita e apresente a proposta com a coluna de ação.

## Fluxo de Trabalho

1. **Identificar o escopo:** os repositórios envolvidos, conforme **Identificação do Escopo**.
2. **Identificar as branches alvo:** a branch de cada repositório que será considerada, conforme **Branch Alvo**.
3. **Analisar:** consultar o remoto e o estado local de cada repositório, conforme **Análise**.
4. **Apresentar:** a tabela no formato de **Proposta**.
5. **Ajustar:** aplicar os ajustes pedidos e reapresentar, conforme **Ciclo de Revisão**.
6. **Confirmar:** aguardar a aprovação explícita da última proposta e, se ficar pendente, a decisão sobre a troca de branch (veja **Troca de Branch**).
7. **Executar:** somente as ações aprovadas, conforme **Execução**.
8. **Informar o resultado**, conforme **Resposta Final**.

## Identificação do Escopo

A organização dos projetos muda de um workspace para outro. Não assuma uma estrutura fixa nem caminhos conhecidos de antemão: descubra a partir do que existe no computador.

Ordem de prioridade:

1. **Repositórios informados** (por exemplo, "só o app-core e o dashboard"): considere apenas esses.
2. **Workspace informado** (por exemplo, "Nutripass" ou "repositórios da Anne"): considere os repositórios Git do workspace.
3. **Nenhum alvo informado:** use o repositório do diretório atual (`git rev-parse --show-toplevel`).
4. **Diretório atual é a pasta de um workspace**, e não de um repositório: sugira esse workspace como escopo e peça confirmação na própria proposta se houver mais de uma leitura possível.
5. **Nada identificado:** pergunte qual é o alvo.

Como encontrar um workspace pelo nome:

- Procure a partir do diretório atual, dos seus ancestrais e dos diretórios de trabalho adicionais da sessão. Se não encontrar, amplie para a pasta pessoal, com profundidade limitada.
- Considere arquivos `.code-workspace` cujo nome contenha o workspace (por exemplo, `anne-workspace.code-workspace` ou `b2b2c.code-workspace`) e pastas com o nome do workspace, sem diferenciar maiúsculas e minúsculas.
- Se existir um `.code-workspace`, use as pastas listadas em `folders` (caminhos relativos ao arquivo) que forem repositórios Git. Os nomes amigáveis (`name`) também servem para casar pedidos como "o dashboard" ou "Anne RH".
- Sem `.code-workspace`, procure repositórios dentro da pasta do workspace com `find <pasta> -maxdepth 3 -name .git -not -path '*/node_modules/*' -not -path '*/vendor/*'`. O `.git` pode ser pasta ou arquivo (worktrees e submódulos).
- Workspaces podem ser aninhados (um workspace pode conter outros). Considere todos os repositórios dentro do escopo pedido.
- Se houver mais de uma pasta candidata ou um nome de repositório casar com mais de um repositório (por exemplo, "dashboard" em dois workspaces), use o workspace do contexto para desempatar; se continuar ambíguo, pergunte.

## Branch Alvo

Defina a branch alvo de cada repositório a partir do pedido:

- **Branch informada pelo nome** (por exemplo, "atualize a develop"): use a branch informada. Se ela não existir no repositório, marque como impedimento; nunca troque por outra parecida sem perguntar.
- **Ambiente informado** (por exemplo, "ambiente de desenvolvimento" ou "branches de produção"): identifique a branch do ambiente em cada repositório, conforme **Branch por Ambiente**.
- **Nada informado:** use a branch atual de cada repositório.
- **Mais de uma branch ou ambiente** (por exemplo, "a main e a develop da Anne"): cada combinação de repositório e branch vira uma linha da tabela.

### Branch por Ambiente

Cada repositório tem a sua convenção. Nomes comuns de desenvolvimento são `develop`, `development`, `dev`, `staging` e `sandbox`; de produção, `main`, `master`, `production`, `prod` e `prd`. Eles são apenas pistas: nunca escolha a primeira branch cujo nome combine.

Reúna evidências antes de decidir:

- Branches existentes no remoto (`git branch -r`) e a branch padrão (`git symbolic-ref --short refs/remotes/origin/HEAD`). A branch padrão nem sempre é a de produção.
- Para onde os PRs recentes foram mesclados, se o `gh` estiver disponível: `gh pr list --state merged --limit 30 --json baseRefName,headRefName`. A branch que recebe as features costuma ser a de desenvolvimento; a que recebe PRs vindos dela costuma ser a de produção.
- Gatilhos de branch e nomes de ambiente nos workflows de CI/CD (`.github/workflows/`, `bitbucket-pipelines.yml`, `.gitlab-ci.yml` ou similares).
- Documentação e orientações do projeto: `README`, `CLAUDE.md`, `AGENTS.md` e `CONTRIBUTING`.
- Atividade recente de cada candidata (`git log -1 --format=%cr <remoto>/<branch>`). Uma branch sem commits há meses provavelmente foi abandonada.

Quando duas candidatas parecerem do mesmo ambiente (por exemplo, `dev` e `develop` no mesmo repositório) ou representarem ambientes diferentes (por exemplo, `develop` e `staging`), use as evidências para entender o papel de cada uma.

Se as evidências forem suficientes, use a branch identificada sem perguntar. Se continuar ambíguo, mostre `?` na coluna da branch alvo e pergunte, em uma linha por repositório, qual branch considerar. É preferível perguntar a atualizar a branch errada.

## Análise

A análise só consulta. Ela pode atualizar as referências remotas, mas nunca altera branches locais, arquivos da worktree ou o histórico.

Para cada repositório:

1. Atualize as referências de todos os remotos com `GIT_TERMINAL_PROMPT=0 git -C <repo> fetch --all --prune`. Em vários repositórios, execute os fetches juntos, em um único comando, para manter a análise rápida. Se o fetch falhar (sem rede, sem acesso), siga com as referências locais e marque a linha com o aviso.
2. Identifique a branch atual com `git -C <repo> branch --show-current`. Saída vazia indica HEAD destacado.
3. Identifique o remoto da branch alvo:
   - Se a branch local existir e tiver upstream, use o upstream (`git -C <repo> rev-parse --abbrev-ref <branch>@{upstream}`), qualquer que seja o remoto.
   - Se não tiver upstream ou não existir localmente, procure a branch nos remotos (`git -C <repo> branch -r --list '*/<branch>'`). Se ela estiver em mais de um remoto, prefira `origin` e mencione o remoto escolhido nas observações.
4. Compare a branch local com a remota: `git -C <repo> rev-list --left-right --count <branch>...<remoto>/<branch>`. O primeiro número são os commits locais não enviados; o segundo, os commits remotos disponíveis.
5. Verifique as alterações locais com `git -C <repo> status --porcelain`.
6. Verifique operações em andamento: a existência de `MERGE_HEAD`, `CHERRY_PICK_HEAD`, `REVERT_HEAD`, `rebase-merge` ou `rebase-apply` no diretório indicado por `git -C <repo> rev-parse --git-path <nome>`.

### Situação

| Situação | Quando | Ação no modo Atualizar |
| --- | --- | --- |
| **Atualizada** | Local e remota no mesmo commit. | Nenhuma. |
| **Atrasada** (`↓N`) | Só há commits remotos. | Atualizar por fast-forward. |
| **Adiantada** (`↑N`) | Só há commits locais não enviados. | Nenhuma; avisar que há commits não enviados. |
| **Divergente** (`↑N ↓N`) | Há commits locais e remotos. | Nenhuma; precisa de merge ou rebase manual. |
| **Sem branch local** | A branch só existe no remoto. | Criar a branch local rastreando a remota, se aprovado. |
| **Impedimento** | Branch inexistente no remoto, branch sem upstream, HEAD destacado, merge ou rebase em andamento, falha no fetch. | Nenhuma; explicar o motivo. |

Alterações locais só interferem quando a branch alvo é a atual ou quando há troca de branch. Nesses casos, mostre `alterações locais` junto da situação. Quando a atualização não toca na worktree, não as mencione.

## Proposta

Comece direto pela proposta, sem narrar o que foi consultado:

```markdown
**Anne · desenvolvimento** · 5 repositórios · 3 com atualização · 1 requer atenção

| # | Repositório | Atual | Alvo | Situação | Ação |
| --- | --- | --- | --- | --- | --- |
| 1 | app-core-wl | feat/login | dev | ↓5 | atualizar sem trocar de branch |
| 2 | app-core-dashboard-wl | develop | develop | ↓2 · alterações locais | atualizar (fast-forward) |
| 3 | nutripass-rh-wl | develop | develop | atualizada | - |
| 4 | webdiet-care-backoffice | dev | dev | ↑1 ↓3 divergente | nenhuma |
| 5 | anne-external-links | main | develop | ↓4 | atualizar sem trocar de branch |

- **2:** as alterações locais não impedem o fast-forward, a não ser que toquem nos mesmos arquivos (nesse caso o Git recusa e nada é alterado).
- **4:** tem 1 commit local não enviado e diverge do remoto; precisa de merge ou rebase manual.

Posso atualizar o 1, o 2 e o 5?
```

Regras da proposta:

- A primeira linha traz o escopo (workspace ou repositório e, se houver, o ambiente) e o resumo: quantos repositórios foram analisados, quantos têm atualização e quantos exigem atenção.
- Numere as linhas para o desenvolvedor poder se referir a elas ("só o 1 e o 3", "no 4 usa a main").
- Ordene na ordem do `.code-workspace` ou alfabética, mantendo juntas as linhas do mesmo repositório.
- Mostre o remoto na coluna **Alvo** apenas quando não for `origin` (por exemplo, `upstream/develop`).
- Observações só para o que exige atenção ou decisão: riscos, impedimentos, remoto diferente do esperado, branch do ambiente escolhida por evidência pouco óbvia ou ambiguidades. Uma linha por item, referenciando o número.
- No modo **Verificar**, omita a coluna **Ação** e a pergunta final.
- No modo **Atualizar**, termine com um pedido de aprovação curto, em tom de conversa normal, citando as linhas que serão executadas, sem listar opções ou sugestões de resposta. Se não houver nada a atualizar, informe em uma linha e encerre.
- Se a troca de branch já tiver sido pedida (por exemplo, "e me deixa na develop"), mostre na coluna **Ação** (por exemplo, `atualizar e trocar para develop`).

## Ciclo de Revisão

- O desenvolvedor pode ajustar o escopo ou as branches a qualquer momento: incluir ou remover repositórios, escolher linhas pelo número, trocar a branch alvo de um repositório ou de todos, ou pedir a troca de branch.
- A cada ajuste, analise de novo apenas os repositórios e as branches afetados (fazendo fetch dos repositórios novos no escopo) e reapresente a proposta completa atualizada.
- A aprovação vale apenas para a última proposta apresentada. Se o escopo ou as branches mudarem, a aprovação anterior deixa de valer.
- Se o desenvolvedor aprovar com um pequeno ajuste (por exemplo, "pode atualizar, mas tira o 5"), aplique o ajuste e siga sem nova aprovação apenas quando o ajuste só reduzir o que será feito. Se ele acrescentar algo, reapresente.
- Se o desenvolvedor encerrar a skill ou mudar claramente de contexto, saia do ciclo sem executar nada e informe que nenhum repositório foi alterado.

## Troca de Branch

Atualizar uma branch diferente da atual não exige troca: a branch é atualizada por fast-forward sem tocar na worktree, e o desenvolvedor continua onde está.

- Se o desenvolvedor já disse se quer trocar ou permanecer, siga o que foi dito.
- Se isso ficou pendente para alguma linha aprovada em que a branch alvo é diferente da atual, depois da aprovação pergunte de forma curta se deve manter a branch atual ou trocar para a branch alvo. Por exemplo: "Aprovado. Depois de atualizar, mantenho o app-core-wl e o anne-external-links nas branches atuais ou troco para as branches alvo?"
- A resposta pode valer para todos ou ser diferente por repositório (por exemplo, "troca só no 1").
- A troca só é feita sem alterações em arquivos rastreados (`git -C <repo> status --porcelain --untracked-files=no` vazio). Se houver alterações, avise na pergunta que aquele repositório vai permanecer na branch atual; nunca use stash nem descarte nada para trocar.
- A troca acontece depois da atualização da branch, e a skill não volta para a branch original depois.

## Estados da Interação

| Estado | Significado | Próximo passo |
| --- | --- | --- |
| **Em revisão** | A proposta foi apresentada e ainda não foi aprovada. | Aplicar ajustes e reapresentar, ou receber a aprovação. |
| **Aguardando branch alvo** | Há linhas com `?` na branch alvo. | Perguntar qual branch considerar nesses repositórios. |
| **Aguardando troca** | A proposta foi aprovada, mas ficou pendente se deve trocar de branch. | Fazer a pergunta curta sobre manter ou trocar. |
| **Pronta para executar** | Proposta aprovada e troca definida (ou não aplicável). | Executar exatamente as ações aprovadas. |

- Uma aprovação já dada não é pedida de novo. Ao sair de **Aguardando troca**, use exatamente a última proposta aprovada.
- Linhas com `?` não podem ser aprovadas. Se o desenvolvedor aprovar o resto, execute só as linhas definidas.
- Se o desenvolvedor pedir ajuste em qualquer estado, volte para **Em revisão**. Uma decisão de troca já dada é mantida e mostrada na proposta reapresentada.

## Interpretação das Respostas

Interpretar a resposta do desenvolvedor é responsabilidade da skill. Ele pode responder do jeito que quiser, e não existe uma lista fixa de frases aceitas: os exemplos desta skill são apenas ilustrativos.

- Entenda a intenção considerando o estado atual e a última pergunta feita: aprovação, seleção de linhas, troca de branch alvo, decisão de troca, ajuste de escopo ou encerramento.
- Uma mesma resposta pode ter mais de uma intenção (por exemplo, "pode atualizar todos e me deixa na develop"). Trate todas.
- Se a intenção estiver clara, siga sem pedir confirmação adicional.
- Se ficar em dúvida se a proposta foi aprovada ou se deve trocar de branch, **não execute**. Faça uma pergunta direta, que possa ser respondida com sim ou não, como "Posso atualizar o 1, o 2 e o 5?"

## Execução

Imediatamente antes de executar cada repositório, confira de novo a branch atual e as alterações locais. Se algo mudou desde a análise e afeta a ação, não execute aquela linha e informe no resultado.

Use os comandos de acordo com a ação aprovada, sempre com o remoto e a branch identificados na análise:

- **Branch alvo é a atual:** `git -C <repo> merge --ff-only <remoto>/<branch>`. Se não for possível avançar por fast-forward, ou se alterações locais forem sobrescritas, o Git recusa e nada é alterado.
- **Branch alvo é outra branch local:** `git -C <repo> fetch . refs/remotes/<remoto>/<branch>:refs/heads/<branch>`. Atualiza a branch por fast-forward sem trocar de branch nem tocar na worktree; se não for fast-forward, o Git rejeita (`non-fast-forward`) e nada é alterado.
- **Criar branch local:** `git -C <repo> branch --track <branch> <remoto>/<branch>`. Cria a branch rastreando a remota, sem trocar de branch.
- **Trocar de branch** (somente se decidido): `git -C <repo> switch <branch>`, depois da atualização.

Depois de cada ação, confirme com `git -C <repo> rev-list --left-right --count <branch>...<remoto>/<branch>` (deve resultar em `0 0`) e, se houve troca, com `git -C <repo> branch --show-current`.

- Execute os repositórios de forma independente. A falha em um não interrompe os outros nem dispara qualquer tentativa de correção.
- Se uma ação for recusada, registre a mensagem do Git e siga para o próximo repositório.

## Regras de Segurança

- Nunca execute `git pull`, `merge --ff-only`, `fetch` em branch local, criação ou troca de branch sem a aprovação explícita da proposta atual.
- Nunca use merge que crie commit, rebase, cherry-pick ou resolução automática de conflitos. Se a atualização exigir isso, interrompa aquela linha e peça orientação.
- Nunca execute `git reset`, `git clean`, `git checkout -- <arquivo>`, `git restore`, `git stash` ou `git branch -f`, nem use `+` ou `--force` em refspecs.
- Nunca faça commit nem push.
- Nunca descarte alterações locais nem presuma que uma branch local pode ser sobrescrita.
- Nunca crie uma branch que não exista no remoto, nem troque a branch alvo por outra parecida sem perguntar.
- Nunca altere repositórios ou branches fora do escopo aprovado.
- Nunca esconda uma falha; informe-a no resultado com o motivo.

## Resposta Final

Responda de forma curta, com o resultado por linha executada:

```markdown
**Resultado** · 2 atualizados · 1 não atualizado

| # | Repositório | Branch | Resultado |
| --- | --- | --- | --- |
| 1 | app-core-wl | dev | atualizada (+5), mantida em feat/login |
| 2 | app-core-dashboard-wl | develop | atualizada (+2) |
| 5 | anne-external-links | develop | não atualizada: arquivos locais seriam sobrescritos |
```

- Inclua apenas as linhas aprovadas e mostre, para cada uma, se houve troca de branch.
- Abaixo da tabela, uma linha para cada falha com a mensagem do Git resumida, e uma linha citando os repositórios que exigiam atenção e ficaram de fora (por exemplo, divergentes).
- No modo **Verificar**, a tabela da proposta já é a resposta final.

Se o ciclo for encerrado sem aprovação, informe apenas que nenhum repositório foi alterado.

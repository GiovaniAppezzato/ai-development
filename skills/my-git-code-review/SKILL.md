---
name: my-git-code-review
description: "Revisão de pull requests do GitHub para o revisor. Use quando receber o link ou o número de um PR para revisar ou para acompanhar uma review já feita: entende o que foi implementado, aponta bugs, erros básicos e violações de regras de negócio, analisa as respostas e correções do autor, e só envia ao PR (request changes, respostas, resolução de conversas ou aprovação) depois da confirmação do revisor. Não use para escrever PRs, commitar ou corrigir código."
---

# /my-git-code-review: Revisão de Pull Requests

## Objetivo

Ajudar o revisor a entender um pull request e encontrar problemas reais antes do merge: bugs, erros básicos e violações de regras de negócio. A skill lê o PR, entende o que foi implementado, analisa as alterações no contexto do projeto e prepara apontamentos curtos para o revisor.

Depois da review, a skill acompanha a conversa: lê as respostas e as correções do autor, analisa se cada apontamento foi resolvido ou se a contestação procede, e ajuda o revisor a concluir cada ponto. Quando o revisor pedir, ela também aprova o PR, com ou sem comentário.

A skill é consultiva: analisa e propõe; o revisor decide. **Nada é enviado ao PR (review, resposta, resolução de conversa ou aprovação) sem a confirmação explícita do revisor.**

## Modos de Operação

Interprete a solicitação do revisor e o estado do PR para decidir o modo:

- **Revisar** (padrão quando o revisor ainda não fez review no PR): siga o **Fluxo de Revisão**.
- **Acompanhar** (quando o revisor já fez review no PR, ou pede para ver as respostas e correções do autor): siga o **Acompanhamento da Review**.
- **Aprovar** (somente quando o revisor pedir): siga a **Aprovação**.

Para saber se o revisor já fez review, identifique o usuário com `gh api user --jq .login` e procure reviews dele no PR com `gh api repos/<dono>/<repositório>/pulls/<número>/reviews`. Se ficar em dúvida entre revisar de novo e acompanhar, pergunte.

## Fluxo de Revisão

1. **Ler o PR**
   - Aceite o link ou o número do PR. Extraia do link o dono, o repositório e o número; se o PR for de outro repositório, passe `--repo <dono>/<repositório>` para o `gh`.
   - Leia os dados do PR com `gh pr view <pr> --json number,title,body,author,state,isDraft,url,baseRefName,headRefName,headRefOid,commits,files`.
   - Se o `gh` não estiver instalado ou autenticado (`gh auth status`), informe que é preciso configurá-lo (por exemplo, com `gh auth login`) e pare: a revisão depende da leitura do PR.
   - Se o PR estiver fechado ou já tiver sido mesclado, informe e pergunte se deve continuar.

2. **Entender o que foi implementado**
   - Entenda a tarefa pelo título, pela descrição e pelos commits, conforme **Contexto da Tarefa**.
   - Escreva um resumo de duas ou três linhas do que o PR faz, em linguagem simples, como você contaria a um colega. Ele abre o relatório e confirma para o revisor que a intenção foi entendida.

3. **Carregar as regras de revisão**
   - Leia todos os arquivos da pasta `rules/` desta skill (veja **Regras de Revisão**).
   - Leia as orientações do projeto: `AGENTS.md`, `CLAUDE.md` e os arquivos de regras que se aplicam às áreas alteradas (por exemplo, `.agents/rules/`). As regras críticas declaradas pelo projeto têm a maior prioridade.

4. **Ler as alterações no contexto**
   - Baixe o código do PR sem mexer na worktree: `git fetch origin pull/<número>/head`. Use o `headRefOid` como o commit do PR.
   - Leia o diff com `git diff origin/<base>...<headRefOid>` e o conteúdo dos arquivos com `git show <headRefOid>:<caminho>`. Se o fetch falhar, use `gh pr diff <pr>`.
   - Nunca faça checkout, troca de branch, stash ou alteração de arquivos na worktree.
   - Para cada arquivo alterado, leia o necessário para avaliar a alteração: o código ao redor, quem chama, os testes relacionados e contratos como rotas, payloads, eventos e migrations.

5. **Analisar**
   - **Bugs:** lógica errada, acesso a valores nulos, valores vazios ou de limite, tratamento de erros, condições de corrida e regressões em rotas, payloads, eventos ou alterações de banco.
   - **Erros básicos:** código de debug esquecido, código morto, imports ou nomes errados, erros de copiar e colar e falta de testes para comportamento alterado quando o projeto exige.
   - **Regras de negócio:** a implementação corresponde à descrição e às regras do projeto, incluindo isolamento de tenant, autorização e validações.
   - **Regras de revisão:** todas as regras da pasta `rules/`.

6. **Verificar cada apontamento antes de reportar**
   - Releia o código e confirme a condição que dispara o problema.
   - Confira se o caso já não é tratado em outro lugar (quem chama, middlewares, validators, testes).
   - Aponte apenas problemas nas linhas alteradas pelo PR ou causados diretamente por elas.
   - Se o apontamento ainda depender de uma informação que você não tem, reporte como dúvida e não como problema. Se for só um palpite, descarte.

7. **Apresentar o relatório e aguardar a curadoria**, conforme **Relatório** e **Ciclo de Curadoria**.

8. **Publicar a review** somente após a aprovação, conforme **Publicação**.

## Severidade

| Nível | Significado |
| --- | --- |
| **Bloqueante** | Quebra comportamento, perde ou vaza dados, expõe dados de outro tenant, abre falha de segurança ou viola uma regra crítica do projeto. |
| **Importante** | Bug real ou violação de regra com impacto limitado; deve ser corrigido antes do merge. |
| **Sugestão** | Melhora clareza ou manutenção; o código está correto hoje. |
| **Dúvida** | Algo que parece errado, mas depende de um contexto que o revisor ou o autor podem confirmar. |

## Relatório

Escreva o relatório em português, a não ser que o revisor peça outro idioma. Ele deve ser limpo e fácil de escanear, mesmo com muitos apontamentos.

```markdown
**PR #45** · <título do PR>
<resumo do que o PR faz, em até duas frases curtas>

**3 apontamentos:** 1 importante · 2 sugestões

---

**1. Importante** · `caminho/do/arquivo.php:42`
> <comentário exatamente como será publicado>

**2. Sugestão** · `caminho/do/arquivo.js:18`
> <comentário exatamente como será publicado>

---

<pergunta curta de curadoria>
```

- Comece direto pelo relatório, sem narrar o que foi lido ou verificado.
- Não use títulos grandes. A primeira linha traz só o número e o título do PR.
- O resumo tem no máximo duas frases curtas.
- Na contagem, mostre apenas os níveis que têm apontamentos.
- Cada apontamento tem uma linha com número, severidade e local, e o comentário em citação (`>`), para separar o que será publicado do resto.
- Numere os apontamentos para o revisor poder se referir a eles ("descarta o 2", "edita o 3") e ordene por severidade, de Bloqueante a Dúvida.
- Não inclua o que está correto, pontos positivos ou explicações extras. Se o revisor quiser detalhes de um apontamento, ele pergunta.
- Se não houver apontamentos, responda só com a primeira linha, o resumo e "Nenhum apontamento.". Nunca sugira aprovar o PR.
- Termine com uma pergunta curta, sem listar opções (por exemplo, "Algum ajuste antes de publicar?").

## Escrita dos Comentários

Os comentários são lidos pelo autor do PR e devem soar como um colega revisando o código: naturais, diretos e com o contexto necessário para corrigir sem precisar perguntar.

- Diga onde está o problema, por que ele importa e o que fazer. Na maioria dos casos, isso cabe em uma ou duas frases; use três só quando o problema realmente pedir.
- Ajuste o tamanho ao problema: um import sem uso pede uma frase; um bug com efeito colateral pode pedir três.
- Varie a forma. Não comece todos os comentários do mesmo jeito nem siga uma estrutura fixa de frase. Uma pergunta também funciona quando o ponto depende da intenção do autor.
- Escreva como se fala com um colega: frases simples, sem jargão formal ("foi identificado", "recomenda-se", "visando").
- Seja específico. Cite a variável, o método ou o caso envolvido.
- Pode nomear o conceito quando ele ajudar a entender (por exemplo, "número mágico"), mas não cite arquivos de regras.
- Sem saudação, sem elogio antes da crítica e sem frases de encerramento.
- Sem blocos longos, listas ou títulos dentro de um comentário. Não explique o conceito em detalhes nem liste vários benefícios ou alternativas.
- Inclua uma sugestão curta de código apenas quando ela deixar a correção mais clara do que palavras.
- Não use o caractere em dash.

Exemplos (ilustrativos; o que deve se repetir é o contexto mínimo, não a forma):

- **Evite (exaustivo):** "Foi identificado que o método `calculate()` utiliza o valor numérico 30 diretamente. Números mágicos são valores literais sem significado explícito, que dificultam a leitura, aumentam o risco de inconsistência quando o valor precisa ser alterado em vários lugares e prejudicam a manutenção. Existem algumas alternativas: criar uma constante de classe, criar um enum caso existam outros prazos relacionados, ou mover o valor para um arquivo de configuração. Recomenda-se avaliar qual opção se encaixa melhor no projeto e aplicar a mesma abordagem nos demais pontos do código."
- **Evite (sem contexto):** "Usa constante aqui."
- **Prefira (número mágico):** "Esse 30 em `calculate()` é o prazo de expiração? Melhor virar uma constante, tipo `EXPIRATION_DAYS`, porque se o prazo mudar fica fácil esquecer algum lugar."
- **Prefira (bug):** "Se o usuário não tiver plano ativo, `$plan` chega nulo aqui e o `->getPrice()` quebra. Dá para tratar antes com um retorno antecipado."
- **Prefira (erro básico):** "O `Logger` não está sendo usado, pode remover o import."

## Ciclo de Curadoria

- O revisor pode aprovar todos os apontamentos, aprovar alguns pelo número, editar um texto, mudar a severidade, descartar apontamentos ou incluir os seus.
- A cada alteração, reapresente a lista atualizada, no mesmo formato, e aguarde nova resposta.
- A aprovação vale apenas para a última lista apresentada.
- Antes de enviar, confirme em uma linha curta o que vai acontecer: quais ações e em quantas conversas. Por exemplo: "Publico 3 apontamentos como request changes no PR #45?" ou "Resolvo 2 conversas e respondo a do item 3?"
- Se o revisor encerrar a skill ou mudar claramente de contexto, saia do ciclo sem enviar nada e informe que nada foi enviado.

## Estados da Interação

| Estado | Significado | Próximo passo |
| --- | --- | --- |
| **Em curadoria** | O relatório foi apresentado e o revisor está fazendo a curadoria. | Aplicar as alterações e reapresentar, ou receber a aprovação. |
| **Aguardando confirmação** | As ações foram aprovadas, mas o envio não foi confirmado. | Fazer a pergunta curta de confirmação. |
| **Pronta para enviar** | Ações aprovadas e envio confirmado. | Executar exatamente as ações aprovadas. |

- Uma aprovação já dada não é pedida de novo. Uma resposta que aprova e confirma ao mesmo tempo (por exemplo, "pode publicar todos") vai direto para **Pronta para enviar**.
- Qualquer alteração nos apontamentos ou nas ações volta para **Em curadoria**.

## Interpretação das Respostas

O revisor pode responder do jeito que quiser; os exemplos desta skill são apenas ilustrativos.

- Entenda a intenção considerando o estado atual e a última pergunta feita: aprovação, edição, descarte, resposta, resolução, aprovação do PR, confirmação ou encerramento.
- Uma mesma resposta pode ter mais de uma intenção (por exemplo, "descarta o 2 e pode publicar o resto" ou "resolve o 1 e responde o 2"). Trate todas.
- Se a intenção estiver clara, siga sem pedir confirmação adicional.
- Se ficar em dúvida se o envio foi aprovado, **não envie**. Faça uma pergunta direta, que possa ser respondida com sim ou não.

## Publicação

- Antes de publicar, confira de novo o commit do PR com `gh pr view <pr> --json headRefOid`. Se ele mudou depois da análise, informe e pergunte se deve revisar os novos commits antes.
- Publique uma única review com todos os apontamentos aprovados:
  - Apontamentos ligados a uma linha do diff viram comentários inline.
  - Apontamentos sem linha no diff vão para o corpo da review.
  - O corpo da review é uma frase curta e natural (por exemplo, "Deixei alguns pontos para ajuste."), seguida dos apontamentos sem linha, se houver.
- Grave o payload em um arquivo temporário fora do repositório e envie com `gh api repos/<dono>/<repositório>/pulls/<número>/reviews --method POST --input <arquivo>`:

```json
{
  "commit_id": "<headRefOid>",
  "event": "REQUEST_CHANGES",
  "body": "<corpo da review>",
  "comments": [
    { "path": "<caminho do arquivo>", "line": 42, "side": "RIGHT", "body": "<comentário>" }
  ]
}
```

- Use exatamente os textos aprovados, sem os rótulos de severidade nem a numeração.
- O tipo da review é `REQUEST_CHANGES`. Use `COMMENT` somente quando o revisor pedir, ou quando os apontamentos aprovados forem apenas sugestões ou dúvidas e o revisor concordar em comentar em vez de pedir alterações.
- O GitHub não permite pedir alterações no próprio PR. Se o revisor for o autor do PR, informe e ofereça publicar como `COMMENT`.
- Se a publicação falhar por causa de um comentário (por exemplo, uma linha fora do diff), mova esse apontamento para o corpo da review e avise o revisor.

## Acompanhamento da Review

Use este fluxo quando o revisor já fez review no PR. O objetivo é concluir cada apontamento: confirmar o que foi corrigido, discutir o que foi contestado e decidir o que falta.

1. **Ler o que aconteceu desde a review**
   - Identifique a última review do revisor e o commit dela (`commit_id`) em `gh api repos/<dono>/<repositório>/pulls/<número>/reviews`.
   - Leia as conversas da review, com o estado de cada uma e todas as respostas:

```bash
gh api graphql -F owner=<dono> -F name=<repositório> -F number=<número> -f query='
query($owner: String!, $name: String!, $number: Int!) {
  repository(owner: $owner, name: $name) {
    pullRequest(number: $number) {
      reviewThreads(first: 100) {
        nodes {
          id isResolved isOutdated path line
          comments(first: 50) { nodes { databaseId author { login } body createdAt } }
        }
      }
    }
  }
}'
```

   - Leia também os comentários gerais do PR (`gh api repos/<dono>/<repositório>/issues/<número>/comments`), porque o autor pode responder fora das conversas.
   - Baixe o código atual do PR, como no passo 4 do **Fluxo de Revisão**, e leia o que mudou desde a review com `git diff <commit_id>..<headRefOid>`.
   - Foque nas conversas abertas pelo revisor. Conversas de outras pessoas servem só como contexto.

2. **Analisar cada conversa**
   - **Corrigido:** o código mudou e o problema não existe mais. Confirme no código, não apenas pela resposta do autor.
   - **Parcialmente corrigido:** parte do problema continua. Diga o que falta.
   - **Contestado:** o autor discordou ou trouxe um contexto que a review não considerou. Avalie o argumento contra o código, as regras do projeto e a descrição do PR, e diga se ele procede e por quê. Um contexto válido que a review não tinha é motivo para retirar o apontamento, não para insistir.
   - **Adiado:** o autor disse que vai tratar depois (por exemplo, em outro PR). Diga se o adiamento é aceitável para a severidade do apontamento.
   - **Pendente:** sem resposta e sem alteração no código.
   - Para cada conversa, sugira uma ação: resolver, responder, responder e resolver, ou manter aberta. Quando sugerir uma resposta, escreva o texto seguindo **Escrita dos Comentários**.

3. **Revisar os commits novos**
   - Analise as alterações feitas depois da review com as mesmas regras do **Fluxo de Revisão**. Uma correção pode introduzir um problema novo.
   - Os problemas novos entram como apontamentos, no mesmo formato do **Relatório**.

4. **Apresentar o acompanhamento e aguardar a curadoria**

```markdown
**PR #45** · acompanhamento
2 commits novos · 3 respostas do autor

**4 conversas:** 1 contestada · 1 pendente · 2 corrigidas

---

**1. Contestado** · `caminho/do/arquivo.php:42` · responder e resolver
Autor: <resumo de uma linha da resposta>
Análise: <uma linha dizendo se o argumento procede e por quê>
> <resposta exatamente como será enviada>

**2. Pendente** · `caminho/do/arquivo.php:80` · manter aberta

**3. Corrigido** · `caminho/do/arquivo.js:18` · resolver

**4. Corrigido** · `caminho/do/arquivo.js:31` · resolver

---

**Novos apontamentos:** nenhum

<pergunta curta de curadoria>
```

   - Siga as mesmas regras de limpeza do **Relatório**.
   - Cada item tem uma linha com número, situação, local e ação sugerida. Itens que pedem decisão (contestados, parciais e adiados) ganham também uma linha com a resposta do autor, uma com a análise e, quando houver, a resposta a enviar em citação. Itens corrigidos e pendentes ficam em uma linha só.
   - Ordene primeiro o que precisa de decisão do revisor, depois o que está pendente e por último o que foi corrigido.
   - Os novos apontamentos seguem o formato do **Relatório**.
   - O revisor faz a curadoria como no **Ciclo de Curadoria**: aceita ou troca a ação de cada item, edita as respostas e aprova ou descarta os novos apontamentos.
   - Se todas as conversas estiverem concluídas, informe em uma linha. Não sugira aprovar o PR; isso só acontece se o revisor pedir.

5. **Enviar as ações aprovadas**, conforme **Envio das Ações**.

## Envio das Ações

Antes de enviar, confira de novo o commit do PR com `gh pr view <pr> --json headRefOid`. Se ele mudou depois da análise, informe e pergunte se deve analisar os novos commits antes.

- **Responder uma conversa:** `gh api repos/<dono>/<repositório>/pulls/<número>/comments/<id-do-primeiro-comentário>/replies --method POST -f body='<resposta>'`, usando o `databaseId` do primeiro comentário da conversa.
- **Resolver uma conversa:** `gh api graphql -F id=<id-da-conversa> -f query='mutation($id: ID!) { resolveReviewThread(input: {threadId: $id}) { thread { isResolved } } }'`.
- **Responder e resolver:** envie a resposta primeiro e depois resolva a conversa.
- **Novos apontamentos:** publique uma nova review, conforme **Publicação**.
- Use exatamente os textos aprovados e execute apenas as ações aprovadas.
- Se uma ação falhar, continue com as outras e informe ao revisor qual falhou e por quê.

## Aprovação

Aprove o PR somente quando o revisor pedir explicitamente, com ou sem comentário.

- Antes de pedir a confirmação, verifique se ainda há conversas abertas do revisor ou apontamentos pendentes e avise em uma linha. Por exemplo: "Ainda tem 1 conversa aberta (item 2). Aprovo mesmo assim?"
- **Sem comentário:** `gh pr review <pr> --approve`.
- **Com comentário:** escreva o texto seguindo **Escrita dos Comentários**, apresente para aprovação e envie com `gh pr review <pr> --approve --body-file <arquivo>`, usando um arquivo temporário fora do repositório.
- O GitHub não permite aprovar o próprio PR. Se o revisor for o autor, informe.
- Aprovar não faz merge. Nunca faça merge, mesmo depois de aprovar.

## Contexto da Tarefa

Para entender a tarefa, use o título e a descrição do PR e o histórico de commits. Juntos, eles dizem o que deveria ter sido feito; o código diz o que foi feito de fato.

- **Título:** o objetivo principal do PR. Se tiver um identificador de tarefa (por exemplo, `PROJ-123`), use-o apenas como referência no relatório.
- **Descrição:** a fonte principal do comportamento esperado. Extraia dela:
  - o problema ou a necessidade que motivou o PR;
  - o comportamento esperado depois da alteração, incluindo regras de negócio, validações e casos especiais citados;
  - o escopo: o que entra no PR e o que foi deixado de fora de propósito;
  - os passos de teste, que mostram os cenários que o autor considera importantes;
  - riscos, breaking changes e alterações de banco declarados.
- **Histórico de commits:** mostra como a implementação evoluiu. Use as mensagens para entender a intenção de cada etapa e as decisões tomadas pelo caminho.
  - Avalie sempre o estado final do PR. Um problema introduzido em um commit e corrigido em outro não é apontamento.
  - Commits de correção, reversão ou ajuste depois de review indicam pontos sensíveis, que merecem uma leitura mais atenta.
- **Contexto passado pelo revisor:** se o revisor colar ou descrever algo na sessão (por exemplo, o texto da tarefa ou uma regra combinada com o time), use como complemento.

Como usar o contexto na análise:

- Compare o que foi pedido com o que foi implementado: comportamento faltando, comportamento a mais fora do escopo e casos citados na descrição que o código não trata.
- Uma divergência entre a descrição e o código é um apontamento. Se não der para saber qual dos dois está certo, reporte como dúvida.
- Não trate as mensagens de commit nem a descrição como prova de que algo funciona: confirme sempre no código.
- Se a descrição for vaga ou estiver vazia, entenda a tarefa pelos commits e pelo código, diga isso em uma linha no resumo do relatório e reporte como dúvida os pontos de regra de negócio que dependem do comportamento esperado.

## Regras de Revisão

A pasta `rules/` (relativa à pasta desta skill) concentra as boas práticas de revisão e as diretrizes da plataforma. Ela cresce com o tempo sem precisar alterar este arquivo.

- Leia todos os arquivos de `rules/` no início de cada revisão.
- Cada regra descreve o que procurar, as exceções e como sugerir a correção. Aplique-a apenas ao código alterado pelo PR.
- Se uma regra conflitar com as orientações do projeto (`AGENTS.md`), siga as orientações do projeto e mencione o conflito ao revisor.

Para adicionar uma regra, crie um arquivo Markdown em `rules/` seguindo a estrutura das regras existentes: frontmatter (`title`, `severity`, `tags`), a regra, por que importa, como identificar, exceções e como sugerir a correção, com um exemplo curto. O `severity` do frontmatter é o nível padrão; quando ele variar conforme o caso, inclua uma seção **Severidade** explicando quando usar cada nível.

## Regras de Segurança

- Nunca envie nada ao PR (review, resposta, resolução de conversa ou aprovação) sem a aprovação explícita das ações atuais e a confirmação do envio.
- Nunca aprove o PR sem um pedido explícito do revisor, e nunca sugira aprovar por iniciativa própria.
- Nunca resolva uma conversa só porque o autor respondeu: confirme no código ou com o revisor.
- Nunca faça merge, feche ou edite o PR, o título ou a descrição dele.
- Nunca faça commit, push, checkout, stash ou altere arquivos na worktree.
- Nunca publique apontamentos descartados ou não aprovados, e nunca altere um texto aprovado.
- Nunca invente regras de negócio ou resultados de teste. Na dúvida, pergunte.
- Nunca inclua segredos, tokens, senhas ou dados pessoais (por exemplo, CPF, e-mail, telefone ou dados de saúde) nos comentários.
- Nunca inclua atribuição de ferramenta na review ou nos comentários.

## Resposta Final

- **Só relatório:** o relatório com a curadoria, pronto para o revisor.
- **Publicado:** o link da review, o tipo da review e quantos comentários foram publicados inline e no corpo.
- **Acompanhamento enviado:** quantas conversas foram respondidas, resolvidas ou mantidas abertas, e o link da nova review, se houver.
- **Aprovado:** o link do PR e se a aprovação teve comentário.

Se o ciclo for encerrado sem aprovação, informe apenas que nada foi enviado.

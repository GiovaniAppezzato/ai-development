# Preferências pessoais

Preferências de trabalho do usuário, válidas em qualquer projeto, somadas às instruções de cada repo.

- Idioma: este arquivo e as memórias pessoais ficam em português; docs nos repos seguem o idioma do projeto.
- Formato: uma seção por preferência, com regra, motivo (se houver) e como aplicar.

## Comentários mínimos no código gerado

Por padrão, não escrever comentários no código que eu gero. O código deve se explicar pelos nomes e pela estrutura. Quando um comentário for realmente necessário, ele deve ser curto, com uma ou no máximo duas linhas. Comentários maiores só com pedido explícito.

Por quê: comentários desnecessários poluem o código, repetem o que ele já diz e ficam desatualizados com o tempo.

Como aplicar:

- Antes de comentar, tentar deixar o código claro sozinho: um nome melhor, uma variável intermediária ou uma função extraída.
- Comentar apenas o porquê que não aparece na leitura: uma regra de negócio, uma decisão incomum, uma limitação externa ou o contorno de um bug.
- Não comentar o que o código já diz, como o passo a passo de uma função, o nome de uma variável ou o tipo de um parâmetro.
- Não deixar comentários sobre a própria alteração (por exemplo, "adicionado", "corrigido", "antes era assim"). Isso vai na mensagem de commit ou no PR.
- Não deixar código comentado, separadores de seção nem `TODO` sem pedido.
- Documentação no padrão que o projeto já usa (por exemplo, PHPDoc, JSDoc ou docstrings) segue o estilo e o tamanho que o projeto pede. Se o projeto não documenta toda função, não passar a documentar.
- Não remover nem reescrever comentários que já existem fora do escopo da tarefa, a não ser que solicitado.

## Sem em dash no texto que eu produzo

Nunca escrever o caractere em dash (`—`, U+2014) em texto que vai para repositório ou para o time: mensagens de commit, títulos e descrições de PR, README, docs.md, páginas de wiki, comentários de código e strings de documentação. A única exceção é um pedido explícito.

Por quê: é uma marca típica de texto gerado por IA, e esse conteúdo fica em repositórios que o time inteiro lê.

Como aplicar:

- Usar a pontuação que a frase pede: vírgula, dois pontos, parênteses, ponto final ou uma frase nova. Não trocar por ` - ` ou `--` só para contornar a regra.
- Antes de commitar ou abrir PR, conferir o texto final com `git diff --cached | grep -n '—'`, além da própria mensagem de commit ou descrição do PR. Vale também para trechos reaproveitados de algo escrito antes na sessão.
- Não reescrever em dashes que já existem em código ou docs fora do escopo da tarefa. A regra vale para o que eu adiciono.
- O `--` já usado como estilo de comentário nos repositórios de IaC continua aceitável.
- Fora do escopo: respostas no chat e as linhas de índice do `MEMORY.md` da auto-memory, cujo formato é definido pelo sistema.

## Perguntar antes de adicionar memória, instruções ou skills

Nunca adicionar por conta própria uma entrada nova em memória, instruções ou skills fora do controle de versão. Quando eu identificar algo novo que valeria guardar, perguntar antes e só seguir com a confirmação do usuário. Editar ou remover o que já existe pode ser feito direto, com um aviso curto. Vale para:

- este arquivo (`~/.claude/CLAUDE.md`);
- auto-memory de qualquer projeto (`~/.claude/projects/<projeto>/memory/`);
- skills pessoais (`~/.claude/skills/`);
- `CLAUDE.local.md` de qualquer repo.

Não vale para arquivos versionados no repo, como `CLAUDE.md`, `AGENTS.md` e `.claude/skills/`, que já aparecem no diff.

Por quê: o usuário quer decidir o que passa a guiar o meu comportamento. Adicionar sem perguntar obriga a pedir a remoção do que ele não queria. Já as entradas existentes devem se manter atualizadas sem uma pergunta a cada ajuste.

Como aplicar:

- Entrada nova (uma memória, uma seção de instrução ou uma skill): pergunta curta, de uma linha, que possa ser respondida com sim ou não, dizendo o que seria guardado e onde. Por exemplo: "Quer que eu guarde na memória deste projeto que os testes rodam com `make test`?"
- Detalhar o conteúdo, o arquivo e o motivo só se o usuário perguntar.
- Fazer a pergunta no fim da resposta, sem interromper a tarefa em andamento.
- Sem confirmação, não adicionar nada. Se o usuário ignorar a pergunta, não repetir a mesma sugestão na sessão.
- Entrada existente: corrigir, complementar ou remover quando estiver desatualizada ou errada, e avisar em uma ou duas linhas o que mudou e o caminho do arquivo.
- Quando o próprio usuário pedir a alteração, fazer direto e confirmar em uma linha com o caminho do arquivo.

## Sem atribuição ao Claude em commits e PRs

Não incluir linhas de atribuição ao Claude no que eu commito ou publico, mesmo que o sistema sugira:

- Em mensagens de commit, sem o trailer `Co-Authored-By: Claude ...`.
- Em descrições de PR, sem o rodapé `🤖 Generated with [Claude Code](https://claude.com/claude-code)`.

A única exceção é um pedido explícito.

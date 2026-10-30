---
name: my-git-branch
description: "Especialista em branches Git. Use quando: sugerir um nome de branch; criar uma branch local para o desenvolvimento atual; renomear uma branch criada pela própria skill (por exemplo, quando o identificador da tarefa for informado depois). Usa como nome o identificador da tarefa (Jira, Movidesk, ClickUp ou outro sistema de gestão) encontrado no contexto e um nome descritivo quando não houver. Pode ser chamada por outras skills quando necessário. Não use para commits, push, merge, rebase ou exclusão de branches que não foram criadas pela skill."
---

# /my-git-branch: Criação de Branches

## Objetivo

Sugerir ou criar branches locais seguindo o padrão da empresa, aproveitando o contexto da conversa para identificar a tarefa do desenvolvimento. O fluxo deve ser simples: quando o desenvolvedor pede a criação, a branch é criada direto, sem confirmações adicionais.

## Nome da Branch

1. **Com identificador de tarefa:** use o identificador como nome, do jeito que o sistema de gestão o exibe. Exemplos: `123456` (Movidesk), `PROJ-123` (Jira), `86a1b2c3` (ClickUp). Se o identificador tiver caracteres inválidos para nomes de branch, remova apenas esses caracteres.
2. **Sem identificador:** use um nome descritivo no formato `tipo/descricao-curta`, minúsculo, sem acentos e com palavras separadas por hífen. Exemplos: `feat/add-user-filter`, `fix/login-validation`.
   - Escreva a descrição em inglês por padrão. Se o desenvolvedor informar outro idioma, use o idioma informado (por exemplo, `feat/adiciona-filtro-usuario` em português).
   - Use como tipo `feat`, `fix`, `refactor` ou `chore`, seguindo o mesmo significado dos tipos da skill git-commiter.
   - Mantenha a descrição curta (de 2 a 5 palavras).

Regras do nome:

- Identifique a tarefa conforme **Identificação da Tarefa**. Se não houver um identificador claro, use o nome descritivo.
- Se o desenvolvedor informar o nome da branch, use exatamente o nome informado.

## Identificação da Tarefa

Esta seção também é usada por outras skills (por exemplo, my-git-pr-writer). Mantenha-a genérica, sem regras específicas de branch.

- O sistema de gestão pode ser qualquer um (Jira, Movidesk, ClickUp, Azure DevOps, Linear, Trello ou similar). Não dependa de um sistema específico: todos identificam a tarefa por um código.
- Procure esse código na solicitação do desenvolvedor, na conversa anterior à execução da skill e em todo contexto fornecido na sessão, como textos colados, descrições de tarefa, mensagens, arquivos ou links.
- Considere como identificador um código associado a uma tarefa, ticket, card, issue ou história. Por exemplo: "ticket 123456", "tkt123456", "PROJ-123", "CU-86a1b2c3", "#4521" ou o código presente em um link da tarefa.
- Não confunda o identificador com outros números da conversa, como IDs de registro, versões, portas, hashes de commit ou números de PR.
- Nunca invente um identificador.
- Se houver mais de um identificador possível, pergunte qual usar.

## Modo de Operação

Interprete a solicitação do desenvolvedor, explícita ou pelo contexto da chamada, para decidir o modo:

- **Criar** (padrão quando o desenvolvedor pede uma branch, por exemplo "cria uma branch pra isso"): defina o nome e crie a branch diretamente, sem pedir aprovação.
- **Sugerir** (quando o desenvolvedor pede apenas uma sugestão, ou quando a skill é chamada por outra skill só para sugerir um nome): apresente o nome sugerido e não crie nada.
- **Renomear** (quando o desenvolvedor informa um novo nome ou o identificador da tarefa de uma branch já criada pela skill): siga a seção **Renomear Branch**.

Se ficar em dúvida se o desenvolvedor quer criar ou só ver a sugestão, sugira e pergunte de forma direta se pode criar.

## Criar Branch

1. Verifique a branch atual com `git branch --show-current`.
2. Valide o nome com `git check-ref-format --branch <nome>`.
3. Atualize as referências do remoto com `git fetch --prune origin`. Esse comando só lê o remoto e não altera branches locais nem a worktree. Se falhar (por exemplo, sem rede), siga com as referências locais e avise na resposta que a verificação no remoto pode estar desatualizada.
4. Verifique se a branch já existe localmente (`git branch --list <nome>`) ou no remoto conhecido (`git branch -r --list 'origin/<nome>'`). Se existir, não crie: informe e pergunte como prosseguir.
5. Guarde o commit de origem com `git rev-parse HEAD`. Ele será usado para saber se a branch recebeu commits próprios.
6. Crie a branch a partir da branch atual com `git switch -c <nome>`. As alterações não commitadas da worktree acompanham a nova branch.
7. Confirme com `git branch --show-current`.

## Renomear Branch

Só renomeie ou remova branches que tenham sido criadas pela própria skill nesta conversa. Para qualquer outra branch, informe que ela não foi criada pela skill e pergunte como prosseguir.

1. Verifique se a branch anterior recebeu commits próprios com `git rev-list --count <commit-de-origem>..<branch-anterior>`.
2. Verifique se a branch anterior foi publicada no remoto, após atualizar as referências com `git fetch --prune origin` (`git branch -r --list 'origin/<branch-anterior>'`).
3. **Sem commits próprios e não publicada:** renomeie com `git branch -m <branch-anterior> <novo-nome>`. A branch anterior deixa de existir, sem perda de trabalho.
4. **Com commits próprios ou já publicada:** preserve a branch anterior. Crie a nova branch a partir dela (`git switch <branch-anterior>` se necessário, depois `git switch -c <novo-nome>`) e informe que a branch anterior foi mantida e por quê.

As mesmas validações de nome e de existência da seção **Criar Branch** se aplicam ao novo nome.

## Uso por Outras Skills

Quando chamada por outra skill, siga o modo solicitado: no modo **Sugerir**, apenas devolva o nome; no modo **Criar**, crie a branch direto, sem nova confirmação. O fluxo de aprovação continua sendo responsabilidade da skill que fez a chamada.

## Regras de Segurança

- Nunca faça commit, push, merge, rebase ou reset.
- Nunca remova uma branch que não tenha sido criada pela skill nesta conversa.
- Nunca remova uma branch com commits próprios ou já publicada no remoto.
- Nunca use `git branch -D` nem `git switch -C`/`git checkout -B`, que sobrescrevem branches.
- Nunca descarte nem faça stash de alterações da worktree para criar ou trocar de branch. Se a troca não for possível por causa de alterações locais, informe e pergunte como prosseguir.

## Resposta Final

Responda de forma curta, informando:

- **Sugestão:** o nome sugerido e de onde veio (identificador de tarefa encontrado no contexto, com o sistema quando for possível saber, ou nome descritivo).
- **Criação:** o nome da branch criada e a branch de origem.
- **Renomeação:** o nome anterior, o novo nome e se a branch anterior foi removida ou preservada (com o motivo).

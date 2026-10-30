# my-personal-skills

Skills pessoais para o Claude Code, usadas no trabalho do dia a dia.

São skills genéricas, que não dependem de um projeto específico. Cada uma resolve uma tarefa da rotina e funciona do seu próprio jeito, conforme o que a tarefa pede.

## Idioma

Por padrão, as skills são escritas em português do Brasil, para facilitar a manutenção. O ideal seria mantê-las em inglês, mas a escolha pelo português é intencional.

## Estrutura

```text
skills/
└── <nome-da-skill>/
    ├── SKILL.md   # instruções da skill (frontmatter com name e description)
    └── ...        # arquivos de apoio, quando houver
```

## Instalação

O Claude Code carrega as skills pessoais da pasta `skills` dentro do diretório `.claude` do usuário:

| Sistema | Pasta das skills |
| --- | --- |
| Linux, WSL e macOS | `~/.claude/skills/` |
| Windows | `%USERPROFILE%\.claude\skills\` |

Para instalar uma skill, coloque a pasta dela (a que contém o `SKILL.md`) dentro dessa pasta, mantendo o nome. Por exemplo, `skills/my-git-branch/` deste repositório vira `~/.claude/skills/my-git-branch/`.

Existem duas formas de fazer isso:

- **Copiar a pasta da skill:** é o jeito mais simples. As alterações feitas no repositório precisam ser copiadas de novo.
- **Criar um link para a pasta da skill:** a skill instalada passa a apontar para o repositório, e um `git pull` já a mantém atualizada.

Exemplos de link, a partir da raiz do repositório clonado:

```bash
# Linux, WSL e macOS
mkdir -p ~/.claude/skills
for skill in skills/*/; do
  ln -sfn "$PWD/$skill" ~/.claude/skills/"$(basename "$skill")"
done
```

```powershell
# Windows (PowerShell); junctions não exigem permissão de administrador
New-Item -ItemType Directory -Force "$env:USERPROFILE\.claude\skills" | Out-Null
Get-ChildItem skills -Directory | ForEach-Object {
  New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\$($_.Name)" -Target $_.FullName
}
```

Se já existir uma pasta com o mesmo nome em `.claude/skills`, remova-a ou mova-a antes de criar o link.

As skills ficam disponíveis a partir da próxima sessão do Claude Code e podem ser chamadas por `/<nome-da-skill>` ou acionadas automaticamente pela descrição.

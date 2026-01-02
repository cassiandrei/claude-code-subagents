# Claude Code Subagents

Repositório com agentes e comandos customizados para o Claude Code.

## Conteúdo

- **agents/** - Agentes personalizados
  - `designer.md` - Agente de design
- **commands/** - Comandos customizados
  - `setup-figma.md` - Comando para configurar Figma

## Instalação

Para sincronizar sua pasta `~/.claude` com este repositório, execute os seguintes comandos:

```bash
cd ~/.claude
git init
git remote add origin https://github.com/cassiandrei/claude-code-subagents.git
git fetch origin
git checkout -b main
git pull origin main
```

## Atualização

Para atualizar os agentes e comandos com as últimas mudanças do repositório:

```bash
cd ~/.claude
git pull origin main
```

## Observações

- Os arquivos locais como `settings.json` e `.credentials.json` estão no `.gitignore` e não serão sincronizados
- Após a instalação, os agentes e comandos estarão disponíveis automaticamente no Claude Code

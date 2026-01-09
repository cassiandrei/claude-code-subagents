---
description: Setup GitLab MCP for Claude Code integration
---

# Setup GitLab MCP

Configura o MCP do GitLab para integração com Claude Code. Funciona com GitLab.com e instâncias self-hosted (incluindo Community Edition).

## Passos

Execute os seguintes passos para configurar o GitLab MCP:

### 1. Verificar pré-requisitos

Verifique se o Node.js está instalado (versão 18+):
```bash
node --version
```

Se não estiver instalado, instrua o usuário a instalar via:
- macOS: `brew install node`
- Ubuntu/Debian: `sudo apt install nodejs npm`
- Windows: https://nodejs.org/

### 2. Coletar informações do usuário

Pergunte ao usuário:

1. **Escopo da configuração:**
   - **Global** (recomendado) - Disponível em todos os projetos (`~/.claude/settings.json`)
   - **Por projeto** - Apenas no projeto atual (`~/.claude.json` seção projects)

2. **Qual GitLab você usa?**
   - GitLab.com (https://gitlab.com)
   - Self-hosted (ex: https://git.empresa.com.br)

3. **Você já possui um Personal Access Token?**

4. **Qual nome dar ao servidor MCP?** (opcional)
   - Sugestão: `gitlab` para GitLab.com ou `gitlab-empresa` para self-hosted

### 3. Criar Personal Access Token (se necessário)

Se o usuário NÃO tiver um token, instrua:

1. Acessar o GitLab e fazer login
2. Clicar no avatar > **Preferences** (ou **Edit profile**)
3. No menu lateral, clicar em **Access Tokens**
4. Clicar em **Add new token**
5. Configurar:
   - **Token name**: `claude-code-mcp`
   - **Expiration date**: Escolher uma data (máx 1 ano)
   - **Scopes**: Selecionar:
     - `api` (acesso completo à API)
     - `read_repository` (leitura de repos)
     - `write_repository` (escrita em repos)
6. Clicar em **Create personal access token**
7. **IMPORTANTE**: Copiar o token imediatamente (só aparece uma vez!)

O token terá o formato: `glpat-XXXXXXXXXXXXXXXXXXXX`

### 4. Configurar o MCP

#### Opção A: Configuração Global (Recomendado)

Edite o arquivo `~/.claude/settings.json` e adicione a configuração do GitLab.

**Para GitLab.com:**
```json
{
  "mcpServers": {
    "gitlab": {
      "command": "npx",
      "args": ["-y", "@zereight/mcp-gitlab"],
      "env": {
        "GITLAB_PERSONAL_ACCESS_TOKEN": "glpat-XXXXXXXXXXXXX",
        "GITLAB_API_URL": "https://gitlab.com/api/v4",
        "GITLAB_READ_ONLY_MODE": "false",
        "USE_GITLAB_WIKI": "true",
        "USE_MILESTONE": "true",
        "USE_PIPELINE": "true"
      }
    }
  }
}
```

**Para GitLab Self-Hosted:**
```json
{
  "mcpServers": {
    "gitlab-empresa": {
      "command": "npx",
      "args": ["-y", "@zereight/mcp-gitlab"],
      "env": {
        "GITLAB_PERSONAL_ACCESS_TOKEN": "glpat-XXXXXXXXXXXXX",
        "GITLAB_API_URL": "https://SEU-GITLAB.com.br/api/v4",
        "GITLAB_READ_ONLY_MODE": "false",
        "USE_GITLAB_WIKI": "true",
        "USE_MILESTONE": "true",
        "USE_PIPELINE": "true"
      }
    }
  }
}
```

**Se já existir um `settings.json`**, mescle a configuração ao objeto `mcpServers` existente.

#### Opção B: Configuração Por Projeto

Edite o arquivo `~/.claude.json` e adicione a configuração na seção `projects`:

```json
{
  "projects": {
    "/caminho/do/projeto": {
      "mcpServers": {
        "gitlab-empresa": {
          "command": "npx",
          "args": ["-y", "@zereight/mcp-gitlab"],
          "env": {
            "GITLAB_PERSONAL_ACCESS_TOKEN": "glpat-XXXXX",
            "GITLAB_API_URL": "https://git.empresa.com.br/api/v4",
            "GITLAB_READ_ONLY_MODE": "false",
            "USE_GITLAB_WIKI": "true",
            "USE_MILESTONE": "true",
            "USE_PIPELINE": "true"
          }
        }
      }
    }
  }
}
```

### 5. Resumo dos locais de configuração

| Arquivo | Escopo | Quando usar |
|---------|--------|-------------|
| `~/.claude/settings.json` | **Global** | Usar o mesmo GitLab em todos os projetos |
| `~/.claude.json` (seção projects) | Por projeto | GitLabs diferentes por projeto |
| `.mcp.json` na raiz do projeto | Por projeto | Compartilhar config via git (sem tokens!)

### 6. Variáveis de ambiente opcionais

Explique as opções adicionais disponíveis:

| Variável | Descrição | Default |
|----------|-----------|---------|
| `GITLAB_PROJECT_ID` | ID do projeto padrão | - |
| `GITLAB_READ_ONLY_MODE` | Modo somente leitura | `false` |
| `USE_GITLAB_WIKI` | Habilita ferramentas de wiki | `true` |
| `USE_MILESTONE` | Habilita ferramentas de milestones | `true` |
| `USE_PIPELINE` | Habilita ferramentas de pipelines | `true` |

### 7. Finalização

Informe ao usuário:

1. **Reiniciar o Claude Code** para carregar o MCP
2. Executar `/mcp` para verificar se está conectado
3. O primeiro uso pode demorar enquanto o npx baixa o pacote (~30s)

### 8. Ferramentas disponíveis

Após configurado, o usuário terá acesso a **95+ ferramentas**, incluindo:

**Repositórios:**
- `list_projects`, `get_project`, `create_repository`, `fork_repository`

**Issues:**
- `list_issues`, `get_issue`, `create_issue`, `update_issue`, `delete_issue`

**Merge Requests:**
- `list_merge_requests`, `get_merge_request`, `create_merge_request`, `merge_merge_request`

**Branches & Commits:**
- `list_commits`, `get_commit`, `create_branch`, `get_branch_diffs`

**Pipelines:**
- `list_pipelines`, `get_pipeline`, `create_pipeline`, `retry_pipeline`, `cancel_pipeline`

**Arquivos:**
- `get_file_contents`, `create_or_update_file`, `push_files`

**E muito mais:** wiki, labels, milestones, releases, discussions, draft notes, etc.

## Troubleshooting

### Erro "401 Unauthorized"
- Verifique se o token está correto
- Confirme se o token tem os scopes necessários (`api`, `read_repository`, `write_repository`)
- Verifique se o token não expirou

### Erro "404 Not Found"
- Verifique se a URL do GitLab está correta
- Confirme se inclui `/api/v4` no final
- Verifique se não há barra extra no final da URL

### MCP não aparece no `/mcp`
- Reinicie o Claude Code completamente
- Verifique se o `settings.json` está no path correto (`~/.claude/settings.json`)
- Verifique se o JSON está válido (sem erros de sintaxe)
- Execute `claude mcp list` para verificar se foi carregado

### Erro de SSL/Certificado
- Para GitLab self-hosted com certificado auto-assinado, pode ser necessário configurar `NODE_TLS_REJECT_UNAUTHORIZED=0` (não recomendado para produção)

### Primeiro uso lento
- O npx precisa baixar o pacote na primeira execução (~30s)
- Execuções subsequentes serão mais rápidas (cache do npm)

## Comandos úteis

```bash
# Listar MCPs configurados
claude mcp list

# Verificar detalhes de um MCP
claude mcp get gitlab

# Remover configuração
claude mcp remove gitlab
```

## Referências

- Repositório MCP: https://github.com/zereight/gitlab-mcp
- GitLab Personal Access Tokens: https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html
- MCP Protocol: https://modelcontextprotocol.io/

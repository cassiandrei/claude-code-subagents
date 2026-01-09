---
description: Setup YouTrack MCP for Claude Code integration
---

# Setup YouTrack MCP

Configura o MCP do YouTrack para integração com Claude Code. Funciona com YouTrack Cloud e instâncias self-hosted.

## Passos

Execute os seguintes passos para configurar o YouTrack MCP:

### 1. Coletar informações do usuário

Pergunte ao usuário:

1. **Escopo da configuração:**
   - **Global** (recomendado) - Disponível em todos os projetos (`~/.claude/settings.json`)
   - **Por projeto** - Apenas no projeto atual (`~/.claude.json` seção projects)

2. **Qual é a URL base do YouTrack?**
   - YouTrack Cloud (ex: https://empresa.youtrack.cloud)
   - Self-hosted (ex: https://youtrack.empresa.com.br)

3. **Você já possui um Permanent Token?**

### 2. Criar Permanent Token (se necessário)

Se o usuário NÃO tiver um token, instrua:

1. Acessar o YouTrack e fazer login
2. Clicar no avatar > **Profile**
3. No menu lateral, clicar em **Account Security**
4. Em **Tokens**, clicar em **New token...**
5. Configurar:
   - **Name**: `claude-code-mcp`
   - **Scope**: Selecionar **YouTrack** (acesso completo)
6. Clicar em **Create**
7. **IMPORTANTE**: Copiar o token imediatamente (só aparece uma vez!)

O token terá o formato: `perm:XXXXXXXX.XX-XX.XXXXXXXXXXXXXXXXXXXX`

### 3. Configurar o MCP

**IMPORTANTE**: O comando `claude mcp add --header` não salva os headers corretamente para HTTP transport. É necessário editar manualmente o arquivo de configuração.

#### Opção A: Configuração Global (Recomendado)

Edite o arquivo `~/.claude/settings.json` e adicione a configuração do YouTrack:

```json
{
  "mcpServers": {
    "youtrack": {
      "type": "http",
      "url": "https://empresa.youtrack.cloud/mcp",
      "headers": {
        "Authorization": "Bearer <SEU_TOKEN>"
      }
    }
  }
}
```

**Se já existir um `settings.json` com outros MCPs**, mescle a configuração ao objeto `mcpServers` existente:

```json
{
  "mcpServers": {
    "gitlab": { ... },
    "youtrack": {
      "type": "http",
      "url": "https://empresa.youtrack.cloud/mcp",
      "headers": {
        "Authorization": "Bearer perm:XXXXX.XXXXX.XXXXXXXXXXXXX"
      }
    }
  }
}
```

#### Opção B: Configuração Por Projeto

Edite o arquivo `~/.claude.json` e adicione a configuração na seção `projects`:

```json
{
  "projects": {
    "/caminho/do/seu/projeto": {
      "mcpServers": {
        "youtrack": {
          "type": "http",
          "url": "https://empresa.youtrack.cloud/mcp",
          "headers": {
            "Authorization": "Bearer <SEU_TOKEN>"
          }
        }
      }
    }
  }
}
```

### 4. Resumo dos locais de configuração

| Arquivo | Escopo | Quando usar |
|---------|--------|-------------|
| `~/.claude/settings.json` | **Global** | Usar o mesmo YouTrack em todos os projetos |
| `~/.claude.json` (seção projects) | Por projeto | YouTracks diferentes por projeto |

### 5. Parâmetros opcionais

É possível personalizar as ferramentas disponíveis adicionando query parameters à URL:

| Parâmetro | Descrição | Exemplo |
|-----------|-----------|---------|
| `tools=` | Lista de ferramentas permitidas | `?tools=getIssue,createIssue` |
| `ignoreTools=` | Ferramentas a excluir | `?ignoreTools=deleteIssue` |
| `enableToolOutputSchema=true` | Habilita schemas de saída | Útil para validação |

**Exemplo com parâmetros:**
```json
"youtrack": {
  "type": "http",
  "url": "https://empresa.youtrack.cloud/mcp?enableToolOutputSchema=true",
  "headers": {
    "Authorization": "Bearer <TOKEN>"
  }
}
```

### 6. Finalização

Informe ao usuário:

1. **Reiniciar o Claude Code** para carregar o MCP
2. Executar `/mcp` para verificar se está conectado
3. As ferramentas do YouTrack estarão disponíveis imediatamente

### 7. Ferramentas disponíveis

Após configurado, o usuário terá acesso a ferramentas como:

**Issues:**
- Buscar, criar, atualizar e deletar issues
- Adicionar comentários e anexos
- Gerenciar links entre issues

**Projetos:**
- Listar e consultar projetos
- Gerenciar boards e sprints

**Agile:**
- Consultar boards ágeis
- Gerenciar sprints e backlogs

**Pesquisa:**
- Buscar issues com queries YouTrack
- Filtrar por projeto, estado, assignee, etc.

## Troubleshooting

### Erro "Invalid URL" com mcp-remote

O `mcp-remote` **NÃO funciona** com YouTrack porque o servidor retorna um path OAuth relativo (`/hub`) que o mcp-remote não consegue processar:

```
Fatal error: TypeError: Invalid URL
  input: '/hub'
```

**Solução**: Use o método HTTP transport direto (configuração manual no `settings.json` ou `.claude.json`) em vez de `mcp-remote`.

### Headers não salvos pelo `claude mcp add`

O comando `claude mcp add --header` não salva os headers para HTTP transport.

**Solução**: Edite manualmente o arquivo de configuração (`~/.claude/settings.json` para global ou `~/.claude.json` para por projeto) e adicione a propriedade `headers` conforme o passo 3.

### Erro "401 Unauthorized"
- Verifique se o token está correto
- Confirme se o token tem o scope "YouTrack"
- Verifique se o token não foi revogado

### Erro "404 Not Found"
- Verifique se a URL base está correta
- Confirme se o endpoint termina em `/mcp`
- Verifique se o YouTrack suporta MCP (requer versão recente)

### Erro "405 Method Not Allowed"
- Este erro em requisições HEAD/GET é esperado - o MCP usa SSE
- O endpoint está funcionando, continue com a configuração

### MCP não aparece no `/mcp`
- Reinicie o Claude Code completamente
- Verifique a configuração com `claude mcp list`
- Para remover e reconfigurar: `claude mcp remove youtrack`

### Erro de conexão/timeout
- Verifique se a URL do YouTrack está acessível
- Teste a conexão: `curl -I https://seu-youtrack.com/mcp`
- Verifique firewalls ou VPNs

## Comandos úteis

```bash
# Listar MCPs configurados
claude mcp list

# Remover configuração do YouTrack
claude mcp remove youtrack

# Verificar detalhes
claude mcp get youtrack
```

## Referências

- Documentação oficial: https://www.jetbrains.com/help/youtrack/cloud/model-context-protocol-server.html
- YouTrack Permanent Tokens: https://www.jetbrains.com/help/youtrack/cloud/Manage-Permanent-Token.html
- MCP Protocol: https://modelcontextprotocol.io/

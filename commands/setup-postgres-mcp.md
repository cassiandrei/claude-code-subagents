---
description: Setup PostgreSQL MCP for Claude Code integration
---

# Setup PostgreSQL MCP

Configura o MCP do PostgreSQL para integração com Claude Code. Permite executar queries SQL diretamente pelo Claude.

## Passos

Execute os seguintes passos para configurar o PostgreSQL MCP:

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
   - **Global** (recomendado) - Disponível em todos os projetos (`~/.claude.json` seção `mcpServers` raiz)
   - **Por projeto** - Apenas no projeto atual (`~/.claude.json` seção `projects`)

2. **Dados de conexão do PostgreSQL:**
   - **Host**: Endereço do servidor (ex: localhost, 192.168.1.100, db.empresa.com)
   - **Porta**: Porta do PostgreSQL (padrão: 5432)
   - **Database**: Nome do banco de dados
   - **Usuário**: Nome de usuário do PostgreSQL
   - **Senha**: Senha do usuário

3. **Qual nome dar ao servidor MCP?** (opcional)
   - Sugestão: `postgres` ou `postgres-empresa`

### 3. Montar a Connection String

A connection string do PostgreSQL segue o formato:
```
postgresql://usuario:senha@host:porta/database
```

**IMPORTANTE**: Se a senha contiver caracteres especiais, eles devem ser URL-encoded:

| Caractere | Encoded |
|-----------|---------|
| `@` | `%40` |
| `:` | `%3A` |
| `/` | `%2F` |
| `?` | `%3F` |
| `#` | `%23` |
| `=` | `%3D` |
| `%` | `%25` |
| ` ` (espaço) | `%20` |

**Exemplo com senha especial:**
- Senha original: `p@ss?word=123`
- Senha encoded: `p%40ss%3Fword%3D123`
- Connection string: `postgresql://usuario:p%40ss%3Fword%3D123@localhost:5432/meubanco`

### 4. Configurar o MCP

#### Opção A: Configuração Global (Recomendado)

**IMPORTANTE**: O arquivo correto para configuração global é `~/.claude.json`, NÃO `~/.claude/settings.json`.

Edite o arquivo `~/.claude.json` e adicione/atualize a seção `mcpServers` no **nível raiz** do JSON (não dentro de `projects`).

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://usuario:senha@host:5432/database"
      ]
    }
  }
}
```

**Exemplo com valores reais:**
```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://admin:minhasenha123@localhost:5432/producao"
      ]
    }
  }
}
```

**Se já existir `mcpServers` com outros MCPs**, mescle a configuração ao objeto existente:

```json
{
  "mcpServers": {
    "gitlab": { ... },
    "youtrack": { ... },
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "postgresql://usuario:senha@host:5432/database"
      ]
    }
  }
}
```

**Alternativa via CLI:**
```bash
claude mcp add postgres -s user -- npx -y @modelcontextprotocol/server-postgres "postgresql://usuario:senha@host:5432/database"
```

#### Opção B: Configuração Por Projeto

Edite o arquivo `~/.claude.json` e adicione a configuração na seção `projects`:

```json
{
  "projects": {
    "/caminho/do/projeto": {
      "mcpServers": {
        "postgres": {
          "command": "npx",
          "args": [
            "-y",
            "@modelcontextprotocol/server-postgres",
            "postgresql://usuario:senha@host:5432/database"
          ]
        }
      }
    }
  }
}
```

### 5. Resumo dos locais de configuração

| Arquivo | Escopo | Quando usar |
|---------|--------|-------------|
| `~/.claude.json` (seção `mcpServers` raiz) | **Global** | Usar o mesmo banco em todos os projetos |
| `~/.claude.json` (seção `projects`) | Por projeto | Bancos diferentes por projeto |
| `.mcp.json` na raiz do projeto | Por projeto | Compartilhar config via git (SEM senhas!) |

**ATENÇÃO**: `~/.claude/settings.json` NÃO é lido pelo Claude Code para MCPs. Use sempre `~/.claude.json`.

### 6. Finalização

Informe ao usuário:

1. **Reiniciar o Claude Code** para carregar o MCP
2. Executar `/mcp` para verificar se está conectado
3. O primeiro uso pode demorar enquanto o npx baixa o pacote (~30s)

### 7. Testar a conexão

Após configurar, peça ao usuário para testar com:
```
Teste a conexão com o banco de dados
```

Ou execute uma query simples:
```
SELECT version()
```

### 8. Ferramentas disponíveis

Após configurado, o usuário terá acesso à ferramenta:

**query**
- Executa queries SQL read-only no banco de dados
- Retorna os resultados em formato JSON
- Suporta SELECT, SHOW, DESCRIBE e outras queries de leitura

**Exemplos de uso:**
```sql
-- Listar tabelas
SELECT table_name FROM information_schema.tables WHERE table_schema = 'public'

-- Consultar dados
SELECT * FROM usuarios LIMIT 10

-- Ver estrutura de tabela
SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'usuarios'
```

## Troubleshooting

### Erro "connection refused"
- Verifique se o PostgreSQL está rodando
- Confirme se o host e porta estão corretos
- Verifique se o firewall permite conexões na porta 5432

### Erro "authentication failed"
- Verifique se o usuário e senha estão corretos
- Confirme se caracteres especiais na senha estão URL-encoded
- Verifique se o usuário tem permissão de acesso ao banco

### Erro "database does not exist"
- Verifique se o nome do banco está correto
- Liste bancos disponíveis: `psql -h host -U usuario -l`

### Erro "SSL required"
Se o servidor requer SSL, adicione `?sslmode=require` à connection string:
```
postgresql://usuario:senha@host:5432/database?sslmode=require
```

Opções de sslmode:
- `disable` - Sem SSL
- `allow` - Tenta sem SSL, depois com SSL
- `prefer` - Tenta com SSL, depois sem SSL (padrão)
- `require` - Requer SSL, não verifica certificado
- `verify-ca` - Requer SSL, verifica CA
- `verify-full` - Requer SSL, verifica CA e hostname

### MCP não aparece no `/mcp`
- Reinicie o Claude Code completamente
- Verifique se a configuração está em `~/.claude.json` (NÃO em `~/.claude/settings.json`)
- Verifique se `mcpServers` está no nível raiz do JSON, não dentro de `projects`
- Verifique se o JSON está válido (sem erros de sintaxe)
- Execute `claude mcp list` para verificar se foi carregado

### Primeiro uso lento
- O npx precisa baixar o pacote na primeira execução (~30s)
- Execuções subsequentes serão mais rápidas (cache do npm)

### Erro "read-only transaction"
- O MCP do PostgreSQL opera em modo **read-only** por segurança
- Apenas queries de leitura (SELECT, SHOW, etc.) são permitidas
- Para operações de escrita, use outro cliente PostgreSQL

## Comandos úteis

```bash
# Listar MCPs configurados
claude mcp list

# Verificar detalhes de um MCP
claude mcp get postgres

# Remover configuração
claude mcp remove postgres

# Testar conexão manualmente
psql "postgresql://usuario:senha@host:5432/database" -c "SELECT 1"
```

## Segurança

**Boas práticas:**
- Nunca commite arquivos com senhas no git
- Use usuários com permissões mínimas necessárias (read-only)
- Considere usar variáveis de ambiente para senhas sensíveis
- Em ambientes de produção, use conexões SSL

**Usando variáveis de ambiente:**
```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-postgres",
        "${POSTGRES_CONNECTION_STRING}"
      ]
    }
  }
}
```

Depois defina a variável no seu shell:
```bash
export POSTGRES_CONNECTION_STRING="postgresql://usuario:senha@host:5432/database"
```

## Referências

- Pacote MCP: https://www.npmjs.com/package/@modelcontextprotocol/server-postgres
- PostgreSQL Connection Strings: https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING
- MCP Protocol: https://modelcontextprotocol.io/

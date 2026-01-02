# Setup Figma MCP

Configura o MCP do Figma para integração com Claude Code.

## Passos

Execute os seguintes passos para configurar o Figma MCP:

### 1. Verificar pré-requisitos

Verifique se o Node.js está instalado:
```bash
node --version
```

### 2. Criar arquivo `.mcp.json`

Crie ou atualize o arquivo `.mcp.json` na raiz do projeto com:

```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "figma-developer-mcp", "--stdio"],
      "env": {
        "FIGMA_API_KEY": "${FIGMA_API_KEY}"
      }
    }
  }
}
```

Se o arquivo já existir, adicione a configuração do Figma ao objeto `mcpServers`.

### 3. Configurar a API Key

Pergunte ao usuário se ele já possui uma API Key do Figma.

**Se NÃO tiver**, instrua:
1. Acessar [figma.com](https://figma.com) e fazer login
2. Clicar no avatar > **Settings**
3. Rolar até **Personal access tokens**
4. Clicar em **Generate new token**
5. Copiar o token gerado

**Depois de obter a key**, configure de uma das formas:

**Opção A - Configuração global (Recomendado):**
Edite `~/.claude/settings.json` e adicione:
```json
{
  "env": {
    "FIGMA_API_KEY": "token-do-usuario"
  }
}
```

**Opção B - Arquivo `.env` no projeto:**
```
FIGMA_API_KEY=token-do-usuario
```

### 4. Adicionar `.env` ao `.gitignore`

Se usar a opção B, garanta que `.env` está no `.gitignore`:
```
.env
.env.local
```

### 5. Finalização

Informe ao usuário:
- Reiniciar o Claude Code para carregar o MCP
- Testar com um link do Figma
- Usar o agente `designer` para trabalhar com designs: `claude --agent designer`

## Referências

- Repositório: https://github.com/GLips/Figma-Context-MCP
- Figma API: https://www.figma.com/developers/api

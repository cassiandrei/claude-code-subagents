---
description: Code review de Merge Request do GitLab do Grupo Voalle
allowed-tools: mcp__gitlab-syntesis__get_merge_request, mcp__gitlab-syntesis__get_merge_request_diffs, mcp__gitlab-syntesis__list_merge_request_diffs, mcp__gitlab-syntesis__mr_discussions, mcp__gitlab-syntesis__create_merge_request_thread, mcp__gitlab-syntesis__create_note, Bash, Read, Grep, Glob
---

# Code Review de Merge Request

Realize um code review completo do Merge Request associado à branch fornecida.

## Argumentos

- `$ARGUMENTS` - Nome da branch source do MR (opcional). Se não fornecido, usa a branch atual.

## Instruções

### 1. Determinar a branch

Se `$ARGUMENTS` estiver vazio ou não fornecido:
```bash
git branch --show-current
```

Use o resultado como a branch source para buscar o MR.

### 2. Buscar o Merge Request

Use a ferramenta `mcp__gitlab-syntesis__get_merge_request` com o parâmetro `source_branch` para encontrar o MR aberto associado à branch.

Se não encontrar um MR:
- Informe ao usuário que não existe MR aberto para essa branch
- Sugira criar um MR primeiro

### 3. Obter os diffs do MR

Use `mcp__gitlab-syntesis__get_merge_request_diffs` ou `mcp__gitlab-syntesis__list_merge_request_diffs` para obter todas as alterações do MR.

### 4. Analisar o código

Para cada arquivo modificado, analise:

**Qualidade do código:**
- Legibilidade e clareza
- Nomes de variáveis e funções
- Complexidade desnecessária
- Código duplicado

**Bugs potenciais:**
- Null/undefined não tratados
- Race conditions
- Memory leaks
- Off-by-one errors
- Edge cases não cobertos

**Segurança:**
- Injeção de SQL/comandos
- XSS (Cross-Site Scripting)
- Exposição de dados sensíveis
- Validação de input
- Autenticação/autorização

**Performance:**
- Loops ineficientes
- Queries N+1
- Operações desnecessárias
- Uso excessivo de memória

**Boas práticas:**
- Tratamento de erros
- Logging adequado
- Testes (se aplicável)
- Documentação necessária

### 5. Verificar discussões existentes

Use `mcp__gitlab-syntesis__mr_discussions` para ver se já existem comentários/discussões no MR que precisam de atenção.

### 6. Apresentar o review

Apresente o review de forma estruturada:

```markdown
## Code Review - MR !{iid}: {título}

### Resumo
[Breve descrição das mudanças e impressão geral]

### Pontos Positivos
- [Liste aspectos bem implementados]

### Problemas Encontrados

#### Críticos (Bloqueiam aprovação)
- [ ] **[arquivo:linha]** - Descrição do problema
  - Sugestão de correção

#### Importantes (Devem ser corrigidos)
- [ ] **[arquivo:linha]** - Descrição do problema
  - Sugestão de correção

#### Sugestões (Nice to have)
- [ ] **[arquivo:linha]** - Descrição da sugestão

### Veredicto
[APROVAR / SOLICITAR MUDANÇAS / BLOQUEAR]

[Justificativa breve]
```

### 7. Perguntar sobre comentários no GitLab

Após apresentar o review, pergunte ao usuário:

> Deseja que eu adicione esses comentários diretamente no MR do GitLab?

Se o usuário concordar, use `mcp__gitlab-syntesis__create_merge_request_thread` para criar threads de discussão nos arquivos específicos com os problemas encontrados.

## Notas

- Seja construtivo e educativo nos comentários
- Foque em problemas reais, não em preferências pessoais de estilo
- Considere o contexto do projeto ao fazer sugestões
- Priorize issues de segurança e bugs sobre estilo

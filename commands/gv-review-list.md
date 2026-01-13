---
description: Lista itens em revisão no YouTrack
allowed-tools: mcp__youtrack__search_issues, mcp__youtrack__get_issue, mcp__youtrack__find_projects
---

# Listar Itens em Revisão - YouTrack

Lista todos os cards em revisão de um projeto no YouTrack.

## Instruções

### 1. Perguntar o projeto

Pergunte ao usuário qual projeto ele deseja consultar. Se necessário, use `mcp__youtrack__find_projects` para ajudar a encontrar o projeto correto.

### 2. Buscar itens em revisão

Use a ferramenta `mcp__youtrack__search_issues` com a seguinte query (substituindo `{PROJETO}` pelo projeto informado):

```
project: {PROJETO} Status: {Backlog-Review}, {In Progress-Review} Type: -Task ordenar por: {Classe de serviço}, {Data repactuada} asc
```

Parâmetros:
- `query`: a query acima com o projeto substituído
- `customFieldsToReturn`: `["Type", "Status", "Atendente", "Classe de serviço", "Data repactuada"]`
- `limit`: 20

### 3. Apresentar resultados

Apresente os resultados em formato de tabela markdown:

```markdown
## Itens em Revisão - {PROJETO}

| ID | Resumo | Tipo | Data Repactuada | Atendente |
|-----|--------|------|-----------------|-----------|
| [XXX-XXXX](url) | Resumo do card | Tipo | DD/MM/YYYY | nome.atendente |

**Total: X cards**
```

### 4. Detalhes adicionais (opcional)

Se o usuário solicitar detalhes de um card específico, use `mcp__youtrack__get_issue` para obter informações completas.

## Notas

- A ordenação é por Classe de serviço e Data repactuada (ascendente)
- Tasks são excluídas do filtro (apenas User Stories, Incidents, etc.)
- Os status considerados são: Backlog-Review e In Progress-Review

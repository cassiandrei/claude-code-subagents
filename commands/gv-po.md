---
description: Cria User Story no YouTrack a partir de um Epic
allowed-tools: mcp__youtrack__*, mcp__gitlab__*, Bash, Read, Glob, Grep, AskUserQuestion, Task
---

**IMPORTANTE:** Antes de executar, carregue a skill `gv-youtrack` para entender o workflow do YouTrack no Grupo Voalle.

# Criar User Story - YouTrack

Cria uma nova User Story no YouTrack vinculada a um Epic, utilizando contexto do Epic, outras US e código fonte do projeto.

## Argumentos

- `$ARGUMENTS` - ID do Epic no YouTrack (ex: ORN-1000) OU descrição da funcionalidade desejada

## Processo

### 1. Identificar o Epic

#### Se `$ARGUMENTS` contém ID do YouTrack (ex: ORN-1000):

```
mcp__youtrack__get_issue com issueId = $ARGUMENTS
```

Verificar se o Type é "Epic". Se não for, buscar o Epic pai:
- Verificar linkedIssueCounts para encontrar "subtask of"
- Navegar até o Epic

#### Se `$ARGUMENTS` é uma descrição ou está vazio:

Perguntar ao usuário usando AskUserQuestion:
> Em qual Epic esta User Story será criada?
> Forneça o ID do Epic (ex: ORN-1000) ou o nome do projeto para buscar Epics disponíveis.

Se usuário fornecer nome do projeto, buscar Epics:
```
mcp__youtrack__search_issues com query = "project: {PROJETO} Type: Epic State: Upstream, Downstream"
```

Apresentar lista de Epics e pedir para escolher.

### 2. Buscar contexto do Epic

Extrair do Epic:
- Summary (objetivo geral)
- Description (requisitos e escopo)
- linkedIssueCounts (US já existentes)

### 3. Buscar User Stories existentes do Epic

```
mcp__youtrack__search_issues com query = "subtask of: {EPIC_ID} Type: {User Story}"
```

Para cada US encontrada, coletar:
- Summary
- Description (resumido)
- Status atual

Isso ajuda a:
- Evitar duplicação de funcionalidades
- Manter consistência de escrita
- Entender o progresso atual do Epic

### 4. Buscar contexto do código fonte (se aplicável)

Se estiver em um diretório de projeto (verificar se existe `package.json`, `go.mod`, `.git`, etc.):

```bash
git remote -v 2>/dev/null | head -1
```

Se for um projeto de código:
1. Identificar estrutura do projeto
2. Buscar arquivos relevantes para a funcionalidade
3. Usar Task tool com subagent_type=Explore se necessário

Isso ajuda a:
- Escrever US mais técnicas e precisas
- Identificar componentes existentes que podem ser reutilizados
- Estimar complexidade

### 5. Coletar informações da nova US

Perguntar ao usuário usando AskUserQuestion:

> Descreva a funcionalidade da User Story:
> - O que o usuário precisa fazer?
> - Qual problema será resolvido?
> - Há requisitos específicos?

Se `$ARGUMENTS` continha descrição, usar como base e pedir confirmação/complemento.

### 6. Buscar schema de campos do projeto

```
mcp__youtrack__get_issue_fields_schema com projectKey = {PROJECT_KEY}
```

Identificar campos obrigatórios e valores permitidos para:
- Type (deve ser "User Story")
- Priority
- Classe de serviço
- Outros campos customizados

### 7. Redigir a User Story

Criar User Story seguindo o padrão:

```markdown
## Descrição

Como [persona/usuário],
Eu quero [funcionalidade/ação],
Para que [benefício/valor].

## Contexto

[Explicação do problema ou necessidade baseada no Epic e código existente]

## Requisitos Funcionais

- [ ] [Requisito 1]
- [ ] [Requisito 2]
- [ ] [Requisito 3]

## Critérios de Aceite

- [ ] [Critério verificável 1]
- [ ] [Critério verificável 2]
- [ ] [Critério verificável 3]

## Observações Técnicas

[Se aplicável, baseado no código fonte analisado]
- Componentes afetados: [lista]
- Dependências: [lista]

## Referências

- Epic: {EPIC_ID} - {EPIC_SUMMARY}
- US relacionadas: {lista de US do mesmo Epic}
```

### 8. Apresentar rascunho para aprovação

Mostrar ao usuário:

```markdown
## Nova User Story para o Epic {EPIC_ID}

### Summary
{SUMMARY_PROPOSTO}

### Description
{DESCRIPTION_FORMATADA}

### Campos
- Type: User Story
- Priority: {SUGESTÃO}
- State: Upstream (inicial)

---

Deseja criar esta User Story?
```

Usar AskUserQuestion:
> Revisar e aprovar a User Story
> Opções:
> 1. Criar como está
> 2. Editar antes de criar
> 3. Cancelar

Se "Editar", permitir ajustes e repetir apresentação.

### 9. Criar a User Story no YouTrack

```
mcp__youtrack__create_issue com:
  project: {PROJECT_KEY}
  summary: {SUMMARY}
  description: {DESCRIPTION}
  parentIssue: {EPIC_ID}
  customFields: {
    "Type": "User Story",
    "Priority": {PRIORITY},
    ... outros campos obrigatórios
  }
```

### 10. Verificar criação

```
mcp__youtrack__get_issue com issueId = {NEW_ISSUE_ID}
```

Confirmar:
- Issue foi criada corretamente
- Está vinculada ao Epic (subtask of)
- Campos estão preenchidos

### 11. Mover para Downstream (se solicitado)

Perguntar ao usuário usando AskUserQuestion:
> A User Story está pronta para desenvolvimento?
> Opções:
> 1. Sim, mover para Downstream
> 2. Não, manter em Upstream para refinamento

Se mover para Downstream:
```
mcp__youtrack__update_issue com:
  issueId: {NEW_ISSUE_ID}
  customFields: {
    "State": "Downstream"
  }
```

Verificar se a mudança foi aplicada (seguir workflow do YouTrack - consultar skill gv-youtrack).

### 12. Resumo final

Apresentar resultado:

```markdown
## User Story Criada

- **ID:** {NEW_ISSUE_ID}
- **Summary:** {SUMMARY}
- **Epic:** {EPIC_ID} - {EPIC_SUMMARY}
- **State:** {Upstream/Downstream}
- **URL:** {YOUTRACK_URL}

### Próximos passos sugeridos:
1. Criar Tasks técnicas (Frontend, Backend, etc.)
2. Definir responsáveis
3. Adicionar na sprint/iteração
```

## Template de User Story

Use este template como base para a description:

```markdown
## Descrição

Como [persona],
Eu quero [funcionalidade],
Para que [benefício].

## Contexto

[Problema ou necessidade que motiva esta US]

## Requisitos Funcionais

- [ ] RF01: [Descrição do requisito]
- [ ] RF02: [Descrição do requisito]

## Critérios de Aceite

- [ ] CA01: [Critério verificável]
- [ ] CA02: [Critério verificável]

## Mockups/Referências

[Links ou descrições visuais, se aplicável]

## Dependências

- [US ou Epic que precisa estar pronto]
- [Serviço externo necessário]

## Observações

[Notas adicionais relevantes]
```

## Boas Práticas

1. **Summary claro**: Deve ser autoexplicativo em uma linha
2. **Critérios verificáveis**: Cada critério deve ser testável
3. **Escopo definido**: Evitar US muito grandes (quebrar em múltiplas se necessário)
4. **Contexto técnico**: Se em projeto de código, incluir observações técnicas relevantes
5. **Consistência**: Manter padrão de escrita similar às outras US do Epic

## Exemplo de Uso

```
/gv-po ORN-1000
```

Fluxo:
1. Busca Epic ORN-1000
2. Lista US existentes do Epic
3. Analisa código fonte (se no projeto)
4. Coleta descrição da nova funcionalidade
5. Redige US seguindo template
6. Apresenta para aprovação
7. Cria no YouTrack vinculada ao Epic
8. Opcionalmente move para Downstream

```
/gv-po "Permitir exportação de relatórios em PDF"
```

Fluxo:
1. Pergunta qual Epic
2. Segue mesmo processo acima usando a descrição como base

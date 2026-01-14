---
description: Desenvolve uma User Story ou Incident completo (YouTrack + GitLab)
allowed-tools: mcp__youtrack__*, mcp__gitlab__*, Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion, Task, TodoWrite
---

**IMPORTANTE:** Antes de executar, carregue a skill `gv-youtrack` para entender o workflow do YouTrack no Grupo Voalle.

# Desenvolver User Story / Incident

Comando completo para desenvolver uma User Story ou Incident, desde a análise até o envio para review.

## Argumentos

- `$ARGUMENTS` - ID da US ou Incident no YouTrack (ex: ORN-1148)

## Pré-requisitos

- Estar dentro do diretório do projeto (IDE VSCode ou terminal)
- GitLab MCP configurado (ver `/setup-gitlab-mcp`)
- YouTrack MCP configurado (ver `/setup-youtrack`)

## Processo

### 1. Validar ambiente

Verificar se está em um projeto:

```bash
# Verificar se é um repositório git
git rev-parse --is-inside-work-tree

# Obter remote do GitLab
git remote -v | grep -E "(gitlab|git\.)" | head -1

# Obter branch atual
git branch --show-current
```

Se NÃO estiver em um projeto, informar:
> Este comando deve ser executado dentro do diretório do projeto no VSCode ou terminal.

### 2. Buscar informações do Card no YouTrack

```
mcp__youtrack__get_issue com issueId = $ARGUMENTS
```

Extrair:
- Summary
- Description
- Type (User Story ou Incident)
- Status atual
- Project
- linkedIssueCounts (Tasks existentes)
- customFields relevantes

### 3. Avaliar completude da descrição

Usar Task tool com subagent_type=Explore para analisar o código existente e comparar com a descrição.

**Checklist de completude:**

- [ ] **Descrição clara**: Explica o que precisa ser feito?
- [ ] **Critérios de aceite**: Estão definidos e são verificáveis?
- [ ] **Regras de negócio**: Estão documentadas?
- [ ] **Casos de borda**: Edge cases estão considerados?
- [ ] **Dependências**: Há dependências com outras US/sistemas?
- [ ] **Mockups/Referências**: Se for UI, há referência visual?

Se faltar informações críticas, usar AskUserQuestion:
> A descrição do card {ID} parece incompleta. Faltam:
> - [item 1]
> - [item 2]
>
> Opções:
> 1. Continuar mesmo assim (vou inferir o que falta)
> 2. Pausar para completar a descrição no YouTrack
> 3. Você me fornece as informações faltantes agora

### 4. Identificar tipo de Task necessária

Analisar o projeto e a demanda para determinar:

```bash
# Identificar tipo de projeto
ls -la package.json go.mod pom.xml Cargo.toml requirements.txt 2>/dev/null
```

Usar AskUserQuestion:
> Qual tipo de Task será criada?
> Opções:
> 1. **[Task Frontend]** - Alterações em UI/componentes
> 2. **[Task Backend]** - Alterações em API/serviços
> 3. **[Task Full-stack]** - Ambos frontend e backend
> 4. **Outro** - Especificar

### 5. Buscar schema de campos do projeto YouTrack

```
mcp__youtrack__get_issue_fields_schema com projectKey = {PROJECT_KEY}
```

### 6. Criar Task no YouTrack

```
mcp__youtrack__create_issue com:
  project: {PROJECT_KEY}
  summary: "[Task {TIPO}] {SUMMARY_DA_US}"
  description: "Task de desenvolvimento para {US_ID}\n\n## Escopo\n{ESCOPO_TÉCNICO}"
  parentIssue: {US_ID}
  customFields: {
    "Type": "Task",
    "State": "Downstream"
  }
```

Verificar criação:
```
mcp__youtrack__get_issue com issueId = {TASK_ID}
```

### 7. Identificar projeto no GitLab

```bash
# Extrair project path do remote
git remote get-url origin | sed 's/.*[:/]\([^/]*\/[^/]*\)\.git/\1/'
```

Ou buscar no GitLab:
```
mcp__gitlab__search_repositories com search = {PROJECT_NAME}
```

### 8. Criar Issue no GitLab

```
mcp__gitlab__create_issue com:
  project_id: {GITLAB_PROJECT_PATH}
  title: "[{TASK_ID}] {SUMMARY}"
  description: |
    ## Referência YouTrack
    - **Task:** {TASK_ID}
    - **User Story:** {US_ID} - {US_SUMMARY}

    ## Descrição
    {DESCRIÇÃO_TÉCNICA}

    ## Critérios de Aceite
    {CRITÉRIOS_DA_US}
  labels: ["desenvolvimento"]
```

### 9. Criar Merge Request vinculado à Issue

```
mcp__gitlab__create_merge_request com:
  project_id: {GITLAB_PROJECT_PATH}
  title: "Draft: [{TASK_ID}] {SUMMARY}"
  description: |
    ## Referência
    - YouTrack Task: {TASK_ID}
    - YouTrack US: {US_ID}

    Closes #{ISSUE_IID}

    ## Mudanças
    - [ ] TODO: Listar mudanças após desenvolvimento

    ## Checklist
    - [ ] Critérios de aceite validados
    - [ ] Testes unitários
    - [ ] Testes de integração
    - [ ] Code review (auto)
    - [ ] Documentação atualizada
  source_branch: {BRANCH_NAME}
  target_branch: "main"  # ou "master", "develop" conforme projeto
  draft: true
```

**Nota:** O GitLab cria a branch automaticamente se não existir.

### 10. Criar branch e fazer checkout

A branch será criada pelo MR. Fazer checkout local:

```bash
git fetch origin
git checkout -b {BRANCH_NAME} origin/{TARGET_BRANCH} 2>/dev/null || git checkout {BRANCH_NAME}
```

Padrão de nome da branch (GitLab automático):
```
{ISSUE_IID}-{SLUG_DO_TITULO}
```

### 11. Criar TodoList para desenvolvimento

Usar TodoWrite para criar lista de tarefas baseada nos critérios de aceite:

```
- [ ] Analisar código existente relacionado
- [ ] Implementar [critério 1]
- [ ] Implementar [critério 2]
- [ ] ...
- [ ] Criar testes unitários
- [ ] Criar testes de integração
- [ ] Validar todos critérios de aceite
- [ ] Auto code review
- [ ] Atualizar documentação
- [ ] Commit e push
```

### 12. Desenvolver a demanda

Para cada critério de aceite:

1. **Analisar** código existente relacionado
2. **Implementar** a funcionalidade
3. **Testar** manualmente se possível
4. **Marcar** como concluído no TodoList

Usar Task tool com subagent_type=Explore quando precisar entender o código existente.

### 13. Validar critérios de aceite

Para cada critério de aceite da US:

```markdown
| Critério | Status | Evidência |
|----------|--------|-----------|
| CA01: ... | ✅/❌ | [como foi atendido] |
| CA02: ... | ✅/❌ | [como foi atendido] |
```

Se algum critério não puder ser atendido, usar AskUserQuestion para decidir próximos passos.

### 14. Criar testes

#### Testes Unitários

Identificar framework de teste do projeto:

```bash
# JavaScript/TypeScript
grep -E "jest|vitest|mocha" package.json

# Go
ls *_test.go 2>/dev/null

# Python
ls -la pytest.ini setup.cfg pyproject.toml 2>/dev/null
```

Criar testes unitários para:
- Funções/métodos novos
- Casos de sucesso
- Casos de erro
- Edge cases

#### Testes de Integração

Se aplicável, criar testes de integração para:
- Endpoints de API
- Fluxos completos
- Integração entre componentes

### 15. Auto Code Review

Executar análise crítica do código desenvolvido:

**Verificar:**

#### Problemas Críticos (devem ser corrigidos)
- [ ] Vulnerabilidades de segurança (SQL injection, XSS, etc.)
- [ ] Bugs óbvios (null pointer, off-by-one, etc.)
- [ ] Quebra de funcionalidades existentes
- [ ] Dados sensíveis expostos

#### Problemas Médios (devem ser corrigidos)
- [ ] Código duplicado
- [ ] Funções muito longas (>50 linhas)
- [ ] Complexidade ciclomática alta
- [ ] Falta de tratamento de erros
- [ ] Performance ruim (N+1, loops ineficientes)

#### Sugestões (avaliar)
- [ ] Nomes de variáveis/funções pouco claros
- [ ] Falta de comentários em lógica complexa
- [ ] Oportunidades de refatoração

**Processo:**
1. Listar todos os arquivos modificados
2. Analisar cada arquivo
3. Corrigir problemas críticos e médios
4. Repetir até não encontrar problemas

```bash
git diff --name-only HEAD~1
```

### 16. Atualizar documentação

Se necessário, atualizar:
- README.md
- Documentação de API (OpenAPI, Swagger)
- Comentários em código complexo
- CHANGELOG.md (se existir)

### 17. Commit e Push

```bash
# Verificar status
git status

# Adicionar arquivos
git add .

# Commit com mensagem padronizada
git commit -m "[{TASK_ID}] {DESCRIÇÃO_CURTA}

{DESCRIÇÃO_DETALHADA}

- Implementa {funcionalidade}
- Adiciona testes para {componente}
- Atualiza documentação

Refs: {US_ID}
"

# Push
git push -u origin {BRANCH_NAME}
```

### 18. Verificar CI/CD

```
mcp__gitlab__list_pipelines com:
  project_id: {GITLAB_PROJECT_PATH}
  ref: {BRANCH_NAME}
  per_page: 1
```

Se pipeline falhar:
```
mcp__gitlab__get_pipeline_job_output com:
  project_id: {GITLAB_PROJECT_PATH}
  job_id: {JOB_ID}
```

Corrigir problemas e repetir push até passar.

### 19. Atualizar MR (remover Draft)

```
mcp__gitlab__update_merge_request com:
  project_id: {GITLAB_PROJECT_PATH}
  merge_request_iid: {MR_IID}
  draft: false
  description: |
    ## Referência
    - YouTrack Task: {TASK_ID}
    - YouTrack US: {US_ID}

    Closes #{ISSUE_IID}

    ## Mudanças
    {LISTA_DE_MUDANÇAS_REAIS}

    ## Critérios de Aceite
    {CHECKLIST_VALIDADO}

    ## Testes
    - [x] Testes unitários
    - [x] Testes de integração

    ## Code Review
    - [x] Auto code review realizado
    - [x] Sem problemas críticos ou médios
```

### 20. Registrar tempo de trabalho no YouTrack

Usar AskUserQuestion:
> Quanto tempo foi gasto no desenvolvimento?
> (Sugestão baseada no tempo de sessão: {X} minutos)

```
mcp__youtrack__log_work com:
  issueId: {TASK_ID}
  durationMinutes: {MINUTOS}
  description: "Desenvolvimento completo: implementação, testes, code review"
  workType: "Implementation"
```

### 21. Mover Task para Backlog-Review

Seguir workflow do YouTrack (consultar skill gv-youtrack):

```
mcp__youtrack__update_issue com:
  issueId: {TASK_ID}
  customFields: {
    "Status": "Backlog-Review"
  }
```

Verificar se a mudança foi aplicada:
```
mcp__youtrack__get_issue com issueId = {TASK_ID}
```

### 22. Resumo final

```markdown
## Desenvolvimento Concluído

### YouTrack
- **Task:** {TASK_ID} - {TASK_SUMMARY}
- **User Story:** {US_ID} - {US_SUMMARY}
- **Status:** Backlog-Review
- **Tempo registrado:** {X} minutos

### GitLab
- **Issue:** #{ISSUE_IID}
- **MR:** !{MR_IID} - {MR_TITLE}
- **Branch:** {BRANCH_NAME}
- **Pipeline:** ✅ Passou

### Checklist Final
- [x] Código implementado
- [x] Critérios de aceite validados
- [x] Testes unitários criados
- [x] Testes de integração criados
- [x] Auto code review realizado
- [x] Documentação atualizada
- [x] CI/CD passou
- [x] Tempo registrado
- [x] Task em Backlog-Review

### Próximos passos
1. Aguardar code review do time
2. Responder comentários do MR
3. Após aprovação, fazer merge
```

## Notas Importantes

1. **Qualidade primeiro**: Não pule etapas de teste e review
2. **Commits atômicos**: Prefira commits pequenos e focados
3. **Documentação**: Atualize sempre que necessário
4. **Workflow YouTrack**: Sempre verifique se as transições foram aplicadas
5. **CI/CD**: Nunca envie para review com pipeline falhando

## Exemplo de Uso

```
/gv-dev ORN-1148
```

Fluxo completo:
1. Valida ambiente (está no projeto)
2. Busca US ORN-1148 no YouTrack
3. Avalia completude da descrição
4. Cria Task [Task Frontend] ORN-1160
5. Cria Issue #50 no GitLab
6. Cria MR !104 vinculado à Issue
7. Cria branch 50-task-frontend-orn-1148
8. Desenvolve a demanda
9. Valida critérios de aceite
10. Cria testes unitários e integração
11. Faz auto code review
12. Atualiza documentação
13. Commit, push, verifica CI
14. Registra 120 minutos no YouTrack
15. Move Task para Backlog-Review

---
description: Review completo de User Story (YouTrack) e Merge Request (GitLab)
allowed-tools: mcp__youtrack__*, mcp__gitlab__*, Bash, Read, Grep, Glob, AskUserQuestion
---

**IMPORTANTE:** Antes de executar, carregue a skill `gv-youtrack` para entender o workflow do YouTrack no Grupo Voalle.

# Review Completo - US + MR

Realiza um review completo que analisa tanto a User Story no YouTrack quanto o Merge Request no GitLab, verificando se a implementação atende aos requisitos.

## Argumentos

- `$ARGUMENTS` - ID da US no YouTrack (ex: ORN-1148) OU IID do MR no GitLab (ex: !104 ou 104)

## Instruções

### 1. Identificar o tipo de entrada

Analisar `$ARGUMENTS`:
- Se começa com letras e contém hífen (ex: `ORN-1148`): É um ID do YouTrack
- Se começa com exclamação ou é apenas número, como 104 ou !104: É um MR do GitLab
- Se vazio: Extrair ID da branch atual (ex: `41-epic-fundacao...` → `41`, `orn-1148-feature` → `ORN-1148`)
- Se não estiver commitado ou sincronizado com o remoto, fazer code-review do que ainda não está no remoto. (Pergunte para o usuário se a review é exclusivo do código não sincronizado)

### 2. Buscar informações completas

#### Se entrada é ID do YouTrack:

1. Buscar US no YouTrack:
```
mcp__youtrack__get_issue com issueId = $ARGUMENTS
```

2. Buscar MR relacionado no GitLab:
```
mcp__gitlab__list_merge_requests com search = $ARGUMENTS
```

#### Se entrada é MR do GitLab:

1. Buscar MR no GitLab:
```
mcp__gitlab__get_merge_request com merge_request_iid = $ARGUMENTS
```

2. Extrair ID da US do título ou branch do MR (ex: `ORN-1148` no título)

3. Buscar US no YouTrack:
```
mcp__youtrack__get_issue com issueId = {ID extraído}
```

#### Se entrada é vazia:

1. Obter branch atual:
```bash
git branch --show-current
```

2. Extrair ID do YouTrack do nome da branch:
   - Padrão `{ID}-descricao` (ex: `41-epic-fundacao` → buscar issue `41`)
   - Padrão `{PROJETO}-{NUMERO}` (ex: `orn-1148-feature` → buscar `ORN-1148`)
   - Usar regex para extrair: `^(\d+)-` ou `([a-zA-Z]+-\d+)`

3. Buscar MR pela branch:
```
mcp__gitlab__get_merge_request com source_branch = {branch}
```

4. Buscar US no YouTrack com o ID extraído

### 3. Obter diffs do MR

```
mcp__gitlab__get_merge_request_diffs ou mcp__gitlab__list_merge_request_diffs
```

### 4. Review da User Story

Analisar a US no YouTrack:

**Completude dos requisitos:**
- [ ] Description está clara e completa?
- [ ] Critérios de aceite estão definidos?
- [ ] Há dependências não resolvidas?

**Rastreabilidade:**
- [ ] Tasks relacionadas estão vinculadas?
- [ ] MR está referenciado na US?

### 5. Review do Código (MR)

Para cada arquivo modificado, analise:

**Aderência aos requisitos:**
- O código implementa o que a US pede?
- Há funcionalidades faltando?
- Há funcionalidades extras não solicitadas?

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

### 6. Verificar discussões existentes

```
mcp__gitlab__mr_discussions
```

Ver se há comentários/discussões pendentes no MR.

### 7. Apresentar o review completo

```markdown
## Review Completo - {US_ID}: {US_TITLE}

### User Story
- **ID:** {US_ID}
- **Status:** {STATUS}
- **URL:** {YOUTRACK_URL}

### Merge Request
- **MR:** !{MR_IID} - {MR_TITLE}
- **Status:** {MR_STATE}
- **URL:** {GITLAB_URL}

---

## Análise da User Story

### Requisitos Identificados
1. [Requisito extraído da description]
2. [Requisito extraído da description]

### Critérios de Aceite
- [ ] [Critério 1]
- [ ] [Critério 2]

---

## Análise do Código

### Aderência aos Requisitos

| Requisito | Implementado | Observação |
|-----------|--------------|------------|
| Requisito 1 | ✅ Sim / ❌ Não / ⚠️ Parcial | Detalhes |
| Requisito 2 | ✅ Sim / ❌ Não / ⚠️ Parcial | Detalhes |

### Pontos Positivos
- [Aspectos bem implementados]

### Problemas Encontrados

#### Críticos (Bloqueiam aprovação)
- [ ] **[arquivo:linha]** - Descrição
  - Sugestão de correção

#### Importantes (Devem ser corrigidos)
- [ ] **[arquivo:linha]** - Descrição
  - Sugestão de correção

#### Sugestões (Nice to have)
- [ ] **[arquivo:linha]** - Descrição

---

## Veredicto

### User Story
[COMPLETA / INCOMPLETA / PRECISA REFINAMENTO]

### Merge Request
[APROVAR / SOLICITAR MUDANÇAS / BLOQUEAR]

### Justificativa
[Explicação do veredicto considerando US + MR]
```

### 8. Perguntar sobre próximos passos

Usar AskUserQuestion para perguntar:

> O que deseja fazer agora?

Opções:
1. **Adicionar comentários no GitLab** - Criar threads de discussão nos arquivos
2. **Atualizar status no YouTrack** - Mover card para próximo status
3. **Ambos** - Comentários no GitLab + atualizar YouTrack
4. **Nenhum** - Apenas visualizar o review

Se escolher adicionar comentários no GitLab:
- Usar `mcp__gitlab__create_merge_request_thread` para problemas críticos e importantes

Se escolher atualizar YouTrack:
- Seguir workflow correto (consultar skill gv-youtrack)
- Registrar tempo de trabalho (workType: "Code Review")
- Verificar se mudança foi aplicada

## Notas

- Seja construtivo e educativo nos comentários
- Foque em problemas reais, não em preferências pessoais de estilo
- Considere o contexto do projeto ao fazer sugestões
- Priorize: Segurança > Bugs > Performance > Estilo
- Sempre verifique se o código atende aos requisitos da US

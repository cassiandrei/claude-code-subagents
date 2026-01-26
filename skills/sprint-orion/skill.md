# Sprint Órion Dashboard

Gera um dashboard HTML interativo com o progresso da Sprint do projeto EV Orion (ORN).

## Quando Usar

- `/sprint-orion` - Gerar dashboard da sprint atual
- `/sprint-orion [nome-da-sprint]` - Dashboard de sprint específica

## Fluxo de Execução

### 1. Identificar Sprint

Se nenhum nome de sprint for fornecido, use `State: Downstream` para buscar issues em desenvolvimento ativo.

Se um nome for fornecido, use na query:
```
project: {EV Orion} Sprint: {nome-da-sprint}
```

### 2. Calcular Progresso da Missão Órion

O progresso da sprint é baseado em issues com a **tag "Missão órion"** (excluindo Tasks e Landing):

```
# Total de issues da missão (excluindo Tasks e Landing)
project: {EV Orion} tag: {Missão órion} Type: -{Task} State: -{Landing}

# Issues concluídas
project: {EV Orion} tag: {Missão órion} Type: -{Task} State: -{Landing} #Resolved

# Issues não concluídas
project: {EV Orion} tag: {Missão órion} Type: -{Task} State: -{Landing} #Unresolved
```

**Fórmula:** `Progresso = (Resolved / Total) * 100`

**Nota:** Issues em Landing (Production) não são consideradas pois já foram entregues em sprints anteriores.

### 3. Buscar Issues no YouTrack (Kanban)

Use o MCP YouTrack para buscar as issues em desenvolvimento ativo:

```
# Query principal - Issues em Downstream
project: {EV Orion} State: Downstream

# Ou com sprint específica
project: {EV Orion} Sprint: {Gemini Program (EUA, 1965–1966) – Ensaios de acoplamento e caminhadas espaciais.}
```

Campos necessários para cada issue:
- `idReadable` - ID da issue (ex: ORN-1148)
- `summary` - Título
- `Status` - Status atual (campo customizado)
- `Atendente` - Responsável (campo customizado)
- `Type` - Tipo (Epic, User Story, Task)
- `updated` - Data da última atualização

### 4. Agrupar por Status (Kanban)

O Kanban mostra apenas issues de alto nível (sem Tasks).

**Tipos INCLUÍDOS no Kanban:**
- Epic, User Story, Incident, Tech Story, Automation, Quality Assurance

**Tipos EXCLUÍDOS do Kanban:**
- Task (sub-itens das User Stories)

**Categorias de status:**

| Categoria | Status incluídos |
|-----------|------------------|
| Backlog | Backlog |
| In Progress | In Progress DEV, In Progress-Review, In Progress-QA, Waiting for Validation |
| Waiting | Backlog-Review, Backlog-Fixing-DEV, Backlog-QA, Waiting for Milestone, Waiting For Packaging |
| Done | Done, Closed |

### 5. Identificar Alertas

Buscar issues paradas há 2+ **dias úteis** nos status de trabalho ativo:

```
# Paradas há 4+ dias corridos (para cobrir fins de semana)
project: {EV Orion} State: Downstream Status: {In Progress DEV}, {Backlog-Fixing-DEV} updated: * .. {minus 4d}
```

**IMPORTANTE:** A contagem de dias parados deve considerar apenas **dias úteis** (segunda a sexta). Sábados e domingos não contam.

**Algoritmo para calcular dias úteis:**
```python
def dias_uteis(data_atualizacao, data_atual):
    dias = 0
    data = data_atualizacao + 1 dia
    while data <= data_atual:
        if data.dia_da_semana não é sábado nem domingo:
            dias += 1
        data += 1 dia
    return dias
```

**Critérios:**
- Status: `In Progress DEV` ou `Backlog-Fixing-DEV`
- Última atualização: mais de 2 **dias úteis** atrás
- Excluir Tasks (mostrar apenas User Story, Epic, Incident, Tech Story, Automation)
- Ordenar por dias úteis parado (mais antigo primeiro)

### 6. Novos Incidentes / Apoios Técnicos

Buscar issues do tipo Incident ou Technical Support criadas no último dia útil ou hoje.

**IMPORTANTE:** Se hoje for segunda-feira, deve buscar desde sexta-feira. Se for terça a domingo, buscar desde ontem.

```python
# Lógica para determinar a data inicial
if hoje.dia_da_semana == segunda:
    data_inicial = sexta-feira passada  # {minus 3d}
elif hoje.dia_da_semana == domingo:
    data_inicial = sexta-feira passada  # {minus 2d}
elif hoje.dia_da_semana == sabado:
    data_inicial = sexta-feira (ontem)  # {minus 1d}
else:
    data_inicial = ontem  # {minus 1d}
```

**Queries:**
```
# Se hoje for segunda-feira (busca desde sexta)
project: {EV Orion} Type: Incident, {Technical Support} created: {minus 3d} .. today

# Se hoje for terça a sexta (busca desde ontem)
project: {EV Orion} Type: Incident, {Technical Support} created: yesterday .. today
```

**Fluxo:**
1. Verificar o dia da semana atual
2. Ajustar a query para incluir o último dia útil
3. Buscar issues via `search_issues`
4. Se não houver issues, exibir mensagem "Nenhum novo incidente ou apoio técnico registrado"
5. Se houver issues, exibir cards com informações básicas

**Campos exibidos:**
- ID da issue
- Título
- Tipo (Incident ou Technical Support)
- Status atual
- Atendente (se houver)
- Data de criação

**Nota:** Esta seção ajuda a identificar rapidamente novos problemas que chegaram recentemente e precisam de atenção.

### 7. Issues Bloqueadas

Buscar issues com status "Blocked" (excluindo Tasks):

```
project: {EV Orion} State: Downstream Status: Blocked Type: -{Task}
```

**Fluxo:**
1. Buscar issues bloqueadas via `search_issues`
2. Para cada issue, buscar comentários via `get_issue_comments`
3. O motivo do bloqueio geralmente está no comentário mais recente
4. Exibir seção acima das "Mudanças de Status"

**Campos exibidos:**
- ID da issue
- Título
- Tipo (User Story, Epic, etc.)
- Motivo do bloqueio (do comentário)
- Atendente (se houver)
- Data do bloqueio

### 8. Mudanças de Status do Último Dia Útil com Atividade

Buscar mudanças de status via API REST do YouTrack (Activity Stream).

**IMPORTANTE:** Buscar o último dia útil que TEVE atividade de mudança de status, não apenas o último dia útil do calendário. Se sexta-feira não teve movimentações, mostrar as de quinta-feira.

```bash
# Endpoint para buscar atividades de uma issue
GET /api/issues/{issueId}/activities?fields=timestamp,author(login,name),added(name),removed(name),field(name)&categories=CustomFieldCategory

# Filtrar apenas mudanças de Status (field.name == "Status")
```

**Fluxo:**
1. Buscar issues atualizadas na última semana: `updated: {This week}`
2. Para cada issue encontrada, buscar activities via API REST
3. Filtrar mudanças onde `field.name == "Status"`
4. Agrupar mudanças por data
5. Identificar o último dia com atividade (pode ser ontem, anteontem, etc.)
6. Exibir apenas as mudanças desse dia
7. Ordenar por hora (mais recente primeiro)

**Algoritmo para encontrar o último dia com atividade:**
```python
# 1. Coletar todas as mudanças de status da semana
# 2. Extrair as datas únicas
# 3. Ordenar datas em ordem decrescente
# 4. Pegar a primeira data (mais recente)
# 5. Filtrar apenas mudanças dessa data
```

**Campos exibidos:**
- ID da issue
- Título (truncado)
- Transição: `status anterior → status novo`
- Autor da mudança
- Hora da mudança
- Data no header (ex: "23/01/2026 (Quinta-feira) - 6 mudanças")

**Token de API:**
O token está configurado em `~/.claude/settings.json` no objeto `mcpServers.youtrack.headers.Authorization`.

### 9. Gerar HTML

Use o template abaixo para gerar o dashboard. Substitua os placeholders pelos dados reais.

**Arquivo de saída:** `~/Projects/GrupoVoalle/Elleven/docs/sprints/sprint-orion-dashboard.html`

### 10. Abrir no Navegador

```bash
open ~/Projects/GrupoVoalle/Elleven/docs/sprints/sprint-orion-dashboard.html
```

## Template HTML

O template está em `~/.claude/skills/sprint-orion/template.html`.

Substitua os seguintes placeholders:

| Placeholder | Descrição |
|-------------|-----------|
| `{{SPRINT_NAME}}` | Nome da sprint |
| `{{GENERATED_AT}}` | Data/hora de geração |
| `{{TOTAL_ISSUES}}` | Total de issues |
| `{{BACKLOG_COUNT}}` | Issues em backlog |
| `{{PROGRESS_COUNT}}` | Issues em progresso |
| `{{DONE_COUNT}}` | Issues concluídas |
| `{{PROGRESS_PERCENT}}` | Percentual de conclusão |
| `{{BACKLOG_CARDS}}` | HTML dos cards de backlog |
| `{{PROGRESS_CARDS}}` | HTML dos cards em progresso |
| `{{WAITING_CARDS}}` | HTML dos cards aguardando |
| `{{DONE_CARDS}}` | HTML dos cards concluídos |
| `{{NEW_INCIDENTS_COUNT}}` | Quantidade de novos incidentes/apoios |
| `{{NEW_INCIDENTS_CARDS}}` | HTML dos cards de novos incidentes |
| `{{ALERTS_SECTION}}` | HTML dos alertas |
| `{{MOVEMENTS_DATE}}` | Data do dia útil anterior |
| `{{MOVEMENTS_ITEMS}}` | HTML dos itens de movimentação |

### Formato de Card

```html
<div class="issue-card type-{{TYPE_LOWER}}">
    <div class="issue-header">
        <span class="issue-id">{{ISSUE_ID}}</span>
        <span class="issue-type">{{TYPE}}</span>
    </div>
    <div class="issue-title">{{SUMMARY}}</div>
    <div class="issue-footer">
        <span class="issue-assignee">{{ATENDENTE}}</span>
        <span class="issue-updated">{{UPDATED}}</span>
    </div>
</div>
```

## Exemplo de Uso

```
Usuário: /sprint-orion

Claude:
1. Busca issues: project: {EV Orion} State: Downstream
2. Agrupa por status
3. Identifica alertas
4. Busca novos incidentes/apoios técnicos (ontem/hoje)
5. Busca issues bloqueadas
6. Busca issues atualizadas na semana
7. Para cada issue, busca activities via API REST
8. Identifica o último dia COM mudanças de status
9. Exibe apenas as mudanças desse dia
10. Gera HTML com dados
11. Salva em docs/sprints/sprint-orion-dashboard.html
12. Abre no navegador
```

## Nomenclatura de Sprints (Programas Espaciais)

| Sprint | Programa |
|--------|----------|
| Sprint 1 | Sputnik Program (URSS, 1957-1961) |
| Sprint 2 | Mercury Program (EUA, 1958-1963) |
| Sprint 3 | Vostok Program (URSS, 1961-1963) |
| Sprint 4 | Gemini Program (EUA, 1965-1966) |
| Sprint 5 | Apollo Program (EUA, 1967-1972) |
| Sprint 6 | Skylab (EUA, 1973-1979) |
| Sprint 7 | Space Shuttle (EUA, 1981-2011) |
| Sprint 8 | Mir (URSS/Rússia, 1986-2001) |
| Sprint 9 | ISS (Internacional, 1998-presente) |
| Sprint 10 | Artemis Program (EUA, 2022-presente) |

## Configurações

- **URL YouTrack:** `https://grupovoalle.youtrack.cloud/issue/{ID}`
- **Projeto:** EV Orion (ORN)

## Referências

- Skill `gv-youtrack` para workflow e transições de status
- MCP YouTrack: `search_issues`, `get_issue`

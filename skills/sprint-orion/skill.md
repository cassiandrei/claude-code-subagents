# Sprint Órion Dashboard

Gera um dashboard HTML interativo com o progresso da Sprint do projeto EV Orion (ORN).

## Quando Usar

- `/sprint-orion` - Gerar dashboard da sprint atual
- `/sprint-orion [nome-da-sprint]` - Dashboard de sprint específica

## Fluxo de Execução

**⚠️ REGRA FUNDAMENTAL:** O dashboard deve ser **REGENERADO COMPLETAMENTE** a cada execução. TODOS os dados devem ser buscados em tempo real via API YouTrack:
- Total de issues da Missão Órion
- Issues por status (Kanban)
- Alertas de issues paradas
- Novos incidentes
- Mudanças de status do dia útil anterior

**NUNCA** reutilizar dados antigos ou fazer apenas atualizações parciais. O HTML deve refletir o estado ATUAL do YouTrack.

### 1. Identificar Sprint

Se nenhum nome de sprint for fornecido, use `State: Downstream` para buscar issues em desenvolvimento ativo.

Se um nome for fornecido, use na query:
```
project: {EV Orion} Sprint: {nome-da-sprint}
```

### 2. Calcular Progresso da Missão Órion

O progresso da sprint é baseado em issues com a **tag "Missão órion"** (excluindo Tasks).

**⚠️ IMPORTANTE:** SEMPRE buscar o número REAL de issues via API. NUNCA usar números hardcoded.

**Queries:**
```
# Total de issues da missão (excluindo Tasks)
project: {EV Orion} tag: {Missão órion} Type: -{Task}

# Issues com sucesso = Waiting For Packaging ou Done ou Landing
project: {EV Orion} tag: {Missão órion} Type: -{Task} Status: {Waiting For Packaging}, Done
project: {EV Orion} tag: {Missão órion} Type: -{Task} State: Landing
```

**⚠️ PAGINAÇÃO OBRIGATÓRIA:** A API retorna no máximo 20 issues por página. Se `hasNextPage: true`, DEVE buscar as próximas páginas com `offset: 20`, `offset: 40`, etc., até `hasNextPage: false`.

**Fluxo correto:**
1. Buscar primeira página (offset: 0)
2. Se `hasNextPage: true`, buscar próxima página (offset: 20)
3. Repetir até `hasNextPage: false`
4. Somar todas as issues encontradas = **Total real**

**Fórmula:** `Progresso = (Finalizadas / Total) * 100`

**Critério de Sucesso (Finalizado):**
- **Status: Waiting For Packaging** - Issues aprovadas prontas para deploy (critério principal)
- **Status: Done** - Issues já em produção
- **State: Landing** - Issues já entregues em produção

**Nota:** O critério principal de sucesso da sprint é quando a issue atinge "Waiting For Packaging", indicando que está pronta para ser empacotada e entrar em produção.

**Nota:** Tasks são excluídas do cálculo pois são sub-itens das User Stories.

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
| Waiting | Backlog-Review, Backlog-Fixing-DEV, Backlog-QA, Waiting for Milestone |
| Done | Waiting For Packaging, Done, Closed |

**Nota:** "Waiting For Packaging" é o critério de sucesso da sprint - quando uma issue atinge este status, ela é considerada concluída para fins de progresso.

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

### 8. Mudanças de Status do Dia Útil Anterior

Buscar mudanças de status via API REST do YouTrack (Activity Stream).

**⚠️ REGRA OBRIGATÓRIA:** SEMPRE exibir as mudanças do **DIA ÚTIL ANTERIOR**, nunca do dia atual. O objetivo é mostrar o que aconteceu ontem (ou no último dia útil se hoje for segunda-feira).

**Lógica para determinar o dia útil anterior:**
```python
if hoje.dia_da_semana == segunda:
    dia_alvo = sexta-feira passada  # 3 dias atrás
elif hoje.dia_da_semana == domingo:
    dia_alvo = sexta-feira passada  # 2 dias atrás
elif hoje.dia_da_semana == sabado:
    dia_alvo = sexta-feira (ontem)  # 1 dia atrás
else:
    dia_alvo = ontem  # 1 dia atrás
```

**Exemplo prático:**
- Se hoje é **quarta-feira 29/01**, mostrar mudanças de **terça-feira 28/01**
- Se hoje é **segunda-feira 27/01**, mostrar mudanças de **sexta-feira 24/01**

```bash
# Endpoint para buscar atividades de uma issue
GET /api/issues/{issueId}/activities?fields=timestamp,author(login,name),added(name),removed(name),field(name)&categories=CustomFieldCategory

# Filtrar apenas mudanças de Status (field.name == "Status")
# E filtrar pela DATA específica do dia útil anterior
```

**Fluxo:**
1. Calcular qual é o dia útil anterior (usando a lógica acima)
2. Buscar issues atualizadas na última semana: `updated: {This week}`
3. Para cada issue encontrada, buscar activities via API REST
4. Filtrar mudanças onde `field.name == "Status"` E `data == dia_útil_anterior`
5. Ordenar por hora (mais recente primeiro)

**Script para buscar mudanças de status:**
```bash
# Para cada issue, filtrar apenas mudanças do dia útil anterior
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://grupovoalle.youtrack.cloud/api/issues/$ISSUE_ID/activities?fields=timestamp,author(login,name),added(name),removed(name),field(name)&categories=CustomFieldCategory" | \
  python3 -c "
import json, sys
from datetime import datetime
data = json.load(sys.stdin)
for item in data:
    if item.get('field', {}).get('name') == 'Status':
        ts = item.get('timestamp', 0) / 1000
        dt = datetime.fromtimestamp(ts)
        if dt.strftime('%Y-%m-%d') == 'DATA_ALVO':  # Ex: '2026-01-28'
            removed = item.get('removed', [{}])[0].get('name', '-') if item.get('removed') else '-'
            added = item.get('added', [{}])[0].get('name', '-') if item.get('added') else '-'
            author = item.get('author', {}).get('name', 'Desconhecido')
            print(f'{dt.strftime(\"%H:%M\")} | {removed} -> {added} | {author}')
"
```

**⚠️ REGRA OBRIGATÓRIA - Campos exibidos:**
- ID da issue
- **Título da issue (OBRIGATÓRIO)** - Deve ser buscado via `get_issue` ou do cache de issues já buscadas. NUNCA omitir o título.
- Transição: `status anterior → status novo`
- Autor da mudança
- Hora da mudança
- Data no header (ex: "28/01/2026 (Terça-feira) - 24 mudanças")

**IMPORTANTE:** O título da issue é OBRIGATÓRIO em cada item de mudança de status. Se o título não estiver disponível no cache, buscar via API antes de renderizar. O usuário precisa saber QUAL issue mudou, não apenas o ID.

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

**IMPORTANTE: O status da issue DEVE SEMPRE ser exibido em TODOS os cards do Kanban.** Isso é obrigatório porque cada coluna agrupa múltiplos status diferentes, e o usuário precisa saber exatamente em qual status cada issue está.

```html
<div class="issue-card" onclick="window.open('https://grupovoalle.youtrack.cloud/issue/{{ISSUE_ID}}', '_blank')">
    <div class="issue-header">
        <span class="issue-id">{{ISSUE_ID}}</span>
        <span class="issue-type type-{{TYPE_LOWER}}">{{TYPE}}</span>
    </div>
    <div class="issue-title">{{SUMMARY}}</div>
    <!-- STATUS É OBRIGATÓRIO EM TODOS OS CARDS -->
    <span class="issue-status status-{{STATUS_CLASS}}">{{STATUS}}</span>
    <div class="issue-footer">
        <span class="issue-assignee">{{ATENDENTE}}</span>
        <span class="issue-updated">{{UPDATED}}</span>
    </div>
</div>
```

**Classes de status disponíveis:**

| Status | Classe CSS |
|--------|------------|
| Backlog | `status-backlog` |
| In Progress DEV | `status-in-progress` |
| In Progress-Review | `status-review` |
| In Progress-QA | `status-qa` |
| Backlog-Review | `status-review` |
| Backlog-Fixing-DEV | `status-fixing` |
| Backlog-QA | `status-qa` |
| Waiting for Validation | `status-validation` |
| Waiting for Milestone | `status-waiting` |
| Waiting For Packaging | `status-packaging` |
| Blocked | `status-blocked` |

**Regra:** NUNCA gere um card sem o campo `issue-status`. Se o status não estiver disponível, use "Backlog" como padrão.

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

## Nomenclatura de Sprints (Missões Espaciais)

### Década de 1950–1960 (Início da Corrida Espacial)

| Sprint | Missão | Status |
|--------|--------|--------|
| 1 | Sputnik 1 (URSS, 1957) – Primeiro satélite artificial da Terra | Finalizada |
| 2 | Sputnik 2 (URSS, 1957) – Levou a cadela Laika, primeiro ser vivo em órbita | Finalizada |
| 3 | Explorer 1 (EUA, 1958) – Primeiro satélite americano | Finalizada |
| 4 | Luna 2 (URSS, 1959) – Primeira sonda a impactar a Lua | Finalizada |
| 5 | Luna 3 (URSS, 1959) – Primeiras fotos do lado oculto da Lua | Finalizada |
| 6 | Vostok 1 (URSS, 1961) – Yuri Gagarin, primeiro humano no espaço | Finalizada |
| 7 | Mercury-Redstone 3 / Freedom 7 (EUA, 1961) – Alan Shepard, primeiro americano no espaço | Finalizada |
| 8 | Mariner 2 (EUA, 1962) – Primeira sonda a sobrevoar outro planeta (Vênus) | Finalizada |
| 9 | Voskhod 1 (URSS, 1964) – Primeira missão tripulada com mais de um astronauta | Finalizada |
| 10 | Voskhod 2 (URSS, 1965) – Primeiro passeio espacial (Alexei Leonov) | Finalizada |
| 11 | Gemini Program (EUA, 1965–1966) – Ensaios de acoplamento e caminhadas espaciais | **Atual** |
| 12 | Luna 9 (URSS, 1966) – Primeiro pouso suave na Lua | Planejada |
| 13 | Surveyor 1 (EUA, 1966) – Primeiro pouso suave dos EUA na Lua | Planejada |
| 14 | Apollo 8 (EUA, 1968) – Primeira tripulação a orbitar a Lua | Planejada |
| 15 | Apollo 11 (EUA, 1969) – Neil Armstrong e Buzz Aldrin, primeiros humanos na Lua | Planejada |

### Década de 1970

| Sprint | Missão | Status |
|--------|--------|--------|
| 16 | Lunokhod 1 (URSS, 1970) – Primeiro rover a operar na Lua | Planejada |
| 17 | Salyut 1 (URSS, 1971) – Primeira estação espacial | Planejada |
| 18 | Pioneer 10 (EUA, 1972) – Primeira sonda a atravessar o cinturão de asteroides | Planejada |
| 19 | Skylab (EUA, 1973) – Primeira estação espacial americana | Planejada |
| 20 | Viking 1 (EUA, 1976) – Primeiro pouso bem-sucedido em Marte | Planejada |
| 21 | Voyager 1 e 2 (EUA, 1977) – Exploração de Júpiter, Saturno, Urano, Netuno e além | Planejada |

### Década de 1980–1990

| Sprint | Missão | Status |
|--------|--------|--------|
| 22 | Columbia STS-1 (EUA, 1981) – Primeiro voo do Ônibus Espacial | Planejada |
| 23 | Mir (URSS, 1986) – Estação espacial modular de longa duração | Planejada |
| 24 | Magellan (EUA, 1989) – Mapeamento de Vênus por radar | Planejada |
| 25 | Hubble Space Telescope (NASA/ESA, 1990) – Telescópio em órbita | Planejada |
| 26 | Mars Pathfinder (EUA, 1997) – Primeiro rover em Marte (Sojourner) | Planejada |
| 27 | Cassini-Huygens (NASA/ESA/ASI, 1997) – Estudo de Saturno e pouso em Titã | Planejada |

### Década de 2000

| Sprint | Missão | Status |
|--------|--------|--------|
| 28 | International Space Station (1998–presente) – Maior laboratório orbital | Planejada |
| 29 | Mars Odyssey, Spirit e Opportunity (EUA, 2001–2004) – Rovers em Marte | Planejada |
| 30 | Mars Express (ESA, 2003) – Missão europeia a Marte | Planejada |
| 31 | Hayabusa (JAXA, 2003) – Primeira missão a trazer amostras de asteroide | Planejada |
| 32 | New Horizons (EUA, 2006) – Primeira missão a Plutão | Planejada |
| 33 | Chang'e 1 (China, 2007) – Primeira sonda lunar chinesa | Planejada |

### Década de 2010

| Sprint | Missão | Status |
|--------|--------|--------|
| 34 | Curiosity Rover (EUA, 2011) – Grande laboratório robótico em Marte | Planejada |
| 35 | Rosetta/Philae (ESA, 2014) – Primeira aterrissagem em um cometa | Planejada |
| 36 | Chang'e 3 (China, 2013) – Rover Yutu na Lua | Planejada |
| 37 | Juno (EUA, 2016) – Missão a Júpiter | Planejada |
| 38 | Falcon Heavy Demo (SpaceX, 2018) – Foguete reutilizável pesado | Planejada |

### Década de 2020

| Sprint | Missão | Status |
|--------|--------|--------|
| 39 | Mars 2020 / Perseverance & Ingenuity (EUA, 2020) – Rover e primeiro helicóptero em Marte | Planejada |
| 40 | Artemis I (EUA, 2022) – Teste não tripulado do programa de retorno à Lua | Planejada |
| 41 | JUICE (ESA, 2023) – Missão para estudar luas de Júpiter | Planejada |
| 42 | Chandrayaan-3 (Índia, 2023) – Primeiro pouso indiano bem-sucedido na Lua | Planejada |
| 43 | Starship Test Flights (SpaceX, 2023–) – Testes da maior nave espacial já construída | Planejada |

## Configurações

- **URL YouTrack:** `https://grupovoalle.youtrack.cloud/issue/{ID}`
- **Projeto:** EV Orion (ORN)

## Referências

- Skill `gv-youtrack` para workflow e transições de status
- MCP YouTrack: `search_issues`, `get_issue`

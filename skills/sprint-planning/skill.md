# Sprint Planning Tool - EV Orion

Ferramenta interativa para planejamento de sprints do projeto EV Orion (ORN).

## Quando Usar

- `/sprint-planning` - Gerar dashboard de planejamento da sprint atual
- `/sprint-planning [nome-da-sprint]` - Dashboard para sprint especifica

## Fluxo de Execucao

### 1. Buscar Issues em Andamento

Issues que ja sairam do Backlog (em desenvolvimento ativo):

```
project: {EV Orion} State: Downstream Status: -{Backlog}, -{Done}, -{Waiting For Packaging}, -{Waiting for Milestone} #Unresolved
```

### 2. Buscar Backlog Disponivel

#### Incidents Pendentes
```
project: {EV Orion} Type: Incident Status: Backlog
```

### 3. Buscar Issues com Tag Prioridade

Para ordenacao, buscar issues com a tag "Prioridade":
```
project: {EV Orion} tag: {Prioridade}
```

### 4. Escala de Peso (Fibonacci)

| Peso | Horas (Senior) | Dias (Senior) | Criterio |
|------|----------------|---------------|----------|
| 1 | 1h | ~1h | Trivial |
| 2 | 2h | ~2h | Simples |
| 3 | 3h | ~3h | Pequeno |
| 4 | 5h | ~5h | Medio- |
| 5 | 8h | 1 dia | Medio |
| 6 | 13h | ~1.5 dias | Medio+ |
| 7 | 21h | ~2.5 dias | Grande |
| 8 | 34h | ~4 dias | Complexo |
| 9 | 55h | ~7 dias | Muito Grande |
| 10 | 89h | ~11 dias | Epico |

**Fatores por senioridade:**
- Senior: 1x (referencia)
- Pleno: 1.5x mais tempo
- Junior: 2.5x mais tempo

### 5. Calcular Capacidade do Time

A capacidade e editavel pelo usuario diretamente no dashboard.

**Capacidade Padrao:** 62 pontos (8 dias uteis, 20% buffer)

### 5. Gerar Dashboard Interativo

O dashboard permite manipulacao em tempo real:

#### Funcionalidades
- **Adicionar issues** do backlog a sprint
- **Remover issues** da sprint
- **Editar peso** de qualquer issue (click no numero)
- **Criar novo card** (gera pendencia para criacao no YouTrack)
- **Capacidade editavel** (input no header)
- **Recalcular capacidade** automaticamente a cada mudanca
- **Alertas visuais** se ultrapassar capacidade

#### Ordenacao de Cards

Os cards sao ordenados por:
1. **Tag Prioridade** - Issues com tag vem primeiro (indicador vermelho)
2. **Classe de Servico (Type)** - Ordem de prioridade:
   - Incident (1)
   - User Story (2)
   - Tech Story (3)
   - Automation (4)
   - Quality Assurance (5)
3. **Data Repactuada** - ASC (mais antiga primeiro)

#### Filtros do Backlog

- **Todos** - Todas as issues
- **Incident** - Apenas incidents
- **Tech Support** - Apenas Technical Support
- **User Story** - Apenas User Stories
- **Tech Story** - Apenas Tech Stories

#### Indicadores Visuais

- **Estrela vermelha** - Issue com tag "Prioridade"
- **Data vermelha** - Data repactuada vencida
- **Borda roxa** - Card novo (ainda nao criado no YouTrack)
- **Badge verde** - Peso editado
- **Badge amarelo** - Peso sugerido pela IA

#### Layout
```
+------------------------------------------------------------------+
| Sprint Planning - Gemini Program          Capacidade: 35/[43] pts |
+------------------------------------------------------------------+
| [####################__________] 81% utilizado                    |
+----------------------------+-------------------------------------+
| SPRINT ATUAL               |  BACKLOG DISPONIVEL                 |
|                            |  [+ Novo Card]                      |
| +------------------------+ |  [Todos] [Incident] [User Story]... |
| | *ORN-1125        [5]   | |                                     |
| | Historico Conv.        | |  +------------------------+         |
| | alexei                 | |  | *ORN-1200        [?]   |         |
| | 07/07/2025 (vencida)   | |  | Nova Feature           |         |
| | [Remover]              | |  | [+ Adicionar]          |         |
| +------------------------+ |  +------------------------+         |
+----------------------------+-------------------------------------+
| ALTERACOES PENDENTES                                             |
| * ORN-1125: Peso 3 -> 5                                          |
| * ORN-1200: Adicionar a sprint (Peso: 4)                         |
| * CRIAR: Nova issue (Incident)                                   |
|                                                                  |
| [Cancelar Alteracoes]  [Gerar Comandos YouTrack]                 |
+------------------------------------------------------------------+
```

### 6. Criar Novos Cards

O botao "+ Novo Card" abre um modal para criar cards que serao adicionados ao YouTrack:

**Campos:**
- Titulo (obrigatorio)
- Tipo (Incident, Technical Support, User Story, Tech Story)
- Peso (opcional, 1-10)
- Data Repactuada (opcional)
- Tag Prioridade (checkbox)

Cards novos:
- Aparecem na coluna Sprint com borda roxa
- Recebem ID temporario "NOVO-1", "NOVO-2", etc.
- Sao incluidos no JSON de comandos como `create_issue`

### 7. Gerar Output para YouTrack

Apos finalizar o planejamento, o botao "Gerar Comandos YouTrack" gera JSON com:

```json
{
  "description": "Comandos para aplicar no YouTrack via MCP",
  "changes": [
    {
      "action": "update_issue",
      "issueId": "ORN-1125",
      "fields": [
        { "name": "Peso", "value": "Peso 5" }
      ]
    },
    {
      "action": "update_issue",
      "issueId": "ORN-1200",
      "fields": [
        { "name": "State", "value": "Downstream" },
        { "name": "Status", "value": "Backlog" }
      ]
    },
    {
      "action": "create_issue",
      "project": "ORN",
      "summary": "Nova issue",
      "type": "Incident",
      "fields": [
        { "name": "State", "value": "Downstream" },
        { "name": "Status", "value": "Backlog" },
        { "name": "Peso", "value": "Peso 3" },
        { "name": "Data repactuada", "value": "15/02/2026" },
        { "name": "Tags", "value": "Prioridade" }
      ]
    }
  ]
}
```

## Template HTML

O template esta em `~/.claude/skills/sprint-planning/template.html`.

### Placeholders

| Placeholder | Descricao |
|-------------|-----------|
| `{{SPRINT_NAME}}` | Nome da sprint |
| `{{GENERATED_AT}}` | Data/hora de geracao |
| `{{TOTAL_CAPACITY}}` | Capacidade total do time |
| `{{SPRINT_ISSUES}}` | JSON das issues na sprint |
| `{{BACKLOG_ISSUES}}` | JSON das issues no backlog |

### Estrutura de Issue

```json
{
  "id": "ORN-1125",
  "summary": "Historico da Conversa",
  "type": "User Story",
  "status": "Backlog-Review",
  "atendente": "alexei.secretti",
  "weight": 5,
  "dataRepactuada": "30/11/2025",
  "hasPrioridade": true
}
```

## Arquivo de Saida

`~/Projects/GrupoVoalle/Elleven/docs/sprints/sprint-planning.html`

## Campos YouTrack Utilizados

| Campo | Uso |
|-------|-----|
| **Peso** | Complexidade (1-10) |
| **Type** | Incident, User Story, Tech Story, Automation, etc. |
| **Status** | Estado atual no workflow |
| **Atendente** | Responsaveis pela issue |
| **Data repactuada** | Prazo acordado |
| **Tags** | Inclui "Prioridade" para ordenacao |

## Queries Uteis

```
# Issues em andamento (ja saiu do backlog)
project: {EV Orion} State: Downstream Status: -{Backlog}, -{Done}, -{Waiting For Packaging}, -{Waiting for Milestone} #Unresolved

# Incidents pendentes
project: {EV Orion} Type: Incident Status: Backlog

# Issues com tag Prioridade
project: {EV Orion} tag: {Prioridade}

# Por atendente
project: {EV Orion} State: Downstream for: {usuario} #Unresolved

# Issues sem peso
project: {EV Orion} State: Downstream Peso: -?
```

## Exemplo de Uso

```
Usuario: /sprint-planning

Claude:
1. Busca issues em andamento no YouTrack
2. Busca backlog disponivel (Incidents)
3. Busca issues com tag Prioridade
4. Gera dashboard interativo
5. Salva em docs/sprints/sprint-planning.html
6. Abre no navegador

Usuario manipula o dashboard:
- Cria novo card "ORN-NOVO-1"
- Adiciona ORN-1200 a sprint
- Edita peso de ORN-1125 de 3 para 5
- Remove ORN-1180 da sprint

Usuario clica "Gerar Comandos YouTrack":
- Gera JSON com alteracoes
- Copia e cola no chat do Claude
- Claude executa comandos MCP para atualizar YouTrack
```

## Configuracoes

- **URL YouTrack:** `https://grupovoalle.youtrack.cloud/issue/{ID}`
- **Projeto:** EV Orion (ORN)
- **Capacidade padrao:** 43 pontos (editavel)

## Referencias

- Skill `gv-youtrack` para workflow e transicoes de status
- MCP YouTrack: `search_issues`, `get_issue`, `update_issue`, `create_issue`

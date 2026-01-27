---
name: gv-protocols-analysis
description: Analisa protocolos de uma equipe e categoria específica, gerando documentação estruturada em markdown.
user-invocable: true
---

# Análise de Protocolos - Grupo Voalle

Analisa protocolos de uma equipe e categoria específica, gerando documentação estruturada em markdown.

## Parâmetros

O usuário pode fornecer os seguintes parâmetros:
- **equipe** ou **team_id**: ID da equipe (ex: 189 para EV Órion)
- **categoria** ou **category_id**: ID da categoria nível 1 (ex: 154 para Omnichannel)

Se não informados, pergunte ao usuário.

## Instruções

### 1. Extrair Parâmetros

Extraia do prompt do usuário:
- `team_id`: ID numérico da equipe
- `category_id`: ID numérico da categoria nível 1

Exemplos de extração:
- "equipe 189 categoria 154" → team_id=189, category_id=154
- "EV Órion (189) Omnichannel (154)" → team_id=189, category_id=154

### 2. Consultar Protocolos

Execute a seguinte query SQL usando a ferramenta `mcp__postgres__query`:

```sql
SELECT
    ai.protocol as protocol_number,
    a.title as protocol_title,
    a.description as protocol_description,
    STRING_AGG(DISTINCT r.description, E'\n') AS aggregated_reports,
    CONCAT_WS(
        ' | ',
        ssc1.title,
        ssc2.title,
        ssc3.title,
        ssc4.title,
        ssc5.title
    ) AS aggregated_categories,
    cs_main.title AS service_type_title,
    TO_CHAR(ai.date_to_start, 'DD/MM/YYYY HH24:MI:SS') AS opening_date,
    TO_CHAR(a.final_date, 'DD/MM/YYYY HH24:MI:SS') AS SLA,
    ai.incident_status_id,
    ai.team_id
FROM
    assignment_incidents ai
JOIN
    assignments a ON ai.assignment_id = a.id
LEFT JOIN
    reports r ON r.assignment_id = ai.assignment_id
LEFT JOIN
    solicitation_category_matrices scm ON ai.solicitation_category_matrix_id = scm.id
LEFT JOIN
    solicitation_service_categories ssc1 ON scm.service_category_id_1 = ssc1.id
LEFT JOIN
    solicitation_service_categories ssc2 ON scm.service_category_id_2 = ssc2.id
LEFT JOIN
    solicitation_service_categories ssc3 ON scm.service_category_id_3 = ssc3.id
LEFT JOIN
    solicitation_service_categories ssc4 ON scm.service_category_id_4 = ssc4.id
LEFT JOIN
    solicitation_service_categories ssc5 ON scm.service_category_id_5 = ssc5.id
LEFT JOIN
    catalog_services cs_main ON ai.catalog_service_id = cs_main.id
WHERE
    ai.team_id = {team_id}
    AND scm.service_category_id_1 = {category_id}
    AND ai.incident_status_id NOT IN (4, 8)
GROUP BY
    ai.protocol, a.title, a.description,
    ssc1.title, ssc2.title, ssc3.title, ssc4.title, ssc5.title,
    cs_main.title, a.final_date, ai.date_to_start,
    ai.incident_status_id, ai.team_id
ORDER BY ai.date_to_start ASC
```

### 3. Criar Pasta de Protocolos

Crie a pasta `protocolos` no diretório de trabalho atual se não existir:
```bash
mkdir -p ./protocolos
```

### 4. Criar Arquivo para Cada Protocolo

Para cada protocolo retornado, crie um arquivo `{protocol_number}.md` com a seguinte estrutura:

```markdown
# Protocolo {protocol_number}

## Informações Gerais
- **Número:** {protocol_number}
- **Data de Abertura:** {opening_date}
- **Status:** {mapear incident_status_id para texto}
- **Equipe:** {nome da equipe}
- **Categorias:** {aggregated_categories}
- **Serviço:** {service_type_title}

---

## Resumo

{Analisar protocol_description e aggregated_reports para criar um resumo conciso do problema}

---

## Categoria do Problema

**Tipo:** {Incidente/Melhoria/Dúvida baseado nas categorias}
**Módulo:** {Identificar módulo principal}
**Área:** {Identificar área específica}

---

## Criticidade

**Nível:** {Urgente/Alto/Médio/Baixo}

**Justificativa:** {Explicar por que essa criticidade foi atribuída}

---

## Solução Proposta

{Analisar o problema e propor passos de solução}
```

### 5. Determinar Criticidade

Use os seguintes critérios:

- **Urgente:** Sistema travando, funcionalidade 100% indisponível, múltiplos clientes afetados simultaneamente
- **Alto:** Funcionalidade importante comprometida, impacto em produção, workaround difícil
- **Médio:** Problema com workaround disponível, impacto moderado, não bloqueia operação
- **Baixo:** Problemas estéticos, dúvidas, melhorias de baixa prioridade

### 6. Criar README.md com Índice

Crie `protocolos/README.md` com:

```markdown
# Resumo dos Protocolos por Criticidade

> **Equipe:** {nome_equipe} ({team_id})
> **Categoria:** {nome_categoria} ({category_id})
> **Data de Geração:** {data_atual}
> **Total de Protocolos:** {quantidade}

---

## Urgente ({quantidade})

| Protocolo | Problema | Status |
|-----------|----------|--------|
| [XXXXXX](./XXXXXX.md) | Descrição curta | Status |

---

## Alto ({quantidade})

{mesma estrutura}

---

## Médio ({quantidade})

{mesma estrutura}

---

## Baixo ({quantidade})

{mesma estrutura}

---

## Estatísticas

| Criticidade | Quantidade | Percentual |
|-------------|------------|------------|
| Urgente | X | X% |
| Alto | X | X% |
| Médio | X | X% |
| Baixo | X | X% |

---

## Principais Áreas Afetadas

- **Área 1:** X protocolos (lista de números)
- **Área 2:** X protocolos (lista de números)
```

### 7. Mapeamento de Status

Use este mapeamento para `incident_status_id`:
- 1 = Aguardando Atendimento
- 2 = Em Andamento
- 10 = Em Desenvolvimento
- 12 = Aguardando Terceiros
- 25 = Em Desenvolvimento
- 26 = Aguardando Equipe Interna

### 8. Finalização

Ao concluir, exiba um resumo no formato:

```
Análise concluída!

Pasta: ./protocolos
Total de protocolos: XX
   - Urgente: X
   - Alto: X
   - Médio: X
   - Baixo: X

Arquivos criados:
   - README.md (índice)
   - {lista de arquivos .md}
```

## Equipes Conhecidas

| ID | Nome |
|----|------|
| 189 | EV Órion |

## Categorias Nível 1 Conhecidas

| ID | Nome |
|----|------|
| 154 | Omnichannel |

---
name: pbi-billing-report-generator
description: Agente para criação automatizada de relatórios de faturamento SLA no Power BI, seguindo o guia completo de indicadores, calendário, medidas financeiras e dashboards HTML.
---

# PBI Billing Report Generator

Este skill automatiza a implementação completa de um sistema de faturamento baseado em SLA (Nível de Serviço) no Power BI.

## Fluxo de Trabalho Obrigatório

### 1. Ingestão e Validação de Dados
- **JSON de Indicadores:** O agente deve ler um arquivo `.json` com a definição de metas e glosas. Se não houver, solicite ao usuário.
- **Conexão:** Garanta conexão ativa com o Power BI Desktop via ferramentas MCP.
- **Schema:** Verifique se a tabela de fatos possui colunas de `Data de Abertura` e `Tempo de Solução`.

### 2. Infraestrutura de Modelo (dCalendario)
- Verifique a existência da tabela `dCalendario`.
- Caso ausente, crie-a utilizando o template em [references/dax-templates.md](references/dax-templates.md).
- Configure relacionamentos: Data de Abertura (Ativo), outras datas (Inativos).

### 3. Medidas de Desempenho e Financeiras

#### Padrão de Criação de Medidas por Indicador (3 Níveis Hierárquicos)
Para cada indicador (IND01, IND02, etc.), siga **obrigatoriamente** este padrão de 3 níveis:

**Nível 1 - Total:**
- Nome: `Total de [Métrica]` (ex: `Total de Chamados IND02`)
- Função: `CALCULATE(COUNTA(tabela[id]))` ou `CALCULATE(COUNTROWS(tabela))`
- Propósito: Contar o universo total da métrica, sem filtros

**Nível 2 - Critério:**
- Nome: `[Métrica] com [Condição]` (ex: `Chamados com Início ≤30min`)
- Função: `CALCULATE(COUNTROWS(FILTER(tabela, condição)))`
- Propósito: Contar apenas os registros que atendem ao SLA/critério específico

**Nível 3 - Percentual:**
- Nome: `IND[XX] - [Nome Indicador] %`
- Função: `DIVIDE([Métrica com Critério], [Total de Métrica], 0) * 100`
- Propósito: Calcular o percentual de conformidade referenciando as medidas anteriores

**Exemplo Completo:**
```
Total de Chamados IND02 = CALCULATE(COUNTA(tb_chamados_ca[id]))

Chamados com Início ≤30min = CALCULATE(COUNTROWS(FILTER(tb_chamados_ca, tb_chamados_ca[Tempo ate EM ATENDIMENTO] <= 30)))

IND02 - Tempo Início Atendimento % = DIVIDE([Chamados com Início ≤30min], [Total de Chamados IND02], 0) * 100
```

- **Medidas de Glosa:** Para cada indicador, crie a medida `Glosa INDXX` usando a lógica de `SWITCH(TRUE())` baseada nas faixas de ajuste do JSON.
- **Totais:** Crie as medidas `Total de Glosas` e `Faturamento Líquido`.

### 4. Auditoria de SLA (Colunas Calculadas)
- Crie colunas calculadas na tabela de fatos para classificar cada registro como "VIOLADO" ou "NÃO VIOLADO" conforme a regra de cada indicador.

### 5. Camada Visual (Dashboards HTML)
- **Dashboard Consolidado:** Gere a medida `Dashboard HTML` integrando os 5 indicadores principais. Use estritamente o layout com `_kpiHtml` no topo e a grade de cards abaixo.
- **Detalhamentos Individuais:** Gere uma medida `Detalhe Indicador X HTML` para cada indicador, focando na explicação da regra e nas faixas de glosa.
- **Layout:** Utilize as referências em [references/html-templates.md](references/html-templates.md).

## Diretrizes de Implementação
- **Nomenclatura:** Siga o padrão `IND[XX] - [Nome] %`.
- **Display Folders:** Organize as medidas em pastas (`FINANCEIRO`, `IND01`, `IND02`, etc.).
- **CSS:** Não altere a estrutura de grid (5 colunas para o topo, 3 colunas para os indicadores).

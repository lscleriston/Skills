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

#### Formatação Obrigatória Antes de Criar Cada Medida
Antes de registrar qualquer medida DAX no Power BI via MCP:
1. Componha o código DAX conforme os templates em [references/dax-templates.md](references/dax-templates.md)
2. Formate o código chamando a API do DAX Formatter (ver [references/dax-templates.md](references/dax-templates.md), Seção 5)
3. Use o código formatado retornado pela API ao criar a medida via MCP

- **Medidas de Glosa:** Para cada indicador, crie a medida `Glosa INDXX` usando a lógica de `SWITCH(TRUE())` baseada nas faixas de ajuste do JSON.
- **Totais:** Crie as medidas `Total de Glosas` e `Faturamento Líquido`.

### 4. Auditoria de SLA (Colunas Calculadas)
- Crie colunas calculadas na tabela de fatos para classificar cada registro como "VIOLADO" ou "NÃO VIOLADO" conforme a regra de cada indicador.

### 5. Camada Visual (Dashboards HTML)
- **Dashboard Consolidado:** Gere a medida `Dashboard HTML` integrando os 5 indicadores principais. Use estritamente o layout com `_kpiHtml` no topo e a grade de cards abaixo.
- **Detalhamentos Individuais:** Gere uma medida `Detalhe Indicador X HTML` para cada indicador, seguindo **obrigatoriamente** o padrão em [references/dax-templates.md](references/dax-templates.md), Seção 6. **O layout não deve ser alterado em nenhuma circunstância.**
- **Layout:** Utilize as referências em [references/html-templates.md](references/html-templates.md).

#### Estrutura Obrigatória do Detalhe Indicador X HTML
Cada medida `Detalhe Indicador X HTML` deve:
1. Declarar VARs para armazenar: percentual, total, atingido, cores (baseadas em glosa), faixas
2. Construir HTML com 3 seções:
   - **Cabeçalho:** Nome do indicador + badge de glosa (dinâmico)
   - **Grid Central:** 3 colunas = Finalidade | Mecanismo | Faixas de Glosa
   - **Rodapé:** 3 cards = Total de [Métrica] | [Métrica] Atingida | % Atingido
3. Usar as cores e espaçamento do template fornecido — **nunca customize o layout**
4. Formatar valores numéricos com `FORMAT()` conforme exemplos
5. Antes de aplicar no Power BI, passar o código DAX pelo formatador (ver Seção 5 de dax-templates.md)

### 6. Páginas de Indicadores (Relatório Visual)
Para cada indicador (IND01 a IND05):
1. **Crie uma página** no relatório Power BI nomeada `IND[XX] - [Nome]`
2. **Insira um visual HTML** contendo a medida `Detalhe Indicador X HTML`, seguindo o modelo TMDL em [references/tmdl-templates.md](references/tmdl-templates.md), Seção 1
3. **Não customize as propriedades do visual:**
   - Posição e tamanho: use as dimensões padrão (1104px × 336px)
   - Tipo de visual: sempre `htmlContent` (tipo Microsoft HTML)
   - Projeção: sempre a medida `Detalhe Indicador X HTML`
   - Drill filter: deve estar ativado (`drillFilterOtherVisuals: true`)
4. **Valide no Power BI Desktop** que o layout do HTML renderiza corretamente (cores, espaçamento, responsive)

## Diretrizes de Implementação
- **Nomenclatura:** Siga o padrão `IND[XX] - [Nome] %` para medidas de percentual.
- **Display Folders:** Organize as medidas em pastas (`FINANCEIRO`, `IND01`, `IND02`, etc.).
- **Formatação DAX:** Todo código DAX deve ser formatado via daxformatter.com antes de ser aplicado. Nunca crie medidas com código não formatado.
- **Páginas de Indicadores:** Nome das páginas: `IND[XX] - [Nome Indicador]` (ex: `IND02 - Tempo Início Atendimento`).
- **Visual HTML:** Use sempre o tipo `htmlContent` (Microsoft HTML). Posicionamento padrão: 1104px × 336px. Não customize properties.
- **CSS:** Não altere a estrutura de grid (3 colunas no cabeçalho, 3 colunas no rodapé). Cores e espaçamento são fixos.

## Sequência Recomendada de Implementação (Agente)

1. **Leia o JSON de indicadores** (metas, glosas, descrições)
2. **Crie as medidas de 3 níveis** por indicador (Total → Critério → %) → **Formate via DAX Formatter**
3. **Crie as medidas de Glosa** (SWITCH com faixas) → **Formate via DAX Formatter**
4. **Componha o HTML** para `Detalhe Indicador X HTML` (seguindo template Seção 6 de dax-templates.md) → **Formate via DAX Formatter**
5. **Crie páginas** no relatório: uma página por indicador nomeada `IND[XX] - [Nome]`
6. **Insira visual HTML** em cada página usando template TMDL em [references/tmdl-templates.md](references/tmdl-templates.md)
7. **Valide no Power BI Desktop:** layout, cores, responsividade, interatividade de filtros
8. **Dashboard Consolidado:** Crie medida `Dashboard HTML` agregando todos os 5 indicadores

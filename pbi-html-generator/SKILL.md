---
name: pbi-html-generator
description: Agente especializado em mapear medidas do Power BI e gerar dashboards/detalhes visuais em HTML baseados em dicionários de indicadores (JSON).
---

# PBI HTML Generator

Este skill automatiza a criação de medidas visuais complexas em HTML no Power BI, integrando lógica de negócio (extraída de um JSON) com medidas reais do modelo semântico.

## Fluxo de Trabalho

### 1. Preparação e Conexão
- O agente DEVE garantir que há uma conexão ativa com o Power BI Desktop.
- Use `connection_operations(operation: "ListLocalInstances")` e `Connect` se necessário.

### 2. Leitura do Dicionário de Indicadores
- Busque no diretório de trabalho por arquivos `.json`.
- Identifique o arquivo que contém a lista de indicadores (com campos como `indice`, `meta`, `finalidade`).
- Se não encontrar, peça ao usuário: "Por favor, forneça o caminho do arquivo JSON com a descrição dos indicadores."

### 3. Mapeamento de Medidas no Modelo
- Liste as medidas do modelo usando `measure_operations(operation: "List")`.
- **Heurística de Mapeamento:** Para cada indicador no JSON, tente encontrar no modelo as medidas correspondentes para:
    - **Total/Denominador:** Procure por termos como "Total", "Abertos", "Operação".
    - **Atingido/Numerador:** Procure por "Prazo", "Iniciado", "Bom/Ótimo", "Downtime".
    - **Resultado %:** Geralmente contém o nome do indicador e o símbolo "%".
    - **Glosa:** Medidas que iniciam com "Glosa" e o ID do indicador.
- **Validação:** Informe ao usuário quais medidas foram mapeadas para cada indicador. Se algo faltar, pergunte antes de gerar o HTML.

### 4. Geração do DAX HTML
- Use o template definido em [references/html-template.md](references/html-template.md).
- Substitua as variáveis `VAR` pelos nomes reais das medidas encontradas.
- Mantenha estritamente o CSS e a estrutura de classes (`.card`, `.badge`, `.track`, `.fill`, `.fin-card`) fornecidos no template.
- Para o **Indicador 05**, note que ele possui duas visões (ISBO e ISPR), cada uma deve gerar um card específico no grid.

### 5. Aplicação
- Crie ou atualize a medida `Dashboard HTML` no modelo usando `measure_operations(operation: "Create")` ou `Update`.
- Informe o sucesso da operação e instrua o usuário a atualizar o relatório no Power BI Desktop.

## Diretrizes de Design
- **Cores:** Use apenas a paleta definida no template (Verde `#1D9E75`, Vermelho `#E24B4A`).
- **Badges de Glosa:** Se glosa > 0, badge vermelho; se 0, badge verde.
- **Tipografia:** Use 'Segoe UI'.

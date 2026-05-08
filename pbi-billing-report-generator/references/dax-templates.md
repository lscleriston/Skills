# Templates DAX: Relatório de Faturamento

Este documento contém os padrões de fórmulas DAX que o agente deve utilizar para automatizar a criação do relatório.

## 1. Tabela dCalendario
```dax
dCalendario = 
VAR vAnoMin = YEAR(MIN(TabelaFatos[Data de Abertura]))
VAR vAnoMax = YEAR(MAX(TabelaFatos[Data de Abertura]))
VAR vDataInicial = DATE(vAnoMin, 01, 01)
VAR vDataFinal = DATE(vAnoMax, 12, 31)
RETURN
ADDCOLUMNS(
CALENDAR(vDataInicial, vDataFinal),
    "Ano" , FORMAT([Date], "yyyy"),
    "Anomês nome", FORMAT([Date], "mmm/yy"),
    "Anomês" , FORMAT([Date],"yyyy/mm"),
    "Trim num" , QUARTER([Date]),
    "Trim nome", FORMAT([Date], "\Qq"),
    "Mês num" , MONTH([Date]),
    "Mês nome", FORMAT([Date], "mmm"),
    "Dia" , DAY([Date]),
    "Dia da semana", WEEKDAY([Date],2),
    "Dia da semana nome", left(FORMAT([Date],"dddd"),3),
    "Dia de trabalho", IF(WEEKDAY([Date],2)=1||WEEKDAY([Date],2)=7,"N","S"),
    "Ano atual", IF(YEAR([Date])=YEAR(TODAY()),"S","N"),
    "Mês atual", IF(YEAR([Date])=YEAR(TODAY()) && MONTH([Date])=MONTH(TODAY()),"S","N"),
    "Dia atual", IF(YEAR([Date])=YEAR(TODAY()) && MONTH([Date])=MONTH(TODAY()) && DAY([Date])=DAY(TODAY()),"S","N"),
    "Ano móvel", DATEDIFF(TODAY(),[Date],YEAR),
    "Mês móvel", DATEDIFF(TODAY(),[Date],MONTH),
    "Semana móvel", DATEDIFF(TODAY(),[Date],WEEK),
    "Dia movel", DATEDIFF(TODAY(),[Date],DAY)
)
```

## 2. Padrão de 3 Níveis para Indicadores (Total → Critério → %)

### Nível 1: Total da Métrica
```dax
Total de [Métrica] = 
CALCULATE(
    COUNTA(tb_fatos[id]),
    ALL(dCalendario)
)
```
**Variações:**
- Use `COUNTA()` para contar valores não-em-branco
- Use `COUNTROWS()` para contar linhas (melhor para tabelas filtradas)
- Omita `ALL()` se quiser respeitar filtros do contexto

### Nível 2: Filtragem por Critério
```dax
[Métrica] com [Condição] = 
CALCULATE(
    COUNTROWS(
        FILTER(
            tb_fatos,
            tb_fatos[Coluna de Tempo] <= [PRAZO_SLA_SEGUNDOS]
        )
    )
)
```
**Variações:**
- `tb_fatos[coluna] <= valor` (menor ou igual)
- `tb_fatos[coluna] > valor` (maior que)
- `tb_fatos[status] = "APROVADO"` (igualdade)
- Combine múltiplas condições com `&&` (AND) ou `||` (OR)

### Nível 3: Percentual
```dax
IND[XX] - [Nome do Indicador] % = 
DIVIDE(
    [Métrica com Critério],
    [Total de Métrica],
    0
) * 100
```
**Notas:**
- O terceiro parâmetro `0` define valor padrão se divisor for zero
- Multiplique por `100` para exibir como percentual
- Se quiser exibir com 2 casas decimais, use formato de célula ou `ROUND(..., 2)`

## 3. Medidas de Glosa (Padrão)
```dax
Glosa IND[XX] = 
VAR _os = [Valor da ordem de serviço]
VAR _val = [MEDIDA_PERCENTUAL_INDICADOR]
RETURN
    SWITCH( TRUE(),
        _val >= [META_OK], 0,
        _val >= [FAIXA_1], _os * [PERC_1],
        _val >= [FAIXA_2], _os * [PERC_2],
        _os * [PERC_MAX]
    )
```

## 4. Colunas de Status SLA
```dax
Status IND[XX] (SLA) = 
IF([COLUNA_TEMPO] <= [PRAZO_SLA_SEGUNDOS], "NÃO VIOLADO", "VIOLADO")
```

## 5. Formatação via DAX Formatter API

Antes de criar qualquer medida no Power BI, formate o código usando a API pública do daxformatter.com.

### Endpoint
```
POST https://www.daxformatter.com/api/daxformatter/DaxFormat
```

### Headers
```
Content-Type: application/json; charset=UTF-8
```

### Payload
```json
{
  "Dax": "<código DAX aqui>",
  "ListSeparator": ",",
  "DecimalSeparator": "."
}
```

**Parâmetros:**
- `Dax` (string): O código DAX a ser formatado
- `ListSeparator` (char): Use `","` para padrão BR/US
- `DecimalSeparator` (char): Use `"."` para padrão US ou `","` para padrão BR

### Resposta
String JSON com o código DAX formatado, por exemplo:
```
"DIVIDE (\n    [Chamados com Início ≤30min],\n    [Total de Chamados IND02],\n    0\n) * 100\n"
```

### Exemplo completo (cURL)
```bash
curl -X POST https://www.daxformatter.com/api/daxformatter/DaxFormat \
  -H "Content-Type: application/json" \
  -d '{"Dax":"DIVIDE([Chamados com Início ≤30min],[Total de Chamados IND02],0)*100","ListSeparator":",","DecimalSeparator":"."}'
```

### Fluxo para o Agente
1. Componha o DAX bruto a partir dos templates desta seção ou baseado na regra do indicador
2. Faça POST para o endpoint acima com o DAX no payload
3. Extraia o valor retornado (remova as aspas externas da string JSON)
4. Use esse código formatado ao criar a medida no Power BI via MCP
5. **Fallback:** Se a API retornar erro (timeout, status != 200), use o código bruto sem formatação e avise o usuário

### Notas de Implementação
- A API é gratuita e não requer autenticação
- Tempo de resposta típico: <1 segundo por medida
- Ideal para DAX complexo (SWITCH com múltiplas faixas, FILTER aninhados)
- A formatação melhora significativamente a legibilidade do código no Power BI Desktop

## 6. Template: Detalhe Indicador X HTML

**Nomenclatura:** `Detalhe Indicador [XX] HTML`

**Estrutura obrigatória (3 seções + cores dinâmicas):**

```dax
Detalhe Indicador [XX] HTML = 
VAR _val = [IND[XX] - [Nome Indicador] %]
VAR _total = [Total de [Métrica Base]]
VAR _atingido = [[Métrica] com [Critério SLA]]
VAR _glosaPerc = DIVIDE([Glosa IND[XX]], [Valor da ordem de serviço], 0)
VAR _badgeBg = IF(_glosaPerc = 0, "#E1F5EE", "#FCEBEB")
VAR _badgeText = IF(_glosaPerc = 0, "#085041", "#791F1F")

VAR _glosas = 
"<div style='border-left:3px solid #1D9E75;padding:8px 12px;background:#f0faf5;border-radius:0 6px 6px 0'>
  <div style='font-size:10px;color:#5F5E5A;text-transform:uppercase;margin-bottom:6px'>Faixas de glosa</div>
  <div style='display:flex;flex-direction:column;gap:3px'>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>[FAIXA_1]</span><span style='background:[COR_1];color:[TEXT_1];padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>[GLOSA_1]</span></div>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>[FAIXA_2]</span><span style='background:[COR_2];color:[TEXT_2];padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>[GLOSA_2]</span></div>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>[FAIXA_3]</span><span style='background:[COR_3];color:[TEXT_3];padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>[GLOSA_3]</span></div>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>[FAIXA_4]</span><span style='background:[COR_4];color:[TEXT_4];padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>[GLOSA_4]</span></div>
  </div>
</div>"

RETURN
"<div style='font-family:Segoe UI,sans-serif;font-size:13px;line-height:1.5;max-width:100%;padding:0'>
  <div style='background:#fff;border:0.5px solid #d3d1c7;border-radius:12px;padding:16px 20px'>
    <div style='display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:14px'>
      <div>
        <div style='font-size:10px;color:#888780;text-transform:uppercase;letter-spacing:0.5px;margin-bottom:3px'>Indicador de serviço</div>
        <div style='font-size:15px;font-weight:500;color:#1b2320'>[NOME_COMPLETO_INDICADOR]</div>
      </div>
      <span style='background:" & _badgeBg & ";color:" & _badgeText & ";padding:3px 12px;border-radius:999px;font-size:11px;font-weight:500;white-space:nowrap;margin-left:12px'>Glosa: " & FORMAT(_glosaPerc, "0%") & "</span>
    </div>
    <div style='display:grid;grid-template-columns:repeat(3,1fr);gap:10px;margin-bottom:14px'>
      <div style='border-left:3px solid #1D9E75;padding:8px 12px;background:#f0faf5;border-radius:0 6px 6px 0'>
        <div style='font-size:10px;color:#5F5E5A;text-transform:uppercase;margin-bottom:4px'>Finalidade</div>
        <div style='font-size:12px;color:#3a4440'>[DESCRIÇÃO_FINALIDADE_DO_INDICADOR]</div>
      </div>
      <div style='border-left:3px solid #1D9E75;padding:8px 12px;background:#f0faf5;border-radius:0 6px 6px 0'>
        <div style='font-size:10px;color:#5F5E5A;text-transform:uppercase;margin-bottom:4px'>Mecanismo de cálculo</div>
        <div style='font-size:11px;color:#3a4440;font-family:Consolas,monospace'>[FÓRMULA_RESUMIDA]</div>
      </div>
      " & _glosas & "
    </div>
    <div style='border-top:0.5px solid #e4e7e5;padding-top:12px;display:grid;grid-template-columns:repeat(3,1fr);gap:10px'>
      <div style='background:#f8f9f8;border-radius:8px;padding:10px 12px;text-align:center'>
        <div style='font-size:22px;font-weight:500;color:#1b2320'>" & FORMAT(_total, "#,##0") & "</div>
        <div style='font-size:10px;color:#888780;margin-top:2px;text-transform:uppercase'>Total de [MÉTRICA] no período</div>
      </div>
      <div style='background:#f8f9f8;border-radius:8px;padding:10px 12px;text-align:center'>
        <div style='font-size:22px;font-weight:500;color:#1b2320'>" & FORMAT(_atingido, "#,##0") & "</div>
        <div style='font-size:10px;color:#888780;margin-top:2px;text-transform:uppercase'>Total de [MÉTRICA] atingidos</div>
      </div>
      <div style='background:#f8f9f8;border-radius:8px;padding:10px 12px;text-align:center'>
        <div style='font-size:22px;font-weight:500;color:#1b2320'>" & FORMAT(_val, "0.0") & "%</div>
        <div style='font-size:10px;color:#888780;margin-top:2px;text-transform:uppercase'>SLA Atingido</div>
      </div>
    </div>
  </div>
</div>"
```

### Instruções Críticas de Implementação

**NÃO ALTERE:**
- Estrutura do grid (`grid-template-columns:repeat(3,1fr)` no cabeçalho, `repeat(3,1fr)` no rodapé)
- Cores e espaçamento dos elementos (border-radius, padding, gap)
- Tamanhos de fonte da estrutura básica
- Ordem das 3 seções (cabeçalho → grid → rodapé)

**CUSTOMIZE APENAS:**
- Nomes de medidas nos VAR (adequar ao seu indicador)
- Valores entre `[COLCHETES]`: nome completo, finalidade, fórmula resumida
- Faixas de glosa e suas cores (vindo do JSON de indicadores)
- Labels dos cards do rodapé (Total de quê, Atingidos, SLA)

### Exemplo Concreto Preenchido (IND02)

Veja em [html-templates.md](./html-templates.md) a medida `Detalhe Indicador 2 HTML` como exemplo completo com todos os campos preenchidos.

### Pós-Criação

1. Antes de aplicar a medida no Power BI, passe o código DAX pelo formatador (ver Seção 5)
2. Crie um card visual com a medida em uma página do relatório
3. Valide que as cores, espaçamento e layout aparecem exatamente como esperado
4. Se precisar ajustar, **modifique apenas o conteúdo entre colchetes, não o HTML/CSS**

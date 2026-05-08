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

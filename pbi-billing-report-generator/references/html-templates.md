# Templates HTML: Relatório de Faturamento

Este documento contém os layouts HTML para o Dashboard Consolidado e Detalhamentos Individuais.

## 1. Dashboard HTML (Consolidado)
```dax
Dashboard HTML = 
// MAPEAMENTO DINÂMICO
VAR _fatBruto = [Valor da ordem de serviço]
VAR _totalGlosa = [Total de Glosas]
VAR _fatLiquido = [Faturamento Líquido]
VAR _glosaPercTotal = DIVIDE(_totalGlosa, _fatBruto, 0)
VAR _kpiFatLiqBg = IF(_glosaPercTotal=0,"#E1F5EE","#FCEBEB")
VAR _kpiFatLiqBorder = IF(_glosaPercTotal=0,"#9FE1CB","#F7C1C1")
VAR _kpiFatLiqLabel = IF(_glosaPercTotal=0,"#0F6E56","#A32D2D")
VAR _kpiFatLiqVal = IF(_glosaPercTotal=0,"#085041","#791F1F")

VAR _kpiHtml = 
"<div style='display:grid;grid-template-columns:repeat(5,minmax(0,1fr));gap:8px;margin-bottom:12px'>
  <div style='background:#fff;border:0.5px solid #d3d1c7;border-radius:10px;padding:12px 14px'>
    <div style='font-size:9px;color:#888780;text-transform:uppercase;letter-spacing:0.4px;margin-bottom:6px'>Faturamento bruto</div>
    <div style='font-size:17px;font-weight:500;color:#1b2320;margin-bottom:6px'>" & FORMAT(_fatBruto,"R$ #,##0.00") & "</div>
    <div style='height:2px;background:#f1f3f2;border-radius:999px'><div style='height:2px;width:100%;background:#d3d1c7;border-radius:999px'></div></div>
  </div>
  // ... (Outros KPIs: Pts acumulado, Glosa %, Glosa R$)
  <div style='grid-column:span 2;background:" & _kpiFatLiqBg & ";border:0.5px solid " & _kpiFatLiqBorder & ";border-radius:10px;padding:12px 16px;display:flex;justify-content:space-between;align-items:center'>
    <div>
      <div style='font-size:9px;color:" & _kpiFatLiqLabel & ";text-transform:uppercase;letter-spacing:0.4px;margin-bottom:4px'>Faturamento líquido</div>
      <div style='font-size:22px;font-weight:500;color:" & _kpiFatLiqVal & "'>" & FORMAT(_fatLiquido,"R$ #,##0.00") & "</div>
    </div>
    <div style='text-align:right'>
      <div style='font-size:9px;color:" & _kpiFatLiqLabel & ";text-transform:uppercase;letter-spacing:0.4px;margin-bottom:4px'>Desconto aplicado</div>
      <div style='font-size:13px;font-weight:500;color:" & _kpiFatLiqVal & "'>− " & FORMAT(_totalGlosa,"R$ #,##0.00") & "</div>
    </div>
  </div>
</div>"

RETURN
"<div style='font-family:Segoe UI,sans-serif;font-size:12px;line-height:1.4;color:#1b2320;background:transparent'>
  " & _kpiHtml & "
  <div style='display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:10px'>
    // REPETIR CARDS PARA CADA INDICADOR
    <div style='background:#fff;border:0.5px solid #d3d1c7;border-radius:12px;padding:14px 16px'>
      <div style='display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:12px'>
        <div>
          <div style='font-size:10px;color:#888780;text-transform:uppercase;letter-spacing:0.4px;margin-bottom:2px'>INDXX</div>
          <div style='font-size:12px;font-weight:500;color:#1b2320'>[NOME]</div>
        </div>
        <span style='background:" & IF(_indCls="ok","#4bbf88","#e87070") & ";color:#fff;padding:2px 9px;border-radius:999px;font-size:10px;font-weight:600;white-space:nowrap'>Glosa: " & FORMAT(_glosaPerc,"0%") & "</span>
      </div>
      // ... (Barra de progresso e rodapé com Total, Atingido e Meta)
    </div>
  </div>
</div>"
```

## 2. Detalhe Indicador X HTML (Padrão Obrigatório)

### Template Genérico
Ver [dax-templates.md - Seção 6](./dax-templates.md) para o template completo e genérico que deve ser adaptado para cada indicador.

**Pontos críticos:**
- ✅ O layout de 3 seções (cabeçalho | grid 3 colunas | rodapé) **NÃO DEVE SER ALTERADO**
- ✅ Customize apenas: nomes de medidas, rótulos, finalidade, fórmula resumida e faixas de glosa
- ✅ Sempre passe pelo formatador DAX antes de aplicar no Power BI

### Exemplo Completo (IND02 - Modelo a Seguir)

```dax
Detalhe Indicador 2 HTML = 
VAR _val = [IND02 - Tempo Início Atendimento %]
VAR _total = [Total de Chamados IND02]
VAR _iniciado = [Chamados com Início ≤30min]
VAR _glosaPerc = DIVIDE([Glosa IND02], [Valor da ordem de serviço], 0)
VAR _badgeBg = IF(_glosaPerc = 0, "#E1F5EE", "#FCEBEB")
VAR _badgeText = IF(_glosaPerc = 0, "#085041", "#791F1F")

VAR _glosas = 
"<div style='border-left:3px solid #1D9E75;padding:8px 12px;background:#f0faf5;border-radius:0 6px 6px 0'>
  <div style='font-size:10px;color:#5F5E5A;text-transform:uppercase;margin-bottom:6px'>Faixas de glosa</div>
  <div style='display:flex;flex-direction:column;gap:3px'>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>≥ 90%</span><span style='background:#E1F5EE;color:#085041;padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>sem desconto</span></div>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>≥ 75% e &lt; 90%</span><span style='background:#FAEEDA;color:#633806;padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>1% OS</span></div>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>≥ 60% e &lt; 75%</span><span style='background:#FAEEDA;color:#633806;padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>2% OS</span></div>
    <div style='display:flex;justify-content:space-between;align-items:center;font-size:11px'><span style='color:#3a4440'>&lt; 60%</span><span style='background:#FCEBEB;color:#791F1F;padding:1px 7px;border-radius:999px;font-size:10px;font-weight:500'>4% OS</span></div>
  </div>
</div>"

RETURN
"<div style='font-family:Segoe UI,sans-serif;font-size:13px;line-height:1.5;max-width:100%;padding:0'>
  <div style='background:#fff;border:0.5px solid #d3d1c7;border-radius:12px;padding:16px 20px'>
    <div style='display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:14px'>
      <div>
        <div style='font-size:10px;color:#888780;text-transform:uppercase;letter-spacing:0.5px;margin-bottom:3px'>Indicador de serviço</div>
        <div style='font-size:15px;font-weight:500;color:#1b2320'>Índice do Prazo para Início do Atendimento</div>
      </div>
      <span style='background:" & _badgeBg & ";color:" & _badgeText & ";padding:3px 12px;border-radius:999px;font-size:11px;font-weight:500;white-space:nowrap;margin-left:12px'>Glosa: " & FORMAT(_glosaPerc, "0%") & "</span>
    </div>
    <div style='display:grid;grid-template-columns:repeat(3,1fr);gap:10px;margin-bottom:14px'>
      <div style='border-left:3px solid #1D9E75;padding:8px 12px;background:#f0faf5;border-radius:0 6px 6px 0'>
        <div style='font-size:10px;color:#5F5E5A;text-transform:uppercase;margin-bottom:4px'>Finalidade</div>
        <div style='font-size:12px;color:#3a4440'>Apurar o tempo para início do atendimento (meta 30 min).</div>
      </div>
      <div style='border-left:3px solid #1D9E75;padding:8px 12px;background:#f0faf5;border-radius:0 6px 6px 0'>
        <div style='font-size:10px;color:#5F5E5A;text-transform:uppercase;margin-bottom:4px'>Mecanismo de cálculo</div>
        <div style='font-size:11px;color:#3a4440;font-family:Consolas,monospace'>(Iniciados ≤ 30min ÷ Total) × 100</div>
      </div>
      " & _glosas & "
    </div>
    <div style='border-top:0.5px solid #e4e7e5;padding-top:12px;display:grid;grid-template-columns:repeat(3,1fr);gap:10px'>
      <div style='background:#f8f9f8;border-radius:8px;padding:10px 12px;text-align:center'>
        <div style='font-size:22px;font-weight:500;color:#1b2320'>" & FORMAT(_total, "#,##0") & "</div>
        <div style='font-size:10px;color:#888780;margin-top:2px;text-transform:uppercase'>Total de chamados no período</div>
      </div>
      <div style='background:#f8f9f8;border-radius:8px;padding:10px 12px;text-align:center'>
        <div style='font-size:22px;font-weight:500;color:#1b2320'>" & FORMAT(_iniciado, "#,##0") & "</div>
        <div style='font-size:10px;color:#888780;margin-top:2px;text-transform:uppercase'>Total de chamados iniciados até 30 min</div>
      </div>
      <div style='background:#f8f9f8;border-radius:8px;padding:10px 12px;text-align:center'>
        <div style='font-size:22px;font-weight:500;color:#1b2320'>" & FORMAT(_val, "0.0") & "%</div>
        <div style='font-size:10px;color:#888780;margin-top:2px;text-transform:uppercase'>SLA Atingido</div>
      </div>
    </div>
  </div>
</div>"
```

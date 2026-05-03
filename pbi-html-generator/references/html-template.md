# Template de Referência: Dashboard HTML Power BI

Este documento contém o modelo de código DAX que o agente deve utilizar como base para gerar dashboards HTML no Power BI.

## Estrutura do Código DAX (Template)

```dax
Dashboard HTML = 
// MAPEAMENTO DINÂMICO DE VARIÁVEIS
VAR _ind01Total = [Medida_Total_ID01] 
VAR _ind01Prazo = [Medida_Atingido_ID01] 
VAR _ind01Val = [Medida_Percentual_ID01] 
VAR _ind01GlosaPerc = DIVIDE([Medida_Glosa_ID01], [Valor da ordem de serviço], 0)
VAR _ind01FillColor = IF(_ind01Val>=90,"#1D9E75",IF(_ind01Val>=80,"#BA7517","#E24B4A"))
VAR _ind01Cls = IF(_ind01GlosaPerc > 0, "bad", "ok")

// ... (Repetir para outros indicadores)

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
    <div style='height:2px;background:#f1f3f2;border-radius:999px'>
      <div style='height:2px;width:100%;background:#d3d1c7;border-radius:999px'></div>
    </div>
  </div>

  <div style='background:#fff;border:0.5px solid #d3d1c7;border-radius:10px;padding:12px 14px'>
    <div style='font-size:9px;color:#888780;text-transform:uppercase;letter-spacing:0.4px;margin-bottom:6px'>Pts acumulado</div>
    <div style='font-size:17px;font-weight:500;color:#1b2320;margin-bottom:6px'>0</div>
    <div style='height:2px;background:#f1f3f2;border-radius:999px'>
      <div style='height:2px;width:0%;background:#d3d1c7;border-radius:999px'></div>
    </div>
  </div>

  <div style='background:#fff;border:0.5px solid #d3d1c7;border-radius:10px;padding:12px 14px'>
    <div style='font-size:9px;color:#888780;text-transform:uppercase;letter-spacing:0.4px;margin-bottom:6px'>Glosa</div>
    <div style='display:flex;align-items:baseline;gap:8px;margin-bottom:6px'>
      <span style='font-size:17px;font-weight:500;color:" & IF(_glosaPercTotal=0,"#1b2320","#791F1F") & "'>" & FORMAT(_glosaPercTotal,"0%") & "</span>
      <span style='font-size:11px;color:#888780'>" & FORMAT(_totalGlosa,"R$ #,##0.00") & "</span>
    </div>
    <div style='height:2px;background:#f1f3f2;border-radius:999px;overflow:hidden'>
      <div style='height:2px;width:" & FORMAT(_glosaPercTotal * 100,"0") & "%;background:" & IF(_glosaPercTotal=0,"#d3d1c7","#E24B4A") & ";border-radius:999px'></div>
    </div>
  </div>

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

    <div style='background:#fff;border:0.5px solid #d3d1c7;border-radius:12px;padding:14px 16px'>
      <div style='display:flex;justify-content:space-between;align-items:flex-start;margin-bottom:12px'>
        <div>
          <div style='font-size:10px;color:#888780;text-transform:uppercase;letter-spacing:0.4px;margin-bottom:2px'>IND01</div>
          <div style='font-size:12px;font-weight:500;color:#1b2320'>[NOME_INDICADOR]</div>
        </div>
        <span style='background:" & IF(_ind01Cls="ok","#4bbf88","#e87070") & ";color:#fff;padding:2px 9px;border-radius:999px;font-size:10px;font-weight:600;white-space:nowrap'>Glosa: " & FORMAT(_ind01GlosaPerc,"0%") & "</span>
      </div>
      <div style='display:flex;align-items:baseline;gap:6px;margin-bottom:4px'>
        <span style='font-size:26px;font-weight:500;color:#1b2320;line-height:1'>" & FORMAT(_ind01Val,"0.0") & "%</span>
        <span style='font-size:11px;color:#888780'>resultado</span>
      </div>
      <div style='margin-bottom:10px'>
        <div style='height:3px;background:#f1f3f2;border-radius:999px;overflow:hidden'>
          <div style='height:3px;width:" & FORMAT(_ind01Val,"0") & "%;background:" & _ind01FillColor & ";border-radius:999px'></div>
        </div>
      </div>
      <div style='border-top:0.5px solid #e4e7e5;padding-top:8px;display:flex;justify-content:space-between;align-items:center'>
        <div style='display:flex;flex-direction:column;gap:1px'>
          <span style='font-size:9px;color:#888780;text-transform:uppercase'>[DENOMINADOR_LABEL]</span>
          <span style='font-size:12px;font-weight:500;color:#1b2320'>" & FORMAT(_ind01Total,"#,##0") & "</span>
        </div>
        <div style='width:0.5px;height:24px;background:#e4e7e5'></div>
        <div style='display:flex;flex-direction:column;gap:1px;align-items:center'>
          <span style='font-size:9px;color:#888780;text-transform:uppercase'>[NUMERADOR_LABEL]</span>
          <span style='font-size:12px;font-weight:500;color:#1b2320'>" & FORMAT(_ind01Prazo,"#,##0") & "</span>
        </div>
        <div style='width:0.5px;height:24px;background:#e4e7e5'></div>
        <div style='display:flex;flex-direction:column;gap:1px;align-items:flex-end'>
          <span style='font-size:9px;color:#888780;text-transform:uppercase'>Meta</span>
          <span style='font-size:12px;font-weight:500;color:#1b2320'>[META_PERCENTUAL]</span>
        </div>
      </div>
    </div>

    // ... (Repetir blocos de cards para outros indicadores)

  </div>
</div>"
```

## Paleta de Cores Recomendada
- Verde Sucesso: `#1D9E75` / `#4bbf88`
- Vermelho Erro: `#E24B4A` / `#e87070`
- Amarelo Atenção: `#BA7517` / `#f4c560`
- Cinza Borda/Sutil: `#d3d1c7` / `#e4e7e5` / `#f1f3f2`
- Texto Principal: `#1b2320`
- Texto Secundário: `#888780` / `#5a645f`

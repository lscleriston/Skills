# Templates TMDL: Páginas de Indicadores

Este documento contém os modelos TMDL (Tabular Model Definition Language) para criar páginas de indicadores com visuals HTML.

## 1. Visual Container HTML para Detalhe Indicador

**Nomenclatura da página:** `IND[XX] - [Nome Indicador]`

**Arquivo:** `definition/model.tmdl` → `pages/[page-name]/definition.tmdl`

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/fabric/item/report/definition/visualContainer/2.8.0/schema.json",
  "name": "[UNIQUE_ID_HEX_32_CHARS]",
  "position": {
    "x": 151.62790697674419,
    "y": 89.302325581395351,
    "z": 0,
    "height": 335.81395348837208,
    "width": 1104.1860465116279,
    "tabOrder": 0
  },
  "visual": {
    "visualType": "htmlContent443BE3AD55E043BF878BED274D3A6855",
    "query": {
      "queryState": {
        "content": {
          "projections": [
            {
              "field": {
                "Measure": {
                  "Expression": {
                    "SourceRef": {
                      "Entity": "_Medidas"
                    }
                  },
                  "Property": "Detalhe Indicador [XX] HTML"
                }
              },
              "queryRef": "_Medidas.Detalhe Indicador [XX] HTML",
              "nativeQueryRef": "Detalhe Indicador [XX] HTML"
            }
          ]
        }
      },
      "sortDefinition": {
        "isDefaultSort": true
      }
    },
    "drillFilterOtherVisuals": true
  }
}
```

### Campos a Personalizar

| Campo | Valor | Exemplo |
|---|---|---|
| `name` | ID único hexadecimal de 32 caracteres | `0e589550b47eb26b03d0` (ou gerar novo UUID em hex) |
| `position.x`, `.y` | Coordenadas do visual na página (pixels) | `151.62...`, `89.302...` (defaults recomendados) |
| `position.height`, `.width` | Dimensões: altura × largura (pixels) | `335.81...` × `1104.18...` (padrão: preenche a largura) |
| `visual.query.content.projections[0].field.Measure.Property` | Nome exato da medida | `Detalhe Indicador 2 HTML` |
| `visual.query.content.projections[0].queryRef` | Referência completa | `_Medidas.Detalhe Indicador 2 HTML` |
| `visual.query.content.projections[0].nativeQueryRef` | Nome nativo | `Detalhe Indicador 2 HTML` |

### Propriedades Obrigatórias (NÃO ALTERAR)

- `visualType: "htmlContent443BE3AD55E043BF878BED274D3A6855"` — sempre este tipo para renderizar HTML
- `drillFilterOtherVisuals: true` — permite filtros cruzados
- `sortDefinition.isDefaultSort: true` — mantém ordem padrão

---

## 2. Estrutura de Página Completa (YAML/TMDL)

Para referência, a estrutura da página em TMDL:

```yaml
page: "IND[XX] - [Nome Indicador]"
  filters: []
  displayOption: "fitToPage"
  visualContainers:
    - $ref: "#/definitions/visualContainer_[UNIQUE_ID]"
```

---

## 3. Fluxo de Criação (Agente)

Ao gerar a página para um indicador:

1. **Gere um ID único** para o visual:
   - Use um UUID em hexadecimal (32 caracteres, sem hífens)
   - Exemplo: `0e589550b47eb26b03d0` (use ferramentas de geração UUID)

2. **Preencha o template acima** com:
   - ID único do visual
   - Número do indicador (XX)
   - Nome exato da medida `Detalhe Indicador [XX] HTML`

3. **Posicionamento padrão:**
   - x: `151.6` (margem esquerda)
   - y: `89.3` (margem superior)
   - width: `1104.2` (preenche ~90% da página)
   - height: `335.8` (altura fixa para o card)

4. **Insira no arquivo de definição** da página Power BI (ou use MCP para criar visual programaticamente)

5. **Valide no Power BI Desktop:**
   - Abra a página criada
   - Confirme que o visual HTML renderiza com o layout correto
   - Verifique cores, espaçamento e responsividade

---

## 4. Erro Comum: Medida Não Encontrada

Se o visual exibir "Erro ao avaliar a medida", verifique:
- ✅ Nome exato da medida: `Detalhe Indicador [XX] HTML` (maiúsculas, espaços)
- ✅ Medida foi criada e formatada (via DAX Formatter)
- ✅ Entity "_Medidas" existe no modelo
- ✅ Sintaxe TMDL: `"Property": "Detalhe Indicador [XX] HTML"`

---

## 5. Variações de Layout (Opcional)

Se precisar alterar **apenas posicionamento/tamanho** de múltiplas páginas:

**Para tela inteira (full-width):**
```json
"width": 1456,
"height": 896,
"x": 0,
"y": 0
```

**Para sidebar + conteúdo (layout 2 colunas):**
```json
"width": 1000,
"x": 456,
"y": 20
```

**NUNCA altere:** `visualType`, `drillFilterOtherVisuals`, ou a estrutura interna da `query`.

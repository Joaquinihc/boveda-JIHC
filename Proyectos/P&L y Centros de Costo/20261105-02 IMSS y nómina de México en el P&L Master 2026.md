---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-11-05
temas: [pnl, odoo, cierre-mensual]
metricas: []
---

# IMSS y nómina de México en el P&L Master 2026

**Pregunta**: ¿Por qué el IMSS de México en el P&L Master no cuadraba con Odoo, qué se cambió para dejarlo igual y cuánto mueve el P&L?

## Conclusión
> - **Desde mayo, el gasto del IMSS estaba doble.** La nómina provisiona el IMSS del empleador todos los meses, y además las facturas bimestrales del IMSS se registraron como gasto. Contabilidad lo corrigió en Odoo el 17-sep y el 27-oct, cuando el Master ya estaba cerrado.
> - **Criterio: manda Odoo** ([[Pablo Lucero]], 4-nov). Por eso septiembre queda con el IMSS en positivo: ahí cae la reversa de los duplicados de mayo y julio.
> - **El Master quedó alineado el 5-nov.** De junio a octubre, la nómina de México por área es igual a Odoo. La única diferencia es Marketing de julio: 1.383 MXN (≈79 USD).
> - **Efecto enero–octubre** (USD, P&L current FX al 5-nov):
>   - EBITDA: +49.607. Se compone de IMSS +40.466, nómina de julio +9.584 y ajustes SIPARE −443.
>   - Taxes: −6.895, por el Impuesto sobre Nómina.
>   - Net income: +42.712.
> - **Antes de agregar líneas de Odoo al Master, comparar el total por área, no cuenta por cuenta.** La nómina original de julio trae las cargas del empleador dentro de las líneas de remuneraciones (601.01.05x). Agregar además el IMSS por área de julio lo contó dos veces. Se detectó y se corrigió en la misma sesión.
> - **Abierto:** al rehacer la nómina de julio (1-sep), Sales y Sales ISR bajaron 167.392 MXN en total. La consulta a [[Nicolás Astudillo]] está en borrador en su DM (5-nov).

## Método
- **Odoo**, vía XML-RPC (solo lectura), el 4 y el 5-nov: cuentas 601.26.001 (IMSS), 601.26.002 (Cesantía y Vejez), 601.27.001 (INFONAVIT), 601.28.001 (SAR), 601.29.001 (ISN) y remuneraciones de México, por mes y analítica.
- **Master:** planilla en vivo del 5-nov, hoja Financials DB. El efecto se midió en la hoja P&L (current FX) antes y después de cada etapa.
- **Plan inicial:** `Correcciones IMSS Mexico 2026.md` (4-nov), en `/Users/joaquinherreracabrejos/Documents/Cierre de mes/P&L/Octubre/`. Julio y el anexo de ese archivo quedaron superados (ver Caveats).
- **Documento para el equipo:** Notion, [Ajustes P&L cierre octubre](https://app.notion.com/p/agendapro/Ajustes-P-L-cierre-octubre-40fab34c233b803f8850e2432b10ee9c), sección 2.

## Evidencia

### IMSS (601.26.001) por mes, en MXN
| | May | Jun | Jul | Ago | Sep | Oct |
|---|---:|---:|---:|---:|---:|---:|
| Master antes | — | −94.950,38 | −288.390,59 | −163.699,80 | −363.604,55 | −75.070,52 |
| Master después | −70.016,97 | −69.156,98 | −141.322,55 | −74.064,71 | 141.621,41 | −75.070,52 |
| Odoo | −70.016,97 | −69.156,98 | −215.732,29 | −74.064,71 | 141.621,41 | −75.070,52 |
| Efecto EBITDA (USD) | −4.044 | 1.485 | 8.421 | 5.250 | 29.354 | – |

En julio, la 601.26.001 del Master tiene 74.410 MXN menos que Odoo, porque la provisión del IMSS de julio va dentro de las líneas de remuneraciones de la nómina original. Por área, julio cuadra con Odoo.

### Qué se cambió, mes a mes
- **Mayo:** entra el CEAV de abril duplicado (BILL/2026/05/0049, −70.016,97). La reversa de septiembre lo incluye: sin esta fila, septiembre descontaría 70 mil que nunca entraron. El resto de la nómina de enero a mayo se armó a mano por persona en el Master y no se tocó.
- **Junio:** sale la factura BILL/2026/06/0032 (−94.950,38), que Odoo aplicó al pasivo, y entra la provisión del IMSS por área (−69.156,98). Las líneas de junio del Master ya traían Cesantía, INFONAVIT y SAR, pero no el IMSS.
- **Julio:** la factura BILL/2026/07/0035 queda en −141.322,55, lo que Odoo mantiene como gasto (el CEAV de mayo y junio duplicado). No se agrega el IMSS por área.
- **Agosto:** la factura BILL/2026/08/0038 queda en −105,53; el resto se aplicó al pasivo.
- **Septiembre:**
  - Sale la factura BILL/2026/09/0032 (período 08, −295.079,27), que Odoo revirtió como duplicada (MISC/2026/09/0015).
  - Entra la reversa neta, +210.146,69: la reversa de mayo y julio (RECLA/2026/09/0007, +211.339,52) menos el ajuste SIPARE de abril a septiembre (MISC/2026/09/0019, −1.192,83).
- **Ajustes SIPARE:**
  - Julio: −4.504,54 de Cesantía y Vejez (MISC/2026/07/0019).
  - Agosto: −2.244,29 de INFONAVIT y −921,48 de SAR (MISC/2026/08/0033).
- **Impuesto sobre Nómina** (601.29.001 G4009, línea Taxes):
  - Entran junio −45.160, julio −38.451 y agosto −48.904.
  - Septiembre pasa de −48.904 a −35.656, por el ajuste CARGA/2026/09/0002 (+13.248).

### Nómina de julio rehecha (Sales e ISR)
Odoo rehízo la nómina de julio el 1-sep (SLR/2026/07/0003), después del cierre del Master. El Master se dejó igual a Odoo:

| Code | Antes (nómina original, cuentas 601.01.056 y 601.01.058) | Ahora (Odoo) | Cambio |
|---|---:|---:|---:|
| G1100 Sales | −62.566 | −185.507,57 | −122.941,57 |
| G1108 Sales ISR | −778.577 | −488.243,14 | 290.333,86 |
| **Total** | **−841.143** | **−673.750,71** | **167.392,29** (≈ 9.584 USD) |

Parte del costo pasó de ISR a Sales, y además el total bajó. La causa de la baja está consultada a Contabilidad.

### Doble conteo de julio: cómo se detectó
- El plan del 4-nov agregaba en julio la provisión del IMSS por área, igual que en junio (6 filas, −74.410 MXN). Se aplicó en la etapa 7.
- Al preparar el bloque de otras cuotas, el total de la nómina de julio por área del Master se comparó contra Odoo, y julio quedaba con el IMSS dos veces.
- Se detuvo el trabajo y se avisó a Joaquín. Con su OK, se sacaron esas 6 filas (24999–25004, agregadas en la misma sesión) y G1100 y G1108 de julio quedaron iguales a Odoo.
- Por la misma razón no se agregaron las 36 filas por área de Cesantía y Vejez, INFONAVIT y SAR de junio y julio que proponía el anexo del plan (−280.140 MXN, ≈16 mil USD).

## Caveats
- `Correcciones IMSS Mexico 2026.md` (4-nov) está desactualizado en dos partes: en julio agrega el IMSS por área, y el anexo agrega las 36 filas de cargas por área. No usarlo para reaplicar.
- Marketing de julio difiere de Odoo en 1.383 MXN (≈79 USD). No se ajustó.
- G4009 está en Mappings como «G4009 - Other taxes», en la categoría Taxes. Por eso el ISN mueve el Net income y no el EBITDA.
- La nómina de enero a mayo del Master se armó por persona y no se tocó, salvo la fila del CEAV duplicado de mayo.

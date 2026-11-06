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
> - **El 6-nov se agregaron en mayo los finiquitos de Sales ISR** (272.139,97 MXN, ≈15.720 USD). La baja de 167.392 MXN de julio son finiquitos de mayo que Odoo pasó a su mes al rehacer la nómina; los otros 104.748 MXN son dos finiquitos de mayo registrados en Odoo el 2-oct. El Master no los tenía en ningún mes.
> - **Efecto enero–octubre** (USD, P&L current FX al 6-nov):
>   - EBITDA: +33.887. Se compone de IMSS +40.466, nómina de julio +9.584, finiquitos de mayo −15.720 y ajustes SIPARE −443.
>   - Taxes: −6.895, por el Impuesto sobre Nómina.
>   - Net income: +26.992.
> - **Antes de agregar líneas de Odoo al Master, comparar el total por área, no cuenta por cuenta.** La nómina original de julio trae las cargas del empleador dentro de las líneas de remuneraciones (601.01.05x). Agregar además el IMSS por área de julio lo contó dos veces. Se detectó y se corrigió en la misma sesión.
> - **Cuando Odoo rehace un asiento después del cierre, buscar asientos creados después del cierre en cualquier mes.** Lo que baja en el mes rehecho puede haberse movido a otro: así aparecieron los finiquitos de julio en mayo.

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

Parte del costo pasó de ISR a Sales, y además el total bajó. La baja son finiquitos de mayo: la nómina original de julio los traía y, al rehacerla, Contabilidad los pasó a mayo (ver la sección siguiente).

### Finiquitos de mayo (agregados el 6-nov)
En Odoo (consulta del 6-nov, XML-RPC), mayo tiene tres asientos de finiquitos en G1108, todos creados después del cierre:
- **SLR/2026/05/0002** «Centraliza finiquitos 2026-05»: 167.391,68 MXN (Indemnización 90 días 141.810,30 y Prima de antigüedad 25.581,38). Se creó el 1-sep a las 18:48, 44 minutos antes que el SLR/2026/07/0003. Es la baja de julio; los 0,61 MXN de diferencia vienen del redondeo de la nómina original en el Master.
- **SLR/2026/05/0003**, finiquito de Stephanie Padilla: 73.948,00 MXN (2-oct).
- **SLR/2026/05/0004**, finiquito de Mauricio López, «complemento SLR/2026/05/0002»: 30.800,29 MXN (2-oct).

En el Master, Padilla y López figuraban solo con su sueldo mensual de enero a mayo, y no había filas de finiquito entre enero y junio. Con el OK de Joaquín, el 6-nov se agregaron 5 filas en mayo (Financials DB 25023–25027; Mexico / Mexico / G1108; respaldo previo en Drive, 1T6n1CrvXLk89s81HzO6WGgGiRrtTJqZHTVtDmi9cxqI):

| Detail | MXN |
|---|---:|
| 601.07.011 Indemnización 90 días \| Payroll ISR | −203.160,97 |
| 601.07.010 Prima de Antigüedad \| Payroll ISR | −36.805,38 |
| 601.06.001 Vacaciones \| Payroll ISR | −19.667,81 |
| 601.12.001 Aguinaldo \| Payroll ISR | −7.187,53 |
| 601.07.001 Prima Vacacional \| Payroll ISR | −5.318,28 |
| **Total** | **−272.139,97** |

P&L (current FX), mayo: EBITDA de (181.421) a (197.141) y Net income de (136.860) a (152.579) USD. Los 167.392 MXN solo cambian de mes (efecto neto −85 USD por la tasa de mayo frente a la de julio); lo nuevo para el año son los finiquitos del 2-oct (≈−6.050 USD).

### Doble conteo de julio: cómo se detectó
- El plan del 4-nov agregaba en julio la provisión del IMSS por área, igual que en junio (6 filas, −74.410 MXN). Se aplicó en la etapa 7.
- Al preparar el bloque de otras cuotas, el total de la nómina de julio por área del Master se comparó contra Odoo, y julio quedaba con el IMSS dos veces.
- Se detuvo el trabajo y se avisó a Joaquín. Con su OK, se sacaron esas 6 filas (24999–25004, agregadas en la misma sesión) y G1100 y G1108 de julio quedaron iguales a Odoo.
- Por la misma razón no se agregaron las 36 filas por área de Cesantía y Vejez, INFONAVIT y SAR de junio y julio que proponía el anexo del plan (−280.140 MXN, ≈16 mil USD).

## Caveats
- `Correcciones IMSS Mexico 2026.md` (4-nov) está desactualizado en dos partes: en julio agrega el IMSS por área, y el anexo agrega las 36 filas de cargas por área. No usarlo para reaplicar.
- Marketing de julio difiere de Odoo en 1.383 MXN (≈79 USD). No se ajustó.
- G4009 está en Mappings como «G4009 - Other taxes», en la categoría Taxes. Por eso el ISN mueve el Net income y no el EBITDA.
- La nómina de enero a mayo del Master se armó por persona y no se tocó, salvo el CEAV duplicado y los finiquitos de mayo. Odoo rehízo completas la nómina de abril (SLR/2026/04/0001, 21-sep) y la de mayo (SLR/2026/05/0001, 1-sep). Solo se comparó Sales e ISR (G1100, G1107 y G1108 del Master contra G1100, G1108 y G1128 de Odoo): abril difiere ≈8 mil MXN, y mayo ≈3 mil después de agregar los finiquitos. El resto de las áreas de enero a mayo no se comparó.

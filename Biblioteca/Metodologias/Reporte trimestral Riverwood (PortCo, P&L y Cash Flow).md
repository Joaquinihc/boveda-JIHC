---
type: metodologia
temas: [board, reportes]
actualizado: 2026-10-19
---

# Reporte trimestral Riverwood (PortCo, P&L y Cash Flow)

Cada trimestre (≈ día 1, plazo entre el 5 y el 21) Riverwood (Ignacio Ortiz, reporting@rwcm.com) pide el *reporting package*; [[Pablo Lucero]] lo reenvía. Carpeta de trabajo: `~/Documents/Análisis/RW-Quarterly-Requests/`; lo que se envía queda en `To send/`.

## Archivos
| Archivo | Fuente | Notas |
|---|---|---|
| `0. PortCo Reporting Metrics_v1 - Constant FX.xlsx` | Board book + DWH (mapeo abajo) | Hoja Input, una columna por trimestre, USD M; celdas verdes = inputs |
| `1. P&L (variable exchange rate).xlsx` | Hoja **"P&L (current FX)"** del Master del P&L (sin redondear) | Hoja única `P&L`, una columna por mes, valores pegados; solo fecha (fila 1) y Annualized (fila 28) son fórmulas. El DWH redondea por fila: llenar desde él deja diferencias de $1–7 |
| `2. Cash Flow Statement.xlsx` | Planilla de tesorería `Cash flow (Causal)` | Desde Q3'26 en estructura nueva: [[Cash flow — estructura nueva vs formato Causal]] |
| `3. Headcount.xlsx` | `finance.headcount` | BizOps (5–8 personas) va en G&A, pero su costo del P&L está en R&D |
| Cap table y ESOP | Los exporta Pablo desde el software | Cruce Q3'26: [[Slack — Cap table vs Grants Q3, diferencia de 34 mil opciones explicada]] |

## Mapeo del PortCo (fila → fuente)
Reconstruido desde los trimestres que había cargado Finanzas (Q2'26 calzó a ±0,01).
- **ARR bridge**: `waterfall_saas`, **todos los segmentos**, FX constante, ΣΔMRR × 12. New = `new`; Upsell = `upgrade`; Downsell = `downgrade`; Churn = churn_graduado + churn_onboarding + **reactivation (neteada)**. AI no entra. (El ARR Walk del board book es solo B2B3: no calza con esto.)
- **Logos**: New = suma trimestral de `dwh.country_new_merchants`; Churned = stock(fin) − stock(inicio) − New, con stock = `country_merchants` del último mes del trimestre. No usar el waterfall.
- **P&L** (`profit_and_loss`, amount_usd, Real): Recurring revenue = Software + AI; Non-recurring = Payments; Non-rec COGS = Payment Processing; Rec COGS = resto de Service Costs; S&M = 3- Acquisition; R&D = Development + Product; G&A = Administration + Other Corporate; D&A y Capex en positivo; Taxes 0.
- **Interest**: toda la historia entra como −(neto de 5- Financing), aunque la etiqueta dice Income/(Expense). Se mantiene por consistencia.
- **Caja**: CFO = Cash from Operations; CFI = Financial income; CFF = resto de Cash from Finance + FX rate effect (la plantilla no tiene fila de TC). CAPEX = −(7- Capex) del P&L.
- BoP/EoP son fórmulas encadenadas desde Q1'24: si la historia no se actualiza, el EoP no calza con la caja real.

## Cómo editar los xlsx
Son exports de Google Sheets con dibujos: **editar el XML de la hoja dentro del zip**, nunca guardar con openpyxl (destruye los dibujos). Inyectar el valor en caché (`<v>`) de las fórmulas para que se vean sin recalcular. Sin LibreOffice en el Mac: verificar evaluando las fórmulas en Python. Respaldar antes (`.BACKUP-AAAA-MM-DD`).

## Q3'26 (7–8 oct 2026)
- **Decisiones de Joaquín**: P&L solo jul–sep actualizado desde el Master (abr–jun y la historia anterior quedan como se reportaron); hoja Quarterly del Cash Flow formato Causal sin tocar (rota desde la col. K); Gross Margin Payments de jun-26 queda en 33,94 (typo, se ve como 3394%).
- Se armaron dos pares PortCo ↔ Cash Flow (Causal: EoP Q3'26 12,581; nueva estructura: 12,641 → 12,643 con BBVA MX USD restaurada). En `To send/` va la estructura nueva, sin la hoja Notes.
- **Q3'26 en el PortCo** (USD M): Recurring revenue 2,6622 · Non-recurring 0,6708 · Rec COGS 0,6052 · Non-rec COGS 0,4502 · S&M 1,0257 · R&D 1,3206 · G&A 0,6140 → EBITDA −0,683 (−20,5%), igual al P&L enviado. Caja: CFO −0,7695 · CFI 0,1195 · CFF −0,0798 · CAPEX 0,0010.
- Paquete enviado a Ignacio el 8-oct (según el hilo de Slack del cap table).

### Pendientes para Q4'26
- **Historia de caja del PortCo**: al cierre de la sesión, el EoP de Q3'26 en `To send/` daba 12,575 contra 12,643 del Cash Flow. Para que calce: BoP Q1'24 = 1,0078; Q1'26 = −0,8536 / 0,1062 / −0,0485; Q2'26 = −0,6789 / 0,1224 / −0,1174; Q1'24 Investing 0,0106 y Financing −0,1842; Q3'25 −0,4746 / 14,8541; Q4'25 −0,6076 / −0,0711. Verificar si la versión enviada lo incorporó.
- **Q2'26 con valores redondeados** (Riverwood calcula el crecimiento de Q3 contra ellos: revenue 7,5% vs 6,4% real). Reales consistentes con el P&L: 2,5272 / 0,6056 / 0,5439 / 0,3875 / S&M 1,0341 / R&D 1,2722 / G&A 0,4754; D&A 0,0305, Interest −0,0971, CAPEX 0,0047; ARR 0,7014 / 1,1380 / −0,2856 / −1,0892; logos 3.958 / −3.748. Ojo: Q1'26 y Q4'25 en FX constante también difieren de los datos actuales.
- **Logos nuevos Q3'26**: 3.525 cargados vs 3.524 en la fuente (churned −3.403 vs −3.406 con datos del 8-oct). Menor.
- Preguntar de nuevo si se re-expresan los meses anteriores del P&L (abr–jun 2026) y el Cash Flow.
- Plantilla de Riverwood: PBT y Net Income no incluyen el EBIT, y la hoja Output trae `#REF!` en *Avg Attainment by Reps*. No son nuestros.

## Checklist del próximo trimestre
1. Refrescar `profit_and_loss`, `cash_flow_*`, `waterfall_saas`, `country_new_merchants`, `country_merchants` (Finanzas edita el mes de cierre por semanas). Si el DWH no tiene la última planilla, trabajar desde el Master y la planilla de tesorería.
2. Recalcular los USD de la planilla de tesorería (filas sin fórmula de Amount USD).
3. Llenar P&L desde "P&L (current FX)", Cash Flow en estructura nueva, PortCo con el mapeo de arriba.
4. Cruzar: EBITDA del PortCo = suma del P&L del trimestre; EoP de caja del PortCo = Final del Cash Flow; cap table vs ESOP persona por persona.

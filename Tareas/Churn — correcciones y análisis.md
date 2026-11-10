---
type: tarea
estado: pendiente
prioridad: media
tiempo:
tipo: reporte
origen: manual
proyecto: "[[_Churn y Graduación]]"
temas:
  - churn
  - evidence
fuente: https://agendapro.slack.com/archives/D0A5A19KWPL/p1791466522101369
fecha_limite:
creado: 2026-10-08
revisar: false
id: T-090
---

# Churn — correcciones y análisis

Tarea general de churn. Incluye los tres ajustes que pidió [[Pablo Santa Inés]] (CX) el 8-oct para que CX y Finanzas lean la misma cifra de churn: Involuntary Churn, Churn Tracker y Card Payment Recovery. Diagnóstico y verificación en [[20261008-01 Ajustes de churn en Evidence pedidos por CX (Pablo Santa Inés)]]. Incluye también el análisis del alza del churn voluntario de octubre.

## Subtareas
- [ ] Correcciones de churn en Evidence que pidió Pablo Santa Inés: dedup de `cancel_reason` (Involuntary Churn), etiqueta Graduado en el Churn Tracker y grupo propio para el código 317 en Card Payment Recovery. Listo cuando las tres estén en producción (detalle en Notas).
- [ ] Analizar el alza del churn voluntario de octubre, con foco en Colombia (la trajo Pablo Santa Inés). Listo cuando el alza esté explicada por país y causa.

## Notas
### Detalle de las correcciones (antes eran tres subtareas)
- Filtrar `status = 'cancelled'` antes del dedup de `cancel_reason` en las 8 queries de extracción que lo copian (incluye `churn_executive`) — listo cuando sep-2026 muestre 549 involuntarias
- Churn Tracker: etiqueta `'Graduado'` en el budget de la query `agg` y titular condicional — listo cuando el budget graduado de oct muestre USD 11.332
- Card Payment Recovery: grupo propio para el código 317 (bloqueo «high risk» de dLocal), separado de «Rechazo del banco» — listo cuando las cuentas 317 salgan fuera de ese grupo

### Alza del churn voluntario de octubre
- El churn voluntario venía bajando jul–sep (mes cerrado), pero octubre iba camino a ser el peor de los últimos 13 meses, con un alza fuerte en Colombia. Venía de [[Revisar el alza del churn voluntario de octubre (foco Colombia)]] (T-103), de [[2026-10-26 Coordinación Finanzas]].

### Pendientes menores
- 2026-10-08 · Responderle a Pablo con el plan (le respondí «Lo reviso!» en el hilo).
- 2026-10-08 · Cuando el fix del dedup esté en producción, corregir el grano en la skill `finance-metric-definitions` (hoy dice «most recent cancellation per subscription»), de donde sale [[Churn voluntario vs involuntario]].
- 2026-10-08 · CX ya aplicó el fix 1 desde el 8-oct: hasta el deploy, su involuntario queda ~7 logos sobre Evidence en sep.

## Historial
- 2026-10-08 · Creada desde Claude Code (diagnóstico del pedido de Pablo Santa Inés).
- 2026-11-10 · Pasa a ser la tarea general de churn: título «Ajustes de churn en Evidence pedidos por Pablo Santa Inés» → «Churn — correcciones y análisis»; las tres correcciones se fusionan en una subtarea (detalle en Notas) y se suma el análisis del alza de octubre de T-103 (Joaquín en conversación)

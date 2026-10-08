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

# Ajustes de churn en Evidence pedidos por Pablo Santa Inés

Tres ajustes que pidió [[Pablo Santa Inés]] (CX) el 8-oct para que CX y Finanzas lean la misma cifra de churn: Involuntary Churn, Churn Tracker y Card Payment Recovery. Diagnóstico y verificación en [[20261008-01 Ajustes de churn en Evidence pedidos por CX (Pablo Santa Inés)]].

## Subtareas
- [ ] Filtrar `status = 'cancelled'` antes del dedup de `cancel_reason` en las 8 queries de extracción que lo copian (incluye `churn_executive`) — listo cuando sep-2026 muestre 549 involuntarias
- [ ] Churn Tracker: etiqueta `'Graduado'` en el budget de la query `agg` y titular condicional — listo cuando el budget graduado de oct muestre USD 11.332
- [ ] Card Payment Recovery: grupo propio para el código 317 (bloqueo «high risk» de dLocal), separado de «Rechazo del banco» — listo cuando las cuentas 317 salgan fuera de ese grupo

## Notas

### Pendientes menores
- 2026-10-08 · Responderle a Pablo con el plan (le respondí «Lo reviso!» en el hilo).
- 2026-10-08 · Cuando el fix del dedup esté en producción, corregir el grano en la skill `finance-metric-definitions` (hoy dice «most recent cancellation per subscription»), de donde sale [[Churn voluntario vs involuntario]].
- 2026-10-08 · CX ya aplicó el fix 1 desde el 8-oct: hasta el deploy, su involuntario queda ~7 logos sobre Evidence en sep.

## Historial
- 2026-10-08 · Creada desde Claude Code (diagnóstico del pedido de Pablo Santa Inés).

---
type: analisis
proyecto: "[[_Churn y Graduación]]"
estado: activo
fecha: 2026-10-08
temas: [churn, evidence]
metricas: []
personas: ["[[Pablo Santa Inés]]"]
---

# Ajustes de churn en Evidence pedidos por CX (Pablo Santa Inés)

**Pregunta**: ¿A qué reportes de Evidence se refiere Pablo Santa Inés con sus tres ajustes del 8-oct, y tiene razón en cada uno?
**Origen**: [DM de Slack del 8-oct-2026](https://agendapro.slack.com/archives/D0A5A19KWPL/p1791466522101369). Objetivo: que CX y Finanzas lean la misma cifra de churn.

## Conclusión
> Los tres son bugs reales en `agendapro-dashboards-evidence` y se reproducen. (1) **Involuntary Churn** cuenta 542 involuntarias en sep en vez de 549, porque el dedup de `cancel_reason` elige una cancelación programada que nunca se ejecutó. CX ya aplicó el fix el 8-oct, así que hasta corregirlo Evidence queda 7 logos abajo. (2) **Churn Tracker** muestra budget 0 para graduado por un desajuste de etiquetas (`'Graduated'` vs `'Graduado'`). Además, el titular «at or under plan» está escrito fijo. (3) **Card Payment Recovery** mete el código 317 (bloqueo «high risk» de dLocal) en «Rechazo del banco», cuya acción es contactar al cliente, pero se resuelve desbloqueando en dLocal. El fix 1 no puede ir solo en una query: el mismo dedup está copiado en 8.

## Método
Lectura del código de las páginas y de las queries de extracción (rama `main`, 8-oct). Verificación contra Redshift (MCP agendapro-dwh) de las cifras que cita Pablo.

## Evidencia

### 1. Involuntary Churn → `/finance-ops/involuntary-churn`
- Query: `sources/dwh/merchants/involuntary_churn_facts.sql`, CTE `csc` (líneas 29-36): `ROW_NUMBER() OVER (PARTITION BY id ORDER BY cancelled_at DESC)` sobre `staging.stg_chargebee__subscriptions_snapshot` con `cancelled_at IS NOT NULL`, sin filtrar `status`.
- Cuando una cuenta se reactiva, el snapshot de un mes anterior queda en `non_renewing`, sin `cancel_reason` y con un `cancelled_at` programado *posterior* a la cancelación real. El dedup elige esa fila.
- Ejemplo `16CJRgVH484An4wGp`:

| Snapshot | status | cancel_reason | cancelled_at |
|---|---|---|---|
| may-26 | non_renewing | NULL | 17-jun ← la elige hoy |
| jun-26 | cancelled | not_paid | 8-jun ← la real |

- Con el filtro `status = 'cancelled'` antes del `ROW_NUMBER()`, sep-2026 queda así (Redshift al 8-oct, suscripciones con churn en `dwh.revenue_waterfall_software`):

| | Hoy | Con fix |
|---|---|---|
| Involuntario (`not_paid`) | 542 | **549** |
| Desconocido (NULL) | 680 | 673 |

```sql
-- Fix: filtrar status antes del dedup
SELECT id, cancel_reason, cancelled_at FROM (
    SELECT id, cancel_reason, cancelled_at,
           ROW_NUMBER() OVER (PARTITION BY id ORDER BY cancelled_at DESC NULLS LAST) AS rn
    FROM staging.stg_chargebee__subscriptions_snapshot
    WHERE cancelled_at IS NOT NULL
      AND status = 'cancelled'
) t WHERE rn = 1
```

- **El mismo dedup está en 8 queries de extracción**: `involuntary_churn_facts`, `involuntary_churn_usage_trajectory`, `involuntary_lifetime_bookings`, `card_rescue_book`, `card_on_file_churn`, `card_on_file_new_merchants`, `churn_executive` y `churn_executive_ai`. `churn_executive` alimenta Churn Operational, Churn Report, Churn Cohorts, Churn BU, Card on File y el board book. Si solo se arregla `involuntary_churn_facts`, Involuntary Churn deja de cuadrar con esos reportes.

### 2. Churn Tracker → `/general/churn-tracker`
- En la página, la query `agg` etiqueta el bucket del budget como `'Graduated'` (`pages/general/churn-tracker/index.md`, línea 222). Los actuals vienen como `'Graduado'` desde `dwh.churn_tracker_vs_budget`. Por eso el `FULL OUTER JOIN ... USING (segment, bucket)` no empareja, y el KPI que filtra `bucket = 'Graduado'` queda con budget 0.
- El total no se ve afectado, porque suma todas las filas. Solo se rompe el desglose de graduado.
- Lo introdujo el commit `c3e557a1` (i18n del Churn Tracker, 24-sep).
- Segundo problema: la frase «graduated is at or under plan» (línea 24) está escrita fija y no depende del dato. Hay que hacerla condicional aunque se arregle la etiqueta.
- Budget `MRR Change Churn Graduado` de oct-2026 en `dwh.budget_2026_waterfall` (al 8-oct): **USD −11.332** (B2B3 −7.662 · B2C −2.651 · B2B2 −1.019). Cuadra con lo que cita Pablo.

### 3. Card Payment Recovery → `/finance-ops/card-recovery`
- `sources/dwh/payments/card_rescue_book.sql`, línea 266: el código `317` está en la lista de «5. Rechazo del banco», cuya acción sugerida es «contactar / cambiar medio».
- Según Pablo, el 317 es el bloqueo «high risk» de dLocal y se resuelve desbloqueando en dLocal, sin contacto con el cliente. Necesita su propio grupo.
- Hay que tocar el `CASE` de la query, el `seriesOrder` y el mapeo ES→EN de la página, la tabla de grupos (línea 233) y la taxonomía en `docs/METRIC_DEFINITIONS.md`.
- Esta query también tiene el dedup del punto 1: al arreglarlo, cambia el conteo de cuentas INACTIVA.

## Caveats
- El «+15% sobre el plan» del punto 2 no está verificado: depende de la proyección lineal del mes en curso al momento del refresh.
- Las 35 cuentas y USD 1.925 de MRR con código 317 no están verificadas: no hay parquet local de `card_rescue_book` y la query es pesada (escanea payloads SUPER de `stg_chargebee__events`).
- La ficha [[Churn voluntario vs involuntario]] (y la skill `finance-metric-definitions` de la que viene) define el grano como «most recent cancellation per subscription». Ese criterio es el que produce el error y hay que corregirlo en la skill cuando el fix esté en producción.

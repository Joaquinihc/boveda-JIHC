---
type: metrica
fuente: skill finance-metric-definitions
temas: [clasificaciones]
actualizado: 2026-09-22
---

# Convenciones del DWH (leer primero)

## Data layer & conventions (read this first)

All table names refer to the **AgendaPro DWH on Redshift** — the canonical data layer, queryable
independently of any BI/dashboard project. Reference queries are **Redshift / PostgreSQL SQL** and use
the **real DWH column names**. Schemas you'll see: `dwh.*`, `finance.*`, `staging.*`,
`dwh_marketplace.*`.

- **FX convention (important for column names):** the base `_usd` column is **current / variable FX**
  (spot rate of the month); `_usd_constant` is **Constant FX**; `_usd_budget_2026` is **Budget** (only
  where present, e.g. `dwh.payments_revenue`). There is **no `_usd_current` suffix** in the DWH — a
  column literally named `mrr_usd` already *is* the current-FX figure.
- **Segment column naming:** the pre-aggregated revenue tables (`dwh.mrr_saas`, `dwh.mrr_ai`,
  `dwh.payments_revenue`) expose the segment as **`segment`**; the waterfalls
  (`dwh.revenue_waterfall_saas`) use **`customer_segment`**; the company master uses
  **`dwh.companies.first_customer_segment_2`**. Values are the same set `{B2B3, B2B2, B2C}`.
- **BI-layer metrics:** a few metrics are materialized only in the BI/dashboard layer (Evidence) under
  names that are **not** Redshift base tables. Those cards are marked **(BI-layer)** and give the
  **Redshift base inputs** so the metric stays reproducible against the DWH. Also note some tables are
  mirrored in a `reports.*` schema; the canonical source is the `dwh.*` / `finance.*` version.

## Confidentiality disclaimer

This document is **methodology only**. It deliberately contains **no live figures** — no revenue, MRR,
ARR, GMV, cost, salary, headcount or merchant-count values; no budget targets or budget FX rates; no
merchant names/ids or whitelists; no personal attributions. It **does** name internal DWH tables and
**purely definitional thresholds** (e.g. activation bookings by segment, GMV outlier caps by grain)
because those are part of the method, not sensitive results. Treat table/column names as internal, and
do not add live numbers when reusing this file.

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

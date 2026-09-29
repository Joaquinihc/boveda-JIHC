---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# FX "constant"

- **Definition**: A volume-isolating FX scenario.
- **Formula**: Each currency converted at its **last available month's rate** — NOT a fixed calendar
  month. Materialized in the `_usd_constant` columns; rates from `dwh.int_exchange_rates_unpivoted`.
- **Grain**: Per currency.
- **Key rules / filters**: Never hardcode a calendar month (e.g. "January 2026") as "constant FX".
  Applies whenever the FX selector = Constant / you read a `_usd_constant` column.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

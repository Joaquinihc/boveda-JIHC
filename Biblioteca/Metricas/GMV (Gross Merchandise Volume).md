---
type: metrica
fuente: skill finance-metric-definitions
temas: [volumen]
actualizado: 2026-09-22
---

# GMV (Gross Merchandise Volume)

- **Definition**: Total value of sales/bookings registered in AgendaPro (before fees), USD. Distinct from
  **GPV** (processed payment volume) and from **Payments Revenue** (fees earned).
- **Formula**: `SUM(gmv_usd)`; avg ticket = `gmv_usd / booking_count`.
- **Grain**: Varies by surface (location-month, company-month, zone-month).
- **Key rules / filters — OUTLIER RULE IS PER GRAIN (do NOT harmonize to one threshold)**:

  | Grain | Rule | Note |
  |---|---|---|
  | **LOCATION** | drop **avg ticket > USD 300/booking** | + whitelist of legit high-ticket merchants |
  | **COMPANY-MONTH** | drop company-months with **avg ticket > USD 300/booking** | same whitelist |
  | **COMPANY** | drop **avg ticket > USD 5,000** | same whitelist |
  | **COMPANY** | drop **GMV > $1M/mo OR avg ticket > $10K** | FX-corrupted-row cap |
  | **COMPANY** | prev-month **GMV ≥ $500K** as a **sizing cap** | different purpose, not an outlier drop |

  Each threshold is canonical **only at its grain** — the LOCATION $300/booking rule is not a substitute
  for the COMPANY-grain $5K/$10K/$1M caps. A separate GMV-Growth surface *truncates* at
  `LEAST(gmv_usd, 100000)` and filters `gpv_usd <= 100000 AND gmv_usd < 500000` — a truncate, not an
  avg-ticket drop, so its absolute GMV is not comparable to the $300/booking surfaces. The high-ticket
  whitelist is a maintained list applied at each grain (BI-layer).
- **Source tables**: `dwh.company_sales_months`; location/zone marketplace GMV feeds.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

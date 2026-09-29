---
type: metrica
fuente: skill finance-metric-definitions
temas: [adquisicion]
actualizado: 2026-09-22
---

# Total Merchants (EOM stock)

- **Definition**: End-of-month **stock** of active merchants across all streams — the board book's headline
  "Total Merchants". Counted at the **venue/location grain**: each active branch counts, so one company
  with N branches contributes N merchants.
- **Formula**: `SUM(dwh.mrr.venues)` filtered to `is_active = TRUE`, per `date_month`, joined to
  `dwh.int_company_dimensions` for the slices. The **level of a period = the value at the last month with
  data** in that period (a stock — for the in-progress period it's QTD, i.e. the latest closed month).
- **Grain**: Venue/location; monthly (EOM by `date_month`); sliceable by `country_group × segment2 ×
  vertical × macro_niche`. B2B3 Total Merchants = same with `segment2 = 'B2B3'`.
- **Key rules / filters**:
  - **The count gate is `is_active`, NOT `is_active_and_paid`.** In `dwh.mrr`, `is_active` gates the
    merchant/venue **count**; `is_active_and_paid` gates the **MRR** figure. The `country_merchants`
    extraction proves the split: it counts `merchants` with `WHERE is_active` and sums `mrr_usd` with
    `WHERE is_active_and_paid` off the *same* table. Using `is_active_and_paid` for the count undercounts
    active-but-not-currently-paying merchants.
  - **Stock, not flow**: for a quarter/window take the **latest month** (`max_by(merchants, month)` / last
    snapshot) — never `SUM` across months. (Contrast New Merchants, which *is* summed.)
  - Uses the **`venues`** column gated by row-level `is_active` — not `venues_active_and_paid`.
  - **Venue grain is canonical** for total merchants (ties New Merchants and the Budget plan). This is the
    opposite grain from **AI company counts** (unique companies, `is_active_and_paid` — see the AI Revenue
    card and the "How many companies / merchants" default in `SKILL.md`): a bare "total merchants" is
    venue-grain here; an AI-vertical "how many companies" is unique-company-grain there.
- **Source tables**: `dwh.mrr` (`venues`, `is_active`, `date_month`, `company_id`),
  `dwh.int_company_dimensions` (`country_group`, `segment2`, `vertical`, `macro_niche`, `company_id`).
  (BI-layer: the `country_merchants` extraction pre-aggregates exactly this. A *by-vertical* merchants
  pivot in the board book instead reads a separate market-share mart, `dwh.market_share_merchants_monthly`,
  which can differ slightly — the **headline Total Merchants is the `dwh.mrr` venue/`is_active` definition
  above**.)
- **Reference query**:
```sql
-- [DWH · Redshift] canonical EOM Total Merchants (venue grain, active)
-- Stock: filter to ONE target month; do NOT sum across months. In-progress month is partial.
SELECT m.date_month::date AS report_month,
       cd.country_group, cd.segment2, cd.vertical,
       SUM(m.venues) AS merchants
FROM dwh.mrr m
JOIN dwh.int_company_dimensions cd ON cd.company_id = m.company_id
WHERE m.is_active                              -- count gate (NOT is_active_and_paid)
  AND m.date_month = DATE '2026-07-01'         -- your target EOM month
GROUP BY 1, 2, 3, 4
ORDER BY 1, 2, 3, 4;
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

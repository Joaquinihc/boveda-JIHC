---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# ASP AI (average new-sale ticket per new AI company)

- **Definition**: The average selling price of AI to a **newly acquired** AI company — new AI revenue
  (recurring + setup) per new AI company in the period. It's an **acquisition/pricing** metric (average
  new deal), NOT the ARPA of the whole base. Reported in the Board Book, by country and by vertical.
- **Formula** (recompute the ratio from **period sums** — never average monthly ASPs):
  `ASP AI = (new AI MRR + AI setup charges) / new AI companies`, where
  - **new AI MRR** = `SUM(mrr_change_usd)` from `dwh.revenue_waterfall_ai` where `change_category='new'`
    (current FX; the raw waterfall carries only `mrr_change_usd`, no `_constant`). This is a **flow** —
    legitimate use of the AI waterfall (distinct from the retired cumsum-as-stock). `dwh.revenue_waterfall_ai`
    is **company × month grain and has no `vertical` column** — for a per-vertical/country apertura,
    attach the company's vertical via a join (e.g. from `dwh.mrr_ai` or `dwh.company_attributes`).
  - **AI setup charges** = `SUM(mrr_plus_setup_charges_usd − mrr_usd)` from `dwh.mrr_ai` (one-time,
    charged the sale month).
  - **denominator = new AI companies** = unique companies at their **first appearance** in `dwh.mrr_ai`
    (incl. setup-only) — **company grain, NOT venues** (the AI product is company-wide).
- **Grain**: period (month or quarter) × apertura (`country_group` or `vertical`).
- **Key rules / filters**:
  - Denominator is **unique companies**, consistent with "AI is measured in companies" — never
    venues/`plan_quantity`.
  - Recompute from period sums: `(Σ new_mrr + Σ setup) / Σ new_companies`; do not average monthly ASPs.
  - New-companies count is materialized in the BI-layer as `new_companies_ai` (first appearance in
    `dwh.mrr_ai`); reconstruct from the first-appearance month against `dwh.mrr_ai`.
- **Source tables**: `dwh.revenue_waterfall_ai` (new-MRR flow) + `dwh.mrr_ai` (setup + new companies).
- **Reference query** (pattern):
```sql
-- [DWH · Redshift] ASP AI = (new AI MRR + setup) / new AI companies, all-up, current FX
-- Per-vertical apertura: join company_id -> vertical (dwh.revenue_waterfall_ai has no vertical column)
WITH first_appear AS (   -- new AI company = its first month present in dwh.mrr_ai (incl. setup-only)
  SELECT company_id, MIN(date_trunc('month', month)) AS first_m FROM dwh.mrr_ai GROUP BY 1
),
new_mrr AS (
  SELECT date_trunc('month', month) AS m, SUM(mrr_change_usd) AS new_mrr
  FROM dwh.revenue_waterfall_ai WHERE change_category = 'new' GROUP BY 1
),
setup_new AS (
  SELECT date_trunc('month', a.month) AS m,
         SUM(a.mrr_plus_setup_charges_usd - a.mrr_usd) AS setup,
         COUNT(DISTINCT CASE WHEN date_trunc('month', a.month) = f.first_m THEN a.company_id END) AS new_companies
  FROM dwh.mrr_ai a JOIN first_appear f ON f.company_id = a.company_id
  GROUP BY 1
)
SELECT s.m,
       (COALESCE(n.new_mrr, 0) + s.setup) / NULLIF(s.new_companies, 0) AS asp_ai
FROM setup_new s LEFT JOIN new_mrr n ON n.m = s.m
ORDER BY 1;
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

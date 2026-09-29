---
type: metrica
fuente: skill finance-metric-definitions
temas: [adquisicion]
actualizado: 2026-09-22
---

# New Merchants (nuevas altas)

- **Definition**: Volume of new paying merchants added in a month, counted in **venues/locations** — not
  distinct companies.
- **Formula** (two equivalent venue-grain sources):
  - `SUM(quantity_impact)` from `dwh.revenue_waterfall_saas` where `change_category = 'new'`.
  - New-first-month-paying counts from `dwh.merchants_segments` (`plan_quantity`, `is_first_month_paying`).
- **Grain**: Per month × `customer_segment` (in `dwh.revenue_waterfall_saas`) × country.
- **Key rules / filters**:
  - **Venue grain is canonical** (consistent with the merchant stock and the Budget plan). One company
    onboarding several branches = N venues, 1 company.
  - Segment column in `dwh.revenue_waterfall_saas` is **`customer_segment`** (not `segment2`).
  - **Exception — POS Attachment Rate denominator stays company-grain** (`COUNT(DISTINCT company_id)`),
    because its numerator is company-grain; both sides of that rate must be companies.
- **Source tables**: `dwh.revenue_waterfall_saas`, `dwh.merchants_segments`, `dwh.int_company_dimensions`.
  (A per-country/segment "new merchants detail" is materialized in the **BI-layer**; base:
  `dwh.revenue_waterfall_saas`.)
- **Reference query**:
```sql
-- [DWH · Redshift] canonical venue-grain New Merchants
SELECT date_trunc('month', month) AS month, customer_segment,
       SUM(quantity_impact) AS new_merchants
FROM dwh.revenue_waterfall_saas
WHERE change_category = 'new'
GROUP BY 1, 2 ORDER BY 1, 2;
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

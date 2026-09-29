---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# Payments Revenue Churn ("Revenue, not MRR")

- **Definition**: Monthly churn of Payments processing revenue (fees), reported on a **Revenue** basis,
  not an MRR basis.
- **Grain**: Monthly. Payments stream only.
- **Key rules / filters (load-bearing)**: **Cap `month <= current/selected`** to hide a future-month
  cancellation spike. Scheduled `non_renewing` cancellations (Chargebee) pile into a future-month bucket
  (~tens of times normal revenue) with **no per-row date** to filter — the month cap is the only lever.
- **Source tables**: Payments churn feed derived from the Payments waterfall / Chargebee
  (`staging.stg_chargebee__subscriptions_snapshot` for `non_renewing`).
- **Reference query**:
```sql
-- [DWH · Redshift] pattern: always cap the month to hide the future scheduled-cancellation spike
-- (replace <payments_churn_source> with the Payments churn feed; churn is on fee revenue, not MRR)
SELECT month, SUM(churned_fee_usd) AS payments_revenue_churn
FROM <payments_churn_source>
WHERE month <= date_trunc('month', CAST(:selected_month AS DATE))
GROUP BY 1 ORDER BY 1;
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

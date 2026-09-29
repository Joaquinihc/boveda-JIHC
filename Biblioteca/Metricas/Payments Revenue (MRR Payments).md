---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# Payments Revenue (MRR Payments)

- **Definition**: The Payments stream: processing fees (in-person POS + online web) **plus** rental and
  sale of POS terminals.
- **Formula**: `Payments Revenue = SUM(fee_usd)` from `dwh.payments_revenue`, where
  **`fee_usd = fee_pos_usd + fee_web_usd + pos_devices_usd`** (in-person POS processing + online
  processing + POS terminal rental & sale). FX columns `fee_usd` (current) / `fee_usd_constant` /
  `fee_usd_budget_2026`.
- **Grain**: Monthly × `segment` (`month` is a timestamp).
- **Key rules / filters**:
  - Distinct from **GMV** (booked value, before fees) and **GPV** (processed payment volume) — this is
    the *fee* AgendaPro earns plus device revenue, not the volume.
  - POS recurring (terminal rental, inside `pos_devices_usd`) lives **here**, not in SaaS
    (no-double-count rule).
  - `dwh.payments_revenue` has **no `is_active_and_paid` flag** — these are realized fees; do **not**
    add such a filter (unlike SaaS/AI).
- **Source tables**: `dwh.payments_revenue`; upstream GMV→GPV→Fee path `dwh.company_sales_months`
  (legacy, not the canonical source).
- **Reference query**:
```sql
-- [DWH · Redshift] Payments Revenue by month, current FX
SELECT month, SUM(fee_usd) AS payments_revenue,
       SUM(fee_pos_usd) AS fee_pos, SUM(fee_web_usd) AS fee_web, SUM(pos_devices_usd) AS pos_devices
FROM dwh.payments_revenue
GROUP BY 1 ORDER BY 1;
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

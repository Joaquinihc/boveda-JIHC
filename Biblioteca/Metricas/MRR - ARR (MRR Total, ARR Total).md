---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# MRR / ARR (MRR Total, ARR Total)

- **Definition**: Recurring monthly (MRR) / annualized (ARR) revenue run-rate across the three streams.
- **Formula**:
  - `MRR Total = MRR SaaS + MRR Payments + MRR AI`; `ARR Total = MRR Total × 12`.
  - `MRR SaaS` = `SUM(mrr_usd)` from `dwh.mrr_saas` (already net of IA + POS recurring — see next card).
  - `MRR Payments` = `SUM(fee_usd)` from `dwh.payments_revenue`.
  - `MRR AI` = `SUM(mrr_usd)` from `dwh.mrr_ai` (stock).
- **Grain**: Monthly; FX via the `_usd` (current) / `_usd_constant` columns.
- **Key rules / filters**:
  - **No double counting (load-bearing).** The three streams are mutually exclusive: POS recurring
    (terminal rental) lives **only** in Payments; AI lives **only** in the AI stream; SaaS is stripped
    of both. Do **not** add raw `dwh.mrr` (which already includes IA + addons + POS recurring) on top of
    the stream legs.
  - **Use one FX basis consistently** — sum all three legs in `_usd` (current) *or* all in
    `_usd_constant`; never mix.
  - **Apply `is_active_and_paid = TRUE` on `dwh.mrr_saas` and `dwh.mrr_ai`.** These tables **carry the
    flag and are NOT pre-filtered** — without it you also sum recurring MRR that isn't being collected
    (churn/dunning). `dwh.payments_revenue` has **no** such flag (realized fees) → no filter there.
    `is_active` is a *different* flag that gates only the merchant/venue **count**, not the MRR figure.
    (Since the 2026-07 redefinition of `is_active_and_paid`, one-time setup/OTC charges of brand-new
    subscriptions count as active-and-paid, so a single `WHERE is_active_and_paid` on the
    `plus_setup` / `plus_one_time` column is correct — no need to gate the recurring leg separately.)
  - **AI MRR canonical = the `dwh.mrr_ai` stock**, NOT a waterfall cumsum (the old cumulative
    `SUM(mrr_change_usd)` understated the level and carried no country/FX dimension — retired).
  - **MRR (run-rate) ≠ Revenue stream**: Finance "Revenue" adds one-time charges (SaaS Revenue = MRR
    SaaS + OTC → `mrr_plus_one_time_charges_usd`; AI Revenue = MRR AI + setup →
    `mrr_plus_setup_charges_usd`). Don't equate "MRR SaaS" with "SaaS Revenue".
  - **Tax basis**: MRR SaaS is treated **net of IVA**, to compare like-for-like with cash-basis
    recognized revenue (invoice `sub_total`).
- **Source tables**: `dwh.mrr_saas`, `dwh.payments_revenue`, `dwh.mrr_ai`; upstream (not the stream
  sources) `dwh.mrr`, `dwh.company_sales_months`.
- **Reference query**:
```sql
-- [DWH · Redshift] MRR Total = SaaS + Payments + AI, current FX (base _usd), no double counting
WITH s AS (SELECT month, SUM(mrr_usd) AS mrr FROM dwh.mrr_saas         WHERE is_active_and_paid GROUP BY 1),
     p AS (SELECT month, SUM(fee_usd) AS mrr FROM dwh.payments_revenue GROUP BY 1),  -- no is_active_and_paid flag (realized fees)
     a AS (SELECT month, SUM(mrr_usd) AS mrr FROM dwh.mrr_ai           WHERE is_active_and_paid GROUP BY 1)
SELECT COALESCE(s.month, p.month, a.month) AS month,
       COALESCE(s.mrr,0) + COALESCE(p.mrr,0) + COALESCE(a.mrr,0)         AS mrr_total,
      (COALESCE(s.mrr,0) + COALESCE(p.mrr,0) + COALESCE(a.mrr,0)) * 12   AS arr_total
FROM s FULL OUTER JOIN p USING (month) FULL OUTER JOIN a USING (month)
ORDER BY 1;
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

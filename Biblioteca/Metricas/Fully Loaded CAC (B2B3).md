---
type: metrica
fuente: skill finance-metric-definitions
temas: [unit-economics]
actualizado: 2026-09-22
---

# Fully Loaded CAC (B2B3)

- **Definition**: **Total Acquisition Costs ÷ New Merchants B2B3**, where Total Acquisition Costs = the
  full P&L bucket `3- Acquisition Costs` (Marketing + Sales + Customer Success Onboarding — not just
  media spend). Marketplace acquisition costs stay **included**.
- **Formula**:
```
Total Acquisition Costs (month, country) = -SUM(amount_usd) WHERE p_l = '3- Acquisition Costs'
Fully Loaded CAC = Total Acquisition Costs ÷ New Merchants B2B3
```
- **Grain**: Month (or calendar quarter — sum numerator and denominator before dividing) × country_group.
- **Key rules / filters**:
  - By-country: HQ costs prorated by each period's New Merchants B2B3 share; TikTok (booked under Mexico)
    reallocated pro-rata by Paid Social B2B3 leads. Both zero-sum.
  - Distinct from the Board-Book Key-Metrics CAC (which **excludes** Marketplace acquisition costs and so
    runs slightly below this).
  - Denominator is **B2B3 only** even though the cost pool funds all segments (intentional).
- **Source tables**: `finance.profit_and_loss` (bucket `p_l = '3- Acquisition Costs'`); denominator =
  New Merchants B2B3 (**BI-layer**; base: `dwh.revenue_waterfall_saas` `change_category='new'`,
  `customer_segment='B2B3'`).

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

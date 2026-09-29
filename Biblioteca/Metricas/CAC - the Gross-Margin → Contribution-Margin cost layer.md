---
type: metrica
fuente: skill finance-metric-definitions
temas: [unit-economics]
actualizado: 2026-09-22
---

# CAC — the Gross-Margin → Contribution-Margin cost layer

- **Definition**: For the Yield-on-CAC / Growth-Efficiency family, CAC = the **per-acquired-merchant
  cost layer that sits between a line's Gross Margin and its Contribution Margin**. Per business
  line × geography, at **Constant FX**.
- **Formula**:
```
CAC layer (per unit) = (Gross Margin − Contribution Margin) of the line/geo, summed over the pooled cohort months
CAC per merchant     = CAC layer ÷ new merchants acquired in those months (period-1 cohort survivors)
```
  - SaaS → `3- Acquisition Costs` (direct + prorated HQ pool). Payments → Payments team + POS/ops +
    card-integration + Sales-Payments, allocated to country by revenue share. AI → the vertical's P&T
    salaries.
- **Grain**: Per line × geo; pooled over the mature cohort window; Constant FX.
- **Key rules / filters**:
  - **NOT** the retired `CAC ÷ ARPU` payback input, and **NOT** the marketing channel-level "CAC $".
  - HQ acquisition pool is split by country **dynamically, by each period's New Merchants B2B3 share**
    (zero-sum).
  - Per-**segment** SaaS CAC is an **explicit modeling assumption** (P&L not segment-split): Selfserve
    (B2B2+B2C) = a flat assumed per-merchant CAC; B2B3 = remainder that reconciles to the pool; All =
    blended. Only SaaS is segmented — Payments/AI are `segment='All'`.
- **Source tables**: `finance.profit_and_loss` (`type='Real'`, Constant FX); cohort/new-merchant split
  from `dwh.nrr_cohorts`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

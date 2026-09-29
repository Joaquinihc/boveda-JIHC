---
type: metrica
fuente: skill finance-metric-definitions
temas: [unit-economics]
actualizado: 2026-09-22
---

# Payback (Mensual)

- **Definition**: Months to recover the acquisition cost of a monthly cohort of new **B2B3** merchants,
  via gross margin on their New MRR (SaaS) plus a Payments contribution.
- **Formula** (unified canonical method):
```
          Direct_acq + Allocated_HQ + Discount_window[-1,+2]
Payback = ───────────────────────────────────────────────────────────
          New_MRR_SaaS (gross = net + month-0 discount) × GM_SaaS + Payments
```
  - Numerator = acquisition cost + the **full discount window [−1, +2]** (current FX). `Allocated_HQ` =
    HQ cost **prorated by each country's share of B2B3 New Merchants** (`HQ × nm_country / nm_total`);
    `Direct_acq` = `cost_center ≠ 'HQ'`.
  - Denominator = New MRR SaaS already **gross** (net + month-0 list-price discount, so month-0 discount
    is not re-added) × GM SaaS, plus a Payments term.
  - `GM SaaS = (Software + AI Revenue + Service Costs excl. Payment Processing) / (Software + AI
    Revenue)`; `GM Payments = (Payments Revenue + Payment Processing) / Payments Revenue` — global
    monthly % from `finance.profit_and_loss`.
- **Grain**: Monthly × country, **B2B3 only**. Reliable window ≈ 2024-10 → last closed month; drop
  partial-load months.
- **Key rules / filters**:
  - Legacy `CAC ÷ ARPU` variant is **RETIRED**; activation-rate scaling was **REJECTED** — discount
    enters the numerator at 100%.
  - Payback **Total** (incl. HQ prorated by B2B3 New-Merchant share) vs **Variable** (direct only,
    `cost_center ≠ 'HQ'`).
  - Marketplace cost toggle (default Exclude; acquisition code `G1011` + Marketplace ad spend).
  - Discount classification: **standard (3-month)** vs **non-standard (6-month, e.g. Happy Week)** coupon
    buckets — the window `[−1,+2]` is applied regardless of bucket.
  - *Payback Cohorts* (realized cohort GM with churn, no discount toggle) is a **different** construct —
    not directly comparable.
- **Source tables**: `finance.profit_and_loss`; discount amount and New-Merchant B2B3 counts are
  materialized in the **BI-layer** (base inputs: `dwh.revenue_waterfall_saas`,
  `staging.stg_chargebee__invoices` / coupons).

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

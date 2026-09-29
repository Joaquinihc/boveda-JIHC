---
type: metrica
fuente: skill finance-metric-definitions
temas: [retencion]
actualizado: 2026-09-22
---

# Churn — Gross/Net × MRR/Logo

- **Definition**: Loss of MRR (revenue basis) or accounts (logo basis) from the installed base, gross
  (before reactivations) or net (after).
- **Formula**:
  - `Gross MRR Churn % = MRR cancelled / MRR start-of-period`.
  - `Net MRR Churn % = (MRR cancelled − MRR reactivations) / MRR start-of-period` (can be negative).
  - `Gross / Net Logo Churn %` = the same on account counts.
  - Neither includes expansion/contraction (that is NRR).
- **Grain**: Monthly; rate of month M = numerator ÷ starting base of month M−1.
- **Key rules / filters**:
  - Denominator from `dwh.mrr_saas` (segment×vertical grain) so it respects Segment + Vertical filters —
    **not** the country-only MRR base.
  - Churn types (in `dwh.revenue_waterfall_software.change_category`): `churn_graduado`
    (post-activation) vs `churn_onboarding` (pre-activation, < 3 months active MRR).
  - Payments/AI churn = addon fee/MRR to $0 (excludes downgrade); scheduled `non_renewing`
    cancellations land in a future-month bucket (see Payments Revenue Churn).
- **Source tables**: `dwh.mrr_saas`, `dwh.revenue_waterfall_software`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

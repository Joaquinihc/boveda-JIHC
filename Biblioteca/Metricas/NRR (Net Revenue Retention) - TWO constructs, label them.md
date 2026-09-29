---
type: metrica
fuente: skill finance-metric-definitions
temas: [retencion]
actualizado: 2026-09-22
---

# NRR (Net Revenue Retention) — TWO constructs, label them

- **Definition**: How much MRR/revenue the installed base retains including expansion/contraction. **Two
  distinct constructs — do not conflate.**
- **Formula**:
  - **NRR Rolling 12m** = `(MRR 12m ago + Expansion + Reactivation + Contraction + Churn) / MRR 12m ago`.
    Benchmark 100%. Country grain.
  - **NRR por cohorte** = `revenue of the cohort's still-active accounts at tenure n / cohort revenue at
    month 1`. Default weighting = MRR×0.85 + (GPV fee − GPV cost), Constant FX. Source `dwh.nrr_cohorts`.
- **Grain**: Rolling-12m = monthly snapshot; cohort = (cohort month × tenure month) grid.
- **Key rules / filters (RATIFIED rule)**: The two numbers are **not expected to match**. **Never present
  both as "NRR" on the same surface without an explicit qualifying label** ("NRR Rolling 12m" vs "NRR por
  cohorte").
- **Source tables**: `dwh.nrr_cohorts` (cohort). NRR Rolling 12m uses a monthly MRR-base **(BI-layer;**
  base: `dwh.mrr`, country grain**)**.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

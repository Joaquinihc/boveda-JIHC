---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# MRR / Revenue SaaS (SaaS-pure)

- **Definition**: SaaS-only recurring revenue — SaaS stripped of IA and POS recurring.
- **Formula**: canonical = `SUM(mrr_usd)` from `dwh.mrr_saas` (current FX; `mrr_usd_constant` for
  Constant). Definitionally this equals `dwh.mrr − dwh.mrr_ai − POS_recurring`, where POS recurring =
  `dwh.int_pos_terminals_monthly.amount_local` with `category='recurring'`, subtracted per
  **company × month × currency** and converted to USD at row level. `SaaS Revenue` = MRR SaaS + OTC =
  `SUM(mrr_plus_one_time_charges_usd)`; OTC alone = `one_time_charges_usd`.
- **Grain**: Monthly × `segment` (× company/currency upstream).
- **Key rules / filters**: **Apply `WHERE is_active_and_paid = TRUE`** — `dwh.mrr_saas` carries the flag
  and is NOT pre-filtered; omitting it also sums recurring MRR from merchants that aren't paying
  (churn/dunning). The DWH table `dwh.mrr_saas` is canonical. A BI surface may recompute SaaS from raw
  `dwh.mrr` with a different FX source (~1% gap) — that is a source/FX difference, **not** a definitional
  one; the `dwh.mrr_saas` table is the arbiter.
- **Source tables**: `dwh.mrr_saas` (canonical); upstream `dwh.mrr`, `dwh.mrr_ai`,
  `dwh.int_pos_terminals_monthly`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

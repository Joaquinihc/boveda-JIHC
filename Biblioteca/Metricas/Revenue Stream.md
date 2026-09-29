---
type: metrica
fuente: skill finance-metric-definitions
temas: [otros]
actualizado: 2026-09-22
---

# Revenue Stream

- **Definition**: The product line a dollar of revenue comes from.
- **Values**: `{SaaS, Payments, AI}`, with the identity `MRR Total = MRR SaaS + MRR Payments + MRR AI`.
- **Grain**: Per movement / per month.
- **Key rules / filters**: The three streams are **mutually exclusive** (see MRR/ARR "no double
  counting"). Canonical per-stream sources: SaaS `dwh.mrr_saas`, Payments `dwh.payments_revenue`,
  AI `dwh.mrr_ai`. The raw upstream `dwh.mrr` and `dwh.company_sales_months` are **not** the stream
  sources.
- **Source tables**: `dwh.mrr_saas`, `dwh.payments_revenue`, `dwh.mrr_ai`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

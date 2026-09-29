---
type: metrica
fuente: skill finance-metric-definitions
temas: [volumen]
actualizado: 2026-09-22
---

# "Active POS merchant" — OVERLOADED (no single definition)

- **Definition**: "Comercio activo en POS" is **not one metric** — it has multiple, non-comparable
  definitions across surfaces (e.g. transacted with POS in the last 30 days; processed GPV > 0 in the
  month; reached ≥ N POS transactions lifetime; installed-base variants).
- **Key rules / filters**: **Always confirm which definition a surface uses before quoting or
  comparing.** Do not compare an "active POS merchant" count from one surface against another — they
  answer different questions. Not a canonical single figure.
- **Source tables**: varies (POS transaction / GPV / installed-base feeds off `dwh.company_sales_months`
  and POS terminal tables). Treat each occurrence as surface-local.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

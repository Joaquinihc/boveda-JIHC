---
type: metrica
fuente: skill finance-metric-definitions
temas: [pnl]
actualizado: 2026-09-22
---

# AI Gross Margin / AI Margin %

- **Definition**: AI gross margin (before any salaries) and its percentage of AI Revenue.
- **Formula**: `AI Gross Margin = AI Revenue − AI Cost`; `AI Margin % = (AI Revenue − AI Cost) / AI
  Revenue` (guarded, only when AI Revenue > 0).
- **Grain**: Monthly × vertical, USD.
- **Key rules / filters**: **Excludes salaries** — a *gross* margin, distinct from full-P&L Contribution
  Margin and from the AI "Result after P&T cost" (which subtracts P&T salaries and is **not** EBITDA).
  AI Cost is mapped directly per vertical (no shared-cost allocation).
- **Source tables**: AI Revenue from `dwh.mrr_ai`; AI Cost per the AI-P&L allocation.

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

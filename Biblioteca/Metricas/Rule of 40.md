---
type: metrica
fuente: skill finance-metric-definitions
temas: [pnl]
actualizado: 2026-09-22
---

# Rule of 40

- **Definition**: SaaS-health metric — revenue growth plus profitability margin should sum to ≥ 40%.
- **Formula**: `Rule of 40 = Revenue Growth YoY + EBITDA Margin YTD`, where
  `Revenue Growth YoY = MRR Total(month) / MRR Total(month − 12) − 1` (MRR-based, current FX) and
  `EBITDA Margin YTD = EBITDA_ytd / Revenue_ytd`.
- **Grain**: Monthly; YTD for the margin leg.
- **Key rules / filters**: A **snapshot** metric. The growth leg uses **MRR-based** ARR YoY, not P&L
  recognized-revenue growth — keep consistent across surfaces.
- **Source tables**: MRR sources (MRR/ARR card) + `finance.profit_and_loss`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

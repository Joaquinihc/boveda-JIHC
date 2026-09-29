---
type: metrica
fuente: skill finance-metric-definitions
temas: [pnl]
actualizado: 2026-09-22
---

# EBITDA / EBITDA Margin

- **Definition**: Operating profit before interest, tax, D&A; company-level P&L result.
- **Formula**: `EBITDA = SUM(amount_usd)` over `1- Revenue + 2- Service Costs + 3- Acquisition Costs +
  4- Corporate Costs` (costs negative), `type='Real'`. `EBITDA Margin = EBITDA / Revenue`;
  `EBITDA Margin YTD = EBITDA_ytd / Revenue_ytd`.
- **Grain**: Monthly and YTD.
- **Key rules / filters**: The Budget scenario budgets Payment Processing at $0 while actuals carry it —
  Budget EBITDA can look better than Real (not a sign bug).
- **Source tables**: `finance.profit_and_loss`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

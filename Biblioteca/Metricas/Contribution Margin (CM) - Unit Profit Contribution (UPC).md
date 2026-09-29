---
type: metrica
fuente: skill finance-metric-definitions
temas: [pnl]
actualizado: 2026-09-22
---

# Contribution Margin (CM) / Unit Profit Contribution (UPC)

- **Definition**: Country-unit P&L margins. `CM = Gross Profit − Acquisition Costs`;
  `UPC = CM − Other Direct Costs`.
- **Formula**: `Gross Profit Margin = Gross Profit / Revenue`; `CM % = CM / Revenue`;
  `UPC % = UPC / Revenue`.
- **Grain**: Monthly × country.
- **Key rules / filters**: Two views — **Direct Only** vs **Fully Loaded** (HQ prorated); both correct.
  Distinct from the AI-P&L "AI Margin %" and from the pre-overhead Contribution Margin in the
  Yield-on-CAC family (which does **not** sum to EBITDA).
- **Source tables**: `finance.profit_and_loss`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

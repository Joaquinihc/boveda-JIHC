---
type: metrica
fuente: skill finance-metric-definitions
temas: [pnl]
actualizado: 2026-09-22
---

# P&L Business-Line Decomposition (Verticales / Payments / Resto → EBITDA)

- **Definition**: A **MECE, exhaustive** partition of the company P&L into three business lines —
  Verticales (product SW+AI), Payments, Resto (overhead/GTM) — whose contributions **sum exactly to
  EBITDA**, with **no arbitrary allocation**.
- **Formula** (over `finance.profit_and_loss`, `type='Real'`, costs negative):
```
Verticales = (1- Revenue WHERE p_l_category <> 'Payments Revenue')
           + (2- Service Costs WHERE p_l_category <> 'Payment Processing')
Payments   =  Payments Revenue + Payment Processing              (= Payments Gross Margin)
Resto      =  3- Acquisition Costs + 4- Corporate Costs          (residual)
Identity:  Verticales + Payments + Resto  =  EBITDA
```
- **Grain**: Monthly × business-line; rolling 12 closed months; FX-driven.
- **Key rules / filters**:
  - **Two senses of "vertical" — do not conflate.** Line-of-business "Verticales" = the product (SW+AI)
    line; a **different** concept from the 4-industry Vertical taxonomy.
  - Only the P&L-sourced lines reconcile to EBITDA; per-industry / per-country mixes shown alongside are
    **indicative run-rate**, not recognized P&L revenue.
  - Confirm the exact `p_l` / `p_l_category` labels against the data
    (`SELECT DISTINCT p_l, p_l_category FROM finance.profit_and_loss`).
- **Source tables**: `finance.profit_and_loss`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

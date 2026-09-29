---
type: metrica
fuente: skill finance-metric-definitions
temas: [unit-economics]
actualizado: 2026-09-22
---

# Yield on CAC (annualized IRR on CAC)

- **Definition**: The **annualized IRR** of the per-merchant cohort cash-flow vector
  `[ −CAC (month 0), GM·cash₁, …, GM·cash₃₆ ]`. Conceptually the maximum self-funded growth rate.
- **Formula**:
```
Solve monthly IRR r:  −CAC + Σ_{t=1..36} GM·cash(t) / (1+r)^t = 0
GM·cash(t) = (month-1 GM cash per acquired merchant) × cohort GM-retention curve at t
Yield on CAC = (1 + r)^12 − 1          -- annualize
```
  - Month-0 outflow = the **CAC (GM→CM layer)** above. Cohort revenue per stream: SaaS subscription,
    Payments `gpv_fee`, AI, at Constant FX; "per merchant" = ÷ initial cohort size. Solved by bisection
    on `r`, then annualized — a **true IRR solve**, not a year-1 cash-on-cash ratio.
- **Grain**: Per business line × geography. Cohorts pooled over mature vintages; GM% = trailing 12
  closed months; Constant FX throughout.
- **Key rules / filters**:
  - Interpretive bands (Monashees machine model): 🟢 ≥ 100% self-funds growth · 🟡 80–100% · 🔴 < 80%.
  - Yearly Yield **clamped at 250%** per cell (`min(y, 2.5)`).
  - AI is **provisional/directional** (short history; tail extrapolated).
  - Uses recognized recurring revenue × GM% as a **cash proxy**.
  - **Not** interchangeable with Payback (Mensual): different segment scope, cohort survival, cost layer.
- **Source tables**: cohort feed `dwh.nrr_cohorts` + `dwh.mrr_ai`; CAC/GM% from `finance.profit_and_loss`
  (Constant FX).

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

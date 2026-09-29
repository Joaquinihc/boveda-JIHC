---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# Budget 2026 (plan / presupuesto)

- **Definition**: The 2026 plan (budget) targets — the yellow "Budget" line across the board-book and
  OKR. Split across **revenue/volume** tables (org-accessible) and **cost/headcount** tables
  (Finance-only).
- **Where it lives — and what the rest of the org can actually query:**
  - **Accessible (org-wide), `dwh` schema** — verified **identical** to the `finance` copies (same rows,
    0 diff), so **always point cross-team consumers at the `dwh` tables**, not `finance`:
    - `dwh.budget_2026_revenue` — budget of **revenue & volume KPIs** by `revenue_source ∈ {software,
      payments, ai}` × `metric`. Metrics available: **software** — Leads, New Merchants, New MRR,
      Activation Rate, Activated POS, Discount amount; **payments** — Revenue, New Revenue, GMV, GPV,
      New POS Sold, Active Merchants with POS, Activated POS, Cost, New Cost; **ai** — MRR, New MRR,
      Merchants, New Merchants, Churn MRR, Churn Merchants, ASP Julia, ASP Sofia.
    - `dwh.budget_2026_waterfall` — budget of the **MRR movement waterfall** (by `metric`: new / churn /
      expansion …).
  - **Restricted (Finance-only, NOT queryable by the rest of the org):**
    - `finance.budget_2026_costs` — budgeted P&L **cost** lines (`p_and_l` / `p_and_l_category` /
      `cost_center`).
    - `finance.budget_2026_hiring_plan` — budgeted **headcount / hiring plan**.
    - `finance.budget_2026_marketing_spend` — budgeted marketing-spend detail.
- **Structure**: **wide format** — dimension columns (`revenue_source`, `metric`, `country_group`,
  `segment`, `vertical`, `macro_niche`, `lead_source`, `canal`, `producto`) + **one numeric column per
  month** named `'YYYY-MM-01'` (`'2025-12-01'` … `'2026-12-01'`). Select or unpivot the month column(s).
- **Key rules / filters**:
  - Revenue / volume / MRR / GMV / GPV / POS / Leads / AI budget → **query the `dwh` tables** (accessible
    and identical to `finance`). Do **not** route anyone to `finance.budget_2026_revenue/waterfall`.
  - **Cost, headcount and marketing-spend budget can only be reproduced by Finance** (`finance.*`,
    restricted). If someone without Finance access needs the EBITDA / cost / headcount budget, that
    figure is **not reproducible for them** — say so; don't route them to a table they can't read.
  - It's a **plan**, not actuals — pair with the matching actual metric to compute variance
    (actual vs budget).
- **Source tables**: accessible `dwh.budget_2026_revenue`, `dwh.budget_2026_waterfall`; restricted
  `finance.budget_2026_costs`, `finance.budget_2026_hiring_plan`, `finance.budget_2026_marketing_spend`.
- **Reference query**:
```sql
-- [DWH · Redshift] budget for a revenue-side metric (wide → pick the month column). Org-accessible.
SELECT country_group, segment, vertical, "2026-06-01" AS budget_jun_2026
FROM dwh.budget_2026_revenue
WHERE revenue_source = 'ai' AND metric = 'MRR';
-- Cost / headcount budget lives only in finance.budget_2026_costs / _hiring_plan (restricted) — not
-- reproducible without Finance access.
```

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

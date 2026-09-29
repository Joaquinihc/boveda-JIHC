---
type: metrica
fuente: skill finance-metric-definitions
temas: [revenue]
actualizado: 2026-09-22
---

# AI Revenue (AI MRR run-rate)

- **Definition**: Monthly recurring revenue from the AI add-ons (Julia + Sofía + Agente AI) **plus
  one-time setup charges**.
- **Formula**: `AI Revenue = SUM(mrr_plus_setup_charges_usd)` from `dwh.mrr_ai` (stock; current FX).
  AI MRR (pure, no setup) = `SUM(mrr_usd)`. Constant FX = the `_constant` variants
  (`mrr_usd_constant`, `mrr_plus_setup_charges_usd_constant`).
- **Grain**: Monthly × `segment` (per-product columns `mrr_julia_usd`, `mrr_sofia_usd`,
  `mrr_agente_ai_usd` also exist).
- **Key rules / filters**:
  - **Canonical = the `dwh.mrr_ai` stock** (not a waterfall cumsum). Do not accumulate
    `SUM(mrr_change_usd)`.
  - `dwh.mrr_ai` already **includes Agente AI** (`mrr_agente_ai` / `mrr_agente_ai_usd`) — do **not** add
    a separate Agente-AI waterfall on top (double counts).
  - **Apply `WHERE is_active_and_paid = TRUE`** — `dwh.mrr_ai` carries the flag and is NOT pre-filtered.
    Since the 2026-07 redefinition, setup charges of brand-new AI subscriptions count as active-and-paid,
    so the filter no longer drops them; it only excludes genuinely unpaid recurring AI (a negligible
    amount). Before that redefinition, gating dropped new-subscription setup and understated AI Revenue —
    that was the cause of an AI figure reading ~8% low on standalone queries.
  - The label is "AI Revenue" for suite naming consistency; the underlying value is an **MRR run-rate**,
    not recognized/booked revenue.
  - **Per-vertical AI revenue** — `dwh.mrr_ai` has a `vertical` column (`{Beauty, Health, Medspa, Other}`);
    filter it for one vertical. This is the **default lens** when someone asks for the revenue of the
    **Beauty / Medspa / Health** verticals (AI-focused teams) — answer with their AI revenue first (see
    the "Response focus & defaults" rule in `SKILL.md`).
  - **AI count grain = unique companies, NOT venues.** The AI product is company-wide (it works across
    **all** of a company's branches), so count `COUNT(DISTINCT company_id)` (active-and-paid), **not**
    `SUM(venues)` / `plan_quantity`. This is the default when asked "how many merchants / companies" for
    the AI verticals. SaaS instead uses the **venue** grain (`dwh.mrr.venues` /
    `dwh.merchants_segments.plan_quantity`; see MRR SaaS / New Merchants).
- **Source tables**: `dwh.mrr_ai`. (A per-vertical AI split is materialized in the **BI-layer**; its
  base is `dwh.mrr_ai`.)
- **Reference query**:
```sql
-- [DWH · Redshift] AI Revenue (MRR + setup) and pure AI MRR, current FX
SELECT month,
       SUM(mrr_usd)                     AS ai_mrr_pure,
       SUM(mrr_plus_setup_charges_usd)  AS ai_revenue
FROM dwh.mrr_ai
WHERE is_active_and_paid
GROUP BY 1 ORDER BY 1;

-- AI revenue for a single vertical (default lens for Beauty / Medspa / Health)
SELECT month, SUM(mrr_plus_setup_charges_usd) AS ai_revenue
FROM dwh.mrr_ai
WHERE is_active_and_paid AND vertical = 'Beauty'   -- or 'Medspa' / 'Health'
GROUP BY 1 ORDER BY 1;

-- AI company count (unique companies; AI is company-wide, NOT per-venue)
SELECT month, COUNT(DISTINCT company_id) AS ai_companies
FROM dwh.mrr_ai
WHERE is_active_and_paid   -- add: AND vertical = 'Beauty' for one vertical
GROUP BY 1 ORDER BY 1;
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

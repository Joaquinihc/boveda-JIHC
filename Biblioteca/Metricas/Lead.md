---
type: metrica
fuente: skill finance-metric-definitions
temas: [adquisicion]
actualizado: 2026-09-22
---

# Lead

- **Definition**: **1 lead = 1 HubSpot contact**, counted by the month of its **first conversion**
  (`first_conversion_date`, an **immutable** event) — NOT by `hubspot_owner_assigneddate` (mutable).
- **Formula**: `leads = COUNT(*)` of contacts grouped by `DATE_TRUNC('month', first_conversion_date)`;
  country from `contact.country` (normalize `México→Mexico`), segment from `numemployees`.
- **Grain**: Per owner × country_group × segment × conversion month; internal owners only.
- **Key rules / filters**:
  - Use `first_conversion_date` (immutable). `hubspot_owner_assigneddate` moves on bulk reassignments and
    causes false spikes — do not use it.
  - **Two-owner attribution (load-bearing)**: leads → the **lead owner** (`hubspot_owner_id`); New
    Merchants → the **commission owner** (via `finance.int_comisiones_hubspot_owners`, ~93% coverage).
    Different owners ⇒ a **per-rep CVR is invalid** — do not compute one.
  - Two lead universes: productivity lens = `staging.stg_hubspot__contacts` by `first_conversion_date`;
    funnel/OKR lens = `dwh.fct_lead_conversions`. Not guaranteed to tie 1:1 — pick the right one.
- **Source tables**: `staging.stg_hubspot__contacts`, `finance.int_comisiones_hubspot_owners`,
  `dwh.fct_lead_conversions`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

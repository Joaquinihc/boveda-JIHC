---
type: metrica
fuente: skill finance-metric-definitions
temas: [retencion]
actualizado: 2026-09-22
---

# Churn voluntario vs involuntario

- **Definition**: Split of a cancelled SaaS account by whether cancellation was driven by a
  billing/collection failure (Involuntario) or the customer choosing to leave (Voluntario).
- **Canonical predicate**: **Involuntario ⟺ `cancel_reason = 'not_paid'`** (dunning exhausted), over SaaS
  churn (`change_category IN ('churn_graduado','churn_onboarding')`). `NULL` → **"Desconocido"** (own
  bucket, not involuntary).
- **Grain**: Per cancelled account-month (SaaS only); `cancel_reason` deduped to the most recent
  cancellation per subscription.
- **Key rules / filters**:
  - **`no_card` is EXCLUDED** from the canonical predicate (validated vs Redshift: it barely churns as
    SaaS). A legacy convention (`not_paid + no_card`) survives on some surfaces; the difference is
    immaterial, but new definitions use `not_paid`-only.
  - Any new Chargebee `cancel_reason` defaults to "Otro"/Voluntario until reviewed.
  - It's a **routing signal**, not a revenue measure — it partitions the same lost MRR gross churn counts.
- **Source tables**: `dwh.revenue_waterfall_software`, `staging.stg_chargebee__subscriptions_snapshot`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

---
type: metrica
fuente: skill finance-metric-definitions
temas: [retencion]
actualizado: 2026-09-22
---

# Activation

- **Definition**: A merchant is "activated" once it crosses a bookings threshold for its segment.
- **Formula / thresholds**: **B2C 20+ · B2B2 40+ · B2B3 100+** bookings.
- **Grain**: Per company, cumulative bookings since first paying.
- **Key rules / filters**: Same thresholds repo-wide; splits churn into graduado (post-activation) vs
  onboarding (pre-activation). Configurable velocity-threshold buttons (25/50/…/500) are exploratory
  curve knobs, **not** a redefinition (default = 100).
- **Source tables**: `dwh.bookings`, `dwh.first_paying_date`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

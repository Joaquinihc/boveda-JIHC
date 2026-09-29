---
type: metrica
fuente: skill finance-metric-definitions
temas: [retencion]
actualizado: 2026-09-22
---

# Activation Velocity

- **Definition**: Time (weeks since first paying) for a cohort to reach a bookings threshold.
- **Formula**: Per (cohort × week × profile), % of cohort that has hit the threshold by that week.
- **Grain**: Weekly cohort curves.
- **Key rules / filters**: Segment coherence enforced. **Counts impersonating bookings** (no
  `creative_source` filter). Canonical source is the `dwh.*` table (a `reports.*` mirror also exists).
- **Source tables**: `dwh.activation_velocity_weekly_cohort_report`, `dwh.first_paying_date`.

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

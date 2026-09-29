---
type: metrica
fuente: skill finance-metric-definitions
temas: [otros]
actualizado: 2026-09-22
---

# Segment

- **Definition**: Merchant size/sophistication tier.
- **Values**: `{B2B3, B2B2, B2C}`.
- **Grain**: Per company.
- **Key rules / filters**: Column name varies by table — `dwh.companies.first_customer_segment_2`
  (master), `segment` in the pre-aggregated revenue tables, `customer_segment` in the waterfalls (see
  the layer conventions above). Activation thresholds differ by segment (B2C 20+ / B2B2 40+ / B2B3
  100+ bookings). Only B2B3 differs materially between venue and company grain; B2C/B2B2 are
  effectively mono-location.
- **Source tables**: `dwh.companies.first_customer_segment_2`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

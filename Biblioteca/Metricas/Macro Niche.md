---
type: metrica
fuente: skill finance-metric-definitions
temas: [otros]
actualizado: 2026-09-22
---

# Macro Niche

- **Definition**: The finer industry taxonomy the 4-bucket Vertical rolls up from.
- **Canonical set (6, standardized)**: `{Beauty, Fitness, Health Doctors, Health Non-doctors, Spa,
  Others}`.
- **Grain**: Per company.
- **Key rules / filters**: Same canonical source as Vertical. Roll-up: Beauty→Beauty ·
  Health Doctors→Health · Health Non-doctors→Health · Spa→Medspa · Fitness→Other · Others→Other. Do not
  re-invent slug maps or inline `CASE` families — back them by the attributes table.
- **Source tables**: `dwh.company_attributes` (`name='macro_niche'`);
  view `dwh.company_attribute_macro_niche`.

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

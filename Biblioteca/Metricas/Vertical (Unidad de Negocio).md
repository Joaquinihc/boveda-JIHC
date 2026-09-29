---
type: metrica
fuente: skill finance-metric-definitions
temas: [otros]
actualizado: 2026-09-22
---

# Vertical (Unidad de Negocio)

- **Definition**: The industry bucket a merchant belongs to. **Exactly four (4) verticals** — not a
  3-way collapse.
- **Canonical display values**: `{Beauty, Health, Medspa, Other}`. The underlying attribute may store
  the Spa bucket as `'MedSpa'`; **normalize to `'Medspa'` on display** (label decision only, no numbers
  change).
- **Grain**: Per company; stable attribute.
- **Key rules / filters**:
  - Single canonical source is the attributes table — **do not re-derive vertical inline** with
    `CASE`/`VALUES`.
  - Macro-niche → Vertical mapping (authoritative): Beauty ← Beauty · **Health ← Health Doctors +
    Health Non-doctors (both)** · Medspa ← Spa · Other ← Fitness + Others.
- **Source tables**: `dwh.company_attributes` (`name='vertical'`, columns `company_id`/`name`/`value`);
  convenience view `dwh.company_attribute_vertical`.
- **Reference query**:
```sql
-- [DWH · Redshift] canonical vertical join (do NOT re-derive with CASE)
SELECT c.company_id,
       CASE WHEN v.value = 'MedSpa' THEN 'Medspa' ELSE v.value END AS vertical
FROM dwh.companies c
JOIN dwh.company_attributes v
  ON v.company_id = c.company_id AND v.name = 'vertical';
```

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

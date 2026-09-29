---
type: metrica
fuente: skill finance-metric-definitions
temas: [otros]
actualizado: 2026-09-22
---

# Impersonating bookings (personified)

- **Definition**: Bookings created by an AgendaPro agent acting on behalf of a merchant.
- **Formula**: `creative_source = 'schedule - personified'` (join `dwh.locations` for `company_id`).
- **Grain**: Per booking.
- **Key rules / filters**: **Activation Velocity COUNTS them** (no `creative_source` filter). A
  "Personified" view toggle isolates them elsewhere. Transversal rule — do not silently drop personified
  bookings when measuring activation speed.
- **Source tables**: `dwh.bookings` (+ `dwh.locations`).

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

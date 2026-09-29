---
type: metrica
fuente: skill finance-metric-definitions
temas: [volumen]
actualizado: 2026-09-22
---

# Marketplace "new booking" vs "Chargeable (grado-cobro)"

- **Definition**: Two distinct counts of new demand the marketplace brings a merchant — differ ~3–4×; do
  not conflate.
- **Formula / rules**:
  - **New booking (to merchant)** = a marketplace booking that is the **first time that person
    (`client_ref`) ever booked at that `company_id`** (`created_at = MIN(created_at) OVER (client_ref,
    company_id)`).
  - **Chargeable (grado-cobro)** = stricter — a valid booking bringing a client with **no prior ficha**
    at the merchant, matched by **email ∪ phone** (not just `client_ref`), > 60s before the booking.
    `dwh_marketplace.mart_marketplace__chargeable_bookings.chargeable_bookings_grado_cobro`.
  - `% new = SUM(new_bookings) / NULLIF(SUM(bookings), 0)` (denominator = all bookings, new + returning).
- **Grain**: Monthly × merchant (`company_id`); some surfaces count active `location_id`.
- **Key rules / filters**: monetization / OKR → **chargeable**; "how much new demand" → **new_bookings**.
  Chargeable < new (the email∪phone prior-ficha test excludes people already on file).
- **Source tables**: `dwh_marketplace.marketplace_demand_unified`,
  `dwh_marketplace.mart_marketplace__chargeable_bookings`. (A per-merchant monthly rollup is materialized
  in the **BI-layer** off these.)

---

> [!info] Fuente canónica
> Esta ficha proviene de la skill `finance-metric-definitions` (construida desde `agendapro-dbt-redshift` y `agendapro-dashboards-evidence`). Ante cualquier duda, la skill y los repos son el árbitro. Solo metodología: nunca guardar cifras vivas aquí.

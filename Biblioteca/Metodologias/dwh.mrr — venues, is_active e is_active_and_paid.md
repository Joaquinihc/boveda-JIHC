---
type: metodologia
temas: [dwh, mrr, definiciones]
fuente: agendapro-dbt-redshift · origin/refactor/postgres-to-redshift (models/mrr.sql, models/venues.sql, models/mrr/customer_revenue_by_month.sql)
fecha: 2026-10-22
---

# dwh.mrr — venues, is_active e is_active_and_paid

Qué significa cada columna de `dwh.mrr` que se usa para contar merchants y filtrar MRR, y por qué no coinciden entre sí. Verificado el 22-oct-2026 contra la rama de Redshift.

## Las tres columnas

| Columna | Definición en dbt | Para qué sirve |
|---|---|---|
| `venues` | `plan_quantity` de la suscripción Chargebee del mes (estados `active` y `non_renewing`), forzado a **0 cuando el MRR del mes es 0**. Meses previos a Chargebee salen de `stg_agendapro__venues_historical`. | Contar sucursales (Total Merchants, New Merchants). |
| `is_active` | **MRR del mes > 0** a nivel company (`customer_revenue_by_month`: `jf.mrr > 0`). | Gate del **conteo** de merchants. |
| `is_active_and_paid` | `chargebee_status in ('active','non_renewing')` **y** `start_date_pago` no nulo con mes ≤ mes actual **y** no es cuenta test (`is_test_company` de `stg_chargebee__customers`, emails @agendapro / @yopmail). | Gate del **MRR** (`dwh.mrr_saas`, `dwh.mrr_ai` lo heredan). |
| `venues_active_and_paid` | `plan_quantity` cuando se cumple `is_active_and_paid`, aunque el MRR sea 0. | Contar sucursales pagando incluyendo cortesías y descuentos del 100 %. |

## Por qué `venues` ≠ `plan_quantity`
1. MRR del mes = 0 (cortesía, cupón del 100 %): `venues` se fuerza a 0 aunque la suscripción siga activa.
2. Suscripción fuera de `active` / `non_renewing` ese mes: no hay fila en `venues` y queda 0, aunque el MRR siga positivo por la convención del mes de cancelación.
3. Meses previos a Chargebee: vienen de la tabla histórica, no de `plan_quantity`.

Además, `dwh.merchants_segments.plan_quantity` (el que usa `country_new_merchants` del board book) es el valor **vigente hoy** (atributo `plan_quantity` de `company_values`), no el del mes del alta.

## Por qué hay cuentas que «pagan sin estar activas»
Los dos flags tienen lógica distinta y ninguno es subconjunto del otro:
- Suscripción `active` con MRR 0 (cupón del 100 %, cortesía) → `is_active_and_paid = true`, `is_active = false`. Es el caso de ~1.7k filas company-mes (comentario en `sources/dwh/merchants/country_merchants.sql`).
- Cancelada en el mes con MRR aún > 0 → `is_active = true`, `is_active_and_paid = false`.

## Regla práctica
- Stock de merchants: `SUM(venues) WHERE is_active` (board book, Total Merchants).
- MRR: `WHERE is_active_and_paid` sobre `dwh.mrr_saas` / `dwh.mrr_ai`.
- Si se quiere contar sucursales pagando con cortesías incluidas: `venues_active_and_paid`.

Relacionado: [[Total Merchants (EOM stock)]], [[New Merchants (nuevas altas)]], [[20261022-01 New Merchants board book vs new del waterfall software]].

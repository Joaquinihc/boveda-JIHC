---
type: analisis
proyecto:
estado: activo
fecha: 2026-10-22
temas: [definiciones, waterfall, ventas]
metricas: ["[[New Merchants (nuevas altas)]]"]
---

# New Merchants board book vs new del waterfall software

**Pregunta**: ¿calza el New Merchants que reporta el board book (extracción `country_new_merchants`: `dwh.merchants_segments` + `dwh.mrr.is_first_month_paying`, en sucursales) con `SUM(quantity_impact)` de `change_category = 'new'` en `dwh.revenue_waterfall_software`?

## Conclusión
> No calzan exactamente, pero están cerca. El waterfall queda entre 1 % y 5 % por debajo del board book en casi todos los meses de 2025 y la brecha se achica en 2026 (de −41 sucursales en ene-2025 a ±2 en jun, ago y sep-2026). Para el titular de altas la definición vigente es la del board book; el waterfall sirve para los movimientos del MRR, no para contar altas.

## Método
- Board book: parquet local `build/data/dwh/country_new_merchants/…/country_new_merchants.parquet` del repo `agendapro-dashboards-evidence` (extraído el 6-oct-2026), agregado por mes con DuckDB. Lógica: `SUM(ms.plan_quantity)` de `dwh.merchants_segments` por `date_trunc('month', start_date_pago_mx)`, sin cuentas test (`stg_chargebee__customers.is_test_company`), exigiendo `dwh.mrr.is_first_month_paying`.
- Waterfall: query directa al DWH el 22-oct-2026.

```sql
select date_trunc('month', month)::date as m,
       sum(quantity_impact) as nm_wf_software,
       count(distinct company_id) as companies_wf_software
from dwh.revenue_waterfall_software
where change_category = 'new'
  and month >= '2025-01-01' and month < '2026-10-01'
group by 1 order by 1;
```

- Control adicional: conteo directo en `dwh.mrr` (`SUM(venues)` con `is_first_month_paying`, sin test), para dimensionar el efecto del universo de `merchants_segments`.

## Evidencia
Sucursales por mes (corte 22-oct-2026; board book al 6-oct):

| Mes | Board book | Waterfall software | Diferencia |
|---|---:|---:|---:|
| 2025-01 | 1.095 | 1.054 | −41 |
| 2025-02 | 1.091 | 1.032 | −59 |
| 2025-03 | 1.083 | 1.059 | −24 |
| 2025-04 | 1.161 | 1.133 | −28 |
| 2025-05 | 1.390 | 1.352 | −38 |
| 2025-06 | 1.249 | 1.232 | −17 |
| 2025-07 | 1.213 | 1.182 | −31 |
| 2025-08 | 1.250 | 1.234 | −16 |
| 2025-09 | 1.299 | 1.271 | −28 |
| 2025-10 | 1.490 | 1.466 | −24 |
| 2025-11 | 1.448 | 1.424 | −24 |
| 2025-12 | 1.261 | 1.244 | −17 |
| 2026-01 | 1.563 | 1.547 | −16 |
| 2026-02 | 1.601 | 1.573 | −28 |
| 2026-03 | 1.929 | 1.912 | −17 |
| 2026-04 | 1.518 | 1.493 | −25 |
| 2026-05 | 1.320 | 1.327 | +7 |
| 2026-06 | 1.121 | 1.119 | −2 |
| 2026-07 | 1.078 | 1.058 | −20 |
| 2026-08 | 1.218 | 1.216 | −2 |
| 2026-09 | 1.228 | 1.230 | +2 |

Causas de la brecha:
1. **Estado en el mes del primer pago.** El `new` del waterfall exige suscripción `active` o `non_renewing` ese mes; las altas que figuran `in_trial`, `paused` o `cancelled` no entran como `new` y aparecen después como `reactivation` (pendiente anotado con Pablo Marambio, Q-1).
2. **Ancla de fecha.** Board book: `start_date_pago_mx` (primera transacción exitosa). Waterfall: primer mes con MRR en el snapshot de Chargebee. En los bordes de mes se corren casos.
3. **Cantidad.** Board book: `plan_quantity` vigente hoy en `merchants_segments` (infla retroactivamente a quien después sumó sucursales). Waterfall: `plan_quantity` del mes del alta.

Control con `dwh.mrr` directo (venues en el mes con `is_first_month_paying`, sin test): da bastante más alto, por ejemplo 2.169 en mar-2026 contra 1.929 del board book y 1.912 del waterfall. La diferencia es que `merchants_segments` restringe el universo a companies con suscripción Chargebee (`cb_subscription_id`) y flags «cliente activo» y «cliente pagando» no nulos.

## Caveats
- El parquet del board book es del 6-oct; sep-2026 ya estaba cerrado, pero una reextracción puede mover unidades por el `plan_quantity` vigente.
- La query sobre `merchants_segments` directa al DWH dio timeout tres veces; por eso se usó el parquet.
- No se midió por país ni segmento.

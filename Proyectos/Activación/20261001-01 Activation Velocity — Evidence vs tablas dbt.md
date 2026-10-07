---
type: analisis
proyecto: "[[_Activación]]"
estado: activo
fecha: 2026-10-01
temas: [activacion, definiciones, evidence, dwh, okrs, board]
metricas: ["[[Activation Velocity]]"]
---

# Activation Velocity: Evidence vs tablas dbt

**Pregunta**: ¿Qué diferencias de definición aparecen si Evidence deja de calcular la activación sobre `company_sales_days` y pasa a leer las tablas dbt de velocity (`activation_velocity_*_report` / `int_company_booking_cohorts`)? ¿Cuánto pesa cada una?

## Conclusión
> - **Pasar tal cual sube la activación B2B3 de Semana 9 entre 1,1 y 3,2 pp** según la cohorte (ene–jul 2026). El Mes 3 del board book del Q3 (cohortes abr–jun) pasaría de 54,3% a 55,3%.
> - **Lo que más pesa son los estados**: dbt cuenta cancelados y no-show (+1,6 a +3,3 pp). Las metas del Budget se calibraron con solo activos.
> - **El universo dbt resta** (−0,3 a −1,8 pp): incluye 89 empresas que nunca pagaron, 3 cuentas test y 2 que no están en `merchants_segments`. Además, 51 empresas caen en otro mes por el ancla UTC.
> - **Para migrar sin mover las cifras** habría que: decidir activos o todos; llevar a dbt el universo de New Merchants (o filtrarlo en la extracción); igualar el borde de semana y la definición de clases; y elegir la ponderación.

## Método
- Puente por cohorte en Redshift (1-oct-2026), cambiando una pieza por vez: Evidence → todos los estados → resto de la definición de reservas → universo dbt → ponderación dbt. Los extremos cuadran con la página y con las tablas dbt.
- B2B3, umbral 100, ponderado por sedes. Las dos fuentes todavía contaban las importadas (el PR 415 no estaba mergeado).

## Evidencia
**Diferencias de definición**

| Dimensión | Evidence | Tablas dbt de velocity | ¿Pesa? |
|---|---|---|---|
| Estados de booking | Solo activos (`company_sales_days`) | Todos (`fct_bookings_created_daily`) | **Sí** |
| Universo | New Merchants (primer mes pagado, sin test) | Todo `first_paying_date` | Sí, en contra |
| Día 0 | `start_date_pago_mx` (México) | `first_paying_date.start_date_pago` (UTC) | Poco |
| Borde de semana / mes | Semana 9 = días 0–62; Mes 3 = 0–89 | 0–63 y 0–90 | Poco |
| Clases | Por día de la clase, sin canceladas ni no-show | Por creación, todos los estados, solo demanda | Poco |
| Filtros | Pago y outliers de GMV | Fake bookings; corte en el fin de la suscripción | Poco |
| Ponderación | `merchants_segments.plan_quantity` | `initial_plan_quantity` (Chargebee). `plan_quantity` en dbt es la última | Poco, en ambos sentidos |
| Segmento | `company_attribute_first_segment2` | `companies.first_customer_segment_2` | Igual en todas |
| País | `merchants_segments.country` (Perú en "Otros") | Moneda de la suscripción | 9 empresas (Otros → Argentina) + etiquetas |

**Puente, Semana 9** (% de sedes):

| Cohorte | Evidence | + todos los estados | + resto de reservas | + universo dbt | + ponderación | dbt |
|---|---|---|---|---|---|---|
| ene-26 | 53,6 | +3,3 | +0,4 | −1,4 | −0,1 | 55,7 |
| feb-26 | 51,2 | +2,6 | +1,0 | −0,6 | −1,1 | 53,1 |
| mar-26 | 47,8 | +2,4 | +0,5 | −0,3 | +0,5 | 51,0 |
| abr-26 | 47,9 | +3,3 | +0,8 | −1,8 | +0,1 | 50,3 |
| may-26 | 41,3 | +2,4 | +1,0 | −1,5 | −0,1 | 43,1 |
| jun-26 | 47,6 | +2,1 | +1,2 | −1,0 | −0,1 | 49,7 |
| jul-26 | 54,8 | +1,6 | +0,8 | −1,7 | +0,3 | 55,9 |

Mes 3 se comporta parecido: entre −0,1 y +2,9 pp según la cohorte.

**Diferencias de estructura**
- Evidence oculta las últimas 4 semanas abiertas de cada cohorte; dbt no tiene ese concepto, y el reporte mensual ya muestra Mes 4 de julio con ~92 días transcurridos.
- Topes: Evidence llega a 30 semanas y 12 meses; dbt a 53 y 36.
- La tabla semanal dbt es empresa × semana (2,1 millones de filas); Evidence tendría que agregarla en la extracción.
- `initial_mrr_usd` (dbt) sale de otra fuente que `mrr_usd` (Evidence); no se midió. Macro nicho por defecto: "Other" en dbt, "Others" en Evidence.
- Solo `monthly_weekly` y `monthly_monthly` pueden migrar. `by_onboarder` necesita la asignación de onboarders; Retention, R1 y PMF necesitan días exactos al umbral y fechas de churn.

## Caveats
- Hoy `first_paying_date` tiene una fila por empresa, pero agrupa también por `customer_id` y `plan_id`, así que podría duplicar empresas.
- Al migrar, Evidence también heredaría la exclusión de importadas del PR 415 (ver [[20260930-01 Reservas importadas — impacto en activación y waterfalls]]).
- Seguimiento en [[Arreglar definición de activación]].
- Definiciones y fuentes vigentes: [[Activación — fuentes y definiciones vigentes]].

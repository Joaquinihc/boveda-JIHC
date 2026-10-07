---
type: metodologia
temas: [activacion, definiciones, evidence, dwh]
fuente: trabajo propio (repos agendapro-dbt rama refactor/postgres-to-redshift y agendapro-dashboards-evidence)
fecha: 2026-10-07
---

# Activación: fuentes y definiciones vigentes

Dónde se calcula la activación hoy, qué cuenta cada versión y qué hay que saber antes de tocarla. Solo metodología: las cifras viven en los análisis del proyecto [[_Activación]]. La definición oficial es la de la skill `finance-metric-definitions` ([[Activation]], [[Activation Velocity]]); al 7-oct-2026 no coincide con el código (ver [[Deriva — Activation]]).

## Qué lee cada consumidor
| Consumidor | Fuente de reservas |
|---|---|
| Evidence: Activation Velocity, OKR y board book (`monthly_weekly`, `monthly_monthly`, `by_onboarder`) | `dwh.company_sales_days` + clases de `stg_platform_classes__*` |
| Evidence: Activation → Retention, Lead → Activación, PMF Fitness, R1 (PR 772) | Lo mismo |
| Evidence: Onboarder Capacity; filtro Plan de la página | `dwh.activation_velocity_weekly_report` (dbt) |
| Evidence: graduation | `dwh.int_company_activation_status` (dbt) |
| dbt: waterfall SaaS / Software (graduado vs onboarding) | `int_company_activation_status` → `int_subscription_mrr_breakdown_changes` |
| dbt: waterfall Payments | Activación de **pagos** (30 transacciones ≥ USD 5), no reservas |

## Las tres definiciones de "activado"
| | Evidence | Reportes dbt de velocity | `int_company_activation_status` |
|---|---|---|---|
| Bookings | `company_sales_days.booking_count` (solo activos) | `fct_bookings_created_daily` (todos los estados) | Igual que velocity |
| Importadas | Las cuenta | `bookings_count_demand` las excluye (PR 415) | Igual (PR 415) |
| Clases | Día de la clase, sin canceladas ni no-show, todos los orígenes | `enrollments_count_demand`: creación, todos los estados, solo orígenes de demanda | Igual que velocity |
| Universo | New Merchants: primer mes pagado en `dwh.mrr`, sin cuentas test | Todo `first_paying_date` | Todas las empresas |
| Día 0 | `merchants_segments.start_date_pago_mx` | `first_paying_date.start_date_pago` (UTC, primera transacción exitosa) | Sin día 0: toda la vida de la empresa, trial incluido |
| Ponderación | Sedes (`merchants_segments.plan_quantity`) | `initial_plan_quantity` (primera suscripción en Chargebee) | Por empresa |

## Códigos de estado
- **Bookings** (`staging.stg_agendapro__statuses`): 1 reservado, 2 confirmado, 3 asiste, 5 cancelado, 6 no asiste, 7 en espera, 8 pendiente. `booking_count_active` = estados 1, 2, 3, 7 y 9.
- **Enrollments** (plt-classes, `InstanceEnrollment.status`): 0 enrolled, 1 attended, 2 no_show, 3 cancelled. La asistencia la carga el comercio y casi nadie la llena.
- **Orígenes de enrollments que son demanda**: `marketplace_app`, `marketplace_web`, `recurring` y NULL (caja). No son demanda: `membership_migration` y `migracion_ikigai` (backfills), `clase_prueba` y `admin`.

## Reservas importadas desde otro software
- No hay un flag. Se identifican por el comentario `Creada vía archivo .csv` (workflow n8n de BizOps "booking migration via csv V2") más la carga de la company 514942, que entró por otra vía (`creative_source` NULL).
- El comentario lo puede editar el comercio. Una edición fuera de la ventana de reproceso solo se corrige con `--full-refresh` de `fct_bookings_created_daily`.
- Lo de fondo es pedirle a Producto un origen propio (`csv_import`).

## Cortes y bordes
- **Semana N**: Evidence = `FLOOR(días/7)+1` (días 0 a 7N−1); dbt = `CEIL(días/7)`, con el día 0 en la semana 1 (días 0 a 7N). Lo mismo con bloques de 30 días para "Mes N".
- **Topes**: Evidence llega a 30 semanas y 12 meses; dbt a 53 y 36.
- **Semanas abiertas**: Evidence oculta por defecto las últimas 4 de cada cohorte. El reporte mensual dbt habilita "Mes N" por calendario, así que muestra meses todavía incompletos.

## Operación (dbt)
- Los cuatro modelos de velocity corren a diario en `run-dbt-marts-redshift` desde el PR 412. Antes solo se actualizaban a mano.
- Los incrementales con acumulados (`fct_bookings_created_daily`, `fct_enrollments_created_daily`) necesitan `--full-refresh` cuando se les agrega una columna acumulada: el incremental lee el acumulado anterior desde la propia tabla.
- El esquema `reports.*` tiene copias viejas de las tablas de velocity (paradas en 202603): no usarlas.

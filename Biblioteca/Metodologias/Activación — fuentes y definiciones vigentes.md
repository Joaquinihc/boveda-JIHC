---
type: metodologia
temas: [activacion, definiciones, evidence, dwh]
fuente: trabajo propio (repos agendapro-dbt rama refactor/postgres-to-redshift, agendapro-dashboards-evidence y bizops-cx-apps)
fecha: 2026-10-08
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
| CX (`bizops-cx-apps`): panel ejecutivo (`cx-head-app`: Activación & Churn, Semanal CEO, pulso) y reporte del comité (`_shared/ceo_report/cohorts.py`) | `dwh.bookings` directo (efectivas) + clases + cobros de `staging.stg_agendapro__payments` |
| CX: Onboarding TL, CS Comisiones y apps de onboarders (`_shared/onboarding_core.py`) | Igual que el panel de CX, con umbral por nicho del motor de comisiones |

## Las cuatro definiciones de "activado"
| | Evidence | Reportes dbt de velocity | `int_company_activation_status` | CX directorio @M4 |
|---|---|---|---|---|
| Bookings | `company_sales_days.booking_count` (solo activos) | `fct_bookings_created_daily` (todos los estados) | Igual que velocity | `dwh.bookings.booking_count_active` sin fake ("efectivas"), sin los filtros de pago ni de outliers de `company_sales_days` |
| Cobros | No | No | No | Máximo entre reservas efectivas y cobros pagados |
| Importadas | Las cuenta | `bookings_count_demand` las excluye (PR 415) | Igual (PR 415) | Las cuenta |
| Clases | Día de la clase, sin canceladas ni no-show, todos los orígenes | `enrollments_count_demand`: creación, todos los estados, solo orígenes de demanda | Igual que velocity | Igual que Evidence (es la de CX) |
| Universo | New Merchants: primer mes pagado en `dwh.mrr`, sin cuentas test | Todo `first_paying_date` | Todas las empresas | `merchants_segments.segment2 = 'B2B3'` con primer mes pagado; no filtra cuentas test |
| Día 0 | `merchants_segments.start_date_pago_mx` | `first_paying_date.start_date_pago` (UTC, primera transacción exitosa) | Sin día 0: toda la vida de la empresa, trial incluido | `merchants_segments.start_date_pago` |
| Ventana | Semana N (7 días) / Mes N (30 días) | Igual que Evidence | Sin ventana | Meses calendario 0 a 4 desde el mes del primer pago |
| Ponderación | Sedes (`merchants_segments.plan_quantity`) | `initial_plan_quantity` (primera suscripción en Chargebee) | Por empresa | Cuentas |
| Umbral | 100 (con selector) | Cualquiera (da el acumulado) | 20 / 40 / 100 por segmento | 100 fijo (universo B2B3) |

## Otras definiciones de CX (`bizops-cx-apps`)
- **Onboarding y comisiones** (`onboarding_core.is_activated` y `umbral_for`): umbral por nicho y tamaño de la tabla del motor de comisiones (cuenta chica de 1–2 profesionales activos con 2+ ciclos: 70 reservas) **y** 2 ciclos pagados (o 5 semanas en planes de más de 2 meses). La actividad es el máximo entre reservas y cobros.
- **"Activación de calidad"** (`activated_paid`): 100 reservas + 3 ciclos pagados. Ya no está en las pantallas principales.
- **Graduación** (`graduacion_cx.py`): 100 reservas + 4 ciclos pagados + un mes con reservas en sus 4 semanas. Equivale a la vía objetiva de la graduación v2 en dbt. Sin cobros.
- El código de CX dice que la activación "la pone Finanzas en `int_company_activation_status`", pero su activación del directorio no calcula eso. Ver [[20261008-01 Activación en CX (bizops-cx-apps) vs Evidence y dbt]].

## Códigos de estado
- **Bookings** (`staging.stg_agendapro__statuses`): 1 reservado, 2 confirmado, 3 asiste, 5 cancelado, 6 no asiste, 7 en espera, 8 pendiente. `booking_count_active` = estados 1, 2, 3, 7 y 9. **El 9 no existe y Pendiente es el 8**, así que deja fuera las reservas pendientes (~1% según CX). Afecta a `company_sales_days` (Evidence) y a CX, que usan `booking_count_active`. No afecta al fact dbt (`fct_bookings_created_daily`), que cuenta todos los estados. El filtro está en el modelo dbt `bookings_incremental`.
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

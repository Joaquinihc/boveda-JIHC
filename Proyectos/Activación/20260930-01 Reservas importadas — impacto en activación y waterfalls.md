---
type: analisis
proyecto: "[[_Activación]]"
estado: activo
fecha: 2026-09-30
temas: [activacion, graduacion, churn, waterfall, definiciones, dwh]
metricas: ["[[Activation Velocity]]", "[[Activation]]"]
---

# Reservas importadas: impacto en activación y waterfalls

**Pregunta**: ¿Cuánto mueve la activación, mercado por mercado, dejar de contar las reservas que un comercio importa desde otro software? ¿Y qué cambia en los revenue waterfalls si `int_company_activation_status` pasa a `bookings_count_demand`?

## Conclusión
> - **Mueve poco y solo B2B3.** Las 28 empresas con importadas desde dic-2025 son todas B2B3. El total de cada cohorte baja 0,3 pp como máximo. **Argentina** concentra el efecto (−0,6 a −1,1 pp por cohorte): 5 de las 7 empresas que no llegan a 100 reservas sin las importadas son argentinas.
> - **El efecto grande es en la curva semanal y dura una semana**: la 514942 (Colombia, 36 de 124 sedes de la cohorte abr-26) cruza 100 reservas en la semana 11 en vez de la 10. En esa semana, Colombia abril cae 29 pp y el total de abril 6,0 pp.
> - **Board book Q3** (Mes 3, cohortes abr–jun): 54,3% → 54,1%.
> - **Waterfalls: cambia un solo evento.** La 424648 (México, 1 sede) pasa de `churn_graduado` a `churn_onboarding` en ene-2026, en v1 y v2: USD −99,31 de MRR en `revenue_waterfall_saas` (USD −85,12 en Software). El churn total no cambia. Payments no cambia, porque usa la activación de pagos.

## Método
- **Regla de importadas** (PR 414): `company_comment ILIKE 'Creada vía archivo .csv%'` o `creative_source IS NULL` en la company 514942. Son 448.951 bookings en 41 empresas (validado tras el full refresh del 30-sep).
- **Activación como la mide Evidence**: universo New Merchants, `company_sales_days` (solo bookings activos) + clases, B2B3, umbral 100, por sedes. A cada empresa se le restan sus importadas activas dentro de la ventana. Para la 514942 se usaron directo sus reservas propias.
- **Waterfalls**: recalculé la activación y las dos fechas de graduación de las 41 empresas con `bookings_count_demand` y crucé con los eventos de churn de `int_subscription_mrr_breakdown_changes`.
- Corte: Redshift, 30-sep-2026.

## Evidencia
**Semana 9** (la página y la comparación con CX):

| Cohorte | País | Antes | Después | Δ pp | Total de la cohorte |
|---|---|---|---|---|---|
| mar-26 | Argentina | 52,3 | 51,7 | −0,6 | 47,7 → 47,6 |
| jul-26 | Argentina | 55,6 | 54,4 | −1,1 | 54,8 → 54,5 |
| ago-26 (abierta) | Chile | 42,1 | 41,1 | −1,1 | 42,4 → 41,9 |
| ago-26 (abierta) | México | 50,0 | 49,2 | −0,8 | (misma cohorte) |

**Mes 3** (el OKR y el board book):

| Cohorte | País | Antes | Después | Δ pp | Total de la cohorte |
|---|---|---|---|---|---|
| mar-26 | Argentina | 57,6 | 57,0 | −0,6 | 54,5 → 54,4 |
| abr-26 | Argentina | 54,9 | 54,2 | −0,7 | 58,8 → 58,6 |
| jun-26 | Argentina | 47,8 | 46,9 | −0,9 | 53,1 → 52,9 |
| jul-26 (abierta) | Argentina | 58,9 | 57,8 | −1,1 | 58,4 → 58,1 |

- No llegan a 100 sin importadas: 505265, 494697, 526693, 530525 y 542958 (Argentina), 557725 (Chile) y 563200 (México).
- Activan más tarde: 499653 (semana 1 → 3), 514942 (10 → 11), 526720 (5 → 6) y 530824 (7 → 8).

**Activación dbt (`int_company_activation_status`, toda la vida de la empresa)**
- 10 empresas pierden `is_saas_activated`; ninguna había graduado.
- Solo la 424648 pierde la graduación. La 543659 y la 514942 corren su fecha 2 días dentro del mismo mes, así que no cambian en el waterfall, que compara por mes.
- Las categorías de AI, Julia, Sofia, agente AI y POS no usan la graduación.

## Caveats
- **`company_sales_days` ya filtraba parte de lo importado por la 514942**, probablemente por la regla de outliers de GMV: tiene 140.890 reservas cuando las importadas en la ventana son 217.451.
- **No calculé las cohortes 2025.** Hay 13 empresas más con importaciones anteriores a dic-2025 que las afectan.
- **Evidence sigue contando las importadas** aunque se aplique en dbt, porque lee `company_sales_days`.
- **Enero 2026 ya estaba reportado**: lo de la 424648 es una pequeña re-expresión.
- PRs: [[PR 414 — add import-free booking counters to fct_bookings_created]] y [[PR 415 — exclude imported bookings from activation and velocity cohorts]] (abierto al 7-oct). Seguimiento en [[Arreglar definición de activación]].
- Definiciones y fuentes vigentes: [[Activación — fuentes y definiciones vigentes]].

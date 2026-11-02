---
type: deriva
metrica: "[[Activation Velocity]]"
estado: abierta
temas: [activacion, definiciones, dwh]
proyecto: "[[_Activación]]"
fuente: https://github.com/agendapro/agendapro-dbt/blob/refactor/postgres-to-redshift/models/reports/activation_velocity/cte/int_company_booking_cohorts.sql
creado: 2026-11-02
revisar: false
---

# Deriva — Activation Velocity

> Detectada por el mantenimiento mensual (2-nov-2026). La ficha [[Activation Velocity]] es idéntica a la card de la skill `finance-metric-definitions`; la diferencia es contra el modelo dbt (`agendapro/agendapro-dbt`, rama `refactor/postgres-to-redshift`). No se tocó la ficha. Si corresponde, actualizar primero la skill y luego la ficha.

## Qué dice la ficha (= skill)
- Tiempo (semanas desde el primer pago) en que una cohorte llega a un umbral de **bookings**.
- Cuenta los bookings personificados (sin filtro de `creative_source`).
- Fuentes: `dwh.activation_velocity_weekly_cohort_report`, `dwh.first_paying_date`.
- No menciona enrollments ni bookings importados.

## Qué dice el modelo (`int_company_booking_cohorts`, alimenta `activation_velocity_*_report`)
- «Bookings» = **bookings + enrollments de fitness** (decisión del 2-sep-2026), con las mismas fuentes que `int_company_activation_status`: `fct_bookings_created_daily` (sin fake bookings) y `fct_enrollments_created_daily` (`enrollments_count_demand`).
- **Sin bookings importados** desde el PR 415 (mergeado el 19-oct-2026): lee `bookings_count_demand`, que excluye la marca «Creada vía archivo .csv» y la carga de la company 514942.
- Los personificados se siguen contando (el filtro de importados no usa `creative_source`): esa regla de la ficha sigue vigente.
- El reloj sigue siendo `start_date_pago`; la actividad previa al primer pago se descarta.

## Relacionado
- [[Deriva — Activation]] (misma fuente y misma regla de importados).
- `Inbox/PR 414 — …`, `Inbox/PR 415 — …` y [[20261001-01 Activation Velocity — Evidence vs tablas dbt]].

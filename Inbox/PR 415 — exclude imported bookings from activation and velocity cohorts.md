---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 415
url: https://github.com/agendapro/agendapro-dbt/pull/415
autor: Joaquinihc
estado_pr: abierto
merge:
temas: [activacion, graduacion, churn, waterfall, definiciones]
creado: 2026-10-02
revisar: true
---

# PR 415 — activación y cohortes de velocity sin reservas importadas

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `refactor/postgres-to-redshift`. **Abierto** (creado 30-sep, sin merge al 2-oct). Usa el contador del PR 414 (ver [[PR 414 — add import-free booking counters to fct_bookings_created]]).

**Proyecto**: sin link (el PR toca graduación pero no nombra el proyecto; candidato: Churn y Graduación). `revisar: true`.

## Qué cambió
- `int_company_booking_cohorts` e `int_company_activation_status` leen `bookings_count_demand` en vez de `bookings_count`: dejan de contar el historial importado (marca CSV y carga 514942).
- Heredan el cambio: activación SaaS y las dos rutas de graduación (journey y objetiva), porque parten de `saas_activation_date` y la misma serie diaria.
- Impacto medido por el PR (Redshift, 30-sep): 41 compañías con reservas importadas. Velocity: 29 de 39 cambian su acumulado a Week 9 (ninguno sube). Activación: 10 pierden `is_saas_activated` (ninguna graduada) y 12 activan más tarde. Graduación: 424648 la pierde.
- Waterfalls: Payments, AI/Julia/Sofia/agente_ai/POS sin cambios. SaaS/Software: **un evento** — 424648 (MX), **2026-01**, `churn_graduado` → `churn_onboarding` en `change_category` y `change_category_v2` (USD −99,31 en `revenue_waterfall_saas`). El churn total no cambia, solo el split.

## Por qué
- Mismo principio que `enrollments_count_demand`: activación y velocity miden uso de AgendaPro.
- ⚠️ Declarado en el PR: enero 2026 es un mes cerrado y reportado, así que es una pequeña re-expresión del split graduado/onboarding.

## Link
https://github.com/agendapro/agendapro-dbt/pull/415

## Biblioteca que podría quedar desactualizada
- [[Activation]]: umbral de bookings ahora sin importadas; fuente declarada `dwh.bookings`.
- [[Activation Velocity]]: cohortes sin reservas importadas.
- [[Metodología Churn B2B3]]: usa `int_company_activation_status`; cambia el split graduado/onboarding (un evento en ene-2026).

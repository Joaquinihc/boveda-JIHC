---
type: deriva
metrica: "[[Activation]]"
estado: abierta
temas: [activacion, definiciones, dwh]
fuente: https://github.com/agendapro/agendapro-dbt/blob/refactor/postgres-to-redshift/models/intermediate/revenue/int_company_activation_status.sql
creado: 2026-10-01
revisar: false
---

# Deriva — Activation

> Detectada por el mantenimiento mensual (1-oct-2026). La ficha [[Activation]] es idéntica a la card de la skill `finance-metric-definitions`; la diferencia es contra el modelo dbt (`agendapro/agendapro-dbt`, rama `refactor/postgres-to-redshift`). No se tocó la ficha. Si corresponde, actualizar primero la skill y luego la ficha.

## Qué dice la ficha (= skill)
- Grano: por compañía, bookings acumulados **desde el primer pago**.
- Fuentes: `dwh.bookings`, `dwh.first_paying_date`.
- La activación "separa el churn en graduado (post) vs onboarding (pre)".

## Qué dice el modelo (`int_company_activation_status`)
- Umbrales iguales (B2C 20 · B2B2 40 · B2B3+ 100, segmento de `first_customer_segment_2`).
- Una "reserva" = booking **o enrollment de fitness** (`fct_bookings_created_daily` sin fake bookings + `fct_enrollments_created_daily`, solo `enrollments_count_demand`).
- Se cuenta **toda la vida de la compañía, trial incluido, sin corte por primer pago**.
- La graduación (lo que separa churn graduado/onboarding) ya no es solo activación: `change_category` usa activación + marca de journey (definición inicial) y `change_category_v2` journey O vía objetiva (ver [[Deriva — Churn - Gross-Net × MRR-Logo]]).

## Relacionado
- `Inbox/PR 384 — publish two graduation classifications side by side (initial + CX-aligned v2)` y [[2026-09-22 Reunión — Revenue Waterfall, CAC y bookings]] (activación suma bookings y enrollments por igual).

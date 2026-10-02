---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 412
url: https://github.com/agendapro/agendapro-dbt/pull/412
autor: Joaquinihc
merge: 2026-09-29
temas: [activacion, dwh, evidence]
creado: 2026-10-02
revisar: false
---

# PR 412 — Activation Velocity entra a la corrida diaria (`redshift_marts`)

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `refactor/postgres-to-redshift`.

**Proyecto**: sin link (el PR no nombra proyecto).

## Qué cambió
- Tag `redshift_marts` inline en `int_company_booking_cohorts` y los tres `activation_velocity_*_report` (monthly, weekly, weekly_cohort), para que el flow diario `run-dbt-marts-redshift` los seleccione.
- La corrida diaria pasa de 250 a 254 modelos (sus upstream ya corrían ahí). `onboarding_company_activity` queda fuera a propósito (nunca se creó en Redshift).

## Por qué
- Los cuatro modelos no tenían tag: su último build en Redshift fue manual el **2026-09-07**, así que los tableros de Activation Velocity en Evidence (`dwh.activation_velocity_*`) estuvieron congelados desde esa fecha.
- Verificación post-merge declarada: `pg_class_info.relcreationtime` de las cuatro tablas debe coincidir con la corrida diaria.

## Link
https://github.com/agendapro/agendapro-dbt/pull/412

## Biblioteca que podría quedar desactualizada
- [[Activation Velocity]]: si documenta frecuencia de actualización o fuente, ahora se reconstruye a diario; datos entre 7-sep y 29-sep estaban congelados.

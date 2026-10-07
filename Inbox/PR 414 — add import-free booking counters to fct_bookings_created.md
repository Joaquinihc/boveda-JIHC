---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 414
url: https://github.com/agendapro/agendapro-dbt/pull/414
autor: Joaquinihc
merge: 2026-09-30
temas: [activacion, dwh, definiciones]
proyecto: "[[_Activación]]"
creado: 2026-10-02
revisar: false
---

# PR 414 — contadores de reservas sin importaciones en `fct_bookings_created`

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `refactor/postgres-to-redshift`. Lo consume el PR 415 (ver [[PR 415 — exclude imported bookings from activation and velocity cohorts]]).

**Proyecto**: [[_Activación]] (asignado por Joaquín el 2026-10-07).

## Qué cambió
- Columnas nuevas `bookings_count_demand` y `cum_bookings_count_demand` en `fct_bookings_created_daily` y `fct_bookings_created_monthly`: excluyen reservas importadas desde otro software al hacer onboarding. Espejo de `enrollments_count_demand` (que ya excluye migraciones de clases).
- `bookings_count` / `cum_bookings_count` no cambian (siguen contando todo); cada consumidor elige.
- Regla de importación (no hay flag en la fuente): `company_comment ilike 'Creada vía archivo .csv%'` (workflow n8n de BizOps "booking migration via csv V2", confirmado con Sebastián Hevia) + compañía 514942 (231k reservas con `creative_source` NULL por otra vía).
- Requiere `--full-refresh` de ambos modelos tras el merge; hasta entonces el incremental falla a propósito.

## Por qué
- Que activación y velocity midan uso de AgendaPro y no historial migrado. Los consumidores se cambian en el PR siguiente (415).
- Validación del PR: 448.960 reservas excluidas en 41 compañías (cuadra con `stg_agendapro__bookings`, gap 0). El full refresh también repara días parciales del incremental en prod (02/03/08/10-sep; el 10-sep faltaban 35k).

## Link
https://github.com/agendapro/agendapro-dbt/pull/414

## Biblioteca que podría quedar desactualizada
- [[Activation]] y [[Activation Velocity]]: hablan de "bookings" sin distinguir importadas; ahora existe el contador `_demand` que las excluye.

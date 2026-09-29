---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 393
url: https://github.com/agendapro/agendapro-dbt/pull/393
autor: Joaquinihc
merge: 2026-09-21
temas: [headcount, dwh]
creado: 2026-09-23
revisar: false
---

# PR 393 — fix `headcount`: una fila por persona y mes entre pestañas de país

> Borrador cosechado por la tarea PRs (vie). Rama `refactor/postgres-to-redshift`.

## Qué cambió
- `finance.headcount` arma una llave interna de persona (forma "Apellido, Nombre" reordenada, sin tildes, minúsculas; no se expone) y deja una fila por `(date, llave)`.
- Prioridad ante colisión: clasificación resuelta (`sheet`/`seed`) sobre `cost_center` sin mapear o NULL → mismo país que el mes anterior → Chile > México > Argentina > Colombia > USA → `Inactive` sobre `Active` dentro del mismo país → desempate determinístico.
- Se descartan filas con `date` no parseable (una hoja rota materializó una fila `#REF!` en 2026-09). Nuevo test singular `assert_headcount_one_row_per_person_month` y `not_null` en `date`.

## Por qué
- La pestaña de México escribe nombres como "Apellido, Nombre" y el resto "Nombre Apellido", así que los duplicados entre países no se veían con `group by name`; además había filas idénticas repetidas (Argentina 2023, USA 2025).
- Impacto reportado: 42 persona-mes eliminadas en 14 meses; como `headcount_monthly` / `headcount_country_monthly` de Evidence hacen `count(*)`, el headcount activo baja en esa cantidad en esos meses.
- Pendiente fuera del PR: `not_null_headcount_classification_source` sigue fallando por un caso de México 2026-09 con `cost_center` NULL en la hoja (requiere código de People).

## Link
https://github.com/agendapro/agendapro-dbt/pull/393

## Biblioteca que podría quedar desactualizada
- [[Guía clasificación de personas (headcount)]]: describe `finance.headcount` como fuente; no menciona el grano una-fila-por-persona-mes ni las reglas de dedup / prioridad de país.

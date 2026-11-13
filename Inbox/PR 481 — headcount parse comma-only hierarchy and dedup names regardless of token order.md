---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 481
url: https://github.com/agendapro/agendapro-dbt/pull/481
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-09
temas: [headcount, dwh, cierre-mensual]
creado: 2026-11-13
revisar: false
---

# PR 481 — `finance.headcount`: jerarquía con coma sin espacio y deduplicación de nombres en cualquier orden

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `fix/headcount-separators-person-key` → `refactor/postgres-to-redshift`. Creado y mergeado el 9-nov (sin review humana registrada; revisión de CodeRabbit).

**Proyecto**: sin link en frontmatter (el PR no nombra proyecto). Ya citado como PR #481 en [[_P&L y Centros de Costo]], [[20261109-01 Auditoría del informe de cierre sobre finance.headcount (octubre 2026)]], [[Guía clasificación de personas (headcount)]] y [[finance.headcount — employer, clave de persona y carga desde la planilla]].

## Qué cambió
- **Jerarquía de oct-26 rota en producción**: la planilla de People empezó a escribir `division_area_subarea` como `Div,Area,Subarea` (coma sin espacio) en Chile, Argentina y USA; el modelo separaba por `, `, así que 162 de 204 filas de octubre quedaron sin `area`/`subarea` y con `business` fuera de lista (33 valores). Ahora toda coma se normaliza a `, ` antes de separar.
- **Persona duplicada entre países** (Head of BizOps en Chile y Colombia, con el nombre en distinto orden): la clave de persona pasa a ser los tokens del nombre ordenados alfabéticamente; las reglas de prioridad mantienen la fila de Chile.
- **Tests que fallan fuerte**: `business` accepted_values pasa de warn a `error`; `area` not_null (`error`) desde 2026-07.
- Validación en Redshift: oct-26 de 204 a 203 filas, 0 áreas vacías, 0 `business` fuera de lista; meses anteriores idénticos (conteos y checksums por año).

## Por qué
- Corregir el headcount de octubre usado en el cierre.

## Link
https://github.com/agendapro/agendapro-dbt/pull/481

## Biblioteca que podría quedar desactualizada
- Ninguna identificada: [[Guía clasificación de personas (headcount)]] y [[finance.headcount — employer, clave de persona y carga desde la planilla]] ya lo reflejan.

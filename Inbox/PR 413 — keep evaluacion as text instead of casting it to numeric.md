---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 413
url: https://github.com/agendapro/agendapro-dbt/pull/413
autor: Joaquinihc
merge: 2026-09-29
temas: [okrs, dwh]
creado: 2026-10-02
revisar: true
---

# PR 413 — `finance.okr.evaluacion` como texto

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `refactor/postgres-to-redshift`.

**Proyecto**: sin link (el PR habla de OKR pero no nombra el proyecto; candidato: OKRs Finanzas). `revisar: true`.

## Qué cambió
- `models/intermediate/finance/okr.sql`: `evaluacion` se pasa como string (junto a `tipo_calculo`) en vez de la limpieza numérica + cast `decimal(18,4)`.
- `valor_actual`, `target`, `target_mensual_ytd`, `target_final` y `baseline` mantienen la conversión numérica (vienen en formato es-CL y parsean bien).

## Por qué
- `evaluacion` en `finance.raw__okr` es la frecuencia de evaluación del KR (`Mensual` / `Trimestral` / `Semestral` / `Anual`), no un número; con el cast quedaba NULL en las 2.124 filas.
- Validación del PR: tras `dbt run -s okr`, la columna es `varchar` y cuadra 1:1 con `raw__okr`. El tablero OKR de Evidence solo la lee con `MAX()`, así que el cambio de tipo no lo rompe.

## Link
https://github.com/agendapro/agendapro-dbt/pull/413

## Biblioteca que podría quedar desactualizada
- Ninguna nota de `Biblioteca/` documenta `finance.okr`.

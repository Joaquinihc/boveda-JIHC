---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 482
url: https://github.com/agendapro/agendapro-dbt/pull/482
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-09
temas: [headcount, centros-de-costo, dwh]
creado: 2026-11-13
revisar: false
---

# PR 482 — `finance.headcount`: columna `employer` (AgendaPro | Reservo) y centros de costo de Reservo

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/headcount-employer` → `refactor/postgres-to-redshift`. Creado y mergeado el 9-nov (sin review humana registrada; revisión de CodeRabbit). Sigue al PR 481.

**Proyecto**: sin link (el PR no nombra proyecto). Ya citado como PR #482 en [[finance.headcount — employer, clave de persona y carga desde la planilla]], [[Definir cómo clasificar a las personas de Reservo en la planilla de headcount]] y [[Distribuir por mercado las remuneraciones del equipo de soporte (Daniela Droguet e Ignacio Monsalve)]].

## Qué cambió
- Columna **`employer`** (`AgendaPro` | `Reservo`) en `finance.headcount`, desde la columna I de la pestaña «Headcount» de «2026 DWH Loading». Si la tabla raw aún no trae la columna, el modelo lo detecta al compilar y emite NULL → `AgendaPro`.
- La clave de deduplicación incluye `employer` (un homónimo entre compañías no se colapsa).
- Tests `employer` not_null y accepted_values en `error`; el test singular de una fila por persona-mes también usa `employer`.
- Seed `headcount_cost_center_mapping`: se commitean 7 códigos que ya estaban en producción pero faltaban en la rama y se agregan `REV-SUPP-AR` / `REV-SUPP-MX` (soporte dedicado a un mercado que además lo onboardea; confirmado con Pablo Saintard el 2-nov).
- Espejos sin PII regenerados (`employer` declarado `pii: false`).
- Validación: filas ago/sep/oct-26 = 191 / 195 / 203, igual que tras el 481.

## Por qué
- Cargar al personal de Reservo en la misma tabla sin mezclarlo con las cifras de AgendaPro.

## Link
https://github.com/agendapro/agendapro-dbt/pull/482

## Biblioteca que podría quedar desactualizada
- Ninguna identificada: [[finance.headcount — employer, clave de persona y carga desde la planilla]] ya documenta este cambio.

---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 395
url: https://github.com/agendapro/agendapro-dbt/pull/395
autor: Joaquinihc
merge: 2026-09-22
temas: [margenes, payments, dwh]
creado: 2026-09-23
revisar: true
---

# PR 395 — fix `company_margin`: revenue canónico en el spine; clamp de drivers solo en la asignación

> Borrador cosechado por la tarea PRs (vie). Rama `refactor/postgres-to-redshift`. Continúa el PR 394.

**Proyecto**: sin link (el PR no nombra el proyecto; candidato: Márgenes por Cliente y Producto). `revisar: true` para que Joaquín decida.

## Qué cambió
- `int_company_margin_spine_monthly` ya no fuerza `revenue_*_usd` / `fee_pos_usd` a `>= 0`: conserva el revenue canónico con negativos (refunds).
- El clamp `>= 0` se movió al CTE `drv` de `int_company_margin_cost_allocation_monthly`, único lugar donde se leen drivers (un driver negativo sigue sin recibir costo; Σ share = 1).
- Doc de metodología actualizado.

## Por qué
- La primera corrida programada (Prefect, 22-sep 09:50) falló el test `assert_company_margin_revenue_reconciles`: el revenue Payments del mart quedaba sobre `dwh.payments_revenue` por la suma exacta de las filas de fee negativas descartadas (0,109 % en sep-2026 parcial, tolerancia 0,1 %). Tras el fix, diferencia 0,00 en ago y sep.

## Link
https://github.com/agendapro/agendapro-dbt/pull/395

## Biblioteca que podría quedar desactualizada
- Ninguna directa. Si se crea una nota de método para el margen por compañía, debe decir que el revenue incluye refunds y que el clamp aplica solo a drivers.

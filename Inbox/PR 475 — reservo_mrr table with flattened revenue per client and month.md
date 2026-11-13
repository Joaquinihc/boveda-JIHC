---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 475
url: https://github.com/agendapro/agendapro-dbt/pull/475
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-07
temas: [revenue, mrr, dwh]
creado: 2026-11-13
revisar: false
---

# PR 475 — Tabla `reservo.reservo_mrr`: revenue aplanado por cliente y mes

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/reservo-mrr` → `refactor/postgres-to-redshift`. Creado el 6-nov, mergeado el 7-nov. Revisado por igutir-AgendaPro (lógica OK; pidió dar a `evidence_user` acceso solo a las dos tablas, no el rol `role_reservo_readonly`).

**Proyecto**: sin link (el PR no nombra proyecto). Ya citado como dbt PR #475 en [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]] (T-111).

## Qué cambió
- `reservo.reservo_mrr`: una fila por cliente de Reservo y mes, con la forma de `dwh.mrr`. Revenue aplanado en moneda local y en USD (current, constant y budget 2026), caja cobrada, fecha de primer pago, `change_category` (new, reactivation, upgrade, downgrade, churn) y `segment`.
- `segment`: plan Individual = B2C; cualquier otro plan informado = B2B3 (regla acordada con Pablo Lucero); pagos sin plan → NULL.
- `reservo.int_reservo_payments_flattened`: una fila por pago y mes cubierto, con la regla de cobertura del workbook de Reservo (días del período, rangos fijos, tope de 12 meses, pagos de 8+ cuotas tratados como anuales).
- New merchant = primer pago de la historia en el mes; quien vuelve tras años sin pagar es `reactivation`.
- **No sirve para contar merchants**: Reservo los cuenta desde su tabla de estado de clientes (incluye sucursales pagadas por su cadena y cuentas nuevas sin pago aún).
- Churn ruidoso (~200–280 filas/mes, muchos pagadores tardíos que reactivan); el mes del último pago cargado va `partial` y sin churn.
- Validación vs snapshot del 2-nov: cobertura igual al workbook en 79.004 pagos; revenue aplanado igual al seed de Flash Revenue en 268 celdas; `mrr_usd` igual al USD de Flash (dif. máx. 0,02 USD); 27 clientes que el seed contaba como new salen como `reactivation`.
- Ambos modelos con tag `redshift_marts` y grant `SELECT` a `role_reservo_readonly`.

## Por qué
- Llevar el revenue de Reservo al DWH para que Evidence lo lea sin el workbook (lo usa el PR 888 de Evidence).

## Link
https://github.com/agendapro/agendapro-dbt/pull/475

## Biblioteca que podría quedar desactualizada
- [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]] — dice que Reservo no está en dbt; ahora existe el esquema `reservo` con estos dos modelos.
- [[New Merchants (nuevas altas)]] — definición de new de Reservo (primer pago de la historia).
- [[Segment]] — regla de segmento de Reservo (Individual = B2C).

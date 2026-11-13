---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 888
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/888
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-07
temas: [board, revenue, evidence, definiciones]
creado: 2026-11-13
revisar: false
---

# PR 888 — Board Book: revenue y new merchants de Reservo desde el DWH, gráficos B2B3 del grupo

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/board-book-reservo-dwh` → `main`. Creado y mergeado el 7-nov (aprobado por nicofrem-agpro, sin observaciones). Dependía del PR 475 de dbt.

**Proyecto**: sin link (el PR no nombra proyecto). Ya citado como #888 en [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]] (T-111).

## Qué cambió
- **Revenue** y **new merchants** de Reservo en el Board Book pasan a leerse del DWH, con dos extracciones nuevas sobre el esquema dbt `reservo`: `reservo_revenue_monthly` (revenue aplanado por mes × país × moneda, con `month_status` y `data_through`; los meses posteriores al último pago son ingreso diferido y quedan fuera) y `reservo_new_merchants_monthly` (`change_category = 'new'` por mes × país × segmento).
- El **stock de merchants** de Reservo sigue en el seed del workbook a propósito (Reservo cuenta activos desde el estado del cliente, que sus pagos no traen).
- **Lámina 7**: dos gráficos de grupo, «B2B3 Merchants — AgendaPro + Reservo» y «New B2B3 Merchants — AgendaPro + Reservo», con Reservo desde oct-26. Reservo B2B3 = cualquier plan contratado distinto de Individual (regla acordada con Pablo Lucero). En este PR el stock B2B3 mostraba solo AgendaPro hasta tener el split por plan (pedido a Christopher Lagos).
- **Cambio de definición de new merchants de Reservo**: el seed contaba el primer pago dentro de la ventana del workbook (desde 2024-01); el DWH cuenta el primer pago de la historia, así que quienes vuelven tras pagos de 2018–2023 pasan a reactivación. Oct-26: 136 → 135. Efecto por mes en `docs/METRIC_DEFINITIONS.md` (ítem 112, inconsistencia 60).
- **El revenue no cambia**: sobre el snapshot del 2-nov cuadra al centavo con el seed por mes, país y moneda.
- Brecha conocida en el PR: Flash Revenue seguía leyendo el seed (oct-26: 135 en Board Book vs 136 en Flash).
- Tests `npm run test:sql` 456/456.

## Por qué
- Dejar de depender del workbook para revenue y altas de Reservo una vez que el DW de Reservo llegó a Redshift.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/888

## Biblioteca que podría quedar desactualizada
- [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]] — dice que Reservo «no está en el DWH ni en dbt»; revenue y new merchants ya salen del esquema `reservo`.
- [[New Merchants (nuevas altas)]] — new merchants de Reservo = primer pago de la historia (no de la ventana del workbook).

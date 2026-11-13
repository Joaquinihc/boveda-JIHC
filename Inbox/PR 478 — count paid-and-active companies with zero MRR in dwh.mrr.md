---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 478
url: https://github.com/agendapro/agendapro-dbt/pull/478
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-09
temas: [mrr, waterfall, dwh, definiciones]
creado: 2026-11-13
revisar: true
---

# PR 478 — `dwh.mrr` cuenta a las compañías pagadas y activas con MRR 0

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `fix/mrr-first-active-paid-active` → `refactor/postgres-to-redshift`. Creado el 6-nov, mergeado el 9-nov (sin review humana registrada). No se replicó a `master` (Postgres).

**Proyecto**: sin link (el PR no nombra proyecto). Candidatas por afinidad, sin mención explícita: [[Cierre de mes octubre]] (tema promo 3x1) y la tarea descartada [[Proponer cómo contar como New Merchants a los comercios de la promo 3x1]] — por eso `revisar: true`.

## Qué cambió
- La promo 3x1 de oct-26 aplica un cupón del 100 % (`1X3-SAAS-MES2Y3-FREE`) por dos meses tras el primer pago, así que el MRR de Chargebee es 0. `first_active_month` solo miraba MRR > 0, por lo que `mrr.sql` descartaba a esas compañías y el gate de New Merchants del Board Book (`is_first_month_paying`) no las contaba: B2B3 de octubre mostraba 471 venues en vez de 533.
- `first_active_month` = primer mes con MRR > 0 **o** pagada y activa (`active`/`non_renewing` con pago exitoso, misma condición que `is_active_and_paid`; sin cuentas de prueba). `is_active` sigue siendo MRR > 0: **stock de merchants y churn no cambian**.
- Cuando una de estas compañías pasa a MRR > 0 sin cancelar (termina el cupón), el mes se clasifica `upgrade` y no `reactivation` (como free_to_paid en el waterfall canónico).
- Validación: entran 87 compañías a `dwh.mrr` (67 con primer pago en 2026, casi todas 3x1) y 112 adelantan su mes `new`; cambios de categoría solo `(null)` → `new` (199 filas) y `new` → `upgrade` (112). Venues B2B3 del Board Book: oct 471 → 533, ene +2, feb +1. COM-01, COM-02 e ING-37 sin fallas.
- **Efectos aguas abajo que el PR deja por revisar**: payback de Evidence (`mbr_new_mrr_payback`) toma el MRR del mes `is_first_month_paying`, así que las 3x1 entran a la cohorte 2026 con new MRR 0 y las 112 que adelantan su primer mes caen en un mes con MRR 0 (arreglo separado en Evidence: tomar el primer mes con MRR > 0); las cohortes NRR (`nrr`, `nrr_cohorts`) asignan esas 112 a una cohorte anterior con MRR inicial 0.

## Por qué
- Que los merchants de la promo 3x1 (MRR 0 los primeros meses) cuenten como New Merchants.

## Link
https://github.com/agendapro/agendapro-dbt/pull/478

## Biblioteca que podría quedar desactualizada
- [[dwh.mrr — venues, is_active e is_active_and_paid]] — nueva definición de `first_active_month` / `is_first_month_paying` y la rama `upgrade`.
- [[New Merchants (nuevas altas)]] — las altas con cupón del 100 % ahora cuentan (oct-26 B2B3 471 → 533).
- [[Payback (Mensual)]] y [[Payback mensual en Evidence — implementación y fuentes]] — cohortes con new MRR 0.
- [[LTV por cohortes (nrr_cohorts) — método y trampas]] y [[NRR (Net Revenue Retention) - TWO constructs, label them]] — 112 compañías cambian de cohorte con MRR inicial 0.

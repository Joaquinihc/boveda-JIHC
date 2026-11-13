---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 891
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/891
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-09
temas: [pnl, board, evidence]
creado: 2026-11-13
revisar: false
---

# PR 891 — P&L Evolution para AgendaPro, Reservo y AgendaPro + Reservo

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/pl-three-companies` → `main`. Creado y mergeado el 9-nov (aprobado por nicofrem-agpro, «con observaciones»).

**Proyecto**: sin link (el PR no nombra proyecto). Ya citado como PR #891 en [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]] (T-111).

## Qué cambió
- Financial Performance y el Board Book mensual y trimestral muestran tres tablas de P&L: **AgendaPro**, **Reservo** y **AgendaPro + Reservo**. En los books, la 5.2b pasa a «P&L Evolution — AgendaPro» y se agregan 5.2b2 (Reservo) y 5.2b3 (grupo).
- Extracción nueva `dwh.profit_and_loss_by_company`: P&L mensual por compañía y categoría en las tres bases de FX, desde 2023. El grupo suma Reservo desde **oct-26** (mes de adquisición), igual que la hoja «P&L (AP+Reservo)» del P&L Master. `company` es una vista, no una partición: nunca sumar entre compañías.
- Tarjetas KPI, Objectives y Actuals vs Budget siguen solo AgendaPro (el Budget 2026 excluye Reservo).
- Notas en página: cuándo entra Reservo al consolidado; por qué el revenue contable de Reservo difiere de los pagos aplanados; que la nómina de Reservo antes de oct-26 no está separada por centro de costo (pendiente en el P&L Master), así que sus líneas de costo previas no son comparables, pero su EBITDA sí.
- Datos al cargar: las hojas Split 1/2/3 - DB del P&L Master dejaron de filtrar `type = 'Real'` el 9-nov y `finance.profit_and_loss` trae 912 filas de Reservo (ene–oct 2026). AgendaPro cuadra exacto con un snapshot del mismo día; Reservo cuadra con la hoja «P&L (Reservo)» a ±2 USD por celda.
- Para AgendaPro no cambian los números; cambios visibles: «–» en vez de celda vacía y la tabla siempre con 25 filas.
- Decisión abierta en el PR: el parquet nuevo `/data/dwh/profit_and_loss_by_company/` no está protegido por el auth-guard (opciones: renombrar a `profit_and_loses`, pedir a DevOps el gate de `/data/dwh/profit_and_loss`, o dejarlo). Ver [[Claude Code — Parquet del P&L sin gate y auth-guard.zip desactualizado]].
- Tests `npm run test:sql` 490/490; test nuevo `pl-three-companies` (22 casos).

## Por qué
- Mostrar el P&L de Reservo y del grupo junto al de AgendaPro desde la adquisición.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/891

## Biblioteca que podría quedar desactualizada
- [[EBITDA - EBITDA Margin]] y [[P&L Business-Line Decomposition (Verticales - Payments - Resto → EBITDA)]] — no documentan la vista por compañía ni el consolidado con Reservo desde oct-26.
- [[Definiciones KPIs del Board (DB KPIs)]] — láminas nuevas 5.2b2 y 5.2b3.

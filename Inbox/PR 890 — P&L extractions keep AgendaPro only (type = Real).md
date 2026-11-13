---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 890
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/890
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-09
temas: [pnl, cac, evidence]
creado: 2026-11-13
revisar: false
---

# PR 890 — Las extracciones del P&L se quedan solo con AgendaPro (`type = 'Real'`)

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `fix/pl-agendapro-only-guard` → `main`. Creado y mergeado el 9-nov (aprobado por nicofrem-agpro, «con observaciones»).

**Proyecto**: sin link (el PR no nombra proyecto). Ya citado como PR #890 en [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]] (T-111) y en [[Claude Code — Parquet del P&L sin gate y auth-guard.zip desactualizado]].

## Qué cambió
- `finance.profit_and_loss` iba a traer el P&L de Reservo (`type = 'Reservo'`, con país Chile y centro de costo Chile / HQ). Sin filtro, esas filas se sumarían al CAC, payback, gasto de marketing, OKR y costo de personas de AgendaPro.
- Toda extracción que lee el P&L filtra `type = 'Real'` (AgendaPro). La compartida `dwh.profit_and_loss` se filtra en el origen. Editadas: `profit_and_loss`, `marketing_actuals_2026`, `marketing_actuals_ex_payroll_2026`, `marketing_cost_detail_2026`, `marketing_spend_channel`, `network_effects_metrics` (CTE `cac`); otras cinco ya filtraban.
- **Sin cambio de cifras hoy**: al 9-nov el DWH solo tenía `type = 'Real'`; `main` vs rama en Redshift dieron filas y totales idénticos.
- Test nuevo `pl-agendapro-only.test.mjs` (fixture con la misma línea para AgendaPro y Reservo; exige un `type = 'Real'` por referencia al P&L en todo `sources/`).
- `docs/METRIC_DEFINITIONS.md`: entrada nueva **P&L company scope (`type`)**. `CLAUDE.md` y docs del repo dejan de decir que el parquet del P&L está protegido.
- **No incluido**: el gate de Lambda@Edge apunta a `/data/dwh/profit_and_loses/`, que ya no calza con nada; se deja igual porque seis páginas fuera de `/finance/` leen ese parquet y `/people/costs` ya muestra las líneas de costo sin gate.
- Tests `npm run test:sql` 467/467.

## Por qué
- Evitar que la carga de Reservo en el P&L contamine las métricas de AgendaPro.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/890

## Biblioteca que podría quedar desactualizada
- [[Fully Loaded CAC (B2B3)]] y [[CAC - the Gross-Margin → Contribution-Margin cost layer]] — el CAC desde el P&L ahora filtra `type = 'Real'`.
- [[Payback mensual en Evidence — implementación y fuentes]] — mismas extracciones de marketing con filtro de compañía.
- [[EBITDA - EBITDA Margin]] — alcance de compañía del P&L (`type`).

---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 767
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/767
autor: Joaquinihc
merge: 2026-09-28
temas: [board, reportes, evidence, definiciones]
creado: 2026-10-02
revisar: false
---

# PR 767 — puente de Cash Flow: saldo inicial = cierre previo + línea de efecto tipo de cambio

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `main`. Aprobado por nicofrem-agpro.

**Proyecto**: sin link (el PR no nombra proyecto). Sin tema `cash-flow` en el vocabulario (propuesto en el registro).

## Qué cambió
- Puente de caja: `Initial balance total` de cada período = `Final balance total` del período previo, y línea nueva **FX rate effect** (re-expresión de saldos iniciales en moneda local) que cierra `Initial + Operations + Finance + FX = Final`.
- La fila contable `Efecto tipo de cambio en Saldo Inicial` sale de Cash from Finance.
- Aplicado en Board Book mensual y trimestral (slide 5.4) y en Financial Performance → Cash Flow (ventana de 10 meses; la fila residual "Working Capital Propay" se reemplaza por FX rate effect).
- Definición canónica nueva "Cash Flow bridge" en `docs/METRIC_DEFINITIONS.md` (acordada con el Head of Accounting y el CFO el 2026-09-24). FCF = variación del saldo final menos entradas de financiamiento.
- `cash_flow_monthly.sql`: limpia cuentas inexistentes en la ventana de 25 meses y una columna sin uso; output sin cambios.

## Por qué
- El puente no cuadraba: cada período abría con el saldo inicial al tipo de cambio del mes nuevo, y la fila contable de FX se contaba como flujo de financiamiento. Riverwood observó que EoP ≠ BoP siguiente y que los subtotales no sumaban.
- Es la estructura del *Cash Flow by Quarter* revisado enviado a Riverwood. El tipo de cambio de cierre mensual ya vive en `finance.cash_flow`.
- Validación del PR: todos los períodos cuadran y encadenan; el trimestral reproduce al dólar el archivo interno a tasas promedio.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/767

## Biblioteca que podría quedar desactualizada
- Ninguna nota de `Biblioteca/` documenta Cash Flow ni el puente de caja (búsqueda sin coincidencias).

---
type: borrador
origen: Claude Code
fecha: 2026-11-10
temas: [evidence, pnl]
revisar: true
---

# Claude Code — Parquet del P&L sin gate y auth-guard.zip desactualizado

- **Qué pasa**: el Lambda@Edge de Evidence (`aws/lambda-edge/auth-guard/index.mjs`) protege `/data/dwh/profit_and_loses/`, pero la extracción se llama `profit_and_loss` desde el 13-abr (commit `bfa6f45d`). Por eso `/data/dwh/profit_and_loss/` y `/data/dwh/profit_and_loss_by_company/` (que incluye el P&L de Reservo) se pueden descargar con el permiso `view`, sin `finance`.
- **Agravante**: el `auth-guard.zip` versionado (7-abr) no trae las entradas `company_margin_` ni `headcount_fin_` que sí están en `index.mjs`, así que no está confirmado que esas protecciones estén desplegadas.
- **Por qué no se corrigió solo**: seis páginas fuera de `/finance/` leen el parquet del P&L (marketing-spend, payback ×2, okr, people/costs, people/forecast) y se romperían para quien no tiene `finance`. Además `/people/costs` ya muestra todas las líneas de costo en una ruta sin protección.
- **Pendiente (decisión de Joaquín)**: preguntar a DevOps qué versión del Lambda está desplegada; luego elegir entre mover esas páginas a `/finance/`, darles extracciones más acotadas o renombrar la extracción por empresa a `profit_and_loses` para que quede protegida hoy. Repo `agendapro-dashboards-evidence`; docs corregidos en el PR #890.

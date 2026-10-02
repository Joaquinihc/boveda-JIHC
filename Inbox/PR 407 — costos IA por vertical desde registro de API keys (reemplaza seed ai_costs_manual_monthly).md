---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 407
url: https://github.com/agendapro/agendapro-dbt/pull/407
autor: Joaquinihc
estado_pr: abierto
merge:
temas: [ai, margenes, dwh, definiciones]
creado: 2026-10-02
revisar: true
---

# PR 407 — costos IA por vertical desde registro de API keys

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `refactor/postgres-to-redshift`. **Abierto** (creado 28-sep, sin merge al 2-oct). Modifica archivos del PR 394 (ver [[PR 394 — per-company operational gross margin by revenue stream (company_margin)]]).

**Proyecto**: sin link (el PR no nombra el proyecto; candidato: Márgenes por Cliente y Producto). `revisar: true`.

## Qué cambió
- `finance.ai_costs_manual_monthly` deja de ser seed manual y pasa a ser **modelo** (mes × vertical: costo de modelos, comercios activos IA, tarifa aproximada, costo de plataforma, total).
- Seed nuevo `ai_api_key_usage_monthly`: costo por API key y mes con el equipo responsable de la key **en ese mes** (ene–ago 2026 desde `ai_cost_detail.csv` de Evidence). Reasignar una key solo afecta meses nuevos; los cerrados no se recalculan. Tests YML: equipo no nulo, tipo válido, tipo Vertical solo Beauty/Health/Medspa.
- Costo de plataforma = tarifa por comercio activo IA × comercios activos IA de `mrr_ai` (tarifas escritas en el modelo); `int_company_margin_cost_pools_monthly` lee de ahí el recorte de AWS hacia IA.
- Costo de modelos IA solo entra al margen en meses con P&L real de Service Costs (filtro de mes cerrado).
- **Azure 100% IA**: el P&L registra Azure dentro de AWS (G0001); para el análisis de margen se descuenta todo Azure del pool de AWS (Software) y queda como pool excluido. La cuadratura contra el P&L sigue cerrando; el P&L no cambia.
- Skill `update-ai-api-costs` (un archivo): lee exports de Claude, OpenAI Costs y Azure, hereda el equipo del último mes y pregunta solo por keys nuevas.
- Sin cambios en spine, drivers, asignación, marts `company_margin_*` ni tests `assert_company_margin_*`.

## Por qué
- Pasar de un seed manual a un registro trazable por API key y equipo, con la plataforma IA visible y Azure tratado como costo de modelos IA.
- Pendiente declarado en el PR: en Evidence, `company_margin_reconciliation.sql` mostrará en la línea Platform una diferencia igual al Azure retirado (etiquetarla "Azure en AI Models" y cambiar el texto "seed manual"). Tras el merge queda huérfana `finance.ai_api_key_team_assignment` (borrar a mano).

## Link
https://github.com/agendapro/agendapro-dbt/pull/407

## Biblioteca que podría quedar desactualizada
- [[AI Gross Margin - AI Margin %]]: dice "AI Cost per the AI-P&L allocation" y "mapped directly per vertical"; la fuente del costo IA pasa a ser el registro de API keys + costo de plataforma, con Azure 100% IA.

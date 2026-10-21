---
type: tarea
id: T-044
estado: bloqueada
prioridad: alta
tipo: reporte
origen: notion
proyecto: "[[_P&L y Centros de Costo]]"
temas:
  - pnl
  - board
notion_id: 39dab34c233b80629d2de60f8c9ef019
notion_estado: Blocked
notion_prioridad: High
notion_actualizado: 2026-10-20T13:45Z
fuente: https://app.notion.com/39dab34c233b80629d2de60f8c9ef019
creado: 2026-07-14
revisar: false
---

# Agregar P&L verticales a Board Book

Arreglar P&L del equipo de Payments. El gasto de API Keys es engorroso (registrar cada consola); buscar otra manera — el acceso a consolas fue para no depender de DevOps. Hacerlo 1 semana antes del cierre.

## Subtareas
- [ ] Arreglar P&L de Payments
- [ ] Automatizar registro de API Keys

## Notas

### Pendientes menores
- 2026-10-20 · Abrir el PR de Evidence del AI P&L by vertical cuando se mergee el PR #407 de dbt: rama `feat/ai-pl-models-from-api-key-registry` (worktree `agendapro-worktrees/ai-pl-registry`, sin commit). Costo de modelos desde `finance.ai_api_key_usage_monthly`, keys de desarrollo en línea propia, septiembre cerrado, plataforma en gris como aproximación y gráfico de revenue vs meta OKR como el de Informasayuno.
- 2026-10-20 · El Consolidado (`pages/finance/pl-by-vertical/consolidado.md`) cuenta dos veces el costo IA: clasifica IA por texto (`anthropic|azure|openai|chatgpt|foundry`), así que G0030–G0032 (desde sep-2026) y el Azure «AWS - Microsoft» de G0001 (desde abr-2026, 53k USD abr–ago) caen al costo central que absorbe SaaS, mientras la unidad AI ya trae ese costo desde el registro.

## Historial
- 2026-09-22 · Importada desde Notion (In progress).
- 2026-09-29 · Notion: página editada
- 2026-10-20 · Avance desde Claude Code: 2 pendientes menores (PR de Evidence del AI P&L y doble conteo del Consolidado).
- 2026-10-21 · Notion: In progress → Blocked

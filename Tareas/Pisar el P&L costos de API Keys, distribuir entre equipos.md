---
type: tarea
id: T-058
estado: en-curso
prioridad: media
tipo: gestion
origen: notion
proyecto: "[[_P&L y Centros de Costo]]"
temas: [pnl, ai, centros-de-costo]
notion_id: 3d6ab34c233b801599b2fcb1a686c710
notion_estado: "In progress"
notion_prioridad: "Medium"
notion_actualizado: "2026-09-09T18:03Z"
fuente: https://app.notion.com/3d6ab34c233b801599b2fcb1a686c710
creado: 2026-09-09
revisar: false
---

# Pisar el P&L costos de API Keys, distribuir entre equipos

## Subtareas
- [ ] Recargar el seed `ai_api_key_usage_monthly` con `--full-refresh` y mergear el PR #407 — listo cuando el PR esté mergeado
- [ ] Crear en Odoo y en la Master las 12 cuentas G y las 8 analíticas nuevas — listo cuando existan en ambos lados
- [ ] Asiento de reclasificación ene–sep 2026 en Odoo con contabilidad — listo cuando esté contabilizado

## Notas

### 2026-10-02 · Imputación contable de API keys
- Análisis con los montos y el desfase de facturación: [[20261002-01 Reclasificación contable de costos de API keys de IA]].
- Reglas de centro de costo y cuenta, más la aprobación: [[Imputación contable de costos de IA (API keys)]].
- **Cuentas G nuevas:**
  - G0030–G0035: servicio por equipo, Platform.
  - G0205: soporte Marketplace.
  - G2112–G2116: tecnología por vertical, Development.
- **Analíticas nuevas:** BTY-PLAT, HLT-PLAT, MKP-PLAT, MSP-PLAT, P&T-PLAT, PAY-PLAT, MKP-SUPP y PAY-TECH.
- **Repo:** `agendapro-dbt`, rama `feat/ai-api-key-team-seed`, commit `5bfed446`, [[PR 407 — costos IA por vertical desde registro de API keys (reemplaza seed ai_costs_manual_monthly)]]. El registro lleva centro de costo y cuentas por key, y la skill `update-ai-api-costs` exige la aprobación de Joaquín para las keys nuevas.

### 2026-10-20 · Estado del PR #407
- El `--full-refresh` del seed `ai_api_key_usage_monthly` ya está corrido (Joaquín, 14-oct): el DWH tiene las 19 columnas, con `usage_type` y las cuentas por key. Falta el merge del PR #407, que espera el rediseño del margen a solo P&L ([[Análisis margen por cliente y por producto|T-056]]).

### Pendientes menores
- 2026-10-20 · Agregar dos tests al registro de API keys: toda key de Azure es de una vertical (si no, su costo desaparece), y toda key imputada a Platform de una vertical es de ese equipo.
- 2026-10-20 · Arreglar 5 errores del script de la skill `update-ai-api-costs`: una key de Claude en dos workspaces genera filas duplicadas; al recargar un mes pasado hereda una reasignación posterior; no reconoce fechas en inglés del export de Azure; los cargos de Azure sin resource group se asignan a Beauty sin aprobación; `--assign` distingue mayúsculas en Azure.

## Historial
- 2026-09-22 · Importada desde Notion (In progress).
- 2026-10-02 · Mencionada de nuevo en [[2026-10-01 Costos Azure y OpenAI]] (catalogar costos de Azure y OpenAI en el P&L)
- 2026-10-19 · Avance desde Claude Code: imputación contable de las API keys (cuentas G nuevas, centro de costo y cuentas en el registro, PR #407 commit 5bfed446); 3 subtareas nuevas.
- 2026-10-20 · Avance desde Claude Code: full-refresh del seed ya corrido; 2 pendientes menores (tests del registro y errores del script de la skill).

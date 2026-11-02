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
- [ ] Que Nicolás cree en Odoo las analíticas G2112 y G2114 y registre el asiento de octubre — listo cuando el asiento esté publicado
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

### 2026-11-02 · Octubre y planilla para Nicolás
- **Octubre en la planilla** [API Keys Costs 2026](https://docs.google.com/spreadsheets/d/1JHzp6l86pgBbR6rX6scnkc974nlOojlcNB_8X4s5Z0g), hoja «API Keys Costs»: 31 keys y 7.542,21 USD con IVA. Por proveedor: Claude 3.532,55, OpenAI 2.897,02 y Azure 1.112,64. Por uso: servicio 6.007,09 y desarrollo 1.535,11. Fuente: `finance.ai_api_key_usage_monthly`, corte 2-nov-2026.
- **6 keys nuevas, todas de Claude:** finance-odoo-mcp (ADM-HQ), marketplace-backoffice (MKP-TECH-HQ), medspa-engage-production-r (MSP-PLAT-HQ), medspa-engage-staging-r (MSP-TECH-HQ), prisma-production-v2 (HLT-PLAT-HQ) y prisma-staging-v2 (HLT-TECH-HQ).
- **Contabilizado en Odoo en octubre** (AgendaPro Inc, lectura del 2-nov): Claude 25.903,99 en 5.2.25, OpenAI 3.042,45 en 5.2.22 y Azure 1.215,74 en 5.2.0. Alcanza para la resta de los tres proveedores.
- **Planilla alineada con Odoo:** la cuenta de servicio es 5.2.73 Platform IA (columna L); la columna M «Analítica en Odoo» marca «(por crear)» cuando la analítica no existe; la hoja «CECOs Odoo» tiene las 164 analíticas activas del plan Centros, y «Cuentas G nuevas» lleva la analítica y su estado.
- **Decisión de Joaquín (2-nov):** desde octubre, servicio y desarrollo van separados. El desarrollo usa la analítica de su vertical sobre 5.2.25 G2104; septiembre no se corrige. Detalle en [[Imputación contable de costos de IA (API keys)]].
- **Para [[Nicolás Astudillo]]:** las instrucciones del asiento de octubre quedaron en un borrador de DM en Slack (hoja «Last month», un asiento por proveedor), sin enviar al 2-nov. Faltan en Odoo las analíticas G2112 y G2114. El cierre automático de períodos corre el 3-nov a las 9:50.
- **Skill nueva:** `prepare-ai-api-costs-reclass` en `~/.claude/skills/`, la segunda parte después de `update-ai-api-costs`.
- **Repo:** en `agendapro-dbt` (worktree `company-margin`, rama `feat/ai-api-key-team-seed`), el CSV del seed con las 31 filas de octubre está sin commit.
- **Subtarea reemplazada:** «Crear en Odoo y en la Master las 12 cuentas G y las 8 analíticas nuevas — listo cuando existan en ambos lados». Al 2-nov, los 12 G-codes están en Mappings de la Master. En Odoo son analíticas del plan Centros, no cuentas: las 8 analíticas de CECO existen, y de los G-codes faltan G2112 y G2114. Lo que queda pasó a la segunda subtarea.

### Pendientes menores
- 2026-10-20 · Agregar dos tests al registro de API keys: toda key de Azure es de una vertical (si no, su costo desaparece), y toda key imputada a Platform de una vertical es de ese equipo.
- 2026-10-20 · Arreglar 5 errores del script de la skill `update-ai-api-costs`: una key de Claude en dos workspaces genera filas duplicadas; al recargar un mes pasado hereda una reasignación posterior; no reconoce fechas en inglés del export de Azure; los cargos de Azure sin resource group se asignan a Beauty sin aprobación; `--assign` distingue mayúsculas en Azure.

## Historial
- 2026-09-22 · Importada desde Notion (In progress).
- 2026-10-02 · Mencionada de nuevo en [[2026-10-01 Costos Azure y OpenAI]] (catalogar costos de Azure y OpenAI en el P&L)
- 2026-10-19 · Avance desde Claude Code: imputación contable de las API keys (cuentas G nuevas, centro de costo y cuentas en el registro, PR #407 commit 5bfed446); 3 subtareas nuevas.
- 2026-10-20 · Avance desde Claude Code: full-refresh del seed ya corrido; 2 pendientes menores (tests del registro y errores del script de la skill).
- 2026-11-02 · Avance desde Claude Code: octubre cargado en la planilla API Keys Costs 2026 y planilla alineada con Odoo; borrador para Nicolás; skill `prepare-ai-api-costs-reclass`. La subtarea de cuentas y analíticas se reemplazó por la de G2112/G2114 y el asiento de octubre (Joaquín en conversación).

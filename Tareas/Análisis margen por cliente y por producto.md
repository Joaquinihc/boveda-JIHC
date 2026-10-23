---
type: tarea
id: T-056
estado: en-curso
prioridad: alta
tipo: analisis
origen: notion
proyecto: "[[_Márgenes por Cliente y Producto]]"
temas: [margenes]
notion_id: 3d6ab34c233b805091dde8c188a8591e
notion_estado: "In progress"
notion_prioridad: "High"
notion_actualizado: "2026-10-20T13:40Z"
fuente: https://app.notion.com/3d6ab34c233b805091dde8c188a8591e
creado: 2026-09-09
revisar: false
---

# Análisis margen por cliente y por producto

3 revenue streams. Pagos: fees y costos listos (location_sales_days → company_sales_months) ✅. AI: ingresos en mrr_ai; costos = API Keys aprox (P&L by vertical) + plataforma ponderada por clientes; falta separar por producto (agente_ai/Julia vs Sofía) — idea: pedir consumo de tokens por cliente/producto a las verticales. SaaS: relacionar con uso de features (reporte general/features-usage; 'Company feature adoption' como inspiración, ojo con población vs venues de MRR). Llegar hasta costos directos del P&L (Plataforma); dejarlo en Company management. Considerando costos operacionales = Costos de servicio − expansión + modelos de IA. Modelo documentado en Notion: 'Cómo está construido el modelo de margen por compañía'.

## Subtareas
- [x] Verificar el plan antes de dejar a Claude desarrollar los modelos dbt
- [ ] Pasar el margen por compañía a solo P&L en el PR #407: costo IA desde G0030–G0032 a partir de sep-2026 (antes en nulo), sin el registro de API keys — listo cuando el cambio esté en el PR y la cuadratura contra el P&L dé 0
- [ ] Hablar con Nicolás y mandarle el detalle de costos de IA por equipo vertical y por mes (mar–ago 2026), con el desglose por proveedor, cuenta y centro de costo, para que los reclasifique en Odoo (asiento de [[Pisar el P&L costos de API Keys, distribuir entre equipos|T-058]]) — listo cuando Nicolás tenga el detalle y acuerden cómo contabilizarlo
- [ ] Definir distribución de costos AI por producto
- [x] Definir proxy de uso de features para SaaS --> usé cantidad de bookings y locations de cada company.
- [ ] Elegir 46 o 64 columnas y construir `company_margin_by_company_monthly` (una fila por compañía y mes) — listo cuando la tabla esté en un PR

## Notas

### Pendientes menores
- 2026-10-02 · Antes de que las cuentas G0030–G0035 tengan movimiento en el P&L, agregar reglas en `company_margin_cost_pool_rules` (stream AI, con su vertical) y dejar de usar el registro de API keys en los meses contabilizados. Si no, el costo de modelos IA se cuenta dos veces. Ver [[Imputación contable de costos de IA (API keys)]].
- 2026-10-20 · El aviso del 2026-10-02 se confirmó con el cierre de septiembre (ver la sección de abajo). Lo cubre la subtarea «Pasar el margen por compañía a solo P&L».
- 2026-10-20 · Decidir qué hacer con el Azure que queda en G0001 («AWS - Microsoft», facturado con un mes o más de atraso; sep-2026: 6.190 USD, más un abono de reclasificación de +1.153 con detalle «5.2.0 G0001 - AWS»). Si se mantiene como costo IA, pedirle a Contabilidad que el abono de cada reclasificación diga «AI models», como el cargo en G0030.
- 2026-10-20 · Si el rediseño deja sin uso el modelo `ai_costs_manual_monthly`, sacarlo del PR #407 y borrar la tabla `finance.ai_costs_manual_monthly` (hoy es el seed viejo, ene–ago 2026).
- 2026-10-20 · Tras el merge del PR #407 ([[Pisar el P&L costos de API Keys, distribuir entre equipos|T-058]]): borrar a mano `finance.ai_api_key_team_assignment` (huérfana, 111 filas, sin vistas dependientes).
- 2026-10-20 · Ajustar `sources/dwh/finance/company_margin_reconciliation.sql` en Evidence al diseño final (hoy dice «seed manual» y no etiqueta el Azure excluido).

### 2026-10-20 · Revisión del PR #407 y criterio solo P&L
- **Doble conteo en septiembre.** El P&L real de sep-2026 ya trae G0030 2.531 · G0031 2.859 · G0032 569 USD (corte DWH 14-oct, `finance.profit_and_loss`), iguales al registro de API keys (2.530,61 · 2.859,09 · 568,97). Sin reglas en `company_margin_cost_pool_rules`, esas cuentas caían en Platform → Software, y el pool AI Models sumaba los mismos 5.959 USD desde el registro.
- **Criterio en definición (Joaquín, 2026-10-20):** el margen por compañía y por stream usa solo el P&L; el registro de API keys queda solo para el AI P&L by vertical. Idea: calcular el margen IA desde sep-2026, primer mes con las cuentas G0030–G0032, y dejarlo en nulo antes.
- **Pregunta abierta:** qué muestra la línea IA de una compañía antes de sep-2026, y qué pasa con el costo que ya se asigna a IA en esos meses (recorte de plataforma desde AWS, soporte Health G0202).
- Los arreglos del 2026-10-19 (worktree `agendapro-worktrees/company-margin`, rama `feat/ai-api-key-team-seed`, sin commit) excluían G0030–G0032 y seguían usando el registro; con este criterio quedan obsoletos y hay que rehacerlos.

## Historial
- 2026-09-22 · Importada desde Notion (In progress).
- 2026-09-29 · Notion: In progress → To Review
- 2026-10-02 · Mencionada de nuevo en [[2026-09-29 FP&A Weekly]] (dejar costos de plataforma como estimación aparte y redondeada)
- 2026-10-19 · Avance desde Claude Code: propuesta de tabla ancha por compañía (46 o 64 columnas), falta elegir; aviso de doble conteo con las cuentas G0030–G0035.
- 2026-10-20 · Avance desde Claude Code: revisión del PR #407 (doble conteo sep-2026), criterio solo P&L en definición; 1 subtarea y 5 pendientes menores.
- 2026-10-21 · Notion: To Review → In progress
- 2026-10-21 · Avance desde Claude Code: subtarea nueva «Hablar con Nicolás…» a pedido de Joaquín (4 abiertas, con su OK); acordado en la FP&A Weekly del 20-oct (notas en Notion).
- 2026-10-21 · Avance desde Claude Code: PR #407 con el costo de modelos IA solo desde el P&L (G0030–G0032 por vertical, repartido por uso medido), margen nulo sin revenue o costo y `ai_costs_manual_monthly` eliminado (commit `3da73212`). QA jul–sep sin fallas; la cuadratura contra el P&L da 0.
- 2026-10-23 · Mencionada de nuevo en [[2026-10-20 FP&A Weekly]] (mandar a Nicolás los costos de IA de septiembre para reclasificarlos hacia atrás, suma cero)

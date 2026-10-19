---
type: tarea
id: T-056
estado: en-revision
prioridad: alta
tipo: analisis
origen: notion
proyecto: "[[_Márgenes por Cliente y Producto]]"
temas: [margenes]
notion_id: 3d6ab34c233b805091dde8c188a8591e
notion_estado: "To Review"
notion_prioridad: "High"
notion_actualizado: "2026-09-29T11:13Z"
fuente: https://app.notion.com/3d6ab34c233b805091dde8c188a8591e
creado: 2026-09-09
revisar: false
---

# Análisis margen por cliente y por producto

3 revenue streams. Pagos: fees y costos listos (location_sales_days → company_sales_months) ✅. AI: ingresos en mrr_ai; costos = API Keys aprox (P&L by vertical) + plataforma ponderada por clientes; falta separar por producto (agente_ai/Julia vs Sofía) — idea: pedir consumo de tokens por cliente/producto a las verticales. SaaS: relacionar con uso de features (reporte general/features-usage; 'Company feature adoption' como inspiración, ojo con población vs venues de MRR). Llegar hasta costos directos del P&L (Plataforma); dejarlo en Company management. Considerando costos operacionales = Costos de servicio − expansión + modelos de IA. Modelo documentado en Notion: 'Cómo está construido el modelo de margen por compañía'.

## Subtareas
- [x] Verificar el plan antes de dejar a Claude desarrollar los modelos dbt
- [ ] Definir distribución de costos AI por producto
- [x] Definir proxy de uso de features para SaaS --> usé cantidad de bookings y locations de cada company.
- [ ] Elegir 46 o 64 columnas y construir `company_margin_by_company_monthly` (una fila por compañía y mes) — listo cuando la tabla esté en un PR

## Notas

### Pendientes menores
- 2026-10-02 · Antes de que las cuentas G0030–G0035 tengan movimiento en el P&L, agregar reglas en `company_margin_cost_pool_rules` (stream AI, con su vertical) y dejar de usar el registro de API keys en los meses contabilizados. Si no, el costo de modelos IA se cuenta dos veces. Ver [[Imputación contable de costos de IA (API keys)]].

## Historial
- 2026-09-22 · Importada desde Notion (In progress).
- 2026-09-29 · Notion: In progress → To Review
- 2026-10-02 · Mencionada de nuevo en [[2026-09-29 FP&A Weekly]] (dejar costos de plataforma como estimación aparte y redondeada)
- 2026-10-19 · Avance desde Claude Code: propuesta de tabla ancha por compañía (46 o 64 columnas), falta elegir; aviso de doble conteo con las cuentas G0030–G0035.

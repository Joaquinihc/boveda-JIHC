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
notion_actualizado: "2026-11-20T00:47Z"
fuente: https://app.notion.com/3d6ab34c233b805091dde8c188a8591e
creado: 2026-09-09
revisar: false
---

# Análisis margen por cliente y por producto

3 revenue streams. Pagos: fees y costos listos (location_sales_days → company_sales_months) ✅. AI: ingresos en mrr_ai; costos = API Keys aprox (P&L by vertical) + plataforma ponderada por clientes; falta separar por producto (agente_ai/Julia vs Sofía) — idea: pedir consumo de tokens por cliente/producto a las verticales. SaaS: relacionar con uso de features (reporte general/features-usage; 'Company feature adoption' como inspiración, ojo con población vs venues de MRR). Llegar hasta costos directos del P&L (Plataforma); dejarlo en Company management. Considerando costos operacionales = Costos de servicio − expansión + modelos de IA. Modelo documentado en Notion: 'Cómo está construido el modelo de margen por compañía'.

## Subtareas
- [x] Verificar el plan antes de dejar a Claude desarrollar los modelos dbt
- [ ] Aplicar la versión 1 en dbt: PR 1 de la página 3 de Notion (tabla `company_margin_by_company_monthly` en rama nueva desde la base) — listo cuando el PR esté abierto y la cuadratura contra el P&L dé 0
- [ ] Hablar con Nicolás y mandarle el detalle de costos de IA por equipo vertical y por mes (mar–ago 2026), con el desglose por proveedor, cuenta y centro de costo, para que los reclasifique en Odoo (asiento de [[Pisar el P&L costos de API Keys, distribuir entre equipos|T-058]]) — listo cuando Nicolás tenga el detalle y acuerden cómo contabilizarlo
- [ ] Definir distribución de costos AI por producto
- [x] Definir proxy de uso de features para SaaS --> usé cantidad de bookings y locations de cada company.
- [x] Cerrar las decisiones abiertas de la versión 1 (A2 y S2 con Pablo; S3, P2, X2 y T1; terminales POS y llave de Software) — listo cuando estén marcadas en la página 0 de Notion

## Notas

### Pendientes menores
- 2026-11-19 · Decidir la propuesta P4 de la página 0 de Notion: sacar los contracargos de la llave de Payments (no aparecen en el P&L de Payment Processing).
- 2026-11-19 · Seguimiento con [[Nicolás Astudillo]]: Argentina no tiene procesadores contabilizados en mayo de 2026 (10,2k USD) y Colombia no los tiene desde mayo (unos 0,75k USD al mes).
- 2026-11-19 · Decidir los cuatro puntos del tablero Company Margin de Evidence (página 4 de Notion, sección 4): costo por línea del P&L, tipo de cambio constante, columnas de contexto y ranking.
- 2026-11-19 · Reducir el PR #407 al registro de API keys en una rama nueva desde la base y cerrarlo; coordinar antes con los cambios sin commit del worktree `agendapro-worktrees/company-margin`. Ver [[PR 407 — costos IA por vertical desde registro de API keys (reemplaza seed ai_costs_manual_monthly)]].
- 2026-11-19 · Pasar a «Vista previa» los diagramas nuevos de las páginas 2, 3 y 4 de Notion (quedan en modo código al crearse).
- 2026-11-19 · Subtarea reemplazada: «Pasar el margen por compañía a solo P&L en el PR #407: costo IA desde G0030–G0032 a partir de sep-2026 (antes en nulo), sin el registro de API keys — listo cuando el cambio esté en el PR y la cuadratura contra el P&L dé 0». La versión 1 se construye en una rama nueva desde la base y ya trae el costo de IA solo desde el P&L; pasó a la subtarea «Aplicar la versión 1 en dbt».
- 2026-11-18 · Preguntar a Yonathan Durán qué namespaces componen «BEAUTY» en su portal de costos de EKS y los montos actuales de RDS y Valkey por vertical (para refrescar la estimación de plataforma de AI, hoy de ago-2026).
- 2026-11-18 · Averiguar qué son las cuentas «frust…» de la factura de AWS (unos 13,5k USD en oct-2026) y si la cuenta AgendaPro Payments (1,4k USD) debería ir al stream Payments.
- 2026-11-18 · Pasar los dos diagramas de la página 2 de Notion («Tablas y variables») al formato que no corta el texto. Ver [[Diagramas Mermaid en Notion]]. → Resuelto el 19-nov: la página 2 se reescribió con diagramas en el formato nuevo.
- 2026-11-18 · Investigación de compañías que pagan y no agendan (2.112 en oct-2026; hipótesis de Emilio Latorre sobre el link de pago): en curso; falta dejarla en la hoja de Notion «Análisis inactividad» y escribirle a Emilio por Slack. → Resuelto el 18-nov: los hallazgos quedaron en esa hoja y se le escribió a Emilio por Slack; el link de pago no explica la inactividad.
- 2026-11-18 · Revisar por qué cerca de 1.100 compañías pagan menos de 3 USD al mes de Software (quedan con margen negativo con cualquier llave).
- 2026-11-18 · Subtarea reemplazada: «Elegir 46 o 64 columnas y construir `company_margin_by_company_monthly` (una fila por compañía y mes) — listo cuando la tabla esté en un PR». La elección quedó resuelta con la tabla de 18 columnas de la versión 1; construirla pasa al plan dbt (página 3 de Notion).
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

### 2026-11-18 · Versión 1 simplificada (plan en Notion, sin cambios en dbt)
- Diseño completo en las subpáginas de la tarea en Notion: 0 objetivo, reglas y decisiones · 1 reparto por stream (con diagramas) · 2 tablas y variables · 3 plan dbt · 4 borrador de pedido a los equipos verticales. Las páginas 0 a 3 traen cifras del P&L y no salen de Finanzas.
- Reparto: Payments = Payment Processing del P&L según el costo medido de `company_sales_months`; AI = modelos (G0030–G0032) más plataforma (tarifa 8,60 / 11,74 / 14,84 USD × compañías con AI, descontada de G0001), por revenue AI dentro de cada vertical; Software = el resto, con pasarelas (G0004) por revenue Software.
- En dbt: 27 reglas pasan a 11 en la macro `company_margin_cost_pool_rules`; el motor de reparto no se toca. Tabla nueva `finance.company_margin_by_company_monthly` de 18 columnas (tres ingresos, tres costos, margen en USD y en % por stream).
- Abierto: llave de Software (propuesta: nube y mensajería por reservas, soporte y retención por sucursales), terminales POS (propuesta: por sucursal con transacciones POS) y las propuestas A2, S2, S3, P2, X2 y T1 de la página 0.
- Hallazgos con números: [[20261118-01 Costos por stream — hallazgos versión 1]].

### 2026-11-19 · Versión 1 cerrada: decisiones, plan dbt y hojas de Notion
- **Hojas de Notion de la tarea**: 0 objetivo, reglas y decisiones · 1 reparto por stream · 2 la tabla y cómo se construye · 3 plan para dbt, escrito como guía de tareas · 4 cambios río abajo (Evidence) · 5 ideas para una versión 2. El borrador de pedido a los equipos verticales se eliminó; lo útil quedó en la hoja 5. Todas traen cifras del P&L y no salen de Finanzas.
- **Decisiones de Joaquín del 18 y 19-nov** (están en la tabla de la hoja 0): terminales POS por sucursal con POS; llaves de Software (mensajería por reservas, nube y soporte mitad sucursales y mitad reservas, retención y resto por sucursales); reservas sin importadas y con tope en el percentil 95 por sucursal; un solo factor mensual en Payments; soporte y retención siempre entre todas las compañías; facturación electrónica por el país de la cuenta; llave de AI, pasarelas y Azure como estaban propuestos. Son 6 llaves y 15 reglas.
- **Plan en dbt** (hoja 3, aprobado el 19-nov; nada aplicado): una sola tabla `finance.company_margin_by_company_monthly` con dos pasos intermedios ephemeral, en una rama nueva desde la base y no sobre el PR #407. Tres PRs: dbt agrega la tabla, Evidence migra el tablero, dbt borra la cadena antigua. Etapa final: mover la tabla de `finance` a `dwh` cuando esté validada.
- **Abierto**: propuesta P4 (contracargos en la llave de Payments), que no bloquea. Joaquín todavía no pidió empezar el PR 1.
- Hallazgos con números: [[20261119-01 Llaves de Software, país y ejecución — hallazgos versión 1]]. Scripts y prototipo: `_analisis/2026-11-company-margin-v1/` (carpeta GitHub).

### Archivo
- 2026-10-02 · Antes de que las cuentas G0030–G0035 tengan movimiento en el P&L, agregar reglas en `company_margin_cost_pool_rules` (stream AI, con su vertical) y dejar de usar el registro de API keys en los meses contabilizados. Si no, el costo de modelos IA se cuenta dos veces. Ver [[Imputación contable de costos de IA (API keys)]].

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
- 2026-10-28 · Notion: página editada
- 2026-10-30 · Mencionada de nuevo en [[2026-10-28 Márgenes por negocio, compañías y cash flow]] (vincular costos a producto y cuenta Odoo/P&L, criterios de distribución y tabla intermedia por cliente; además Companies Sales Months, payroll comercial de Payments a CAC, costos IA sin vertical con Jonathan, conciliación con el P&L y tablero en Evidence)
- 2026-10-30 · Avance desde Claude Code: tres subpáginas en Notion (costos por stream, tablas y variables, plan de avance y merge del PR #407); verificado que Payments no doble cuenta (costo medido solo es llave de reparto); carve-out IA de AWS marcado como único supuesto fuera del P&L, decisión abierta.
- 2026-11-02 · Notion: página editada
- 2026-11-16 · 1 pendiente menor pasó a Archivo (Monitor)
- 2026-11-18 · Notion: página editada
- 2026-11-18 · Avance desde Claude Code: auditoría de costos por stream (16-nov) y reorden de las subpáginas de Notion; el árbol antiguo «Cómo está construido el modelo» quedó marcado como desactualizado.
- 2026-11-18 · Avance desde Claude Code: versión 1 simplificada en Notion (17 y 18-nov): 4 llaves, plataforma de AI dentro del margen, tabla de 18 columnas, diagramas por stream. Subtarea de las 46 o 64 columnas reemplazada por «Cerrar las decisiones abiertas de la versión 1» (con OK de Joaquín; 4 abiertas). 5 pendientes menores.
- 2026-11-19 · Notion: página editada
- 2026-11-19 · Avance desde Claude Code: versión 1 cerrada (6 llaves, 15 reglas; decisiones del 18 y 19-nov en la hoja 0 de Notion), plan dbt de una sola tabla aprobado y hojas 4 y 5 nuevas. Subtarea «Cerrar las decisiones abiertas» tachada y «Pasar el margen a solo P&L en el PR #407» reemplazada por «Aplicar la versión 1 en dbt» (con OK de Joaquín; 3 abiertas). 5 pendientes menores.
- 2026-11-20 · Notion: página editada
- 2026-11-20 · Mencionada de nuevo en [[2026-11-17 FP&A Weekly]] (cerrar el primer cálculo corrigiendo los costos duplicados entre IA y software y cuadrarlo con el P&L; publicarlo en Finance y, si sirve, en el DWH: ya lo cubren la versión 1 cerrada el 19-nov y la subtarea «Aplicar la versión 1 en dbt») (Reuniones)

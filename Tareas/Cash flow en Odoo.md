---
type: tarea
estado: en-curso
prioridad: alta
tiempo:
tipo: otro
origen: notion
proyecto:
temas:
  - odoo
notion_id: 401ab34c233b80e59ec2d863538e559e
notion_estado: In progress
notion_prioridad: High
notion_actualizado: 2026-11-09T19:52Z
fuente: https://app.notion.com/401ab34c233b80e59ec2d863538e559e
fecha_limite:
creado: 2026-10-22
revisar: false
id: T-100
---

# Cash flow en Odoo

## Subtareas
- [ ] Cuadrar el cash flow recreado en Odoo con la planilla de Rafa: saldos primero, después movimientos, validando con la cartola del banco. Incluye lo que hoy está en un solo lado: el working capital de ProPay en Santander, los pagos con tarjeta de crédito Santander (≈ −USD 56k en Suppliers CL ene–sep) y el capex de computadores. Listo cuando octubre cuadre por país y las diferencias que queden estén explicadas.
- [ ] Proponerle a Pablo cómo presentar Payments en el cash flow (bloque Evidence arriba y bloque ProPay/pasarelas abajo, con referencias como los EEFF de dLocal). Listo cuando Pablo la valide o la ajuste.
- [ ] Cerrar la auditoría de la v2 de Odoo: revisar las hojas de Notion (4 diagramas a «Vista previa» y etiquetas D1–D7), llevarle D1–D6 a Pablo y mandar C1–C19 a Nicolás y Rafael. Listo cuando Pablo haya respondido las seis y Contabilidad tenga la lista.

## Notas
### 2026-11-10 · Propuestas de cash flow que pasaron aquí
- Subtarea 1: acción de [[2026-11-09 Coordinación Finanzas]] (lo principal del cierre de octubre). Suma [[Incorporar en Evidence los pagos con tarjeta de crédito Santander]] (T-107) y [[Integrar el capex (computadores) al cashflow desde Odoo]] (T-092).
- Subtarea 2: viene de [[Proponer cómo presentar Payments en el cash flow (saldo de comercios ProPay y pasarelas)]] (T-105). El saldo de comercios ProPay en Santander (~318 M CLP) es plata en tránsito que hoy solo Odoo muestra como caja.
- También pasaron aquí, como pendientes menores: [[Definir el Free Cash Flow respecto de Loan y ESOP]] (T-106), [[Definir cómo entra el cash flow al proceso de cierre (incluido Reservo)]] (T-104) y [[Analizar el efecto tipo de cambio en el cashflow con Claudio]] (T-093; su diagnóstico está en [[Cash flow — estructura nueva vs formato Causal]]).

### 2026-11-19 · Auditoría de la v2 de Odoo y comparación con Evidence
- Pablo dejó en producción la versión 2 del reporte de Odoo el 17-nov. Se explicó, se comparó con Evidence de enero a octubre de 2026 por sociedad y se auditó. Hallazgos y cifras: [[20261119-01 Cash flow Odoo v2 vs Evidence — comparación ene–oct 2026 y auditoría]].
- Notion: [Cash Flow en Odoo](https://app.notion.com/p/41dab34c233b80fba44fc2690deecf01), página principal y 7 subhojas (cambios de Pablo, definiciones con D1–D7, cuentas y caja, comparación mensual, auditoría, Evidence con Reservo, pendientes C1–C19).
- Archivos y scripts: `~/Documents/GitHub/_analisis/2026-11-cash-flow-odoo-v2/`. Cómo repetirlo: [[Cash flow en Odoo (v2) — estructura, mapa a Evidence y cómo reproducir]].
- Sobre la subtarea 1: octubre quedó comparado por sociedad con cada diferencia atribuida a una causa; faltan las correcciones de Contabilidad y lo «por confirmar». No se cierra todavía.
- Sobre la subtarea 2: la v2 ya separa Payments en una sección propia y en el reporte «Payments position». Joaquín decide si con eso queda cubierta.
- No se escribió en Odoo, Evidence ni dbt, y no se envió nada a Pablo, Nicolás ni Rafael.

### Pendientes menores
- 2026-11-19 · FCF: el de Odoo difiere del de Evidence por definición (deja fuera financing repayment y tipo de cambio, y registra el cobro al entrar a la pasarela), no por error. Cuál queda es la decisión D1 de la hoja 2 de Notion.
- 2026-11-19 · Marzo de México quedó sin cuadre contra la pantalla de Odoo (timeout). Repetir el export de ese mes.
- 2026-11-19 · 64 causas de diferencia quedaron «por confirmar» (109 mil USD) y 58 mil USD sin explicar, entre ellos los proveedores de SC3 de octubre. Se aclaran con las respuestas de Contabilidad.
- 2026-11-19 · Arreglos de Evidence detectados: fondo BBVA Colombia contado dos veces (773 USD), filas de saldo repetidas en `finance.cash_flow` y modelo dbt `cash_flow.sql` sin columna de sociedad.
- 2026-11-10 · FCF: el 27-oct se decidió excluir Loan y ESOP (FCF = operaciones + inversión, intercompany neteado). Falta que Odoo lo calcule igual que Evidence.
- 2026-11-10 · Pedir acceso a las cuentas bancarias de las sociedades de Reservo para incluirlas en el cash flow del cierre.
- 2026-11-10 · Efecto tipo de cambio: decidir si los reportes muestran la cuenta de tesorería como FX rate effect, y explicarle a Pablo cómo pega el tipo de cambio en cada país según su mezcla de ingresos y costos por moneda.

## Historial
- 2026-10-23 · Importada desde Notion (In progress)
- 2026-10-30 · Mencionada de nuevo en [[2026-10-27 Cash flow en Odoo — FCF, estructura y Payments (Rafa)]] (cerrar el cash flow por país y luego el consolidado, objetivo 30-oct)
- 2026-10-30 · Mencionada de nuevo en [[2026-10-28 Márgenes por negocio, compañías y cash flow]] (preparar la comparación Evidence vs Odoo y consultar a Rafa por las visualizaciones)
- 2026-10-30 · Mencionada de nuevo en [[2026-10-29 Réplica del cash flow por país para conciliar con Odoo (Rafa)]] (avance para la reunión con Pablo; documentar diferencias y listado de cambios Odoo vs Evidence)
- 2026-11-06 · Mencionada de nuevo en [[2026-11-05 Diferencia de tipo de cambio y cierre del P&L de octubre (Pablo)]] (revisar con Rafa el cierre del cash flow de octubre el 5-nov; da 3,1 vs 2,8 por Vambe y otros ítems)
- 2026-11-10 · Notion: página editada
- 2026-11-10 · Mencionada de nuevo en [[2026-11-09 Coordinación Finanzas]] (cuadrar el cash flow recreado en Odoo contra la planilla de Rafa: cuentas que faltan en un lado, como el working capital de ProPay, y montos distintos; saldos primero y validar con el estado de cuenta del banco)
- 2026-11-10 · Subtareas y contexto de T-092, T-093, T-104, T-105, T-106 y T-107 (Joaquín en conversación)
- 2026-11-19 · Avance desde Claude Code: explicación de la v2 de Odoo, comparación con Evidence ene–oct 2026 y auditoría, en Notion y en [[20261119-01 Cash flow Odoo v2 vs Evidence — comparación ene–oct 2026 y auditoría]]
- 2026-11-19 · Agregada como subtarea: «Cerrar la auditoría de la v2 de Odoo» y 4 pendientes menores, desde [[20261119-01 Cash flow Odoo v2 vs Evidence — comparación ene–oct 2026 y auditoría]] (Joaquín en conversación)

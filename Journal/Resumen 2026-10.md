---
type: resumen-mensual
mes: 2026-10
creado: 2026-11-02
---

# Resumen 2026-10

> Generado por el mantenimiento mensual. Las decisiones salen de reuniones con `verificado: false`: verificar antes de citar.

## Análisis creados (12)
- **P&L y Centros de Costo** (3): [[20261002-01 Reclasificación contable de costos de API keys de IA]] · [[20261007-01 Varianza P&L septiembre vs agosto 2026]] · [[20261029-01 Revisión del P&L de Reservo en Odoo vs estructura AgendaPro]]
- **SoT Ventas POS** (2): [[20261006-01 Board book sobre pos_sales_sot — impacto en ventas, attachment y payback]] · [[20261006-02 POS vs sedes del plan — tope de unidades y POS de respaldo]]
- **Activación** (2): [[20261001-01 Activation Velocity — Evidence vs tablas dbt]] · [[20261008-01 Activación en CX (bizops-cx-apps) vs Evidence y dbt]]
- **Churn y Graduación** (1): [[20261008-01 Ajustes de churn en Evidence pedidos por CX (Pablo Santa Inés)]]
- **Ad hoc** (4): [[20261007-01 Leads con cuenta creada en t0 por país y canal]] · [[20261008-01 LTV self-serve por cohortes]] · [[20261022-01 New Merchants board book vs new del waterfall software]] · [[20261029-01 Inconsistencias del cash flow — respuestas de Rafa y caso Global66 ago-26]]

## Decisiones
- **Reservo**: adquisición cerrada el 8-oct. El directorio sigue viendo solo AgendaPro (dicho explícito) hasta integrar los datos ([[2026-10-26 Coordinación Finanzas]]). Prioridades de FP&A: revenue de Reservo → cash flow → P&L y márgenes; margen por cliente y waterfall quedan después ([[2026-10-27 FP&A Weekly]]).
- **Cash flow**: el FCF excluye Loan y ESOP (solo operación + inversión). Working Capital ProPay va en línea aparte. La conciliación Evidence vs Odoo se hace país por país, partiendo por Chile ([[2026-10-27 Cash flow en Odoo — FCF, estructura y Payments (Rafa)]], [[2026-10-26 Conciliación del cash flow por país, partiendo por Chile (Rafa)]]).
- **POS**: hay cuatro tipos oficiales (venta, renta, cuotas, regalado) y los cupones y notas de crédito se centralizan ([[2026-10-07 Definición venta y renta POS en Chargebee]]). Las reactivaciones quedan fuera del payback y del attachment: hay consenso, falta la decisión formal ([[2026-10-05 Payback y migración a tablas POS]]). El board book ya corre sobre `pos_sales_sot` (PRs 832, 856 y 860, mergeados el 28-oct).
- **Margen por cliente**: el costo de modelos IA sale solo del P&L y Contabilidad reclasifica hacia atrás ([[2026-10-20 FP&A Weekly]]; PR 407 sigue abierto).
- **Activación y velocity**: dejan fuera los bookings importados de otro software (PR 414 y PR 415, mergeado el 19-oct).

## Tareas cerradas (1)
T-077 Enviar a Mariana el cashflow corregido. Descartadas (5): T-070, T-083, T-084, T-086, T-087. Se crearon 30 tareas en el mes, de las que 15 siguen en `propuesta`.

## Proyectos que avanzaron
- **SoT Ventas POS**: 2 análisis y board book migrado. Siguen T-097 Payback Payments y la migración del resto de Evidence (propuesta T-108).
- **P&L y Centros de Costo**: 3 análisis. Abiertos el P&L de Reservo en Odoo y los dos mappings (1.0 y 2.0) de la reestructuración.
- **Activación**: 2 análisis y PR 414/415 en producción. T-089 «Arreglar definición de activación» no se toca desde el 7-oct.
- **Márgenes por Cliente y Producto**: reunión de márgenes objetivo por negocio (28-oct); PR 407 abierto.
- **Churn y Graduación**: ajustes pedidos por CX; churn voluntario de octubre al alza en Colombia (propuesta T-103).
- Sin movimiento: **Vertical Fitness** (T-048 y T-049 desde el 5-ago) y **MDR AE Change** (T-064 desde el 23-sep). **Presupuesto 2027** sigue sin partir: está en propuesta T-102, «Recordarle a Pablo Lucero retomar el proceso de presupuesto».

## Pendientes que se repiten
- **Cash flow**: tema de 10 reuniones de octubre y de varias notas de Slack. T-078 sigue en curso con fecha vencida (29-sep), y T-104 a T-107 esperan aprobación. El tag `cashflow`/`cash-flow` se propuso en septiembre y otra vez el 30-oct.
- **Reservo**: integrar el revenue (T-111, «Prioridad #1»), el headcount (T-112), el P&L y las definiciones comunes. Todo está en propuesta o recién empieza.
- **Propuestas sin decidir**: hay 15. Las más antiguas son T-092 y T-093 (6-oct) y T-094 (7-oct), y ningún hilo del Monitor tiene respuesta.
- **En curso sin tocar hace más de 14 días**: T-021, T-048, T-049, T-050, T-057 (Reminders a Sofía), T-064, T-066, T-085 y T-089.
- **Definiciones**: hay 3 notas Deriva abiertas (Activation, Churn y Activation Velocity, nueva) y la skill tampoco recoge esos cambios.

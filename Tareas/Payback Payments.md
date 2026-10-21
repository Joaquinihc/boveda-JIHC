---
type: tarea
estado: en-curso
prioridad: alta
tiempo:
tipo: analisis
origen: notion
proyecto:
temas:
  - payments
  - pos
notion_id: 3ffab34c233b801ea61ccc9c31579316
notion_estado: In progress
notion_prioridad: High
notion_actualizado: 2026-10-20T15:08Z
fuente: https://app.notion.com/3ffab34c233b801ea61ccc9c31579316
fecha_limite:
creado: 2026-10-20
revisar: false
id: T-097
---

# Payback Payments

Objetivo: rehacer el cálculo de payback de Payments y desafiar sus supuestos (el cálculo de referencia está en una página enlazada desde la fila de Notion). Alcance acordado con Pablo Lucero (weekly 20-oct-2026): el payback de **pagos como venta de POS a la base** (flujo KAM, clientes del stock), separado del payback de adquisición (flujo ISR, que ya está dentro del payback mensual de Evidence) y del de IA ([[Payback AI]]).

## Subtareas
- [ ] 1. Corregir el término Payments del payback mensual en Evidence: `pos_sold_isr` en la CTE `pos` de `payback_base`, nota de metodología («New_POS = POS sold in the month (ISR + KAM)») y correr `npm run test:sql` (`scripts/test/payback-window.test.mjs`). Listo cuando el PR esté abierto y Pablo lo apruebe.
- [ ] 2. Validar con datos los supuestos de la planilla de Ignacio: % de cierres que compran terminal (hoy 67%), costo POS + delivery + POP por país, activación y GPV por cohorte.
- [ ] 3. Armar el cálculo por país (CL y MX) con las dos versiones de CAC (con y sin sueldos KAM) y la comparación payback estimado al inicio vs real por cohorte mes a mes.

## Notas

### 2026-10-21 · Cálculo actual en Evidence y corrección de Pablo
- **Dónde**: `pages/acquisition/payback/index.md` (Payback — Monthly, https://evidence.agendaprops.com/acquisition/payback/). Cómo está implementado, fuentes y toggles: [[Payback mensual en Evidence — implementación y fuentes]]. Definición canónica: [[Payback (Mensual)]].
- **Fórmula**: Payback = (costos de adquisición directos + HQ prorrateado + descuento ventana [−1,+2]) ÷ (New MRR SaaS bruto × GM SaaS + **New POS × ASP Payments × GM Payments**). El término Payments es el único aporte de pagos al payback y es de **adquisición** (cohorte de merchants nuevos B2B3), no del flujo KAM.
- **Corrección que levanta Pablo** (DM del 21-oct, 00:23; [mensaje](https://agendapro.slack.com/archives/D095G9DL97T/p1792553028207479)): la CTE `pos` hace `SUM(pos_sold)` sobre `pos_sold_monthly`, y `pos_sold` es **ISR + KAM**. Los KAM son POS vendidos a la base, pero el payback se los asigna completos a la cohorte del mes: infla el margen por merchant nuevo y achica el payback.
  - Cuánto pesa (Chile, jul–sep 2026, cifras de Pablo): 187 POS = 106 ISR + 81 KAM. Solo con ISR, el GM por merchant nuevo baja de ~USD 113 a ~USD 89/mes y el Payback Variable L3M de Chile sube de **5,5 a ~6,9 meses**. México también vende POS; Argentina y Colombia no tienen.
  - Fix: usar la columna `pos_sold_isr`, que la extracción ya trae (verificado en `sources/dwh/payments_weekly/pos_sold_monthly.sql`), y corregir la nota de metodología. Referencia de cómo separar bien ISR y KAM: página `budget-2026-follow-up`.
  - Coordinación: Pablo pedía hacerlo después de sus PRs #837 (toggle Monthly / Trailing 3m) y #844 (Variable Payback por componente). Al 21-oct **ambos están mergeados** en `main`, así que el fix va directo sobre `main`.
- Contexto de la semana: el 19-oct Pablo ya había avisado en el DM «OJO con el cálculo del payback de payments», tras revisar la migración del board book a `pos_sales_sot` (PR #832, abierto) descrita en [[20261006-01 Board book sobre pos_sales_sot — impacto en ventas, attachment y payback]].

### 2026-10-20 · Reunión con Pablo: planilla de Ignacio (Payback POS flujo KAM)
Fuente: transcripción «Pablo - Joaquín» del 20-oct en Notion (https://app.notion.com/p/3ffab34c233b81f9be74c327980e8376). Pablo dejó la estructura en la página [Payback POS - flujo KAM](https://app.notion.com/p/3ffab34c233b8026a8efe40482c3ae07) (FP&A → Definiciones 2025), enlazada desde la tarjeta de Notion de esta tarea.
- **Qué se revisó**: la planilla de [[Ignacio Embry]] para el payback de POS vendidos por KAM. Él la calcula como cash flow (K = costo del hardware menos la venta). Pablo encontró «pifias» de partida: la planilla está hecha para un proveedor (OL), no por país, y los supuestos no están validados.
- **Estructura acordada**:
  - CAC = costo POS + delivery + material POP (cajas, stickers, rollos).
  - CAC fully loaded = CAC + sueldos KAM + onboarders. **Ops no entra en el CAC: entra en el margen.** Los sueldos están en otra cuenta (la planilla hoy pone la comisión del KAM).
  - Payback = CAC (cualquiera de los dos) ÷ (margen POS % × GPV POS × activación + arriendo y venta del cohort).
  - A diferencia del payback de adquisición, acá **sí se multiplica por activación**, porque es mucho más baja.
  - El attachment rate no impacta acá: ya está incluido en el flujo ISR del payback normal.
- **Supuestos a validar contra real**: activación, costo POS (delivery, merma, cajas, rollos; «si Ignacio pone USD 5 de delivery, veamos cuánto es y cómo varía mes a mes»), arriendo y venta del cohort, GPV. Supuesto actual: 67% de los cierres compran la terminal.
- **Metodología de seguimiento** (como siempre): payback estimado al inicio con supuestos vs payback real de la cohorte mes a mes, cuánto aporta cada mes vs la estimación original.
- **Rehacer por país**, no por proveedor.
- Pablo sospecha doble conteo entre la venta/arriendo y el margen de pagos en la planilla; hay que revisarlo.

### Pendientes menores
- 2026-10-20 · Decisión de Pablo: si el payback POS se calcula con o sin sueldos KAM (y onboarders). Propuesta en la reunión: mostrar las dos versiones.
- 2026-10-20 · Confirmar si el costo de la máquina está incluido en el revenue total de la planilla de Ignacio (Pablo cree que sí; falta como ítem en la página de Notion).
- 2026-10-20 · Revisar el posible doble conteo de arriendo y venta del cohort con el margen de pagos.
- 2026-10-20 · Pablo iba a compartir las notas de la reunión (quedaron en la transcripción de Notion).
- 2026-10-21 · `pos_sold_monthly` solo trae datos desde 2026-01: el término Payments del payback de Evidence vale 0 antes de 2026 (observación del PR #844).
- 2026-10-21 · Revisar si el payback de Key Metrics del board book (PR #832, lee `pos_sales_sot` con «todo menos reactivación») arrastra el mismo problema: ahí entran New (ISR), Stock (KAM) y Others.

## Historial
- 2026-10-21 · Importada desde Notion (In progress)
- 2026-10-21 · Avance desde Claude Code: contexto del cálculo actual en Evidence, corrección de Pablo (ISR + KAM) y reunión del 20-oct sobre la planilla de Ignacio; 3 subtareas y 6 pendientes menores; tema `pos` agregado.

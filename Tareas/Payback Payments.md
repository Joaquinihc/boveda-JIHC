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

### 2026-10-22 · Fix ISR listo para PR y supuestos del payback POS KAM validados con datos
**Subtarea 1 (fix en Evidence)** — hecho en worktree `agendapro-worktrees/payback-pos-isr`, rama `fix/payback-pos-isr-only` sobre `main` (db7a0be7, ya incluye #837 y #844). Sin commit ni PR todavía: Joaquín revisa el diff primero.
- Cambios: CTE `pos` de `payback_base` pasa de `SUM(pos_sold)` a `SUM(pos_sold_isr)`; la nota de metodología dice ahora que New_POS = POS vendidos a merchants nuevos (canal ISR) y que los KAM van a la base; test nuevo en `scripts/test/payback-window.test.mjs` que corre la CTE `pos` sobre un fixture `pos_sold_monthly` con filas ISR y KAM (falló antes del fix, pasa después). `npm run test:sql`: 148/148.
- Verificación en DWH (corte 22-oct, lógica de `pos_sold_monthly.sql` sobre `dwh.pos_sales2`): Chile jul–sep 2026 = 187 POS = 106 ISR + 81 KAM, igual a las cifras de Pablo. México jul–sep: 60 = 28 ISR + 32 KAM.

**Escenarios de New POS (duda de Joaquín: ¿contar los KAM a merchants nuevos por la regla de la semana?)** — corte DWH 22-oct, mismo SQL de la página sobre caché reextraído. Artifact para Pablo: https://claude.ai/artifact/MkNHu2N99XLQnrGMfKcHJU (privado hasta compartirlo). Chile jul–sep: solo 3 POS KAM a merchants ≤ 30 días (México 1); 9 de 10 KAM van a merchants > 90 días. Payback T3M jul–sep (actual → solo ISR → ISR + KAM ≤ 30 d): Chile variable 5,5 → 6,9 → 6,8; B2B3 total 14,3 → 15,8 → 15,7; México total 17,6 → 18,8 → 18,7. Conclusión: el tramo en duda mueve ≤ 0,1 mes; se propone dejar el fix solo ISR. `pos_sold_monthly` ya clasifica como ISR a los agentes no KAM ≤ 30 días desde el alta. Caveat México: 31 de 88 KAM ene–sep sin fecha de addon/alta, quedan como KAM. Script y datos en `scratchpad/payback-scen/` de la sesión.

**Planilla de referencia** — es el Google Sheet «Lifetime Value - CAC - IRR Merchant params 2026 POS» (dueño Julio, compartido el 20-oct; https://docs.google.com/spreadsheets/d/1lUNdH1QrL0V3EYtGXYqrhdvZ3yjpbIaXanLxMY3bFHY). Hojas: `POS_2026_KAM_new_grid` (Chile, Jaime), `POS_2026_MDR/AE_new_grid` y `temp_ISR_2026_MX_OEL` (México/Janet/OEL, armada el 4-sep-2026; es la «de un proveedor» que vio Pablo).
- Cómo calcula la hoja KAM: CAC USD 337 = hardware 108 − venta 60 + delivery 10 + merma 5 + KAM fijo 150 + comisión KAM 100 + onboarding 23 (ya es «fully loaded»). Revenue = ARPA 113 (GMV 12k × 54 % GPV/GMV × fee 1,75 %) × GM 35 % × activación (85 % → 75 % → −0,5 %/mes). Payback 10 meses.
- **Doble conteo (duda de Pablo)**: en la planilla la venta de la terminal resta dentro del CAC y el arriendo no está en el revenue, así que ahí no hay doble conteo. Lo habría si se suma «arriendo y venta» al denominador y además se usa el GM Payments del P&L: Payments Revenue ya incluye los equipos (en `dwh.payments_revenue`, `fee_usd = fee_pos + fee_web + pos_devices`). Regla propuesta: denominador = margen de procesamiento (`fee_pos − cost_pos`) + arriendo/venta (`pos_devices_usd`), y la venta sale del CAC.
- **Costo de la máquina en el revenue total** (duda de Pablo): no está en el revenue de la planilla; está como «Sales Hardware −60» dentro del CAC.

**Supuestos vs datos** (corte DWH 22-oct-2026; P&L real hasta sep-2026; SQL en `scratchpad/payback-kam/sql/` de la sesión, pendiente de llevar a repo):

| Supuesto | Planilla | Chile medido | México medido | Fuente |
|---|---|---|---|---|
| % cierres que compran la terminal | 67 % | **~5 %**: 8 `setup-smart-pos` de ~146; 139 con `smart-pos-arriendo` | **63 %**: Chargebee 51 compra / 81; HubSpot 39 de 63 deals con monto | `pos_all_commissions`, `stg_hubspot__deals` (pipeline 82319002, Ganado) |
| Precio de venta / arriendo | venta neta USD 60 | venta CLP 59.900; arriendo CLP 21–23k/mes (USD 22–27), lo paga ~60 % de la cohorte | venta MXN 1.732–2.500 (≈USD 100–145); renta MXN 270–316/mes (USD ~16) | `int_pos_terminals_monthly`, `payments_revenue.pos_devices_usd` |
| Costo hardware | USD 108 (CL) / MXN 2.832 (MX) | **no hay compra en el P&L**: G0012 Transbank USD 60,9k ene–sep ≈ USD 34 por terminal arrendado-mes, más que el arriendo cobrado (22–27). Hipótesis: arriendo mensual a Transbank → el hardware es margen, no CAC | G0012 OEL USD 5.462 jun–sep ≈ USD 64/unidad vendida; antes de jun no hay línea | `finance.profit_and_loss` |
| Delivery | USD 10 (CL) / 5 (MX) | USD 6,2/unidad (Chilexpress 2,7k ÷ 433 unidades todos los canales) | **USD 33/unidad** (DHL 5,3k ÷ 159) | P&L G0012/G0016/G0013 |
| POP (cajas, stickers, rollos) | merma 5 | USD 7,9/unidad (Aloprint G1113 + Cavara + Boxhosting 3,4k) | USD 6,3/unidad (papelería + Ideas de Alto Impacto) | P&L |
| Activación | 85 % m1 → 75 % | 75 % m1; ~60 % m2–m6 (GPV POS > 0) | 84 % m1; ~73 % m2–m6 | `location_sales_months` |
| GPV por comercio activo | USD 6,5k | 6,3k m1 → 12,9k m2 → 14–20k m4–m6 | 2,3k m1 → 3,4–3,9k m2–m4 | idem |
| Margen de procesamiento | 35 % del fee (0,61 % del GPV) | 30–42 % del fee (0,6–1,0 % del GPV) | 12–18 % del fee (0,30–0,49 % del GPV) | `fee_pos − cost_pos` |
| Sueldos KAM | USD 150 fijo + 100 comisión (10 POS/KAM/mes) | G1125 Payroll Payments Expansion USD 13,4k/mes (jul–sep) ÷ 28,7 unidades/mes = **USD 466/unidad** (5 vendedores en Q3: jaime, rocio, alvaro, valentina, ignacia; 6–9 u/KAM/mes) | G1125 USD 2,2k (sep, Janet, incl. comisiones 402) ÷ 14 u/mes = USD 156/unidad | P&L, `pos_sales_sot` |
| Onboarders | USD 23 | sin cuenta propia (G1200 es onboarding SaaS); uso USD 16 por POS activado ≈ 10/unidad | 0 (sin onboarder) | planilla |
| Ops | fuera del CAC | G1124 Payroll Payments Ops USD 7,4–7,5k/mes (va al margen) | — | P&L |

Nota: hasta jun-2026 el payroll KAM estaba mezclado en G1110 «Remuneraciones Payments» (Chile + HQ) y G0105 Expansion (con los KAM de IA); desde jul el P&L separa G1125 (Payments Expansion), G1124 (Ops) y G1121–G1123 (verticales IA). El P&L trae filas de presupuesto desde oct-2026 con `type = 'Real'`: filtrar por fecha.

**Resultado del modelo** (cohortes KAM 2026 válidas con unidades, grano compañía; escenario A = hardware CL como costo mensual Transbank y MX al costo del P&L; B = hardware upfront de la planilla):
- **Chile**: contribución estable USD 60–80 por comercio/mes (procesamiento 65 + arriendo 15 − Transbank 21). CAC base USD 14 (A) / 119 (B); fully loaded USD 490 / 595. Payback estimado: base < 2 meses; **fully loaded 7–8 meses**. Real: las cohortes de ene y feb recuperaron el CAC fully loaded en m3–m4; mar y abr llevan 46–47 % a los 6–7 meses.
- **México**: contribución estable USD 16 por comercio/mes (procesamiento 9 + renta/venta 7). CAC base 22 (A) / 124 (B); fully loaded 178 / 280. Payback estimado: base 1–8 meses; **fully loaded 11–18 meses**. Real: mar recuperó 103 % a los 7 meses; may–jul van en 64–83 % a los 3–5 meses (la venta de la terminal en m0 adelanta la recuperación).
- Lectura: el payback de POS a la base lo manda el payroll KAM, no el hardware; el CAC «base» de Pablo es casi cero en Chile porque casi no hay compra de terminal.

**Decisiones de método que tomé** (corregibles): cohorte por `effective_sale_date` de la primera venta KAM válida con unidades (incluye reactivaciones con compra y POS adicional); activación = GPV POS > 0 en el mes; delivery y POP prorrateados por unidades de todos los canales; onboarding con el supuesto de la planilla.

### Pendientes menores
- 2026-10-22 · Confirmar con Finanzas (Nicolás/Rafael) qué es G0012 Transbank (arriendo mensual vs compra de terminales) y si alguna terminal se capitaliza; con la hipótesis actual el arriendo deja margen negativo (cobra USD 22–27, paga USD 34).
- 2026-10-22 · Costo real de la terminal OEL en México (planilla asume = precio de venta MXN 2.832; P&L muestra USD 64/unidad desde jun).
- 2026-10-22 · Qué personas incluye G1125 Payments Expansion (¿solo KAM POS? ¿comisiones en «asignaciones»?) y dónde van los onboarders POS (¿G1124 Ops?).
- 2026-10-22 · Delivery México USD 33/unidad vs 5 supuesto: ver qué más viaja por DHL.
- 2026-10-22 · Dónde dejar el SQL del modelo en repo (¿página Evidence en `fintech/`?): decisión de Joaquín.
- 2026-10-22 · Resueltos hoy: costo de la máquina (está en el CAC de la planilla, no en el revenue) y doble conteo (ver regla arriba); quedan para que Pablo los confirme.
- 2026-10-20 · Decisión de Pablo: si el payback POS se calcula con o sin sueldos KAM (y onboarders). Propuesta en la reunión: mostrar las dos versiones.
- 2026-10-20 · Confirmar si el costo de la máquina está incluido en el revenue total de la planilla de Ignacio (Pablo cree que sí; falta como ítem en la página de Notion).
- 2026-10-20 · Revisar el posible doble conteo de arriendo y venta del cohort con el margen de pagos.
- 2026-10-20 · Pablo iba a compartir las notas de la reunión (quedaron en la transcripción de Notion).
- 2026-10-21 · `pos_sold_monthly` solo trae datos desde 2026-01: el término Payments del payback de Evidence vale 0 antes de 2026 (observación del PR #844).
- 2026-10-21 · Revisar si el payback de Key Metrics del board book (PR #832, lee `pos_sales_sot` con «todo menos reactivación») arrastra el mismo problema: ahí entran New (ISR), Stock (KAM) y Others.

## Historial
- 2026-10-21 · Importada desde Notion (In progress)
- 2026-10-21 · Avance desde Claude Code: contexto del cálculo actual en Evidence, corrección de Pablo (ISR + KAM) y reunión del 20-oct sobre la planilla de Ignacio; 3 subtareas y 6 pendientes menores; tema `pos` agregado.
- 2026-10-22 · Avance desde Claude Code: fix ISR hecho en rama `fix/payback-pos-isr-only` con test (148/148), sin PR aún; planilla identificada; supuestos validados contra DWH y P&L; modelo por país con CAC con y sin sueldos; 5 pendientes menores nuevos.

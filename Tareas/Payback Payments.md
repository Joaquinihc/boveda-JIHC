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
notion_actualizado: 2026-10-27T13:36Z
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
**Subtarea 1 (fix en Evidence)** — hecho en worktree `agendapro-worktrees/payback-pos-isr`, rama `fix/payback-pos-isr-only` sobre `main` (db7a0be7, ya incluye #837 y #844). **PR #856 abierto el 22-oct** (https://github.com/agendapro/agendapro-dashboards-evidence/pull/856), reviewers Pablo Lucero (`plucero1`) y Nicolás Cofré (`nicofrem-agpro`; la cuenta `nicofrem` no es colaboradora del repo). Aviso a Pablo por DM el 22-oct con PR, artifact e impacto: https://agendapro.slack.com/archives/D095G9DL97T/p1792683241617689. Falta su aprobación para cerrar la subtarea.
- Cambios: CTE `pos` de `payback_base` pasa de `SUM(pos_sold)` a `SUM(pos_sold_isr)`; la nota de metodología dice ahora que New_POS = POS vendidos a merchants nuevos (canal ISR) y que los KAM van a la base; test nuevo en `scripts/test/payback-window.test.mjs` que corre la CTE `pos` sobre un fixture `pos_sold_monthly` con filas ISR y KAM (falló antes del fix, pasa después). `npm run test:sql`: 148/148.
- Verificación en DWH (corte 22-oct, lógica de `pos_sold_monthly.sql` sobre `dwh.pos_sales2`): Chile jul–sep 2026 = 187 POS = 106 ISR + 81 KAM, igual a las cifras de Pablo. México jul–sep: 60 = 28 ISR + 32 KAM.

**Escenarios de New POS (duda de Joaquín: ¿contar los KAM a merchants nuevos por la regla de la semana?)** — corte DWH 22-oct, mismo SQL de la página sobre caché reextraído. Artifact para Pablo: https://claude.ai/artifact/MkNHu2N99XLQnrGMfKcHJU (privado hasta compartirlo). Chile jul–sep: solo 3 POS KAM a merchants ≤ 30 días (México 1); 9 de 10 KAM van a merchants > 90 días. Payback T3M jul–sep (actual → solo ISR → ISR + KAM ≤ 30 d): Chile variable 5,5 → 6,9 → 6,8; B2B3 total 14,3 → 15,8 → 15,7; México total 17,6 → 18,8 → 18,7. Conclusión: el tramo en duda mueve ≤ 0,1 mes; se propone dejar el fix solo ISR. `pos_sold_monthly` ya clasifica como ISR a los agentes no KAM ≤ 30 días desde el alta. Caveat México: 31 de 88 KAM ene–sep sin fecha de addon/alta, quedan como KAM. Script y datos en `scratchpad/payback-scen/` de la sesión.

**Dónde se habló del payback en #comité-adquisición** (revisado 22-oct): no hay un hilo propio sobre ISR vs KAM; eso solo está en el DM de Pablo del 21-oct. El payback aparece dentro del hilo de Agustín del 16-oct sobre el competidor barato en Argentina (plan 3–5 profesionales a ARS 20.000): https://agendapro.slack.com/archives/C03QTERM2TA/p1792178193089939. Ahí Pablo dijo el 19-oct que el payback total B2B3 está en 20 meses y el variable en 10 (Argentina pesa por ticket bajo con costos corporativos prorrateados por merchants B2B3), linkeó la página y el Notion de referencia, y Julio pidió el L3M (origen del PR #837). Carlos Rojas dejó un modelo de comisiones (planilla) con payback 14,5 vs 11 meses. Relacionado: hilo del 15-oct de descuentos ISR por país, donde Chile ofrece «POS sin cargo» en comodato (afecta el arriendo del cohort en el frente 2).

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

### 2026-10-23 · Preguntas para destrabar los costos (las manda Joaquín)
Decisión del 23-oct: el cálculo del payback POS KAM se arma primero como artifact HTML con supuestos editables, para cerrar la metodología con Pablo antes de invertir en una página Evidence. **Artifact publicado el 23-oct** (privado hasta compartirlo): https://claude.ai/artifact/MXK4y1zZwTygVJ2fp7ct6h — metodología, supuestos planilla vs medido editables por país, escenario A/B de hardware, CAC base y fully loaded, payback estimado con curva de ramp-up vs real por cohorte (gráfico y tabla), lectura y abiertos. Con los valores medidos: Chile CAC 14 / 490 USD por POS, contribución 51 USD/mes, payback fully loaded ~8 meses (9,6 en estado estable); México CAC 22 / 178, contribución 15 USD/mes, payback fully loaded ~8 meses con la venta de la terminal en m0 (11,6 en estado estable). Fuente del modelo: `scratchpad/payback-kam/` (cohorts.json, model.py, sql/). Las preguntas no bloquean el modelo (va con escenarios); cierran los supuestos de costo.

**Para Finanzas (Nicolás / Rafael)**
1. La línea del P&L `5.0.01.10 G0012 - POS y MPOS (ProPay) - TRANSBANK S.A.` (Chile, USD 5,8k en ene-2026 → 7,9k en sep-2026, Payment Processing) ¿qué es? ¿Arriendo mensual de los SmartPOS que AgendaPro re-arrienda a los comercios, compra de terminales en cuotas, u otra cosa? Dividida por los comercios con arriendo vigente da USD 32–37 por terminal al mes, más que los CLP 21–23k (USD 22–27) que se cobran de arriendo. ¿Hay terminales capitalizadas en el balance (activo fijo) que no pasen por el P&L?
2. México: ¿cuál es el costo de la terminal OEL por unidad y dónde se registra? En el P&L solo aparece `501.08.003 G0012 - POS y MPOS (ProPay) - OPERACIONES EN LINEA` desde jun-2026 (USD 1,35–1,38k/mes ≈ USD 64 por unidad vendida); la planilla asume MXN 2.832 (≈USD 166) y antes de junio no hay línea.
3. ¿Quiénes están en `G1125 - Payroll Payments Expansion` (Chile USD 13,4k/mes jul–sep; México USD 2,2k en sep)? ¿Solo los KAM de POS? ¿Las comisiones van dentro de «asignaciones»? ¿Y los onboarders de POS están en `G1124 Payments Ops` o en el onboarding de SaaS (G1200)?

**Para Payments (Ignacio)**
4. Delivery México: DHL (`G0013`) suma USD 5,3k ene–sep 2026, USD 33 por unidad vendida, contra USD 5 supuestos en la planilla. ¿Qué más viaja por DHL además de terminales (devoluciones, POP, reposiciones)?
5. Chile: ¿la promo «POS sin cargo» en comodato para cierres ISR (hilo del 15-oct en #comité-adquisición) aplica también a ventas KAM? Afecta el arriendo del cohort en el modelo.

### 2026-10-23 · Qué revenue de pagos entra al término Payments (pregunta de Joaquín)
- **Hoy entra todo**: `mrr_payments` = `SUM(fee_usd)` de `dwh.payments_revenue`, y `fee_usd = fee_pos + fee_web + pos_devices` (definición canónica de Payments Revenue). ASP Payments = ese total ÷ locations con GPV POS > 0 (`pos_penetration_merchants.merchants_pos`). GM Payments = % global del P&L, que también mezcla web y POS.
- **Desajuste**: el numerador del ASP incluye los fees de pago en línea de merchants que no tienen POS (Chile sep-2026: 689 compañías B2B3 con web y sin POS vs 557 con POS), pero el denominador son solo locations con POS. Infla el ASP: Chile jul–sep USD 236–254 (actual) vs 178–190 solo POS + equipos (−25 %); México 183–194 vs 132–143 (−27 %). Corte DWH 23-oct.
- **Margen**: el GM web es más alto que el POS (Chile jul–sep: 48,5 % web vs 37,1 % POS; México 21,8 % vs 16,7 %, `location_sales_months`), así que el % global también sobreestima el margen del POS nuevo.
- **Opciones**: (a) ASP solo POS = (fee_pos + pos_devices) ÷ merchants con POS, con GM POS por país; (b) mantener todo el revenue pero dividir por merchants que pagan (propay) y multiplicar por merchants nuevos en vez de New POS (cambia el constructo). Sin decidir; conversarlo con Pablo antes de tocar la página. Si va (a), es un PR aparte del #856.

- 2026-10-23 · Reporte reescrito (misma URL https://claude.ai/artifact/MkNHu2N99XLQnrGMfKcHJU) para el comité de adquisición: se sacaron los escenarios ≤30/≤90 d; queda solo actual → solo ISR, y un capítulo 4 con la propuesta de dos términos Payments (POS y pago en línea, cada uno con attachment, ARPA y GM propios). Decisión de Joaquín: por ahora el revenue de pagos sigue entrando completo (el POS es 67–72 % del stream). Datos nuevos: cohortes B2B3 ene–jun 2026, Chile 687 nuevos, 32 % cobran en línea en 3 meses vs 11 % con POS, activos m2 20 % / 9 %, ARPA m2 USD 30 / 87; México 664, 23 % / 4 %, 14 % / 4 %, USD 8 / 69 (corte 23-oct). Pendiente: Joaquín revisa el reporte y manda el mensaje al hilo del comité (propuesta de texto en la sesión del 23-oct).

### 2026-10-23 · Migración de ventas POS a pos_sales_sot (decisión de Joaquín) y paso 2 hecho
**Por qué el payback no tenía Payments antes de 2026**: no es la tabla; `dwh.pos_sales2` tiene ventas desde abr-2023 (172 / 410 / 800 u en 2023–25). El corte `>= 2026-01-01` está en la extracción `pos_sold_monthly.sql`, nacida para el Payments Weekly.
**Respaldo de cada fuente**: regla legacy (dedup empresa+día, roster KAM de 6, ventana 30/90 d) acordada con Miguel Cárdenas en jul-2026, operativa, sin Notion. SoT de Ignacio: reunión 7-sep (Pablo, Miguel, Ignacio), reunión 5-oct (acuerdo de migrar el payback a la tabla nueva con compra válida; reactivaciones fuera), definición en Notion. No hay decisión escrita de Pablo de migrar todo Evidence; PR #832 (board book) en borrador desde el 19-oct.
**Decisiones de Joaquín (23-oct)**: migrar todas las consultas de ventas POS de Evidence a `pos_sales_sot` y dejar `pos_sales`/`pos_sales2`; Others no cuenta en payback ni attachment; término Payments desde 2024-01 (propuesta mía, por confirmar); eliminar el MBR (versión legacy del board book); OKR y CEO dashboard pasan a la tabla nueva aunque cambien; el roster KAM/ISR en dbt se lo plantea él a Ignacio.
**Paso 2 hecho**: worktree `agendapro-worktrees/payback-pos-sot`, rama `feat/pos-sold-monthly-on-pos-sales-sot` sobre `main` (41b40f1d), 1 commit, sin push ni PR. `pos_sold_monthly.sql` sobre `pos_sales_sot` (válidas, sin reactivación, `effective_sale_date` ≥ 2024-01-01, canal por deal, columna nueva `pos_sold_others`, mismas columnas previas). Test nuevo `scripts/test/pos-sold-monthly-sot.test.mjs`; suite 194/194. Consumidores de la extracción: payback, payments-sales (su «total» pasa a incluir Others), budget-2026-follow-up (lee ISR + KAM, sin cambio de lógica).
**Impacto (caché 22/23-oct, FX current, mkt excl., descuento ventana)**: unidades ISR 2026 casi iguales en Chile (legacy 36 → SoT 38 en sep; jul–sep 106 → 110) y menores en México (13 → 8 en sep; jul–sep 28 → 14: la SoT manda a Others ventas sin deal que parecen ISR, y Others queda fuera por decisión). Payback T3M sep-2026, #856 → SoT: B2B3 total 15,8 → 15,9; Chile 10,6 → 10,5; México 18,8 → 19,4. Historia: 2024–2025 pasan de Payments = 0 a tener término; B2B3 total jun-2025 mensual 14,5 → 12,7 (Chile 12,8 → 10,1), dic-2024 18,9 → 16,7. Caveat: el mes en curso llega tarde en la SoT (oct-2026 Chile ISR 0 vs 9 legacy) porque la venta se valida al pagarse la factura y completarse el formulario. Script y JSON en `scratchpad/payback-sot/` de la sesión del 23-oct.

### 2026-10-23 · Plan de migración aprobado y pasos 2 y 3 ejecutados
- **Plan acordado con Joaquín**: (1) mergear #856 tal cual; (2) PR de `pos_sold_monthly` sobre la SoT; (3) rebasar #832 y sacar Others del payback; (4) `country_pos` y sus 6 consumidores + borrar MBR + numeradores de attachment; (5) Payments Weekly (8 extracciones, avisar a Miguel); (6) cohortes de activación y retención. Un PR por grupo por el riesgo de grano (CLAUDE.md del repo).
- **Paso 2 → PR #860** (https://github.com/agendapro/agendapro-dashboards-evidence/pull/860), reviewers Pablo y Nicolás.
- **Paso 3 → PR #832** actualizado con un merge de `main` (estaba 201 commits detrás; el hook del repo bloquea force-push, así que no se rebasó; conflictos en `country_pos_sot.sql` y las dos páginas del board book, resueltos manteniendo el `act` canónico de main), fuera de borrador, reviewers Pablo y Nicolás. Cambios: término Payments del payback = solo Inbound (ISR); Key Metrics New POS = Inbound + Stock (universo del plan); Others solo en slide 8; test nuevo `board-book-pos-sot.test.mjs` para las dos ediciones; suite 197/197. Worktree limpio `agendapro-worktrees/board-book-sot` (el viejo `pos-sales-sot-dbt` quedó en estado raro, sin HEAD; no usarlo).
- **Impacto board book** (`cty_payback`, sep-2026, caché 15/23-oct), antes = Inbound + Stock legacy → después = Inbound SoT: Chile total 6,9 → 8,6; México total 16,2 → 17,4; variable Chile 4,5 → 5,6, México 12,1 → 13,1. Julio se mueve más (Chile 8,5 → 12,1). Casi todo es sacar Stock; el cambio de fuente mueve 0,0–0,2 meses. POS en el término jul–sep: CL 201 → 110, MX 62 → 14.
- Nicolás ya aprobó #856; falta Pablo.

### 2026-10-25 · Revisión en Odoo de las cuentas de costo del modelo (líneas analíticas 2026, plan «Centros»)
Fuente: `account.analytic.line` ene–sep 2026 vía la librería de `mcp-odoo-agendapro` (solo lectura), analíticas G0012, G0013, G0016, G1112, G1113, G2005, G3102, G0105, G1110, G1124, G1125; USD con FX mensual del DWH. Datos en `scratchpad/odoo-check/` de la sesión. Lo que cambia respecto a la nota del 22-oct:
- **G0012 Transbank = «ARRIENDO POS PSP»**, factura mensual de CLP 5,1M (dic-25) a 7,5M (jul-26). Confirmado: es arriendo, no compra. Con Haulmer (CLP 599k fijos ene–may, facturas 916472/916340), Cavara (CLP 378k fijos ene–jun, «Otros Gastos») y Boxhosting (CLP 175k ago–sep, «Pos -FAC») suma USD 67,0k ene–sep = USD 37,7 por comercio-mes en arriendo (solo Transbank 34,0). Las tres cuotas fijas parecen compras de terminales en cuotas: queda como pregunta a Finanzas.
- **Delivery Chile = USD 12,5 por POS** (antes 6,2): Chilexpress «encomiendas nacionales/local» en G0012 (USD 2,7k) + honorarios de traslado de equipos en G0016 (José Jacinto Hernández y Bastián Cáceres, USD 2,7k). El G0016 del P&L del DWH («Transporte POS - Chilexpress», USD 318) no coincide con Odoo: el master de Finanzas remapea.
- **Delivery México = USD 16,5 por envío, 1,7 envíos por POS**: DHL (G0013) USD 4,9k ene–sep = USD 31 por POS vendido, pero son 208 envíos domésticos abr–sep (MXN 287 c/u) para 121 POS. El envío no es cinco veces Chile; DHL despacha más veces por POS (reposiciones, devoluciones o rollos: pregunta a Ignacio).
- **G0013 México también trae UNOS Estrategia IOT** (SIM y datos de las terminales): MXN 23k/mes abr–sep = USD 7 por terminal OEL activa al mes. Es costo recurrente de operación, no CAC; agregado al margen de México en el modelo (contribución baja de 15 a ~10 USD/POS/mes).
- **Hardware México**: OEL factura MXN 23.600 fijos al mes por «TERMINAL UROVO i9100» desde abril (USD 67,6 por POS vendido abr–sep). Parece lote financiado o arriendo, no compra por unidad: pregunta a Finanzas.
- **POP**: Chile USD 5,3 por POS (cajas Elefont en G0016 USD 1,6k + Aloprint G1113 USD 0,7k; dejé fuera etiquetas, sintra y sobres de Aloprint en G2005, USD 1,7k, por ser del equipo Producto). Cavara y Boxhosting salen del POP (son hardware). México USD 1,8 (rollos Goldenstar y cajas Novalos); las playeras y portagafetes de Ideas de Alto Impacto son merch, no POP.
- **Payroll KAM Chile (G1125)** existe en Odoo todo 2026: USD 3,6k (ene) → 6,1k (abr) → 9,5k (jun) → 16,2k (sep); 35 % son «asignaciones». Jul–sep promedio USD 13,5k ÷ 29 POS = USD 466 por POS, igual al P&L; ene–sep USD 436. El P&L del DWH solo lo muestra desde julio porque antes el master lo tenía en G1110/G0105. G1124 Ops: USD 5–8k/mes (junio 23k, probable finiquito). México: Janet solo en G1125 en septiembre (MXN 37,7k).
- Artifact del modelo actualizado (v2, 25-oct) con estos valores y el input de SIM: https://claude.ai/artifact/MXK4y1zZwTygVJ2fp7ct6h
- Preguntas del 23-oct: la 1 queda respondida en su parte Transbank (arriendo) y abierta en Haulmer/Cavara/Boxhosting; la 4 queda respondida en el costo por envío y abierta en qué más se despacha.

### Pendientes menores
- 2026-10-23 · Término Payments = 0 antes de 2026 porque `pos_sold_monthly` parte en 2026-01: extenderla hacia atrás sobre `pos_sales2` exige validar que el canal ISR/KAM se reconstruya en 2025. No es parte del PR #856.
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
- 2026-10-22 · PR #856 abierto (fix ISR, reviewers Pablo y Nicolás) y DM a Pablo con PR, artifact de escenarios New POS e impacto.
- 2026-10-22 · Segundo DM a Pablo pidiendo revisión del PR #856 y aprobación para el merge: https://agendapro.slack.com/archives/D095G9DL97T/p1792702130644489
- 2026-10-23 · Mencionada de nuevo en [[2026-10-20 One-time de IA en el MRR y Payback POS (Pablo)]] (supuestos de la planilla de Ignacio, CAC con o sin sueldos KAM, tarjeta en FP&A Priorities)
- 2026-10-23 · Frente 2: decisión de armar el cálculo como artifact HTML primero; 5 preguntas de costos redactadas para que Joaquín las envíe a Finanzas y Payments.
- 2026-10-23 · Artifact del payback POS KAM publicado (supuestos editables, CAC con/sin sueldos, estimado vs real por cohorte); pendiente revisión de Joaquín y luego Pablo.
- 2026-10-23 · Reporte con mensual por defecto; borrador en Slack del mensaje al hilo del comité (Agustín 16-oct), pendiente de envío por Joaquín.
- 2026-10-23 · Decisión de migrar ventas POS a pos_sales_sot; paso 2 (extracción + test + comparación) hecho en rama sin PR.
- 2026-10-23 · Plan de migración a pos_sales_sot aprobado por Joaquín; PR #860 abierto; PR #832 actualizado con merge de main, corregido (Others fuera) y listo para revisión.
- 2026-10-25 · Revisión en Odoo de G0012/G0013/G0016 y demás cuentas de costo: Transbank confirmado como arriendo, delivery CL 12,5 y MX 16,5 por envío, SIM México al margen; artifact v2.
- 2026-10-28 · Notion: página editada

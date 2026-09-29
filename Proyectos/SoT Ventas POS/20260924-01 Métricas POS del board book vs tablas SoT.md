---
type: analisis
proyecto: "[[_SoT Ventas POS]]"
estado: activo
fecha: 2026-09-24
temas: [pos, ventas, activacion]
metricas: []
---

# Métricas POS del board book vs tablas SoT

**Pregunta**: ¿Cuántos POS reporta hoy el board book de Evidence (ventas, activaciones, attachment, retención), cómo se calcula cada número y cuánto difiere de las tablas nuevas de Ignacio (`dwh.pos_sales_sot`, `dwh.pos_ncro`, `dwh.pos_resales`)?

Contexto: [[20260909-01 POS activated per week — board book vs deck de Ignacio]] (definición del gráfico semanal), [[20260910-01 Cuadre ventas POS ISR jul–sep 2026]] (cuadre company_id a company_id), [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]] (qué trae cada tabla nueva). El deck de Ignacio está en [[Reporte SoT ProPay de Ignacio (board book recalculado)]].

## Conclusión
> - **POS activated per week** (slide 2.4), con septiembre seleccionado: **153 locales** entre el 16-abr y el 22-sep (108 CL + 45 MX = 136 compañías). Con agosto: **151**. Son locales, no máquinas; sin el filtro de cohortes de 12 meses serían 185.
> - **Ventas POS 2026** (slide 8, `country_pos_sot`), ene–ago: **489 unidades** Inbound + Stock (CL 348 = 192 New + 156 Stock; MX 141 = 46 + 95), más 35 reactivaciones (compañías) en jul–sep, que van aparte. Septiembre parcial: 52.
> - `pos_sales_sot` válida da **476 unidades** en ene–ago (−3%). El total cuadra pero **la composición no**: la SoT deja 119 u en `Others`; el Stock del board es casi todo primera compra vendida por KAM (78 de 101 eventos en jul–sep), y 20 eventos del board son inválidos en la SoT.
> - Las tablas nuevas no traen activación ni retención. Para migrar las slides 2.4 y 2.5 hay que definir antes población, grano y regla. Con `pos_sales_sot` (primeras ventas válidas, 30 trx acumuladas por compañía) salen 187 activados, contra 153.

## Qué muestra hoy el board book

Página `pages/finance/board-book/index.md` del repo `agendapro-dashboards-evidence` (rama `main` al 24-sep, último commit c5ada72). Parámetro único: `selected_month` (MonthPicker, L264, mínimo 2026-01-01; llega como `'YYYY-MM-01'`). Los bloques SQL de la página son DuckDB sobre los parquets de `sources/dwh/…`, que son SQL de Redshift.

| Slide | Métrica | Grano | Fuente (bloque de la página → archivo → tablas DWH) | Definición |
|---|---|---|---|---|
| 8 Payments | POS Sales — units · Inbound (New) vs Stock · CL y MX | Unidades (máquinas) | `pay_pos_sales_hist` (L10038), columnas `*_u` → `sources/dwh/payments/country_pos_sot.sql` → `pos_sales2`, `pos_sales`, `int_chargebee__subscription_addon_changes`, `pos_v2` (planilla KAM), `stg_hubspot__deals/contacts/deal_stages/owners`, `companies` | 12 meses hasta `selected_month`; el mes en curso sale parcial. Chargebee ∪ planilla, match compañía × mes con rescate a ±1 mes. Fecha: cambio de addon más cercano → emisión de factura → día de planilla. Cualquier estado de factura. Unidades: `item_qty` (máximo por día) y, si no hay factura, `cant` de la planilla. Canal por ownership: planilla → deal ganado «Seguimiento POS Expansión» → agente Chargebee (roster KAM de 6) → `kam_pos_owner` → ventana de 30 días (desde jun-26; antes, 90) desde la creación de la empresa. New = ISR; Stock = todo lo demás. Excluye a Tabata. Reactivaciones fuera. País por `companies.country_name` |
| 8 | POS Sales — ProPay SoT · New vs Stock vs Reactivation | Compañías (evento = compañía × mes) | Mismo bloque, columnas sin `_u` | Igual al anterior + Reactivación = etiqueta declarada en la planilla (`tipo_venta LIKE '%reactiva%'`), sin validar días fríos |
| 8 y 5.1i | B2B3 POS Attachment Rate | Unidades / compañías | `pay_pos_attachment` (L10082): numerador `country_pos_sot` con `category = 'New'`; denominador `sources/dwh/payments/pos_attachment_segment.sql` → `revenue_waterfall_software` (`change_category = 'new'`) × `int_company_dimensions` (`segment2 = 'B2B3'`) | Unidades New de todos los segmentos / compañías nuevas B2B3. 12 meses hasta `selected_month` |
| 2.4 Payments Activation — Deep dive | Matriz por cohorte: POS sold (first POS invoice), Companies, M1–M12, y promedio ponderado | «POS» = locales, con tope por compañía en `est_venues_sold` del mes de cohorte | `payd_activation_matrix` (L5333) y `payd_activation_avg` (L5391) → `sources/dwh/payments_cohorts/pos_activation_cohort.sql` → `pos_sales`, `location_sales_days` | Cohorte = mes de la primera factura POS de la compañía (emisión, cualquier estado, desde 2024). Activado = ≥ 30 trx en días qualifying (ticket promedio del día ≥ $10.000 CLP / ≥ $250 MXN) dentro de un mes relativo anclado al día de venta; `tx_threshold = 30`. País por moneda de factura. Muestra las últimas 8 cohortes ≤ `selected_month`. Una celda se muestra si cohorte + rel + 1 mes ≤ fecha de extracción y cohorte + rel ≤ `selected_month`. Recompras y reactivaciones quedan fuera por construcción |
| 2.4 | POS activated per week — last 24 weeks | Locales (fila = compañía × local dentro del tope) | `payd_weekly_activations` (L5424) → `sources/dwh/payments_cohorts/pos_activation_cohort_merchants.sql` → `pos_sales`, `pos_sales2`, `location_sales_days`, `company_locations` (el resto solo aporta nombres) | `first_activation_date_30` = primer día en que la suma corrida de trx qualifying **dentro del mes relativo** llega a 30; se reinicia cada mes. El tope se rankea con umbral 15. Filtro `cohort_month ≥ selected_month − 11 meses`. Ventana `[selected_month + 1 mes − 24 semanas, selected_month + 1 mes)`, semanas desde el lunes. Con septiembre: 16-abr → 30-sep (la semana del 13-abr queda parcial y sin datos) |
| 2.5 Payments Retention — Deep dive | Matriz por cohorte: POS activated, M1–M12, y promedio | Locales activados | `payd_retention_matrix` (L5444) y `payd_retention_avg` (L5509) → `sources/dwh/payments_cohorts/pos_retention_cohort.sql` → `pos_sales`, `location_sales_days` | M1 = mes de activación (100%). Activo en M_n = vuelve a cumplir ≥ 30 trx qualifying. `pos_observed` cuenta solo ventanas cerradas. Si la celda cerró en el calendario pero su ventana no, repite la tasa del mes anterior (solo display; el promedio la ignora) |
| 5.1i / 5.1j | POS Sales units vs Budget; POS Units país × canal vs Budget | Unidades | `pay_pos_sales_budget` (L9931), `bud_pos_units_4comp` (L7809) → `country_pos_sot` + `pw_budget_2026` | New + Stock, sin reactivación. Budget = `pos_sold_isr + pos_sold_kam` |
| 5.1k | Inbound y Stock por país vs Budget (mensual 2026) | Unidades | `payd_{chile,mexico}_{inbound,stock}_chart` y sus `_qtbl` (L5073–5332) → `country_pos_sot` + `pw_budget_2026` | Actual hasta `selected_month` |
| 5 y 4.1 Key Metrics | Fila «New POS (units)» vs Budget «New POS Sold» | Unidades | `km_country_all_data` (L8837, CTE `pos` L8900) → `country_pos_sot` | `category IN ('New', 'Stock')` |
| 5 y 4.1 (payback) | Término Payments del payback (ASP × POS) | Unidades | `km_payback_components` (L8750) y `cty_payback` (L9660) → `sources/dwh/payments/country_pos.sql` | Definición histórica (`pos_sales2`, país por moneda de factura). Es la única superficie que no se movió a `country_pos_sot` |

Hay tres poblaciones distintas de «POS vendidos» en el mismo deck: `country_pos_sot` (slide 8, budget, Key Metrics), la primera factura POS de `pos_sales` (activación y retención) y `country_pos` (payback).

## Números al 24-sep-2026

Corridos hoy en Redshift (transacciones hasta el 22-sep), aplicando en SQL la lógica del bloque DuckDB.

### a) POS activated per week (slide 2.4)

| Variante | CL | MX | Total | Compañías |
|---|---|---|---|---|
| **Board, septiembre seleccionado** (16-abr → 30-sep) | **108** | **45** | **153** | 136 (93 + 43) |
| **Board, agosto seleccionado** (17-mar → 31-ago) | **98** | **53** | **151** | 140 (89 + 51) |
| Septiembre sin el filtro de cohorte de 12 meses | 131 | 54 | 185 | — |
| Agosto sin el filtro de cohorte de 12 meses | 124 | 58 | 182 | — |
| SoT: primeras ventas válidas de `pos_sales_sot`, grano compañía, 30 trx brutas acumuladas desde `declared_sale_date`, ventana de septiembre | 124 | 63 | 187 | 187 |
| Igual, solo ventas desde el 1-oct-2025 (equivale al filtro de 12 meses) | 111 | 59 | 170 | 170 |

Detalle semanal con septiembre seleccionado (lo que dibuja el gráfico; la semana del 21-sep todavía no tiene activaciones):

| Semana | CL | MX | Total | | Semana | CL | MX | Total |
|---|---|---|---|---|---|---|---|---|
| 20-abr | 3 | 3 | 6 | | 06-jul | 4 | 5 | 9 |
| 27-abr | 2 | 1 | 3 | | 13-jul | 4 | 2 | 6 |
| 04-may | 3 | 1 | 4 | | 20-jul | 5 | 1 | 6 |
| 11-may | 6 | 4 | 10 | | 27-jul | 7 | 0 | 7 |
| 18-may | 4 | 3 | 7 | | 03-ago | 7 | 2 | 9 |
| 25-may | 3 | 3 | 6 | | 10-ago | 2 | 1 | 3 |
| 01-jun | 3 | 3 | 6 | | 17-ago | 8 | 3 | 11 |
| 08-jun | 2 | 2 | 4 | | 24-ago | 6 | 3 | 9 |
| 15-jun | 2 | 2 | 4 | | 31-ago | 7 | 4 | 11 |
| 22-jun | 5 | 0 | 5 | | 07-sep | 12 | 1 | 13 |
| 29-jun | 9 | 1 | 10 | | 14-sep | 4 | 0 | 4 |

Con agosto seleccionado cambian la ventana y el filtro de cohorte: aparecen 5 semanas de marzo y abril (entre ellas 06-abr con 9, de las que 8 son de MX) y la semana del 31-ago queda con un solo día (1).

### b) Ventas POS por mes 2026 (slide 8, `country_pos_sot`)

Unidades, con compañías entre paréntesis. React = compañías (unidades de la planilla en Chile: 12 / 13 / 9).

| Mes | CL New | CL Stock | CL React | MX New | MX Stock | MX React | New + Stock (u) |
|---|---|---|---|---|---|---|---|
| ene | 29 (24) | 12 (10) | — | 4 (4) | 5 (5) | — | 50 |
| feb | 21 (19) | 6 (6) | — | 5 (5) | 1 (1) | — | 33 |
| mar | 20 (18) | 10 (10) | — | 10 (10) | 11 (10) | — | 51 |
| abr | 15 (15) | 27 (24) | — | 11 (11) | 8 (8) | — | 61 |
| may | 21 (21) | 17 (17) | — | 2 (2) | 18 (16) | — | 58 |
| jun | 8 (8) | 30 (28) | — | 8 (4) | 15 (15) | — | 61 |
| jul | 31 (29) | 26 (19) | 12 | 1 (1) | 20 (18) | 1 | 78 |
| ago | 47 (43) | 28 (26) | 13 | 5 (5) | 17 (17) | 1 | 97 |
| sep (parcial) | 24 (23) | 15 (13) | 7 | 4 (4) | 9 (8) | 1 | 52 |
| **ene–ago** | **192** | **156** | | **46** | **95** | | **489** |

Con `selected_month` = 2026-09 el gráfico muestra oct-25 → sep-26; con 2026-08, sep-25 → ago-26. Los meses que se repiten tienen el mismo valor, porque el parámetro solo corta la serie. Sep–dic 2025 en unidades: CL New 23 / 28 / 14 / 23, CL Stock 14 / 7 / 4 / 5; MX New 18 / 16 / 15 / 8, MX Stock 10 / 6 / 12 / 9.

### c) Activación mensual por cohorte (slide 2.4, septiembre seleccionado)

% acumulado de «POS sold» (locales tope) con ≥ 30 trx qualifying en el mes relativo. CL + MX.

| Cohorte | POS sold | Compañías | M1 | M2 | M3 | M4 | M5 | M6 | M7 |
|---|---|---|---|---|---|---|---|---|---|
| 2026-02 | 30 | 29 | 23,3% | 36,7% | 36,7% | 40,0% | 40,0% | 40,0% | 40,0% |
| 2026-03 | 47 | 45 | 17,0% | 31,9% | 36,2% | 38,3% | 38,3% | 42,6% | |
| 2026-04 | 55 | 54 | 18,2% | 43,6% | 45,5% | 45,5% | 47,3% | | |
| 2026-05 | 54 | 52 | 18,5% | 40,7% | 46,3% | 48,1% | | | |
| 2026-06 | 59 | 53 | 25,4% | 33,9% | 42,4% | | | | |
| 2026-07 | 76 | 64 | 25,0% | 44,7% | | | | | |
| 2026-08 | 91 | 85 | 18,7% | | | | | | |
| 2026-09 | 47 | 45 | | | | | | | |

Con agosto seleccionado las celdas son las mismas, porque la madurez a la fecha de extracción corta antes que el mes elegido. Sale la cohorte 2026-09 y entra la 2026-01: 40 POS, 38 compañías; M1–M8 = 10,0 / 20,0 / 20,0 / 22,5 / 27,5 / 30,0 / 32,5 / 35,0%.

La población calza con `pos_ncro` en compañías (primera venta POS por comercio, por `ncro_date`): ene 40, feb 29, mar 47, abr 54, may 52, jun 54, jul 58, ago 87, sep 47. La diferencia es de 6 compañías o menos por mes. Las unidades de `pos_ncro` (`pos_quantity`: 43, 30, 50, 56, 54, 58, 67, 94, 49) no son `est_venues_sold`.

### d) Attachment B2B3 (slides 8 y 5.1i) y retención

| Mes | Denominador CL (nuevos B2B3) | CL board | CL con SoT (ISR válidas) | Denominador MX | MX board | MX con SoT (ISR válidas) |
|---|---|---|---|---|---|---|
| ene | 124 | 23,4% | 12,1% | 116 | 3,4% | 1,7% |
| feb | 94 | 22,3% | 16,0% | 90 | 5,6% | 3,3% |
| mar | 125 | 16,0% | 12,0% | 119 | 8,4% | 2,5% |
| abr | 127 | 11,8% | 7,9% | 124 | 8,9% | 2,4% |
| may | 116 | 18,1% | 15,5% | 106 | 1,9% | 1,9% |
| jun | 78 | 10,3% | 6,4% | 102 | 7,8% | 5,9% |
| jul | 78 | 39,7% | 29,5% | 77 | 1,3% | 0,0% |
| ago | 95 | 49,5% | 42,1% | 111 | 4,5% | 0,9% |
| sep (parcial) | 54 | 44,4% | 35,2% | 74 | 5,4% | 2,7% |

Si el numerador SoT suma ISR + Others de primeras ventas, Chile queda más cerca del board: jul 34,6%, ago 48,4%. La retención (2.5) no la reproduje: su fuente es la más pesada del capítulo y ninguna tabla nueva trae actividad posterior a la venta para compararla.

### e) Tablas SoT, mismos meses (`pos_sales_sot`, `is_valid_purchase`, por `effective_sale_date`)

Unidades de compras (`new` + `pos_additional`, sin reactivación) por canal. Reactivaciones válidas en compañías.

| Mes | CL ISR | CL KAM | CL Others | CL total (compañías) | CL React | MX ISR | MX KAM | MX Others | MX total (compañías) | MX React |
|---|---|---|---|---|---|---|---|---|---|---|
| ene | 15 | 5 | 14 | 34 (30) | 0 | 2 | 2 | 5 | 9 (8) | 0 |
| feb | 15 | 3 | 16 | 34 (29) | 1 | 3 | 1 | 4 | 8 (8) | 0 |
| mar | 15 | 7 | 9 | 31 (27) | 0 | 3 | 8 | 5 | 16 (16) | 0 |
| abr | 10 | 19 | 6 | 35 (32) | 0 | 3 | 6 | 7 | 16 (16) | 0 |
| may | 18 | 12 | 4 | 34 (32) | 3 | 2 | 14 | 3 | 19 (17) | 0 |
| jun | 5 | 28 | 6 | 39 (36) | 1 | 6 | 14 | 5 | 25 (19) | 1 |
| jul | 23 | 24 | 8 | 55 (47) | 14 | 0 | 17 | 3 | 20 (18) | 1 |
| ago | 40 | 23 | 20 | 83 (72) | 14 | 1 | 13 | 4 | 18 (18) | 1 |
| sep (parcial) | 19 | 12 | 7 | 38 (33) | 5 | 2 | 4 | 1 | 7 (7) | 0 |
| **ene–ago** | **141** | **121** | **83** | **345** | | **20** | **75** | **36** | **131** | |

`pos_additional` dentro de esos totales, en unidades: CL 3 / 12 / 2 / 3 / 3 / 4 / 4 / 14 / 6; MX 4 / 2 / 1 / 1 / 2 / 4 / 2 / 0 / 0. Unidades inválidas que quedan fuera: CL 5 / 1 / 3 / 8 / 2 / 1 / 2 / 3 / 11; MX 1 / 2 / 4 / 4 / 1 / 0 / 3 / 2 / 7. Si en vez de la fecha efectiva (pago) se usa `declared_sale_date`, ene–ago da CL 350 y MX 131.

**Board vs SoT, unidades de compra**

| Mes | CL board | CL SoT | Δ | MX board | MX SoT | Δ |
|---|---|---|---|---|---|---|
| ene | 41 | 34 | +7 | 9 | 9 | 0 |
| feb | 27 | 34 | −7 | 6 | 8 | −2 |
| mar | 30 | 31 | −1 | 21 | 16 | +5 |
| abr | 42 | 35 | +7 | 19 | 16 | +3 |
| may | 38 | 34 | +4 | 20 | 19 | +1 |
| jun | 38 | 39 | −1 | 23 | 25 | −2 |
| jul | 57 | 55 | +2 | 21 | 20 | +1 |
| ago | 75 | 83 | −8 | 22 | 18 | +4 |
| **ene–ago** | **348** | **345** | **+3** | **141** | **131** | **+10** |
| sep (parcial) | 39 | 38 | +1 | 13 | 7 | +6 |

Por canal, en ene–ago: New del board 192 CL / 46 MX contra ISR de la SoT 141 / 20; Stock 156 / 95 contra KAM 121 / 75. La diferencia está en `Others` (83 / 36), que el board reparte entre New y Stock.

**Cruce compañía × mes, jul–sep 2026 (CL + MX)**: cada evento del board contra la fila de `pos_sales_sot` del mismo comercio y mes.

| Board | Clase en `pos_sales_sot` | Eventos | Unidades board |
|---|---|---|---|
| New (105 ev / 112 u) | `new` / ISR / válida | 73 | 78 |
| | `new` / Others / válida | 16 | 16 |
| | `pos_additional` / Others / válida (494549, 373415, 453116, 511752, 520667: conversiones arriendo → cuotas / `addon_restart`) | 5 | 5 |
| | inválida (4 `new`/Others, 1 `pos_additional`, 1 `new`/ISR) | 6 | 6 |
| | en la SoT, pero en un mes vecino | 5 | 7 |
| Stock (101 ev / 115 u) | `new` / KAM / válida | 78 | 89 |
| | `new` / KAM / inválida | 11 | 12 |
| | `pos_additional` / Others (3 válidas, 2 inválidas) | 5 | 7 |
| | `new` / Others (2 válidas, 1 inválida) y `new` / ISR (1) | 4 | 4 |
| | en la SoT, pero en un mes vecino | 3 | 3 |
| Reactivación (35 ev / 37 u) | `reactivation` / KAM / válida | 31 | 33 (la SoT: 8) |
| | `reactivation` / KAM / inválida | 4 | 4 (la SoT: 2) |

Al revés, en jul–sep hay 13 filas válidas de la SoT que el board no tiene: 7 `pos_additional`/Others (13 u; 4 son de Tabata, 2 `rent_resumed` y 1 `purchase_with_pos`), 4 reactivaciones KAM (7 u; 2 de Tabata), 1 `pos_additional`/KAM de Tabata y 1 `new`/ISR.

## Diferencias y por qué

1. **Canal.** El board obliga a que todo sea New o Stock, porque su jerarquía termina en una ventana de recencia de 30/90 días. La SoT deja en `Others` lo que no tiene evidencia de vendedor: 119 u en ene–ago (25% de las válidas). En la SoT, ISR es un placeholder (el usuario que agregó el addon está en el equipo ISR de `hubspot_mapping`) y el roster KAM tiene 9 emails; el board usa 6. En México el «Inbound» del board (46 u) cae a 20 en la SoT, porque se apoyaba en la ventana de recencia.
2. **Tipo de compra no es lo mismo que canal.** El Stock del board no es «recompra de la base POS»: 78 de sus 101 eventos de jul–sep son la **primera** compra POS del comercio, vendida por KAM. Y 5 eventos New del board son `pos_additional` en la SoT (las conversiones arriendo → cuotas del cuadre del 10-sep). Si al migrar «New» pasa a significar `purchase_type = 'new'`, cambian las dos barras. Ojo: `purchase_type = 'new'` también incluye 45 reventas KAM con deal «Venta Nueva». Para primeras ventas hay que usar `pos_ncro` o un anti-join contra `pos_resales`.
3. **Validez.** El board cuenta toda factura emitida, en cualquier estado, y los eventos que solo están en la planilla. La SoT exige factura pagada + formulario (o trx como proxy antes de sep-26). En jul–sep, 20 eventos New/Stock del board son inválidos en la SoT; los motivos más comunes son `form_not_completed`, `no_trx_pre_sep26_form_proxy` y `no_invoice`. Hay 4 reactivaciones más en la misma situación.
4. **Alcance.** El board excluye a Tabata y no ve `rent_resumed` ni `purchase_with_pos` sin evento en Chargebee o en la planilla. La SoT incluye todo eso: son 13 filas válidas en jul–sep, la mayoría de Tabata.
5. **Unidades de reactivación.** El universo de compañías coincide (31 de 35), pero el board toma las máquinas de la planilla (37 u) y la SoT da 10, porque una `reactivation_soft` vale 0 unidades y solo `reactivation_with_purchase` trae máquina.
6. **Fecha.** El board fecha por cambio de addon → emisión de factura → día de planilla. La `effective_sale_date` de la SoT es la fecha de pago, y `declared_sale_date` es la fecha del deal o del primer evento. 8 eventos de jul–sep caen en un mes vecino; el neto del trimestre es chico, pero mueve meses sueltos (feb −7 y ago −8 en Chile).
7. **Activación.** El board usa una tercera población: la primera factura en `pos_sales`, con país por moneda. Además cuenta por local, con días qualifying, reinicio mensual y filtro de 12 meses. Las tablas nuevas no traen activación. Con primeras ventas válidas y 30 trx brutas acumuladas por compañía salen 187 (170 con el filtro de 12 meses), contra 153 locales / 136 compañías. La población de la matriz de cohortes sí calza con `pos_ncro` (±6 compañías por mes).
8. **Payback.** Sigue en `country_pos`, la definición histórica: es una tercera cifra de «POS vendidos» dentro del mismo deck.

### Qué corregir o decidir al migrar a la tabla consolidada
- **Canal `Others`**: decidir si se reporta como tercera barra o si se reparte. El budget viene en Inbound / Field Sales (Stock), así que la comparación vs budget necesita una regla explícita.
- **Qué significa New/Stock**: canal (ISR vs KAM, como hoy) o tipo de compra (primera venta vs reventa). Hoy son cosas distintas y el título del gráfico sugiere la segunda.
- **Validez**: adoptar `is_valid_purchase` cambia el criterio de «factura emitida, cualquier estado», acordado el 7-sep, y saca ~20 eventos por trimestre. Hay que decidirlo explícitamente con Payments.
- **Tabata**: el board la excluye y la SoT no.
- **Reactivación**: pasar de la etiqueta declarada en la planilla a `is_valid_reactivation` (15 trx hasta el día 10). Definir si las soft cuentan en unidades (hoy suman 1 en el board y 0 en la SoT).
- **Activación y retención (2.4 / 2.5)**: renombrar «POS activated» a «Locales activados», o migrar a grano compañía. Revisar el reinicio mensual de la suma corrida y el filtro de cohorte de 12 meses en un gráfico de flujo semanal. Definir si la población pasa a `pos_ncro`.
- **Payback**: sacarlo de `country_pos` o documentar por qué se queda ahí.
- **Detalles del código**: el comentario de `payd_activation_matrix` dice «regrouped to calendar quarters», pero agrupa por mes. Los textos fijos de la slide 8 («+51% YoY in 2026 Q2», «POS sales working … except Mexico inbound») no se recalculan con el mes elegido.

## Caveats
- Los números salen de Redshift hoy, no del parquet publicado. El mes en curso es parcial y el parquet puede ser de otro día.
- El semanal y la matriz se reprodujeron con una versión reducida de las fuentes: solo días qualifying de `location_sales_days`. Para contar activados es equivalente, porque un local sin días qualifying nunca activa y queda al final del ranking del tope. No se replicaron nombre, vendedor, onboarder ni GMV, que no cambian el conteo.
- La nota del 9-sep reportó 150 con otra ventana (30-mar → 07-sep). La ventana real con septiembre es 16-abr → 30-sep. Desde el 9-sep además cambió el ancla de cohorte a fecha de emisión (commit 7f905ea).
- En `pos_sales_sot` hay 433 filas sin `effective_sale_date`, casi todas inválidas. En el cruce por compañía usé `COALESCE(effective, declared, deal, invoice_created)`; en las tablas mensuales, solo `effective_sale_date`.
- Hay tres unidades distintas: el board usa `item_qty` o `cant`, la matriz de activación `est_venues_sold` (mín. entre `item_qty` y locales) y la SoT `pos_quantity` (deal → factura → 1).
- País: `country_pos_sot` usa `companies.country_name`, las cohortes de activación la moneda de factura y la SoT el código `CL` / `MX`.
- Las tablas de Ignacio se recrean en cada corrida de dbt, así que estas cifras se mueven.
- La página pesa 697 KB y pasa el límite de 512 KB de Code Explorer (tampoco la indexa el code search). La leí desde el raw de GitHub; las líneas citadas son de `main` al 24-sep.

## Cómo reproducir

1. **Ventas slide 8**: `sources/dwh/payments/country_pos_sot.sql` tal cual, con el `SELECT` final filtrado a `date >= '2025-09-01' AND country_group IN ('Chile','Mexico')`. Es `pay_pos_sales_hist` sin el corte `m <= selected_month`. El attachment divide `category = 'New'` (unidades) por las compañías nuevas B2B3 de `revenue_waterfall_software` × `int_company_dimensions`.
2. **POS activated per week** (versión reducida de `pos_activation_cohort_merchants.sql`; usar `company_id` bigint sin `::text`, o da timeout):
```sql
WITH cfs AS (SELECT company_id, MIN(invoice_created_at)::date first_sale_date,
                    DATE_TRUNC('month', MIN(invoice_created_at))::date cohort_month
             FROM dwh.pos_sales WHERE invoice_created_at >= '2024-01-01' GROUP BY 1),
cm  AS (-- tope = MAX(est_venues_sold) del mes de cohorte; país por invoice_currency (CLP/MXN)
        ...),
ldr AS (SELECT l.company_id, l.location_id, l.day::date AS day, l.pos_payment_count,
               <mes relativo anclado al día de first_sale_date> AS rel_month
        FROM dwh.location_sales_days l JOIN cfs ON cfs.company_id = l.company_id
        WHERE l.day >= cfs.first_sale_date AND l.day <= CURRENT_DATE - 1 AND l.pos_payment_count > 0
          AND ((l.currency = 'CLP' AND l.gpv_pos / l.pos_payment_count >= 10000)
            OR (l.currency = 'MXN' AND l.gpv_pos / l.pos_payment_count >= 250)))
-- tope: ROW_NUMBER() por compañía según el primer rel_month con >= 15 trx; rn <= tope
-- fad30: MIN(day) donde SUM(trx) OVER (PARTITION BY compañía, local, rel_month ORDER BY day) >= 30
SELECT DATE_TRUNC('week', fad30) AS semana, country, COUNT(*) AS locales
FROM base
WHERE cohort_month >= DATE '2025-10-01'
  AND fad30 >= DATE '2026-04-16' AND fad30 < DATE '2026-10-01'   -- septiembre
GROUP BY 1, 2;
```
3. **Matriz de cohortes**: igual, con el tope rankeado a 30 y `a_r = COUNT(locales con primer rel_month >= 30 <= r)` / `SUM(tope)` por cohorte.
4. **SoT mensual**: `dwh.pos_sales_sot` con `is_valid_purchase`, `country IN ('CL','MX')`, agrupado por `DATE_TRUNC('month', effective_sale_date)`, `purchase_type` y `channel_attribution`; `SUM(pos_quantity)` y `COUNT(DISTINCT company_id)`.
5. **Cruce compañía × mes**: eventos del board (CTE `tipado` de `country_pos_sot`, jul–sep) contra `pos_sales_sot` agregada por `company_id` × mes de `COALESCE(effective_sale_date, declared_sale_date, deal_sale_date, invoice_created_date)`. Hacerlo en una sola query da timeout; lo hice en dos pasos, con la lista de `company_id` del board embebida.
6. **Activación con la SoT**: primeras ventas = `pos_sales_sot` anti-join `pos_resales` por `sale_id`, `is_valid_purchase`. Trx diarias por compañía desde `location_sales_days` (desde `declared_sale_date`), suma acumulada sin reinicio y primer día ≥ 30; se cuenta en la ventana 16-abr → 30-sep.

---
type: recurso
temas: [pos, ventas, dwh]
proyecto: "[[_SoT Ventas POS]]"
autor: "[[Ignacio Embry]]"
fecha: 2026-09-24
---

# Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)

**Propósito**: referencia de las tres tablas nuevas de ventas POS de Ignacio en el DWH: qué trae cada una, cómo se construye, cómo consultarla y qué casos borde tiene hoy.

- **Repo**: `agendapro/agendapro-dbt`, rama `refactor/postgres-to-redshift`, carpeta `models/marts/sales/`. En `master` no existen.
- **PRs de Ignacio** (todos contra `refactor/postgres-to-redshift`): #396 (22-sep, primera versión: una tabla de «intentos», solo 2026) → #400 (22-sep, dos fechas `declared_sale_date` / `effective_sale_date` y plazo de las soft) → #401 (cerrado sin merge, reemplazado) → #402 (23-sep, v4: se separa en `pos_ncro` + `pos_resales`, toda la historia). En producción está la v4.
- **Documentación**: Notion *DWH documentation → purchase_attribution_model*. Contexto: [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]].
- Las tres son `materialized='table'` en el schema `dwh`, tags `redshift_marts, pos, propay`. Todavía no las lee ningún tablero (PR #402 y búsqueda en repos): el board book sigue en `country_pos_sot.sql`.

| Tabla | Qué es | Grano (llave) | Filas al corte |
|---|---|---|---|
| `dwh.pos_ncro` | Primera venta POS de cada comercio (la «POS First Sales» de la reunión; NCRO = New Card Reader Owner) | 1 fila por `company_id` | 2.019 |
| `dwh.pos_resales` | Ventas posteriores a la primera: reactivaciones y POS adicionales | 1 fila por evento (`sale_id`) | 273 |
| `dwh.pos_sales_sot` | Tabla maestra = `pos_ncro` ∪ `pos_resales` + checklist de venta efectiva | 1 fila por evento de venta (`sale_id`) | 2.292 |

## dwh.pos_ncro — primera venta del comercio

**Modelo**: `models/marts/sales/pos_ncro.sql` (yml compartido `_pos_sales_sot.yml`). Tests: `company_id` unique + not_null, `ncro_date` not_null, accepted_values en `ncro_evidence` y `channel_attribution`.

**Fuentes**: `stg_hubspot__deals` + `stg_hubspot__owners` (deals KAM), `pos_sales` menos `pos_omitted_invoices` (facturas POS), `int_chargebee__subscription_addon_changes` (altas y bajas de addon y quién las hizo), `propay_transactions` + `haulmer/sumup/oel_augmented_transactions` (trx reales), `stg_google_sheets__hubspot_mapping` (equipo ISR), `companies` (país).

**Lógica**
- **Deal KAM**: pipeline `82319002` (Seguimiento POS Expansión), etapa `154822774` (Ganado), con `company_id_agendapro` y `fecha_de_venta_pos`, y owner en un roster fijo de 9 emails: rocio, janet, jaime, alvaro, fernandobarra, silcris, mariajosehurtado, tabata, ignaciavalderrama (el SoT del board usa 6).
- **Factura POS**: líneas de `pos_sales` en CLP/MXN, sin ítems «Integración Terminal» ni «Aceleración», sin facturas omitidas. Una línea de renta (`renta|arriendo|cuotas`) cuenta como venta solo si es la primera renta, sube la cantidad, llega más de 60 d después de la renta anterior o hubo una baja de addon POS entre las dos; si no, es la mensualidad. Líneas a 10 d o menos son una misma venta; setup + primera renta a 45 d o menos, también.
- **Trx real**: providers POS del gateway (1, 3, 6, 7, 34, 35, 36, 67, 68, 69, 100, 166) sin la prueba de instalación (100 CLP en Chile, 1 MXN o menos fuera), fechas entre 2018 y hoy, más externas Haulmer/SumUp/OEL con monto > 0.
- **`ncro_date`**: lo que ocurra primero entre primer deal KAM, primera factura POS y primera trx real. `ncro_evidence` dice cuál (`deal` / `invoice` / `trx`).
- **Deal del NCRO**: el deal KAM más cercano a ±45 d de `ncro_date`. Absorbe todas las líneas de venta a ±45 d del deal. Sin deal, las facturas del NCRO son el primer grupo de venta, si empieza a 45 d o menos del primer evento.
- **Canal**: `KAM` si hay deal del NCRO. `ISR` si el usuario Chargebee que agregó el addon POS (el más cercano a ±31 d de la *primera factura del comercio*) está en el equipo AE/ISR/MDR de `hubspot_mapping` y no en el roster KAM. Es un placeholder hasta tener la fuente ISR. Si no, `Others`.
- **Cantidad** (`pos_quantity`): cantidad del deal (`cantidad_de_terminales`, 1 si viene vacía) → máximo `item_qty` de las facturas → 1.

| Columna | Significado |
|---|---|
| `company_id` | Comercio. Llave. |
| `country` | `CL` / `MX` (desde `companies`; 2 nulos). |
| `ncro_date`, `ncro_evidence` | Fecha y tipo del primer evento POS. |
| `channel_attribution`, `seller_email` | Canal y vendedor (owner del deal o usuario del addon). |
| `deal_id`, `deal_sale_date`, `deal_sale_type` | Deal KAM que absorbe la primera venta. |
| `invoice_ids`, `invoice_created_date`, `invoice_paid_date`, `is_invoice_paid` | Facturas de la primera venta; pagada = alguna pagada. |
| `pos_quantity` | Terminales de la primera venta. |
| `first_trx_date` | Primera trx real (activación; no valida nada). |

## dwh.pos_resales — reactivaciones y POS adicionales

**Modelo**: `models/marts/sales/pos_resales.sql`. Lee `pos_ncro` y las mismas fuentes. Tests: `sale_id` unique + not_null, `company_id` con `relationships` a `pos_ncro`, `declared_sale_date` not_null, accepted_values en canal (`KAM`, `Others`), `purchase_sub_type` y `sale_evidence`.

**Grano**: un evento de venta posterior al NCRO. `sale_id` = `deal-<deal_id>` (KAM) o `inv-<company_id>-<n>` (venta por factura).

**Lógica**
- **Eventos**: (a) todo deal KAM Ganado que no es el del NCRO (absorbe las líneas de venta a ±45 d); (b) grupo de venta por factura que no cae a ±45 d de ningún deal KAM ni en la ventana del NCRO. `sale_evidence` dice qué regla lo abrió: `deal`, `qty_up` (renta con más cantidad), `purchase_with_pos` (compra con POS previo), `addon_restart` (renta con baja de addon entre medio), `rent_resumed` (renta tras más de 60 d sin renta), `first_invoice` (primera factura de un NCRO que nació por trx).
- **Tipo** (`purchase_sub_type`): en KAM manda el deal: «Reactivación» con cantidad 0 → `reactivation_soft`; «Reactivación» → `reactivation_with_purchase`; «POS adicional» → `pos_additional`; «Venta Nueva» o vacío → `new`. En Others manda el historial: con actividad POS (renta, trx real, deal previo o el propio NCRO) en los 60 d anteriores → `pos_additional`; si no → `reactivation_with_purchase`.
- `history_purchase_type` / `history_mismatch`: lo que dice el historial y si el deal KAM lo contradice. Un «Venta Nueva» después del NCRO siempre es mismatch. Solo se expone, no valida.
- **Nunca ISR**: una reventa no es primera venta (regla de Ignacio, 23-sep).
- **Cantidad**: KAM = la del deal (0 en soft); Others = máximo `item_qty` del grupo de facturas.

| Columna | Significado |
|---|---|
| `sale_id` | Llave del evento. |
| `declared_sale_date` | Fecha del deal (KAM) o emisión de la factura que abre el evento (Others). |
| `channel_attribution`, `seller_email` | `KAM` / `Others`; owner del deal o usuario del addon. |
| `purchase_sub_type`, `sale_evidence` | Tipo de reventa y regla que la abrió. |
| `pos_quantity` | Terminales del evento. |
| `deal_*`, `invoice_*`, `is_invoice_paid` | Deal y facturas del evento. |
| `last_pos_activity_date`, `days_inactive` | Última evidencia de POS antes del evento y días desde ahí. |
| `history_purchase_type`, `history_mismatch` | Tipo según historial y si el deal lo contradice. |

## dwh.pos_sales_sot — tabla maestra

**Modelo**: `models/marts/sales/pos_sales_sot.sql`. Lee `pos_ncro`, `pos_resales`, `pos_cl_form_assignments` (formulario POS CL), `stg_google_sheets__control_terminales` (proxy MX) y las trx reales. Tests: `sale_id` unique + not_null, `company_id` not_null, accepted_values en `channel_attribution` y `purchase_type`, not_null en `declared_sale_date`, `history_mismatch` e `is_valid_purchase`.

**Grano**: un evento de venta. `sale_id` = `deal-<deal_id>` (KAM, primera venta o reventa), `ncro-<company_id>` (primera venta sin deal) o `inv-<company_id>-<n>` (reventa por factura).

**Parámetros** (CTE `params`): `strict_rules_from = 2026-09-01`, `soft_trx_target = 15`, `soft_deadline_day = 10`.

**Checklist y validez**
- **Formulario** (`is_form_completed`): ISR copia `is_invoice_paid` (no se le exige). KAM/Others en CL: formulario POS CL no cancelado, asignado entre −15 y +60 d de la venta declarada, y completado. En MX: proxy, una fila en *Control terminales* entre −15 y +90 d. Antes del 1-sep-2026 basta 1 trx o más después de la venta, prueba de instalación incluida.
- **`is_valid_purchase`**: `new`, `pos_additional` y `reactivation_with_purchase` = factura pagada + formulario. `reactivation_soft` = desde el 1-sep-2026, 15 trx reales o más entre la venta declarada y el día 10 del mes siguiente; antes del 1-sep vale tal como se declaró. Pasado el plazo sin la meta, no valida nunca.
- **`invalid_reason`** (NULL si es válida): `no_invoice`, `invoice_not_paid`, `form_not_sent` (solo CL), `form_not_completed`, `no_trx_pre_sep26_form_proxy`, `soft_reactivation_trx<15 (n, until fecha)`, `soft_reactivation_expired (…)`. Se concatenan con `;`.
- **Fechas**: `declared_sale_date` = fecha del deal (KAM), primer evento (NCRO sin deal) o emisión de la factura (reventa Others). `effective_sale_date` = primer pago de las facturas del evento; en soft, la fecha declarada cuando llega a la meta. Queda NULL si la factura no está pagada o la soft no llega. Puede haber fecha efectiva con venta inválida (pagada sin formulario).
- `purchase_type` agrega `purchase_sub_type` en `new` / `reactivation` / `pos_additional`.

| Columna | Significado |
|---|---|
| `sale_id`, `company_id`, `country` | Llave, comercio, `CL` / `MX`. |
| `channel_attribution`, `seller_email` | `KAM` / `ISR` / `Others` y vendedor. |
| `purchase_type`, `purchase_sub_type` | Tipo; el sub tipo separa `reactivation_soft` de `reactivation_with_purchase`. |
| `sale_evidence` | Qué abrió la fila (deal, invoice, trx, qty_up…). |
| `history_purchase_type`, `history_mismatch` | Historial vs deal. |
| `pos_quantity` | Terminales (0 en soft). |
| `declared_sale_date`, `effective_sale_date` | Venta declarada y venta efectiva. |
| `deal_id`, `deal_sale_date`, `invoice_ids`, `invoice_created_date`, `invoice_paid_date`, `is_invoice_paid` | Evidencia comercial y de cobro. |
| `form_sent_date`, `form_completed_date`, `is_form_completed` | Formulario (fechas solo CL). |
| `trx_post_purchase`, `trx_by_soft_deadline`, `last_trx_pre_purchase` | Trx reales después de la venta, hasta el plazo soft y la última antes. |
| `last_pos_activity_date`, `days_inactive` | Solo reventas (NULL en NCRO). |
| `is_valid_reactivation`, `is_valid_purchase`, `invalid_reason` | Veredicto. |

## Cómo se relacionan

- `pos_sales_sot` = `pos_ncro` (2.019) + `pos_resales` (273) = 2.292 filas, sin llaves repetidas. `pos_resales` depende de `pos_ncro` (usa su `ncro_date` y su ventana) y `pos_sales_sot` agrega el checklist encima.
- Para quedarse solo con primeras ventas no basta `purchase_type = 'new'`: 45 reventas KAM con deal «Venta Nueva» también son `new` (9 en 2026). Usar anti-join contra `pos_resales` o `JOIN dwh.pos_ncro`.
- **Contra `POS_VENTAS_SOT.sql` y `country_pos_sot.sql`** ([[SQL — SoT Ventas POS]]): el board book lee `dwh.country_pos_sot` (repo `agendapro-dashboards-evidence`, `sources/dwh/payments/country_pos_sot.sql`), que también usan `pos_activation_cohort.sql` y `pos_retention_cohort.sql`. Las tablas nuevas todavía no reemplazan nada.

| | `POS_VENTAS_SOT.sql` / `country_pos_sot.sql` | `pos_sales_sot` |
|---|---|---|
| Grano | empresa × mes de venta | evento (`sale_id`); puede haber 2 filas por empresa y mes |
| Fuentes | `pos_sales2` + planilla KAM `pos_v2` + deals HubSpot | deals HubSpot + `pos_sales` + formularios + trx reales; no lee `pos_sales2` ni la planilla |
| Canal | planilla → deal → agente Chargebee → `kam_pos_owner` → ventana de antigüedad | deal KAM → usuario ISR del addon (placeholder) → Others |
| Tipo | planilla, o derivado (60 d sin trx = reactivación) | deal en KAM; historial de 60 d en Others; ISR siempre `new` |
| Fecha | addon más cercano → emisión de factura → día de planilla | declarada (deal / primera evidencia) y efectiva (pago) |
| Cantidad | Chargebee y si no, planilla | deal → factura → 1 |
| Impagas | cuentan | quedan con `is_valid_purchase = false` |
| Cobertura | desde 2025 | toda la historia (trx desde dic-2021, facturas desde 2023, deals desde 2023–24) |
| Roster KAM | 6 emails | 9 emails |

- Para comparar con el listado de sedes ISR del cuadre ([[20260910-01 Cuadre ventas POS ISR jul–sep 2026]]) hay que sumar `ISR + Others` de primeras ventas por `effective_sale_date`: julio sigue calzando (27).

## Cómo consultarlas

`country` viene como código: `'CL'` / `'MX'`, no `'Chile'`. Hay 4 filas con otro valor o nulo. Las unidades salen de `SUM(pos_quantity)` (las soft suman 0). Las tablas se recrean en cada corrida de dbt, así que los números se mueven.

1. Ventas válidas por mes, país, tipo y canal:
```sql
SELECT DATE_TRUNC('month', effective_sale_date)::date AS mes,
       country, purchase_type, channel_attribution,
       COUNT(*)                   AS ventas,
       COUNT(DISTINCT company_id) AS companies,
       SUM(pos_quantity)          AS unidades
FROM dwh.pos_sales_sot
WHERE is_valid_purchase
  AND country IN ('CL', 'MX')
  AND effective_sale_date >= DATE '2026-01-01'
GROUP BY 1, 2, 3, 4
ORDER BY 1, 2, 3, 4;
```

2. Primeras ventas de Chile por canal (comparable con el listado: ISR + Others):
```sql
SELECT DATE_TRUNC('month', s.effective_sale_date)::date AS mes,
       SUM(CASE WHEN s.channel_attribution = 'ISR'    THEN s.pos_quantity ELSE 0 END) AS u_isr,
       SUM(CASE WHEN s.channel_attribution = 'Others' THEN s.pos_quantity ELSE 0 END) AS u_others,
       SUM(CASE WHEN s.channel_attribution = 'KAM'    THEN s.pos_quantity ELSE 0 END) AS u_kam
FROM dwh.pos_sales_sot s
LEFT JOIN dwh.pos_resales r ON r.sale_id = s.sale_id
WHERE r.sale_id IS NULL                 -- solo NCRO
  AND s.is_valid_purchase
  AND s.country = 'CL'
  AND s.effective_sale_date >= DATE '2026-07-01'
GROUP BY 1
ORDER BY 1;
```

3. Qué está pendiente o inválido este mes:
```sql
SELECT sale_id, company_id, country, channel_attribution, purchase_sub_type,
       declared_sale_date, invoice_ids, is_invoice_paid, form_sent_date,
       trx_by_soft_deadline, invalid_reason
FROM dwh.pos_sales_sot
WHERE NOT is_valid_purchase
  AND declared_sale_date >= DATE_TRUNC('month', CURRENT_DATE)
ORDER BY invalid_reason, declared_sale_date;
```

4. Historia completa de un comercio:
```sql
SELECT n.ncro_date, n.ncro_evidence, n.first_trx_date, s.*
FROM dwh.pos_sales_sot s
JOIN dwh.pos_ncro n ON n.company_id = s.company_id
WHERE s.company_id = 494549
ORDER BY s.declared_sale_date;
```

## Perfil al 24-sep-2026 17:31 (hora Chile)

Consultas SELECT del 24-sep entre 17:31 y 17:38 (UTC−3). Las tres tablas se habían recreado ese mismo día entre 17:11 y 17:12 (`pg_class_info`).

**Volumen y grano**

| Tabla | Filas | Llaves distintas | Duplicados | Companies | Fecha declarada | Fecha efectiva / pago |
|---|---|---|---|---|---|---|
| `pos_sales_sot` | 2.292 | 2.292 | 0 | 2.019 | 17-dic-2021 a 23-sep-2026 | 12-jun-2023 a 23-sep-2026 |
| `pos_ncro` | 2.019 | 2.019 | 0 | 2.019 | 17-dic-2021 a 23-sep-2026 | 12-jun-2023 a 23-sep-2026 |
| `pos_resales` | 273 | 273 | 0 | 223 | 29-jun-2023 a 23-sep-2026 | 26-jul-2023 a 23-sep-2026 |

**`pos_sales_sot` por año (fecha declarada)**

| Año | Filas | new | reactivation | pos_additional | Pagadas | Válidas | Efectiva nula | KAM | ISR | Others |
|---|---|---|---|---|---|---|---|---|---|---|
| 2021 | 2 | 2 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 2 |
| 2022 | 172 | 172 | 0 | 0 | 0 | 0 | 172 | 0 | 0 | 172 |
| 2023 | 240 | 223 | 0 | 17 | 114 | 88 | 126 | 4 | 0 | 236 |
| 2024 | 465 | 432 | 2 | 31 | 406 | 354 | 59 | 110 | 0 | 355 |
| 2025 | 814 | 761 | 7 | 46 | 772 | 707 | 42 | 139 | 81 | 594 |
| 2026 | 599 | 473 | 55 | 71 | 545 | 508 | 31 | 262 | 171 | 166 |

2021–2022 son primeras ventas por trx sin factura (anteriores a Chargebee): todas inválidas. ISR aparece recién en abr-2025.

**2026 por mes y país (fecha declarada; «U. válidas» = `SUM(pos_quantity)` de las válidas)**

| Mes | País | Filas | new | react. | adic. | KAM | ISR | Others | Pagadas | Válidas | U. válidas | Efectiva nula |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ene | CL | 39 | 32 | 1 | 6 | 5 | 14 | 20 | 37 | 32 | 38 | 2 |
| feb | CL | 31 | 23 | 2 | 6 | 3 | 14 | 14 | 31 | 30 | 33 | 0 |
| mar | CL | 32 | 29 | 1 | 2 | 8 | 13 | 11 | 30 | 26 | 29 | 2 |
| abr | CL | 43 | 39 | 0 | 4 | 22 | 10 | 11 | 40 | 32 | 35 | 3 |
| may | CL | 41 | 34 | 4 | 3 | 15 | 18 | 8 | 38 | 35 | 37 | 3 |
| jun | CL | 43 | 37 | 3 | 3 | 31 | 7 | 5 | 42 | 41 | 48 | 1 |
| jul | CL | 62 | 42 | 15 | 5 | 31 | 21 | 10 | 53 | 58 | 58 | 1 |
| ago | CL | 94 | 65 | 14 | 15 | 38 | 36 | 20 | 78 | 87 | 89 | 4 |
| sep | CL | 54 | 36 | 7 | 11 | 20 | 19 | 15 | 45 | 36 | 36 | 8 |
| ene | MX | 12 | 7 | 1 | 4 | 3 | 2 | 7 | 12 | 9 | 10 | 0 |
| feb | MX | 8 | 5 | 0 | 3 | 0 | 3 | 5 | 8 | 8 | 8 | 0 |
| mar | MX | 19 | 18 | 1 | 0 | 8 | 3 | 8 | 19 | 15 | 15 | 0 |
| abr | MX | 23 | 16 | 2 | 5 | 5 | 3 | 15 | 20 | 17 | 17 | 3 |
| may | MX | 17 | 17 | 0 | 0 | 14 | 2 | 1 | 16 | 16 | 18 | 1 |
| jun | MX | 23 | 20 | 1 | 2 | 15 | 3 | 5 | 22 | 21 | 26 | 1 |
| jul | MX | 21 | 18 | 1 | 2 | 18 | 0 | 3 | 20 | 19 | 20 | 0 |
| ago | MX | 21 | 20 | 1 | 0 | 16 | 1 | 4 | 20 | 19 | 18 | 0 |
| sep | MX | 16 | 15 | 1 | 0 | 10 | 2 | 4 | 14 | 7 | 7 | 2 |

Las reactivaciones de jul–sep en CL son sobre todo soft (cantidad 0). Las pagadas pueden superar a las válidas (pagadas sin formulario) y al revés (soft válidas sin factura).

**Motivos de invalidez en 2026 (filas)**

| `invalid_reason` | CL | MX |
|---|---|---|
| `no_trx_pre_sep26_form_proxy` | 28 | 15 |
| `no_invoice` | 12 | 3 |
| `form_not_sent` | 10 | 0 |
| `form_not_completed` | 0 | 7 |
| `invoice_not_paid;no_trx_pre_sep26_form_proxy` | 3 | 4 |
| `invoice_not_paid` | 3 | 0 |
| `invoice_not_paid;form_not_sent` | 3 | 0 |
| `soft_reactivation_trx<15 (…)` | 2 | 0 |
| `no_invoice;no_trx_pre_sep26_form_proxy` | 1 | 0 |

Todos los `form_not_sent` / `form_not_completed` son de septiembre (reglas estrictas desde el 1-sep).

**Canal × tipo (toda la historia; filas / válidas / de 2026)**

| Canal | Tipo | Evidencia | Filas | Válidas | 2026 |
|---|---|---|---|---|---|
| ISR | new | invoice / trx | 252 | 245 | 171 |
| KAM | new | deal / invoice / trx (incluye 45 reventas «Venta Nueva») | 471 | 427 | 218 |
| KAM | reactivation_soft | deal | 26 | 24 | 26 |
| KAM | reactivation_with_purchase | deal / invoice | 15 | 13 | 15 |
| KAM | pos_additional | deal | 3 | 1 | 3 |
| Others | new | invoice | 965 | 787 | 76 |
| Others | new | trx | 375 | 18 | 8 |
| Others | pos_additional | addon_restart / purchase_with_pos / qty_up / rent_resumed / first_invoice | 72 / 46 / 21 / 16 / 7 | 48 / 46 / 21 / 11 / 7 | 40 / 6 / 13 / 8 / 1 |
| Others | reactivation_with_purchase | varias | 23 | 9 | 14 |

**Controles**
- Duplicados de llave: 0 en las tres. `pos_ncro` tiene 1 fila por comercio.
- **ISR dentro de `pos_resales`: 0** (forzado por código y por test). En `pos_sales_sot` todo ISR es primera venta sin deal (`ncro-…`).
- KAM 2026: 262 filas = 262 deals Ganado 2026 del roster; ningún deal sin fila.
- Nulos en `pos_sales_sot`: 0 en `company_id`, `channel_attribution`, `purchase_type`, `pos_quantity`, `declared_sale_date`, `is_invoice_paid`, `is_form_completed`, `is_valid_purchase`. `country` 2; `seller_email` 975 (89 en 2026); `invoice_ids` 410; `effective_sale_date` 432 (31 en 2026).
- `pos_ncro`: `seller_email` nulo 847, sin factura 365, sin trx real 443, país nulo 2. `pos_resales`: `seller_email` nulo 128, sin factura 45 (todas KAM); 88 KAM y 185 Others.
- **Reactivaciones soft (`pos_quantity = 0`)**: 26, todas KAM y de jul–sep 2026. 24 válidas; 2 en curso con plazo 10-oct (149087, 501961). 25 de 26 tienen `is_invoice_paid = false`.
- **Fecha efectiva nula**: 432 filas, ninguna válida (0 válidas sin fecha efectiva). 28 filas con fecha efectiva anterior a la declarada; las 12 de 2026 son KAM.
- 51 filas con `history_mismatch = true` (45 son reventas KAM «Venta Nueva»).

## Qué confirma o contradice la reunión del 24-sep

Contraste contra [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]] (resumen de Notion, `verificado: false`).

| Punto de la reunión | Qué hacen el código y los datos | Veredicto |
|---|---|---|
| `POS Sales SOT` = `POS First Sales` ∪ `POS Resales` | `pos_sales_sot` = `pos_ncro` ∪ `pos_resales`. «First Sales» se llama `pos_ncro`; no existe `pos_first_sales`. | Confirma (cambia el nombre) |
| La consolidada filtra por factura pagada | No filtra: trae todos los eventos y los marca con `is_valid_purchase`. El filtro se aplica al consultar. | Matiz |
| `IsValidPurchase` combina condiciones según el tipo | new, adicional y reactivación con máquina = pagada + formulario; ISR = pagada; soft = 15 trx. El formulario no aparece en el resumen de la reunión. | Confirma y agrega el formulario |
| `InvalidReason`: `no invoice`, `no transactions`… | Valores reales: `no_invoice`, `invoice_not_paid`, `form_not_sent`, `form_not_completed`, `no_trx_pre_sep26_form_proxy`, `soft_reactivation_…`. No existe `no_transactions`: las trx solo cuentan como proxy de formulario antes de sep-26 y en las soft. | Parcial |
| `IsInvoicePaid` marca venta real | `is_invoice_paid` = alguna factura del evento pagada. Es necesaria, no suficiente. | Confirma |
| Fecha de primera venta: Deal > Factura > Transacción | `declared_sale_date` = deal si lo hay; si no, el evento **más temprano** entre factura y trx, no la factura primero. Hay primeras ventas fechadas por una trx anterior a la factura (528843, 531844). La fecha efectiva es el pago. | Parcial |
| Cantidad: trx = 1; la factura manda sobre el deal | `COALESCE(deal, factura, 1)`: **manda el deal**. 11 filas KAM difieren (518906: deal 2 / factura 1; 84364 y 388767: deal 1 / factura 3; 2590: 1 / 2). Trx = 1, sí. | Contradice |
| Deal ↔ factura a ~45 d hacia adelante | Ventana de ±45 d (también hacia atrás) y el deal absorbe todas las facturas de la ventana. | Parcial |
| Reactivación con máquina exige factura pagada | Sí, y además formulario. | Confirma |
| Soft: 15 trx hasta el día 10 del mes siguiente; `post_quantity = 0`, `IsInvoicePaid = false` | La columna es `pos_quantity`, en 0 en las 26. Las 15 trx solo se exigen desde el 1-sep-2026: 9 soft de jul–ago son válidas con menos (0 trx en 203659, 506857, 621, 161984, 190960, 338864 y 3451; 2 en 9223; 5 en 23445). 501961 tiene factura pagada. La fecha efectiva de una soft es la declarada, aunque llegue a la meta después. | Parcial |
| ISR vía propiedad «contra topos»; historial vía «Migration» | No implementado. ISR = usuario Chargebee que agregó el addon, si está en el equipo ISR/AE/MDR de `hubspot_mapping`. En 2026, 143 de 171 filas ISR son de barbara@, justo lo que la reunión dijo que no sirve. | Todavía no está |
| `POS Resales` no debería tener ISR | 0 filas ISR, forzado por código y por test. | Confirma |
| «Other» = primeras ventas históricas sin atribución | Others también incluye primeras ventas de 2026 sin deal ni addon ISR (84 filas NCRO en 2026, p. ej. 504406 y 535145) y todas las reventas sin deal (185). | Parcial |
| Tablas a nivel company, no location | No hay columna de location. | Confirma |
| Caso borde MX: compra convertida a arriendo | 557935 queda como una sola fila KAM (el deal absorbe las dos facturas). Hoy es inválida por `form_not_completed` (proxy MX). | Cubierto |

La reunión no menciona el corte `strict_rules_from = 2026-09-01`, que cambia las reglas desde septiembre (ver caso 6).

## Casos borde y dudas

1. **`addon_restart` cuenta cambios de plan y bajas como venta.** La regla dispara con cualquier baja de addon entre dos rentas: el `BETWEEN` incluye el día de la renta anterior y no exige un alta después de la baja. De las 74 filas `addon_restart`, ninguna sube la cantidad respecto de la renta anterior, 71 tienen actividad POS en los 35 d previos y 30 tienen solo baja, sin alta. La conversión arriendo → cuotas genera una o dos filas `pos_additional`: 494549 (jul y ago, ambas válidas); 373415, 453116, 520667, 11787 y 38388 (dos filas cada una); 511752. Y 75776 / 490058, que el PR #396 usaba como control de «swap no es venta», hoy tienen fila (inválida solo por `form_not_sent`). Bajas sin alta: 1566 (dos filas en mar-2026 tras quitar el arriendo el 1-mar) y 498146 (baja de 3 a 2 terminales el 17-jul, que genera una fila de 2 unidades el 31-jul). Antes de sep-2026 casi todas quedan válidas.
2. **`qty_up` guarda la cantidad total, no el incremento.** Las 21 filas suman 48 unidades; el incremento real contra la renta anterior es 24. Ej.: 32381 tiene deal en jun (1), `qty_up` el 20-ago (3) y `addon_restart` el 31-ago (3), 7 unidades para unos 3 terminales; 498146 tiene ISR 2 + `qty_up` 3 + `addon_restart` 2, 7 para 3.
3. **Mismas facturas en dos filas.** 19517: dos deals KAM (ago y sep-2024; 3 y 1 unidades) absorben las facturas 264985 y 275928, y las dos filas son válidas.
4. **`purchase_type = 'new'` no es solo NCRO.** Hay 45 reventas KAM con deal «Venta Nueva»; en 2026 son 9: 370509, 7743, 161934, 430053, 341790, 170863, 25280, 10119 y 487289. Al revés, deals «Reactivación» sobre comercios activos (471474 y 388767, con 1–2 días sin actividad) quedan como reactivación.
5. **ISR = quien cargó el addon.** 569393 aparece ISR por barbara@, no por el deal Sales ISR de monika@. 504406 y 535145 (ISR según HubSpot) siguen como Others sin vendedor. 339880 y 452621 son ISR sin factura en el NCRO: el addon se buscó contra la primera factura del comercio, que quedó fuera de la ventana.
6. **Corte de reglas el 1-sep-2026.** En septiembre suben las inválidas por formulario: CL 36 válidas de 54, MX 7 de 16. En MX las 7 son `form_not_completed` con factura pagada (529415, 458124, 557935, 574554, 580562, 544651, 450004) y dependen de que *Control terminales* esté al día. Septiembre no es comparable con agosto sin ajustar.
7. **Soft antes de septiembre.** 9 válidas con menos de 15 trx (ver tabla de la reunión). 501961 es soft con factura pagada (678653, 680248): ¿debería ser `reactivation_with_purchase`? Varias soft son de comercios activos (459098 y 120798, con 1 día sin actividad).
8. **Cantidad del deal vs factura**: 518906, 84364, 388767, 2590, 417441, 471474. Duda aparte: en líneas `smart-pos-cuotas`, ¿`item_qty` son terminales o número de cuotas? (32381 pasa de 1 terminal en arriendo a 3 en cuotas el mismo día).
9. **Primeras ventas por trx sin factura en 2026**: 461716, 456582, 492241, 506885, 74427 (262 trx y formulario enviado, sin factura) y 564233. Todas `no_invoice`.
10. **Ventas revertidas con factura pagada**: 562659 y 563154 siguen como ISR `new` válidas (subtarea 7 de [[Locations POS y reactivaciones]]).
11. **Fecha efectiva antes de la declarada**: 2590 (declarada 1-sep, pagada 21-ago), 541887, 557772. La `fecha_de_venta_pos` del deal a veces es el día 1 del mes o es posterior al pago.
12. **País**: 2 nulos, 1 Nicaragua y 1 Venezuela; un filtro `country IN ('CL','MX')` los deja fuera.

**Ajustes a lo anotado en [[20260910-01 Cuadre ventas POS ISR jul–sep 2026]]** (esa nota no se editó):
- «Cobertura desde el 12-jun-2026»: la tabla trae toda la historia y la primera fecha efectiva es el 12-jun-**2023**. Parece una confusión de año.
- «1.338 de 2.288 filas en Others (58%)»: al corte, Others son 1.525 de 2.292 (67%) en toda la historia y 166 de 599 (28%) en 2026. Las primeras ventas Others son 1.340, cerca del 1.338 citado.
- Primeras ventas CL, ISR + Others por fecha efectiva: jul 27, ago 46, sep 21 (antes 20).

**Preguntas para Ignacio**
1. ¿`addon_restart` debería exigir un alta después de la baja y excluir el mismo día de la renta anterior? Hoy infla `pos_additional` con swaps arriendo → cuotas y con bajas.
2. En `qty_up`, ¿`pos_quantity` debería ser el incremento y no el total?
3. Cantidad: ¿manda el deal (código) o la factura (reunión)?
4. Ventana deal ↔ factura: ¿±45 d o solo hacia adelante?
5. Soft: ¿es intencional que antes del 1-sep valgan sin 15 trx? ¿La fecha efectiva debería ser el día en que llegan a 15 y no la declarada?
6. ¿Cuándo entran «contra topos» y la planilla ISR, y cómo se reclasifican los Others de 2026?
7. MX: ¿*Control terminales* llega a tiempo como proxy del formulario? ¿Cuándo se ingesta *Status afiliación* de OEL?
8. ¿Qué hacer con 19517 (facturas compartidas) y con los deals «Venta Nueva» sobre comercios que ya tenían POS?

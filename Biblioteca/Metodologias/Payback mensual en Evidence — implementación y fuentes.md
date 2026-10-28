---
type: metodologia
temas: [cac, payments, pos, evidence, definiciones]
metricas: ["[[Payback (Mensual)]]"]
fuente: agendapro-dashboards-evidence · pages/acquisition/payback/index.md (lectura del 2026-10-21)
fecha: 2026-10
---

# Payback mensual en Evidence — implementación y fuentes

Cómo está implementado el payback mensual en Evidence, qué tabla alimenta cada término y qué defectos se conocen. La definición canónica está en [[Payback (Mensual)]] (skill `finance-metric-definitions`); el documento de referencia de Pablo Lucero es [Payback y Payback variable](https://app.notion.com/p/agendapro/Payback-y-Payback-variable-2a7ab34c233b8043a04ccfdbd63ec301) en Notion. Sin cifras: las cifras van en las tareas o análisis que las usan.

## Dónde vive
- Repo `agendapro-dashboards-evidence`, página `pages/acquisition/payback/index.md` → https://evidence.agendaprops.com/acquisition/payback/ («Payback — Monthly»). La página hermana `cohorts.md` («Payback Cohorts») es otro constructo (GM realizado por cohorte, con churn, sin toggle de descuento) y no es comparable.
- Test: `scripts/test/payback-window.test.mjs`, corre con `npm run test:sql`.
- Grano: mes × país (Chile, Mexico, Argentina, Colombia, Other), solo B2B3, más agregado compañía.

## Fórmula implementada
```
          Direct_acq + HQ_prorrateado + Descuento_ventana[-1,+2]
Payback = ─────────────────────────────────────────────────────────────
          New_MRR_SaaS_bruto × GM_SaaS  +  New_POS × ASP_Payments × GM_Payments
```
- **Total** usa numerador completo; **Variable** solo `Direct_acq` (+ descuento). CAC B2B3 = (directo + HQ) ÷ merchants nuevos B2B3, sin descuento.
- El descuento de mes 0 ya está dentro del New MRR bruto; en el numerador entra el de la ventana completa.

## Qué alimenta cada CTE de `payback_base`
| CTE | Fuente (extracción en `sources/dwh/…`) | Qué saca |
|---|---|---|
| `acq_raw` / `direct` / `hq` | `profit_and_loss`, `p_l = '3- Acquisition Costs'` | costos directos por `cost_center` (≠ HQ) y HQ; marketplace = `code = 'G1011'` o detalle con «marketplace» |
| `month_valid` | idem | solo meses con > USD 50k de costos de adquisición (descarta cargas parciales del P&L) |
| `nm` | `new_merchants_first_mrr`, `segment2 = 'B2B3'` | merchants nuevos, New MRR SaaS bruto (SW + AI) y descuento de mes 0, en el FX elegido |
| `gm` | `profit_and_loss`, revenue y service costs | GM SaaS = (SW + AI revenue + service costs sin Payment Processing) ÷ (SW + AI revenue); GM Payments = (Payments revenue + Payment Processing) ÷ Payments revenue. **Un % global por mes, igual para todos los países.** |
| `pay_rev_c` | `mrr_payments`, B2B3 | revenue de pagos por país |
| `mpos` | `pos_penetration_merchants` | merchants con POS activo (locations con GPV POS > 0 en el mes) → ASP Payments = revenue pagos ÷ merchants_pos |
| `pos` | `pos_sold_monthly` | **New POS = `SUM(pos_sold_isr)`** (PR #856, 28-oct-2026; antes `SUM(pos_sold)` = ISR + KAM, ver defecto 1) |
| `discw` | `mbr_discount_amount`, `first_customer_segment_2 = 'B2B3'` | descuento de la ventana [−1,+2], siempre en FX current |

## Toggles de la página
- **FX**: Variable (promedio del mes) / Constant (último mes cerrado) / Budget 2026.
- **Marketplace costs**: Exclude (default) / Include. Solo afecta el numerador.
- **Discount adjustment**: With discount (window) (default) / No adjustment.
- **Period**: Monthly / Trailing 3m (PR #837, mergeado oct-2026): en T3M el payback es razón de sumas de 3 meses, no promedio de razones.
- **Variable Payback by component** (PR #844, mergeado oct-2026): Sales + Marketing + Onboarding + descuento, cada uno ÷ el mismo denominador, así los segmentos suman el Payback Variable. Botones Last month / L3M / L12M propios.
- El Dropdown de mes abre en el último mes cerrado según los datos (arreglado en #844).

## La fuente `pos_sold_monthly` (ISR vs KAM)
`sources/dwh/payments_weekly/pos_sold_monthly.sql`, sobre `dwh.pos_sales2`, solo desde 2026-01:
- **Dedup** (acordado con Miguel, jul-2026): 1 venta por empresa + día; unidades = la línea de mayor `item_qty` (setup + arriendo y las líneas OEL :1/:2/:3 de la misma terminal se contaban 2–3 veces).
- **Canal** (acordado jul-2026): KAM si el agente está en la lista KAM (rocio, janet, jaime, fernandobarra, alvaro, tabata); si no, por recencia desde `companies.created_at`: ventas desde jun-2026 → ISR si ≤ 30 días, ventas hasta may-2026 → ISR si ≤ 90 días; sin fechas → `is_new_merchant` original.
- Columnas: `pos_sold` (total), `pos_sold_isr`, `pos_sold_kam`, por `date_month`, `country_group`, `channel`.

### 2026-10-28 · Reglas vigentes (PR #860, sobre `dwh.pos_sales_sot`)
Lo de arriba es la regla legacy, reemplazada el 28-oct-2026. Desde entonces la extracción lee la SoT de Ignacio ([[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]]):
- Cuenta una venta si `is_valid_purchase` (factura pagada; formulario POS también desde el 1-sep-2026), `purchase_type <> 'reactivation'`, `pos_quantity > 0` y `effective_sale_date >= 2024-01-01` (el corte 2026-01 desapareció).
- Mes = `effective_sale_date`. Unidades = `pos_quantity`. Canal = `channel_attribution` del deal POS de HubSpot: ISR (merchant nuevo), KAM (base instalada), Others (venta válida sin deal).
- Columnas: las mismas más `pos_sold_others`. Others va en su fila y columna para que el total cuadre con la SoT, pero no entra en `pos_sold_isr` ni en `pos_sold_kam`: el payback lee solo ISR; `budget-2026-follow-up` lee ISR + KAM.
- `country_group`: Chile, Mexico, Argentina, Colombia, Otros (vocabulario del contrato; la SoT solo trae CL y MX).
- Test: `scripts/test/pos-sold-monthly-sot.test.mjs` (DuckDB sobre el bloque `-- rules:begin/end` del archivo).
- Caveat: el mes en curso llega con semanas de rezago (la venta se valida al pagarse la factura y al matchear el deal); `fintech/payments-sales` mezcla este real con el forecast legacy de `pos_sold_current.sql` hasta migrarla.

## Defectos conocidos (al 2026-10-21)
1. **KAM dentro de la cohorte nueva** (Pablo Lucero, 21-oct): `New_POS` suma ISR + KAM, pero los KAM se venden a la base. Infla el margen por merchant nuevo y achica el payback en Chile y México. Fix: `pos_sold_isr` en la CTE `pos` y corregir la nota de metodología de la página. Seguimiento en [[Payback Payments]]. **Resuelto el 28-oct-2026**: PR #856 mergeado (`SUM(pos_sold_isr)`).
2. **Término Payments = 0 antes de 2026**: la extracción parte en 2026-01 (observación del PR #844). **Resuelto el 28-oct-2026**: PR #860 extiende la serie a 2024-01 sobre la SoT.
3. **Nota de metodología desactualizada**: dice que la ventana confiable parte en 2024-10 y que 2024-07/08/09 están incompletos. **Confirmado el 23-oct-2026** sobre el caché reextraído el 22-oct: `payback_base` emite los 33 meses 2024-01 → 2026-09 para los 5 países, y los costos de adquisición de todo 2024 superan el umbral de USD 50k/mes (mínimo USD 133k en ene-2024; jul–sep 2024 entre 173k y 185k). La frase de la página describe una carga parcial que ya se completó. Falta solo corregir el texto (cambio de documentación, sin lógica).
4. El GM es un % global por mes: la página no distingue margen por país.

## Relación con otros reportes
- El KR de payback del OKR y el MBR usan la misma metodología de Notion. La extracción legacy `okr_payback` (CAC ÷ ARPU) quedó en desuso.
- El board book (Key Metrics) calcula su propio payback. Desde el PR #832 (mergeado 28-oct-2026) lee `country_pos_sot` sobre `pos_sales_sot` con **solo New (ISR)** en el término Payments, la misma regla que esta página; Key Metrics «New POS» = New + Stock y Others solo en el slide 8. Impacto estimado antes del merge: [[20261006-01 Board book sobre pos_sales_sot — impacto en ventas, attachment y payback]]; cifras finales en [[Payback Payments]] (nota del 23-oct).
- El payback de **pagos a la base (flujo KAM)** y el de **IA** no están en esta página; son construcciones aparte: [[Payback Payments]] y [[Payback AI]].

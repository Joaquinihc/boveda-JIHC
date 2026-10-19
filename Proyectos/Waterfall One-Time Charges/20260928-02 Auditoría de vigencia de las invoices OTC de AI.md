---
type: analisis
proyecto: "[[_Waterfall One-Time Charges]]"
estado: activo
fecha: 2026-09-28
temas: [otc, ai, dwh]
metricas: []
---

# Auditoría de vigencia de las invoices OTC de AI

**Pregunta**: ¿están bien configurados `date_from` y `date_to` (período hábil del add-on) en las líneas one-time de AI en Chargebee? Es la fuente de la regla n−1 ([[20260928-01 OTC de AI como MRR — regla n−1]]).

## Conclusión
> Bien en la mayoría, mal en el 19 %: de 158 líneas pagadas (122 compañías, datos al 2026-09-28), 34 tienen el período mal cargado, concentradas en agosto y septiembre, cuando empezó el modelo de cobrar la mensualidad como OTC. Hay que corregirlas en Chargebee antes del cierre y agregar un test de alerta en dbt.

## Método
Unnest de `line_items` de `staging.stg_chargebee__invoices` (status `paid`), catálogo `staging.stg_chargebee__addons` con `period_unit = 'not_applicable'`, `is_test_company = false`. Período en meses calendario UTC. Cruce con cupones 100 % de `dwh.chargebee_subscriptions_snapshot_coupons` y con el primer MRR del producto en `dwh.mrr_ai`.

## Evidencia
| Período declarado | Líneas | % |
|---|---:|---:|
| ≈1 mes (25–49 días) | 86 | 54 % |
| ≈2 meses (50–95 días) | 41 | 26 % |
| Sin período (`date_from = date_to`) | 17 | 11 % |
| Parcial (<25 días) | 13 | 8 % |
| Un año (381 días) | 1 | 1 % |

- Correcto: 138 de 141 líneas con período arrancan el mismo día de la factura; los cobros mes a mes (3252, 7045, 13455, 20701, 1966) son contiguos.
- Sin período (facturas): 570583 (69190, dos líneas), 580215, 581062, 586266, 652556, 655031, 657348, 657977, 660841, 666464, 667404, 667405, 675641, 678612, 682678, 683294. Agosto 8, septiembre 4, mayo–julio 0.
- Parciales de 2 a 18 días: 381847 (8 d), 359106 (16 d, ARS 557.912), 15739 (18 d), 100902, 160890, 23435, 3886, 120202 (2 d) y 1988 (4 d, CLP 360.000).
- Retroactivas (inicio >1 mes antes de la factura): 450300, 160538, 1988.
- Duplicados y solapes: 461283 (dos facturas idénticas de CLP 180.000 el mismo día), 62008 (dos líneas mismo período, MXN 12.150 y 5.210), 196041 (solape de 25 días). Huecos: 15739 (9 días), 1988 (un mes).
- Anual: 1966 Clínica Talus, Sofia por CLP 999.000 con período 27-mar-26 → 12-abr-27 (es un plan anual facturado como one-time).
- Cupón vs período: solo 82 de 158 líneas traen `subscription_id` y 23 tienen cupón 100 %; en 19 coincide la duración. El cupón no sirve como respaldo del período: la fuente debe ser la línea.

## Caveats
- Test de gobernanza propuesto para `int_ai_one_time_charges`: alerta cuando una línea one-time AI pagada tenga período nulo, <25 días, >95 días, inicio >3 días antes de la factura, o el mismo período que otra línea de la misma compañía y producto. Hoy dispararía 34 veces: partir con aviso, no con bloqueo.
- Las 86 compañías-producto donde `mrr_ai` muestra MRR antes de que termine el período no son error de configuración: es el MRR forward-looking del add-on con cupón.

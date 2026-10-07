---
type: analisis
proyecto: "[[_Ad hoc]]"
estado: activo
fecha: 2026-10-07
temas: [leads, adquisicion, marketing, cac, ventas, definiciones, board]
metricas: ["[[Lead]]", "[[CVR - tasa de conversión]]", "[[New Merchants (nuevas altas)]]"]
---

# Leads con cuenta creada en t0 por país y canal

**Pregunta**: ¿Están llegando leads de peor calidad? En concreto: ¿bajó el % de leads que llegan con cuenta AgendaPro creada al momento del lead (hipótesis de Matías Ulloa), por país y canal? ¿Convierte mejor el lead que llega con cuenta? ¿Cómo se ven el gasto, el CPL y el CAC por canal (criterios de Carlos Rojas)? Tarea: [[Cruce de leads con cuenta creada por país y canal]] (T-085). Origen: [[2026-10-01 Leads con cuenta creada y PACE (Matías)]].

**Reporte interactivo**: https://claude.ai/artifact/YAEH1sxBmFakK7QkuSoqhG (privado; compartido con personas puntuales). Tiene selectores de país, canal, periodo y ventana de creación de cuenta.

## Conclusión
> Corte: DWH al 5-oct-2026. Universo: leads efectivos del Board Book, segmento B2B3, ene-2025 a sep-2026.
> - **Sí cayó el % de leads con cuenta en t0**: 4 países core 56,7% (H1-2026) → 50,8% (Q3-2026), septiembre 48%. Chile 62,0% → 52,4%. El crecimiento de leads desde julio es todo del lado "sin cuenta".
> - **La caída es sobre todo mix**: Paid Social (formularios de Meta, ~20% con cuenta) pasó de 38% a 45% de los leads B2B3 → 2/3 de la caída total (−6,7 pp = mix −4,5 + tasa −2,2). **Chile es tasa** (−12,5 pp = mix −2,1 + tasa −10,4): Paid Social 30% → 19% y Paid Search Brand 86% → 58% con cuenta.
> - **No es un efecto de medir en t0**: ampliando la ventana a 30 días el nivel sube ~5 pp, pero la caída H1 → Q3 se mantiene (~6 pp) en todas las ventanas.
> - **La cuenta en t0 predice conversión solo en Paid Social**: 16,4% vs 6,3% a 90 días (2,6×), transversal a los 4 mercados. En Directo, Orgánico y Search los leads sin cuenta convierten igual o más (piden demo y Ventas les crea la cuenta).
> - **Más leads, menos cierres**: Q3 vs H1-2026, leads +13% a +23% al mes en los 4 países core; New Merchants B2B3 −8% a −25%.
> - **Chile no es solo calidad**: la conversión a 30 días de sus leads con cuenta cayó de 17–21% (ene–may 2026) a 9–13% (jun–ago), y en Directo la conversión del mes bajó de 23% a 16% con el % con cuenta estable. Apunta a capacidad o proceso comercial.
> - **El CPL engaña**: medios pagados Q2 → Q3-2026: CPL USD 30 → 29, costo por lead con cuenta 63 → 72 (+14%), CAC de medios 262 → 328 (+25%; ~310 corrigiendo el P&L de sep de México).

## Método
- **Lead efectivo** = `leads.sql` de Evidence (Board Book): cada evento de `dwh.fct_lead_conversions` (incluye reconversiones) × `staging.stg_hubspot__contacts`. Excluye fuente OFFLINE/UNKNOWN/vacía. En B2B3: Chile exige owner, México excluye sector "otro". B2B3 = regla residual de `numemployees`. Cuadra con el Board Book (diferencias ≤5 leads/mes por duplicados del mes).
- **Cuenta en t0**: `split_part(external_id,'.',1)::bigint = dwh.companies.company_id` (calza 100%; `company_id` de HubSpot no sirve). "Con cuenta en t0" = `companies.created_at` (UTC→Santiago, día) ≤ `conversion_date` (ya en hora Chile). **t0 es día calendario, no ventana de 24 h.** En leads nuevos, 97% de las cuentas del mismo día nacen ±5 min del lead (el signup crea el lead).
- **Ventanas** (t0, 1, 7, 30, 90 días, sin límite): el % solo usa leads que ya cumplieron la ventana al 5-oct, y se omite si menos de la mitad del periodo la cumplió.
- **Conversión por lead**: primer pago (`dwh.first_paying_date.start_date_pago_mx`) de la cuenta del lead entre t0 y t0+30/90 días. Es analítica; **no** es el CVR del Board Book (NM del periodo ÷ leads del periodo), que también se muestra.
- **Gasto, CPL y CAC** como la lámina 2.2 del Board Book: P&L G1005 (Paid Social) + G1006 (Paid Search, separado Brand/Non-Brand por gasto en `staging.stg_google_ads__campaign`) + TikTok prorrateado por leads Paid Social. NM B2B3 en sedes (`country_new_merchants`), atribuidos por el primer contacto de HubSpot (`company_marketing_source`). CAC de medios ≠ Fully Loaded CAC.
- SQL reproducible: [[SQL — Leads efectivos × cuenta creada (t0 y ventanas)]].

## Evidencia
**% de leads B2B3 efectivos con cuenta, 4 países core**

| Ventana | H1-2025 | H1-2026 | Q3-2026 |
|---|---|---|---|
| En t0 | 60,5% | 56,7% | 50,8% |
| Hasta 7 días | 62,9% | 59,5% | 53,1% |
| Hasta 30 días | 65,9% | 61,5% | 55,5% |
| Paid Social, en t0 | 25,5% | 23,9% | 19,3% |
| Resto de canales, en t0 | 81,6% | 82,2% | 82,3% |

**Conversión a 90 días por lead, cohortes ene-2025 a jun-2026 (todos los países)**

| Canal | Con cuenta en t0 | Sin cuenta en t0 |
|---|---|---|
| Paid Social | 16,4% | 6,3% |
| Paid Search Brand | 18,3% | 16,4% |
| Paid Search Non-Brand | 7,5% | 9,5% |
| Directo | 11,6% | 18,1% |
| Orgánico + otros | 12,3% | 15,8% |

Según cuándo se creó la cuenta: previa 11%, mismo día 13%, 1–7 días después 62% (la crea Ventas al cerrar), 8+ días 28%. "Cuenta previa" = 95% reconversiones; 62% con cuentas de más de 6 meses; 10% ya había pagado antes.

**Desasignación en Chile** (ver [[Desasignación real de leads — spec del fix]]): hasta may-26 quedaban sin owner 25–108 leads B2B3/mes (promedio 59), fuera del conteo de efectivos; desde jun-26, 2–11. ~70% de ellos tenía cuenta en t0. Suma ~11% de leads efectivos en Chile (hasta 18% en abril): infla el denominador del CVR de Chile pero no explica la caída del % con cuenta.

**Medios pagados, Q2 → Q3-2026 (todos los países)**: gasto 214k → 235k; leads 7.147 → 8.204; % con cuenta 47% → 40%; NM B2B3 818 → 716. Paid Social: CPL 23 → 18, CAC 194 → 270. Chile Non-Brand: gasto 15,6k → 23,3k, NM 30 → 11 (CAC 521 → 2.118); Brand: gasto a la mitad con los mismos NM.

## Caveats
- `external_id` es el estado actual del contacto, no el histórico. Leads sin `external_id` no pueden convertir en esta medición (si el cierre quedó en otra cuenta no se ve; en MDR→AE ~28% de los NM B2B3 no se atribuyó a un lead).
- Reconversiones (15–20% de los leads): solo traen fecha, sin hora; el orden lead/cuenta del mismo día no se puede verificar. Con regla de hora exacta para leads nuevos, el % con cuenta baja ~1,3 pp (no medido mes a mes).
- Una misma conversión puede contarse en dos eventos de lead del mismo contacto (T1 y T2) — levantado por Matías el 6-oct.
- Owner de Chile y `hs_analytics_source` son campos vivos de HubSpot. NM se cuentan en sedes (una cadena genera saltos). "Orgánico + otros" absorbe los NM sin contacto en HubSpot.
- P&L de sep-2026 preliminar: Paid Search de México USD 35,6k vs 22,8k en Google Ads. Paid Search de HQ (USD 1,5k en Q3) sin asignar.
- **Abiertos** (no son subtareas de T-085): validar con Marketing el traslado Brand → Non-Brand de Chile en Q3 y el nombre de las campañas; revisar el P&L de sep de México; evaluar la regla mixta de t0; proponer ajustar la definición de lead efectivo de Chile (hoy depende del owner); Matías reportaba >50% → ~35% y el 6-oct ya trabajó con 57% → 51%.

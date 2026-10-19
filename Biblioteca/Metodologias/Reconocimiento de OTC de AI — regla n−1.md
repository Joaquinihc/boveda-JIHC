---
type: metodologia
temas: [otc, mrr, definiciones, ai]
actualizado: 2026-10-19
---

# Reconocimiento de OTC de AI — regla n−1

Propuesta (28-sep-2026, pendiente de decisión de Pablo Lucero) para reconocer los one-time charges (OTC) de Julia, Sofia y Agente AI como MRR del producto. Análisis con cifras: [[20260928-01 OTC de AI como MRR — regla n−1]].

## Por qué como MRR
Bajo NIIF 15 (B48–B51) un cobro único que no transfiere un servicio distinto de la suscripción y que el cliente no vuelve a pagar al renovar no es un fee de instalación: es la suscripción cobrada con otro instrumento. Los equipos verticales cobran el OTC al precio de la mensualidad por el período de instalación (dos semanas a un mes, a veces dos o tres meses como promoción) y después pasan al add-on recurrente. Ninguna empresa pública suma lo único al ARR/MRR ni divide el monto por el precio para decidir cuántos meses cubre: lo que decide el período es la línea de factura.

## La regla
1. Fuente: líneas one-time AI pagadas en `staging.stg_chargebee__invoices` (catálogo `staging.stg_chargebee__addons` con `period_unit = 'not_applicable'`), monto neto de `discount_amount`, CLP sin dividir y resto ÷ 100.
2. `n` = meses calendario UTC que toca `date_from → date_to` (fin exclusivo).
3. `n = 1`: todo el monto a ese mes. `n ≥ 2`: partes iguales en los primeros `n − 1` meses; el mes `n` queda para la suscripción regular. Sin período (`date_to ≤ date_from`): `n = 1`, mes de pago.
4. Convivencia con el MRR recurrente del mismo producto: en los meses retenidos manda el OTC mensual; desde el mes `n` manda el MRR neto. Nunca se suman (el add-on con cupón 100 % muestra MRR forward-looking antes de facturar). La variante "máximo" se evaluó y absorbe OTC en el MRR; no se recomienda.
5. Conservación: la suma del reparto por compañía × producto es igual al neto cobrado.
6. Grano compañía × producto: solo la mitad de las líneas trae `subscription_id`.

## Qué cambia en cada tabla (propuesta, no implementado)
- `int_ai_one_time_charges` (nuevo): una fila por línea con período, producto, neto, `n` y meses retenidos, más test de gobernanza (sin período, <25 d, >95 d, retroactiva >3 d, duplicada).
- `int_ai_one_time_schedule` (nuevo): reparto n−1 por compañía × producto × mes.
- `int_subscription_mrr_breakdown_changes`: columnas aditivas `otc_<producto>_mrr` y `addons_<producto>_mrr_adjusted`; categorías recalculadas sobre el ajustado.
- `revenue_waterfall_ai/_julia/_sofia/_agente_ai`: leen el ajustado; `new` cae en el primer mes retenido; el traspaso al recurrente es upgrade/downgrade solo si cambia el precio.
- `mrr_ai`: `mrr_<producto>` pasa al ajustado; se conserva `setup_charges` como caja informativa; se deprecan `mrr_plus_setup_charges` y `companies_with_setup_split`.
- Evidence: "AI Revenue" pasa a leer `mrr_usd`.

## Caja, devengo y MRR: tres series distintas
- Caja: OTC en el mes de pago (`mrr_ai.setup_charges` usa fecha de factura y bruto).
- Devengo contable (CFO, `revenue_recognition.sql`): prorrateo diario sobre `date_from → date_to`; sigue vigente para el P&L.
- MRR (gestión, waterfall): regla n−1. Las tres reconocen lo mismo al cierre del período; difieren mes a mes y en si suman o no el MRR forward-looking de la ventana.

## Cómo reproducir
Consultas de solo lectura con psycopg2 y el perfil `redshift` de `~/.dbt/profiles.yml` (`readonly=True`, `statement_timeout`), cuando el conector MCP del DWH no responde. Scripts de la sesión del 28-sep: `extract.py` (insumos), `otc_rule.py` (reglas, con tests), `waterfall.py` (reconstructor de categorías), `build_series.py`, `aggregate.py`, `build_report.py`. Viven en el scratchpad de Claude Code, no en el repo; el spec y el plan están en `agendapro-dbt-redshift/docs/superpowers/` (working tree).

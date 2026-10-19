---
type: analisis
proyecto: "[[_Waterfall One-Time Charges]]"
estado: activo
fecha: 2026-09-28
temas: [otc, mrr, waterfall, ai]
metricas: ["[[AI Revenue (AI MRR run-rate)]]"]
---

# OTC de AI como MRR — regla n−1

**Pregunta**: si los one-time charges (OTC) de Julia, Sofia y Agente AI se cobran al precio de la mensualidad por el período de instalación y después entra el recurrente, ¿cómo deben reconocerse en `mrr_ai` y en los waterfalls AI, y qué pasa con las cifras si se aplica la regla n−1?

Reporte para Pablo Lucero (artefacto privado): https://claude.ai/code/artifact/LeuoFwcCdJwbkznx4LamXs. Spec y plan de trabajo (working tree, sin commit) en `agendapro-dbt-redshift/docs/superpowers/specs/2026-09-28-otc-ai-regla-n-menos-1-design.md` y `docs/superpowers/plans/2026-09-28-otc-ai-regla-n-menos-1.md`.

## Conclusión
> El OTC de AI no es un servicio distinto de la suscripción (NIIF 15, B48–B51): cobra la mensualidad, cubre el mismo servicio y el cliente no vuelve a pagarlo al renovar. Por eso se contabiliza como MRR del producto, no como fee aparte, y se reparte con la regla **n−1**: n = meses calendario que toca el período `date_from → date_to`; con n = 1 todo a ese mes; con n ≥ 2 partes iguales en los primeros n−1 meses (el último queda para la suscripción regular); sin período, mes de pago. Dentro de la ventana manda el OTC y el MRR recurrente entra desde el mes n: nunca se suman. Método en [[Reconocimiento de OTC de AI — regla n−1]].
>
> Efecto (datos al 2026-09-28, USD corriente, todas las compañías con MRR AI): el AI revenue publicado baja entre 6 % y 14 % según el mes; en agosto 2026 de 57.086 a 48.936. No es revenue perdido: lo publicado suma caja y MRR forward-looking del add-on con cupón 100 % en el mismo mes. En el waterfall AI (mar–sep, compañía × producto) los `new` pasan de 123 a 158, los `upgrade` de 15 a 50 y los `churn` de 78 a 93; 33 compañías con OTC que hoy no existen para el waterfall pasan a verse. Decisión pendiente de Pablo Lucero.

## Método
- Fuentes de solo lectura: `staging.stg_chargebee__invoices` (líneas one-time, catálogo `period_unit = 'not_applicable'`), `dwh.mrr_ai`, `dwh.revenue_waterfall_julia/_sofia/_agente_ai/_ai`. Conexión psycopg2 con el perfil `redshift` de `~/.dbt/profiles.yml`, `readonly=True`. Sin modelos editados ni tablas creadas.
- Cálculo local en Python con pruebas unitarias (15 en verde): reglas de reparto (n−1 igual, n−1 por días, devengo diario del CFO, caja), reconstructor de categorías del waterfall (reproduce los 130 `new` del DWH y las categorías mar–sep antes de aplicar la regla), series y conservación del monto.
- Cuatro bases comparadas: publicado hoy (MRR + OTC caja), CFO (MRR + OTC devengado por días), n−1 con máximo contra el MRR, n−1 con OTC mandando en la ventana (recomendada). Universo principal: ~300 compañías con MRR AI por mes; la regla solo altera a las 122 que pagaron OTC.

## Evidencia
| Agosto 2026, todas las compañías, USD | Valor |
|---|---:|
| MRR AI recurrente (`dwh.mrr_ai`) | 45.704 |
| Publicado hoy: MRR + OTC en caja | 57.086 |
| Propuesta: n−1, OTC manda | 48.936 |
| Diferencia | −8.149 (−14,3 %) |

- 158 líneas OTC pagadas de 122 compañías, USD 41,2k (mar–sep). Distribución de n: 23 líneas con n = 1, 69 con n = 2, 58 con n = 3, 8 con n ≥ 4.
- Mensualidad equivalente bajo n−1 ÷ primer MRR recurrente del producto: mediana 0,9x; 68 de 120 dentro de ±25 %. Con n = 2 la mediana es 1,0x; con n = 3 es 0,5x (una mensualidad cubre dos meses de servicio), por eso el traspaso al recurrente aparece como upgrade en 34 de 76 traspasos.
- Descomposición de la brecha publicado − propuesta (122 compañías, mar–sep): 26,4k de MRR forward-looking suprimido dentro de la ventana (8,2k solo en agosto); 2,9k (7 % de lo cobrado) diferido a octubre o después por períodos de tres meses vendidos en ago–sep. A nivel de factura solo 9 de 135 períodos OTC se solapan con una línea recurrente pagada.
- Con "máximo" en lugar de "OTC manda" el MRR forward-looking absorbe USD 19,8k de OTC en 113 de 219 meses con OTC.
- Cuadratura: la serie de caja por fecha de factura y monto bruto reproduce `mrr_ai.setup_charges_usd` al peso en los siete meses; por fecha de pago y neto difiere en seis facturas y USD 463.
- Cambios propuestos por modelo: `int_ai_one_time_charges` e `int_ai_one_time_schedule` (nuevos), columnas aditivas en `int_subscription_mrr_breakdown_changes`, waterfalls AI y `mrr_ai` sobre el MRR ajustado (se deprecan `mrr_plus_setup_charges` y `companies_with_setup_split`), ~30 consultas de Evidence.

## Caveats
- Septiembre 2026 parcial; ventana de eventos feb–sep para evitar bordes.
- Decisiones abiertas para Pablo Lucero: reparto igual vs por días dentro de los meses retenidos (71 vs 50 upgrades); OTC manda vs máximo; restatement desde marzo 2026; el P&L contable sigue con el devengo diario del CFO (`revenue_recognition.sql`).
- 34 de 158 líneas tienen el período mal cargado en Chargebee: ver [[20260928-02 Auditoría de vigencia de las invoices OTC de AI]].
- Antecedente: [[20260903-01 Setup AI en el Waterfall — auditoría y benchmark]].

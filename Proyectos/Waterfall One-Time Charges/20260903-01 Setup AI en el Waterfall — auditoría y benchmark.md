---
type: analisis
proyecto: "[[_Waterfall One-Time Charges]]"
estado: activo
fecha: 2026-09-03
temas: [otc, waterfall, ai, benchmarks]
metricas: ["[[AI Revenue (AI MRR run-rate)]]"]
---

# Setup AI en el Waterfall — auditoría y benchmark

**Pregunta**: qué había implementado sobre los one-time charges de AI en dbt y Evidence, qué dicen los datos sobre su timing, y cómo tratan las empresas públicas los cobros únicos con período frente al revenue recurrente.

Reporte (artefacto, con §8 de benchmark y datos re-medidos el 21-sep): https://claude.ai/code/artifact/f01787d8-e5a3-4767-b8f0-dbcd9e25f462. Documento fuente: `agendapro-dbt-redshift/docs/AI_ONE_TIME_CHARGES_WATERFALL_PLAN.md` (working tree, sin commit).

## Conclusión
> En dbt no había nada implementado en la cadena del waterfall; solo `mrr_ai` con tres variantes de setup (caja, split 50/50, MRR + caja) y Evidence usando MRR + caja como "AI Revenue". La regla comercial del 1,5x no está en los datos (dos de cada tres setups son ≈1x el primer MRR). El `new` del waterfall no está atrasado: en cohortes maduras 22 de 52 coinciden con la primera caja recurrente y 10 se adelantan un mes por el flip del cupón 100 % (MRR forward-looking). Ninguna empresa pública suma un cobro único al ARR o MRR; el período de reconocimiento sale de si el servicio es distinto, no del monto. Este análisis fue superado por [[20260928-01 OTC de AI como MRR — regla n−1]], que cambia el diseño de "OTC separado en tres ejes" a "OTC como MRR".

## Método
Lectura de `int_subscription_mrr_breakdown(_changes)`, `mrr_ai`, los waterfalls AI y las páginas Board Book y P&L by vertical de Evidence; consultas de solo lectura a Redshift (3-sep y 21-sep); 10-K y 20-F de Toast, HubSpot, Olo, Paycom, Paylocity, RealPage, Shopify, Lightspeed, Squarespace, Kaltura, OneSpan, Nasdaq y Samsara; guías PwC 8.4 y NIIF 15 B48–B51.

## Evidencia
- Setup cobrado mar–sep 2026: USD 38,8k en 115 compañías; en agosto 24,9 % del MRR AI. 20 compañías con setup invisibles al waterfall; el split 50/50 generaba 17 filas en octubre.
- `mrr_ai` tomaba `amount` bruto sin restar `discount_amount`; Evidence contaba compañías con el split y revenue con caja íntegra; `revenue_recognition.sql` clasificaba agente_AI como SaaS.
- Benchmark: servicio distinto → se reconoce al prestarse (Toast al completar la instalación, HubSpot a medida que presta onboarding); no distinto con derecho material → vida esperada del cliente (Paycom 10 años, Paylocity hasta 24 meses, RealPage 3 años, Olo implementación compleja). Toast, Shopify, Squarespace, Kaltura, OneSpan, Nasdaq y Samsara excluyen lo único del ARR/MRR; el conteo de clientes se define por estado de servicio o uso (Toast Locations), nunca por revenue > 0.
- PR 387 (merge 21-sep) eliminó las columnas del split 50/50 de `mrr_ai`.

## Caveats
- Las cifras son del 21-sep; el DWH se reescribe hacia atrás (mayo pasó de 3.539 a 4.933 de setup entre el 3 y el 21-sep).
- Las listas de `company_id` por celda de cada tabla están en los desplegables del artefacto.

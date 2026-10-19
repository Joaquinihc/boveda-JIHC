---
type: analisis
proyecto: "[[_Ad hoc]]"
estado: activo
fecha: 2026-10-08
temas: [retencion, saas, revenue, board, definiciones]
metricas: ["[[Yield on CAC (annualized IRR on CAC)]]"]
---

# LTV self-serve por cohortes

**Pregunta**: ¿Cuánto revenue deja en promedio un cliente self-serve (B2B2 + B2C) durante su vida? Pedido de Emilio Latorre (Product, Core Experience) para cuantificar una mejora de conversión.

## Conclusión
> Un cliente self-serve paga en promedio **US$265 de revenue SaaS en 3 años** y **US$389 en 57 meses** (toda la historia observada). La **vida completa** es **~US$770**, pero la mitad es proyección (rango ~US$680–1.120 según el churn de largo plazo). Para board o inversionistas usar **margen bruto a 5 años: ~US$325** (57 de los 60 meses son observados), siempre junto a LTV/CAC y payback. No hay LTV oficial en la empresa ni CAC self-serve medido. Corte: DWH al 8-oct-2026, meses de vida hasta sep-2026.

## Método
- Fuentes: `dwh.nrr_cohorts` (MRR por compañía y mes de vida) + `dwh.companies.first_customer_segment_2` (self-serve = segmento inicial distinto de B2B3). Solo SaaS.
- Cohorte = primer mes pagando SaaS. Cohortes de ene-2022 a sep-2026: el self-serve existe en volumen desde 2022 (antes, menos de 10 por año), así que el máximo observable son 57 meses.
- Revenue por cliente adquirido en el mes t = MRR de las compañías activas ÷ todas las que entraron. El churn ya queda dentro (LTV = ARPA × vigencia): no se multiplica de nuevo por la vigencia.
- Cohortes encadenadas (chain-ladder): el mes 1 sale de todas las cohortes; para cada mes t se aplica la variación t → t+1 de todas las cohortes que llegaron a t+1. Los primeros meses reflejan a los clientes recientes y los meses largos salen de las cohortes 2022–23.
- Vida completa: después del mes 57 se supone churn constante de ~1,5% mensual (observado entre los meses 24 y 48) y ARPA plano de ~US$41 para el ~14% que sigue activo.
- Margen bruto: × ~80% (margen bruto SaaS de los últimos 12 meses, `finance.profit_and_loss` a FX constante).
- Método reutilizable, SQL y trampas: [[LTV por cohortes (nrr_cohorts) — método y trampas]].

## Evidencia
LTV acumulado por cliente que entra (US$):

| Mes | Revenue | Margen bruto | Siguen activos |
|---|---|---|---|
| 6 | 64 | 51 | 44% |
| 12 | 112 | 90 | 31% |
| 18 | 154 | 124 | 25% |
| 24 | 192 | 154 | 22% |
| 30 | 229 | 183 | 19% |
| **36** | **265** | **212** | **18%** |
| 42 | 301 | 241 | 16% |
| 48 | 336 | 269 | 15% |
| 54 | 370 | 297 | 14% |
| 57 (máximo observado) | 389 | 311 | 14% |
| 60 (5 años)* | 406 | 325 | 14% |
| Vida completa* | ~770 | ~616 | — |

\* Con proyección: 3 meses en el de 5 años; ~la mitad del valor en vida completa.

- Por año de vida: año 1 US$112 · año 2 US$80 · año 3 US$73. Vigencia promedio ~24 meses; la mitad se va antes del mes 6.
- Clientes que siguen activos en el mes 3 (ya pasaron la oferta de US$1): US$362 en sus siguientes 3 años. A esa altura ya se fue ~28% de los que entraron.
- Retención por año de cohorte (activas al mes 12): 2022 39% · 2023 40% · 2024 34% · 2025 28%. Las cohortes recientes se van más rápido; por eso se encadena en vez de promediar simple.
- Sensibilidad de la vida completa (revenue): churn de largo plazo 2% → ~US$680; 0,8% → ~US$1.120. A valor presente (10% anual, margen bruto) ~US$380.
- Versión anterior (cohortes oct-23 a abr-26 del reporte Yield on CAC, cortada a 36 meses): US$233 de margen a 36 meses, contra US$212 ahora.

**Para cuantificar una mejora de conversión** (lo que se le respondió a Emilio): multiplicar por clientes que pagan, no por leads ni registros. US$265 si la mejora es en el primer pago; US$362 si es en la permanencia después de la oferta. Son 3 años, no 1 (para el año en curso usar US$112). Mostrar un rango de 70–100% del valor, porque los clientes marginales pueden quedarse menos.

**Para el board**: margen bruto (no revenue), horizonte de 5 años (no vida completa), por cohortes y por segmento, siempre junto a LTV/CAC (referencia > 3x) y payback.

Hilo en Slack (DM con Emilio, 8-oct):
- [Conclusiones](https://agendapro.slack.com/archives/D0B83VD95JT/p1791490159222319) · [Metodología](https://agendapro.slack.com/archives/D0B83VD95JT/p1791490172001509)
- [Versión simple en revenue](https://agendapro.slack.com/archives/D0B83VD95JT/p1791491332631929) · [Versión para board](https://agendapro.slack.com/archives/D0B83VD95JT/p1791492203883459)
- [LTV cada 6 meses](https://agendapro.slack.com/archives/D0B83VD95JT/p1791492506284889) · [Cómo usarlo para cuantificar conversión](https://agendapro.slack.com/archives/D0B83VD95JT/p1791492836871369)

## Caveats
- Solo SaaS (sin Payments ni AI), en USD nominales (`mrr` de `dwh.nrr_cohorts`).
- La mitad del valor de vida completa es proyección. Los meses 50 a 57 se calculan con 8 cohortes o menos, todas de 2022.
- El CAC self-serve no está medido: el reporte Yield on CAC usa US$50 como supuesto. No presentar LTV/CAC self-serve al board con ese número.
- La oferta de US$1 se ve en los datos solo desde las cohortes de 2024 (meses 1–2 con ~US$5 promedio); en 2022–23 el mes 1 ya se cobra ~US$20, aunque la oferta se vende desde 2022. No verificado: puede que esos meses de prueba no quedaran registrados como pagados.
- El margen bruto SaaS es el mismo para todos los segmentos (el P&L no lo separa).
- Sin cambios en repos: calculado con consultas directas al DWH.

---
type: metodologia
temas: [retencion, saas, revenue, definiciones]
metricas: ["[[Yield on CAC (annualized IRR on CAC)]]"]
fuente: trabajo propio (DWH Redshift, 2026-10-08)
fecha: 2026-10
---

# LTV por cohortes (nrr_cohorts) — método y trampas

Cómo calcular el LTV de un segmento a partir de las cohortes de MRR del DWH. No es una definición oficial: no hay LTV oficial en la empresa (a oct-2026). Cifras y aplicación a self-serve en [[20261008-01 LTV self-serve por cohortes]].

## Método
1. **Cohorte** = primer mes pagando SaaS (`cohort_date` de `dwh.nrr_cohorts`). **Segmento** = segmento con el que entró (`dwh.companies.first_customer_segment_2`), no el actual.
2. **Revenue por cliente adquirido** en el mes de vida t = MRR de las compañías activas ÷ compañías que entraron. El churn queda dentro: LTV = ARPA × vigencia, no se multiplica de nuevo por la vigencia.
3. **Encadenar cohortes (chain-ladder)**: el mes 1 sale de todas las cohortes; para cada t se aplica la variación t → t+1 calculada con todas las cohortes que llegaron a t+1. Evita que el promedio simple mezcle cohortes viejas (meses largos) con recientes (meses cortos), lo que sobreestima cuando las recientes retienen peor.
4. **LTV a horizonte H** = suma de los meses 1..H. **Vida completa** = observado + cola proyectada: los activos del último mes observado × ARPA plano × (1 − c) / c, con c = churn mensual de largo plazo (geométrico sobre un tramo maduro, p. ej. meses 24–48).
5. **Margen bruto**: × margen bruto SaaS de los últimos 12 meses del P&L (`finance.profit_and_loss`, FX constante). El P&L no separa por segmento.

## SQL (Redshift)
Curva por mes de vida y variaciones t → t+1 para encadenar. Cambiar el filtro de segmento y el mes de corte.

```sql
WITH agg AS (
  SELECT c.cohort_date, c.period,
         COUNT(DISTINCT CASE WHEN c.is_active THEN c.company_id END) AS surv,
         SUM(c.mrr) AS rev
  FROM dwh.nrr_cohorts c
  JOIN dwh.companies co ON co.company_id = c.company_id
  WHERE COALESCE(co.first_customer_segment_2, 'x') <> 'B2B3'   -- self-serve
    AND c.cohort_date >= '2022-01-01'
    AND c.date_month  <= '2026-09-01'                           -- último mes cerrado
    AND c.period >= 1
  GROUP BY 1, 2
),
init AS (SELECT cohort_date, surv AS init FROM agg WHERE period = 1),
pair AS (   -- variación t -> t+1 sobre las cohortes que llegaron a t+1
  SELECT a.period,
         SUM(b.rev) / NULLIF(SUM(a.rev), 0)            AS r_rev,
         SUM(b.surv)::float / NULLIF(SUM(a.surv), 0)  AS r_surv,
         COUNT(*)                                      AS n_cohortes
  FROM agg a JOIN agg b ON b.cohort_date = a.cohort_date AND b.period = a.period + 1
  GROUP BY 1
),
simple AS (
  SELECT a.period,
         SUM(a.rev) / SUM(i.init)          AS rev_por_adquirido,
         SUM(a.surv)::float / SUM(i.init)  AS activos_pct
  FROM agg a JOIN init i ON i.cohort_date = a.cohort_date
  GROUP BY 1
)
SELECT s.period, s.rev_por_adquirido, s.activos_pct, p.r_rev, p.r_surv, p.n_cohortes
FROM simple s LEFT JOIN pair p ON p.period = s.period
ORDER BY 1;
```
El encadenado se hace afuera (planilla o script): `rev(1) = rev_por_adquirido(1)`; `rev(t+1) = rev(t) × r_rev(t)`; mismo esquema para `activos_pct` con `r_surv`.

## Trampas
- **El tope de 36 meses no es del DWH.** Viene de la extracción `tir_on_cac_cohorts` (`period BETWEEN 1 AND 36`) y de la ventana del reporte Yield on CAC (cohortes entre max−35 y max−5 meses). `dwh.nrr_cohorts` tiene cohortes desde 2015.
- **El self-serve existe en volumen desde 2022**: antes casi todo está clasificado como B2B3. El máximo observable para self-serve es ~57 meses (a sep-2026).
- **Filas de baja**: `dwh.nrr_cohorts` deja una fila con `is_active = false` y `mrr = 0` en el mes de baja. `tir_on_cac_cohorts` cuenta compañías sin filtrar `is_active`, así que sobreestima a los sobrevivientes (el revenue no se ve afectado). Usar `is_active` para supervivencia.
- **`period ≤ 0`**: hay filas de meses previos al primer pago. Filtrar `period >= 1`.
- **Oferta de US$1**: en los datos aparece como 2 meses de ingreso bajo solo desde las cohortes de 2024; en 2022–23 el mes 1 ya se cobra precio normal. No verificado por qué.
- **Moneda**: `mrr` está en USD nominales; `mrr_fixed` a FX fijo (no disponible para todas las monedas).
- **Para el board**: margen bruto, horizonte de 5 años, por segmento y junto a LTV/CAC y payback. Un LTV en revenue infla el LTV/CAC ~25%.

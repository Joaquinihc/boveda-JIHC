---
type: query
temas: [activacion, dwh, evidence]
lenguaje: sql (Redshift)
---

# SQL — Activación: reservas importadas y puente Evidence vs dbt

Queries usadas en [[20260930-01 Reservas importadas — impacto en activación y waterfalls]] y [[20261001-01 Activation Velocity — Evidence vs tablas dbt]]. Definiciones en [[Activación — fuentes y definiciones vigentes]].

> El MCP del DWH corta las consultas a ~30 s. Redshift recalcula cada CTE en cada referencia: hay que armar el cruce en **una sola pasada** (un `LEFT JOIN` por empresa y una sola agregación) y no con subconsultas escalares por columna.

## 1. Reservas importadas por empresa
Misma regla que `bookings_count_demand` en `fct_bookings_created_daily`. Los estados 1, 2, 3, 7 y 9 son los activos, como `booking_count_active` en `company_sales_days`.

```sql
SELECT l.company_id,
  COUNT(*) AS importadas_todas,
  SUM(CASE WHEN b.status_id IN (1,2,3,7,9) THEN 1 ELSE 0 END) AS importadas_activas,
  MIN(b.created_at)::date AS primera_carga, MAX(b.created_at)::date AS ultima_carga
FROM staging.stg_agendapro__bookings b
JOIN staging.stg_agendapro__locations l ON l.location_id = b.location_id
WHERE b.created_at >= '2025-12-01'
  AND (b.company_comment ILIKE 'Creada vía archivo .csv%'
       OR (b.creative_source IS NULL AND l.company_id = 514942))
GROUP BY 1 ORDER BY 2 DESC;
```

Ya con el PR 414 en producción, lo mismo sale directo del fact: `SUM(bookings_count - bookings_count_demand)` en `dwh.fct_bookings_created_daily`.

## 2. Puente Evidence → dbt (B2B3, Semana 9, una cohorte)
Cambiar `'202601'` por la cohorte. Columnas: `a` = Evidence, `b1` = todos los estados, `b` = conteo dbt, `c` = universo dbt con sedes de `merchants_segments`, `d` = tabla dbt. Sin clases: en las cohortes con fitness, sumar aparte las empresas que cruzan el umbral gracias a las clases.

```sql
WITH ev AS (
  SELECT ms.company_id, ms.start_date_pago_mx::date AS s, COALESCE(ms.plan_quantity,1)::int AS q
  FROM dwh.merchants_segments ms
  JOIN dwh.company_attribute_first_segment2 fs ON fs.company_id = ms.company_id
  WHERE fs.first_customer_segment_2 = 'B2B3' AND TO_CHAR(ms.start_date_pago_mx,'YYYYMM') = '202601'
    AND ms.company_id IN (SELECT company_id FROM dwh.mrr WHERE is_first_month_paying IS TRUE)
    AND ms.customer_id NOT IN (SELECT id FROM staging.stg_chargebee__customers WHERE is_test_company IS TRUE)
),
evc AS (
  SELECT e.company_id, e.q,
    SUM(CASE WHEN c.day::date <= e.s+62 THEN c.booking_count ELSE 0 END) AS w9a,
    SUM(CASE WHEN c.day::date <= e.s+62 THEN c.booking_count_all_bookings ELSE 0 END) AS w9t
  FROM ev e LEFT JOIN dwh.company_sales_days c
    ON c.company_id = e.company_id AND c.day >= '2026-01-01' AND c.day::date BETWEEN e.s AND e.s+89
  GROUP BY 1,2
),
dw AS (SELECT company_id_corr AS company_id, initial_plan_quantity AS qd, cantidad_bookings_a_ese_corte AS bkw
       FROM dwh.activation_velocity_weekly_report
       WHERE nombre_corte='Semana 9' AND customer_segment='B2B3' AND month_paid_corr='202601'),
u AS (SELECT COALESCE(e.company_id, d.company_id) AS company_id, e.q, e.w9a, e.w9t, d.qd, d.bkw
      FROM evc e FULL OUTER JOIN dw d ON d.company_id = e.company_id),
uh AS (
  SELECT u.*, h.w9 AS hw9, m.mq FROM u
  LEFT JOIN (SELECT company_id, MAX(cantidad_bookings_a_ese_corte) AS w9 FROM dwh.int_company_booking_cohorts
             WHERE granularity='weekly' AND nombre_corte='Semana 9' GROUP BY 1) h ON h.company_id = u.company_id
  LEFT JOIN (SELECT company_id, MAX(COALESCE(plan_quantity,1)) AS mq FROM dwh.merchants_segments GROUP BY 1) m
    ON m.company_id = u.company_id
)
SELECT SUM(q) AS base_ev,
  SUM(CASE WHEN w9a >= 100 THEN q ELSE 0 END) AS a,
  SUM(CASE WHEN w9t >= 100 THEN q ELSE 0 END) AS b1,
  SUM(CASE WHEN COALESCE(hw9,0) >= 100 THEN q ELSE 0 END) AS b,
  SUM(CASE WHEN qd IS NOT NULL THEN COALESCE(mq,qd) END) AS base_c,
  SUM(CASE WHEN qd IS NOT NULL AND bkw >= 100 THEN COALESCE(mq,qd) ELSE 0 END) AS c,
  SUM(qd) AS base_d, SUM(CASE WHEN bkw >= 100 THEN qd ELSE 0 END) AS d
FROM uh;
```

---
type: query
temas: [payments, revenue]
metricas: ["[[Payments Revenue (MRR Payments)]]"]
lenguaje: sql (Redshift)
---

> Query original: `REDSHIFT payment_processing_revenue_and_MRR.sql` (copia literal).

```sql
----------------------------------------------------
------------------ REVENUE FINAL -------------------
----------------------------------------------------
-- MRR Chile
select month,
       currency_code,
       sum(mrr)::integer        as mrr,
       sum(mrr_usd)             as mrr_usd,
       count(venues)            as subscriptions,
       sum(venues)              as merchants
from dwh.mrr
where currency_code = 'CLP'
  and (date_month < '2026-04-01' or is_active_and_paid = true)
group by 1,2
order by month asc;

-- MRR Colombia
select month,
       currency_code,
       sum(mrr)::integer        as mrr,
       sum(mrr_usd)             as mrr_usd,
       count(venues)            as subscriptions,
       sum(venues)              as merchants
from dwh.mrr
where currency_code = 'COP'
  and (date_month < '2026-04-01' or is_active_and_paid = true)
group by 1,2
order by month asc;

-- MRR México
select month,
       currency_code,
       sum(mrr)                 as mrr,
       sum(mrr_usd)             as mrr_usd,
       count(venues)            as subscriptions,
       sum(venues)              as merchants
from dwh.mrr
where currency_code = 'MXN'
  and (date_month < '2026-04-01' or is_active_and_paid = true)
group by 1,2
order by month asc;

-- MRR Argentina
select month,
       currency_code,
       sum(mrr)                 as mrr,
       sum(mrr_usd)             as mrr_usd,
       count(venues)            as subscriptions,
       sum(venues)              as merchants
from dwh.mrr
where currency_code = 'ARS'
  and (date_month < '2026-04-01' or is_active_and_paid = true)
group by 1,2
order by month asc;

-- MRR Otros Latam
select month,
       sum(mrr_usd)             as mrr_usd,
       count(venues)            as subscriptions,
       sum(venues)              as merchants
from dwh.mrr
where currency_code not in ('CLP', 'COP', 'ARS', 'MXN')
  and (date_month < '2026-04-01' or is_active_and_paid = true)
group by 1
order by month asc;

-- MRR Total
select month,
       sum(mrr_usd)             as mrr_usd,
       count(venues)            as subscriptions,
       sum(venues)              as merchants
from dwh.mrr
where (date_month < '2026-04-01' or is_active_and_paid = true)
group by 1
order by month asc;



-- Conciliación al último día del mes
SELECT *
FROM staging.stg_propay_conciliation__haulmer_transactions
ORDER BY transaction_date DESC
LIMIT 100;


-- Payment processing fee - Chile
select month,
       sum(fee)::int                                              as fee,
       (sum(fee) - sum(cost))::int                                as tr
from dwh.company_sales_months
where company_id is not null
  and country_name = 'Chile'
group by 1
order by 1 desc;

-- Payment processing fee - Colombia
select month,
       sum(fee)::int                                              as fee,
       (sum(fee) - sum(cost))::int                                as tr,
       round((sum(fee) / sum(gpv))::decimal, 4) * 10000           as fee_bps,
       round(((sum(fee) - sum(cost)) / sum(gpv))::decimal, 4) * 10000 as take_rate_bps
from dwh.company_sales_months
where company_id is not null
  and gpv <> 0
  and country_name = 'Colombia'
group by month
order by 1 desc;

-- Payment processing fee - Mexico
select month,
       sum(fee)::int                                              as fee
from dwh.company_sales_months
where country_name = 'México'
group by 1
order by 1 desc;

-- Payment processing fee - Argentina
select month,
       sum(fee)::int                                              as fee,
       (sum(fee) - sum(cost))::int                                as tr,
       round((sum(fee) / sum(gpv))::decimal, 4) * 10000           as fee_bps,
       round(((sum(fee) - sum(cost)) / sum(gpv))::decimal, 4) * 10000 as take_rate_bps
from dwh.company_sales_months
where gpv > 0
  and company_id is not null
  and country_name = 'Argentina'
group by 1
order by 1 desc;

-- Payment processing fee - Other
select month,
       country_name,
       currency,
       sum(fee)::int                                              as fee,
       sum(fee_usd)::int                                          as fee_usd,
       sum(cost_usd)::int                                         as cost_usd,
       (sum(fee) - sum(cost))::int                                as tr,
       round((sum(fee) / sum(gpv))::decimal, 4) * 10000           as fee_bps,
       round(((sum(fee) - sum(cost)) / sum(gpv))::decimal, 4) * 10000 as take_rate_bps
from dwh.company_sales_months
where gpv > 0
  and company_id is not null
  and country_name not in ('Argentina', 'México', 'Chile', 'Colombia')
group by 1,2,3
order by 1 desc;




--------------------------------
-- Revenue ARRIENDO Y VENTA POS
--------------------------------
WITH pos_addon_catalog AS (
    -- Catálogo dinámico (igual que B) — solo se usa para el branch ONE-TIME
    SELECT
        a.id AS addon_id,
        a.currency_code AS currency,
        CASE a.currency_code
            WHEN 'CLP' THEN 'CL' WHEN 'MXN' THEN 'MX' WHEN 'COP' THEN 'CO'
            WHEN 'PEN' THEN 'PE' WHEN 'ARS' THEN 'AR' WHEN 'BRL' THEN 'BR'
            WHEN 'USD' THEN 'US' ELSE a.currency_code END AS country,
        CASE WHEN a.period_unit IN ('month','year') THEN 'recurring' ELSE 'one_time' END AS category,
        a.period_unit, a.period
    FROM staging.stg_chargebee__addons a
    WHERE a.id ILIKE '%pos%' OR a.id ILIKE '%terminal%' OR a.id ILIKE '%getnet%'
       OR a.id ILIKE '%netpay%' OR a.id ILIKE '%oel%'
),
-- ============================================================
-- RECURRENTES: ahora desde int_subscription_mrr_breakdown.
-- mrr_pos_rent_sale ya viene NETO de cupones/descuentos (item + invoice,
-- con overflow) y prorrateado mensual — exactamente como A.
-- Los ~9 cupones POS (8 smart-pos-arriendo_2025-cl, 1 terminal-oel-renta_2025-mx)
-- reducen el monto vía la lógica de descuentos del breakdown.
-- ============================================================
recurrente_rows AS (
    SELECT
        b.month::date AS month,
        CASE b.currency_code WHEN 'CLP' THEN 'CL' WHEN 'MXN' THEN 'MX' END AS country,
        b.currency_code AS currency,
        'recurring' AS category,
        b.mrr_pos_rent_sale::double precision AS amount_local_monthly
    FROM dwh.int_subscription_mrr_breakdown b
    WHERE b.mrr_pos_rent_sale > 0
      AND b.currency_code IN ('CLP', 'MXN')
),
-- ============================================================
-- ONE-TIME: sin cambios (invoices.line_items ya trae discount_amount restado)
-- ============================================================
inv AS (
    SELECT id, DATE_TRUNC('month', date)::date AS month, currency_code,
           JSON_PARSE(line_items) AS items_super
    FROM staging.stg_chargebee__invoices
    WHERE status = 'paid' AND line_items IS NOT NULL AND line_items <> ''
      AND (line_items ILIKE '%pos%' OR line_items ILIKE '%terminal%' OR line_items ILIKE '%getnet%'
        OR line_items ILIKE '%netpay%' OR line_items ILIKE '%oel%')
),
invoice_lines_exploded AS (
    SELECT i.month, i.currency_code,
           CAST(item.entity_id AS VARCHAR(200)) AS addon_id,
           CAST(item.entity_type AS VARCHAR(20)) AS entity_type,
           CASE WHEN i.currency_code='CLP'
                THEN CAST(item.amount AS double precision) - COALESCE(CAST(item.discount_amount AS double precision), 0)
                ELSE (CAST(item.amount AS double precision) - COALESCE(CAST(item.discount_amount AS double precision), 0)) / 100.0
           END AS amount_local
    FROM inv i, i.items_super AS item
),
one_time_rows AS (
    SELECT il.month, c.country, c.currency, c.category, il.amount_local AS amount_local_monthly
    FROM invoice_lines_exploded il
    INNER JOIN pos_addon_catalog c ON il.addon_id = c.addon_id AND c.category = 'one_time'
    WHERE il.entity_type = 'addon'
)
SELECT category, month, country, currency,
       ROUND(SUM(amount_local_monthly)::numeric, 0) AS amount_local
FROM (SELECT * FROM recurrente_rows UNION ALL SELECT * FROM one_time_rows) all_rows
WHERE month >= '2025-01-01'
GROUP BY 1, 2, 3, 4
ORDER BY 1, 2, 3, 4;

--------------------------------
--------------------------------
--------------------------------
-- AI Revenue
--------------------------------
SELECT m.month,
       --m.country_group,
       SUM(m.mrr)              AS mrr_ai_local,
       SUM(m.mrr_usd)          AS mrr_ai_usd_current,
       SUM(m.mrr_usd_constant) AS mrr_ai_usd_constant
FROM dwh.mrr_ai m
GROUP BY m.month
--, m.country_group
ORDER BY m.month desc
--, m.country_group
;
--------------------------------
--------------------------------




SELECT m.month,
       m.country_group,
       SUM(m.mrr)              AS mrr_ai_local,
       SUM(m.mrr_usd)          AS mrr_ai_usd_current,
       SUM(m.mrr_usd_constant) AS mrr_ai_usd_constant
FROM dwh.mrr_ai m
GROUP BY m.month, m.country_group
ORDER BY m.month DESC, m.country_group;

--SELECT * FROM staging.stg_google_sheets__exchange_rate_monthly_average LIMIT 200;





-- Merchants & subscriptions totales
select month,
       count(1) as q,
       sum(plan_quantity) as plan_quantity_total
from staging.stg_chargebee__subscriptions_snapshot
where status in ('active', 'non_renewing')
  and mrr > 0
group by 1
order by 1 asc;


----------------------------------------------------
-- Costos Propay (Redshift: UNION ALL en vez de unnest)
----------------------------------------------------
WITH fx AS (
    SELECT month,
           country_name,
           SUM(fee) / NULLIF(SUM(fee_usd), 0) AS fx_rate
    FROM dwh.company_sales_months
    WHERE gpv > 0
    GROUP BY month, country_name
),
exploded AS (
    SELECT month, country_name, 'cl_web_dlocal' AS provider,
           COALESCE(fee_cl_web_dlocal_i_usd, 0)  AS fee_usd,
           COALESCE(cost_cl_web_dlocal_i_usd, 0) AS cost_usd
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_pos_klap',
           COALESCE(fee_cl_pos_klap_i_usd, 0),  COALESCE(cost_cl_pos_klap_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_web_pagofacil',
           COALESCE(fee_cl_web_pagofacil_i_usd, 0), COALESCE(cost_cl_web_pagofacil_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_pos_haulmer',
           COALESCE(fee_cl_pos_haulmer_i_usd, 0), COALESCE(cost_cl_pos_haulmer_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_pos_haulmer_e',
           COALESCE(fee_cl_pos_haulmer_e_usd, 0), COALESCE(cost_cl_pos_haulmer_e_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_pos_sumup',
           COALESCE(fee_cl_pos_sumup_i_usd, 0), COALESCE(cost_cl_pos_sumup_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_pos_sumup_e',
           COALESCE(fee_cl_pos_sumup_e_usd, 0), COALESCE(cost_cl_pos_sumup_e_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_web_mercadopago',
           COALESCE(fee_cl_web_mercadopago_i_usd, 0), COALESCE(cost_cl_web_mercadopago_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_web_webpay',
           COALESCE(fee_cl_web_webpay_i_usd, 0), COALESCE(cost_cl_web_webpay_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'cl_pos_tbk',
           COALESCE(fee_cl_pos_tbk_i_usd, 0), COALESCE(cost_cl_pos_tbk_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_pos_clip',
           COALESCE(fee_mx_pos_clip_i_usd, 0), COALESCE(cost_mx_pos_clip_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_web_dlocal',
           COALESCE(fee_mx_web_dlocal_i_usd, 0), COALESCE(cost_mx_web_dlocal_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_pos_netpay1',
           COALESCE(fee_mx_pos_netpay1_i_usd, 0), COALESCE(cost_mx_pos_netpay1_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_pos_netpay2',
           COALESCE(fee_mx_pos_netpay2_i_usd, 0), COALESCE(cost_mx_pos_netpay2_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_pos_getnet',
           COALESCE(fee_mx_pos_getnet_i_usd, 0), COALESCE(cost_mx_pos_getnet_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_web_redpay',
           COALESCE(fee_mx_web_redpay_i_usd, 0), COALESCE(cost_mx_web_redpay_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_web_mercadopago',
           COALESCE(fee_mx_web_mercadopago_i_usd, 0), COALESCE(cost_mx_web_mercadopago_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_pos_oel',
           COALESCE(fee_mx_pos_oel_i_usd, 0), COALESCE(cost_mx_pos_oel_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'mx_pos_oel_e',
           COALESCE(fee_mx_pos_oel_e_usd, 0), COALESCE(cost_mx_pos_oel_e_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'co_web_mercadopago',
           COALESCE(fee_co_web_mercadopago_i_usd, 0), COALESCE(cost_co_web_mercadopago_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'ar_web_mercadopago',
           COALESCE(fee_ar_web_mercadopago_i_usd, 0), COALESCE(cost_ar_web_mercadopago_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'uy_web_mercadopago',
           COALESCE(fee_uy_web_mercadopago_i_usd, 0), COALESCE(cost_uy_web_mercadopago_i_usd, 0)
    FROM dwh.company_sales_months WHERE gpv > 0
    UNION ALL SELECT month, country_name, 'pe_aggregate',
           CASE WHEN country_name = 'Perú' THEN COALESCE(fee_usd, 0)  ELSE 0 END,
           CASE WHEN country_name = 'Perú' THEN COALESCE(cost_usd, 0) ELSE 0 END
    FROM dwh.company_sales_months WHERE gpv > 0
)
SELECT e.month,
       e.country_name,
       CASE
           WHEN e.provider LIKE '%haulmer%'     THEN 'haulmer'
           WHEN e.provider LIKE '%sumup%'       THEN 'sumup'
           WHEN e.provider LIKE '%klap%'        THEN 'klap'
           WHEN e.provider LIKE '%tbk%'         THEN 'tbk'
           WHEN e.provider LIKE '%dlocal%'      THEN 'dlocal'
           WHEN e.provider LIKE '%pagofacil%'   THEN 'pagofacil'
           WHEN e.provider LIKE '%mercadopago%' THEN 'mercadopago'
           WHEN e.provider LIKE '%webpay%'      THEN 'webpay'
           WHEN e.provider LIKE '%clip%'        THEN 'clip'
           WHEN e.provider LIKE '%getnet%'      THEN 'getnet'
           WHEN e.provider LIKE '%netpay%'      THEN 'netpay'
           WHEN e.provider LIKE '%oel%'         THEN 'oel'
           WHEN e.provider LIKE '%redpay%'      THEN 'redpay'
           WHEN e.provider = 'pe_aggregate'     THEN 'peru'
           ELSE e.provider
       END AS provider_group,

       ROUND(SUM(e.fee_usd)::numeric, 0)                                          AS fee_usd,
       ROUND(SUM(e.fee_usd * fx.fx_rate)::numeric, 0)                             AS fee_local_currency,
       ROUND(SUM(e.cost_usd)::numeric, 0)                                         AS cost_usd,
       ROUND(SUM(e.cost_usd * fx.fx_rate)::numeric, 0)                            AS cost_local_currency,
       ROUND((SUM(e.fee_usd) - SUM(e.cost_usd))::numeric, 0)                      AS net_margin_usd,
       ROUND(((SUM(e.fee_usd) - SUM(e.cost_usd)) * AVG(fx.fx_rate))::numeric, 0)  AS net_margin_local,
       ROUND(AVG(fx.fx_rate)::numeric, 2)                                         AS fx_rate
FROM exploded e
LEFT JOIN fx
       ON e.month = fx.month
      AND e.country_name = fx.country_name
WHERE e.fee_usd > 0 OR e.cost_usd > 0
GROUP BY e.month, e.country_name, provider_group
ORDER BY e.month DESC, e.country_name, provider_group;


SELECT * FROM dwh.company_sales_months LIMIT 20;


----------------------------------------------------
-------------- MRR MONTHLY STATEMENT ---------------
----------------------------------------------------
select month, currency_code, sum(mrr)
from dwh.mrr
where currency_code = 'CLP'
group by 1,2
order by month asc;

select month, sum(mrr_usd)
from dwh.mrr
where currency_code not in ('CLP', 'COP', 'ARS', 'MXN')
  and first_paying_month is not null
group by 1
order by month asc;

select currency_code, sum(mrr)
from dwh.mrr
where month = '202209'
group by 1
order by currency_code asc;

select month, currency_code, sum(mrr), sum(mrr_usd)
from dwh.mrr
where currency_code = 'ARS'
group by 1,2
order by month asc;


----------------------------------------------------
----- MRR CON EMPALME PRE-CORRECCION DWH.MRR -------
----------------------------------------------------
-- Desde 2021-06
select month,
       sum(case when status = 'active' then mrr
                else plan_amount end) as value,
       count(1) as q,
       sum(plan_quantity) as plan_quantity_total
from staging.stg_chargebee__subscriptions_snapshot
where currency_code = 'ARS'
  and status in ('active', 'non_renewing')
  and mrr > 0
group by 1
order by 1 asc;

-- mrr customer level Chile 2022-09
select *
from staging.stg_chargebee__subscriptions_snapshot
where currency_code = 'CLP'
  and status in ('active', 'non_renewing')
  and month = '2022-09-01';

-- Antes 2021-05
select month,
       currency_code,
       sum(mrr),
       sum(mrr_usd),
       count(1) as q,
       sum(venues)
from dwh.mrr
where currency_code = 'CLP'
  and month <= '202105'
group by 1,2
order by month asc;

select *
from dwh.mrr
where currency_code = 'CLP'
  and month = '202105';

select * from dwh.merchants_segments limit 20;
select * from staging.stg_agendapro__bookings limit 20;


-- Detalle por proveedor
select month,
       country_name,
       round(avg(conversion_market)::decimal, 2)             as rate_market,
       coalesce(sum(fee_cl_pos_haulmer_i_usd_m), 0)::int +
       coalesce(sum(fee_cl_pos_haulmer_e_usd_m), 0)::int     as fee_pos_haulmer,
       coalesce(sum(fee_cl_pos_tbk_i_usd_m), 0)::int         as fee_pos_tbk,
       coalesce(sum(fee_cl_pos_sumup_e_usd_m), 0)::int +
       coalesce(sum(fee_cl_pos_sumup_i_usd_m), 0)::int       as fee_pos_sumup,
       coalesce(sum(fee_mx_pos_getnet_i_usd_m), 0)::int      as fee_getnet,
       coalesce(sum(fee_mx_pos_netpay1_i_usd_m), 0)::int +
       coalesce(sum(fee_mx_pos_netpay2_i_usd_m), 0)::int     as fee_pos_netpay,
       coalesce(sum(fee_mx_pos_oel_i_usd_m), 0)::int +
       coalesce(sum(fee_mx_pos_oel_i_usd_m), 0)::int         as fee_pos_oel,
       coalesce(sum(fee_cl_web_webpay_i_usd_m), 0)::int      as fee_web_tbk,
       coalesce(sum(fee_cl_web_dlocal_i_usd_m), 0)::int +
       coalesce(sum(fee_mx_web_dlocal_i_usd_m), 0)::int      as fee_web_dlocal,
       coalesce(sum(fee_mx_web_redpay_i_usd_m), 0)::int      as fee_web_redpay,
       coalesce(sum(fee_ar_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(fee_cl_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(fee_co_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(fee_mx_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(fee_uy_web_mercadopago_i_usd_m), 0)::int as fee_web_mp,
       coalesce(sum(cost_cl_pos_haulmer_i_usd_m), 0)::int +
       coalesce(sum(cost_cl_pos_haulmer_e_usd_m), 0)::int    as cost_pos_haulmer,
       coalesce(sum(cost_cl_pos_tbk_i_usd_m), 0)::int        as cost_pos_tbk,
       coalesce(sum(cost_cl_pos_sumup_e_usd_m), 0)::int +
       coalesce(sum(cost_cl_pos_sumup_i_usd_m), 0)::int      as cost_pos_sumup,
       coalesce(sum(cost_mx_pos_getnet_i_usd_m), 0)::int     as cost_getnet,
       coalesce(sum(cost_mx_pos_netpay1_i_usd_m), 0)::int +
       coalesce(sum(cost_mx_pos_netpay2_i_usd_m), 0)::int    as cost_pos_netpay,
       coalesce(sum(cost_mx_pos_oel_i_usd_m), 0)::int +
       coalesce(sum(cost_mx_pos_oel_i_usd_m), 0)::int        as cost_pos_oel,
       coalesce(sum(cost_cl_web_webpay_i_usd_m), 0)::int     as cost_web_tbk,
       coalesce(sum(cost_cl_web_dlocal_i_usd_m), 0)::int +
       coalesce(sum(cost_mx_web_dlocal_i_usd_m), 0)::int     as cost_web_dlocal,
       coalesce(sum(cost_mx_web_redpay_i_usd_m), 0)::int     as cost_web_redpay,
       coalesce(sum(cost_ar_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(cost_cl_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(cost_co_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(cost_mx_web_mercadopago_i_usd_m), 0)::int +
       coalesce(sum(cost_uy_web_mercadopago_i_usd_m), 0)::int as cost_web_mp
from dwh.company_sales_months
where month >= '202301'
group by 1, 2
having sum(fee_usd_m) > 1;
```

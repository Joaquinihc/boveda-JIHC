---
type: query
temas: [leads, adquisicion, cac, dwh]
lenguaje: sql (Redshift)
---

# SQL — Leads efectivos × cuenta creada (t0 y ventanas)

Queries usadas en [[20261007-01 Leads con cuenta creada en t0 por país y canal]]. Universo = lead efectivo B2B3 del Board Book (`leads.sql` de Evidence). Definición del lead en [[Lead]].

> **Trampas de Redshift vistas en este análisis**: un `SELECT` con ~65 `SUM(CASE…)` concatenados (pivot por canal) supera el timeout del MCP (~30 s); agrupar por `mes, país, canal` con pocas sumas sí corre (~10 s). Las subconsultas correlacionadas (`EXISTS … x.conversion_date < l.t0`) no están soportadas, y un `JOIN` extra contra `fct_lead_conversions` para marcar reconversiones también se cae: usar un CTE `MIN(conversion_date)` por contacto. `percentile_cont` sobre el mismo cruce también lo hace caer.

## 1. Base: lead efectivo B2B3 × cuenta en t0 × conversión
- Llave lead → cuenta: `split_part(trim(external_id),'.',1)` numérico = `dwh.companies.company_id` (calza 100%). `stg_hubspot__contacts.company_id` es el id de HubSpot y no sirve.
- `conversion_date` ya viene en hora Chile; `companies.created_at` está en UTC.
- Con cuenta en t0 = buckets A + B.

```sql
with l as (
  select lc.contact_id, lc.conversion_date as t0,
    case c.country when 'Chile' then 'Chile' when 'Argentina' then 'Argentina'
         when 'Colombia' then 'Colombia' when 'México' then 'Mexico' when 'Mexico' then 'Mexico'
         else 'Other' end as country_group,
    case when upper(trim(coalesce(c.hs_analytics_source,''))) in ('DIRECT_TRAFFIC','DIRECT') then '4- Direct Traffic'
         when upper(trim(coalesce(c.hs_analytics_source,''))) in ('PAID_SOCIAL','PAID SOCIAL') then '2- Paid Social'
         when upper(trim(coalesce(c.hs_analytics_source,''))) in ('PAID_SEARCH','PAID SEARCH') then
           case when lower(coalesce(c.hs_analytics_source_data_1,'')) like '%brand%'
                then '3.1- Paid Search Brand' else '3.2- Paid Search Non-Brand' end
         else '1- Organic + Referral + Social' end as marketing_source,
    case when split_part(trim(c.external_id),'.',1) ~ '^[0-9]+$'
          and len(split_part(trim(c.external_id),'.',1)) <= 17
         then split_part(trim(c.external_id),'.',1)::bigint end as company_id
  from dwh.fct_lead_conversions lc
  join staging.stg_hubspot__contacts c on lc.contact_id = c.id
  where c.country is not null and c.recent_conversion_date is not null
    and lc.conversion_date >= '2025-01-01'
    -- is_effective (Board Book) + segmento B2B3 residual
    and upper(trim(coalesce(c.hs_analytics_source,''))) not in ('OFFLINE','UNKNOWN','')
    and coalesce(trim(c.numemployees),'') not in ('','1','No aplica','None','null','2')
    and not (c.country = 'Chile' and coalesce(trim(c.hubspot_owner_id),'') = '')
    and not (c.country in ('México','Mexico') and lower(coalesce(trim(c.sector_economico),'')) = 'otro')
)
select date_trunc('month', l.t0)::date as month, l.country_group, l.marketing_source,
  case when convert_timezone('UTC','America/Santiago', co.created_at)::date < l.t0 then 'A cuenta previa'
       when convert_timezone('UTC','America/Santiago', co.created_at)::date = l.t0 then 'B mismo día (signup)'
       when convert_timezone('UTC','America/Santiago', co.created_at)::date <= l.t0 + 7 then 'C creada 1-7 días después'
       when co.created_at is not null then 'D creada 8+ días después'
       else 'E sin cuenta' end as cuenta_t0,
  count(*) as leads,
  sum(case when p.start_date_pago_mx between l.t0 and l.t0 + 30 then 1 else 0 end) as conv_30d,
  sum(case when p.start_date_pago_mx between l.t0 and l.t0 + 90 then 1 else 0 end) as conv_90d
from l
left join dwh.companies co on co.company_id = l.company_id
left join dwh.first_paying_date p on p.company_id = l.company_id
group by 1, 2, 3, 4;
```

## 2. Ventanas de creación de cuenta (con madurez)
Paso 1: conteos sobre todos los leads (`dd` = días entre el lead y la cuenta; negativo = cuenta previa). Paso 2, solo para los meses recientes: cuántos leads ya cumplieron cada ventana a la fecha de corte. En el reporte, el % de una ventana usa solo leads maduros y se omite si menos de la mitad del periodo la cumplió.

```sql
-- Paso 1 (mismo CTE l de la sección 1, con códigos cortos de país/canal)
, z as (select to_char(l.t0,'YYMM') m, l.country_group, l.marketing_source,
          convert_timezone('UTC','America/Santiago',co.created_at)::date - l.t0 as dd
        from l left join dwh.companies co on co.company_id = l.company_id)
select m, country_group, marketing_source, count(*) leads,
  sum(case when dd <= 0  then 1 else 0 end) as en_t0,
  sum(case when dd <= 1  then 1 else 0 end) as hasta_1d,
  sum(case when dd <= 7  then 1 else 0 end) as hasta_7d,
  sum(case when dd <= 30 then 1 else 0 end) as hasta_30d,
  sum(case when dd <= 90 then 1 else 0 end) as hasta_90d,
  sum(case when dd is not null then 1 else 0 end) as sin_limite
from z group by 1, 2, 3;

-- Paso 2 (corte 2026-10-05): leads maduros por ventana, solo jul–sep 2026
select m, country_group, marketing_source,
  sum(case when t0 <= date '2026-09-28' then 1 else 0 end) as maduros_7d,
  sum(case when t0 <= date '2026-09-28' and dd <= 7 then 1 else 0 end) as con_cuenta_7d,
  sum(case when t0 <= date '2026-09-05' then 1 else 0 end) as maduros_30d,
  sum(case when t0 <= date '2026-09-05' and dd <= 30 then 1 else 0 end) as con_cuenta_30d,
  sum(case when t0 <= date '2026-07-07' then 1 else 0 end) as maduros_90d,
  sum(case when t0 <= date '2026-07-07' and dd <= 90 then 1 else 0 end) as con_cuenta_90d
from z where t0 >= '2026-07-01' group by 1, 2, 3;
```

## 3. Reconversión vs lead nuevo
```sql
with fc as (select contact_id, min(conversion_date) d0 from dwh.fct_lead_conversions group by 1)
-- en el CTE l: join fc on fc.contact_id = lc.contact_id  →  (lc.conversion_date = fc.d0) as is_first
```
Para leads nuevos se puede ordenar lead y cuenta del mismo día con hora: `c.new_create_date::timestamp` (UTC) vs `co.created_at`. Las reconversiones no traen hora.

## 4. Gasto por canal (P&L) y separación Brand / Non-Brand
```sql
select date_trunc('quarter', date)::date as q,
  case when detail ilike '%tik%tok%' then 'TIKTOK' else cost_center end as cc,
  case when detail ilike '%tik%tok%' then 'TikTok'
       when code = 'G1005' then 'Paid Social' when code = 'G1006' then 'Paid Search' end as canal,
  -sum(amount_usd) as usd
from finance.profit_and_loss
where (code in ('G1005','G1006') or (p_l_category = 'Marketing' and detail ilike '%tik%tok%'))
group by 1, 2, 3;

-- Proporción Brand del gasto en Google Ads (moneda local), por país y trimestre
select date_trunc('quarter', segments_date)::date q,
  case split_part(campaign_resource_name,'/',2) when '7396270341' then 'Mexico' when '1328378189' then 'Chile'
       when '6145732756' then 'Argentina' when '2193040376' then 'Colombia' else 'Other' end cg,
  sum(case when lower(campaign_name) like '%brand%' then metrics_cost_micros end)::float
    / nullif(sum(metrics_cost_micros),0) as brand_share
from staging.stg_google_ads__campaign
where campaign_status <> 'REMOVED'
  and split_part(campaign_resource_name,'/',2) not in ('1395975599','2835613588')  -- Marketplace
group by 1, 2;
```
TikTok (G1009) se reparte a Paid Social según los leads Paid Social B2B3 de cada país. Para validar el P&L contra la plataforma: `dwh.google_ads_spend_daily.cost_usd` por país y mes.

## 5. New Merchants B2B3 atribuidos al canal
Mismo universo que `country_new_merchants` / `new_merchants_detail` (sedes) + canal del primer contacto HubSpot de la cuenta (`company_marketing_source`).

```sql
with nm as (
  select date_trunc('month', ms.start_date_pago_mx)::date m, a3.cf_company_id::int company_id,
    case when ms.country in ('Mexico','México') then 'Mexico'
         when ms.country in ('Chile','Argentina','Colombia') then ms.country else 'Other' end cg,
    sum(ms.plan_quantity) nm
  from dwh.merchants_segments ms
  left join staging.stg_chargebee__customers a3 on a3.id = ms.customer_id
  left join dwh.companies c on c.company_id = a3.cf_company_id::int
  left join dwh.mrr mr on mr.company_id = a3.cf_company_id::int
  where coalesce(a3.is_test_company,false) = false and mr.is_first_month_paying
    and c.first_customer_segment_2 = 'B2B3'
  group by 1, 2, 3),
cms as (
  select cid, ch from (
    select split_part(trim(external_id),'.',1)::bigint cid,
      case when upper(trim(coalesce(hs_analytics_source,''))) in ('DIRECT_TRAFFIC','DIRECT') then '4- Direct Traffic'
           when upper(trim(coalesce(hs_analytics_source,''))) in ('PAID_SOCIAL','PAID SOCIAL') then '2- Paid Social'
           when upper(trim(coalesce(hs_analytics_source,''))) in ('PAID_SEARCH','PAID SEARCH') then
             case when lower(coalesce(hs_analytics_source_data_1,'')) like '%brand%'
                  then '3.1- Paid Search Brand' else '3.2- Paid Search Non-Brand' end
           else '1- Organic + Referral + Social' end ch,
      row_number() over (partition by split_part(trim(external_id),'.',1) order by createdate asc) rn
    from staging.stg_hubspot__contacts
    where split_part(trim(external_id),'.',1) ~ '^[0-9]+$' and len(split_part(trim(external_id),'.',1)) <= 17) t
  where rn = 1)
select m, cg, coalesce(cms.ch,'1- Organic + Referral + Social') canal, sum(nm) nm_b2b3
from nm left join cms on cms.cid = nm.company_id
group by 1, 2, 3;
```
Nota: el filtro `external_id NOT LIKE '%[^0-9]%'` de `company_marketing_source.sql` en Evidence es sintaxis de SQL Server y en Redshift no filtra nada; usar `~ '^[0-9]+$'`.

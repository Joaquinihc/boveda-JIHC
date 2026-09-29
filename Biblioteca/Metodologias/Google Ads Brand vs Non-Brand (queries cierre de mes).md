---
type: metodologia
temas: [marketing, google-ads, cierre-mensual]
fuente: trabajo propio
fecha: 2026
---

# Google Ads: Brand vs Non-Brand — Queries para Cierre de Mes

Documento para el equipo de Finanzas con las queries necesarias para separar Brand vs Non-Brand en Google Ads: gasto, leads, merchants y MRR.

---

## 1. Definicion de Brand vs Non-Brand

La clasificacion se hace por **pattern matching sobre el nombre de la campana**:

```
Brand:     campaign_name ILIKE '%brand%'
Non-Brand: todo lo demas
```

### Campanas Brand actuales

| Campana | Pais |
|---------|------|
| `900_a_agendapro_mexico_branding` | Mexico |
| `900_a_agendapro_chile_branding` | Chile |
| `1200_Agendapro_cl_brand - Chile` | Chile |
| `900_argentina_branding` | Argentina |
| `1300_Agendapro_ar_brand - Argentina` | Argentina |
| `900_a_agendapro_colombia_branding` | Colombia |
| `900_a_agendapro_peru_branding` | Peru |
| `900_agendapro_branding` | Latam |
| `900_agendapro_uruguay_branding` | Uruguay |
| `901_agendapro_costarica_branding` | Costa Rica |
| `900_a_agendapro_panama_branding` | Panama |
| `900_a_agendapro_españa_branding` | Espana |
| `1100_a_agendapro_mexico_branding_RMKT` | Mexico (RMKT) |
| `1100_a_agendapro_partner_regional_demo_mx_brand - Mexico` | Mexico (Partner) |

### Mapeo de cuentas Google Ads a pais

| customer_id | Pais | Moneda |
|-------------|------|--------|
| 7396270341 | Mexico | MXN |
| 1328378189 | Chile | CLP |
| 6145732756 | Argentina | ARS |
| 2193040376 | Colombia | COP |
| 8660066686 | Peru | PEN |
| 6910919582 | Costa Rica | USD |
| 1668531839 | Uruguay | USD |
| 5823823870 | Panama | USD |

**Excluir cuentas Marketplace**: `1395975599`, `2835613588`

---

## 2. Gasto Brand vs Non-Brand (moneda local + USD)

**Tabla fuente**: `staging.google_ads__campaign` (ejecutar en `postgres_dwh`)

**Parametros**: Cambiar `'2026-03-01'` y `'2026-04-01'` por el mes que se necesite.

```sql
WITH spend_raw AS (
    SELECT 
        CASE WHEN LOWER(c.campaign_name) LIKE '%brand%' 
             THEN 'Brand' ELSE 'Non-Brand' 
        END AS campaign_type,
        CASE 
            WHEN cid = '7396270341' THEN 'Mexico'
            WHEN cid = '1328378189' THEN 'Chile'
            WHEN cid = '6145732756' THEN 'Argentina'
            WHEN cid = '2193040376' THEN 'Colombia'
            WHEN cid = '8660066686' THEN 'Peru'
            WHEN cid = '6910919582' THEN 'Costa Rica'
            WHEN cid = '1668531839' THEN 'Uruguay'
            WHEN cid = '5823823870' THEN 'Panama'
            ELSE 'Otros'
        END AS country,
        CASE 
            WHEN cid = '7396270341' THEN 'MXN'
            WHEN cid = '1328378189' THEN 'CLP'
            WHEN cid = '6145732756' THEN 'ARS'
            WHEN cid = '2193040376' THEN 'COP'
            WHEN cid = '8660066686' THEN 'PEN'
            ELSE 'USD'
        END AS currency,
        c.campaign_name,
        SUM(c.metrics_cost_micros) / 1000000.0 AS spend_local
    FROM staging.google_ads__campaign c,
         LATERAL (SELECT SPLIT_PART(c.campaign_resource_name, '/', 2) AS cid) x
    WHERE c.segments_date >= '2026-03-01'          -- << CAMBIAR MES
      AND c.segments_date < '2026-04-01'           -- << CAMBIAR MES
      AND c.campaign_status != 'REMOVED'
      AND SPLIT_PART(c.campaign_resource_name, '/', 2) 
          NOT IN ('1395975599', '2835613588')
    GROUP BY 1, 2, 3, 4
)
SELECT 
    campaign_type,
    country,
    currency,
    ROUND(SUM(spend_local)::numeric, 2) AS spend_local
FROM spend_raw
GROUP BY 1, 2, 3
ORDER BY country, campaign_type;
```

### Variante con conversion a USD (usando FX del DWH)

Para tener el gasto en USD, unir con la tabla de tipo de cambio:

```sql
WITH spend_raw AS (
    SELECT 
        CASE WHEN LOWER(c.campaign_name) LIKE '%brand%' 
             THEN 'Brand' ELSE 'Non-Brand' 
        END AS campaign_type,
        CASE 
            WHEN cid = '7396270341' THEN 'Mexico'
            WHEN cid = '1328378189' THEN 'Chile'
            WHEN cid = '6145732756' THEN 'Argentina'
            WHEN cid = '2193040376' THEN 'Colombia'
            WHEN cid = '8660066686' THEN 'Peru'
            WHEN cid = '6910919582' THEN 'Costa Rica'
            WHEN cid = '1668531839' THEN 'Uruguay'
            WHEN cid = '5823823870' THEN 'Panama'
            ELSE 'Otros'
        END AS country,
        CASE 
            WHEN cid = '7396270341' THEN 'MXN'
            WHEN cid = '1328378189' THEN 'CLP'
            WHEN cid = '6145732756' THEN 'ARS'
            WHEN cid = '2193040376' THEN 'COP'
            WHEN cid = '8660066686' THEN 'PEN'
            ELSE 'USD'
        END AS currency,
        c.segments_date::date AS fecha,
        SUM(c.metrics_cost_micros) / 1000000.0 AS spend_local
    FROM staging.google_ads__campaign c,
         LATERAL (SELECT SPLIT_PART(c.campaign_resource_name, '/', 2) AS cid) x
    WHERE c.segments_date >= '2026-03-01'          -- << CAMBIAR MES
      AND c.segments_date < '2026-04-01'           -- << CAMBIAR MES
      AND c.campaign_status != 'REMOVED'
      AND SPLIT_PART(c.campaign_resource_name, '/', 2) 
          NOT IN ('1395975599', '2835613588')
    GROUP BY 1, 2, 3, 4
),
fx AS (
    SELECT 
        date_monday::date AS week_start,
        usd_clp::numeric,
        COALESCE(usd_cop::numeric, 4200) AS usd_cop,
        usd_mxn::numeric,
        usd_ars::numeric,
        COALESCE(usd_pen::numeric, 3.7) AS usd_pen
    FROM dwh.stg_google_sheets__exchange_rate_weekly_closing
),
spend_with_fx AS (
    SELECT 
        s.*,
        CASE 
            WHEN s.currency = 'CLP' THEN s.spend_local / f.usd_clp
            WHEN s.currency = 'MXN' THEN s.spend_local / f.usd_mxn
            WHEN s.currency = 'ARS' THEN s.spend_local / f.usd_ars
            WHEN s.currency = 'COP' THEN s.spend_local / f.usd_cop
            WHEN s.currency = 'PEN' THEN s.spend_local / f.usd_pen
            ELSE s.spend_local  -- ya es USD
        END AS spend_usd
    FROM spend_raw s
    LEFT JOIN fx f ON f.week_start = (
        SELECT MAX(week_start) FROM fx WHERE week_start <= s.fecha
    )
)
SELECT 
    campaign_type,
    country,
    currency,
    ROUND(SUM(spend_local)::numeric, 2) AS spend_local,
    ROUND(SUM(spend_usd)::numeric, 2) AS spend_usd
FROM spend_with_fx
GROUP BY 1, 2, 3
ORDER BY country, campaign_type;
```

### Variante desglosada por campana

Para ver el detalle campana por campana dentro de brand/non-brand:

```sql
SELECT 
    CASE WHEN LOWER(c.campaign_name) LIKE '%brand%' 
         THEN 'Brand' ELSE 'Non-Brand' 
    END AS campaign_type,
    CASE 
        WHEN cid = '7396270341' THEN 'Mexico'
        WHEN cid = '1328378189' THEN 'Chile'
        WHEN cid = '6145732756' THEN 'Argentina'
        WHEN cid = '2193040376' THEN 'Colombia'
        ELSE 'Otros'
    END AS country,
    c.campaign_name,
    ROUND(SUM(c.metrics_cost_micros) / 1000000.0, 2) AS spend_local,
    SUM(c.metrics_impressions)::bigint AS impressions,
    SUM(c.metrics_clicks)::bigint AS clicks
FROM staging.google_ads__campaign c,
     LATERAL (SELECT SPLIT_PART(c.campaign_resource_name, '/', 2) AS cid) x
WHERE c.segments_date >= '2026-03-01'
  AND c.segments_date < '2026-04-01'
  AND c.campaign_status != 'REMOVED'
  AND SPLIT_PART(c.campaign_resource_name, '/', 2) 
      NOT IN ('1395975599', '2835613588')
GROUP BY 1, 2, 3
ORDER BY country, campaign_type, spend_local DESC;
```

---

## 3. Leads Brand vs Non-Brand

**Tablas fuente**: `dwh.fct_lead_conversions` + `dwh.stg_hubspot__contacts` (ejecutar en `postgres_dwh`)

La clasificacion de leads como Brand/Non-Brand se hace a traves del campo `hs_analytics_source_data_1` de HubSpot, que contiene el nombre de la campana de origen.

```sql
SELECT 
    CASE WHEN LOWER(c.hs_analytics_source_data_1) LIKE '%brand%' 
         THEN 'Brand' ELSE 'Non-Brand' 
    END AS campaign_type,
    c.country,
    COUNT(*) AS total_leads,
    COUNT(*) FILTER (
        WHERE UPPER(TRIM(COALESCE(c.numemployees, ''))) IN 
              ('3-5','5-16','6-15','16','16+','+16','5-25','25-50','50-100','1000+','6+')
    ) AS leads_b2b3_plus
FROM dwh.fct_lead_conversions lc
INNER JOIN dwh.stg_hubspot__contacts c ON lc.contact_id = c.id
WHERE c.hs_analytics_source = 'PAID_SEARCH'
  AND c.hsa_cam IS NOT NULL
  AND c.country IN ('Mexico', 'Chile', 'Argentina', 'Colombia')
  AND lc.conversion_date >= '2026-03-01'           -- << CAMBIAR MES
  AND lc.conversion_date < '2026-04-01'            -- << CAMBIAR MES
GROUP BY 1, 2
ORDER BY 2, 1;
```

### Nota sobre `hs_analytics_source_data_1`

Este campo de HubSpot almacena el nombre de la campana de Google Ads que genero el lead. Tiene un fill rate de ~99% para leads de PAID_SEARCH. Funciona porque Google Ads pasa el `campaign_name` a HubSpot via el tracking `hsa_cam`.

---

## 4. Merchants Brand vs Non-Brand (con MRR)

Esta es la query mas importante para Finanzas. Conecta la cadena completa:

**Google Ads campana -> Lead HubSpot -> Company -> Merchant activo pagando -> MRR USD**

**Tablas fuente**: `dwh.stg_hubspot__contacts` + `dwh.mrr` (ejecutar en `postgres_dwh`)

```sql
WITH lead_merchants AS (
    SELECT DISTINCT
        CASE WHEN LOWER(c.hs_analytics_source_data_1) LIKE '%brand%' 
             THEN 'Brand' ELSE 'Non-Brand' 
        END AS campaign_type,
        c.country,
        SPLIT_PART(c.external_id, '.', 1)::bigint AS company_id
    FROM dwh.stg_hubspot__contacts c
    WHERE c.hs_analytics_source = 'PAID_SEARCH'
      AND c.hsa_cam IS NOT NULL
      AND c.country IN ('Mexico', 'Chile', 'Argentina', 'Colombia')
      AND c.external_id IS NOT NULL 
      AND c.external_id != ''
      AND c.external_id ~ '^\d'
      -- Sin filtro de fecha: todos los merchants que vinieron de Paid Search
),
merchant_mrr AS (
    SELECT company_id, mrr_usd
    FROM dwh.mrr
    WHERE date_month = (SELECT MAX(date_month) FROM dwh.mrr)
      AND is_active = true
      AND mrr > 0
)
SELECT 
    lm.campaign_type,
    lm.country,
    COUNT(DISTINCT lm.company_id) AS total_companies_originadas,
    COUNT(DISTINCT mm.company_id) AS merchants_activos_pagando,
    ROUND(SUM(mm.mrr_usd)::numeric, 2) AS total_mrr_usd,
    ROUND(AVG(mm.mrr_usd)::numeric, 2) AS avg_mrr_usd
FROM lead_merchants lm
LEFT JOIN merchant_mrr mm USING (company_id)
GROUP BY 1, 2
ORDER BY 2, 1;
```

### Variante: Merchants con segmento (B2B3+, B2C, B2B2)

Agrega el segmento del merchant usando `dwh.merchants_segments`:

```sql
WITH lead_merchants AS (
    SELECT DISTINCT
        CASE WHEN LOWER(c.hs_analytics_source_data_1) LIKE '%brand%' 
             THEN 'Brand' ELSE 'Non-Brand' 
        END AS campaign_type,
        c.country,
        SPLIT_PART(c.external_id, '.', 1)::bigint AS company_id
    FROM dwh.stg_hubspot__contacts c
    WHERE c.hs_analytics_source = 'PAID_SEARCH'
      AND c.hsa_cam IS NOT NULL
      AND c.country IN ('Mexico', 'Chile', 'Argentina', 'Colombia')
      AND c.external_id IS NOT NULL 
      AND c.external_id != ''
      AND c.external_id ~ '^\d'
)
SELECT 
    lm.campaign_type,
    lm.country,
    ms.segment2,
    COUNT(DISTINCT lm.company_id) FILTER (WHERE ms.paying = true) AS merchants_pagando,
    ROUND(SUM(CASE WHEN ms.paying = true THEN ms.mrr_usd ELSE 0 END)::numeric, 2) AS total_mrr_usd,
    ROUND(AVG(CASE WHEN ms.paying = true THEN ms.mrr_usd END)::numeric, 2) AS avg_mrr_usd
FROM lead_merchants lm
LEFT JOIN dwh.merchants_segments ms ON lm.company_id = ms.company_id
WHERE ms.active = true
GROUP BY 1, 2, 3
ORDER BY 2, 1, 3;
```

### Variante: Merchants con fecha de conversion (cohortes)

Para ver merchants por mes de adquisicion:

```sql
WITH lead_merchants AS (
    SELECT DISTINCT
        CASE WHEN LOWER(c.hs_analytics_source_data_1) LIKE '%brand%' 
             THEN 'Brand' ELSE 'Non-Brand' 
        END AS campaign_type,
        c.country,
        SPLIT_PART(c.external_id, '.', 1)::bigint AS company_id,
        c.recent_conversion_date::date AS conversion_date
    FROM dwh.stg_hubspot__contacts c
    WHERE c.hs_analytics_source = 'PAID_SEARCH'
      AND c.hsa_cam IS NOT NULL
      AND c.country IN ('Mexico', 'Chile', 'Argentina', 'Colombia')
      AND c.external_id IS NOT NULL 
      AND c.external_id != ''
      AND c.external_id ~ '^\d'
      AND c.recent_conversion_date >= '2025-01-01'
),
merchant_mrr AS (
    SELECT company_id, mrr_usd
    FROM dwh.mrr
    WHERE date_month = (SELECT MAX(date_month) FROM dwh.mrr)
      AND is_active = true
      AND mrr > 0
)
SELECT 
    lm.campaign_type,
    lm.country,
    TO_CHAR(lm.conversion_date, 'YYYY-MM') AS mes_conversion,
    COUNT(DISTINCT lm.company_id) AS leads_con_company,
    COUNT(DISTINCT mm.company_id) AS merchants_activos_hoy,
    ROUND(SUM(mm.mrr_usd)::numeric, 2) AS total_mrr_usd_actual
FROM lead_merchants lm
LEFT JOIN merchant_mrr mm USING (company_id)
GROUP BY 1, 2, 3
ORDER BY 2, 1, 3;
```

---

## 5. Query completa de cierre: Spend + Merchants + MRR unificado

Esta query une gasto y merchants en un solo resultado por pais y tipo brand/non-brand.

**Nota**: Se ejecutan como dos queries separadas porque el gasto viene de `staging.google_ads__campaign` y los merchants de `dwh.stg_hubspot__contacts` + `dwh.mrr`. Ambas se ejecutan en `postgres_dwh`.

### Query A: Gasto del mes

```sql
-- QUERY A: Gasto del mes en moneda local
SELECT 
    CASE WHEN LOWER(c.campaign_name) LIKE '%brand%' 
         THEN 'Brand' ELSE 'Non-Brand' 
    END AS campaign_type,
    CASE 
        WHEN cid = '7396270341' THEN 'Mexico'
        WHEN cid = '1328378189' THEN 'Chile'
        WHEN cid = '6145732756' THEN 'Argentina'
        WHEN cid = '2193040376' THEN 'Colombia'
        ELSE 'Otros'
    END AS country,
    ROUND(SUM(c.metrics_cost_micros) / 1000000.0, 2) AS spend_local,
    SUM(c.metrics_impressions)::bigint AS impressions,
    SUM(c.metrics_clicks)::bigint AS clicks
FROM staging.google_ads__campaign c,
     LATERAL (SELECT SPLIT_PART(c.campaign_resource_name, '/', 2) AS cid) x
WHERE c.segments_date >= '2026-03-01'              -- << CAMBIAR MES
  AND c.segments_date < '2026-04-01'               -- << CAMBIAR MES
  AND c.campaign_status != 'REMOVED'
  AND SPLIT_PART(c.campaign_resource_name, '/', 2) 
      NOT IN ('1395975599', '2835613588')
GROUP BY 1, 2
ORDER BY country, campaign_type;
```

### Query B: Merchants y MRR originados por Paid Search

```sql
-- QUERY B: Merchants activos pagando + MRR por tipo brand/non-brand
WITH lead_merchants AS (
    SELECT DISTINCT
        CASE WHEN LOWER(c.hs_analytics_source_data_1) LIKE '%brand%' 
             THEN 'Brand' ELSE 'Non-Brand' 
        END AS campaign_type,
        c.country,
        SPLIT_PART(c.external_id, '.', 1)::bigint AS company_id
    FROM dwh.stg_hubspot__contacts c
    WHERE c.hs_analytics_source = 'PAID_SEARCH'
      AND c.hsa_cam IS NOT NULL
      AND c.country IN ('Mexico', 'Chile', 'Argentina', 'Colombia')
      AND c.external_id IS NOT NULL 
      AND c.external_id != ''
      AND c.external_id ~ '^\d'
),
merchant_mrr AS (
    SELECT company_id, mrr_usd
    FROM dwh.mrr
    WHERE date_month = (SELECT MAX(date_month) FROM dwh.mrr)
      AND is_active = true
      AND mrr > 0
)
SELECT 
    lm.campaign_type,
    lm.country,
    COUNT(DISTINCT lm.company_id) AS total_companies,
    COUNT(DISTINCT mm.company_id) AS merchants_activos_pagando,
    ROUND(SUM(mm.mrr_usd)::numeric, 2) AS total_mrr_usd,
    ROUND(AVG(mm.mrr_usd)::numeric, 2) AS avg_mrr_usd
FROM lead_merchants lm
LEFT JOIN merchant_mrr mm USING (company_id)
GROUP BY 1, 2
ORDER BY 2, 1;
```

---

## 6. Cadena de datos (data lineage)

```
Google Ads Campaign                    HubSpot Contact                    Company/Merchant
========================               ====================               ===================
staging.google_ads__campaign    -->    dwh.stg_hubspot__contacts    -->   dwh.mrr
  campaign_name (brand flag)           hsa_cam = campaign_id              company_id
  metrics_cost_micros (spend)          hs_analytics_source_data_1         mrr_usd
  campaign_resource_name (pais)        external_id = company_id           is_active
                                       country (pais del lead)            
                                       numemployees (segmento)            dwh.merchants_segments
                                                                           segment2 (B2B3+/B2C/B2B2)
                                                                           mrr_usd (snapshot actual)
                                                                           paying, active
```

### Join keys

| De | A | Join key |
|----|---|----------|
| Google Ads campaign | HubSpot contact | `campaign_id` = `hsa_cam` |
| HubSpot contact | Merchant/MRR | `SPLIT_PART(external_id, '.', 1)::bigint` = `company_id` |
| HubSpot contact | Merchants Segments | misma llave que arriba |

### Nota sobre `external_id`

El campo `external_id` de HubSpot a veces tiene formato decimal (ej: `352141.000000000`). Por eso se usa `SPLIT_PART(external_id, '.', 1)::bigint` para limpiarlo. Tambien se filtra con `external_id ~ '^\d'` para excluir valores no numericos.

---

## 7. Tablas y MCP a usar

| Query | Tabla principal | MCP recomendado |
|-------|----------------|-----------------|
| Gasto Google Ads | `staging.google_ads__campaign` | `postgres_dwh` |
| Leads | `dwh.fct_lead_conversions` + `dwh.stg_hubspot__contacts` | `postgres_dwh` |
| Merchants + MRR | `dwh.stg_hubspot__contacts` + `dwh.mrr` | `postgres_dwh` |
| Merchants + Segmento | `dwh.stg_hubspot__contacts` + `dwh.merchants_segments` | `postgres_dwh` |
| Tipo de cambio | `dwh.stg_google_sheets__exchange_rate_weekly_closing` | cualquiera |

**Importante**: La tabla `staging.google_ads__campaign` solo es accesible desde `postgres_dwh`, no desde `agendapro-dwh`.

---

## 8. Definiciones de negocio

| Concepto | Definicion |
|----------|-----------|
| **Brand** | Campana cuyo nombre contiene "brand" (case insensitive). Incluye "branding". |
| **Non-Brand** | Toda campana que NO es brand (verticales, competencia, PMax, etc.) |
| **B2B3+** | Leads con `numemployees` IN ('3-5','5-16','6-15','16','16+','+16','5-25','25-50','50-100','1000+','6+') |
| **Merchant activo pagando** | Company con `is_active = true` y `mrr > 0` en el ultimo mes de `dwh.mrr` |
| **MRR USD** | Monthly Recurring Revenue convertido a dolares. En `dwh.mrr` se usa FX del mes. En `dwh.merchants_segments` es snapshot actual. |
| **Spend local** | `metrics_cost_micros / 1,000,000` en la moneda de la cuenta de Google Ads |

---
type: metodologia
temas: [churn, b2b3]
metricas: ["[[Churn - Gross-Net × MRR-Logo]]"]
fuente: trabajo propio (repo Evidence)
fecha: 2026-09
proyecto: "[[_Churn y Graduación]]"
---

# Metodologia Churn Neto B2B3

Guia para reconstruir el reporte de Churn Neto del segmento B2B3 a partir de las tablas del DWH.

**Churn Neto = Churn Graduado + Churn Onboarding - Reactivaciones**

Incluye churn graduado, churn de onboarding y reactivaciones. El churn neto representa la perdida real de locations y MRR descontando las empresas que vuelven.

---

## 1. Tabla principal: `dwh.revenue_waterfall_software`

Los tres componentes del churn neto se extraen de la tabla **`dwh.revenue_waterfall_software`**, filtrando:

```sql
WHERE change_category IN ('churn_graduado', 'churn_onboarding', 'reactivation')
```

Esta tabla es un modelo dbt ubicado en `models/marts/revenue/revenue_waterfall_software.sql` y contiene el waterfall de ingresos de Software (plan + addons no-AI). Su fuente es `int_subscription_mrr_breakdown_changes`, que calcula cambios mensuales de MRR comparando snapshots mes a mes de `chargebee_subscriptions_snapshot`.

### Columnas clave de la tabla

| Columna | Descripcion |
|---|---|
| `subscription_id` | ID de suscripcion Chargebee |
| `month` | Primer dia del mes del evento (ej: `2026-04-01`) |
| `company_id` | ID de la compania en AgendaPro |
| `customer_id` | ID del customer en Chargebee |
| `currency_code` | Moneda de la suscripcion (CLP, COP, MXN, ARS, PEN, USD) |
| `item_id` | ID del plan Chargebee (ej: `premium2024-cl`, `pro2024q4-co`) |
| `previous_month_mrr` | MRR del mes anterior en moneda local |
| `mrr_change` | Cambio de MRR en moneda local (negativo para churn) |
| `mrr_change_usd` | Cambio de MRR en USD (usa FX de Chargebee, no Google Sheets) |
| `change_category` | Categoria del movimiento |
| `country` | Pais normalizado (Chile, Colombia, Mexico, Argentina, Otros) |
| `sector_economico` | Sector economico del comercio |
| `sector_group` | Agrupacion legacy de sector (Beauty, Health, Spa, Fitness, Otro) |
| `is_graduated` | Si el cliente paso el proceso de activacion |
| `start_date_pago` | Fecha del primer pago |

### Categorias de cambio disponibles

| Categoria | Significado |
|---|---|
| `new` | Nueva suscripcion (incluye conversion de gratuito a pagado) |
| `churn_graduado` | Cancelacion/pausa de cliente **activado** (paso onboarding) |
| `churn_onboarding` | Cancelacion/pausa durante onboarding |
| `reactivation` | Reactivacion de suscripcion cancelada/pausada |
| `upgrade` | Aumento de MRR |
| `downgrade` | Reduccion de MRR |

### Componentes del churn neto

| Categoria | Significado | Signo en el reporte |
|---|---|---|
| `churn_graduado` | Cancelacion/pausa de cliente que **si** alcanzo el umbral de activacion | Negativo (perdida) |
| `churn_onboarding` | Cancelacion/pausa de cliente que **no** alcanzo el umbral de activacion | Negativo (perdida) |
| `reactivation` | Empresa que vuelve de cancelada/pausada | Positivo (recuperacion) |

**Churn Neto** = churn_graduado + churn_onboarding - reactivaciones

Los umbrales de activacion SaaS que determinan si un churn es graduado u onboarding:

| Segmento | Umbral de activacion |
|---|---|
| B2C | 20 bookings acumulados |
| B2B2 | 40 bookings acumulados |
| B2B3 | 100 bookings acumulados |

El estado de activacion se calcula en `int_company_activation_status`.

---

## 2. Segmento B2B3: `dwh.companies.first_customer_segment_2`

Para filtrar solo empresas **B2B3**, se usa la columna `first_customer_segment_2` de la tabla **`dwh.companies`**:

```sql
JOIN dwh.companies c ON c.company_id = rw.company_id
WHERE c.first_customer_segment_2 = 'B2B3'
```

El modelo dbt esta en `models/agendapro/companies.sql`.

### Por que `first_customer_segment_2` y no `company_attribute_segment2`

Existen varias definiciones de segmento en el DWH. Para el reporte de churn B2B3 (y las presentaciones al Board) se usa **`first_customer_segment_2`** porque clasifica a la empresa segun su **primer snapshot activo** (plan y addons al momento del primer pago). Esto garantiza que una empresa que ingreso como B2B3 siga contandose como B2B3 aunque despues haga downgrade.

| Campo | Tabla | Que mide |
|---|---|---|
| `first_customer_segment_2` | `dwh.companies` | Segmento al momento del **primer pago** (valores: B2C, B2B2, B2B3) |
| `segment2` | `dwh.company_attribute_segment2` | Segmento segun **suscripcion actual** (valores: B2C/B2B2, B2B3+) |
| `Customer_segment_2` | `dwh.companies` | Segmento segun **prestadores activos hoy** |

### Logica de segmentacion de `first_customer_segment_2`

Se evalua el plan y addons del primer snapshot activo de la suscripcion:

- `plan_quantity > 1` → **B2B3**
- Plan `independientes` con 1 prestador incluido → **B2C**
- Plan con 2 prestadores incluidos, sin addon de prestadores extra → **B2B2**
- Cupon `2pp` sin addon de prestadores extra → **B2B2**
- Plan con 5 prestadores incluidos + cupon `1usdx3m-3-5` → **B2B3**
- Plan con 5 prestadores incluidos + cupon `1usdx3m` sin addon extra → **B2B2**
- Cualquier otro caso → **B2B3**

---

## 3. Conversion a USD: FX de Google Sheets

Para convertir MRR de moneda local a USD de forma estandarizada, se usa la tabla de tipo de cambio promedio mensual:

**Tabla**: `staging.stg_google_sheets__exchange_rate_monthly_average`
**Modelo dbt**: `models/staging/google_sheets/stg_google_sheets__exchange_rate_monthly_average.sql`

### Columnas de FX

| Columna | Moneda |
|---|---|
| `usd_clp` | Peso chileno |
| `usd_cop` | Peso colombiano |
| `usd_mxn` | Peso mexicano |
| `usd_ars` | Peso argentino |
| `usd_pen` | Sol peruano |
| `usd_eur` | Euro |
| `usd_uyu` | Peso uruguayo |

### Formula de conversion

```
MRR_USD = MRR_moneda_local / FX_del_mes
```

Ejemplo con FX de marzo 2026:

| Moneda | FX (USD por 1 unidad local) |
|---|---|
| CLP | 910.5810 |
| COP | 3,711.1402 |
| MXN | 17.7841 |
| ARS | 1,396.9652 |
| PEN | 3.4180 |
| USD | 1.0 |

> **Nota**: La tabla `revenue_waterfall_software` ya trae `mrr_change_usd`, pero usa el FX de Chargebee (campo `exchange_rate` de la suscripcion). Para reportes oficiales se recomienda recalcular usando el FX de Google Sheets, que es el promedio mensual aprobado por Finance.

---

## 4. Nombre de la compania: `dwh.companies`

Se obtiene del mismo JOIN con `dwh.companies` que ya se usa para el filtro de segmento:

```sql
JOIN dwh.companies c ON c.company_id = rw.company_id
```

La columna es **`c.company_name`**.

---

## 5. Vertical (definicion oficial)

Segun la [definicion de verticales de Notion](https://www.notion.so/agendapro/Definici-n-verticales-y-categorizaci-n-en-Macro-Nichos-293ab34c233b8097ba51fbe223b04a77), las verticales oficiales de AgendaPro son **4**:

| Vertical | Macro Nichos incluidos |
|---|---|
| **Beauty** | Beauty (barberia, cejas_y_pestanas, estilista_independiente, manicure_pedicure, maquillaje, peluqueria, salon_de_belleza, tatuajes) |
| **Health** | Health Doctors (centro_medico, clinica, consulta_particular, consultorio) + Health Non-doctors (kinesiologo, medicina_alternativa, odontologo, psicologo, veterinaria) |
| **MedSpa** | Spa (centro_de_estetica, spa) |
| **Other** | Fitness (acondicionamiento_fisico, centro_deportivo, clases, crossfit, danza_baile, electroestimulacion, entrenamiento_funcional, gimnasio, karate_combate, personal_trainer, pilates, yoga) + Others (otro, null, etc.) |

### Tabla correcta para obtener la vertical

```sql
-- Opcion 1: tabla directa
LEFT JOIN dwh.company_attribute_vertical v ON v.company_id = rw.company_id
-- columna: v.vertical (Beauty, Health, Medspa, Other)

-- Opcion 2: via company_attributes (EAV)
LEFT JOIN dwh.company_attributes ca ON ca.company_id = rw.company_id AND ca.name = 'vertical'
-- columna: ca.value
```

El modelo dbt esta en `models/agendapro/flags_and_attributes/company_attribute_vertical.sql` y se basa en `dwh.company_attribute_macro_niche`, que selecciona el macro nicho por orden de prioridad:

1. `dwh.company_macro_niche_ai` (clasificacion por IA basada en servicios ofrecidos)
2. `dwh.macro_niche_override` (asignacion manual de 65 empresas)
3. `dwh.hubspot_contacts` (primer lead)
4. `staging.company_economic_sectors` + `dwh.economic_sectors` (sector elegido por el comercio al registrarse)

> **Importante**: No usar `sector_group` de `revenue_waterfall_software` para verticales. Esa columna usa una clasificacion legacy distinta. Siempre usar `dwh.company_attribute_vertical`.

---

## 6. Pais

El campo `country` ya viene precalculado en `revenue_waterfall_software` con valores normalizados: Chile, Colombia, Mexico, Argentina, Otros.

La logica es:

```sql
CASE
    WHEN country_name = 'Chile' THEN 'Chile'
    WHEN country_name = 'Argentina' THEN 'Argentina'
    WHEN country_name = 'México' THEN 'México'
    WHEN country_name = 'Colombia' THEN 'Colombia'
    ELSE 'Otros'
END
```

Fuente: `dwh.companies.country_name`

---

## 7. Fecha del evento (para graficar semanalmente)

El campo `month` en `revenue_waterfall_software` es siempre el **primer dia del mes** (ej: `2026-04-01`), por lo que no sirve para granularidad semanal.

### Fecha de churn (graduado y onboarding)

Para obtener la **fecha exacta de cancelacion**, se consulta `dwh.company_attributes_from_cb`:

```sql
LEFT JOIN dwh.company_attributes_from_cb cb
  ON cb.cb_subscription_id = rw.subscription_id
-- columna: cb.cancelled_at  (timestamp con fecha y hora exacta)
```

Ejemplo: un churn con `month = 2026-04-01` puede tener `cancelled_at = 2026-04-06 07:35:33`.

### Fecha de reactivacion

Las reactivaciones no tienen `cancelled_at` (logico, son empresas que vuelven). Se usa el campo `updated_at` del snapshot del mes correspondiente como proxy:

```sql
LEFT JOIN dwh.chargebee_subscriptions_snapshot css
  ON css.id = rw.subscription_id AND css.month = rw.month
-- columna: css.updated_at  (timestamp de cuando se actualizo la suscripcion)
```

### Agrupar por semana

```sql
-- Para churn:
DATE_TRUNC('week', cb.cancelled_at) AS semana

-- Para reactivaciones:
DATE_TRUNC('week', css.updated_at) AS semana
```

---

## 8. Query de detalle por empresa

```sql
WITH fx AS (
    SELECT
        usd_clp, usd_cop, usd_mxn, usd_ars, usd_pen, usd_uyu
    FROM staging.stg_google_sheets__exchange_rate_monthly_average
    WHERE date = '2026-03-31'  -- Usar el ultimo mes disponible
    LIMIT 1
)
SELECT
    rw.company_id,
    c.company_name,
    rw.country,
    v.vertical,
    c.first_customer_segment_2  AS segment,
    rw.change_category,          -- 'churn_graduado', 'churn_onboarding' o 'reactivation'
    rw.currency_code,
    rw.previous_month_mrr       AS mrr_local,
    rw.mrr_change               AS mrr_cambio_local,
    ROUND(CAST(
        rw.mrr_change / CASE rw.currency_code
            WHEN 'CLP' THEN fx.usd_clp
            WHEN 'COP' THEN fx.usd_cop
            WHEN 'MXN' THEN fx.usd_mxn
            WHEN 'ARS' THEN fx.usd_ars
            WHEN 'PEN' THEN fx.usd_pen
            WHEN 'UYU' THEN fx.usd_uyu
            WHEN 'USD' THEN 1.0
            ELSE 1.0
        END AS numeric), 2)     AS mrr_cambio_usd,
    rw.item_id                  AS plan,
    rw.start_date_pago,
    cb.cancelled_at             AS fecha_cancelacion
FROM dwh.revenue_waterfall_software rw
CROSS JOIN fx
JOIN dwh.companies c
    ON c.company_id = rw.company_id
LEFT JOIN dwh.company_attribute_vertical v
    ON v.company_id = rw.company_id
LEFT JOIN dwh.company_attributes_from_cb cb
    ON cb.cb_subscription_id = rw.subscription_id
WHERE rw.month = '2026-04-01'           -- Mes a analizar
  AND rw.change_category IN ('churn_graduado', 'churn_onboarding', 'reactivation')
  AND c.first_customer_segment_2 = 'B2B3'
ORDER BY mrr_cambio_usd ASC;
```

### Para cambiar el periodo

- Cambiar `rw.month = '2026-04-01'` al primer dia del mes deseado.
- Cambiar la fecha del FX al ultimo dia del mes anterior al mes de analisis.

### Para ver solo un tipo

```sql
-- Solo graduado:
AND rw.change_category = 'churn_graduado'

-- Solo onboarding:
AND rw.change_category = 'churn_onboarding'

-- Solo reactivaciones:
AND rw.change_category = 'reactivation'
```

---

## 9. Query de churn neto semanal (plan quantity + MRR USD)

Esta query consolida los tres componentes en una fila por semana con columnas para graduado, onboarding, reactivaciones y el neto.

Para churn usa `cancelled_at` como fecha; para reactivaciones usa `updated_at` del snapshot mensual.

```sql
WITH fx AS (
    SELECT
        to_char(date + 31, 'yyyymm') AS mes_waterfall,
        usd_clp, usd_cop, usd_mxn, usd_ars, usd_pen, usd_uyu
    FROM staging.stg_google_sheets__exchange_rate_monthly_average
    WHERE date >= '2025-11-30' AND date <= '2026-04-30'
),
churn_data AS (
    SELECT
        DATE_TRUNC('week', cb.cancelled_at)::date AS semana,
        rw.change_category,
        rw.company_id,
        cb.plan_quantity::int AS plan_qty,
        CAST(rw.mrr_change / CASE rw.currency_code
            WHEN 'CLP' THEN fx.usd_clp WHEN 'COP' THEN fx.usd_cop
            WHEN 'MXN' THEN fx.usd_mxn WHEN 'ARS' THEN fx.usd_ars
            WHEN 'PEN' THEN fx.usd_pen WHEN 'UYU' THEN fx.usd_uyu
            WHEN 'USD' THEN 1.0 ELSE 1.0
        END AS numeric) AS mrr_usd
    FROM dwh.revenue_waterfall_software rw
    JOIN dwh.companies c ON c.company_id = rw.company_id
    LEFT JOIN dwh.company_attributes_from_cb cb ON cb.cb_subscription_id = rw.subscription_id
    LEFT JOIN fx ON fx.mes_waterfall = to_char(rw.month, 'yyyymm')
    WHERE rw.month >= '2026-01-01'
      AND rw.change_category IN ('churn_graduado', 'churn_onboarding')
      AND c.first_customer_segment_2 = 'B2B3'
      AND cb.cancelled_at IS NOT NULL
),
react_data AS (
    SELECT
        DATE_TRUNC('week', css.updated_at)::date AS semana,
        'reactivation' AS change_category,
        rw.company_id,
        cb.plan_quantity::int AS plan_qty,
        CAST(rw.mrr_change / CASE rw.currency_code
            WHEN 'CLP' THEN fx.usd_clp WHEN 'COP' THEN fx.usd_cop
            WHEN 'MXN' THEN fx.usd_mxn WHEN 'ARS' THEN fx.usd_ars
            WHEN 'PEN' THEN fx.usd_pen WHEN 'UYU' THEN fx.usd_uyu
            WHEN 'USD' THEN 1.0 ELSE 1.0
        END AS numeric) AS mrr_usd
    FROM dwh.revenue_waterfall_software rw
    JOIN dwh.companies c ON c.company_id = rw.company_id
    LEFT JOIN dwh.company_attributes_from_cb cb ON cb.cb_subscription_id = rw.subscription_id
    LEFT JOIN dwh.chargebee_subscriptions_snapshot css
        ON css.id = rw.subscription_id AND css.month = rw.month
    LEFT JOIN fx ON fx.mes_waterfall = to_char(rw.month, 'yyyymm')
    WHERE rw.month >= '2026-01-01'
      AND rw.change_category = 'reactivation'
      AND c.first_customer_segment_2 = 'B2B3'
),
combined AS (
    SELECT * FROM churn_data
    UNION ALL
    SELECT * FROM react_data
)
SELECT
    semana,
    -- Plan Quantity
    SUM(CASE WHEN change_category = 'churn_graduado' THEN plan_qty ELSE 0 END)    AS grad_qty,
    SUM(CASE WHEN change_category = 'churn_onboarding' THEN plan_qty ELSE 0 END)  AS onb_qty,
    SUM(CASE WHEN change_category = 'reactivation' THEN plan_qty ELSE 0 END) * -1 AS react_qty,
    SUM(CASE WHEN change_category IN ('churn_graduado','churn_onboarding') THEN plan_qty ELSE 0 END)
      - SUM(CASE WHEN change_category = 'reactivation' THEN plan_qty ELSE 0 END)  AS net_qty,
    -- MRR USD
    ROUND(SUM(CASE WHEN change_category = 'churn_graduado' THEN mrr_usd ELSE 0 END), 0)    AS grad_usd,
    ROUND(SUM(CASE WHEN change_category = 'churn_onboarding' THEN mrr_usd ELSE 0 END), 0)  AS onb_usd,
    ROUND(SUM(CASE WHEN change_category = 'reactivation' THEN mrr_usd ELSE 0 END) * -1, 0) AS react_usd,
    ROUND(SUM(mrr_usd), 0)                                                                  AS net_mrr_usd
FROM combined
WHERE semana IS NOT NULL
GROUP BY 1
ORDER BY 1 ASC;
```

### Ejemplo: Churn neto semanal B2B3, enero - abril 2026

| Semana | Grad Qty | Onb Qty | React Qty | **Net Qty** | Grad USD | Onb USD | React USD | **Net MRR USD** |
|---|---|---|---|---|---|---|---|---|
| 29-dic | 26 | 15 | -3 | **38** | -$1,156 | -$558 | -$84 | **-$1,630** |
| 05-ene | 50 | 37 | -20 | **67** | -$2,819 | -$1,259 | -$1,039 | **-$3,039** |
| 12-ene | 30 | 32 | -13 | **49** | -$1,533 | -$1,159 | -$671 | **-$2,021** |
| 19-ene | 53 | 42 | -11 | **84** | -$2,958 | -$1,506 | -$651 | **-$3,813** |
| 26-ene | 37 | 27 | -9 | **55** | -$2,075 | -$842 | -$250 | **-$2,667** |
| 02-feb | 39 | 48 | -12 | **75** | -$118 | -$74 | $0 | **-$192** |
| 09-feb | 40 | 40 | -14 | **66** | -$202 | -$201 | -$26 | **-$377** |
| 16-feb | 29 | 28 | -5 | **52** | -$222 | -$287 | -$53 | **-$456** |
| 23-feb | 59 | 46 | -14 | **91** | -$1,097 | -$206 | -$179 | **-$1,124** |
| 02-mar | 70 | 62 | -40 | **92** | -$4,139 | -$1,754 | -$1,604 | **-$4,289** |
| 09-mar | 109 | 66 | -62 | **113** | -$4,651 | -$3,005 | -$1,741 | **-$5,915** |
| 16-mar | 70 | 82 | -12 | **140** | -$3,458 | -$2,472 | -$943 | **-$4,987** |
| 23-mar | 107 | 90 | 0 | **197** | -$5,420 | -$2,900 | $0 | **-$8,320** |
| 30-mar | 74 | 66 | 0 | **140** | -$2,901 | -$1,049 | $0 | **-$3,950** |
| 06-abr | 26 | 20 | 0 | **46** | -$44 | -$75 | $0 | **-$119** |

> **Lectura de la tabla**: Grad/Onb Qty son locations perdidas (positivo = perdida). React Qty son locations recuperadas (negativo = recuperacion). Net Qty es la perdida neta. Misma logica para USD. Abril va parcial.

---

## 10. Diagrama de dependencias

```
chargebee_subscriptions_snapshot (snapshots mensuales de Chargebee)
        |
        v
int_subscription_mrr_breakdown (desglosa MRR: software vs AI)
        |
        v
int_subscription_mrr_breakdown_changes (calcula deltas mes a mes + categoriza)
        |
        v
revenue_waterfall_software  <-- TABLA PRINCIPAL
        |
        +-- JOIN companies                      --> company_name + filtro B2B3 (first_customer_segment_2)
        +-- JOIN company_attribute_vertical      --> vertical (Beauty/Health/MedSpa/Other)
        +-- JOIN company_attributes_from_cb      --> cancelled_at (fecha exacta churn)
        +-- JOIN chargebee_subscriptions_snapshot --> updated_at (fecha reactivacion)
        +-- CROSS JOIN exchange_rate_monthly_avg --> FX Google Sheets
```

---

## 11. Notas importantes

1. **Signos**: `mrr_change` es negativo para churn y positivo para reactivaciones. El churn neto (suma de los tres) es negativo cuando se pierde mas de lo que se recupera.

2. **FX Chargebee vs Google Sheets**: La tabla ya trae `mrr_change_usd` pero usa el FX de Chargebee. Para reportes oficiales, recalcular con FX de Google Sheets (`staging.stg_google_sheets__exchange_rate_monthly_average`).

3. **Churn pausado = Churn**: Las suscripciones que pasan a estado `paused` tambien se categorizan como churn (graduado u onboarding segun activacion).

4. **Granularidad mensual vs semanal**: `revenue_waterfall_software.month` es mensual. Para semanal, usar `cancelled_at` (churn) o `updated_at` del snapshot (reactivaciones).

5. **Un registro por suscripcion**: La tabla tiene una fila por suscripcion por mes. Cada empresa B2B3 tiene exactamente una suscripcion activa en Chargebee.

6. **Reactivaciones sin fecha semanal**: Algunas reactivaciones pueden no tener `updated_at` en el snapshot del mes, quedando fuera del desglose semanal. El total mensual se mantiene correcto filtrando por `month`.

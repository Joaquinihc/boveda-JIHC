---
type: metodologia
temas: [forecast]
fuente: trabajo propio
fecha: 2026
---

# Forecasting Methodology (Weekly Report CFO)

**Versión:** v3.3 (17 abril 2026)
**Autor:** Pablo Lucero (CFO AgendaPro)

Este documento explica exactamente cómo se construye cada forecast del Weekly Report. Toda la metodología se resume en: (1) linealización MTD con factor unificado, (2) uplift leads-based solo para NewMRR y New Merchants.

---

## TL;DR

```
Forecast EoM = MTD (días completos) × factor_linealización
factor_linealización = días_del_mes / días_completos_con_datos
```

- **Factor abril 2026:** 30 / 16 = **1.875** (uniforme Software, AI, Payments).
- **Regla:** se excluye siempre el día parcial del reporte con filtro SQL `change_date::date < fecha_reporte`.
- **Excepción leads-based:** NewMRR Software y New Merchants B2B3 se ajustan por uplift = (forecast_leads / forecast_lineal) − 1. Aplica solo a la fila New.

---

## 1. Fuente de datos

| Stream | Tabla | Campo clave |
|---|---|---|
| Software Waterfall | `dwh.revenue_waterfall_saas` (filtro `item_type = 'subscription'`) | `mrr_change` (moneda local) |
| AI Waterfall | `dwh.revenue_waterfall_ai` | `mrr_change` (moneda local) |
| Payments Revenue | `dwh.company_sales_months` | `fee` (moneda local) |
| Payments Fee Waterfall | `dwh.revenue_waterfall_payments` | `fee` (por componente) |
| New Merchants B2B3 | `dwh.revenue_waterfall_saas JOIN dwh.companies` | `company_id` con `first_customer_segment_2 = 'B2B3'` |
| Leads B2B3 | `dwh.fct_lead_conversions JOIN staging.stg_hubspot__contacts` | `contact_id` con `numemployees IN ('1-5','6-10','11-25')` |
| BoP Software | `dwh.mrr` mes anterior (menos AI stock) | `mrr` (moneda local) |
| Tipo de cambio Current FX | `staging.stg_google_sheets__exchange_rate_monthly_average` | `usd_clp`, `usd_mxn`, etc. |
| Tipo de cambio Budget FX | Hardcoded (fijo anual 2026) | CLP 900, MXN 17.5, ARS 1500, COP 3700, PEN 3.35, EUR 0.85, UYU 39.0 |

**Importante:** NO usar `exchange_rate` de las tablas waterfall (es promedio ponderado por empresa). Siempre convertir desde moneda local usando staging monthly average.

---

## 2. Factor de linealización unificado

### Definición

```
factor = días_del_mes / días_completos_con_datos
```

### Regla unificada (v3.2+)

Todos los streams usan **el mismo factor**, basado en días completos al corte del reporte. El día del reporte (viernes) se excluye porque los datos están incompletos al momento de generar el reporte.

### Ejemplo abril 2026

- Reporte generado: viernes 17 abril 2026 a las 23:59 SCL
- Último día con datos completos: **16 abril** (día 17 es parcial)
- Días completos con datos: **16**
- Factor: 30 / 16 = **1.875**

### Implementación SQL

```sql
WHERE change_date::date >= '2026-04-01'
  AND change_date::date < '2026-04-17'   -- Estrictamente <, excluye día parcial
```

---

## 3. Forecast por componente (paso a paso)

### 3.1. Software Revenue Waterfall (BoP, New, Upgrade, React, Downgrade, Churn Onb, Churn Grad, Net, EoP)

**Paso 1:** Obtener MTD en moneda local por categoría.

```sql
SELECT change_category,
       SUM(CASE WHEN country = 'Chile'     THEN mrr_change / 903.21
                WHEN country = 'México'    THEN mrr_change / 17.548
                WHEN country = 'Argentina' THEN mrr_change / 1378.92
                WHEN country = 'Colombia'  THEN mrr_change / 3640.67
                WHEN country = 'Peru'      THEN mrr_change / 3.407
                WHEN country = 'España'    THEN mrr_change / 0.858
                WHEN country = 'Uruguay'   THEN mrr_change / 40.00
                ELSE mrr_change / 903.21 END) AS mtd_current_fx,
       SUM(CASE WHEN country = 'Chile'     THEN mrr_change / 900
                WHEN country = 'México'    THEN mrr_change / 17.5
                WHEN country = 'Argentina' THEN mrr_change / 1500
                WHEN country = 'Colombia'  THEN mrr_change / 3700
                WHEN country = 'Peru'      THEN mrr_change / 3.35
                WHEN country = 'España'    THEN mrr_change / 0.85
                WHEN country = 'Uruguay'   THEN mrr_change / 39.0
                ELSE mrr_change / 900 END) AS mtd_budget_fx
FROM dwh.revenue_waterfall_saas
WHERE change_date::date >= '2026-04-01'
  AND change_date::date < '2026-04-17'
  AND item_type = 'subscription'
GROUP BY change_category;
```

**Paso 2:** Linealizar cada componente (excepto New, que se ajusta con leads).

```
forecast_componente = MTD × 1.875   (solo Upgrade, React, Downgrade, Churn Onb, Churn Grad)
```

**Paso 3:** Ajuste leads-based para New (ver sección 4).

**Paso 4:** Construir Net MRR y EoP.

```
Net MRR (Current FX) = New_ajustado + Upgrade + Reactivation + Downgrade + Churn_Onb + Churn_Grad
EoP (Current FX)     = BoP + Net MRR
```

**BoP Software (Current FX):**
```sql
SELECT SUM(CASE WHEN currency_code = 'CLP' THEN mrr / 910.74
                WHEN currency_code = 'MXN' THEN mrr / 20.17
                -- ... otros países con staging monthly average del MES ANTERIOR
                END)
FROM dwh.mrr
WHERE month = '202603';
-- Luego restar AI stock marzo (~$21,016)
```

### 3.2. AI Revenue Waterfall

Misma estructura que Software, pero:

- **Tabla:** `dwh.revenue_waterfall_ai` (no requiere filtro `item_type` porque la tabla entera es AI).
- **Categorías:** New, Upgrade, Reactivation, Downgrade, Churn Onb (no tiene Churn Grad).
- **NO aplicar leads-based a AI.** AI se vende junto con software y el volumen ya queda reflejado en el uplift del Software.

**BoP AI:**
```sql
-- Suma acumulada del waterfall AI hasta el cierre del mes anterior
SELECT SUM(mrr_change_en_current_fx)
FROM dwh.revenue_waterfall_ai
WHERE change_date::date < '2026-04-01';
```

### 3.3. Payments Revenue

**Fuente principal:** `dwh.company_sales_months`, campo `fee` (NO `fee_usd` ni `fee_stock`, que subestiman).

```sql
SELECT SUM(CASE WHEN currency_code = 'CLP' THEN fee / 903.21
                WHEN currency_code = 'MXN' THEN fee / 17.548
                -- ... otros países
                END) AS fee_mtd_current_fx
FROM dwh.company_sales_months
WHERE month = '202604'
  AND day <= 16;   -- Validar con company_sales_days (lag 2-3 días típicamente)
```

**Forecast:**
```
Payments Revenue Forecast = fee_mtd_current_fx × 1.875
```

**Nota especial:** El Downgrade mid-month en Payments refleja empresas que aún no completan sus transacciones del mes (acumulación normal). **NO extrapolable linealmente.** Marcar como `n/m` en Forecast.

### 3.4. New Merchants B2B3

```sql
SELECT COUNT(DISTINCT w.company_id) AS new_merchants_b2b3
FROM dwh.revenue_waterfall_saas w
JOIN dwh.companies c ON c.company_id = w.company_id
WHERE w.change_date::date >= '2026-04-01'
  AND w.change_date::date < '2026-04-17'
  AND w.change_category = 'new'
  AND w.item_type = 'subscription'
  AND c.first_customer_segment_2 = 'B2B3';
```

**Reporte:**
- MTD 16d: valor directo de la query
- Forecast lineal (referencia): MTD × 1.875
- **Forecast oficial = leads-based** (ver sección 4)

---

## 4. Forecast leads-based (NewMRR Software + New Merchants B2B3)

### Problema

La extrapolación lineal **subestima** NewMRR y New Merchants en ~30-40% porque hay **bunching** (acumulación) de firmas en las últimas 2 semanas del mes. Los pagos y confirmaciones de clientes nuevos tienden a concentrarse cerca del cierre. Los primeros 16 días no son representativos.

### Solución

Usar **velocidad diaria de leads B2B3** como leading indicator. Los leads sí se distribuyen uniformemente a lo largo del mes.

### Fórmula

```
Leads_forecast_EoM         = Leads_MTD (16d) × 1.875
New_Merchants_forecast     = Leads_forecast × conversion_rate_trailing_3m_ponderado
New_Merchants_lineal       = MTD_B2B3 × 1.875
uplift_NewMRR              = (New_Merchants_forecast / New_Merchants_lineal) − 1
```

### Aplicación

- Uplift aplicado **solo a la fila New** del Software Waterfall (tanto en Current FX como Budget FX).
- **NO** se aplica a Upgrade, Reactivation, Downgrade, Churn Onb, Churn Grad.
- **NO** se aplica al AI Waterfall.

```
New_ajustado_Current_FX = New_lineal_Current_FX × (1 + uplift)
New_ajustado_Budget_FX  = New_lineal_Budget_FX  × (1 + uplift)
```

### Ejemplo abril 2026

```
Leads_MTD_16d              = 2,177
Leads_forecast             = 2,177 × 1.875 = 4,082
conversion_rate_3m_pond    = 12.41%  (ene-feb-mar 2026)
New_Merchants_forecast     = 4,082 × 0.1241 = 506.56 ≈ 507
New_Merchants_lineal       = 202 × 1.875 = 378.75
uplift                     = 507 / 378.75 − 1 = 0.3375 = +33.75%

New_lineal_Current_FX      = 9,449.03 × 1.875 = 17,716.94
New_ajustado_Current_FX    = 17,716.94 × 1.3375 = 23,696
```

### Query leads B2B3

```sql
SELECT COUNT(DISTINCT l.contact_id) AS leads_b2b3
FROM dwh.fct_lead_conversions l
JOIN staging.stg_hubspot__contacts c
  ON c.contact_id = l.contact_id
WHERE l.recent_conversion_date IS NOT NULL
  AND l.recent_conversion_date >= '2026-04-01'
  AND l.recent_conversion_date < '2026-04-17'
  AND c.numemployees IN ('1-5', '6-10', '11-25');
```

### Cálculo del conversion rate trailing 3m ponderado

```sql
WITH leads AS (
  SELECT DATE_TRUNC('month', l.recent_conversion_date) AS mes,
         COUNT(DISTINCT l.contact_id) AS n_leads
  FROM dwh.fct_lead_conversions l
  JOIN staging.stg_hubspot__contacts c ON c.contact_id = l.contact_id
  WHERE l.recent_conversion_date >= '2026-01-01'
    AND l.recent_conversion_date <  '2026-04-01'
    AND c.numemployees IN ('1-5', '6-10', '11-25')
  GROUP BY 1
),
nuevos AS (
  SELECT DATE_TRUNC('month', w.change_date) AS mes,
         COUNT(DISTINCT w.company_id) AS n_nuevos
  FROM dwh.revenue_waterfall_saas w
  JOIN dwh.companies c ON c.company_id = w.company_id
  WHERE w.change_date >= '2026-01-01'
    AND w.change_date <  '2026-04-01'
    AND w.change_category = 'new'
    AND w.item_type = 'subscription'
    AND c.first_customer_segment_2 = 'B2B3'
  GROUP BY 1
)
SELECT SUM(n.n_nuevos) * 1.0 / SUM(l.n_leads) AS conversion_rate_3m
FROM leads l
LEFT JOIN nuevos n USING (mes);
```

**Regla:** ponderar por volumen total (suma de numerador sobre suma de denominador), **no** promedio simple de los 3 ratios mensuales.

---

## 5. Forecast ARR EoM

```
ARR = (Software_EoP + AI_EoP + Payments_Revenue_Forecast) × 12
```

Todos los términos en Current FX.

Comparar contra Budget en Current FX:
```
vs_Budget = (ARR_forecast − ARR_budget) / ARR_budget
```

### Ejemplo abril 2026

```
Software_EoP (Current FX)         = 807,107
AI_EoP       (Current FX)         = 25,160
Payments_Revenue_Forecast (Current FX) = 186,624
Total mensual                     = 1,018,891
ARR_forecast                      = 1,018,891 × 12 = $12,226,692

ARR_budget                        = $12,840,000
vs_Budget                         = (12,226,692 − 12,840,000) / 12,840,000 = −4.9% 🔴
```

---

## 6. Efecto FX

```
Efecto FX = Valor_Budget_FX − Valor_Current_FX
```

- **Positivo:** FX ayuda (monedas locales se depreciaron MENOS que presupuesto).
- **Negativo:** FX perjudica.

Se reporta línea por línea en las tablas de soporte.

---

## 7. Semáforos

### Categorías positivas (New, Upgrade, Reactivation, Net, EoP, Revenue)

| Rango | Color |
|---|---|
| ≥ 0% | 🟢 |
| 0% a −10% | 🟡 |
| < −10% | 🔴 |

### Categorías negativas (Downgrade, Churn Onb, Churn Grad)

Lógica invertida (más churn = peor):

| Rango | Color |
|---|---|
| ≤ 0% (menos churn que referencia) | 🟢 |
| 0% a +10% | 🟡 |
| > +10% | 🔴 |

BoP no tiene comparación (columna "-").

---

## 8. Errores conocidos a evitar

| Error | Causa | Fix |
|---|---|---|
| FX Effect = $0 en todas las líneas | Bug pipeline Mar 2026: `exchange_rate = dollar_constant` | Convertir desde moneda local con staging monthly average |
| Payments total bajo (~$152K vs ~$177K) | Se usó `fee_stock` (solo empresas con cambios) | Usar `fee` de `company_sales_months` (todas las empresas) |
| Payments sobreestimados ~12% | Se usó `fee_usd` (no ajustado mensual) | Usar `fee_usd_m` o calcular desde `fee` local |
| `companies_b2b3` no existe | Tabla solo en Postgres, no Redshift | Usar `revenue_waterfall_saas JOIN companies` con `first_customer_segment_2 = 'B2B3'` |
| `column mrr_change_local does not exist` | Columna renombrada | Usar `mrr_change` (está en moneda local) |
| `relation dwh.hubspot_contacts does not exist` | Schema incorrecto | Usar `staging.stg_hubspot__contacts` |
| Software Waterfall devuelve 0 filas con `item_type = 'plan'` | En `saas` se llama `'subscription'`; en `software` se llama `'software'` | Filtrar con el valor correcto por tabla |
| Forecast exagerado (30/17) | Día 17 parcial inflaba MTD | Usar 30/16 = 1.875 unificado (v3.2+) |
| NewMRR subestimado ~40% | Bunching últimas 2 semanas del mes | Aplicar uplift leads-based solo a fila New (v3.1+) |
| `date_trunc` falla en `dwh.mrr` | Campo `month` es VARCHAR 'YYYYMM' | Convertir a fecha antes de usar date_trunc |

---

## 9. Resumen ejecutivo (cheat sheet)

1. **MTD** = suma en moneda local / staging monthly average, filtro `change_date::date < fecha_reporte`.
2. **Forecast lineal** = MTD × (30 / días_completos).
3. **Uplift NewMRR** = forecast_leads / forecast_lineal − 1, aplicar solo a fila New.
4. **Net MRR** = New_ajustado + resto_componentes_lineales.
5. **EoP** = BoP + Net MRR.
6. **ARR** = (Software_EoP + AI_EoP + Payments_Revenue) × 12.

---

## 10. Referencias

- [Weekly Reports Ledger (Notion)](https://www.notion.so/25aab34c233b80268356f0d801b91b22)
- [Leads B2B3 metodología comparativa (Notion)](https://www.notion.so/agendapro/Leads-B2B3-Comparaci-n-metodolog-a-antigua-vs-nueva-con-reconversiones-2f4ab34c233b81998080c3ba890e81b0)
- Skill activo: `~/.claude/skills/weekly-report/SKILL.md`
- Documento de metodología general: `metodologia-weekly-report.md`

---

*Documento enfocado exclusivamente en la metodología de forecasting. Para la estructura general del weekly report y convenciones de publicación en Notion, ver `metodologia-weekly-report.md`.*

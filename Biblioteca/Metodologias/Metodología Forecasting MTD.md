---
type: metodologia
temas: [forecast, revenue]
fuente: trabajo propio
fecha: 2026
---

# Metodología de Forecasting MTD — Weekly Report CFO

**Versión:** Abril 2026
**Fuente de verdad:** `dwh` (Redshift) + `staging.stg_google_sheets__exchange_rate_monthly_average`

---

## Principio general

El forecast lineal proyecta el ritmo acumulado hasta hoy (MTD) al cierre del mes, asumiendo que el resto del mes se comportará igual que los días transcurridos.

```
Forecast EOM = Valor_MTD × (días_en_mes / días_completos_con_datos)
```

**Convención de días completos:** el denominador son los días con datos completos. Si el reporte se corre a mitad del día 10, ese día está incompleto y no debe contar como denominador. Solo se usan los días finalizados.

Por ejemplo, si al mediodía del día 10 de un mes de 30 días acumulamos $53,752 (con 9 días completos):

```
Forecast = $53,752 × (30 / 9) = $179,173   ← correcto

NO:  $53,752 × (30 / 10) = $161,256         ← incorrecto (día 10 parcial)
```

La forma práctica: contar los días finalizados, no el número del día del calendario. Si el reporte corre el día 10 al mediodía, el denominador es 9 — no porque "se reste un día" sino porque el día 10 aún no está completo y los únicos días con datos completos son el 1 al 9.

Este método es una aproximación. Su precisión mejora conforme avanza el mes.

---

## 1. Software Revenue — Waterfall

### Fuente de datos
`dwh.revenue_waterfall_software` — campo `mrr_change` (moneda local por empresa).

### Días de datos
El waterfall registra eventos discretos (altas, bajas, upgrades) por mes. Se usa como denominador el número de días completos con datos. Para el reporte del 10 de Abril corrido al mediodía = **9 días completos** (días 1 a 9; el día 10 está incompleto).

### Conversión FX
La conversión a USD se hace manualmente con el staging monthly average del mes correspondiente:

```
mrr_change_USD = mrr_change_local / staging_rate_del_mes
```

**No usar** `mrr_change_usd` ni `mrr_change_constant` de la tabla (bug de pipeline activo desde Mar 2026: ambos campos son iguales).

### Forecast por componente

Cada categoría se proyecta de forma independiente:

| Componente | MTD (9 días completos) | Factor | Forecast (30 días) |
|---|---:|:---:|---:|
| New | 4,308 | × 3.333 | 14,360 |
| Reactivation | 1,109 | × 3.333 | 3,697 |
| Upgrade | 11,185 | × 3.333 | 37,283 |
| Downgrade | -2,962 | × 3.333 | -9,873 |
| Churn Onb | -5,168 | × 3.333 | -17,227 |
| Churn Grad | -5,340 | × 3.333 | -17,800 |
| **Net** | **3,132** | × 3.333 | **10,440** |

### Cálculo del EoP (End of Period)

```
Software EoP Forecast = BoP + Net Forecast
```

El **BoP** (Beginning of Period) = stock real de MRR al cierre del mes anterior, obtenido de `dwh.mrr` convertido con el staging FX del mes actual.

```
BoP Software = Total MRR stock (dwh.mrr, mes anterior, FX actual) − AI Stock acumulado
```

**Nota sobre Churn Onboarding vs Churn Graduado:** La separación entre ambas categorías tiene una inconsistencia conocida en el pipeline. Para análisis de riesgo se usa el **churn total combinado** (Onb + Grad).

---

## 2. AI Revenue — Waterfall y Stock Acumulado

### Fuente de datos
`dwh.revenue_waterfall_ai` — campo `mrr_change` (moneda local).

### Particularidad: stock acumulado

A diferencia de Software (que tiene `dwh.mrr` como fuente de stock), el stock de AI se calcula con una **suma acumulada (running total)** sobre todos los meses históricos del waterfall:

```sql
SUM(net_change_current_fx) OVER (ORDER BY month ROWS UNBOUNDED PRECEDING)
```

Esto da el stock a fin de cada mes. El BoP de Abril = stock al cierre de Marzo = **$21,016**.

### Forecast

Misma lógica que Software: cada componente se multiplica por el factor lineal.

```
AI EoP Forecast = AI BoP + Net AI Forecast
                = 21,016 + (757 × 3.333)
                = 21,016 + 2,523
                = 23,539
```

---

## 3. Payments Revenue

### Fuente de datos
`dwh.company_sales_months` — campo `fee` (moneda local). **No** usar `dwh.revenue_waterfall_payments` para el total: ese waterfall solo incluye empresas con cambios, omitiendo las estables (diferencia de ~$15-20K por mes).

### Días de datos
Se verifica con:
```sql
SELECT MAX(day), COUNT(DISTINCT day) FROM dwh.company_sales_days WHERE month = 'YYYYMM'
```

Payments tiene un lag de pipeline de 2–3 días. Para el reporte del 10 de Abril corrido al mediodía, `MAX(day) = 2026-04-10` pero el día 10 está incompleto: se usan **9 días completos**.

### Forecast

```
Payments Forecast = Fee_MTD_CurrentFX × (30 / 9)
                  = 53,752 × 3.333
                  = 179,173
```

### Nota importante: Payments Fee Waterfall no es forecastable a mitad de mes

`dwh.revenue_waterfall_payments` desagrega el fee por componente (new, upgrade, downgrade, churn). Sin embargo, el componente **"downgrade" es un artificio mid-month**:

El waterfall calcula el fee acumulado hasta hoy vs. el cierre del mes anterior. Una empresa con $100/mes de payments que al día 10 acumuló $33, aparece como "downgrade" de −$67 aunque su comportamiento sea completamente normal. **No es una caída real de revenue.**

Por este motivo:
- El componente `downgrade` del Payments waterfall se marca como **n/m** (not meaningful) en el Forecast.
- El waterfall de Payments solo es interpretable a fin de mes.
- Para el forecast de revenue total se usa `company_sales_months` (fuente correcta).

---

## 4. New Merchants B2B3

### Fuente de datos
```sql
SELECT COUNT(DISTINCT rw.company_id)
FROM dwh.revenue_waterfall_saas rw
JOIN dwh.companies c ON rw.company_id = c.company_id
WHERE rw.change_category = 'new'
  AND c.first_customer_segment_2 = 'B2B3'
  AND rw.month = '2026-04-01'
```

`first_customer_segment_2` captura el segmento al momento de activación (histórico, no el actual).

### Forecast

```
Forecast B2B3 = MTD × (días_mes / días_completos)
              = 97 × (30 / 9)
              = 323 merchants
```

---

## 5. ARR Forecast EOM

El ARR se calcula como el Monthly Recurring Revenue proyectado a fin de mes, anualizado:

```
ARR EOM = (Software EoP + AI Stock EoP + Payments) × 12
```

### Chain of thought (Abril 2026, reporte del 10 de Abril — 9 días completos, factor 3.333×)

| Paso | Cálculo | Resultado |
|---|---|---:|
| 1. Software EoP | BoP $782,135 + Net $10,440 | **$792,575** |
| 2. AI EoP | BoP $21,016 + Net $2,523 | **$23,539** |
| 3. Payments | $53,752 × 3.333 | **$179,173** |
| 4. Suma mensual | 792,575 + 23,539 + 179,173 | **$995,287** |
| 5. ARR | $995,287 × 12 | **$11,943,444** |

---

## 6. Tipos de cambio usados

| Concepto | Uso |
|---|---|
| **Current FX** | Staging monthly average de `staging.stg_google_sheets__exchange_rate_monthly_average`. Columnas: `usd_clp`, `usd_mxn`, `usd_ars`, `usd_cop`, `usd_pen`, `usd_eur`, `usd_uyu`. Una fila por mes (fecha de fin de mes). |
| **Budget FX** | TC fijo anual: CLP/900, MXN/17.5, ARS/1,500, COP/3,700, PEN/3.35, EUR/0.85, UYU/39 |
| **Efecto FX** | Budget FX − Current FX. Positivo = FX ayuda. Negativo = FX perjudica. |

La conversión se hace **siempre manualmente** desde moneda local. No usar columnas `_usd` ni `_constant` de las tablas waterfall (bug de pipeline activo).

---

## 7. Limitaciones del método lineal

| Limitación | Impacto | Mitigación |
|---|---|---|
| Asume comportamiento uniforme en el mes | Errores altos en los primeros días del mes | Interpretar con cautela antes del día 7–8 |
| Pagos concentrados ciertos días (ej: inicio de mes) | Sobreestima/subestima según el patrón | Para Payments, ver distribución diaria en `company_sales_days` |
| Churn y new suelen concentrarse a principio de mes | Puede sobreestimar churn si mayoría ocurrió ya | Comparar forecasts vs mes anterior como sanity check |
| Downgrade en Payments mid-month | No refleja realidad, es acumulación parcial | Marcar como n/m, no extrapolar |
| AI reactivation puede ser 0 mid-month | Forecast de 0 puede ser un artefacto | Verificar si es comportamiento normal |

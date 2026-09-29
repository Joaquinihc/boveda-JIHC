---
type: metodologia
temas: [board, kpis]
fuente: trabajo propio
fecha: 2026
---

# DB - KPIs: Definiciones y Fuentes de Datos

> Documento de referencia para mantener y actualizar la planilla [DB - KPIs](https://docs.google.com/spreadsheets/d/143VsB8uN9gdWTeINDUHpdM1O5fyvFEbODdRhbt5uDp4/edit?gid=0#gid=0).
> Owner: Joaquín Herrera (Finanzas / Reporting)
> Última actualización: 2026-03-12

## Estructura general

- **Hoja:** DB
- **Granularidad:** Mensual, por país (Argentina, Chile, Colombia, México, Otros)
- **Serie histórica:** Desde Dec-2015 en adelante
- **Fuente principal:** Data Warehouse (DWH) - PostgreSQL

---

## 1. # Merchants (Fila 4)

**Definición:**
Total de merchants activos por país en cada mes. Se obtiene sumando el campo `venues` de cada suscripción activa en `dwh.mrr`. El Total es la suma de todos los países.

**Query SQL:**
```sql
-- Total (todos los países)
select month,
       sum(mrr_usd)             as mrr_usd,
       count(venues)            as subscriptions,
       sum(venues)              as merchants
from dwh.mrr
group by 1
order by month asc

-- Chile
select month, currency_code,
       sum(mrr) as mrr, sum(mrr_usd) as mrr_usd,
       count(venues) as subscriptions, sum(venues) as merchants
from dwh.mrr
where currency_code = 'CLP'
group by 1,2 order by month asc

-- Colombia
select month, currency_code,
       sum(mrr) as mrr, sum(mrr_usd) as mrr_usd,
       count(venues) as subscriptions, sum(venues) as merchants
from dwh.mrr
where currency_code = 'COP'
group by 1,2 order by month asc

-- México
select month, currency_code,
       sum(mrr) as mrr, sum(mrr_usd) as mrr_usd,
       count(venues) as subscriptions, sum(venues) as merchants
from dwh.mrr
where currency_code = 'MXN'
group by 1,2 order by month asc

-- Argentina
select month, currency_code,
       sum(mrr) as mrr, sum(mrr_usd) as mrr_usd,
       count(venues) as subscriptions, sum(venues) as merchants
from dwh.mrr
where currency_code = 'ARS'
group by 1,2 order by month asc

-- Otros Latam
select month,
       sum(mrr_usd) as mrr_usd,
       count(venues) as subscriptions, sum(venues) as merchants
from dwh.mrr
where currency_code not in ('CLP', 'COP', 'ARS', 'MXN')
and first_paying_month is not null
group by 1 order by month asc
```

**Fuente:** DWH (`dwh.mrr`)
**Frecuencia:** Mensual
**Columna usada en planilla:** `merchants` (= `sum(venues)`)
**Notas:**
- La query también produce `subscriptions` y `mrr_usd` que se usan en otras secciones de la planilla.
- "Otros Latam" filtra adicionalmente por `first_paying_month is not null`.
- La query original de México tenía un typo (`onth` en lugar de `month`) — corregido aquí.

---

## 2. # New Merchants (Fila 14)

**Definición:**
Cantidad de merchants nuevos por país en cada mes. Se filtra por `change_category = 'new'` en `dwh.nrr`. El Total es la suma de todos los países (query sin filtro de `currency_code`).

**Query SQL:**
```sql
-- Total (todos los países)
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       sum(mrr) as mrr, sum(venues) as merchants
from dwh.nrr
where change_category = 'new'
group by 1, 2 order by 1 asc, 2

-- Argentina
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       sum(mrr) as mrr, sum(venues) as merchants
from dwh.nrr
where change_category = 'new' and currency_code = 'ARS'
group by 1, 2 order by 1 asc, 2

-- Chile
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       sum(mrr) as mrr, sum(venues) as merchants
from dwh.nrr
where change_category = 'new' and currency_code = 'CLP'
group by 1, 2 order by 1 asc, 2

-- Colombia
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       sum(mrr) as mrr, sum(venues) as merchants
from dwh.nrr
where change_category = 'new' and currency_code = 'COP'
group by 1, 2 order by 1 asc, 2

-- México
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       sum(mrr) as mrr, sum(venues) as merchants
from dwh.nrr
where change_category = 'new' and currency_code = 'MXN'
group by 1, 2 order by 1 asc, 2
```

**Fuente:** DWH (`dwh.nrr`)
**Frecuencia:** Mensual
**Columna usada en planilla:** `merchants` (= `sum(venues)`)
**Notas:**
- La query también produce `mrr` que puede usarse para nuevos MRR por país.
- No hay query explícita para "Otros" — se calcula como Total menos la suma de los 4 países.

---

## 3. # Churn (Fila 24)

**Definición:**
Cantidad de merchants que churnearon por país en cada mes. Se filtra por `change_category = 'churn'` en `dwh.nrr`. Usa un `lag` para obtener la cantidad de venues del mes anterior (el último mes activo antes del churn), ya que en el mes de churn el merchant ya no tiene venues activos.

**Query SQL:**
```sql
-- Total (todos los países)
with lag_venues as (
  select lag(venues, 1) over (partition by company_id order by date_month) as q, *
  from dwh.nrr
)
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       case when change_category = 'churn' then sum(q)
            else sum(venues) end as q
from lag_venues
where change_category = 'churn'
group by 1, 2 order by 1 asc, 2

-- Argentina
with lag_venues as (
  select lag(venues, 1) over (partition by company_id order by date_month) as q, *
  from dwh.nrr
)
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       case when change_category = 'churn' then sum(q)
            else sum(venues) end as q
from lag_venues
where change_category = 'churn' and currency_code = 'ARS'
group by 1, 2 order by 1 asc, 2

-- Chile
with lag_venues as (
  select lag(venues, 1) over (partition by company_id order by date_month) as q, *
  from dwh.nrr
)
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       case when change_category = 'churn' then sum(q)
            else sum(venues) end as q
from lag_venues
where change_category = 'churn' and currency_code = 'CLP'
group by 1, 2 order by 1 asc, 2

-- Colombia
with lag_venues as (
  select lag(venues, 1) over (partition by company_id order by date_month) as q, *
  from dwh.nrr
)
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       case when change_category = 'churn' then sum(q)
            else sum(venues) end as q
from lag_venues
where change_category = 'churn' and currency_code = 'COP'
group by 1, 2 order by 1 asc, 2

-- México
with lag_venues as (
  select lag(venues, 1) over (partition by company_id order by date_month) as q, *
  from dwh.nrr
)
select to_char(date_month, 'yyyyMM') as month,
       change_category,
       case when change_category = 'churn' then sum(q)
            else sum(venues) end as q
from lag_venues
where change_category = 'churn' and currency_code = 'MXN'
group by 1, 2 order by 1 asc, 2
```

**Fuente:** DWH (`dwh.nrr`)
**Frecuencia:** Mensual
**Columna usada en planilla:** `q` (venues del mes anterior al churn)
**Notas:**
- Se usa `lag(venues, 1)` porque al momento del churn el merchant ya tiene 0 venues; se necesita el valor del último mes activo para contar cuántos merchants se perdieron.
- El dato se registra en el mes/año correspondiente al cohorte (mes en que ocurre el churn).
- "Otros" no tiene query explícita — se calcula como Total menos los 4 países.

---

## 4. # Reactivated (Fila ~34)

**Definición:**
Cantidad de merchants reactivados por país en cada mes. Se calcula por diferencias, no tiene query propia.

**Fórmula:**
```
Reactivated = Merchants(mes_actual) - (Merchants(mes_anterior) + New Merchants - Churn)
```

**Fuente:** Calculado a partir de secciones #1, #2 y #3
**Frecuencia:** Mensual
**Notas:**
- No se obtiene directo del DWH; es un residual que captura reactivaciones y cualquier otro ajuste neto.

---

## 5. % Net Logo Churn (Fila 52)

**Definición:**
Porcentaje de churn neto de logos (merchants). Es el churn menos las reactivaciones, expresado como porcentaje sobre los merchants del mes anterior.

**Fórmula:**
```
Net Logo Churn = Churn - Reactivations
% Net Logo Churn = Net Logo Churn / Merchants(mes_anterior)
```

**Fuente:** Calculado a partir de secciones #3 y #4
**Frecuencia:** Mensual
**Notas:**
- Se expresa como porcentaje positivo (ej: 2.4% significa que se perdió neto un 2.4% de la base).

---

## 6. P&L Costs (Fila 63)

**Definición:**
Costos del P&L por país. La definición exacta de qué costos incluir debe ser definida por Pablo Lucero (CFO).

**Query SQL:** N/A — fuente manual.

**Fuente:**
- 2015-2016: Mastertrack AgendaPro
- 2017-2020: Estados financieros AP
- 2021+: P&L AgendaPro de Pablo Lucero

**Frecuencia:** Mensual
**Notas:**
- Tipo de cambio variable (*Variable exchange rates).
- Coordinar con Pablo Lucero para definir qué líneas del P&L alimentan esta sección.

---

## 7. Fully-Loaded CAC (Fila 73)

**Definición:**
Costo de adquisición de cliente fully-loaded por país en cada mes. Se divide el costo total del P&L del mes por la cantidad de new merchants del mismo mes.

**Fórmula:**
```
Fully-Loaded CAC = P&L Costs(mes) / New Merchants(mes)
```

**Fuente:** Calculado a partir de secciones #6 y #2
**Frecuencia:** Mensual
**Notas:**
- Incluye todos los costos del P&L, no solo marketing (por eso es "fully-loaded").

---

## 8. Revenue eq. POS (Fila 103)

**Definición:**
Revenue equivalente de POS por país en cada mes. Convierte el ingreso de POS a un equivalente SaaS ajustando por la diferencia de márgenes brutos.

**Fórmula:**
```
Revenue eq. POS = (New POS en el mes × POS_gross_margin) / Gross Margin SaaS
```

**Fuente:** Calculado — requiere datos de POS vendidos y márgenes
**Frecuencia:** Mensual
**Notas:**
- Tipo de cambio variable (*Variable exchange rates).
- Permite comparar el ingreso de POS en términos equivalentes al negocio SaaS.

---

## 9. Payback est. (Fila 113)

**Definición:**
Payback estimado por país en cada mes. Indica cuántos meses toma recuperar el costo de adquisición con el MRR equivalente ajustado por margen bruto SaaS.

**Fórmula:**
```
Payback est. = P&L Costs / (Revenue eq. MRR × Gross Margin SaaS)
```

**Fuente:** Calculado a partir de secciones #6, MRR y #11
**Frecuencia:** Mensual
**Notas:**
- Resultado en meses (ej: 4.3x = 4.3 meses para recuperar la inversión).

---

## 10. Payback est. variable (Fila 143)

**Definición:**
Variante del payback estimado con tipo de cambio variable. Definición exacta pendiente de Pablo Lucero (CFO).

**Fórmula:** Pendiente — definir con Pablo Lucero.

**Fuente:** Pendiente
**Frecuencia:** Mensual
**Notas:**
- Coordinar con Pablo Lucero para documentar la diferencia respecto al Payback est. (sección #9).

---

## 11. Gross Margins (Fila 153)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** Pendiente
**Frecuencia:** Mensual
**Notas:** Actualmente 90.0% fijo en todos los países.

---

## 12. Leads (Fila 163)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** Pendiente
**Frecuencia:** Mensual

---

## 13. Details CR by segment (Fila 192)

### 13a. CR 3+ (Fila 193)
### 13b. CR 2 (Fila 203)
### 13c. CR 1 (Fila 213)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** Pendiente
**Frecuencia:** Mensual

---

## 14. Leads 1 (Fila 243)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** Pendiente
**Frecuencia:** Mensual

---

## 15. Merchants 3+ (Fila 253)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** Pendiente
**Frecuencia:** Mensual

---

## 16. # POS sold (Fila 291)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** *Variable exchange rates
**Frecuencia:** Mensual

---

## 17. Take rate (Fila 301)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** *Variable exchange rates
**Frecuencia:** Mensual

---

## 18. MRR 1 (Fila 333)

**Definición:** Pendiente — definir con Pablo Lucero (CFO). Debe cuadrar con definiciones de OKRs.
**Fuente:** DWH
**Frecuencia:** Mensual

---

## Proceso de actualización

_Pendiente: documentar paso a paso cómo Joaquín debe actualizar la planilla mensualmente._

---

## Comparación con Notion (DB - Metrics - KPIs)

Página Notion existente: [DB - Metrics - KPIs](https://www.notion.so/agendapro/DB-Metrics-KPIs-bd9e1e8e0c4c46598d8e43aa1a8799ce)

| KPI | En Notion | En este doc | Delta |
|-----|-----------|-------------|-------|
| _Pendiente de comparar al finalizar_ | | | |

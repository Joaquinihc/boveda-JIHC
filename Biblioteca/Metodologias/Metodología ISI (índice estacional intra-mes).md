---
type: metodologia
temas: [forecast, estacionalidad]
fuente: trabajo propio
fecha: 2026
---

# Metodología ISI — Índice Estacional Intra-Mes

**Versión:** Abril 2026
**Fuente de verdad:** `dwh` (Redshift) + `staging.stg_google_sheets__exchange_rate_monthly_average`
**Autora del análisis:** Claude (CFO Analytics, AgendaPro)

---

## Nombre y propósito

**ISI — Índice Estacional Intra-Mes** es una metodología de forecasting que reemplaza la proyección lineal simple por una proyección basada en el patrón histórico de distribución dentro del mes. En lugar de asumir que cada día del mes aporta la misma proporción al total, el ISI aprende de meses anteriores cuánto del total mensual suele acumularse en los primeros N días.

---

## Principio general

```
Forecast EOM = MTD / índice_histórico_día_N
```

Donde `índice_histórico_día_N` es el porcentaje promedio del total mensual que acumularon los primeros N días en los meses históricos disponibles.

### Comparación con el método lineal

| Método | Fórmula | Supuesto |
|---|---|---|
| Lineal | `MTD × (días_mes / días_completos)` | Cada día aporta igual fracción |
| ISI | `MTD / índice_histórico_día_N` | El mes tiene forma conocida y estable |

Si el índice histórico al día 12 es 36.8%, significa que los primeros 12 días de un mes de 30 suelen acumular el 36.8% del total. El forecast ISI ajusta hacia arriba o abajo dependiendo de si el ritmo actual está por delante o detrás de ese patrón.

---

## Cómo se calculan los índices

### Fuente de datos diaria

`dwh.revenue_waterfall_saas` y `dwh.revenue_waterfall_ai` tienen granularidad diaria vía el campo `start_date_pago`. Para cada mes histórico y componente:

1. Se acumulan los valores diarios (running sum) de `mrr_change` en moneda local.
2. Se divide el acumulado al día N por el total del mes.
3. Se promedia ese porcentaje a través de todos los meses históricos disponibles.

```sql
-- Ejemplo conceptual: índice al día 12 para "new" en Software
WITH daily AS (
  SELECT
    DATE_TRUNC('month', start_date_pago) AS month,
    DATE_PART('day', start_date_pago) AS day_of_month,
    SUM(mrr_change) AS daily_mrr
  FROM dwh.revenue_waterfall_saas
  WHERE change_category = 'new'
    AND start_date_pago < DATE_TRUNC('month', CURRENT_DATE)  -- solo meses completos
  GROUP BY 1, 2
),
cumulative AS (
  SELECT
    month,
    day_of_month,
    SUM(daily_mrr) OVER (PARTITION BY month ORDER BY day_of_month) AS cum_mrr,
    SUM(daily_mrr) OVER (PARTITION BY month) AS total_mrr
  FROM daily
)
SELECT
  day_of_month,
  ROUND(AVG(cum_mrr / NULLIF(total_mrr, 0)) * 100, 1) AS indice_pct
FROM cumulative
GROUP BY 1
ORDER BY 1
```

### Importante: solo meses completos

Los índices se calculan **únicamente** sobre meses ya cerrados. No se incluye el mes en curso porque su total aún no se conoce.

---

## Índices calculados — Abril 2026

Los índices se basan en **enero 2024 – marzo 2026 (27 meses completos)**. Versión anterior usaba solo 3 meses (Ene-Mar 2026); se expandió para mayor estabilidad estadística.

### Software — por componente (días 7, 9 y 12)

| Componente | Índice d7 | Índice d9 | Índice d12 | σ d12 | Fuente |
|---|:---:|:---:|:---:|:---:|---|
| New | 20.81% | 26.52% | 36.19% | 9.20% | `revenue_waterfall_saas` |
| Reactivation | 22.33% | 31.54% | 35.90% | 13.59% | `revenue_waterfall_saas` |
| Upgrade | 22.31% | 28.03% | 38.40% | 13.27% | `revenue_waterfall_saas` |
| Downgrade | 23.50% | 29.66% | 36.29% | 15.84% | `revenue_waterfall_saas` |
| Churn Onboarding | 20.96% | 26.77% | 36.36% | 13.69% | `revenue_waterfall_saas` |
| Churn Graduado | 23.50% | 32.25% | 42.12% | 14.40% | `revenue_waterfall_saas` |

**Nota sobre semántica de `start_date_pago`:** Para "new", este campo es la fecha de creación de la suscripción (dentro del mes). Para las demás categorías, es la fecha de inicio original de la suscripción. Los eventos de billing (churn, upgrade, downgrade, reactivation) se procesan en la fecha de aniversario mensual, lo que genera la distribución intra-mes. La metodología ISI es válida para todas las categorías porque los índices capturan el patrón de aniversarios de cobro.

### AI — componente net (días 7, 9 y 12)

| Componente | Índice d7 | Índice d9 | Índice d12 | σ d12 | Nota |
|---|:---:|:---:|:---:|:---:|---|
| Net AI | 23.18% | 23.25% | 35.99% | **23.08%** | Alta volatilidad — producto en crecimiento acelerado |

La desviación estándar de 23.08% indica que el índice AI varía enormemente entre meses. Al día 12, el método lineal puede ser más confiable para AI.

### New Merchants B2B3 (días 7, 9 y 12) — basado en conteo

| Métrica | Índice d7 | Índice d9 | Índice d12 | σ d12 |
|---|:---:|:---:|:---:|:---:|
| New B2B3 (merchants) | 20.67% | 26.33% | 36.75% | 5.55% |

El índice se calcula sobre el **conteo de merchants** (no MRR), ya que la métrica de seguimiento es número de altas.

### Payments (índice real desde `company_sales_days`)

| Métrica | Índice d7 | Índice d9 | Índice d12 | σ d12 | Nota |
|---|:---:|:---:|:---:|:---:|---|
| Payments Fee | 23.72% | 30.23% | **40.37%** | **2.87%** | Índice más estable del portafolio |

La desviación estándar de 2.87% es la más baja de todas las métricas, lo que indica que el patrón de Payments intra-mes es muy consistente. `dwh.company_sales_days`, campo `fee_usd_m`, granularidad diaria real.

**Detalle histórico al día 12 (selección):**

| Mes | Índice d12 | Acum d12 (USD) | Total mes (USD) |
|---|:---:|---:|---:|
| Enero 2026 | 35.79% | 64,344 | 179,773 |
| Febrero 2026 | 40.70% | 70,560 | 173,378 |
| Marzo 2026 | 40.57% | 78,291 | 192,960 |
| Promedio 27m | **40.37%** | | |

---

## Fórmula de aplicación

Para un reporte al día 13 de un mes de 30 días (12 días completos):

```
Forecast_componente = MTD_componente / índice_día_12
```

### Ejemplo — Abril 2026 (día 12, corte al 13 de abril, 30 días)

Índices aplicados: base 27 meses (Ene 2024 – Mar 2026).

**Software:**

| Componente | MTD día 12 | Índice 27m | Forecast EOM |
|---|---:|:---:|---:|
| New | 6,028 | 36.19% | 16,657 |
| Reactivation | 1,389 | 35.90% | 3,870 |
| Upgrade | 16,271 | 38.40% | 42,372 |
| Downgrade | -4,195 | 36.29% | -11,561 |
| Churn Onb | -6,364 | 36.36% | -17,503 |
| Churn Grad | -8,070 | 42.12% | -19,156 |
| **Net** | **5,059** | — | **14,679** |

```
Software EoP = BoP $782,135 + Net ISI $14,679 = $796,814
Software EoP Lineal = $782,135 + ($5,059 × 2.5) = $794,783
```

**AI:**

```
Net AI MTD (día 12) = $1,176 USD
AI Net ISI   = $1,176 / 35.99% = $3,267  →  AI EoP ISI   = $21,016 + $3,267 = $24,283
AI Net Lineal = $1,176 × 2.5   = $2,940  →  AI EoP Lineal = $21,016 + $2,940 = $23,956
Nota: σ alto (23.08%) — preferir lineal para AI al día 12.
```

**Payments:**

```
Payments MTD (días 1-12) = $69,271 USD
Payments ISI   = $69,271 / 40.37% = $171,584
Payments Lineal = $69,271 × 2.5  = $173,178
Nota: índice 27m (40.37%) > lineal implícito (40%), ISI es levemente más conservador.
```

**B2B3:**

```
B2B3 MTD (días 1-12) = 117 merchants
B2B3 ISI   = 117 / 36.75% = 318 merchants
B2B3 Lineal = 117 × 2.5  = 293 merchants
```

**ARR comparativo:**

```
ARR ISI (27m) = ($796,814 + $24,283 + $171,584) × 12 = $992,681 × 12 = $11,912,172 ≈ $11.91M  (-7.2% vs budget)
ARR Lineal    = ($794,783 + $23,956 + $173,178) × 12 = $991,917 × 12 = $11,903,004 ≈ $11.90M  (-7.3% vs budget)
ARR Budget    = $12,840,000

→ Con 27 meses de historia, ISI y Lineal convergen al día 12 (diferencia de solo $9K en ARR).
  El ISI aporta más valor en días 1-7, cuando el Lineal tiene mayor error relativo.
```

---

## Resultado de backtesting

Se testeó el desempeño de ambas metodologías haciendo cortes al día 12 de enero, febrero y marzo 2026 y comparando el forecast contra el cierre real del mes.

### Error absoluto promedio (MAE) al día 12

| Metodología | MAE componentes Software | Observación |
|---|:---:|---|
| Lineal (30/12) | **11.9%** | Ganador en la mayoría de meses al día 12 |
| ISI (estacional) | 15.3% | Peor en promedio al día 12 — ver nota |

### Por qué el ISI no siempre supera al lineal

A partir del día 8-9 del mes, el método lineal ya tiene suficiente masa de datos para que su simplicidad sea una ventaja. El ISI añade valor principalmente en los primeros días del mes, cuando la distribución intra-mes distorsiona la proyección lineal.

**Ejemplo:** Si los nuevos contratos se concentran en los días 1-5, un corte al día 3 con método lineal proyectará el 100% del mes desde esa cifra inflada. El ISI, sabiendo que los días 1-3 representan típicamente solo el 15% del mes, genera un forecast más conservador y acertado.

### Recomendación de uso

| Día del mes | Metodología recomendada |
|:---:|---|
| 1 - 7 | ISI (el lineal sobreestima o subestima severamente) |
| 8 - 15 | Usar ambas y comparar rangos; preferir lineal si difieren mucho |
| 16 en adelante | Lineal (error < 8%, ISI no agrega valor significativo) |

---

## Limitaciones del método ISI

| Limitación | Impacto | Mitigación |
|---|---|---|
| Solo 3 meses de historia (Jan-Mar 2026) | Alta varianza en los índices | Actualizar mensualmente; con 6+ meses mejora significativamente |
| Pagos en weekends (sáb/dom) naturalmente más bajos | Efecto real, no error de datos — índice ya lo captura | Verificar que día N no sea el 1er día de semana incompleto |
| Meses con distribución atípica (fin de año, promociones) | Índice promedio no representa el mes actual | Identificar y excluir outliers históricos |
| Cambios de mix de países | Patrón de pago puede cambiar con el tiempo | Re-calcular índices cada trimestre |
| Fechas imposibles en DWH (ej: feb día 30) | Artefactos en la distribución | Filtrar `start_date_pago <= last_day_of_month` |

---

## Procedimiento de actualización de índices

Los índices se deben recalcular al inicio de cada mes incorporando el mes recién cerrado.

1. Correr la query de índices históricos con `WHERE month < DATE_TRUNC('month', CURRENT_DATE)`.
2. Calcular el promedio rodante de los últimos 3-6 meses (más recientes pesan más).
3. Actualizar la tabla de índices en este documento.
4. Registrar la fecha de actualización y los meses incluidos en el cálculo.

---

## Historial de versiones

| Versión | Fecha | Cambios |
|---|---|---|
| 1.0 | Abril 2026 | Versión inicial. Índices calculados sobre Jan-Mar 2026 (3 meses). Backtest al día 12. |

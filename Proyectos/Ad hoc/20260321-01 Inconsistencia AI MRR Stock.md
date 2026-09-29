---
type: analisis
proyecto: ad hoc
estado: cerrado
temas: [ai, mrr, waterfall]
fecha: 2026-03-21
autor: Pablo Lucero (vía Claude Code)
metricas: ["[[AI Revenue (AI MRR run-rate)]]"]
---

# Inconsistencia en AI Total MRR Stock — Análisis
**Fecha:** 21 de marzo 2026
**Autor:** Pablo Lucero (vía Claude Code)
**Para:** Joaquín Herrera

---

## Resumen ejecutivo

Al intentar calcular el AI Total MRR Stock para el reporte semanal de marzo 2026, encontramos que **dos fuentes del DWH reportan valores muy distintos y no reconcilian entre sí**. El waterfall de Julia/Sofia muestra un net MRR positivo en marzo, pero el stock real ya cayó ~$1,938 en los primeros 20 días del mes. Esto sugiere que hay churn real que **no está siendo capturado** en las tablas de waterfall.

---

## Las dos fuentes en cuestión

### Fuente 1: `dwh.int_subscription_mrr_breakdown`
**Qué mide:** Snapshot mensual de todas las suscripciones activas con `mrr_julia > 0` OR `mrr_sofia > 0`.
**Columnas clave usadas:**
- `mrr_ai` — MRR del addon AI en **moneda local** (CLP, MXN, ARS, etc.)
- `currency_code` — para aplicar el tipo de cambio presupuesto
- `status = 'active'`
- `month` — mes del snapshot

**Tipo de cambio aplicado (budget FX):**
```
CLP: 900 · MXN: 17.5 · ARS: 1,500 · COP: 3,700 · EUR: 0.85 · USD: 1.0
```

**Query:**
```sql
WITH fx_budget AS (
    SELECT 'CLP' AS cur, 900.0 AS rate UNION ALL
    SELECT 'COP', 3700.0 UNION ALL
    SELECT 'MXN', 17.5 UNION ALL
    SELECT 'ARS', 1500.0 UNION ALL
    SELECT 'PEN', 3.35 UNION ALL
    SELECT 'EUR', 0.85 UNION ALL
    SELECT 'UYU', 39.0 UNION ALL
    SELECT 'USD', 1.0
)
SELECT
    s.month,
    COUNT(DISTINCT s.subscription_id) AS subs,
    ROUND(SUM(
        CASE WHEN s.currency_code NOT IN ('CLP','MXN','ARS','COP','EUR')
             THEN COALESCE(s.mrr_ai, 0)
             ELSE COALESCE(s.mrr_ai, 0) / NULLIF(f.rate, 0)
        END
    )::numeric, 0) AS ai_stock_budget_fx
FROM dwh.int_subscription_mrr_breakdown s
LEFT JOIN fx_budget f ON f.cur = s.currency_code
WHERE s.month >= '2025-10-01'
  AND (s.mrr_julia > 0 OR s.mrr_sofia > 0)
  AND s.status = 'active'
GROUP BY 1
ORDER BY 1 DESC;
```

**Resultado:**
| Mes | Subs activas | AI Stock (budget FX) | Cambio MoM |
|-----|-------------|----------------------|------------|
| Oct 2025 | 56 | $4,182 | — |
| Nov 2025 | 120 | $10,512 | **+$6,330** |
| Dic 2025 | 178 | $10,524 | +$12 |
| Ene 2026 | 190 | $12,634 | +$2,110 |
| Feb 2026 | 309 | $17,919 | **+$5,285** |
| **Mar 2026 (20d)** | **269** | **$15,981** | **-$1,938** |

→ **Esta es la fuente que el usuario confirma como correcta.** Pablo dijo "estábamos en 18k en febrero", lo que coincide con $17,919. ✅

---

### Fuente 2: `dwh.revenue_waterfall_julia` + `dwh.revenue_waterfall_sofia`
**Qué mide:** Cambios mensuales (flows) de MRR por categoría para suscripciones Julia y Sofia. **Solo incluye suscripciones que tuvieron un evento de cambio ese mes** (new, churn, upgrade, downgrade, reactivation).
**Columnas clave usadas:**
- `mrr_change_constant` — cambio de MRR en USD a tipo de cambio constante (≈ budget FX)
- `mrr_stock_constant` — stock de la suscripción a tipo de cambio constante
- `change_category` — tipo de movimiento

> ⚠️ El campo `dollar_constant` en estas tablas **ya es el tipo de cambio presupuesto**, verificado:
> CLP ≈ 901 · MXN ≈ 17.65 · ARS ≈ 1,405 · COP ≈ 3,737 · EUR ≈ 0.864

**Query de flows:**
```sql
SELECT
    month,
    change_category,
    ROUND(SUM(mrr_change_constant)::numeric, 0) AS mrr_change_budget_fx,
    COUNT(DISTINCT subscription_id) AS subs
FROM (
    SELECT month, change_category, mrr_change_constant, subscription_id
    FROM dwh.revenue_waterfall_julia
    WHERE month >= '2025-11-01'
    UNION ALL
    SELECT month, change_category, mrr_change_constant, subscription_id
    FROM dwh.revenue_waterfall_sofia
    WHERE month >= '2025-11-01'
) t
GROUP BY 1, 2
ORDER BY 1 DESC, 2;
```

**Resultado (waterfall flows):**
| Mes | New MRR | Churn | Downgrade | Upgrade | React. | **Net MRR** |
|-----|---------|-------|-----------|---------|--------|------------|
| Nov 2025 | +$8,116 | -$577 | -$23 | — | — | **+$7,516** |
| Dic 2025 | +$6,048 | -$2,508 | -$3 | +$63 | +$44 | **+$3,644** |
| Ene 2026 | +$7,443 | -$3,504 | -$2,193 | — | — | **+$1,746** |
| Feb 2026 | +$4,313 | -$3,024 | -$1,622 | — | +$46 | **-$282** |
| **Mar 2026 (20d)** | **+$4,032** | **-$2,985** | **-$98** | **+$460** | **+$164** | **+$1,573** |

**Query de stock (waterfall):**
```sql
SELECT
    month,
    ROUND(SUM(mrr_stock_constant)::numeric, 0) AS stock_constant_fx,
    COUNT(DISTINCT subscription_id) AS subs
FROM (
    SELECT month, mrr_stock_constant, subscription_id
    FROM dwh.revenue_waterfall_julia WHERE month >= '2025-11-01'
    UNION ALL
    SELECT month, mrr_stock_constant, subscription_id
    FROM dwh.revenue_waterfall_sofia WHERE month >= '2025-11-01'
) t
GROUP BY 1
ORDER BY 1 DESC;
```

**Resultado (stock del waterfall):**
| Mes | Stock waterfall | Subs |
|-----|----------------|------|
| Nov 2025 | $8,177 | 91 |
| Dic 2025 | $6,310 | 138 |
| Ene 2026 | $7,838 | 155 |
| Feb 2026 | $7,498 | 192 |
| Mar 2026 (20d) | $5,798 | 134 |

→ El stock del waterfall ($7,498 en feb) es **menos de la mitad** del stock real ($17,919). Esto confirma que el waterfall **no puede usarse como fuente de stock**.

---

## La inconsistencia central

Comparando ambas fuentes, el **net MRR del waterfall no explica el cambio real en el stock**:

| Mes | Δ Stock real (Fuente 1) | Net waterfall (Fuente 2) | **Gap** |
|-----|------------------------|--------------------------|---------|
| Nov 2025 | +$6,330 | +$7,516 | -$1,186 |
| Dic 2025 | +$12 | +$3,644 | **-$3,632** |
| Ene 2026 | +$2,110 | +$1,746 | +$364 |
| Feb 2026 | +$5,285 | -$282 | **+$5,567** |
| Mar 2026 (20d) | -$1,938 | +$1,573 | **-$3,511** |

Los gaps son grandes e inconsistentes en dirección, lo que descarta un error sistemático simple de FX.

---

## Hipótesis sobre la causa

La explicación más probable es que **el waterfall no captura todos los tipos de churn de AI**. Las categorías que sí aparecen son:
- `new` · `churn_onboarding` · `downgrade` · `upgrade` · `reactivation`

**Lo que probablemente falta:**
- `churn_graduado` — clientes que terminaron el período de onboarding y luego cancelaron el addon AI
- Otros tipos de baja que no generan un evento explícito en las tablas `revenue_waterfall_julia/sofia`

Esto explicaría por qué el waterfall muestra **net positivo (+$1,573)** en marzo mientras el stock real **ya bajó $1,938** — hay ~$3,500 de churn AI que no está siendo registrado como `churn_onboarding` ni ninguna otra categoría visible.

**Dato adicional que lo confirma:**
- En `int_subscription_mrr_breakdown`: feb=309 subs → mar=269 subs → **-40 subs netas**
- En el waterfall marzo: 52 churns, 59 new → **+7 subs netas**
- Diferencia: **~47 suscripciones que churnearon sin aparecer en el waterfall**

---

## Recomendación

1. **Para el stock total de AI MRR → usar `dwh.int_subscription_mrr_breakdown`** con filtro `mrr_julia > 0 OR mrr_sofia > 0` y `status = 'active'`. Esta fuente es la correcta y coincide con lo que el equipo reporta (~$18K en febrero).

2. **Para el waterfall (new/churn/net) → revisar si existen registros con `change_category` distintos a los actuales** en las tablas `revenue_waterfall_julia/sofia` para entender qué churn no está siendo capturado. Query sugerida:

```sql
SELECT DISTINCT change_category
FROM dwh.revenue_waterfall_julia
ORDER BY 1;
```

3. **Posible acción de ingeniería:** Verificar si las tablas `revenue_waterfall_julia/sofia` están excluyendo el churn de suscripciones "graduadas" o si el modelo dbt que las genera tiene un filtro que deja afuera ciertos tipos de baja.

---

## Impacto en el reporte de marzo 2026

| Métrica | Valor correcto | Fuente |
|---------|---------------|--------|
| AI Total MRR Stock Feb | **$17,919** | `int_subscription_mrr_breakdown` |
| AI Total MRR Stock Mar (20d) | **$15,981** | `int_subscription_mrr_breakdown` |
| AI Total MRR Stock Mar forecast | **~$14,900** | Tasa observada -$97/día × 11 días |
| AI New MRR MTD | **+$4,032** | `revenue_waterfall_julia/sofia` |
| AI Churn MTD (visible) | **-$2,985** | `revenue_waterfall_julia/sofia` |
| AI Churn MTD (real implícita) | **~-$6,500** | Derivada de Δ stock real |
| AI Net MRR MTD (real) | **~-$1,938** | `int_subscription_mrr_breakdown` |

El churn **real implícito** de AI en marzo sería ~$6,500 (no $2,985), lo que eleva significativamente el efecto neto.

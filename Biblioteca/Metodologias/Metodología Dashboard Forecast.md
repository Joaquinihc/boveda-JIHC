---
type: metodologia
temas: [forecast, evidence]
fuente: trabajo propio (repo Evidence)
fecha: 2026
---

# Metodología de forecast — Dashboard B2B3

Documento de referencia para **cómo el dashboard** (`dashboard/app.py` + `dashboard/data_layer.py`) calcula **ACTUAL**, **FORECAST fin de mes** y **desviación vs budget** para Leads, Merchants, New MRR, CVR y ARPA.

Implementación: `compute_forecast()`, `set_month_context()`, `PACE_PATTERN` en `dashboard/data_layer.py`.

Definiciones de datos reales y budget: [BUDGET_DEFINITIONS_2026.md](./BUDGET_DEFINITIONS_2026.md).

---

## 1. Alcance

| Qué incluye | Qué no incluye |
|-------------|----------------|
| Proyección de fin de mes para el **mes seleccionado** (si es el mes en curso) | Forecast de CPL, gasto paid, o métricas de ISR (usan otras reglas) |
| Totales globales, por país, MBR por canal (PS / No PS) | Modelo estadístico ARIMA / ML |
| Comparación vs budget del CSV (`data/budget/budget_b2b3_targets.csv`) | Escalar MRR por ratio `leads forecast ÷ leads budget` |

El forecast responde: *“Si el mes sigue cerrando como los meses recientes de B2B3, ¿en cuánto cerramos Leads / Merchants / MRR?”* — no *“¿cerramos el budget si los leads van al ritmo de meta?”*.

---

## 2. Mes cerrado vs mes en curso

`set_month_context(mi)` se ejecuta en cada render con `get_month_info(year, month)`:

| Situación | Forecast |
|-----------|----------|
| Mes **cerrado** (`is_current = false`) | **Forecast = Actual** (no hay extrapolación) |
| Mes **en curso** (`is_current = true`) | Se extrapola con las reglas de la sección 4 |

---

## 3. Bases de tiempo (calendario vs días hábiles)

El dashboard usa **dos relojes distintos**:

### 3.1 Días calendario (Leads)

- `total_days`: días del mes (28–31).
- `elapsed`: día del mes de hoy (si es el mes actual) o último día del mes (si cerrado).
- Para **leads en mes actual**: el numerador del run-rate usa **`elapsed - 1`** (excluye el día de hoy, parcial).

### 3.2 Días hábiles — lunes a viernes (Merchants y MRR)

- `biz_total`: cantidad de días hábiles del mes.
- `biz_elapsed`: días hábiles desde el día 1 hasta el último **día calendario completo** usado para pace (`elapsed - 1` si `elapsed > 1`).

Los cierres (merchants / MRR) se concentran al final del mes; por eso **no** se extrapolan con división lineal por día calendario.

---

## 4. Definición de ACTUAL (MTD)

Filtros comunes B2B3 en warehouse (resumen):

- Segmento: `companies.first_customer_segment_2 = 'B2B3'`.
- Fecha de cierre: `merchants_segments.start_date_pago_mx` en `[date_start, date_end)` del mes elegido.
- Emails internos/test excluidos (`@agendapro`, `@yopmail`).
- Leads: ver exclusiones en `fetch_leads` (OFFLINE, México `otro`, `contact_center_owner`, tamaños `3-5` / `6-15` / `16+`).

### 4.1 Leads

- Fuente: `dwh.fct_lead_conversions` + `dwh.stg_hubspot__contacts`.
- **ACTUAL** = conteo MTD en el rango del mes.

### 4.2 Merchants

- Fuente: `dwh.merchants_segments` + primer pago en `dwh.mrr`.
- **ACTUAL** = `SUM(plan_quantity)` por país (licencias / cantidad de plan en el cierre).

### 4.3 New MRR — dos cifras en UI

| Campo en código | Significado | Uso en dashboard |
|-----------------|-------------|------------------|
| `mrr` / `mrr_list` | **Precio lista** (cupón de adquisición revertido cuando hay datos Chargebee) | KPI principal, forecast, **vs budget** |
| `mrr_billed` | **MRR neto facturado** primer mes (`mrr_usd` en `dwh.mrr`) | Subtítulo “Fact. 1.er mes” en MBR |

Join a `dwh.mrr` (alineado a `analysis/b2b3_ventas_abril2026_mercado_mrr_neto.sql`):

```sql
mr.is_first_month_paying IS TRUE
AND mr.mrr_usd > 0
AND mr.date_month = date_trunc('month', ms.start_date_pago_mx)::date
```

**Lista vs facturado** (cuando existe `dwh.chargebee_subscriptions_coupons`):

- `mrr_billed` = `mrr_usd` cobrado en el primer mes.
- `mrr_list` = precio lista estimado:
  - Sin descuento: igual a facturado.
  - Con `%` descuento: `mrr_usd / (1 - pct/100)`.
  - Descuento 100%: `2 × sub_mrr_usd` (regla budget 2026).

Si la tabla de cupones no existe o falla el join, lista = facturado (`DASHBOARD_SKIP_CHARGEBEE_COUPON_JOIN=true` fuerza ese modo).

---

## 5. Fórmulas de FORECAST fin de mes

Función central: `compute_forecast(actual, biz_elapsed, kpi_type)`.

### 5.1 Leads — extrapolación **lineal** (días calendario)

\[
\text{Forecast Leads} = \text{round}\left(\frac{\text{Leads MTD}}{\text{días calendario transcurridos}} \times \text{días totales del mes}\right)
\]

- Denominador: `_MONTH_CAL_ELAPSED` (día de hoy excluido en mes actual).
- Numerador: `_MONTH_CAL_TOTAL`.

Hipótesis: los leads entran de forma relativamente uniforme a lo largo del mes (incluye fines de semana en el run-rate).

### 5.2 Merchants — extrapolación **no lineal** (curva de pace)

\[
\text{Forecast Merchants} = \text{round}\left(\frac{\text{Merchants MTD}}{p_{\text{merchants}}(\text{BD})}\right)
\]

donde \(p_{\text{merchants}}\) = fracción acumulada esperada del total mensual al día hábil `BD` transcurrido, según `PACE_PATTERN["merchants"]`.

- Mínimo en denominador: **3%** (`max(pct, 0.03)`) para evitar explosiones si el mes va muy lento al inicio.

### 5.3 New MRR — misma curva que Merchants

\[
\text{Forecast MRR} = \text{round}\left(\frac{\text{MRR lista MTD}}{p_{\text{merchants}}(\text{BD})}\right)
\]

**Importante:** aunque existe `PACE_PATTERN["mrr"]` en código, el forecast usa explícitamente la curva **`merchants`** (`curve = "merchants"` en `compute_forecast` cuando `kpi_type == "mrr"`).

Motivo de producto: el New MRR del mes se reconoce en el mismo ritmo que los **cierres** (merchants); una curva MRR separada había desalineado magnitudes vs el cierre real del mes.

**Base del numerador:** siempre **MRR lista** (`total_mrr`), comparable al `new_mrr_budget` del CSV.

**No se usa:** `budget_mrr × (forecast_leads / budget_leads)` ni extrapolación lineal por día calendario sobre USD.

### 5.4 CVR y ARPA “del forecast” (derivados, no proyectados aparte)

No hay curva propia para CVR ni ARPA. Se calculan **después** del forecast de volumen:

| Métrica | Fórmula |
|---------|---------|
| **CVR implícito forecast** | `forecast_merchants / forecast_leads × 100` |
| **ARPA implícito forecast** | `forecast_mrr / forecast_merchants` |

En MBR, la desviación de CVR vs budget en fila **TOTAL país** usa CVR implícito del forecast país menos `merchants ÷ leads` del budget agregado. Por **canal**, se compara vs `cvr_meta` del roster ISR (ver `mbr_channel_cvr_target`).

---

## 6. Curva `PACE_PATTERN` (calibración histórica)

Patrón de **acumulado** al índice de día hábil 1…23 (referencia histórica ~Nov 2025 – Feb 2026):

- A los **23** días hábiles de referencia, el acumulado = **100%** del mes.
- Los cierres se concentran en la **segunda mitad** y última semana del mes (p. ej. ~43% del total de merchants en la semana 4 en enero).

### Escalado al largo del mes real

`_scaled_pace_cumulative(biz_elapsed, biz_total, kpi_type)`:

1. Calcula `frac_month = biz_elapsed / biz_total` (progreso en el mes actual).
2. Mapea a posición en la curva 1…23: `v = frac_month × 23`.
3. Interpola linealmente entre puntos de la tabla `PACE_PATTERN[kpi_type]`.

Así febrero (menos días hábiles) y marzo (más días hábiles) usan la **misma forma** de curva, ajustada al `biz_total` del mes.

Valores de referencia (merchants / mrr — mismos números):

| BD | % acumulado del mes |
|----|---------------------|
| 5 | ~19% |
| 10 | ~41% |
| 15 | ~63% |
| 20 | ~90% |
| 23 | 100% |

Curva **leads** en `PACE_PATTERN["leads"]` existe pero **no** se usa en `compute_forecast` (leads van por línea calendario).

---

## 7. Desviación vs Budget (columna “vs BUDGET” del MBR)

Para **Leads, Merchants y MRR**:

\[
\Delta\% = \frac{\text{Forecast} - \text{Budget}}{\text{Budget}} \times 100
\]

Implementación: `delta_pct(forecast, budget)` en `data_layer.py`.

- **Budget MRR** del CSV = definición **lista** (comparable a `mrr` / `mrr_list` del dashboard).
- La comparación de desviación MRR es **forecast lista vs budget lista**, no facturado vs budget.

Para **CVR** (puntos porcentuales, no `delta_pct`):

\[
\Delta \text{CVR (pp)} = \text{CVR forecast implícito} - \text{CVR budget implícito}
\]

Para **ARPA**: `delta_pct(arpa_forecast, arpa_budget)`.

---

## 8. MBR: Quarter / YTD (meses cerrados + mes actual)

En vistas **Quarter** o **YTD** del MBR:

1. Meses **anteriores** al mes seleccionado: se suman **actuales cerrados** (sin forecast).
2. **Mes en curso** dentro del rango: solo esa porción se proyecta con `compute_forecast` sobre el delta MTD del mes actual.

Helper `_fcst_mbr` en `app.py`:

```text
forecast_total = past_closed_actual + compute_forecast(current_mtd_only, ...)
```

---

## 9. Atribución Paid Social (MBR y análisis)

El **forecast** no cambia por canal; la **atribución** sí afecta el ACTUAL por fila:

- **Leads PS:** `UPPER(hs_analytics_source) = 'PAID_SOCIAL'`.
- **Merchants PS:** `UPPER(COALESCE(ms.hs_analytics_source, hubspot_email)) = 'PAID_SOCIAL'`.
  - Si `merchants_segments.hs_analytics_source` es NULL → fallback por email a `stg_hubspot__contacts` (contacto más reciente por `id DESC`).

Consultas de validación: `analysis/b2b3_paid_social_ventas_abril2026.sql`, `analysis/b2b3_ventas_abril2026_mercado_mrr_neto.sql`.

---

## 10. Resumen ejecutivo (una tabla)

| KPI | ACTUAL (MTD) | Forecast fin de mes | Base tiempo | vs Budget |
|-----|----------------|---------------------|-------------|-----------|
| **Leads** | Conteo conversiones | Lineal: MTD ÷ días cal × días mes | Calendario | Δ% forecast vs budget leads |
| **Merchants** | Σ plan_quantity | MTD ÷ curva pace merchants | Días hábiles | Δ% forecast vs budget merchants |
| **MRR lista** | Σ mrr_list | MTD lista ÷ **misma** curva merchants | Días hábiles | Δ% forecast vs budget MRR |
| **MRR fact.** | Σ mrr_billed | (solo informativo en UI) | — | No es la base del forecast |
| **CVR** | merchants ÷ leads | fcst_merchants ÷ fcst_leads | Derivado | pp o vs meta canal |
| **ARPA** | mrr ÷ merchants | fcst_mrr ÷ fcst_merchants | Derivado | Δ% vs budget ARPA |

---

## 11. Limitaciones y supuestos

1. **Un solo patrón histórico** de pace (4 meses); no se segmenta por país ni por canal en la curva.
2. **Mes muy lento al inicio** puede inflar forecast si el denominador cae al piso del 3%.
3. **VPN / timeout DWH** no alteran la fórmula; solo impiden cargar datos.
4. **Budget** puede no coincidir 100% con lista si en DWH faltan cupones (modo sin ajuste lista).
5. **CVR/ARPA forecast** heredan el error de las tres proyecciones base; no son modelos independientes.

---

## 12. Referencias en el repo

| Recurso | Ruta |
|---------|------|
| Motor de forecast | `dashboard/data_layer.py` → `PACE_PATTERN`, `_scaled_pace_cumulative`, `compute_forecast`, `set_month_context` |
| UI / MBR | `dashboard/app.py` |
| Definiciones budget | `docs/BUDGET_DEFINITIONS_2026.md` |
| MRR neto por mercado (SQL) | `analysis/b2b3_ventas_abril2026_mercado_mrr_neto.sql` |
| Paid Social por mercado (SQL) | `analysis/b2b3_paid_social_ventas_abril2026.sql` |
| Pace merchants en `merchants_segments` | `models/agendapro/gmv_and_gpv/merchants_segments.sql` (hubspot `ORDER BY id DESC`) |

---

*Última revisión alineada al código del dashboard B2B3 Executive (rama local). Si cambia `compute_forecast` o el join a `mrr`, actualizar este documento.*

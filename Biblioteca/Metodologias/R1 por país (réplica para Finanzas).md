---
type: metodologia
temas: [cx, r1, activacion]
fuente: trabajo propio
fecha: 2026
---

# R1 por país: réplica para Finanzas

## Objetivo

Replicar el indicador mostrado en CX:

> **R1 por país — % realizada en ≤7 días, semana a semana (12 semanas)**

Por cada país, mide qué porcentaje de las nuevas cuentas **B2B3** tuvo su primera asesoría (R1) dentro de los siete días desde su primer pago.

La unidad de análisis es la **empresa** (`company_id`), no la cantidad de locales, contratos ni MRR.

## Definición exacta

Para cada empresa del cohorte:

| Campo | Regla |
|---|---|
| Cohorte | Semana ISO (lunes a domingo) de `start_date_pago` / primer pago |
| País | `dwh.merchants_segments.country` |
| Segmento | `segment2 = 'B2B3'` |
| Pagador nuevo real | Debe existir al menos una fila en `dwh.mrr` con `is_first_month_paying = true` |
| Cuenta evaluable | Al corte, ya pasaron al menos 7 días calendario desde el primer pago |
| R1 | Primera asesoría identificada para la empresa; ver [Obtención de la R1](#obtención-de-la-r1) |
| R1 a tiempo | `GREATEST(DATEDIFF(day, first_payment, r1_date), 0) <= 7` |
| Resultado | `R1 a tiempo / cuentas evaluables` |

Fórmula:

```text
% R1 ≤7d = 100 × empresas_con_R1_en_0_a_7_días / empresas_evaluables
```

La fecha de ejecución debe evaluarse en la zona horaria **America/Santiago**, igual que la aplicación de CX.

## Alcance y reglas que no se deben cambiar

1. Mostrar siempre los cuatro países: **Chile, Argentina, Colombia y México**.
2. Usar las últimas **12 semanas ISO**, incluida la semana en curso; el lunes es la fecha que se muestra en el eje y en la tabla.
3. No excluir una empresa porque luego haya churneado: si era parte del cohorte y ya era evaluable, permanece en el denominador.
4. No exigir MRR vigente ni estado activo. El flag histórico `is_first_month_paying` es el que evita incluir trials, freemium y reactivaciones.
5. No usar `dwh.companies.customer_segment_2` como sustituto. Para este indicador, la fuente del segmento es `dwh.merchants_segments.segment2`.
6. El cálculo no depende de filtros de país de otro tablero. Cada país se calcula sobre su propia cohorte.
7. El SLA operativo general de las apps de onboarding es 5 días, pero **este indicador ejecutivo usa 7 días**. No cambiarlo a 5 días al replicarlo.
8. Una R1 ocurrida antes del primer pago se considera con demora `0` por la lógica vigente (`GREATEST(..., 0)`) y por tanto cuenta como “a tiempo”.

## Obtención de la R1

La R1 se obtiene por empresa usando esta prioridad. Dentro de cada fuente se conserva la fecha más antigua, pero una R1 de DIIO tiene precedencia sobre el respaldo de SegmentsPro:

1. **DIIO:** primera reunión DIIO que se pueda asociar a la empresa.
2. **Respaldo SegmentsPro:** si no hay DIIO, la tarea completada más antigua:
   - Journey de Onboarding: `stage_condition_id = 103`, “Primera asesoría realizada”.
   - Journey de Pre-onboarding: `stage_condition_id = 430`, “Reunión Set up”.

En ambos casos, la fecha efectiva de R1 es:

```text
r1_date = COALESCE(first_diio_meeting_date, first_journey_r1_date)
```

### Match de reuniones DIIO a empresa

Las tablas DIIO viven en PostgreSQL (`dwh.diio_*`), no en Redshift. Para no subcontar R1, el proceso debe resolver cada reunión en este orden:

1. `company_id` presente en el título de la reunión.
2. Email externo exacto de un participante contra emails de HubSpot o `company_locations`.
3. Dominio corporativo único del email externo; excluir dominios genéricos como `gmail.com`, `hotmail.com`, `outlook.com`, etc.
4. Nombre normalizado de la empresa contenido en el título.
5. Asignación manual, si existe.

No filtrar DIIO por `meeting_type`: asesorías válidas llegan con tipos diversos. Las reuniones resueltas a una empresa fuera de este cohorte no cuentan.

El resultado mínimo que Finanzas debe dejar disponible antes de la agregación final es:

```text
finance_stage.r1_first_by_company
  company_id       bigint
  r1_date          date
  r1_source        text  -- 'diio', 'journey_onb' o 'journey_preonb'
```

Debe haber **a lo sumo una fila por `company_id`**. Si existen varias reuniones DIIO válidas, conservar la de menor `scheduled_at`.

## Paso 1: cohorte y respaldo de R1 en Redshift

Ejecutar la siguiente consulta en Redshift. Reemplace `:as_of_date` por la fecha de corte local (por ejemplo, `CURRENT_DATE` si el job se ejecuta en horario de Santiago).

```sql
WITH params AS (
    SELECT CAST(:as_of_date AS date) AS as_of_date
),
period AS (
    SELECT
        DATEADD(week, -11, DATE_TRUNC('week', as_of_date))::date AS first_week,
        DATEADD(week, 1, DATE_TRUNC('week', as_of_date))::date AS next_week
    FROM params
),
base AS (
    SELECT DISTINCT
        ms.company_id,
        ms.country,
        ms.start_date_pago::date AS first_payment,
        DATE_TRUNC('week', ms.start_date_pago)::date AS week_start
    FROM dwh.merchants_segments ms
    CROSS JOIN period p
    WHERE ms.segment2 = 'B2B3'
      AND ms.country IN ('Chile', 'Argentina', 'Colombia', 'México')
      AND ms.start_date_pago >= p.first_week
      AND ms.start_date_pago < p.next_week
      AND EXISTS (
          SELECT 1
          FROM dwh.mrr m
          WHERE m.company_id = ms.company_id
            AND m.is_first_month_paying IS TRUE
      )
),
journey_r1 AS (
    SELECT company_id, r1_date, r1_source
    FROM (
        SELECT
            sc.ag_company_id::bigint AS company_id,
            csc.end_date::date AS r1_date,
            CASE
                WHEN csc.stage_condition_id = 430 THEN 'journey_preonb'
                ELSE 'journey_onb'
            END AS r1_source,
            ROW_NUMBER() OVER (
                PARTITION BY sc.ag_company_id
                ORDER BY csc.end_date ASC
            ) AS rn
        FROM staging.stg_segmentspro__company_stage_conditions csc
        JOIN staging.stg_segmentspro__company_stages cst
          ON cst.id = csc.company_stage_id
        JOIN staging.stg_segmentspro__company_journeys cj
          ON cj.id = cst.company_journey_id
        JOIN staging.stg_segmentspro__companies sc
          ON sc.id = cj.company_id
        CROSS JOIN period p
        WHERE csc.stage_condition_id IN (103, 430)
          AND csc.status = 'completed'
          AND csc.end_date IS NOT NULL
          AND csc.discarded_at IS NULL
          AND sc.ag_company_id IS NOT NULL
          -- Incluye R1 de pre-onboarding anterior al pago.
          AND csc.end_date >= DATEADD(day, -60, p.first_week)
    ) ranked
    WHERE rn = 1
)
SELECT
    b.company_id,
    b.country,
    b.first_payment,
    b.week_start,
    j.r1_date AS journey_r1_date,
    j.r1_source AS journey_r1_source
FROM base b
LEFT JOIN journey_r1 j
  ON j.company_id = b.company_id;
```

Materializar el resultado como `finance_stage.r1_cohort_base` o el equivalente en el modelo de Finanzas.

## Paso 2: materializar la R1 DIIO y aplicar el respaldo

1. Extraer desde PostgreSQL las reuniones de `dwh.diio_meetings_with_transcripts` desde `first_week - 60 días`.
2. Extraer los emails externos desde `dwh.diio_meeting_attendees` y `dwh.diio_meeting_users`.
3. Cruzarlas contra las empresas y los emails de la cohorte; aplicar la prioridad de match descrita arriba.
4. Para cada empresa, obtener `MIN(scheduled_at::date)` como `first_diio_meeting_date`.
5. Combinar DIIO y SegmentsPro:

```sql
SELECT
    b.company_id,
    COALESCE(d.first_diio_meeting_date, b.journey_r1_date) AS r1_date,
    CASE
        WHEN d.first_diio_meeting_date IS NOT NULL THEN 'diio'
        WHEN b.journey_r1_date IS NOT NULL THEN b.journey_r1_source
    END AS r1_source
FROM finance_stage.r1_cohort_base b
LEFT JOIN finance_stage.diio_first_meeting_by_company d
  ON d.company_id = b.company_id;
```

Guardar este resultado deduplicado como `finance_stage.r1_first_by_company`.

## Paso 3: agregación final

Esta consulta asume que el paso anterior dejó disponible `finance_stage.r1_first_by_company`.

```sql
WITH params AS (
    SELECT CAST(:as_of_date AS date) AS as_of_date
),
by_company AS (
    SELECT
        b.week_start,
        b.country,
        b.company_id,
        b.first_payment,
        r.r1_date,
        CASE
            WHEN DATEADD(day, 7, b.first_payment) <= p.as_of_date THEN 1
            ELSE 0
        END AS is_eligible,
        CASE
            WHEN r.r1_date IS NOT NULL
             AND GREATEST(DATEDIFF(day, b.first_payment, r.r1_date), 0) <= 7
            THEN 1
            ELSE 0
        END AS is_r1_in_7d
    FROM finance_stage.r1_cohort_base b
    CROSS JOIN params p
    LEFT JOIN finance_stage.r1_first_by_company r
      ON r.company_id = b.company_id
)
SELECT
    week_start,
    country,
    SUM(is_eligible) AS evaluables,
    SUM(CASE WHEN is_eligible = 1 AND is_r1_in_7d = 1 THEN 1 ELSE 0 END) AS r1_a_tiempo,
    ROUND(
        100.0 * SUM(CASE WHEN is_eligible = 1 AND is_r1_in_7d = 1 THEN 1 ELSE 0 END)
        / NULLIF(SUM(is_eligible), 0),
        1
    ) AS pct_r1_7d
FROM by_company
GROUP BY 1, 2
ORDER BY 1, 2;
```

Para que la tabla siempre tenga los cuatro países, generar un calendario de 12 semanas × los cuatro países y hacer `LEFT JOIN` contra este resultado. Cuando `evaluables = 0`, mostrar `—`, no `0%`.

## Criterio de madurez para resúmenes semanales

El gráfico y la tabla pueden incluir la semana actual como preliminar. Sin embargo, para una tarjeta de “última semana cerrada” o comparaciones semanales, usar solamente cohortes maduras:

```text
week_start <= as_of_date - 13 días
```

La razón es que una cohorte que comienza un lunes termina el domingo (`+6 días`) y sus últimas cuentas recién agotan el plazo de 7 días al día 13. La última semana madura es la última con `evaluables > 0` que cumpla esa condición.

El promedio de cuatro semanas debe ser ponderado, nunca el promedio simple de porcentajes:

```text
100 × SUM(r1_a_tiempo de las últimas 4 cohortes maduras)
    / SUM(evaluables de las últimas 4 cohortes maduras)
```

## Controles de calidad antes de publicar

1. **Grano:** `COUNT(*) = COUNT(DISTINCT company_id)` en `r1_cohort_base` y en `r1_first_by_company`.
2. **Cohorte:** todas las filas son `segment2 = 'B2B3'`, pertenecen a uno de los cuatro países y tienen `is_first_month_paying = true`.
3. **Elegibilidad:** ninguna cuenta con menos de 7 días desde el pago entra al denominador.
4. **Fuente de R1:** reportar la cobertura por `r1_source` (`diio`, `journey_onb`, `journey_preonb`, nulo) para detectar caídas de integración.
5. **Sanidad de porcentaje:** `0 <= pct_r1_7d <= 100`; cuando el denominador sea cero, el valor debe ser nulo.
6. **Revisión de trazabilidad:** conservar una tabla por empresa con `company_id`, `first_payment`, `week_start`, `country`, `r1_date`, `r1_source`, `days_to_r1`, `is_eligible` e `is_r1_in_7d`.
7. **Comparación inicial:** para una misma fecha de corte, comparar por semana y país contra el tablero CX. Las diferencias deben investigarse a nivel de `company_id`, especialmente en el match de DIIO.

## Nota sobre el backlog “Sin R1”

El backlog no es necesario para reproducir el porcentaje de la imagen. Si Finanzas también lo publica, debe contar solo empresas **vivas** sin R1 y con más de siete días desde el pago. La condición “viva” debe obtenerse desde la misma verdad de suscripciones Chargebee; no se debe inferir desde `mrr_usd` ni desde la ausencia de `cancelled_at`.

Una empresa con R1 tardía (>7 días) baja el porcentaje de cumplimiento, pero no entra en ese backlog.

Por eso no se debe imponer la igualdad:

```text
R1 a tiempo + sin R1 = evaluables
```

La igualdad no es válida cuando existen R1 tardías. Para conciliar completamente el denominador, agregar una categoría separada:

```text
R1 tardía = evaluable AND r1_date IS NOT NULL AND days_to_r1 > 7
```

Entonces:

```text
R1 a tiempo + R1 tardía + sin R1 = evaluables
```

## Fuentes canónicas y referencia de implementación

- Cohorte B2B3 y país: `dwh.merchants_segments`.
- Validación de pagador nuevo real: `dwh.mrr.is_first_month_paying`.
- Primer pago alternativo/auditoría: `dwh.first_paying_date.start_date_pago`.
- Respaldo de R1: tablas `staging.stg_segmentspro__company_*`.
- R1 principal: tablas PostgreSQL `dwh.diio_*`.
- Implementación de referencia: `cx-apps/apps/cx-head-app/data_layer.py` e `index.html`.

La fuente de datos y la definición anterior reflejan la implementación vigente del tablero, no una estimación visual del gráfico.

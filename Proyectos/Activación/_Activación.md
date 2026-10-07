---
type: proyecto
estado: activo
temas: [activacion, definiciones, evidence, dwh]
inicio: 2026-09-28
responsable: "[[Joaquín Herrera]]"
---

# Activación

**Objetivo**: Dejar una sola definición de activación (y de Activation Velocity) entre Evidence y dbt: qué reservas cuentan, sobre qué universo y desde qué día, para que el OKR, el board book, CX y los waterfalls midan lo mismo.

## Contexto
- 2026-09-28 · [[Pablo Lucero]] alinea Activation Velocity con New Merchants B2B3 (PR 769 de Evidence), tras conciliarla con el semanal de CX de [[Pablo Santa Inés]]: primer mes pagado, sin cuentas test, día 0 `start_date_pago_mx`, reservas + clases. Mantiene la ponderación por sedes y no adopta la regla de cobros de CX.
- 2026-09-29 · PR 772 de Evidence: las mismas reglas en Activation → Retention, Lead → Activación, PMF Fitness y R1. PR 774: curva de Activation Velocity de Fitness (por defecto, todas las inscripciones).
- 2026-09-29 · PR 412 de dbt: las tablas de velocity entran a la corrida diaria (antes estaban congeladas al 7-sep).
- 2026-09-30 · PR 414 de dbt: `bookings_count_demand`, sin las reservas importadas de otro software. El PR 415 (abierto al 7-oct) lo usa en `int_company_booking_cohorts` e `int_company_activation_status`.
- Estado al 2026-10-07: conviven tres definiciones de "activado" (Evidence, reportes dbt de velocity e `int_company_activation_status`). El detalle está en [[Activación — fuentes y definiciones vigentes]].

## Próximos pasos
Las acciones viven en las tareas del proyecto (lista abajo). La principal es [[Arreglar definición de activación]].

## Referencias
- Definiciones y fuentes: [[Activación — fuentes y definiciones vigentes]] · queries: [[SQL — Activación (importadas y puente Evidence vs dbt)]].
- Fichas oficiales (copia de la skill `finance-metric-definitions`): [[Activation]] · [[Activation Velocity]] · [[Impersonating bookings (personified)]].
- Metodologías que usan la activación: [[Metodología Churn B2B3]] (graduado vs onboarding) · [[R1 por país (réplica para Finanzas)]].
- Proyectos relacionados: [[_Churn y Graduación]] (la graduación parte de la activación) · [[_Vertical Fitness]] (enrollments de clases).

## Notas de este proyecto
Los análisis viven en esta carpeta con ID `YYYYMMDD-NN Título`.

```base
filters:
  and:
    - type == "analisis"
    - file.inFolder("Proyectos/Activación")
views:
  - type: table
    name: Análisis
    order:
      - file.name
      - fecha
      - estado
      - temas
    sort:
      - property: fecha
        direction: DESC
```

## Tareas de este proyecto
```base
filters:
  and:
    - type == "tarea"
    - file.inFolder("Tareas")
    - proyecto.contains("_Activación")
formulas:
  orden_prioridad: if(prioridad == "alta", 1, if(prioridad == "media-alta", 2, if(prioridad == "media", 3, if(prioridad == "media-baja", 4, if(prioridad == "baja", 5, 6)))))
  orden_tiempo: if(tiempo == 1, 1, if(tiempo == 2, 2, if(tiempo == 3, 3, if(tiempo == 4, 4, if(tiempo == 5, 5, 9)))))
  tiempo_txt: if(tiempo == 1, "⏱ ≤2 h", if(tiempo == 2, "⏱ ½ día", if(tiempo == 3, "⏱ 1 día", if(tiempo == 4, "⏱ 2–3 días", if(tiempo == 5, "⏱ 1 sem+", "")))))
properties:
  formula.tiempo_txt:
    displayName: Tiempo
views:
  - type: table
    name: Vivas
    filters:
      and:
        - estado != "hecho"
        - estado != "descartada"
    order:
      - id
      - file.name
      - estado
      - prioridad
      - formula.tiempo_txt
      - fecha_limite
    sort:
      - property: formula.orden_prioridad
        direction: ASC
      - property: formula.orden_tiempo
        direction: ASC
  - type: table
    name: Hechas
    filters:
      and:
        - estado == "hecho"
    order:
      - id
      - file.name
      - creado
    sort:
      - property: creado
        direction: DESC
```

## Borradores en Inbox
PRs, hilos de Slack y derivas del proyecto que todavía no se revisan.

```base
filters:
  and:
    - file.inFolder("Inbox")
    - proyecto.contains("_Activación")
views:
  - type: table
    name: Inbox
    order:
      - file.name
      - type
      - creado
      - revisar
    sort:
      - property: creado
        direction: DESC
```

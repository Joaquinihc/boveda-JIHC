---
type: proyecto
estado: activo
temas: [fitness, activacion, bookings]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
---

# Vertical Fitness

**Objetivo**: Métricas y P&L de la vertical Fitness: bookings, activación, dashboard y (eventual) CECO propio.

## Contexto
- Bookings Fitness: tabla en DWH casi lista; enrollments en dbt sin filtro de primer pago (definición documentada pendiente).
- Activación con Fitness debe entrar a tablas de churn graduado/downboarding y Activation Velocity.
- CECO propio: se cree que Fitness será vertical en ~3 meses; decisión pendiente con Pablo (riesgo sueldos visibles).
- Tareas Notion: Activation con fitness, Fitness dashboard, Bookings fitness, Nicho Centro Deportivo.

## Próximos pasos
- [ ] Definir metodología de activación (4º mes activo, booking semanal) y alinear con la propuesta existente.
- [ ] Ejecutar modelos de enrollments y validar.

## Notas de este proyecto
Los análisis viven en esta carpeta con ID de fecha (`YYYYMMDD-NN Título`). Las reuniones relacionadas se linkean desde `Reuniones/` vía frontmatter `proyectos:`.


## Tareas de este proyecto
```base
filters:
  and:
    - type == "tarea"
    - file.inFolder("Tareas")
    - estado != "hecho"
    - estado != "descartada"
    - proyecto.contains("_Vertical Fitness")
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
```

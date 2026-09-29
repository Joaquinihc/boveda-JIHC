---
type: proyecto
estado: activo
temas: [margenes, pnl, unit-economics]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
notion: https://app.notion.com/3d6ab34c233b805091dde8c188a8591e
---

# Márgenes por Cliente y Producto

**Objetivo**: Calcular márgenes por tiempo, por cliente y por producto (AI y SaaS), distribuyendo costos de plataforma por bookings/sucursales.

## Contexto
- Prioridad asignada el 9-sep (tablero FP&A Priorities: High, in progress). Márgenes objetivo: ~5% productos de pago, ~10% compañía.
- Avance con Claude en modo planning: ~8 modelos dbt propuestos — hay que simplificar.
- Costos de plataforma se distribuyen en razón de bookings y sucursales como punto de partida; uso de features es difícil de medir (14.000 de ~16.700 activas).
- Regla acordada: si el impacto retroactivo es significativo, no se corrige hacia atrás.

## Próximos pasos
- [ ] Simplificar el diseño de modelos dbt y mandar PR.
- [ ] Definir distribución de costos de plataforma (bookings) y validar con Pablo/Nicolás.

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
    - proyecto.contains("_Márgenes por Cliente y Producto")
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

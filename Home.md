---
type: home
cssclasses: [wide]
---

# 🏠 Bóveda FP&A

> Carpetas para lo excluyente, tags para lo transversal. Las vistas de abajo son Bases (se generan solas desde el frontmatter).

## Proyectos activos
```base
filters:
  and:
    - type == "proyecto"
    - estado == "activo"
views:
  - type: table
    name: Activos
    order:
      - file.name
      - temas
      - file.mtime
```

## Análisis recientes
```base
filters:
  and:
    - type == "analisis"
views:
  - type: table
    name: Últimos análisis
    order:
      - file.name
      - proyecto
      - fecha
      - estado
    sort:
      - property: fecha
        direction: DESC
    limit: 15
```

## Reuniones por verificar
```base
filters:
  and:
    - type == "reunion"
    - verificado == false
views:
  - type: table
    name: Pendientes
    order:
      - file.name
      - fecha
      - temas
    sort:
      - property: fecha
        direction: DESC
    limit: 15
```

## Mis tareas (activas)
```base
filters:
  and:
    - type == "tarea"
    - file.inFolder("Tareas")
    - estado != "hecho"
    - estado != "descartada"
formulas:
  orden_prioridad: if(prioridad == "alta", 1, if(prioridad == "media-alta", 2, if(prioridad == "media", 3, if(prioridad == "media-baja", 4, if(prioridad == "baja", 5, 6)))))
  orden_tiempo: if(tiempo == 1, 1, if(tiempo == 2, 2, if(tiempo == 3, 3, if(tiempo == 4, 4, if(tiempo == 5, 5, 9)))))
  tiempo_txt: if(tiempo == 1, "⏱ ≤2 h", if(tiempo == 2, "⏱ ½ día", if(tiempo == 3, "⏱ 1 día", if(tiempo == 4, "⏱ 2–3 días", if(tiempo == 5, "⏱ 1 sem+", "")))))
properties:
  formula.tiempo_txt:
    displayName: Tiempo
views:
  - type: table
    name: Activas
    order:
      - id
      - file.name
      - estado
      - prioridad
      - formula.tiempo_txt
      - proyecto
      - fecha_limite
    sort:
      - property: formula.orden_prioridad
        direction: ASC
      - property: formula.orden_tiempo
        direction: ASC
    limit: 15
```
➡️ Tablero completo: [[Tareas/Tablero de Tareas.base|Tablero de Tareas]]

## Por revisar (clasificación dudosa)
```base
filters:
  and:
    - revisar == true
views:
  - type: table
    name: Cola de curaduría
    order:
      - file.name
      - type
      - creado
    limit: 15
```

## Accesos directos
- 📥 [[Inbox/_acerca del Inbox|Inbox]] · 📓 Journal (nota diaria de hoy: `Ctrl/Cmd+P → Open today's daily note`)
- 📚 Biblioteca: Metricas · SQL · Metodologias · Recursos
- ✅ Tareas: [FP&A Priorities en Notion](https://app.notion.com/p/agendapro/FP-A-Priorities-2abab34c233b806a9e0ae0c39fdc9506)
- 🎙️ [Transcripciones Meet](https://app.notion.com/p/agendapro/Transcripciones-Meet-2b0ab34c233b804fb240d9b882320681)

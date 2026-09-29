---
type: proyecto
estado: activo
temas: [waterfall, revenue, otc]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
notion: https://app.notion.com/394ab34c233b8097bd53f205989384b4
---

# Waterfall One-Time Charges

**Objetivo**: Incorporar los one-time charges al Revenue Waterfall, separados del MRR, sin romper la historia.

## Contexto
- Problema: ~26 empresas y ~$26K (mar-ago) en quantum/one-time charges fuera del Waterfall; cupones 100% sobre addons generan recurrente nulo.
- Acordado (1-sep): agregarlos separados del MRR afectando el movimiento de categorías; definir tratamiento de overrides (mes de cobro vs recurrente).
- Nota histórica relacionada: decisión mrr_pos en revenue_waterfall_saas (Slack #downgrades-monitor, 17-ago).

## Próximos pasos
- [ ] Implementar OTC separado del MRR en la tabla del Waterfall.
- [ ] Definir si los overrides son recurrentes o one-time.

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
    - proyecto.contains("_Waterfall One-Time Charges")
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

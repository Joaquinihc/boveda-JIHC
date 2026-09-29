---
type: proyecto
estado: activo
temas: [presupuesto, forecast]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
notion: https://app.notion.com/3c0ab34c233b8061ad3bec2783d94a5b
---

# Presupuesto 2027

**Objetivo**: Construir el presupuesto 2027: arranca en octubre, ~2 meses, presentación en el directorio de diciembre.

## Contexto
- Para Payments: curva de activación + churn consensuados (no proyección lineal); supuesto de POS por compañía para compras de hardware y GNB/GPV.
- Convergencia de las dos métricas de venta POS al estándar definitivo ocurre en este presupuesto.
- Metodologías de apoyo: [[Metodología Forecasting (general)]], [[Metodología Forecasting MTD]], [[Metodología ISI (índice estacional intra-mes)]].

## Próximos pasos
- [ ] Definir calendario e insumos por área (septiembre).
- [ ] Modelar curva de activación y churn de Payments.

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
    - proyecto.contains("_Presupuesto 2027")
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

---
type: proyecto
estado: activo
temas: [mdr, adquisicion, retencion, cupones]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
---

# MDR AE Change

**Objetivo**: Medir el impacto del cambio de proceso de ventas (una persona + cupón 3 meses) en conversión, activación y retención.

## Contexto
- Hallazgos (16-sep): conversión subió (CL +14%, MX +37%, CO +106%), retención a 6 meses bajó; cupón 50% asociado a la mayor caída. Confundentes: cupón simultáneo al cambio, mix de canales y verticales.
- Pedido por Rob; presentación compartida con Matías, Julio y Carlos.

## Próximos pasos
- [ ] Refinar: reconversiones, atribución al último lead, desagregado por canal y vertical.
- [ ] Evaluar CAC total nuevo vs anterior.
- [ ] Compartir reporte en Claude Artifacts.

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
    - proyecto.contains("_MDR AE Change")
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

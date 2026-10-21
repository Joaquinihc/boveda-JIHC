---
type: proyecto
estado: activo
temas: [okrs, reporting]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
notion: https://app.notion.com/321ab34c233b80cdafdad8b83fa163dc
---

# OKRs Finanzas

**Objetivo**: Mantener el tablero de OKRs/KRs al día, con semáforo en el Informe Desayuno y responsables claros.

## Contexto
- Semáforo por completitud de KRs (lógica de colores a replantear); OKRs de People extendidos a 2027 (definir tratamiento).
- OKR de uso de IA interno → Sebastián Hevia; Core Experience → Pablo (Marambio/Lucero según nota).
- "No Audit Observations" movido a octubre; responsabilidad por aclarar con Nicolás Astudillo.
- 2026-10-21 · Javiera Correa mantiene una planilla en Google Sheets con los KRs de People; Joaquín la copia a la planilla OKR que se carga al DWH (los líderes no tienen acceso a la planilla OKR).
- 2026-10-21 · 6 KRs de People son trimestrales, con meta solo en septiembre y diciembre (KR10.1, KR10.4, KR11.1, KR12.4, KR13.1, KR13.2); sep-26 fue su primera medición.
- 2026-10-21 · En el Informasayuno del 16-oct no se presentaron los OKRs: el tiempo se usó para presentar el proyecto Pulse y la compra de Reservo.

## Próximos pasos
- [ ] Actualización manual de KRs pendientes (Miguel, Alan).
- [ ] Aclarar responsable del OKR de auditoría con Nicolás.

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
    - proyecto.contains("_OKRs Finanzas")
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

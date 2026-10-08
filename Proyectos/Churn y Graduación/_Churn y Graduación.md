---
type: proyecto
estado: activo
temas: [churn, graduacion, retencion]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
---

# Churn y Graduación

**Objetivo**: Cerrar la definición de churn graduado vs onboarding y dejarla en producción (PR376), alineada con CX.

## Contexto
- Definición aprobada (FP&A Weekly 15-sep): criterio de graduación **móvil** — 20/40/100 bookings según segmento desde el primer pago + 4º pago + 4 semanas activas. Impacto: churn graduado 321 → 510 logos.
- Correcciones de Pablo Santa Inés (CX) del 11-sep incorporadas: ver [[20260911-01 Correcciones graduación (Pablo Santa Inés)]].
- Escenarios discutidos el 1-sep: B implementado primero; C (anti-zombies) pendiente de evaluar si la diferencia justifica la complejidad.
- Metodología base: [[Metodología Churn B2B3]].
- 2026-10-08 · CX ya filtra `status='cancelled'` en el churn involuntario; Evidence queda 7 logos abajo en sep (542 vs 549) hasta el fix → [[20261008-01 Ajustes de churn en Evidence pedidos por CX (Pablo Santa Inés)]].

## Próximos pasos
- [ ] Cerrar las 4 diferencias menores vs reporte de Pablo Santa Inés.
- [ ] Evaluar Escenario C (10 bookings última semana) vs B.
- [ ] Pablo envía resumen de la definición al comité de activación.

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
    - proyecto.contains("_Churn y Graduación")
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

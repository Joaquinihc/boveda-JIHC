---
type: proyecto
estado: activo
temas: [pnl, centros-de-costo, odoo, board]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
---

# P&L y Centros de Costo

**Objetivo**: Reestructurar el P&L (visión SaaS multiproducto), dejar los CECOs de Odoo consistentes y llevar P&L por vertical al Board Book.

## Contexto
- CECOs: división+área en el CECO, país aparte; PIT-TEC (G2100) / PIT-PRO (G2000); Revenue Sales (G1100); regla de privacidad (ningún CECO de 1 persona). Retroactivo enero→hoy vía agente de Mario.
- Provisiones/ingresos diferidos: impacto P&L = delta de provisión; tokens IA se redistribuyen por vertical con el informe de tarjetas (asiento mensual de Juanjo).
- P&L SaaS: benchmarkear Shopify/Fresha; armar Notion estructura actual vs propuesta; costos de vendedores hoy castigan margen.
- Guía: [[Guía clasificación de personas (headcount)]]. Tareas Notion: Restructurar P&L, P&L verticales a Board Book, Headcount + clasificación, costos API Keys.

## Próximos pasos
- [ ] Notion con estructura actual del P&L vs propuesta + benchmark de públicos.
- [ ] Cerrar distribución de costos de plataforma entre verticales (G-1000).
- [ ] Seguimiento del P&L retroactivo con nuevos CECOs (agente de Mario).

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
    - proyecto.contains("_P&L y Centros de Costo")
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

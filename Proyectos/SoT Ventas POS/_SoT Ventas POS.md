---
type: proyecto
estado: activo
temas: [pos, payments, ventas, sot]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
---

# SoT Ventas POS

**Objetivo**: Una fuente de verdad única para ventas, activación y reactivación de POS (HubSpot como fuente primaria).

## Contexto
- Decisiones del 7-sep: dos métricas de venta en paralelo (unidades y Company ID); activación = 30 transacciones; dos reactivaciones (comercial 60 días / financiera GPV); fecha de venta = emisión de factura.
- HubSpot fuente de verdad desde octubre (agosto/septiembre en paralelo con planilla). Regla del mínimo locations×POS se mantiene.
- Documento base: [[SoT Ventas POS (documento base)]] · Queries: [[SQL — SoT Ventas POS]], [[SQL — Recalculo tableros POS]].
- Tablas nuevas de Ignacio (24-sep): [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]] · explicadas en [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]. Su reporte (todavía con la SoT anterior): [[Reporte SoT ProPay de Ignacio (board book recalculado)]].
- Análisis: [[20260909-01 POS activated per week — board book vs deck de Ignacio]] · [[20260910-01 Cuadre ventas POS ISR jul–sep 2026]] · [[20260924-01 Métricas POS del board book vs tablas SoT]] · [[20260924-02 Auditoría de casos borde de las tablas SoT de ventas POS]].
- Para repetir la auditoría cuando cambien las tablas: [[Prompt — Auditoría de casos borde de las tablas SoT de ventas POS]].
- Repos en el Mac (copias locales): `~/Documents/GitHub/agendapro-dashboards-evidence/` (board book; en su raíz, `POS_VENTAS_SOT.md/.sql`, `POS_SALES_DBT_HANDOFF.md`, `POS_RECALCULO_TABLEROS.sql`, `POS_WEEKLY_DASHBOARD.md`) y `~/Documents/GitHub/agendapro-dbt-redshift/` (copia de `agendapro/agendapro-dbt`; los modelos de Ignacio están en la rama `refactor/postgres-to-redshift`).

## Próximos pasos
- [ ] Auditar facturas emitidas pendientes de pago (criterio oficial de registro) — mío.
- [ ] Cuadrar la cifra de 104 POS entre reportes.
- [ ] Separar en reportes la reactivación comercial de la financiera.

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
    - proyecto.contains("_SoT Ventas POS")
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

---
type: tarea
estado: descartada
prioridad:
tiempo:
tipo: reporte
origen: reunion
proyecto: "[[_SoT Ventas POS]]"
temas:
  - pos
  - board
  - evidence
fuente: "[[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]"
fecha_limite:
creado: 2026-09-24
revisar: false
id: T-074
---

# Reactualizar los gráficos POS del board book con la tabla consolidada

Cambiar los gráficos POS del board book para que usen `dwh.pos_sales_sot` como única fuente de verdad. Va después de la auditoría de casos borde. Qué reporta hoy cada gráfico y cuánto cambiaría: [[20260924-01 Métricas POS del board book vs tablas SoT]].

## Subtareas
- [ ]

## Notas

### Decisiones antes de migrar (del análisis del 24-sep)
- Qué hacer con `Others` (25% de las unidades de ene–ago), sobre todo para comparar contra el budget.
- Si «New / Stock» significa canal (quién vendió) o tipo de compra (primera vez o recompra).
- Si se adopta `is_valid_purchase` (factura pagada + formulario), que cambia el criterio del 7-sep de contar toda factura emitida.
- La exclusión de Tabata y cómo se cuentan las reactivaciones (compañías vs unidades).
- «POS activated per week»: renombrarlo a locales o pasarlo a grano compañía; revisar el reinicio mensual y el filtro de 12 meses.
- El payback sigue usando `country_pos`.

## Historial
- 2026-09-24 · Creada desde [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]] (propuesta)
- 2026-09-25 · Descartada por Joaquín (Slack): es lo mismo que «Incorporar las tablas SoT de ventas POS a dbt y Evidence». Quedó como paso 3 de la subtarea 11 de [[Locations POS y reactivaciones]], con sus decisiones copiadas ahí

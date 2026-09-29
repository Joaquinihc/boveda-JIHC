---
type: tarea
estado: descartada
prioridad:
tiempo:
tipo: tabla-dwh
origen: reunion
proyecto: "[[_SoT Ventas POS]]"
temas:
  - pos
  - dwh
  - evidence
fuente: "[[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]"
fecha_limite:
creado: 2026-09-24
revisar: false
id: T-072
---

# Incorporar las tablas SoT de ventas POS a dbt y Evidence

Dejar las tablas de Ignacio como parte oficial del proyecto dbt y disponibles en Evidence. Hoy existen solo en la rama `refactor/postgres-to-redshift` del repo `agendapro/agendapro-dbt` (`models/marts/sales/`, PRs #396, #400 y #402; no están en `master`) y ningún tablero las usa. Detalle en [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]].

## Subtareas
- [ ] 1. Confirmar con Ignacio dónde quedan los modelos
- [ ] 2. Agregar las tablas como fuente en Evidence

## Notas
Cada subtarea de arriba tiene aquí su descripción con el mismo número y título.

### 1. Confirmar con Ignacio dónde quedan los modelos
- **Qué es**: los modelos están en una rama de trabajo, no en la principal.
- **Qué hacer**: acordar si se mergean a la rama principal del proyecto dbt y quién los mantiene (tests, documentación, frecuencia de actualización).
- **Resultado esperado**: tablas estables en `dwh`, con dueño y tests.

### 2. Agregar las tablas como fuente en Evidence
- **Qué es**: Evidence lee el DWH a través de las queries de `sources/dwh/` del repo `agendapro-dashboards-evidence`.
- **Qué hacer**: agregar una fuente para `dwh.pos_sales_sot` (y las otras si hacen falta) para que las páginas puedan usarla.
- **Resultado esperado**: la tabla disponible en Evidence para la tarea «Reactualizar los gráficos POS del board book con la tabla consolidada».

## Historial
- 2026-09-24 · Creada desde [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]] (propuesta)
- 2026-09-25 · Pasó a subtarea 11 de [[Locations POS y reactivaciones]] (Joaquín por Slack, aplicado en conversación)

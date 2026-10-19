---
type: tarea
estado: pendiente
prioridad:
tiempo:
tipo: reporte
origen: manual
proyecto:
temas:
  - pos
  - payments
  - evidence
fuente:
fecha_limite:
creado: 2026-10-19
revisar: false
id: T-096
---

# Reporte de utilización de terminales POS en Evidence (Payments)

Página nueva en el espacio de Payments de Evidence (`pages/fintech/`) con la utilización de terminales:
- **Denominador: sedes con POS disponible.** Es el stock mensual de terminales (comprados + arrendados) de cada comercio, topado por las sedes de su plan ese mes.
- **Numerador: sedes que transaccionan con POS.**
- **Cortes:** agregado por país, proveedor de terminal y otras categorías; no a nivel comercio.

Nace de [[20261006-02 POS vs sedes del plan — tope de unidades y POS de respaldo]]: el board book se queda en unidades y el tope por sedes vive aquí.

## Subtareas
- [ ] 1. Cerrar las decisiones de diseño (vida útil del terminal comprado, qué es una sede que usa POS, cómo se atribuye el proveedor) — listo cuando estén respondidas en Notas
- [ ] 2. Escribir el spec y el plan de implementación — listo cuando estén en el repo y aprobados
- [ ] 3. Publicar la página en Evidence — listo cuando el PR esté mergeado

## Notas

### 2026-10-08 · Lo que se sabe (brainstorming con Claude Code)
- **No duplica lo existente.** `fintech/pos-usage` mide comercios activos sobre comercios que alguna vez compraron, y `pos_penetration_merchants` mide sedes con GPV POS sobre todas las sedes pagadas. Ninguno usa como denominador las sedes con POS disponible.
- **Sedes con transacciones:** `dwh.location_sales_months` (`pos_payment_count` por sede y mes). `company_sales_months` es por comercio y no permite contar sedes.
- **Sedes del plan:** `dwh.mrr.venues` por mes (`is_active`).
- **Arriendo y cuotas:** los eventos de `dwh.int_chargebee__subscription_addon_changes` traen `old_quantity` y `new_quantity` desde 2023. El campo `addons` de `staging.stg_chargebee__subscriptions_monthly` es texto JSON y no se desanida directo.
  - Chile: `smart-pos-arriendo_2025-cl` y `smart-pos-cuotas_2023-cl`.
  - México: `terminal-getnet-renta_2023-mx`, `terminal-getnet-cuotas_2024-mx` y `terminal-oel-renta_2025-mx`.
- **Compras al contado:** líneas one-time de factura (catálogo en `int_pos_terminals_monthly.sql` de dbt). Cuotas = terminal propio: la baja de cuotas es fin de pago y el terminal sigue en el stock.
- **Proveedor y terminales usados:** `haulmer_augmented_transactions` y `oel_augmented_transactions` traen `pos_serial` y `pos_location_id`. `location_sales_months` trae el GPV por proveedor en sus columnas `gpv_<país>_pos_<proveedor>_*`.
- **Pregunta abierta: vida útil del terminal comprado.** Mi recomendación es la A.
  - A. Cuenta mientras el comercio tenga plan activo; el tope por sedes lo saca solo si se va.
  - B. Vida útil fija (por ejemplo, 36 meses).
  - C. Hasta N meses sin transacciones. No recomendada: infla la utilización por construcción.

## Historial
- 2026-10-19 · Creada desde Claude Code (conversación con Joaquín del 8-oct)

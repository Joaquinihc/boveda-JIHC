---
type: tarea
estado: propuesta
prioridad:
tiempo:
tipo: analisis
origen: reunion
proyecto:
temas:
  - pos
  - reactivaciones
fuente: "[[2026-10-05 Payback y migración a tablas POS]]"
fecha_limite:
creado: 2026-10-05
revisar: true
id: T-088
---

# Migrar el payback a las tablas nuevas de ventas POS y medir el impacto

Pasar el cálculo del payback a la tabla nueva de ventas POS (columna de compra válida) y medir cuánto cambia la cantidad de POS que considera. Acción de [[2026-10-05 Payback y migración a tablas POS]].

## Subtareas
- [ ] Verificar qué POS toma hoy el cálculo (solo primera venta o también adicionales y reactivaciones)
- [ ] Comparar la cantidad de POS del registro actual vs el nuevo y el efecto en el payback
- [ ] Decidir formalmente si reactivaciones y ventas adicionales entran al payback

## Notas
- Relacionada: [[Locations POS y reactivaciones]] (T-053, subtarea 11: el payback sigue en `country_pos`).
- Duda: la transcripción no identifica hablantes; se asume que la migración la hace Joaquín. Proyecto candidato «SoT Ventas POS» sin mención explícita, no se linkeó.

## Historial
- 2026-10-06 · Creada desde [[2026-10-05 Payback y migración a tablas POS]] (propuesta)

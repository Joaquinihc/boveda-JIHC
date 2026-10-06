---
type: reunion
fecha: 2026-10-05
titulo: "Payback y migración a tablas POS"
personas: []
proyectos: []
temas: [pos, reactivaciones]
fuente: https://app.notion.com/p/3f0ab34c233b80a08772cce6c9330306
verificado: false
revisar: true
---

# Payback y migración a tablas POS

**Acuerdos** (máx 5)
- Migrar el cálculo a la tabla nueva usando la columna de compra válida (`IsWalletPurchased` en la transcripción) en vez de la fuente actual, que no tenía id de transacción.
- El aporte de pagos al payback hoy es: revenue de pagos ÷ merchants con transacciones POS (ASP) × nuevos POS vendidos. No está claro si «nuevos POS vendidos» incluye solo primeras ventas o también adicionales y reactivaciones.
- Consenso: las reactivaciones no entran al payback ni al attachment (el payback es de merchants nuevos); las ventas adicionales tampoco, pero sí en el gráfico de barras de POS vendidos. Decisión formal pendiente: se profundiza si el impacto es grande.
- Se mencionó una caída fuerte de transacciones de JC Studio Polanco (México) y la reunión trimestral con Riverwood (actualizar presentación y archivos pedidos).

**Mis acciones**
- [ ] Migrar el payback a las tablas nuevas de ventas POS y medir el impacto (qué POS cuenta hoy, comparación con el registro nuevo)

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/3f0ab34c233b80a08772cce6c9330306). La transcripción no identifica hablantes ni asistentes.

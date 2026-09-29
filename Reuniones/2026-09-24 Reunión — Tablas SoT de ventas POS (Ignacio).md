---
type: reunion
fecha: 2026-09-24
titulo: "Reunión — Tablas SoT de ventas POS (Ignacio)"
personas: ["[[Ignacio Embry]]"]
proyectos: ["[[_SoT Ventas POS]]"]
temas: [pos, ventas, reactivaciones, activacion]
fuente: https://app.notion.com/p/3e5ab34c233b80ae8d38dcc7866473ab
verificado: false
---

# Reunión — Tablas SoT de ventas POS (Ignacio)

**Acuerdos** (máx 5)
- La tabla consolidada `POS Sales SOT` une **`POS First Sales`** (primera gestión comercial de cada comercio: primera venta o primera factura) y **`POS Resales`** (todas las ventas posteriores: reactivaciones y POS adicionales). Una venta es real si tiene factura pagada (`IsInvoicePaid`); `IsValidPurchase` combina las condiciones según el tipo de venta e `InvalidReason` dice por qué no vale (`no invoice`, `no transactions`, etc.).
- Fecha de la primera venta por prioridad **Deal > Factura > Transacción**. La cantidad la manda la factura (deal con 5 POS y factura con 1 = 1); por transacción siempre es 1. Deal y factura se asocian por ventana de ~45 días hacia adelante; un deal sin factura pagada no cuenta como venta.
- **Reactivación con máquina** exige factura pagada. **Reactivación Soft** (sin máquina) no exige factura, pero el comercio debe hacer **15 transacciones después del deal, hasta el día 10 del mes siguiente** (cierre de comisiones); si no, no vale ese mes. Quedan con `post_quantity = 0` e `IsInvoicePaid = false`.
- Atribución ISR: las facturas no sirven para saber quién vendió (registran a quien cargó el addon, p. ej. Bárbara). Se usará la propiedad de contactos de HubSpot **«contra topos»** (con timestamp del cambio), con el historial poblado vía una propiedad auxiliar «Migration». `POS Resales` no debería tener ISR; «Other» = primeras ventas históricas sin atribución. Las tablas están a nivel **company**, no location (bajar a location exige un deal por location o que ISR cambie su flujo).
- Siguiente paso inmediato: **auditar casos borde** antes de actualizar los gráficos y dejar la tabla como fuente única. Caso borde ya conocido: comercio en México que pagó una compra y la quiso cambiar a arriendo con abono del monto. Presentación con Riverwood en ~2 semanas (día 14).

**Mis acciones**
- [ ] Auditar casos borde de las tablas de ventas y validar contra lo que muestra Evidence hoy; avisar a Ignacio si alguno obliga a cambiar una query (puede afectar resultados previos)
- [ ] Incorporar las tablas al proyecto dbt y a Evidence
- [ ] Reactualizar los gráficos existentes usando la tabla consolidada como única fuente de verdad
- [ ] Aclarar con Payments el tratamiento de ventas revertidas con factura pagada (ya es subtarea de [[Locations POS y reactivaciones]])
- [x] Dar acceso al proyecto de ETE (no queda claro qué proyecto es)
- [ ] Revisar el calendario de Pablo para coordinar la presentación con Riverwood
- [ ] Agregar a Ignacio al canal de Slack de fútbol

**De otros**: Israel carga la propiedad «contra topos» en HubSpot para habilitar la atribución ISR; poblar el pasado de esa propiedad con lo que ISR declaró en la planilla y luego reemplazar esa fuente (responsable no explícito).

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/3e5ab34c233b80ae8d38dcc7866473ab). El resumen de Notion transcribe Evidence como «EBIAS» y KAM como «CAM».

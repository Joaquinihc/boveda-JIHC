---
type: reunion
fecha: 2026-11-02
titulo: "New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)"
personas: []
proyectos: []
temas: [ventas, headcount]
fuente: https://app.notion.com/p/40cab34c233b81ee95abca98924b79e6
verificado: false
revisar: true
---

# New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)

**Acuerdos** (máx 5)
- Joaquín trabaja con el Excel de Christopher (hojas «detalle pagos» y «centros»), no con el DW. El acceso a la réplica de Reservo sigue pendiente, probablemente por los permisos sobre fichas clínicas.
- Referencia de Pablo: agosto fue el récord con 173 primeros pagos (~190 contando los multisede) y el último mes fue malo, con 135. El Excel de Joaquín daba ~160.
- Método de Pablo: tabla dinámica con una fila por cliente y el mínimo de la fecha de pago como mes del primer pago; filtrando ago-2026 da 173, igual que la query sobre `pago cliente mes`. La cifra oficial puede quedar algo sobre eso por 4–5 acuerdos de revenue share con proveedores, cuyo pago entra por otra factura.
- Posibles causas de la brecha: el Excel podría no traer todas las filas con estado de pago 0 (~40 en la hoja) y el cruce no usa la variable activo ni la fecha de creación de sucursal. `cambio estado cliente` no tiene fila inicial para clientes que nunca se bloquean o pausan (de ahí la heurística de Christopher). Siguen sin cuadrar las cifras de otros meses (163, 165).
- Aparte: según Pablo, Daniela Droguet (soporte) atiende Argentina e Ignacio Monsalve México; Joaquín lo necesita para distribuir las remuneraciones por mercado.

**Mis acciones**
- [ ] Rehacer el cruce del Excel (pagos con estado 0, variable activo y fecha de creación de sucursal) hasta cuadrar los 173 de agosto
- [ ] Pedir a Christopher el detalle completo de pagos (incluidos los de estado 0) y cómo calculó los otros meses
- [ ] Distribuir por mercado las remuneraciones de Daniela Droguet (Argentina) e Ignacio Monsalve (México)

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/40cab34c233b81ee95abca98924b79e6). Título en Notion: «Pablo - Joaquín». Pablo habla como parte de Reservo («fue malo para nosotros»); no se linkea persona porque el apellido no aparece. Sigue a [[2026-11-02 Conciliación de clientes e ingresos de Reservo (Christopher)]].

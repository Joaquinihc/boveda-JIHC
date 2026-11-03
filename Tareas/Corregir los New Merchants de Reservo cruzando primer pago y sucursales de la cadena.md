---
type: tarea
estado: descartada
prioridad:
tiempo:
tipo: cuadratura
origen: reunion
proyecto:
temas:
  - ventas
fuente: "[[2026-11-02 Conciliación de clientes e ingresos de Reservo (Christopher)]]"
fecha_limite:
creado: 2026-11-02
revisar: false
id: T-115
---

# Corregir los New Merchants de Reservo cruzando primer pago y sucursales de la cadena

Los Merchants totales de Reservo de octubre ya cuadran (4.119 sucursales), pero los New Merchants no: las 215 altas por fecha de activación no calzan con la referencia de ~170–180 al mes. Criterio acordado: contar por primer pago y abrir cada pago a las sucursales de su cadena. En el cruce de Joaquín agosto da ~160 contra los 173 que reporta Reservo. Acción de [[2026-11-02 Conciliación de clientes e ingresos de Reservo (Christopher)]] y de [[2026-11-02 New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)]].

## Subtareas
- [ ] Pedir a Christopher el detalle completo de pagos (incluidas las filas con estado de pago 0) y cómo calculó los otros meses — listo cuando llegue el archivo
- [ ] Rehacer el cruce primer pago × sucursales de la cadena con la variable activo y la fecha de creación de sucursal — listo cuando agosto dé 173 o la diferencia quede explicada
- [ ] Calcular los New Merchants de octubre con ese criterio y compartirlos con Christopher y Pablo — listo cuando el número quede validado

## Notas

### Pendientes menores
- 2026-11-02 · Contrastar con Pablo (Reservo) cómo salieron las cifras de otros meses (163, 165) y qué versión de datos usó.
- 2026-11-02 · Acceso a la réplica de Reservo pendiente (permisos sobre fichas clínicas); con acceso, el cruce se puede hacer directo sobre `pago cliente mes`.
- 2026-11-02 · La cifra oficial de Reservo suma 4–5 acuerdos de revenue share al mes cuyo pago entra por otra factura: puede quedar algo sobre el conteo por primer pago.

## Historial
- 2026-11-03 · Creada desde [[2026-11-02 Conciliación de clientes e ingresos de Reservo (Christopher)]] (propuesta)
- 2026-11-03 · Mencionada de nuevo en [[2026-11-02 New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)]] (de ahí salen las subtareas 1 y 2 y los pendientes menores)
- 2026-11-03 · Pasó a subtarea de [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]] (Joaquín en conversación con Claude Code). Trabajo cerrado el 3-nov: [[20261103-01 New Merchants y Merchants de Reservo — definiciones finales y cuadre con Pablo Saintard]]

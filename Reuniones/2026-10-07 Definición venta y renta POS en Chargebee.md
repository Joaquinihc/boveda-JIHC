---
type: reunion
fecha: 2026-10-07
titulo: "Definición venta y renta POS en Chargebee"
personas: ["[[Ignacio Embry]]"]
proyectos: []
temas: [pos, definiciones]
fuente: https://app.notion.com/p/3f2ab34c233b816ba953e5fb46dc6c82
verificado: false
revisar: true
---

# Definición venta y renta POS en Chargebee

**Acuerdos** (máx 5)
- Cuatro tipos oficiales de POS: venta (one-time), renta, cuotas y regalado / costo cero (cupón 100%, se activa automáticamente). Existe además el caso de 3 meses de arriendo gratis y luego compra. Unificar los nombres entre Chile y México.
- Cupones en Chargebee: no se pueden combinar dos de porcentaje, y con uno de monto más uno de porcentaje no entran más. Hay cupones históricos activos sin depurar (p. ej. 5% Forever, 100% por 6 meses). Propuesta: cupones genéricos por monto (p. ej. «POS gratis») y limpieza de cupones y tipos de POS.
- Permisos: hoy muchos editan Chargebee. Cupones y notas de crédito se gestionan de forma centralizada: solo [[Ignacio Embry]] y otra persona designada.
- Chargebee obliga a cobrar el arriendo con la misma periodicidad del plan (anual → anual); las devoluciones antes del año se resuelven con notas de crédito manuales. Se evalúa migrar de sección en Chargebee (~20.000+ clientes); consultar a Noco.
- Casos borde (venta registrada como arriendo en México → doble conteo; arriendo que pasa a venta pierde trazabilidad): se prefiere resolverlos en la operación antes que con reglas en dbt.

**Mis acciones**
- [ ] Armar un documento con la estructura de productos POS, reglas de cupones y casos típicos (con la acción recomendada) y compartirlo con los otros tres asistentes para revisión

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/3f2ab34c233b816ba953e5fb46dc6c82). Pendientes de otros o sin responsable claro: limpieza de cupones y tipos de POS, consulta a Noco por la periodicidad y arqueo del miércoles.

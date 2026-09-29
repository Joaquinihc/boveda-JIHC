---
type: tarea
estado: descartada
prioridad:
tiempo:
tipo: cuadratura
origen: reunion
proyecto: "[[_SoT Ventas POS]]"
temas:
  - pos
  - ventas
  - dwh
fuente: "[[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]"
fecha_limite:
creado: 2026-09-24
revisar: false
id: T-071
---

# Auditar casos borde de las tablas SoT de ventas POS

Antes de que `dwh.pos_sales_sot` pase a ser la única fuente de los gráficos, revisar los casos borde de las tablas de Ignacio (`pos_ncro`, `pos_resales`, `pos_sales_sot`) y validar que calcen con lo que muestra hoy Evidence. Si algún caso obliga a cambiar una query, avisarle a Ignacio, porque puede cambiar resultados anteriores. Acordado en [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]].

## Subtareas
- [ ] 1. Revisar el informe de auditoría de casos borde
- [ ] 2. Enviar a Ignacio los casos que obligan a cambiar una query
- [ ] 3. Acordar con Ignacio las definiciones abiertas

## Notas

### Dónde está el detalle
- [[20260924-02 Auditoría de casos borde de las tablas SoT de ventas POS]]: informe de la auditoría (hecha por un subagente el 24-sep, solo lectura).
- [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]]: cómo están construidas las tablas, perfil y primeros casos borde.
- [[20260924-01 Métricas POS del board book vs tablas SoT]]: qué reporta hoy el board book y cuánto difiere.

Cada subtarea de arriba tiene aquí su descripción con el mismo número y título.

### 1. Revisar el informe de auditoría de casos borde
- **Qué es**: leer el informe y marcar con qué hallazgos estás de acuerdo.
- **Qué hacer**: revisar la tabla de hallazgos y su clasificación (cambia la query / decisión de definición / dato de origen / solo documentar). Descartar lo que no aplique.
- **Resultado esperado**: una lista corta y validada de lo que hay que arreglar.

### 2. Enviar a Ignacio los casos que obligan a cambiar una query
- **Qué es**: el informe trae un borrador de mensaje para Ignacio con los casos que cambian la query y las preguntas abiertas.
- **Qué hacer**: ajustar el borrador y enviárselo tú.
- **Resultado esperado**: Ignacio sabe qué cambiar y qué resultados anteriores se mueven.

### 3. Acordar con Ignacio las definiciones abiertas
- **Qué es**: puntos que no son errores sino criterios por decidir. Los principales que salieron hasta ahora: si manda la cantidad del deal o la de la factura; si la ventana deal↔factura es ±45 días o solo hacia adelante; si un cambio arriendo → cuotas o una baja de terminal cuenta como venta (`addon_restart`); si en `qty_up` se cuenta el aumento o el total de terminales; qué hacer con `Others`.
- **Resultado esperado**: criterios escritos y acordados, para que la tabla y el board book midan lo mismo.

## Historial
- 2026-09-24 · Creada desde [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]] (propuesta)
- 2026-09-25 · Pasó a subtarea 10 de [[Locations POS y reactivaciones]] (Joaquín por Slack, aplicado en conversación)

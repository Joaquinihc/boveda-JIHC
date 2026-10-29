---
type: tarea
estado: en-curso
prioridad: media
tiempo:
tipo: cuadratura
origen: manual
proyecto: "[[_P&L y Centros de Costo]]"
temas:
  - pnl
  - odoo
  - centros-de-costo
fuente:
fecha_limite:
creado: 2026-10-29
revisar: false
id: T-109
---

# Revisar P&L de Reservo gasto por gasto (cuentas Odoo)

Revisar línea por línea los gastos 2026 de las dos sociedades de Reservo en Odoo (SC3 SpA, Chile; SOFTWARE RESERVO, México), confirmar a qué cuenta y G-code está asociado cada uno y levantar a [[Nicolás Astudillo]] las reclasificaciones para que el P&L de Reservo se pueda sumar al P&L Master con el mismo criterio que AgendaPro. Contexto y hallazgos de la primera pasada (26-oct-2026): [[20261029-01 Revisión del P&L de Reservo en Odoo vs estructura AgendaPro]].

## Subtareas
- [ ] Cerrar con [[Pablo Lucero]] y Nicolás las tres definiciones previas: nómina 2026 de SC3 (CECO y G-code), si Reservo lleva el plan HQ/país o es unidad propia, y si Kushki/Transbank van a G0004. Listo cuando queden escritas en Notion o en esta nota.
- [ ] Armar la planilla línea por línea (911 facturas SC3 + 11 MX) con cuenta actual, cuenta propuesta, G-code Master y motivo, exportada desde Odoo con `agent_journal_entries`. Listo cuando esté compartida con Nicolás.
- [ ] Sesión con Nicolás para acordar y ejecutar las reclasificaciones (dry-run del agente en modo reclassify, luego posteo) y verificar que el P&L 2026 de SC3 mapee completo a la Master. Listo cuando los asientos estén reclasificados.

## Notas

### 2026-10-29 · Contexto de la primera revisión
- Sociedades creadas el 19-oct-2026; asientos cargados por Nicolás el 22-oct (SC3) y 23-oct (MX). Planes de cuentas clonados de AgendaPro Chile y México; facturas de compra 2026 con analítica G-code al 100 % (plan «Centros»).
- Lo que falta: remuneraciones 2026 de SC3, costos de la operación México (se pagan desde SC3), cost center HQ/país, depreciación de 28,6 MM CLP activados.
- Lo que está distinto: Kushki/Transbank en G0003/G0015; cajones G3104 (30,3 MM), G3102, G3009, G2104, G1106, G1009, G3007; Carey 21 MM en G3004 ago–sep. Detalle y montos en el análisis.

### Propuesta de método
1. **Definiciones primero** (subtarea 1): sin nómina, cost center y criterio de pasarelas no hay P&L comparable; se piden en un solo mensaje a Pablo y Nicolás.
2. **Planilla por línea** (subtarea 2): export de `account.move.line` de ambas sociedades con partner, etiqueta, cuenta, analítica y monto; columnas «cuenta propuesta», «G-code Master», «P&L Category» y «motivo». Las reglas salen del criterio del grupo (dónde lleva AgendaPro a cada proveedor) y de `docs/cost-center-classification-guide.md` del repo `mcp-odoo-agendapro`.
3. **Revisión en bloques por cuenta, de mayor a menor monto**: G3104 → G0003/G0015 → G3004 → G3009 → G3102 → G2104 → G1106/G1009/G3007. Cada bloque cierra con una decisión por proveedor, no por factura.
4. **Ejecución con Nicolás** (subtarea 3): el agente de asientos en modo reclassify cambia `account_id` sobre líneas posteadas; primero dry-run con la planilla, luego ejecución por bloque y verificación con `read_group` por cuenta.
5. **Cuadre final**: el P&L 2026 de SC3 por G-code contra la hoja Mappings de la Master; todo lo que caiga en cuentas sin G-code (6.0.00.02, 5.5.x) queda documentado.

### Pendientes menores
- 2026-10-29 · DTEmite cuesta 12 a 14 MM CLP/mes (3,6x AgendaPro Chile): preguntar a Reservo si revenden boleta electrónica y si ese ingreso va en I0000.
- 2026-10-29 · Tecnologías Saludando (0,6 MM en G2105): proveedor desconocido, preguntar qué es.
- 2026-10-29 · México: borrar la factura de prueba en borrador (N&Q Rehabilitación Oral, 26-oct, cuenta default 401.01.01) y las dos líneas en cero a «Usuario Final»; fijar la cuenta de ingreso por defecto del diario de ventas en 401.01.001.
- 2026-10-29 · Desactivar el plan de cuentas default de Odoo que quedó sin uso en ambas sociedades (310xxx/410xxx en Chile; 401.01.01, 601.84.01 y similares en México).
- 2026-10-29 · Depreciación: SC3 activó 28,6 MM CLP sin asientos de depreciación.
- 2026-10-29 · Carey 21 MM CLP en G3004 ago–sep 2026: confirmar que es costo de la transacción y marcarlo como one-off en la Master.
- 2026-10-29 · Apertura 2025: 601 MM CLP en G3008 y 1.079 MM en Sueldo Base sin analítica; si se quiere comparar 2025 habrá que abrir esos bolsones.

## Historial
- 2026-10-29 · Creada desde Claude Code (revisión de Odoo del 26-oct, repo `mcp-odoo-agendapro`, sin cambios de código).

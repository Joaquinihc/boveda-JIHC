---
type: sugerencia
origen: slack
temas: [odoo, cierre-mensual, reportes]
criterio: "(c) conclusión de análisis que explica un número"
fuente: https://agendapro.slack.com/archives/D0950RBS4DD/p1791479863750799
canal: "DM con Rafael Rodriguez"
fecha_hilo: 2026-10-08
creado: 2026-10-16
revisar: false
---

# Slack — Inconsistencias históricas del cash flow vs Odoo (ene-23 a sep-26)

- Joaquín revisó `finance.cash_flow` mes a mes desde ene-2023 cruzando con Odoo y le pasó a Rafa un listado ordenado por impacto. Lo que mueve el FCF: DAP BICE ene-24 cargado como "Financiamiento" (debería ser traspaso) y saldo PayU Perú abr-23 (~17,8k USD) que desaparece sin retiro.
- Resto: movimientos con el Santander de ProPay cargados como traspaso (cuentas fuera de la planilla), mal clasificados (Colombia, Chile, Argentina), desfases de mes en intercompany, saldos de pasarelas sin cargar y temas del lado Odoo (interés fijo CDS Chase −36,9k USD a ago-26, nómina Bice fechada al mes siguiente, tránsito sin conciliar).
- BBVA MX USD (US$2.100,84) y la cuenta en pesos del BBVA MX están bloqueadas con el saldo congelado (Rafa); Joaquín agrega una línea de saldo final.
- Septiembre está bien; es historia, sin apuro. El 13-oct Rafa respondió con comentarios en la planilla «Inconsistencias CF - Joaco (oct-2026)» (la mayoría por ProPay) y dice que Nico revisa si proponen un cambio al Cash flow de Odoo.
- 2026-10-19 · Planilla de Rafa: 18 de 27 ítems «Listo» (razón encontrada, nada modificado), 1 resuelto por documentar, 6 por corregir (3 planilla, 3 Odoo) y 2 por confirmar (PayU Perú con Pablo; garantías con Bice). El caso Global66 ago-26 (8,5 M CLP de transferencias sin netear) son recargas desde Bice para pagar proveedores, registradas en Bice como Pago a Proveedores: no afecta el FCF.
- 2026-10-19 · Postura de Rafa: son períodos cerrados, no errores sino cambios acumulados; qué corregir queda sin decidir. Detalle y tabla de estados: [[20261029-01 Inconsistencias del cash flow — respuestas de Rafa y caso Global66 ago-26]].
Planilla de Rafa (solo lectura): https://docs.google.com/spreadsheets/d/1SM24tWpXoXk_IqxtprgwTjt9aHQUETqsKKPBKYc4WYo/edit?gid=1444838607#gid=1444838607
Listado: https://agendapro.slack.com/archives/D0950RBS4DD/p1791479863750799

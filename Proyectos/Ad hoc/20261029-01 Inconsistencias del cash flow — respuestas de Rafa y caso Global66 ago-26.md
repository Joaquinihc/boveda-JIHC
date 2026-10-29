---
type: analisis
proyecto: "[[_Ad hoc]]"
estado: activo
fecha: 2026-10-29
temas: [cierre-mensual, odoo, reportes]
metricas: []
---

# Inconsistencias del cash flow — respuestas de Rafa y caso Global66 ago-26

**Pregunta**: ¿Qué explica los CLP 8.500.000 de "Tranferencia entre Cuentas" de Chile que no netean en ago-26, y en qué quedó el listado de 27 inconsistencias del cash flow (ene-23 a sep-26) enviado a Rafael Rodriguez el 8-oct-2026?

## Conclusión
- Los CLP 8,5 M son **tres abonos a Global66 (CL)** registrados como "Tranferencia entre Cuentas" (+1.500.000, +1.000.000 y +6.000.000) sin pata de salida en ninguna otra cuenta. Rafa (Slack, 19-oct): son recargas a Global66, la cuenta que usa RRHH para pagar proveedores; desde Bice él registra la salida como "Pago a Proveedores" y la entrada a Global66 queda como transferencia; la salida real a proveedores sale de Global66. **No afecta el FCF**: Suppliers queda sobreestimado en ~US$ 9,1k y la línea de transferencias en +US$ 9,1k (TC 934,58). No se corrige: período cerrado.
- Según la columna "Respuesta verificada" de la planilla, el 1,0 M es la nómina de proveedores "Notebook" del 19-ago en Bice, el 1,5 M viene del asiento manual MISC/2026/08/0021 de Odoo y los 6,0 M del 6-ago no tienen salida visible ni en la planilla ni en Odoo (puente 1.0.00.90 descuadrado en 6 M). Rafa marcó el ítem "Listo" con la explicación de la convención; los 6,0 M no se profundizaron.
- De los 27 ítems: **18 "Listo"** (= razón encontrada; Rafa no modificó nada: «la razón, no modifiqué nada»), 1 "Resuelto (documentar)", **6 "Por corregir"** (3 en la planilla, 3 en Odoo) y **2 "Por confirmar"** (PayU Perú con Pablo; garantías con Bice). Postura de Rafa: son períodos cerrados, no errores sino cambios y actualizaciones que se fueron haciendo; al revisar hacia atrás con los parámetros actuales saltan estos detalles. La decisión de si se corrige algo quedó abierta (Joaquín preguntó el 19-oct, sin cierre).
- 16-oct: Rafa confirmó que él actualiza la hoja Cash DB del P&L Master con los mismos saldos de cierre del cash flow, tomados de la cartola de Odoo.

## Método
- Fuente: CSV de la planilla de tesorería `Cash flow (Causal) - Dataset.csv` (export del 5-oct-2026; igual a `finance.cash_flow` al centavo hasta ago-26). Redshift no respondió el 19-oct, así que las filas no se verificaron en el DWH.
- Cuadre por cuenta bancaria en CLP: Saldo Inicial + movimientos = Saldo Final (misma lógica del artifact "Cuadre de Cash Flow").
- Búsqueda de contrapartidas de 1,5 M / 1,0 M / 6,0 M en todas las cuentas y países, jul–sep 2026; neto intercompany de agosto en USD.
- Planilla de Rafa «Inconsistencias CF - Joaco (oct-2026)» (https://docs.google.com/spreadsheets/d/1SM24tWpXoXk_IqxtprgwTjt9aHQUETqsKKPBKYc4WYo/edit?gid=1444838607#gid=1444838607), solo lectura: 27 filas con hallazgo, hipótesis, respuesta verificada, comentario de Rafa y estado. Hilo de Slack: https://agendapro.slack.com/archives/D0950RBS4DD/p1792442100442869.

## Evidencia
Chile ago-26, cuadre por cuenta (CLP, CSV del 5-oct):

| Cuenta | Saldo inicial | Movimientos | Saldo final | Residuo |
|---|---:|---:|---:|---:|
| Banco Bice 21-70917-4 | 28.633.986 | 18.271.339 | 46.905.325 | 0 |
| Global66 (CL) | 1.013.873 | 6.512.568 | 7.526.441 | 0 |
| Banco Estado | 8.256.298 | −7.870.026 | 386.272 | 0 |
| Working Capital Propay | 41.000.000 | 92.300.000 | 133.300.000 | 0 |

- Transferencias entre cuentas, Chile ago-26: Bice +102.003.385 (16 abonos) y Global66 −102.003.385 (16 cargos) netean exacto. Global66 tiene además +1.500.000, +1.000.000 y +6.000.000 → neto +8.500.000 CLP (US$ 9.095). Patrón del mes: cada intercompany que entra a Global66 (54,66 M; 27,14 M; 34,92 M) se barre a Bice en tramos de 7 M.
- Sin los tres abonos, Global66 cerraría en −973.559 CLP: la plata entró de verdad.
- No hay cargos de −1,5 M / −1,0 M / −6,0 M en ninguna cuenta ni país entre jul y sep-26, salvo un −1.000.000 "Pago a Proveedores" en Bice (ago). Bice tiene además un −6.014.736 de proveedores.
- Intercompany ago-26 neto: −4,6k USD (Chile +361,9k; Colombia −59,6k; México −66,9k; USA −240,0k): no es intercompany mal etiquetado.

Estado de los 27 ítems de la planilla (comentario de Rafa resumido, al 19-oct):

| # | Caso | Comentario de Rafa | Estado |
|---|---|---|---|
| 1 | Chile ene-24, DAP BICE cargado como Financiamiento | El DAP se llevaba solo como saldo en Cash DB; se pasó a cuenta bancaria para registrar los intereses | Listo |
| 2 | Perú abr-23, saldo PayU desaparece | Se retiró en abr-23 y parte fue a Argentina; la cuenta siguió con saldos hasta oct-24 y quedó en 0 desde nov-24 | Por confirmar (Pablo) |
| 3 | México mar/abr-24, +500k/+400k MXN desde Santander ProPay | SPEI con el Santander 3805 de ProPay; como el CF no maneja cuentas ProPay se cargó como transferencia; reclasificar a Movimiento ProPay cuando se agreguen | Listo |
| 4 | México jul-24, −1.067.856 MXN al Santander | Ídem #3 | Listo |
| 5 | México dic-25, 927 "traspasos" en Global66 | Reclasificar a Movimiento ProPay / Working Capital Propay cuando se agreguen los movimientos ProPay | Listo |
| 6 | Chile sep-26, boleta de garantía Transbank | Ídem #5 | Listo |
| 7 | Chile 2026, gastos del Santander 91756188 fuera de Cash out (~79 M CLP) | Ídem #5; decisión pendiente con Pablo/Nico (incorporar la cuenta o cargar gastos contra Working Capital Propay) | Listo |
| 8 | Chile ago-26, 3 abonos a Global66 sin salida (8,5 M CLP) | Recargas a Global66 para pagos a proveedores; en Bice como Pago a Proveedores; no afecta el FCF | Listo |
| 9 | Colombia sep-24, 50 M y 23,5 M COP como traspaso | Es intercompany (con Chile) | Listo |
| 10 | Colombia jun-24, −48 M como Ingreso Plan en BanColombia | Por cierre de Bancolombia, apresurado | Listo |
| 11 | Chile jun-24, −1,0 M sin destino / +1,0 M Propay | Movimiento ProPay | Listo |
| 12 | Argentina ago-24, −104.306 ARS | Transferencia a Frances Net Cash (CUIT de la pasarela que liquida cobros, no cuenta propia); reclasificar según asiento Odoo AR del 13-08-2024 | Por corregir |
| 13 | Intercompany mar/abr-23 | Desfase por fecha de fin de mes | Listo |
| 14 | Intercompany mar–jun-24 (+9k USD residual) | Desfase por fecha de fin de mes | Listo |
| 15 | Colombia may→jun-24, 500k COP | Desfase por fecha de fin de mes | Listo |
| 16 | Chile ene-23, saldo inicial Bice USD forzado | Desfase por fecha de fin de mes | Listo |
| 17 | USA ene-23, reparto SVB sin traspaso | Desfase por fecha de fin de mes | Listo |
| 18 | México jul-26, BBVA MX USD desaparece | Cuenta bloqueada, saldo congelado; Joaquín agregó la línea en agosto | Listo |
| 19 | Bice ProPay sep-23, faltan 331.812 CLP | Movimiento ProPay | Listo |
| 20 | Pasarelas sin cargar (DLOCAL CL jun-23, PayU MX feb-23) | «¿Saldo en Odoo?» | Por corregir |
| 21 | BBVA MX 1159392408, saldo fijo desde nov-25 | Cuenta bloqueada con saldo congelado | Resuelto (documentar) |
| 22 | finance.cash_flow, filas sin país | Saldo en 0 | Listo |
| 23 | Odoo, CDS Chase devengo fijo 9.881/mes (−36,9k a ago-26) | El CF se registra según saldo por mes | Por corregir (Odoo USA) |
| 24 | Odoo, nómina Bice fechada al mes siguiente | Desfase de día | Por corregir (Odoo CL) |
| 25 | PayPal Gunei 29.348,76 USD, agosto vs 1-sep | Desfase de día | Por corregir |
| 26 | Tránsito/suspenso sin conciliar al cierre de ago-26 | Odoo | Por corregir (Odoo) |
| 27 | Garantías 1.1.04.01 (19,0 M de 2024; 1,6 M intereses) | Odoo | Por confirmar (Bice) |

Decisiones pendientes:
- [[Pablo Lucero]]: PayU Perú abr-23 (#2) y si se corrige algo de los períodos cerrados (#12, #20, #25; en Odoo #23, #24, #26).
- Pablo y [[Nicolás Astudillo]]: cuenta Santander CL dentro del CF o sus gastos contra Working Capital Propay (#7); si los movimientos ProPay entran al CF (#3–#7, #11, #19).
- Bice: vigencia de las garantías de 2024 (#27).

## Caveats
- "Listo" en la planilla significa razón encontrada; Rafa no modificó la planilla ni Odoo.
- Cifras del CSV del 5-oct; no verificadas contra `finance.cash_flow` el 19-oct (Redshift no respondió).
- La planilla es de Rafa y puede cambiar; Joaquín solo tiene lectura.
- Los 6,0 M del 6-ago: la convención explica el patrón, pero su pata en Bice no aparece ni en la planilla ni en Odoo según la respuesta verificada; queda sin cerrar.

Relacionado: [[Slack — Inconsistencias históricas del cash flow vs Odoo (ene-23 a sep-26)]] · [[Cash flow — estructura nueva vs formato Causal]] · [[Revisar con Nicolás Astudillo inconsistencias entre cashflow y balance sheet]].

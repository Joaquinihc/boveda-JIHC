---
type: sugerencia
temas: [pnl, odoo]
creado: 2026-10-19
revisar: true
---

# Códigos G repetidos en la Master del P&L

> Borrador desde Claude Code (agendapro-dbt, rama `feat/ai-api-key-team-seed`, 2-oct-2026). Salió al revisar que las cuentas G nuevas de IA no chocaran con las existentes.

- **Códigos que aparecen dos veces con distinto significado:** G0202 (Finiquitos Soporte y Remuneraciones Health non-doctors Soporte), G1127 (Honorarios Beauty y Remuneraciones Sales Expansion) y G1202–G1204 (Finiquitos, Bonos y ChurnZero de Onboarding, y también Remuneraciones Beauty, Health y Medspa Onboarding).
- **"Account new" con un G-code distinto al de su fila:** G1115 → G2109, G1125 → G1124, G1126 → G1120 y G1127 (Sales Expansion) → G1124.
- **En Odoo, AgendaPro Inc tiene dos cuentas G1120:** 5.2.68 Health non-doctors y 5.2.71 Medspa.
- **Por qué importa:** pueden mandar dos gastos distintos a la misma línea del P&L. Candidato a [[Restructurar P&L]] o [[_P&L y Centros de Costo]].

## 2026-11-02 · Avance
- **Propuesta a Contabilidad:** Joaquín les propuso a [[Nicolás Astudillo]] y a Álvaro Jaramillo, en el grupo de finance, recodificar las cuentas repetidas: G1207 Beauty Onboarding, G1208 Health non-doctors Onboarding, G1209 Medspa Onboarding, G1210 Health non-doctors Soporte y G1128 Sales Expansion (G1206 ya es Payments Onboarding). Las cuentas originales no se tocan. El detalle para Odoo quedó como borrador en ese hilo de #finance: 12 cuentas (G1127 también está en México y Reservo) y 5 analíticas.
- **Master del P&L recodificada el 2-nov.** Joaquín la revisó y la dejó así, siempre que Odoo también cambie. En Mappings: filas 155 (G1128), 186–188 (G1207–G1209) y 190 (G1210). En Financials DB: 46 filas de jun–sep 2026 (11 de G0202 → G1210 y 35 de G1127 → G1128). Efecto: 17.969 USD de Sales Expansion pasan de Sales a CS Expansion, y 2.018 USD de Health non-doctors Soporte pasan de Customer Support a CS Expansion.
- **Seguimiento:** el cambio en Odoo le toca a Contabilidad. Hay que ver qué códigos dejan. Si difieren de G1207–G1210 y G1128, hay que recodificar de nuevo la Master (las 46 filas y las 5 de Mappings) y ajustar en Evidence `headcount_fin_payroll.sql` (lista `payroll_codes`) y la lista de costos del GM (board-book, OKR y MBR), que tienen G0202 y G1127 escritos a mano. → Resuelto el 3-nov: los códigos coincidieron y no hubo que recodificar; Evidence sigue pendiente.
- **Por confirmar:** la categoría de G1207–G1210 en Mappings. Quedaron como Customer Success Expansion, pero en headcount HLT-SUPP es Customer Support.

## 2026-11-03 · Verificación en Odoo
- **Rename aplicado.** [[Nicolás Astudillo]] lo hizo el 3-nov a las 12:39 UTC (usuario UsuarioContableAP), con los códigos propuestos: 12 cuentas contables (G1207 5.6.01.00, G1208 5.6.02.01, G1209 5.6.03.01 y G1210 5.6.02.00 en AgendaPro Chile SpA y SC3 SpA; G1128 5.6.08.07 en Chile y SC3 y 601.01.076 en SOFTWARE AGENDAPRO MEXICO y SOFTWARE RESERVO) y 5 analíticas del plan Centros (G1207–G1209 con nombre = código; HLT-SUPP = «G1210 - Payroll Health non-doctors Support»; REV-EXP = «G1128 - Payroll Sales Expansion»). Las originales siguen igual: G0202 Finiquitos Soporte (5.0.03.02), G1127 Honorarios Beauty (5.2.04.02) y G1204 Onboarding platforms (5.2.41, Inc). Ya no hay códigos apuntando a dos cuentas.
- **La Master cuadra** (lectura del 3-nov; Mappings y Financials DB filas 19500–23227, mar–oct 2026): G1128 en 35 filas jul–sep (Chile HQ, México y USA HQ, −17.969 USD) y G1210 en 11 filas jun–ago (Chile, −2.018 USD), con el mismo código que Odoo. G1207–G1209 no tienen filas en la Master ni líneas en Odoo. No hubo que recodificar de nuevo.
- **Cómo llega el código:** las 12 cuentas contables nuevas tienen 0 líneas. La nómina CL/MX se asienta en cuentas de concepto (Sueldo Base, Gratificación, IMSS…) con la analítica del equipo, así que el G-code del export sale de la analítica de Centros; el rename de HLT-SUPP y REV-EXP es lo que deja consistente el histórico.
- **Descuadres pendientes (de Joaquín):**
  1. Cuatro BHE 2026 en 5.2.04.02 G1127 Honorarios Beauty (Jeremías Álvarez sep y oct, Agustín Navarro práctica sep y oct; −987k CLP en total) llevan analítica REV-EXP: una re-exportación saldría como G1128 y la Master las tiene como G1127 (filas 22636–22637, sep). Decidir si se cambia la analítica en Odoo o el código en la Master.
  2. HLT-SUPP tiene −583k CLP en mayo 2026 y REV-EXP 7–13 M CLP al mes de ene a jun en Chile, pero la Master no tiene G1210 antes de junio ni G1128 antes de julio (probablemente bajo G0200 y G0105): una re-exportación de esos meses cambiaría códigos y categorías.
  3. La cuenta contable G1210 quedó «Remuneraciones Health non-doctors Soporte»; las otras cuatro y la analítica dicen «Payroll …». Cosmético.
  4. 31 filas de Financials DB 2026 con el nombre de cuenta cruzado: G1115 → «G2109 - Payroll Health non-doctors Sales» (fila 20712) y G1125 → «G1124 - Payroll Payments Expansion» (filas 21155, 21715–21724, 22504–22513 y 22974–22982). Se corrigen en Mappings, columna C.
- Siguen por confirmar la categoría de G1210 (CS Expansion en Mappings vs Customer Support en headcount) y la actualización de Evidence (`headcount_fin_payroll.sql` y las listas de costos del GM).
- Fuente: Odoo vía XML-RPC de solo lectura (`mcp-odoo-agendapro`, scripts no commiteados) y el Google Sheet «P&L AgendaPro - Master 2026». Mensaje original en #finance del 2-nov (ts 1793631996.866029). Validar con la contabilidad oficial.

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
- **Seguimiento:** el cambio en Odoo le toca a Contabilidad. Hay que ver qué códigos dejan. Si difieren de G1207–G1210 y G1128, hay que recodificar de nuevo la Master (las 46 filas y las 5 de Mappings) y ajustar en Evidence `headcount_fin_payroll.sql` (lista `payroll_codes`) y la lista de costos del GM (board-book, OKR y MBR), que tienen G0202 y G1127 escritos a mano.
- **Por confirmar:** la categoría de G1207–G1210 en Mappings. Quedaron como Customer Success Expansion, pero en headcount HLT-SUPP es Customer Support.

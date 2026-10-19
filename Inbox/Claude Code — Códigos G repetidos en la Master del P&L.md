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

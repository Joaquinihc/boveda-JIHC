---
type: borrador
origen: Claude Code
fecha: 2026-11-10
temas: [revenue, pnl, cierre-mensual]
revisar: true
---

# Claude Code — Revenue de Reservo P&L vs pagos aplanados (fila Odoo MX)

- **Qué se encontró (9-nov)**: la línea Software Revenue del P&L de Reservo (`finance.profit_and_loss`, `type = 'Reservo'`) es la misma serie que los pagos aplanados del DWH (`reservo.int_reservo_payments_flattened`): en moneda local calza al peso y en USD al dólar de enero a septiembre, porque la tasa de la Master es igual al FX current de Evidence.
- **Octubre**: sobraba la fila de Odoo MX «401.01.001 I0000 - Ingresos por Software (Ingresos Facturados Tasa 16 %)», 43.142 MXN (2.342 USD). Son facturas de SOFTWARE RESERVO a clientes mexicanos que ya pagan por la plataforma: duplicaba la fila «Recurring» de México.
- **Qué se hizo**: Joaquín borró la fila de la Master y se corrió `dbt run -s profit_and_loss --target redshift` (worktree de dbt en `refactor/postgres-to-redshift`). Octubre quedó en 235.575 USD, igual a los pagos aplanados.
- **Por qué importa**: cuando Contabilidad cargue los próximos meses de SOFTWARE RESERVO desde Odoo, esa cuenta no debe sumarse al software de Reservo en la Master. Relacionado con [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]].

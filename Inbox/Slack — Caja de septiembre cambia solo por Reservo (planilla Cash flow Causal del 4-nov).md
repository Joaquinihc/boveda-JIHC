---
type: sugerencia
origen: slack
temas: [reportes, dwh, board]
criterio: "(c) conclusión de análisis que explica un número (FBT y FCF de septiembre)"
fuente: https://agendapro.slack.com/archives/C02QC0NEGA0/p1793994884966279?thread_ts=1793026589.378459&cid=C02QC0NEGA0
canal: "#finance (hilo «Cash Flow en Odoo»)"
fecha_hilo: 2026-11-06
creado: 2026-11-13
revisar: false
---

# Slack — Caja de septiembre cambia solo por Reservo (planilla Cash flow Causal del 4-nov)

- El 4-nov la planilla «Cash flow (Causal)» (entra cada hora a `finance.cash_flow`) trajo a la vez: columna `Company` (razón social), movimientos de SC3 SpA y Software Reservo desde sep-26, y ~950 filas de saldos por banco desde 2021 (vienen de la pestaña Pasarelas, que la QUERY «Para Abacum» exporta completa).
- **Por qué cambió septiembre**: solo por Reservo. AgendaPro sigue en FBT 12.643k y FCF −292k (lo presentado). Reservo entra con 282k de saldo inicial sin existir en agosto, y el FCF lo cuenta como caja generada; su flujo propio de sep es −30k → hoy se ve FBT 12.895k y FCF −40k. Octubre (−2.468k) incluye la compra de Reservo: 3 pagos de −764,6 M CLP el 8-oct (−2.310k USD), confirmados por Rafa como tres pagos distintos.
- El bloque de bancos no mueve FBT ni FCF, pero rompe el estado de flujo (cae en «Finance»), holdings por país y tests. Rafa corrigió la razón social y moneda de los bancos de Reservo en oct-26.
- Propuesta de Joaquín: separar AgendaPro / Reservo / Consolidado con la razón social publicada en dbt (no con la regla «cuenta que nace con saldo inicial»), sacar el M&A del FCF y ojo con BICE DAP duplicado desde ene-24. Abierta: desde qué mes se consolida Reservo en caja (oct-26 como en el P&L → sep solo AgendaPro, caja adquirida 248k al 1-oct, FCF grupo oct ≈ −154k). Sin respuesta de Pablo en el hilo.

Hilo: https://agendapro.slack.com/archives/C02QC0NEGA0/p1793994884966279?thread_ts=1793026589.378459&cid=C02QC0NEGA0

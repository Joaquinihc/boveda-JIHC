---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 394
url: https://github.com/agendapro/agendapro-dbt/pull/394
autor: Joaquinihc
merge: 2026-09-22
temas: [margenes, pnl, dwh, definiciones]
creado: 2026-09-23
revisar: false
---

# PR 394 — margen bruto operacional por compañía y stream (`company_margin_*`)

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `refactor/postgres-to-redshift`.

**Proyecto**: [[_Márgenes por Cliente y Producto]] (el PR se declara para la prioridad FP&A "Análisis margen por cliente y por producto").

## Qué cambió
- Nuevos objetos en schema `finance`: `company_margin_monthly` (compañía × mes × stream Software/Payments/AI: revenue, costo por línea P&L, margen $ y %), `company_margin_summary` (último mes, T3M, T12M, piso de costo, descuento de break-even), `int_company_margin_drivers_monthly` y seed `ai_costs_manual_monthly` (costos API de IA por mes × vertical + tarifas $/compañía de plataforma).
- Revenue desde las fuentes canónicas (`mrr_saas`, `payments_revenue`, `mrr_ai`); costos directos desde `finance.profit_and_loss` (`2- Service Costs`) asignados a compañías.
- Asignación en 2 etapas: pools por G-code (AWS HQ se parte primero en sub-pool IA por vertical = tarifa × compañías IA activas; resto a Software) → `share = driver / Σ driver` con cadena de fallback. Payment processing usa costo medido (`company_sales_months`) escalado a la línea P&L.
- Clasificación G-code → stream → driver vive en la macro `company_margin_cost_pool_rules()`, a propósito no como tabla (para que no se adopte como clasificación oficial del P&L). Metodología en `docs/company_margin_methodology.md` del repo.

## Por qué
- Prioridad FP&A de margen por cliente y producto. Decisiones de alcance (Finanzas, sep-2026): se asignan Payment Processing, Platform, Customer Support y CS Retention; **CS Expansion queda excluido** (rol de venta) y solo se carga para cuadrar contra el P&L. El costo de plataforma IA se recorta de AWS con la misma tarifa que usa el reporte *P&L by vertical*.
- QA reportado: revenue cuadra exacto con las fuentes canónicas y Σ asignado + excluido = `2- Service Costs` por mes.
- Nota operativa del PR: no dar grant de estos marts a `role_finance_odoo_mcp_ro`.

## Link
https://github.com/agendapro/agendapro-dbt/pull/394

## Biblioteca que podría quedar desactualizada
- [[Guía clasificación de personas (headcount)]]: define las categorías de `2- Service Costs` (incl. CS Expansion); el PR fija que CS Expansion no entra al margen por compañía.
- [[P&L Business-Line Decomposition (Verticales - Payments - Resto → EBITDA)]] y [[AI Gross Margin - AI Margin %]]: el PR reutiliza la tarifa de carve-out de plataforma IA del P&L by vertical; conviene que referencien la nueva métrica si se oficializa.
- No existe nota de métrica para "Company Operational Gross Margin" (en Evidence quedó ⚠ PROPUESTO, ver PR 748). Crearla solo si Joaquín lo pide.

---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 748
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/748
autor: Joaquinihc
merge: 2026-09-22
temas: [margenes, evidence, definiciones]
creado: 2026-09-23
revisar: true
---

# PR 748 — tablero Company Margin (`/finance/company-margin`)

> Borrador cosechado por la tarea PRs (vie). Repo de tableros Evidence, rama `main`. Consume los modelos de los PR 394 / 395 de agendapro-dbt.

**Proyecto**: sin link (el PR no nombra el proyecto; candidato: Márgenes por Cliente y Producto). `revisar: true`.

## Qué cambió
- Nuevo reporte Finance `/finance/company-margin`: filtros (mes cerrado, FX variable/constante, país, segmento), KPIs vs mes anterior, tendencia por stream, corte por país/segmento, distribución por tramo de margen, ranking top/bottom 25 (margen 12 meses), drill-down por compañía (tarjeta de negociación: revenue T3M, piso de costo, descuento de break-even y descuento que mantiene 30 % de margen) y cuadratura contra `finance.profit_and_loss` `2- Service Costs`.
- Cinco extracciones pre-agregadas `sources/dwh/finance/company_margin_*.sql`.
- Acceso: ruta bajo `/finance/` (gated) y `/data/dwh/company_margin_` agregado a `ROUTE_PERMISSIONS` del auth-guard — **requiere deploy manual del Lambda@Edge por DevOps** para que los parquet queden protegidos.
- Gobierno de métrica: nueva entrada **Company Operational Gross Margin** en `docs/METRIC_DEFINITIONS.md`, marcada ⚠ PROPUESTO (2026-09-22) a la espera de firma de Finanzas (plucero). Decisiones registradas: CS Expansion excluido, seed de modelos IA incluido, driver 50/50 venues–bookings, carve-out de plataforma IA a la tarifa del P&L by vertical.

## Por qué
- Dar la vista de análisis del margen por compañía construido en dbt. Aprobado por nicofrem-agpro.
- Follow-ups declarados: revisión visual post-deploy; deep link `[company_id]`.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/748

## Biblioteca que podría quedar desactualizada
- No hay nota de métrica de margen por compañía; la definición está en PROPUESTO en el repo (la bóveda referencia, no reemplaza).
- [[Guía clasificación de personas (headcount)]]: categorías de `2- Service Costs`; el tablero muestra CS Expansion como excluido.

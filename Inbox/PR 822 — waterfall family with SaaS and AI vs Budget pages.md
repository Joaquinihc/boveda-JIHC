---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 822
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/822
autor: Joaquinihc
merge: 2026-10-15
temas: [waterfall, presupuesto, forecast, saas, ai, evidence]
creado: 2026-10-16
revisar: false
---

# PR 822 — familia Waterfall en Evidence: Overview + SaaS vs Budget + AI vs Budget

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/budget-vs-actual` → `main`. Revisado por nicofrem-agpro (4 rondas, todas atendidas).

**Proyecto**: sin link (el PR no nombra proyecto).

## Qué cambió
- Nueva familia `/finance/waterfall/` con pestañas **Overview · SaaS vs Budget · AI vs Budget**. `/finance/draft/waterfall` sale de draft y pasa a ser Overview (redirects legacy para las rutas viejas).
- SaaS vs Budget y AI vs Budget en tres capítulos: el mes (walk con marca de budget por barra, scorecard, tabla de brechas por país/vertical/segmento), YTD (walk + brechas) y mes a mes (walk + tabla de movimientos), con KPI cards.
- **Proyección del mes en curso** por componente y corte = acumulado a la fecha ÷ fracción del mes que se suele alcanzar a ese día (curva empírica de 6 meses; New de SaaS mantiene la curva de pace del funnel). Se descartó el método aditivo de `/finance/revenue-forecast` porque en el back-test sub-proyectaba el churn 30–60%.
- Fuentes nuevas: `bva_waterfall.sql`, `bva_budget_waterfall.sql` (SaaS real + proyección y Budget 2026 con vertical); `bva_waterfall_ai.sql`, `bva_budget_ai.sql` (AI real desde `revenue_waterfall_ai` y budget AI desde `budget_2026_revenue`, solo new/churn).
- Modelo compartido `components/bvaModel.js` + componentes de walk, cards, tablas y tabs. Dos entradas PROPUESTO en `docs/METRIC_DEFINITIONS.md`.

## Por qué
- Pedido de Julio: vista operativa del mes en curso vs Budget 2026 y vs los últimos 12 meses.
- Validación del PR: la extracción del budget calza exacto con `dwh.budget_2026_waterfall` por métrica y mes; los meses cerrados calzan con el waterfall; stock MRR + flujos cuadran a FX Budget; el stock AI (`mrr_ai`) cuadra con los flujos de `revenue_waterfall_ai`.
- Back-test (MAPE del total al día 10): SaaS churn graduado 8%, churn onboarding 7%, new 9%; AI new MRR 31%, churn MRR 46% → AI muestra banner de baja confianza hasta el día 20.

## Advertencias que deja el PR
- AI vs Budget: los cargos one-time de add-ons aún no están en el waterfall AI (se lee de suscripciones, no de facturas) → new subestimado y reactivación sobreestimada hasta que entre la regla n−1. Agente AI está en el real pero no en el budget.
- Payments vs Budget queda para fase 2: el budget no tiene componentes de waterfall para Payments; pendiente confirmación FP&A de qué incluye "Revenue POS".
- Fuera del PR: `/finance/revenue-forecast` usa el forecast aditivo que el back-test rechazó; su forecast de churn probablemente está sesgado a la baja.
- Los parquet nuevos `/data/dwh/bva_*` no quedan detrás del gate `/finance/` (el mismo budget ya se sirve sin gate vía `budget_2026`).

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/822

## Biblioteca que podría quedar desactualizada
- [[Budget 2026 (plan - presupuesto)]] — no menciona los tableros vs Budget nuevos ni las fuentes `bva_*`.
- [[Metodología Dashboard Forecast]] y [[Metodología Forecasting MTD]] — documentan otros métodos de proyección intra-mes (curva de pace / lineal); el PR introduce una curva empírica por componente para el waterfall.

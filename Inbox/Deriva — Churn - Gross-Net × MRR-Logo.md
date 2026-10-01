---
type: deriva
metrica: "[[Churn - Gross-Net × MRR-Logo]]"
estado: abierta
temas: [churn, graduacion, definiciones, waterfall]
fuente: https://github.com/agendapro/agendapro-dbt/blob/refactor/postgres-to-redshift/models/intermediate/revenue/int_subscription_mrr_breakdown_changes.sql
creado: 2026-10-01
revisar: false
---

# Deriva — Churn - Gross-Net × MRR-Logo

> Detectada por el mantenimiento mensual (1-oct-2026). La ficha [[Churn - Gross-Net × MRR-Logo]] es idéntica a la card de la skill `finance-metric-definitions`; la diferencia es contra el modelo dbt (`agendapro/agendapro-dbt`, rama `refactor/postgres-to-redshift`). No se tocó la ficha. Si corresponde, actualizar primero la skill y luego la ficha.

## Qué dice la ficha (= skill)
- Tipos de churn en `dwh.revenue_waterfall_software.change_category`: `churn_graduado` (post-activación) vs `churn_onboarding` (pre-activación, **< 3 meses de MRR activo**).
- No menciona una segunda clasificación.

## Qué dice el modelo
- `int_subscription_mrr_breakdown_changes` y `revenue_waterfall_software` (desde PR 384, 16-sep) publican **dos clasificaciones en paralelo**:
  - `change_category` / `is_graduated`: definición inicial (activación del segmento + marca de journey, `graduation_date_via_journey`). Es la serie reportada, la del presupuesto 2026 y la que leen los tableros.
  - `change_category_v2` / `is_graduated_v2`: definición nueva alineada con CX (journey O vía objetiva, `software_graduation_date`), para medirla y armar el presupuesto 2027.
- Solo difieren en el split del churn. El criterio "< 3 meses de MRR activo" no aparece en el modelo.

## Fichas que también tocan esto
- [[Churn voluntario vs involuntario]]: el predicado `change_category IN ('churn_graduado','churn_onboarding')` sigue válido; solo falta aclarar cuál de las dos clasificaciones se usa.
- [[Activation]]: ver [[Deriva — Activation]].

## Relacionado
- `Inbox/PR 384 — …`, `Inbox/PR 758 — …` y `Inbox/Slack — Graduación, medición publicada vuelve a la definición inicial y la nueva va en paralelo (v2)`.

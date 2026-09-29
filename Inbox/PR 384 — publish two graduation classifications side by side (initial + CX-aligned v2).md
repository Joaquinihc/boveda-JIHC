---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 384
url: https://github.com/agendapro/agendapro-dbt/pull/384
autor: Joaquinihc
merge: 2026-09-16
temas: [graduacion, churn, waterfall, definiciones]
creado: 2026-09-23
revisar: true
---

# PR 384 — dos clasificaciones de graduación en paralelo (inicial + v2 alineada con CX)

> Borrador cosechado por la tarea PRs (vie). Rama `refactor/postgres-to-redshift`. Sigue al PR 376.

**Proyecto**: sin link (el PR no nombra el proyecto; candidato: Churn y Graduación). `revisar: true`.

## Qué cambió
- `change_category` / `is_graduated` **vuelven a la definición inicial**: activación de segmento + marca de journey (`graduation_date_via_journey`). Es la serie que leen los tableros y sobre la que se armó el budget 2026.
- Nuevas `change_category_v2` / `is_graduated_v2` con la definición alineada con CX (`software_graduation_date`: journey O ruta objetiva del PR 376), medidas en paralelo para el próximo budget.
- Las dos solo difieren en el split de churn (`churn_graduado` vs `churn_onboarding`); 4 tests nuevos lo garantizan. Tocados: `int_subscription_mrr_breakdown_changes`, `revenue_waterfall_saas`, `revenue_waterfall_software` y sus YAML. `int_company_activation_status` sin cambios.
- Efecto reportado: "Activated EOM Merchants" del dashboard CEO vuelve a sus valores previos al 8-sep. El sufijo `_v2` es provisorio.

## Por qué
- Tras el PR 376 los waterfalls reexpresaron ene-ago 2026 con la regla nueva. Julio Guzmán pidió en #comité-activación (16-sep) no cambiar la historia reportada (cifras enviadas a inversionistas, budget con la definición inicial) y llevar la nueva como propiedad aparte.
- Follow-ups declarados en el PR: aclarar en Notion "Cambio definición Graduación compañías" cuál serie es la publicada; vistas `_v2` opcionales en Evidence.

## Link
https://github.com/agendapro/agendapro-dbt/pull/384

## Biblioteca que podría quedar desactualizada
- [[Metodología Churn B2B3]]: define `change_category` / `is_graduated` y los umbrales de graduación; no distingue definición inicial vs `_v2`.
- [[Activation]], [[Churn - Gross-Net × MRR-Logo]] y [[Churn voluntario vs involuntario]]: usan el split graduado/onboarding de `change_category`; conviene anotar que existe `change_category_v2`.

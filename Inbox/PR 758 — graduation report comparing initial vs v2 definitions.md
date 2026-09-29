---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 758
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/758
autor: Joaquinihc
merge: 2026-09-23
temas: [graduacion, churn, evidence, definiciones]
creado: 2026-09-25
revisar: true
---

# PR 758 — reporte de graduación: definición inicial vs v2

> Borrador cosechado por la tarea PRs (vie). Rama `main`. Consume lo que publica agendapro-dbt PR 384 (ver [[PR 384 — publish two graduation classifications side by side (initial + CX-aligned v2)]]); la ruta objetiva viene del PR 376.

**Proyecto**: sin link (el PR no nombra el proyecto; candidato: Churn y Graduación). `revisar: true`.

## Qué cambió
- Página nueva **`/finance/graduation`** (gated) que compara en `dwh.revenue_waterfall_software` las dos clasificaciones: **inicial** (`change_category` / `is_graduated`: activación + marca de journey; serie reportada, base del Budget 2026) y **v2** (`change_category_v2` / `is_graduated_v2`: activación + journey **o** ruta objetiva; base del Budget 2027).
- v2 contiene a la inicial: el churn solo puede pasar de onboarding a graduado, nunca al revés (verificado: cero filas).
- Secciones: KPIs resumen; % de churn graduado por definición y mix (graduado en ambas / reclasificado / onboarding en ambas); compañías que gradúan por mes (con notas de los picos 25-nov-2024, migración ChurnZero → Segments Pro sep-2025 y cierre masivo de onboarding 21/24-ago-2026); graduados por journey sin ruta objetiva (prueba la hipótesis de CX de que los journeys se cierran cuando la cuenta se va); detalle por `company_id` descargable.
- Dos extracciones nuevas en `sources/dwh/revenue/`: `graduation_churn_definitions.sql` (compañía × mes de churn desde ene-2024) y `graduation_companies.sql` (una fila por compañía graduada), en el shard `payments-revenue` (~11 s, anotado en `docs/extract-shard-timings.md`).
- Gobernanza: entrada nueva en `docs/METRIC_DEFINITIONS.md` (buckets de ruta v2 y timing de la marca como ⚠ PROPOSED) que registra la divergencia de segmento con `/general/churn-tracker/` (`first_customer_segment_2` aquí vs `first_sub_segment2` allá); paleta nueva en `AGENDAPRO_BRANDING.md`; registrada en `components/dashboards.js` y el hub de Finance.

## Por qué
- Dar visibilidad a las dos series de graduación que conviven tras el PR 384 y cuantificar la reclasificación de churn antes del Budget 2027.
- Cuadra con el registro de Notion "Cambio definición Graduación compañías" (diferencias de 1–2 merchants). El MRR es Software MRR (sin add-ons de AI), por eso queda bajo el MRR de Notion, medido sobre `revenue_waterfall_saas`.
- Pendiente declarado: la causa de la carga masiva del 25-nov-2024 no está en los datos.
- Acceso: el parquet `/data/dwh/graduation_companies/` (nombres de compañía + MRR) **no** está gated en el edge.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/758

## Biblioteca que podría quedar desactualizada
- [[Metodología Churn B2B3]]: define `change_category` / `is_graduated`; no menciona `_v2` ni este reporte como lugar de comparación.
- [[Activation]], [[Churn - Gross-Net × MRR-Logo]] y [[Churn voluntario vs involuntario]]: usan el split graduado/onboarding; conviene anotar la divergencia de segmento (`first_customer_segment_2` vs `first_sub_segment2`) entre este reporte y el churn tracker.

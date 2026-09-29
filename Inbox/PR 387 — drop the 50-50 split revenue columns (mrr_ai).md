---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 387
url: https://github.com/agendapro/agendapro-dbt/pull/387
autor: Joaquinihc
merge: 2026-09-21
temas: [ai, mrr, dwh]
creado: 2026-09-23
revisar: true
---

# PR 387 — refactor `mrr_ai`: se eliminan las columnas del split 50/50 del setup

> Borrador cosechado por la tarea PRs (vie). Rama `refactor/postgres-to-redshift`.

**Proyecto**: sin link (el PR no nombra el proyecto; candidato: Waterfall One-Time Charges, por la referencia a `docs/AI_ONE_TIME_CHARGES_WATERFALL_PLAN.md`). `revisar: true`.

## Qué cambió
- Se eliminan de `dwh.mrr_ai` `mrr_with_setup_split` y `mrr_with_setup_split_usd` (34 → 32 columnas).
- Quedan intactas: `setup_charges` (+`_usd`), `mrr_plus_setup_charges` (+`_usd`, `_usd_constant`) y `companies_with_setup_split` (nombre y semántica). Las filas M+1 del spine siguen existiendo, ahora con 0 en todas las columnas de monto, solo para sostener ese conteo.
- Rebuild contra Redshift sin diferencias en filas, meses ni sumas de las columnas que quedan.

## Por qué
- El setup one-time de IA estaba expuesto en tres variantes de reconocimiento sin que ninguna fuera oficial; el reconocimiento recurrente vs one-time se está rehaciendo bajo una metodología única y el split 50/50 ya estaba marcado como deprecable. Ninguna query de Evidence usaba esas columnas.

## Link
https://github.com/agendapro/agendapro-dbt/pull/387

## Biblioteca que podría quedar desactualizada
- [[AI Revenue (AI MRR run-rate)]] y [[ASP AI (average new-sale ticket per new AI company)]]: usan `mrr_plus_setup_charges_usd` y `mrr_usd` (no cambian); no mencionan las columnas eliminadas. Revisar solo si algún SQL guardado en otra parte las usaba.

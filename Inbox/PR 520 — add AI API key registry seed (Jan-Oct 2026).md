---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 520
url: https://github.com/agendapro/agendapro-dbt/pull/520
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-20
temas: [ai, margenes, dwh]
creado: 2026-11-20
revisar: true
---

# PR 520 — Registro de API keys de IA como seed de dbt (ene–oct 2026)

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/ai-api-key-registry` → `refactor/postgres-to-redshift`. Creado 19-nov, mergeado 20-nov. **Reemplaza la parte de registro del PR 407**, que se cerró sin mergear (ver [[PR 407 — costos IA por vertical desde registro de API keys (reemplaza seed ai_costs_manual_monthly)]], cuya nota aún dice «abierto»; no se tocó).

**Proyecto**: sin link (el PR no nombra el proyecto). Candidato: Márgenes por Cliente y Producto. Ya se cita por número (sin URL) en [[Análisis margen por cliente y por producto]] — por eso `revisar: true`.

## Qué cambió
- Lleva a la rama base el registro `finance.ai_api_key_usage_monthly` como seed: 337 filas, enero a octubre 2026. Hasta ahora se cargaba a mano desde la rama sin mergear del 407 (que solo tenía commits hasta septiembre; octubre, 31 filas, se cargó en el DWH el 2-nov sin commit).
- CSV idéntico fila a fila a la tabla del DWH en las 19 columnas; no se recargó el seed.
- 22 tests y `meta.pii` en las 19 columnas: `api_key_name` y `team` = PII (`pii_tipo: nombre`); `api_key_id` no es PII (identificador de la consola, no el secreto). Espejo `_sin_pii` generado.
- Trae la skill `.claude/skills/update-ai-api-costs/SKILL.md` (procedimiento de carga mensual, sin cambios respecto del 407).
- Sin cambios en la cadena `company_margin_*`; el seed antiguo `ai_costs_manual_monthly` se mantiene porque la cadena lo sigue leyendo.
- Carga mensual: sigue manual (`dbt build -s ai_api_key_usage_monthly --target redshift`); los flujos programados de Redshift no corren `dbt seed`.

## Por qué
- El registro ya alimentaba el reporte AI P&L by Vertical de Evidence pero vivía en una rama sin mergear; este PR lo deja en la base y permite cerrar el 407.

## Link
https://github.com/agendapro/agendapro-dbt/pull/520

## Biblioteca que podría quedar desactualizada
- [[Imputación contable de costos de IA (API keys)]] — su `fuente:` apunta a la skill en el PR #407 (cerrado sin merge); la skill ahora vive en la base vía este PR.
- [[AI Gross Margin - AI Margin %]] — si describe la fuente del costo IA como rama sin mergear o como seed manual.

---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 821
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/821
autor: Joaquinihc
merge: 2026-10-15
temas: [evidence, board]
creado: 2026-10-16
revisar: false
---

# PR 821 — extracción de Evidence: 13 shards y tope bulk de 90 min

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `perf/rebalance-extract-shards` → `main`. Aprobado por nicofrem-agpro.

**Proyecto**: sin link (el PR no nombra proyecto).

## Qué cambió
- Matriz de extracción reempaquetada en **13 shards** según el costo por query medido en prod (~34–42 min estimados por shard) y tope bulk por shard de **70 → 90 min** (`active_deadline` 5400 → 6600; `timeout_minutes` 100 → 120).
- `CACHE_VERSION` v2 → v3: el push del merge re-extrae todo con el layout nuevo.
- Arregla la extracción de `dwh.waterfall_saas_b2b3_activated` (#817): estaba al final del shard `payments-revenue` y nunca se había extraído, así que las secciones del Board Book que la leen mostraban error de query.
- `docs/extract-shard-timings.md` con el método y los chequeos post-merge; `country_pos_sot.sql` solo comentario.

## Por qué
- Desde el 5-oct, `payments-revenue`, `finance-ops` y `finance-deferred` chocaban con el tope de 70 min en 50–61% de los runs; ~26 queries por run no partían y se rellenaban con datos viejos detrás de shards en verde.
- El 13 y 15-oct un solo shard trabado bastó para bloquear la publicación (guard del 25% en `consolidate`).
- No más de 13 shards: Redshift usa auto WLM sin concurrency scaling y encola en horario laboral.
- En el peor caso la etapa de extracción toma ahora ~2 h (antes ~1 h 40).

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/821

## Biblioteca que podría quedar desactualizada
- Ninguna nota de `Biblioteca/` documenta la extracción por shards ni `waterfall_saas_b2b3_activated` (búsqueda sin coincidencias).

---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 830
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/830
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-10-19
temas: [board, churn, evidence]
creado: 2026-10-23
revisar: false
---

# PR 830 — Informasayuno: New Merchants total y churn lado a lado

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/informasayuno-nm-churn-layout` → `main`. Creado y mergeado el 19-oct; aprobado por nicofrem-agpro con un comentario en línea (la etiqueta del último mes podría seguir ocultándose en uno de los gráficos). Reemplaza al PR #829 (cerrado sin merge: su rama incluía commits locales ajenos).

**Proyecto**: sin link (el PR no nombra proyecto).

## Qué cambió
- Slide 5: gráfico nuevo **New Merchants por país, todos los segmentos**, a la izquierda del de B2B3+ (misma paleta por país, ventana de 13 meses y filtro de país). Query nueva `nm_total_country`.
- Slide 7: Churn total y Churn B2B3+ pasan a estar lado a lado en vez de apilados.
- Fix de la etiqueta «siempre mostrar el último mes» en los gráficos de churn: `labelLayout` es opción por serie; en la raíz de `echartsOptions` ECharts la ignoraba y, a media anchura, `hideOverlap` ocultaba el último mes. Se movió a `seriesOptions`. El mismo patrón en la raíz está documentado en el CLAUDE.md del repo y se usa en otras páginas (no se tocaron).
- Archivo: `pages/finance/board-book/informasayuno/index.md`.

## Por qué
- Mostrar New Merchants de todos los segmentos junto al corte B2B3+ y comparar churn total vs B2B3+ en la misma fila.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/830

## Biblioteca que podría quedar desactualizada
- [[New Merchants (nuevas altas)]] — solo si documenta qué cortes muestra el Informasayuno (ahora todos los segmentos además de B2B3+).

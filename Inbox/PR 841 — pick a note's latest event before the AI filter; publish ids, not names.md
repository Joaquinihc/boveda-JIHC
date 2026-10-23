---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 841
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/841
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-10-20
temas: [ai, pnl, evidence]
creado: 2026-10-23
revisar: true
---

# PR 841 — notas de crédito IA: último evento antes del filtro IA y parquet sin nombres

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `fix/ai-credit-notes-latest-event-deidentify` → `main`. Creado y mergeado el 20-oct; aprobado por nicofrem-agpro sin observaciones. Seguimiento de [[PR 839 — registry-based costs, live credit notes, models-only margin (AI P&L)]] (atiende los dos hallazgos de su última revisión).

**Proyecto**: sin link (el PR no nombra proyecto; candidato: P&L y Centros de Costo, igual que el PR 839). `revisar: true`.

## Qué cambió
- **Corrección**: `ai_credit_notes_detail` filtraba las líneas no IA antes de elegir el último evento de cada nota; un update que ya no traía línea IA dejaba en pie un evento anterior y un crédito obsoleto. Ahora desanida todas las líneas de cada evento `credit_note_*` (`LEFT JOIN … ON TRUE`, para ver también eventos sin líneas), elige el último evento por nota y recién ahí se queda con sus líneas IA. Grano y montos sin cambio con la data actual.
- **Seguridad**: `/data/dwh/ai_credit_notes_detail/` se sirve fuera del gate `/finance/`, así que la extracción deja de publicar nombres de comercio: queda `company_id` y montos. El hover del drawer muestra ID · número de notas · crédito.
- Archivos: `sources/dwh/finance/ai_credit_notes_detail.sql` (sin `company_name` ni join a `dwh.companies`), `pages/finance/pl-by-vertical/index.md`, tests `ai-credit-notes-detail` y `ai-pl-base`.

## Por qué
- Los dos hallazgos del agente de revisión del PR 839: correctitud de la selección de eventos y datos identificables fuera del gate.

## Advertencias que deja el PR
- Si alguna vez se necesitan los nombres, primero hay que poner el path detrás del gate (`ROUTE_PERMISSIONS` `['/data/dwh/ai_credit_notes_', 'finance']`, deploy de Lambda@Edge por DevOps).
- Tiempo de ejecución: Evidence corre la query dos veces por extracción, ~200 s cada una sobre la ventana de eventos 2026, bajo el timeout de 300 s; si crece, acotar la ventana o materializar el estado de la nota en dbt.
- Verificación declarada: 33 tests sin fallas; re-extracción en Redshift con 424 filas y 352 notas, mismos totales que antes.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/841

## Biblioteca que podría quedar desactualizada
- Ninguna nota adicional a las del PR 839 (cambia la selección interna de la extracción, no la definición).

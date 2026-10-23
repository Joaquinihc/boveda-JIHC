---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 839
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/839
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-10-20
temas: [ai, pnl, margenes, evidence]
creado: 2026-10-23
revisar: true
---

# PR 839 — P&L de IA: costo desde el registro de API keys, notas de crédito en vivo y margen solo modelos

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/ai-pl-registry-costs-live-credit-notes` → `main`. Creado y mergeado el 20-oct. Revisado por nicofrem-agpro (agente de revisión; 2 rondas con cambios pedidos, ambas atendidas; los dos hallazgos de la última ronda se arreglaron en [[PR 841 — pick a note's latest event before the AI filter; publish ids, not names]]).

**Proyecto**: sin link (el PR no nombra proyecto; candidato: P&L y Centros de Costo). `revisar: true`.

## Qué cambió
- **Costo de modelos IA desde el registro de API keys**: `finance.ai_api_key_usage_monthly` (consumo de consola por key y mes, IVA incluido) reemplaza la serie de costo tipeada a mano. Keys de servicio = AI Cost; keys de desarrollo = línea nueva **AI Dev Keys** bajo Net Result (también se resta en el P&L Consolidado, en AI Direct Contribution).
- Beauty: sus keys se re-reparten por consumo solo hasta agosto; desde septiembre cada vertical atiende a sus comercios con sus propias keys.
- **Refunds & Credit Notes en vivo**: extracción nueva `dwh.ai_credit_notes_detail` desde el log de eventos de Chargebee en `staging` (una fila por nota × línea IA; estado de la nota según su último evento, así salen las anuladas y borradas). Reemplaza el bloque `credits` tipeado a mano. Drawer en el capítulo Net result: meses × verticales, con hover que muestra las compañías detrás.
- **Margen bruto solo modelos**: filas nuevas `AI Gross Margin (models only)` y `AI Margin % (models only)` en los tres estados, y un par de gráficos costo/margen solo modelos antes del par de costo total.
- Layout: separadores de capítulo estilo Budget vs Actual; columnas de mes, divisores YTD y etiquetas se derivan de la data (sin ediciones mensuales a la página).
- Archivos: `pages/finance/pl-by-vertical/index.md`, `pages/finance/pl-by-vertical/consolidado.md`, `partials/ai-pl-base.md`, `sources/dwh/finance/ai_api_key_usage_monthly.sql` (agregado mes × equipo × uso × plataforma, sin nombres de key), `sources/dwh/finance/ai_credit_notes_detail.sql`. Tests nuevos en `scripts/test/` bajo `npm run test:sql`.

## Por qué
- Sacar del P&L de IA las dos series tipeadas a mano (costo de modelos y créditos) y leerlas de fuentes vivas.
- Validación declarada: las notas de crédito se cuadraron una a una contra `raw.chargebee__credit_note`.

## Advertencias que deja el PR
- `finance.ai_api_key_usage_monthly` ya existe en Redshift (sembrada el 14-oct desde el PR dbt #407, aún abierto; ver [[PR 407 — costos IA por vertical desde registro de API keys (reemplaza seed ai_costs_manual_monthly)]]); el PR dbt debe seguir para que la tabla quede en código versionado.
- `ai_credit_notes_detail` lee el `content` SUPER de la tabla de eventos: ~160 s en Redshift, dentro del timeout de 5 min del conector; es nueva, sin caché de respaldo, y la línea de refunds del P&L de IA depende de ella.
- Net Results se mueven levemente vs los créditos tipeados (reclasificaciones de vertical en `dwh.mrr_ai` y dos notas borradas después de congelar el bloque: ene Medspa y sep Beauty).

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/839

## Biblioteca que podría quedar desactualizada
- [[AI Gross Margin - AI Margin %]] — dice «AI Cost per the AI-P&L allocation»; el PR agrega la variante «models only» y cambia la fuente del costo de modelos al registro de API keys.
- [[Imputación contable de costos de IA (API keys)]] — documenta la imputación contable de las keys de desarrollo; el P&L de IA en Evidence ahora las muestra como línea propia (AI Dev Keys) bajo Net Result.
- [[P&L Business-Line Decomposition (Verticales - Payments - Resto → EBITDA)]] — AI Direct Contribution del Consolidado ahora resta las keys de desarrollo.

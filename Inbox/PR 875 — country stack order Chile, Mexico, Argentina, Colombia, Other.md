---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 875
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/875
autor: Joaquinihc
estado_pr: abierto
merge:
temas: [board, evidence]
creado: 2026-10-30
revisar: false
---

# PR 875 — Orden de apilado por país: Chile, México, Argentina, Colombia, Otros

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `fix/country-stack-order` → `main`. Creado el 28-oct; **abierto**, sin reviews al 30-oct.

**Proyecto**: sin link (el PR no nombra proyecto).

## Qué cambió
- Todo gráfico apilado por país apila, de abajo hacia arriba: Chile, México, Argentina, Colombia y el residual (Other / Otros LATAM). Las leyendas siguen el mismo orden.
- Regla escrita en `AGENDAPRO_BRANDING.md` (paleta Country), con los dos patrones (columnas `y={[...]}` ordenadas, o `seriesOrder` con `series=country_group`) y el aviso de que una `colorPalette` posicional se reordena junto con las series.
- Sin cambios de queries, métricas ni colores: solo orden de series (y de la paleta cuando era posicional).
- Páginas: Board Book mensual y trimestral, `budget-2026-follow-up`, `payments-revenue-monthly` (GPV por país) y `financial-performance/cash-flow-by-country`. Tests `npm run test:sql` 334/334.

## Por qué
- Mercado más grande sobre el eje y más chico arriba, con un orden único en todos los tableros.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/875

## Biblioteca que podría quedar desactualizada
- Ninguna identificada (no cambia definiciones ni números).

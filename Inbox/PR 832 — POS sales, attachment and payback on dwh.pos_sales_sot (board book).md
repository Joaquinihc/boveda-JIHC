---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 832
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/832
autor: Joaquinihc
estado_pr: abierto
merge:
temas: [pos, payments, board, evidence]
creado: 2026-10-23
revisar: true
---

# PR 832 — ventas POS, attachment y payback del board book sobre `dwh.pos_sales_sot`

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/board-book-pos-sales-sot` → `main`. **Abierto, en draft** (creado 19-oct, sin merge al 23-oct). Sin revisiones aún.

**Proyecto**: sin link (el PR no nombra el proyecto; candidato: SoT Ventas POS; el análisis 20261006-01 de esa carpeta describe esta migración). `revisar: true`.

## Qué cambió
- Mueve las superficies de Evidence que reportan **ventas POS** a `dwh.pos_sales_sot` (modelo de atribución de compras de Payments, Ignacio Embry, `agendapro-dbt` `models/marts/sales/`, v9). Una venta cuenta cuando su factura está pagada (+ formulario POS desde 2026-09-01); el deal POS de HubSpot define el canal: ISR → Inbound/New, KAM → Stock, sin deal → **Others** (categoría nueva).
- Este primer commit cubre el **board book** (mensual y trimestral): unidades, Key Metrics New POS, attachment B2B3 y el término payments del payback leen `country_pos_sot`.
  - `sources/dwh/payments/country_pos_sot.sql` reescrito sobre `dwh.pos_sales_sot` (mismas columnas de salida + `Others`).
  - Board book: serie Others en slide 8, 5.1i, 5.1j y totales; segundo gráfico del slide 8 en unidades; CTEs del payback pasan de `country_pos` a `country_pos_sot` (sin reactivaciones).
  - Docs: `docs/metrics/payments/pos-sales-sot-dbt-migration.md` (impacto) y spec `docs/superpowers/specs/2026-10-19-ventas-pos-sot-migracion-design.md`.
- Pendiente en el mismo PR: merge de la rama base (board book cambió el 14 y 15-oct), fuentes `payments_weekly/*` y `pos_attachment_segment`, páginas de Payments, seguimiento Budget 2026 y payback de adquisición, OKR / MBR / CEO dashboard, y borrar `country_pos`.

## Por qué
- Una sola fuente de ventas POS para todas las superficies, alineada con el modelo de Payments.
- Impacto declarado en el board book (corte 19-oct): unidades ene–sep 2026 Chile 417 → 409 y México 160 → 153; attachment B2B3 Chile baja hasta 9 pp; payback B2B3 de septiembre 12,5 → 12,4 meses.
- Falta la validación de negocio de Pablo Lucero.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/832

## Biblioteca que podría quedar desactualizada
- [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]] — dice que ningún tablero lee aún las tablas nuevas y que el board book sigue en `country_pos_sot.sql` con la lógica anterior; con este PR `country_pos_sot` pasa a leer `pos_sales_sot`.
- [[SQL — SoT Ventas POS]] — documenta `country_pos_sot.sql` en su versión previa.
- [[Payback mensual en Evidence — implementación y fuentes]] y [[Payback (Mensual)]] — el término payments del payback del board book cambia de fuente (sin reactivaciones).
- [[Active POS merchant - OVERLOADED (no single definition)]] — aparece la categoría de canal Others.

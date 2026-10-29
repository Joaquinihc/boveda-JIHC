---
type: tarea
estado: propuesta
prioridad: media
tiempo:
tipo: reporte
origen: manual
proyecto: "[[_SoT Ventas POS]]"
temas:
  - pos
  - evidence
fuente:
fecha_limite:
creado: 2026-10-28
revisar: false
id: T-108
---

# Migrar el resto de Evidence a pos_sales_sot

Pasar a `dwh.pos_sales_sot` las 15 extracciones de ventas POS de `agendapro-dashboards-evidence` que siguen leyendo `dwh.pos_sales2` / `dwh.pos_sales` después de los PRs #856, #860 y #832 (28-oct-2026), para que OKR, MBR, CEO dashboard, Payments Weekly y cohortes cuenten las ventas igual que el board book y el payback (venta válida, canal por deal de HubSpot, Others aparte). Pasos 4 a 6 del plan del 23-oct de [[Payback Payments]]; cierra el frente de [[Locations POS y reactivaciones]]. Un PR por grupo por el riesgo de grano (CLAUDE.md del repo).

## Subtareas
- [ ] 1. Grupo `country_pos` y attachment: `country_pos.sql` (lo leen `general/okr`, `finance/draft/MBR`, `finance/draft/ceo-dashboard`), `pos_attachment_segment.sql`, `attachment_nuevos_monthly.sql`, `r40_pos_attachment_daily.sql`, `pos_installed_base.sql`; borrar el MBR (decisión del 23-oct). Listo cuando el PR esté mergeado.
- [ ] 2. Grupo Payments Weekly (página de Miguel Cárdenas, avisarle antes): `pos_sold_current.sql`, `pos_sales_daily.sql`, `pos_sales_recent_detail.sql`, `pos_sold_acum_budget.sql`, `agent_ranking.sql`, `activation_rate_l3m.sql`, `cohort_activation_8w.sql`. Listo cuando el PR esté mergeado y `payments-sales` deje de mezclar real SoT con forecast legacy.
- [ ] 3. Grupo cohortes: `pos_activation_cohort_merchants.sql`, `pos_activation_cohort_by_person.sql`, `pos_retention_cohort_by_person.sql`. Listo cuando el PR esté mergeado.

## Notas
- Decisiones ya tomadas (23-oct, Joaquín): migrar todas las consultas de ventas POS a `pos_sales_sot` y dejar `pos_sales`/`pos_sales2`; Others no cuenta en payback ni attachment; OKR y CEO dashboard pasan a la tabla nueva aunque cambien sus cifras; el roster KAM/ISR vive en dbt (Ignacio).
- Reglas de referencia ya mergeadas: `sources/dwh/payments_weekly/pos_sold_monthly.sql` (mes × país × canal) y `sources/dwh/payments/country_pos_sot.sql` (categorías New/Stock/Others/Reactivacion). Cómo están construidas las tablas: [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]].
- Caveat para el mes en curso: la SoT llega con semanas de rezago (ver nota del 28-oct en [[Payback Payments]]); las extracciones diarias y el forecast del mes necesitan una regla propia o seguir en legacy hasta que Ignacio lo resuelva.

### Pendientes menores
- 2026-10-29 · Enfriar o borrar páginas de Fintech recién después del PR C, cuando todas lean la SoT: ver [[Apagar o enfriar reportes poco consultados en Evidence]].

## Historial
- 2026-10-28 · Creada desde Claude Code (propuesta) tras el merge de #856, #860 y #832.

---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 892
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/892
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-09
temas: [board, headcount, revenue, evidence]
creado: 2026-11-13
revisar: false
---

# PR 892 — Board Book: stock B2B3 de Reservo, revenue de Reservo por stream y headcount del grupo

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/board-book-reservo-b2b3-stock` → `main`. Creado y mergeado el 9-nov (aprobado por nicofrem-agpro, sin observaciones).

**Proyecto**: sin link (el PR no nombra proyecto). Ya citado como PR #892 en [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]] (T-111) y en [[Definir cómo clasificar a las personas de Reservo en la planilla de headcount]].

## Qué cambió
- **Stock B2B3 de Reservo en la lámina 7**, con el mismo diseño de la 6. El workbook de Reservo (corrida 9-nov) separa el stock de clientes activos por plan (B2C = Individual; B2B3 = cualquier otro). `seed-reservo.py` escribe `reservo_monthly_clients_segment.csv` y aborta si la suma no da el stock por país y mes o si aparece un segmento desconocido. El gráfico B2B3 del grupo apila Reservo desde oct-26.
- **Láminas solo AgendaPro lo dicen** en el título («— AgendaPro»); en láminas mixtas, cada gráfico solo AgendaPro lo nombra.
- **Headcount del grupo en 5.1g y 5.1g2**: `finance.headcount` trae `employer`. Las ocho extracciones que la leen (`headcount_*`, `onboarder_headcount`, `r1_owner_*`) filtran `employer = 'AgendaPro'`, así que toda página que las usa excluye al personal de Reservo (intencional: deja las cifras de AgendaPro como antes; solo salen las 52 personas de Reservo de octubre; el match de R1 owner no cambia). Extracción nueva `headcount_by_employer` (mes × employer × línea del P&L, sin nombres).
- **Revenue de Reservo por stream en 5.1 y 7b**: software (pagos aplanados), payments (Mercado Pago) y AI (Vambe); payments y AI salen del P&L de Reservo (`profit_and_loss_by_company`), la única fuente que los trae.
- Tests: 49/49 (node) y 11/11 (seed). Pendiente en el PR: confirmar que `headcount_by_employer` no quede en la lista `_backfilled` del job `consolidate`.

## Por qué
- Seguimiento de la vista de grupo AgendaPro + Reservo en el Board Book.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/892

## Biblioteca que podría quedar desactualizada
- [[finance.headcount — employer, clave de persona y carga desde la planilla]] — dice que el filtro `employer = 'AgendaPro'` está en la rama sin mergear y que «hasta que se mergee, Chile sube de 127 a 179»; ya está en `main`.
- [[Total Merchants (EOM stock)]] y [[Segment]] — stock B2B3 del grupo con Reservo (regla B2C = plan Individual).
- [[R1 por país (réplica para Finanzas)]] — las extracciones `r1_owner_*` ahora filtran `employer = 'AgendaPro'`.

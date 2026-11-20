---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 505
url: https://github.com/agendapro/agendapro-dbt/pull/505
autor: Joaquinihc
estado_pr: mergeado
merge: 2026-11-18
temas: [dwh, definiciones, ai]
creado: 2026-11-20
revisar: true
---

# PR 505 (y gemelo 506) — Override de macro nicho para 23 compañías

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. PR 505: rama `feat/macro-niche-override-batch1-rs` → `refactor/postgres-to-redshift`. Gemelo Postgres: [PR 506](https://github.com/agendapro/agendapro-dbt/pull/506) (`feat/macro-niche-override-batch1-pg` → `master`), mismo contenido. Ambos creados 17-nov y mergeados 18-nov; el 506 aprobado por nicofrem-agpro «con observaciones» tras una primera revisión sin aprobar.

**Proyecto**: sin link (el PR no nombra proyecto ni tarea). Tarea candidata por afinidad: [[Clasificación de verticales por compañía]] — por eso `revisar: true`.

## Qué cambió
- 23 filas nuevas en `models/hubspot/macro_niche_override.sql` para compañías que se venden y atienden desde una vertical distinta a la que mapea su tipo de negocio: **18 a Beauty** (11 desde Spa, 3 desde Health Non-doctors, 4 desde Others) y **5 a Health Non-doctors** (4 desde Others, 1 desde Fitness).
- Son filas de ruteo (`overrides_lexicon = false`), no correcciones del tipo de negocio.
- **El override mueve la vertical de la compañía en todas las métricas** (merchants, SaaS, Payments, AI) **y en toda su historia**, no solo en el revenue AI. Según el PR, en oct-2026 estas 23 compañías llevan además SaaS, Payments y 32 venues activos que cambian de vertical; los meses anteriores a sep-2026 se mueven menos de 100 USD por vertical (hasta nov-2025).
- Acordado con Beauty, Medspa, Health y Fitness en #vertical-business-units-ai (16–17 nov).
- Validación declarada: el modelo en Redshift da 183 filas / 183 `company_id`; Redshift y Postgres con los mismos pares; `company_attribute_macro_niche` reconstruida en temporal sin duplicados y sin mover compañías fuera del override.

## Por qué
- Alinear la vertical de reporte con la unidad de negocio que efectivamente vende y atiende a esas compañías (impacta sobre todo el revenue AI por vertical de sep–nov 2026).

## Link
https://github.com/agendapro/agendapro-dbt/pull/505 · gemelo Postgres: https://github.com/agendapro/agendapro-dbt/pull/506

## Biblioteca que podría quedar desactualizada
- [[Macro Niche]] — no menciona `macro_niche_override` ni que un override de ruteo mueve la vertical en todas las métricas y en toda la historia.
- [[Vertical (Unidad de Negocio)]] — mismo punto (la vertical de una compañía puede venir de un override y no del tipo de negocio).

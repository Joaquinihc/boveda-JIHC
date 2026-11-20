---
type: borrador-pr
repo: agendapro/agendapro-dbt
pr: 521
url: https://github.com/agendapro/agendapro-dbt/pull/521
autor: Joaquinihc
estado_pr: abierto
merge:
temas: [margenes, saas, payments, ai, dwh]
creado: 2026-11-20
revisar: true
---

# PR 521 — Margen por compañía v1: una tabla por compañía y mes

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/company-margin-v1` → `refactor/postgres-to-redshift`. **Abierto** (creado 20-nov; push `f314a0a6` el mismo día con dos ajustes de una revisión externa).

**Proyecto**: sin link (el PR no nombra el proyecto). Candidato por afinidad: Márgenes por Cliente y Producto. Ya se cita por número (sin URL) en [[Análisis margen por cliente y por producto]] y en [[20261119-01 Llaves de Software, país y ejecución — hallazgos versión 1]] — por eso `revisar: true`.

## Qué cambió
- Tabla nueva `finance.company_margin_by_company_monthly`: una fila por compañía y mes con revenue, costo asignado y margen (USD y %) para Software, Payments y AI. 18 columnas, sin totales. Será la única tabla física de la cadena de margen por compañía.
- **Costos solo desde el P&L** (Real, `2- Service Costs`). Cada cuenta va a un stream y se reparte entre compañías con una de seis llaves: venues activos, reservas (sin importadas, tope en el percentil 95 mensual por venue), revenue Software, revenue AI, costo de pagos medido y venues con transacciones POS. La única cuenta partida entre streams es AWS (G0001), de la que se separa la plataforma de agentes IA.
- Dos pasos ephemeral, sin macro ni seed: `int_company_margin_keys_monthly` (revenue, dimensiones y llaves) e `int_company_margin_cost_groups_monthly` (montos del P&L por grupo de costo, con las reglas de reparto). Tasa mensual = monto del grupo / suma de la llave en las compañías del alcance; costo de la compañía = su llave × tasa.
- Cuatro tests `canary` (severity error): unicidad compañía-mes; costos asignados vs P&L (excluye Customer Success Expansion; falla con sobreasignación > 1 USD o > 0,5% sin repartir); grupos vs Service Costs del P&L y separación AWS / plataforma IA; revenue por stream vs `mrr_saas`, `payments_revenue` y `mrr_ai`.
- Espejo `_sin_pii` generado; `vertical` marcado PII por linaje.
- **No toca la cadena actual** (`int_company_margin_drivers_monthly`, `company_margin_monthly`, `company_margin_summary`), que sigue corriendo. El tablero Company Margin de Evidence pasa a la tabla nueva en un PR aparte; la cadena antigua se elimina después.
- Decisiones anotadas en el PR: chargebacks se quedan en la llave de pagos (decisión FP&A del 20-nov, revisable); centro de costo o país del P&L que no sea `country_group` (HQ, USA, vacío o nuevo) se reparte entre todas las compañías (el push del 20-nov extiende esto a `pos_venues`); `is_closed_month` nunca es nulo; un mes cuenta como cerrado apenas aparecen Service Costs en el P&L.
- Verificación declarada: construida como tabla temporal ene-2025 → hoy (~407k filas) en 18–35 s (vs ~150 s de la cadena actual); los 4 tests en 0 filas; nada sin repartir en meses cerrados; totales por stream dentro de 1% de la simulación FP&A.

## Por qué
- Reemplazar la cadena actual de margen por compañía por una sola tabla con costos que cuadran contra el P&L y reglas de reparto explícitas por grupo de costo.

## Pendiente tras el merge (según el PR)
- La tabla se crea en la siguiente corrida programada de marts; si un test falla, la corrida queda en rojo (revertir o bajar el test a warning).
- Refrescar `scripts/pii/catalogo_fisico.json` y confirmar lectura de `role_finance_readonly`.

## Link
https://github.com/agendapro/agendapro-dbt/pull/521

## Biblioteca que podría quedar desactualizada
- Ninguna nota de `Biblioteca/` describe hoy la cadena `company_margin_*`. Si se mergea, [[AI Gross Margin - AI Margin %]] y [[Contribution Margin (CM) - Unit Profit Contribution (UPC)]] podrían necesitar una referencia a la tabla nueva (no verificado que la citen).

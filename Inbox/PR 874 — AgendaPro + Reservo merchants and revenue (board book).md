---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 874
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/874
autor: Joaquinihc
estado_pr: abierto
merge:
temas: [board, revenue, evidence]
creado: 2026-10-30
revisar: true
---

# PR 874 — Board Book: merchants y revenue de AgendaPro + Reservo

> Borrador cosechado por la tarea PRs (vie). Hechos del PR, sin juicio. Rama `feat/board-book-reservo` → `main`. Creado el 28-oct; **abierto** (aprobado por nicofrem-agpro el 28-oct, sin observaciones; aún sin merge al 30-oct).

**Proyecto**: sin link (el PR no nombra proyecto). Tarea candidata por afinidad, sin mención explícita: «Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)» (T-111) — por eso `revisar: true`.

## Qué cambió
- Board Book mensual (`pages/finance/board-book/index.md`), arriba de la slide 6: merchants del grupo = venues activos de AgendaPro + cuentas pagadoras de Reservo (apilado AgendaPro + Reservo y luego ambas compañías lado a lado), para total y para new merchants. En la slide 7b, sobre el gráfico de MRR por stream: revenue del grupo (MRR por stream + pagos de Reservo aplanados). Gráficos y queries existentes no se tocan.
- Fuente de Reservo (no está en el DWH): seed CSV agregado `sources/reservo_seed/`, regenerado con `scripts/seed-reservo.py` desde la planilla que manda el equipo de datos de Reservo (solo meses cerrados; aborta si los montos locales no cuadran con la serie USD de la planilla; nunca se commitean filas por cliente).
- Montos en moneda local, convertidos con el selector de FX de la página vía extracción nueva `sources/dwh/common/exchange_rates_monthly.sql` (FX current / constant / budget por moneda desde 2025).
- Serie de Reservo = caja reconocida linealmente (cada pago repartido en los meses que cubre, desde el mes del pago); se rotula como revenue, **nunca como MRR**. Excluye tres sectores por decisión de Reservo (4,1 % de los pagos); se pidió una corrida sin exclusiones.
- El revenue combinado no tiene comparación contra budget. Si falta dato de Reservo o el tipo de cambio, se muestra como no disponible (no cero). Hasta refrescar el seed tras el cierre, el mes por defecto del selector va un mes adelante del último de Reservo (los footers lo dicen).
- Color de marca de Reservo `#009FE3` (paleta Company nueva en `AGENDAPRO_BRANDING.md`). `docs/METRIC_DEFINITIONS.md`: sección 2026-10-28, ítems (109)–(110). Diseño y decisiones en `docs/superpowers/specs/2026-10-28-reservo-board-book-design.md`. Tests `npm run test:sql` 268/268.

## Por qué
- Llevar Reservo (empresa adquirida) al Board Book: merchants y revenue del grupo, y de cada compañía por separado.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/874

## Biblioteca que podría quedar desactualizada
- [[Total Merchants (EOM stock)]] — el titular del board book pasa a tener una versión de grupo con cuentas pagadoras de Reservo (si se mergea).
- [[New Merchants (nuevas altas)]] — new merchants de grupo y de Reservo en la slide 6.
- [[Definiciones KPIs del Board (DB KPIs)]] — no documenta la vista de grupo AgendaPro + Reservo ni el revenue de Reservo (caja lineal, no MRR).
- [[FX constant]] — nueva extracción `dwh.exchange_rates_monthly` (current / constant / budget) usada por el board book.

---
type: analisis
proyecto: "[[_SoT Ventas POS]]"
estado: activo
fecha: 2026-09-09
temas: [pos, activacion]
metricas: []
---

# POS activated per week — board book vs deck de Ignacio

**Pregunta**: ¿Por qué el gráfico "POS activated per week — last 24 weeks" da distinto en el board book y en el deck de Ignacio, si tienen el mismo título?

Tarea: [[Locations POS y reactivaciones]] (T-053).

## Conclusión
> Mismo título, dos métricas distintas. Los efectos se compensan y por eso los totales se parecen.

## Hallazgo: "POS activated per week — last 24 weeks", board book vs deck de Ignacio (9-sep-2026)

Mismo título, dos métricas distintas. Es el mismo tema de fondo de esta tarea (locations vs compañías, y qué venta cuenta), visto desde la activación en vez de la venta.

**Dónde vive cada uno**

| Qué | Board book (mío) | Deck de Ignacio |
|---|---|---|
| Reporte | https://evidence.agendaprops.com/finance/board-book — slide **2.4**, "Payments Activation — Deep dive", gráfico al pie | Artifact "Board Book · Payments recalculado con SoT ProPay" — slide **5** · https://claude.ai/code/artifact/201054f6-2379-4028-87f3-e2329d1f8e0a |
| Página / código | `pages/finance/board-book/index.md` (repo `agendapro-dashboards-evidence`): slide ~L1183, chart ~L1221 | HTML standalone generado desde el DWH; la lógica del gráfico está en la función JS `renderWact()` y los datos en `M.weekly_activated` (4 variantes: `in`, `ex`, `in_x`, `ex_x`) |
| Query del dashboard | bloque `payd_weekly_activations` (~L5424, DuckDB) | Sin SQL publicado en el artifact. Base: `POS_VENTAS_SOT.sql` (raíz del repo de Ignacio) |
| Query de extracción | `sources/dwh/payments_cohorts/pos_activation_cohort_merchants.sql` → `dwh.pos_activation_cohort_merchants` | SoT = Chargebee ∪ planilla KAM (`dwh.pos_v2`) + `dwh.location_sales_days` / `company_sales_months` |
| Columna del eje X | `first_activation_date_30` → `DATE_TRUNC('week')` (lunes ISO) | Día en que la empresa cruza 30 trx acumuladas → semana, también lunes |
| Refrescar parquet | `npm run sources -- --queries "dwh.pos_activation_cohort_merchants"` | — |
| Otros consumidores de la misma tabla | `pages/fintech/pos-activation-retention/index.md`, `pages/finance/board-book/quarterly/index.md` | — |

Tablas del DWH detrás del board book: `dwh.pos_sales` (factura → `first_sale_date`, `cohort_month`, país por moneda), `dwh.location_sales_days` (trx diarias por local). El cálculo del día de activación está en los CTEs `location_days_relative` (define `rel_month` y `qualifying_day`, ~L360–382), `qualifying_days_running` y `location_activation_day` (~L485–507).

**En qué se diferencian**

| Dimensión | Board book | Ignacio (SoT) |
|---|---|---|
| Universo de ventas | Facturas Chargebee (`pos_sales`), fecha = factura | Chargebee ∪ planilla KAM, fecha en cascada addon → factura → planilla |
| Población | Toda fila de cohorte, incluidas recompras y reactivaciones (ya transaccionan y "activan solas") | Solo **nuevas puras** (1er POS de la empresa y 0 trx previas). Toggle suma reactivaciones frías (≥60 d sin trx) con el reloj reiniciado |
| Grano | **Local**. Qué local activó se infiere rankeando por velocidad y cortando en las unidades vendidas | **Empresa**. Toggle "POS" pondera por máquinas (cota superior, declarada) |
| Qué trx cuentan | Solo días *qualifying*: ticket promedio del día ≥ $10.000 CLP / ≥ $250 MXN | Trx brutas. Toggle para descartar el día de prueba del técnico (ticket ≤ $150 CLP / ≤ $3 MXN) |
| Acumulación | Suma corrida **dentro de cada mes relativo**, se reinicia cada mes (20 trx en M1 + 20 en M2 nunca activa) | Acumulado **desde la venta**, sin reinicio (ese caso activa en M2) |
| Filtro de cohorte | Solo cohortes de los últimos 12 meses vs el mes seleccionado (`cohort_month >= selected_month − 11 MONTH`) | Sin filtro |
| Ancla temporal | 24 semanas hasta el cierre del **mes del dropdown** | 24 semanas hasta **hoy** (última barra = semana en curso, parcial y sin marca) |
| País | Moneda de la factura | `companies.country_name` |

**Efecto en números** — mismas 24 semanas (30-mar a 07-sep-2026), CL+MX, parquet local del 9-sep:

| Serie | Activaciones |
|---|---|
| Ignacio, nuevas puras, con trx de prueba | 160 comercios / 174 POS |
| Ignacio, + reactivaciones frías | 167 comercios / 181 POS |
| Board book, septiembre seleccionado | 150 locales |
| Board book sin el filtro de cohorte de 12 meses | 187 locales |

Los efectos se compensan y por eso los totales se parecen. El filtro de 12 meses le saca 37 activaciones al board (ventas de 2025 que activan tarde), y el criterio qualifying + reinicio mensual retrasa cruces. En contra, el board incluye recompras/reactivaciones y el grano local puede contar dos veces una empresa con dos sucursales.

**Ojo con el panel "Evidence hoy" del deck**: es una captura del board con **julio** seleccionado (semanas 09-feb a 27-jul, total 132). No es comparable barra a barra con su serie SoT que llega al 07-sep.

**Para conversar con Ignacio**

- Grano local vs empresa: su aviso es correcto, "POS activados" del board es una heurística de reparto porque ninguna fuente ata máquina ↔ local. Mismo punto que la subtarea de compañías/locations/units.
- El reinicio de la suma corrida por mes relativo es discutible en el board: hace que el día de activación dependa del mes en que cae el cruce.
- El filtro de cohortes de 12 meses no tiene sentido en un gráfico de flujo semanal: oculta activaciones tardías reales.
- En su deck: marcar la semana parcial y anclar "Evidence hoy" al mismo corte que la SoT.

Cómo reproducir los números del board en local: `node scripts/ddb-model.cjs` con la query `payd_weekly_activations` reemplazando `${inputs.selected_month}` por `'2026-09-01'`.

---
type: metodologia
temas: [revenue, board, evidence, definiciones, reportes]
metricas: ["[[Total Merchants (EOM stock)]]", "[[New Merchants (nuevas altas)]]"]
fuente: agendapro-dashboards-evidence · sources/reservo_seed/README.md y docs/superpowers/specs/2026-10-28-reservo-board-book-design.md (lectura del 2026-11-03)
fecha: 2026-11
---

# Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación

Cómo entran los datos de Reservo (empresa adquirida en octubre de 2026) al Board Book y al Flash Revenue mientras no exista conexión al DW de Reservo. Sin cifras: las cifras van en los análisis que las usan, p. ej. [[20261103-01 New Merchants y Merchants de Reservo — definiciones finales y cuadre con Pablo Saintard]].

## Dónde vive
- Reservo **no está en el DWH ni en dbt**. La fuente es el libro `pagos_mensuales_aplanados*.xlsx` que corre Christopher Lagos (Datos, Reservo) y manda por Slack; cada corrida es un snapshot distinto (ventana, hojas y flags pueden cambiar).
- Repo `agendapro-dashboards-evidence`: fuente CSV `sources/reservo_seed/` (README con la ficha de cada archivo), script `scripts/seed-reservo.py`, tests `scripts/test/test_seed_reservo.py` (`npm run test:seed`) y los tests de página `scripts/test/board-book-reservo.test.mjs` y `scripts/test/flash-revenue-reservo.test.mjs` (`npm run test:sql`). Diseño y decisiones fechadas: `docs/superpowers/specs/2026-10-28-reservo-board-book-design.md`; definiciones en `docs/METRIC_DEFINITIONS.md` (ítems 110–111).
- Páginas: `pages/finance/board-book/index.md` (slides 5.1, 6 y 7b: merchants, new merchants, revenue y ARR del grupo; 7b pro forma de Reservo por país) y `pages/general/flash-revenue/index.md` (ARR, merchants, new merchants y MRR del grupo con selector de país, bridge FX con Reservo como cuarto stream).
- Análisis de consistencia del método de pagos aplanados en Notion: «Reservo: pagos aplanados vs. nuestro reconocimiento de revenue» (407ab34c233b80c7acd1ec6da9bab455).

## Archivos de la semilla
| CSV | Grano | Qué trae |
|---|---|---|
| `reservo_monthly_revenue.csv` | mes × moneda | revenue aplanado y bruto en moneda local, `month_status` (closed / partial) |
| `reservo_monthly_revenue_country.csv` | mes × país × moneda | el mismo revenue atribuido al país del pagador (cuadra con la serie por moneda) |
| `reservo_monthly_clients.csv` | mes | merchants, nuevos y la apertura por `cobertura_pago` (pago propio, de la cadena, cuenta nueva sin pago, solo actividad) y `leaving` |
| `reservo_monthly_clients_country.csv` | mes × país | lo mismo por país |
| `reservo_run.csv` | una fila | fecha de corrida, ventana, último mes cerrado, `data_through`, nota de FX |

Nunca se commitean filas por cliente; el script aborta si los montos locales no cuadran con la hoja `aportes_por_moneda` o si el conteo de activos no calza con `conteo_clientes_activos`.

## Definiciones vigentes (3-nov-2026)
- **Merchants** = clientes activos vigentes de Reservo: hoja `clientes_activos` (no demo, `cuenta_lista = 1`, activada antes del cierre, sin baja vigente, actividad en 12 meses o menos de 90 días de vida) con `activo_vigente = True` (saca las que se están dando de baja, bloqueadas o apagadas sin pago). Es cuenta, no sucursal ni cadena. Incluye «cuenta nueva sin pago» porque Reservo las cuenta (decisión abierta).
- **New merchants** = cuentas cuyo primer pago pagado cae en el mes: mínimo de `fecha_pago` por `cliente_id` con `estado_pago = 1`, solo filas aplanadas (con `n_meses_cobertura`); una fila sin fecha ni cobertura no cuenta. Es la regla de Reservo (tabla dinámica de Pablo Saintard). Descartadas: activaciones del mes y primer pago por cadena.
- **Revenue** = caja reconocida linealmente: cada pago se reparte 1/n en los meses que cubre, empezando en el mes del pago (no en el día: queda ~1 % anual bajo nuestra regla diaria). Es revenue, **nunca MRR** (no hay historia de planes). `monto` ya viene con cupones y descuentos. IVA: Chile neto; Argentina y Colombia monto final sin IVA; México +16 % solo con factura (verificado: el xlsx los trae netos). Falta el revenue share de Vambe (factura aparte).
- **FX**: montos en moneda local convertidos en la página con `dwh.exchange_rates_monthly` (`sources/dwh/common/`), en las tres bases del selector (current, constant, budget 2026). No usar los USD del xlsx (spot del día de la corrida para toda la historia).
- **Mes en curso**: la corrida incluye el mes de la fecha de corrida como `partial`; las tablas MoM/YoY usan el último mes cerrado.

## Consolidación (decisión del 3-nov-2026)
- Los gráficos de grupo suman Reservo **solo desde octubre de 2026** (mes de la adquisición): bloque `reservo_consolidation` (`DATE '2026-10-01'`) y vistas `_cons` en ambas páginas; línea punteada «Reservo consolidated» en los gráficos del grupo.
- La historia de Reservo se muestra pro forma en gráficos propios (revenue por país en 7b; merchants, nuevos y revenue por país en el Flash).
- Tablas de crecimiento con dos lecturas: *reported* (like-for-like, AgendaPro contra AgendaPro) y *consolidated* (total del grupo contra lo reportado el período anterior; octubre muestra el salto). Sin comparación contra budget para el total del grupo; el Flash agrega una fila «Total AgendaPro Budget Comparison» antes de la del grupo.

## Cómo regenerar con una corrida nueva
1. Guardar el xlsx fuera del repo y correr `python3 scripts/seed-reservo.py /ruta/pagos_mensuales_aplanados.xlsx` (necesita `openpyxl`; CI nunca lo ejecuta, solo lee los CSV).
2. Revisar `reservo_run.csv`: ventana, último mes cerrado, `excluded_sectors` (debe ser `none`; la corrida del 20-oct excluía tres rubros) y que no cambien las hojas o flags (si el script aborta, cambió la definición y hay que mirar la hoja antes de tocar el código).
3. `npm run test:seed` y `npm run test:sql`; luego `npm run sources -- --queries "reservo_seed.reservo_monthly_revenue,reservo_seed.reservo_monthly_clients,…"` para refrescar el parquet local.
4. Documentar la corrida en el README de la semilla y en la spec (addendum fechado).

## Revenue share de Vambe y comisiones de Mercado Pago (5-nov-2026)
Dos flujos de revenue de Reservo que no están en los pagos de suscripción y entran aparte en la planilla de conciliación «DB - MRR & fees - DWH - Conciliación tabla» (hoja Reservo, una fila por mes). Criterio común: la columna de moneda local es la moneda en la que Reservo **recibe** el dinero, no el país del comercio.
- **Vambe (columna S, CLP)**: Reservo factura a Vambe SpA desde SC3 SpA el 25 % de la suma de los planes Vambe de los clientes de la alianza activos al día 1 del mes siguiente (foto, no promedio). La factura va en CLP al dólar del día, el 25 % es el total con IVA incluido y el neto es ÷ 1,19. Se reconoce por el «Mes al que corresponde la factura». Fuentes: hoja `facturas` de la planilla «Clientes Reservo Vambe» (Pablo Saintard) para el monto, y las consultas mensuales «Query Vambe - Reservo (Mes)» de Sebastián Cruzat (carpeta Drive, solo lectura) para el detalle por cliente; un archivo «Activa al 01-MM» alimenta la factura de MM−1 y la suma de filas activas manda sobre el encabezado «MRR Activo». La hoja `clientes` es una foto actual sin historia y no sirve para la serie. El share no está en `detalle_pagos`: cargarlo aparte no duplica. Una sola factura en CLP aunque haya clientes de otros países.
- **Mercado Pago (columna M, CLP)**: solo Chile; fee por transacción y venta de bono Fonasa, 100 % de Reservo, acumulado en la cuenta de Mercado Pago de SC3 SpA sin retiros ni factura. Mientras no haya reporte de liquidaciones, se aproxima por la variación del **saldo total** (incluye «a liberar») entre cartas de saldo de cierre de mes. Es caja, no devengo. Abierto si es bruto o neto de IVA.
- Cifras, cuadre mes a mes y pendientes: [[20261105-01 Revenue share de Vambe y comisiones de Mercado Pago en el cierre de octubre de Reservo]].

## Camino a fuente automática (pendiente)
Christopher carga en el DW de AWS de Reservo las tablas de pagos (con `id_cadena` y `estado_pago`), el maestro de cuentas y un snapshot diario del estado de cada cliente; Israel conecta ese DW al DWH de AgendaPro; después la semilla y el script se reemplazan por una extracción en `sources/dwh/`. El acceso a la réplica de Reservo está trabado por los permisos sobre fichas clínicas.

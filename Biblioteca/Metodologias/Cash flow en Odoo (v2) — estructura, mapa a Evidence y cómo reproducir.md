---
type: metodologia
temas: [odoo, evidence, definiciones, reportes, cierre-mensual]
fuente: agendapro/finance-odoo-mcp (PRs #1006, #1010 y #1023; YAML cash_flow_management.yaml y payments_position.yaml) · agendapro-dashboards-evidence sources/dwh/finance/ · lectura del 2026-11-19
actualizado: 2026-11-19
---

# Cash flow en Odoo (v2) — estructura, mapa a Evidence y cómo reproducir

Cómo está armado el reporte «Cash flow management» de Odoo desde el 17-nov-2026 (versión 2, de [[Pablo Lucero]]), cómo se traduce cada línea al cash flow de Evidence y cómo sacar las dos fotos para compararlas. Sin cifras: están en [[20261119-01 Cash flow Odoo v2 vs Evidence — comparación ene–oct 2026 y auditoría]]. La estructura del lado de Evidence está en [[Cash flow — estructura nueva vs formato Causal]].

## Cómo está armado en Odoo
- **Dos reportes**: «Cash flow management» (`account.report` 47) y «Payments position» (48).
- **Dos cajas**, definidas por la etiqueta de la cuenta: operacional (etiqueta 205, `CFM: Cash & Banks`) y Fintech (etiqueta 220, `CFM: Fintech Cash`). La plata de comercios de ProPay vive en la caja Fintech y no entra al Free cash flow.
- **Clasificación**: cada movimiento de caja va a la línea que indica la etiqueta `CFM:` de la cuenta contrapartida. Una cuenta debe tener una sola etiqueta.
- **Free cash flow = Cash from Operations**, con Capex dentro. Los pagos de facturas de POS y de activo fijo se rastrean hasta la factura y salen de Suppliers hacia POS y Capex.
- **Investing & Financing** es una sola sección: intercompany, M&A, financing repayment, Loan, ESOP, Working capital – Propay y Other.
- **Tapones**: Other (caja operacional sin clasificar) y Other (Fintech). Deben dar 0 por sociedad y mes.
- **Cuenta suspense de bancos**: los abonos sin conciliar suman como Software (7 cuentas, por id) y los cargos van a Other. Un banco sin conciliar infla Software.
- **Fecha**: la contable de la cartola. Una nómina pagada el último día puede caer al mes siguiente.

## Mapa de líneas: planilla → Evidence → Odoo

| Cuenta de la planilla | Línea en Evidence | Línea en Odoo | Qué cambia |
|---|---|---|---|
| Ingreso Plan AgendaPro / Reservo | Software revenue | Software | Planilla: cuando la liquidación llega al banco. Odoo: cuando entra a la pasarela, si el saldo de esa pasarela es caja |
| Ingreso por Propay | Propay margin | Payments margin | En Odoo son tres hijas: traspasos y tarjetas desde Fintech, comisiones de partners POS y gastos pagados desde Fintech |
| Remuneraciones | Payroll | Payroll | Fecha contable de la cartola |
| Pago Previsional, Finiquitos, Impuestos, Comisiones Bancarias | Payroll taxes, Severance, Taxes, Bank fees | Igual | — |
| Pago a Proveedores | Suppliers | Suppliers + POS + Capex | Comparar la suma de las tres |
| POS Purchase | POS Purchase (Cash from Finance) | POS (Cash out) | Cambia de sección |
| Intereses Ganados | Financial income (Cash from Finance) | Financial income (Cash in) | En Odoo entra al Free cash flow |
| Movimiento Intercompany | Intercompany flows | Intercompany flows | El fondeo a la cuenta Propay va a Payments (Fintech) |
| Pago de Financiamiento, Financiamiento, Financiamiento capital, Inversiones SC3 / Reservo | Financing Repayment, Loan, ESOP, M&A | Igual, en Investing & Financing | — |
| Tranferencia entre Cuentas | Transfer between bank accounts | No aparece | Traspaso dentro de la caja; si una punta es Fintech, cae en Payments margin |
| Compra/Venta de divisas | Currency exchange | Exchange rate effect | — |
| (variación del saldo de pasarelas) | Payment gateways | Payment gateways | Evidence: todas las pasarelas. Odoo: solo las que no son caja |
| (variación de fondos e inversiones) | Línea con el nombre del fondo | No aparece | Son caja en ambos lados |
| — | — | Working capital – Propay, Other, Payments (Fintech) | Sin equivalente en la planilla |

El mapa completo, cuenta por cuenta, está en `datos/mapa_categorias.csv` de la carpeta de análisis.

## Mapa país → sociedad
La planilla lleva país; Odoo lleva sociedad.

| En la planilla | Sociedad en Odoo |
|---|---|
| Chile | AgendaPro Chile SpA |
| cuentas con sufijo `(CL.R)` | SC3 SpA |
| Mexico | SOFTWARE AGENDAPRO MEXICO |
| cuentas con sufijo `(MX.R)` | SOFTWARE RESERVO |
| Argentina | AGENDA PRO ARGENTINA S.A. |
| Colombia | AGENDAPRO COLOMBIA SAS |
| USA | AgendaPro Inc |

Excepción: dLocal USD (XB) está en Chile en la planilla y en AgendaPro Inc en Odoo.

## Cómo sacar las dos fotos
**Odoo**
- La copia local de `mcp-odoo-agendapro` está cientos de commits atrás: usar un worktree de `origin/main`, con el `.venv` de la copia principal y un enlace a su `.env`.
- Export:

```bash
python -m agents.fpa.agent_cash_flow_export.orchestrator \
  --desde 2026-01 --hasta 2026-10 --formato excel --comparar-odoo todos
```

- Tarda unos 35 minutos para 7 sociedades y 10 meses. La hoja del xlsx se llama «1 - Datos».
- Columnas: `Country, Company, Section, Group, Line, Order, Type (Flujo|Saldo), Account, Account Name, Partner, Amount Local Currency, Currency, Date, Subsection`.

**Evidence**
- `/usr/bin/python3` tiene `redshift_connector`, `duckdb` y `pandas` (sin `pyarrow`: guardar en CSV).
- Leer `finance.cash_flow` con el target `redshift` de `~/.dbt/profiles.yml`.
- Aplicar las consultas de `sources/dwh/finance/` (`cash_flow_detail`, `cash_flow_monthly`, `cash_holdings`, `intercompany_legs`) de `origin/main`.
- El bloque `fp_cash_flow_table` del Board Book corre tal cual en DuckDB reemplazando `${cash_ref_month}`.

Scripts de la sesión del 19-nov (no versionados): `~/Documents/GitHub/_analisis/2026-11-cash-flow-odoo-v2/scripts/` (`comun.py` tiene los cargadores y los mapas de líneas).

## Trampas
- **Permiso**: `finance_dbt_user` (target `redshift-finance`) no puede leer `finance.cash_flow`; el target `redshift` sí.
- **Monedas mezcladas**: Chile, Argentina, México y SC3 tienen cuentas en USD. Llevarlas a moneda local con la tasa implícita del mes antes de sumar; si no, aparecen diferencias falsas en divisas.
- **Tasas de AgendaPro Inc**: Odoo no tiene tasas de cambio de esa sociedad antes del 9-jul-2026. El consolidado en USD de Odoo no sirve para meses anteriores; convertir con la tasa de la planilla.
- **Pantalla de México**: el cuadre contra la pantalla puede dar timeout (marzo-2026 no respondió).
- **Sociedades sin fecha de bloqueo** (SC3 y SOFTWARE RESERVO): sus meses cerrados pueden seguir cambiando.
- **Filas de saldo repetidas** en `finance.cash_flow`: la extracción de Evidence las filtra; una lectura directa de la tabla las duplica.
- **Reservo en la planilla**: aparece desde sep-2026 y Evidence lo consolida desde oct-2026; en Odoo están todos los meses.
- **`read_group` por XML-RPC** falla con `groupby` vacío: agrupar por `account_id`.

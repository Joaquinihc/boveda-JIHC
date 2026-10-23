---
type: metodologia
temas: [cierre-mensual, definiciones, reportes, board]
actualizado: 2026-10-23
---

# Cash flow — estructura nueva vs formato Causal

Dos formas de presentar el mismo cash flow mensual en USD. La **nueva** es la de Evidence (board book y Financial Performance, desde el PR #767, ver [[PR 767 — anchor opening balance to prior closing and add FX rate effect line]]) y la del archivo `2. Cash Flow Statement.xlsx` enviado a Riverwood en Q3'26. El **formato Causal** es el del export histórico de Causal. Ambas parten de la planilla de tesorería `Cash flow (Causal).xlsx` → `finance.cash_flow`, donde cada mes se convierte a su propio TC de cierre (ver [[Slack — Cash Flow (Causal) pasa a tipo de cambio de cierre mensual]]).

## Qué cambia

| | Formato Causal | Estructura nueva |
|---|---|---|
| Initial balance | Saldo bancario de apertura al TC del mes nuevo + pasarelas e inversiones del mes anterior. No calza con el Final anterior. | **Final balance del mes anterior.** Encadena siempre. |
| Efecto de TC | Cuenta de tesorería "Efecto tipo de cambio en Saldo Inicial" dentro de Cash from Finance (fila "Exchange rate"). Como el Initial ya viene reconvertido, el efecto se cuenta dos veces y Initial + Ops + Finance ≠ Final. | Esa cuenta sale de Finance. Fila propia **FX rate effect** = Final − Initial − Ops − Finance. En Finance queda **Currency exchange** (solo compra/venta real de divisas). |
| Equity financing | Dentro de "Investment" | Línea propia (en Evidence se llama "ESOP") |
| Free cash flow | ΔFinal − Loan − Investment (equity) | ΔFinal − Loan − Equity financing. Mismo criterio: Causal ya excluía el equity (ago-24, 40,5k) y el préstamo (jul-25, US$15M). |
| Vista trimestral | Sin fila de TC | Fila *Effect of exchange rate changes* y fila de FCF; BoP = EoP anterior |

No cambian: Final balance total, Cash from Operations, Financial income, Financing Repayment ni Loan.

Ejemplo sep-26 (DWH del 7-oct): Causal 12.917.103 − 286.527 − 5.079 = 12.625.497 ≠ Final 12.641.388 (faltan justo los −15.892 contados dos veces). Nueva 12.932.994 − 286.527 + 10.812 − 15.892 = 12.641.388. Con la cuenta BBVA MX USD restaurada (8-oct) el Final queda en 12.643.489.

## Qué mide el FX rate effect
- Es un **residuo**, no un registro. Como tesorería convierte movimientos y saldo final al TC de cierre del mes, el residuo es la **revalorización de los saldos bancarios de apertura** (saldo local × variación del TC), más cualquier descuadre de datos. No mide efecto de TC sobre los flujos del mes.
- Si todo está bien registrado, el residuo iguala la cuenta de efecto de TC de tesorería. Sep-26: −15.892 en ambos (al dólar).
- **Limitación**: solo reconvierte bancos. El TC sobre inversiones y pasarelas queda dentro de Other (Finance) o de Payment gateways (Operations). Ene-24: −29,4k de TC real del depósito BICE DAP quedan en Other; el FX rate effect muestra solo −2.325 de bancos.

## Descuadres conocidos (residuo ≠ cuenta de tesorería)
- **Jul-26 (−2.101)**: la cuenta **BBVA MX USD** (US$2.100,84 fijos desde may-2025) desapareció de la planilla en julio sin movimiento. Tesorería la volvió a cargar el 8-oct: el residuo de julio pasó de +509 a +2.610 (= cuenta de tesorería). La cuenta está bloqueada con saldo congelado, ver [[Slack — Inconsistencias históricas del cash flow vs Odoo (ene-23 a sep-26)]].
- **Ago-26 (+33,50)**: no es redondeo de TC (diagnóstico corregido el 20-oct). La hoja `Efecto tipo de cambio` de la planilla de tesorería (*Cash Flow 2025*) toma el saldo inicial de cada banco con un SUMIFS sobre la hoja `Auxiliar`, y el QUERY de `Auxiliar` no incluye la pestaña `Shinkansen (MX)` (sí está en `Para Abacum`). Por eso la cuenta abierta en jul-26 (31.518,41 MXN) quedó con efecto de TC 0 en agosto, cuando le correspondían +570,59 MXN = +US$33,50. Arreglo: agregar `'Shinkansen (MX)'!A:F` al QUERY de `Auxiliar`. Corregido por Rafael Rodríguez y verificado en `finance.cash_flow` el 22-oct: cuenta de ago-26 = +1.531,56 vs residuo +1.531,53.
- **Ene-23**: primer mes de la serie; la apertura se toma como Saldo Inicial − cuenta de efecto de TC.

## Trampas de la planilla de tesorería
- **Amount USD sin fórmula en las últimas filas** de la hoja Dataset: esas filas llegan al DWH con 0 USD. El 8-oct, la cuenta de efecto de TC de sep-26 estaba en −10.215 en el DWH y en −15.892 en la planilla (6 filas: Global66 CL −301, Working Capital Propay −5.332, BBVA FI CO −44). Recalcular siempre USD = monto local ÷ TC de la hoja Exchange rates. Sep-26 resuelto: Joaquín extendió la fórmula el 20-oct; el 21-oct `finance.cash_flow` traía la cuenta en −15.891,56 contra un residuo de −15.891,53.
- **Rescate del DAP BICE de ene-24** (522,6 MM CLP, US$561,5k) registrado como "Financiamiento": la estructura nueva lo trata como préstamo y el FCF de ene-24 da −676,7k en vez de −115,2k (Q1'24 −0,855M vs −0,293M). En el archivo de Riverwood se reclasificó a Other; en Evidence sigue como préstamo hasta que tesorería lo corrija.

## Por qué la caja de cierre difiere entre formatos
Encadenando trimestres desde Q1'24, el formato Causal llega a **12,581M** en Q3'26 y la estructura nueva a **12,641M** (datos del 7-oct; 12,643M con la cuenta BBVA MX USD restaurada). Los +60,2k:

| Causa | Impacto |
|---|---:|
| BoP de partida: 0,976 (apertura de ene-24 ya reconvertida al TC de enero, y la misma pérdida de TC −31.950 otra vez en los flujos de Q1'24) vs 1,008 (cierre real dic-23) | +32,0k |
| Financial income de abr-26 reexpresado por tesorería después del export de Causal (+31,2k) | +26,6k |
| Financial income de Q1'24 reexpresado (+5,0k) | +4,4k |
| Reclasificaciones menores Q3'25–Q4'25 | +2,1k |
| Diferencias menores en Finance Q1'26 | −2,8k |
| BBVA MX USD (jul) y Shinkansen sin efecto de TC (ago) | −2,1k |
| **Total** | **+60,2k** |

En los flujos la convención casi no mueve números: la diferencia viene del BoP de partida y de reexpresiones de tesorería.

## 2026-10-22 · ¿Mostrar la cuenta de tesorería como FX rate effect?
Pregunta de Joaquín: presentar el efecto de TC como la cuenta de reevaluación de tesorería en vez del residuo Final − Initial − Ops − Finance. No se aplicó; quedó como subtarea de [[Analizar el efecto tipo de cambio en el cashflow con Claudio]].
- **Miden lo mismo.** La cuenta es exactamente Saldo Inicial (al TC nuevo) − Saldo Final anterior (al TC viejo), banco por banco. Tras los arreglos de sep-26 (fórmula de Amount USD) y ago-26 (Shinkansen), cuenta y residuo difieren a lo más **US$0,10 por mes** en feb–sep 2026 (`finance.cash_flow`, 22-oct):

  | Mes | Residuo | Cuenta | Dif. |
  |---|---:|---:|---:|
  | Jul-26 | 2.610,18 | 2.610,16 | +0,02 |
  | Ago-26 | 1.531,53 | 1.531,56 | −0,03 |
  | Sep-26 | −15.891,53 | −15.891,56 | +0,03 |

- **Por qué quedan centavos**: el modelo dbt `models/intermediate/finance/cash_flow.sql` (agendapro-dbt-redshift) castea `amount` y `amount_usd` a `decimal(18,2)`, y el Dataset muestra ambas columnas con formato `#,##0.00`; cada fila se redondea por separado. Recalculado a precisión completa desde la planilla (8-oct), la diferencia es 0 salvo centavos históricos: Banco Estado (CL) nov-23 +0,23 y dic-23 −0,13 (efecto de TC en 0 con saldo de ~US$8), y descuadres internos de pestañas bancarias (saldo final ≠ inicial + movimientos): Galicia AR ene-24 −12,98 ARS, Brex feb-24 +0,02, BBVA FI MX jul-24 +0,12 MXN, FI CHASE abr/may-26 ±0,005.
- **Qué cambiaría en el reporte**: solo la fila FX rate effect. Initial, Ops, Finance, Final y FCF quedan iguales. Sin descuadres no hace falta una fila de diferencia.
- **A favor**: la línea pasa a ser un registro auditable y se puede abrir por país (sep-26: CLP −7.524,72, MXN −6.720,48, COP −910,58, ARS −735,78).
- **En contra**: depende de que tesorería cargue bien la cuenta (los dos descuadres de 2026 eran de la planilla, no de TC). Las listas de cuentas de `Para Abacum` y `Auxiliar` se mantienen a mano por separado, y una cuenta nueva puede volver a quedar fuera.
- **Si se aplica**: cambiar la fila en board book 5.4 (`pages/finance/board-book/index.md`), board book trimestral y `pages/finance/financial-performance/cash-flow.md` (ya separan la cuenta como `'FX account'`), ajustar el texto "foots exactly" y agregar un control que avise si |residuo − cuenta| > US$1 en algún mes.

## Cómo reproducir
- Evidence: `cash_flow_detail` / `fp_cash_flow_table` (board book 5.4). Archivo mensual armado con scripts locales (sesión del 7–8 oct, no versionados en el repo); la convención está en la hoja Notes de `2. Cash Flow Statement - nueva estructura.xlsx` (carpeta `~/Documents/Análisis/RW-Quarterly-Requests/`; la versión de `To send/` va sin Notes).
- Uso en el reporte trimestral: [[Reporte trimestral Riverwood (PortCo, P&L y Cash Flow)]]. Decisión de FCF y caja Q3: [[Slack — Cash Flow nuevo formato, FCF excluye Loan y Equity y caja Q3 cuadra a 12,64M]].

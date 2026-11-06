---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-11-05
temas: [pnl, odoo, cierre-mensual]
metricas: []
---

# Diferencia de cambio y cashback en el P&L Master 2026 (F1003, F1004 y G4004)

**Pregunta**: ¿Dónde va la diferencia de cambio en el P&L de gestión? ¿Qué era en realidad G4004? ¿Qué hay que corregir en el Master 2026 y en Odoo?

## Conclusión
> - **La diferencia de cambio va siempre debajo del EBITDA, en Financial expenses** (decisión de [[Pablo Lucero]], 5-nov-2026). Va en dos códigos: **F1003** para la intercompañía (existe desde el 2-nov) y **F1004** para la de terceros (código nuevo, creado el 5-nov). Las comisiones bancarias, los cashbacks y otros quedan dentro del EBITDA.
> - **G4004 vuelve a ser «Cashback tarjetas corporativas»**, dentro del EBITDA (Other Corporate). Es lo que el Master trae desde 2021: cashback de Jeeves, Tribal, SVB y Brex, más algunos cargos de tarjeta. G4005 «Cashback» nunca tuvo filas.
>   - El error venía de la fila 68 de Mappings, que decía «G4004 - Diferencias de Cambio» en la columna B y «G4005 - Cashback» en la C. El 3-feb, [[Nicolás Astudillo]] corrigió la C a «G4004 - Exchange rate».
> - **Odoo quedó alineado el 5-nov** (Nico; verificado el mismo día). **El Master también, el mismo día**, por etapas que Joaquín revisó: 18 filas de diferencia de cambio modificadas y 76 agregadas, sin eliminar filas existentes. El plan del 4-nov decía 15, 9 y 54; se rehízo para no eliminar filas y dejar una fila por cuenta y mes, como el export. F1003 + F1004 cuadran con Odoo por país y mes. Documento para el equipo: [Ajustes P&L cierre octubre](https://app.notion.com/p/agendapro/Ajustes-P-L-cierre-octubre-40fab34c233b803f8850e2432b10ee9c) (Notion).
> - **Efecto enero–octubre 2026** (USD mil; estimado sobre el Master (6) del 3-nov y confirmado en la planilla en vivo al aplicarlo el 5-nov):
>   - EBITDA: −16,4 (de −2.013,7 a −2.030,1).
>   - Financial expenses: −146,9. Entran −152,8 de diferencia de cambio, de los que −146,9 son intercompañía, y salen +5,9 de las filas manuales F0001 de Argentina.
>   - Resultado neto: −163,3.
> - **Con el IMSS y la nómina de México** ([[20261105-02 IMSS y nómina de México en el P&L Master 2026]]), el efecto total del 5-nov es (USD mil): EBITDA +33,2, Financial expenses −146,9, Taxes −6,9 y Net income −120,6. El Brex no mueve el EBITDA, porque G4011 y G4004 están los dos en Other Corporate.

## Método
- **Odoo**, vía XML-RPC (solo lectura), del 3 al 5-nov:
  - cuentas y analíticas con G4004, F1003 y F1004 en las 7 sociedades;
  - líneas de 2026 por cuenta, analítica y mes;
  - cuentas de diferencia de cambio automática de cada sociedad (`res.company`);
  - rubros de consolidación.
- **Master:** «P&L AgendaPro - Master 2026 (6).xlsx» (3-nov) para la historia y el efecto, y la planilla en vivo (5-nov) para ubicar las filas.
- **Slack:**
  - hilo de #finance del 2 al 5-nov, con Pablo y Nico;
  - DM de Rafael Rodríguez del 5-nov a las 12:55.

## Evidencia

### G4004 en el Master
| Año | Filas | Contenido |
|---|---:|---|
| 2021 | 1 | Jeeves Cashback |
| 2022–2024 | 41 / 51 / 47 | Cashback de Jeeves, Tribal, SVB y Brex |
| 2025 | 46 | Cashback de Jeeves y Brex, más Jeeves fee y Clara (cargo por pago atrasado) |
| 2026 | 34 | 20 de cashback (19 vacías de plantilla + Brex de febrero, 2.508,78 USD) y 14 de diferencia de cambio, que entraron por el export de Odoo desde mayo |

En el DWH, G4004 por año es 15.439 USD en 2022, 11.266 en 2023, 7.323 en 2024 y 9.942 en 2025 (`finance.profit_and_loss`, que copia la categoría del Master). Pasar G4004 debajo del EBITDA habría cambiado el EBITDA de esos años.

### Cambios en Odoo (Nico, 5-nov; verificados el mismo día)
- **Analíticas:**
  - Se creó la analítica F1004 (id 365) y la G4004 (id 245) se renombró a «G4004 - Cashback tarjetas corporativas».
  - 56 líneas de 2026 pasaron de G4004 a F1004.
  - 13 líneas de la 5.3.0.00.09 de Argentina (decimales) y las 2 de venta y baja de activos de Argentina pasaron a G4011.
  - Ninguna línea de 2026 queda con G4004.
- **25 cuentas renombradas:**
  - Terceros, como «F1004 - Diferencia de Cambio» en CL, SC3, AR, MX, Reservo, CO e Inc (en Inc, «F1004 - Exchange Difference»).
  - SC3 6.0.01.01 a 6.0.01.04, con F1003.
  - Cashback, en G4004: la 5.2.01.06 (CL y SC3) y la 5.2.33 (Inc).
- **México:**
  - Se crearon las cuentas F1003 701.01.003 y 702.01.002.
  - Se reclasificaron ahí 24 revaluaciones con Inc y Gunei de abril a octubre: 529.197,43 MXN de pérdida y 986.385,81 de utilidad. El neto, +457.188,38, es igual al F1003 de México del plan.
  - Octubre quedó cerrado al 31-10.
- **Argentina:**
  - La 5.3.0.00.01 y la 5.3.0.00.02 son ahora las cuentas de diferencia de cambio de la sociedad.
  - La .02 quedó con tipo «Otros ingresos». El export calcula crédito − débito, así que el signo del Master no cambia.
  - La 5.3.0.00.10 ya tenía el rubro «Gains (losses) on exchange differences».
- **Brex:**
  - La línea de octubre (x5497/2026/00009, 7.736,04 USD) es reward (cash redemption).
  - El canje del 4-feb-2026 (2.508,78 USD) está en Odoo, pero sin efecto en resultados: el UPLOA/2026/02/0072 (3-feb, «Brex») tiene las dos líneas en la 5.2.35 G2106, una al debe y otra al haber. El P&L solo lo trae por la fila manual del Master (G4004, febrero). Esa fila se mantiene y se saca solo si Contabilidad registra el canje en Odoo como cashback (Joaquín, 5-nov).

### Argentina (Rafael Rodríguez, 5-nov)
Odoo tiene el efecto completo: sueldos en USD, valuación de las cuentas en USD y compra y venta de dólares. La revaluación intercompañía va en F1003. El Master solo tomaba la compra y venta y la valuación. En los tres casos se usa lo de Odoo:
- **Enero a abril:** la reconstrucción de sueldos (OP/2026/01/0004, 02/0002, 03/0009 y 04/0012) reemplaza las filas manuales F0001 «Cambio USD AR» (−5.869 USD en el Master contra −5.877 en Odoo).
- **Mayo:** +1.112.470,70 ARS en la 5.3.0.00.02, en vez de los +31,2 MM manuales, y −530.468,94 en la 5.3.0.00.01.
- **Agosto:** −706.949,06 ARS en la 5.3.0.00.01, en vez del −1,5 MM estimado.

Después de los cambios, el Master cuadra con Odoo de enero a octubre: −20.419.528,66 ARS en la 5.3.0.00.01 y +12.234.682,20 en la 5.3.0.00.02.

### Cambios en el Master (aplicados el 5-nov)
- **Lo aplicado** (planilla en vivo, 5-nov):
  - **Diferencia de cambio:** 18 filas modificadas y 76 agregadas, sin eliminar filas.
    - Argentina: 13 modificadas y 16 agregadas.
    - Chile: 1 y 21.
    - Colombia: 3 y 24.
    - México: 1 y 15.
  - **Bienes de uso de Argentina:** 4 filas pasan a G4011, y se agrega una de 856 ARS (decimales de agosto).
  - **Brex:** el de mayo y el de octubre pasan de G4011 a G4004.
  - **Reservo Chile (SC3):** 18 filas F1004 de enero a septiembre, con Type «Reservo». Suman −2.683.914 CLP (−2.959 USD) y afectan solo P&L (Reservo).
  - **Bitácora por etapa:** `Bitácora cambios P&L Master 2026-11-05.md`, en la carpeta de Octubre.
  - **Respaldo previo:** copia del Master en Drive, del 5-nov.
- **Plan original** (referencia):
- **Mappings:**
  - Fila 68 (G4004): pasa a «G4004 - Cashback tarjetas corporativas» / «G4004 - Corporate card cashback» y sigue en Other Corporate.
  - Fila 167 (F1003): pasa a Financial expenses.
  - Fila 168: nueva F1004, en Financial expenses.
- **Financials DB 2026** (números de fila de la planilla en vivo, 5-nov):
  - 15 filas modificadas: 9 de diferencia de cambio pasan a F1004, con los montos de Odoo; 4 de Argentina pasan a G4011; y en la 6.1.1 de Inc, el Brex de mayo y el de octubre pasan a G4004.
  - 9 filas eliminadas: 3 de Colombia, que son intercompañía cargadas como G4004, y 6 F0001 de Argentina.
  - 54 filas agregadas: 35 F1003, 18 F1004 y 1 G4004.
- **Hallazgo aparte:** la baja de activo de Argentina de abril (5.3.0.00.07) cambió en Odoo el 6-oct. El Master tenía −303.270,30 y Odoo, −222.398,22. Se corrigió en el Master el 5-nov.
- **Detalle fila por fila:** `/Users/joaquinherreracabrejos/Documents/Cierre de mes/P&L/Octubre/`, archivos `Correcciones diferencia de cambio 2026.md` y `Ajustes diferencia de cambio 2026 v2.xlsx`. Usan la numeración del Master (6); en la planilla en vivo las filas están corridas.

## Caveats
- El efecto se calculó sobre el Master (6) del 3-nov. La planilla en vivo tiene filas nuevas (Reservo y Colombia de octubre), así que la base del EBITDA puede diferir. El delta de −16,4 mil USD no depende de eso: al aplicarlo en la planilla en vivo dio lo mismo.
- **Pendientes de Odoo que no mueven el Master:**
  - la 6.0.00.02 de Chile y la 641000 de Inc tienen el rubro de consolidación «Operating Expenses», no el de diferencia de cambio;
  - la 5.3.0.00.10 no tiene la etiqueta «CFM: Exchange Rate»;
  - en México, el costo de cambio de Global 66 de enero y marzo quedó en F1004 (−681,53 MXN).
- En el repo `mcp-odoo-agendapro`, el PR #908 de Nico (abierto) alinea Rule 57, el pre-cierre y las guías de cierre con F1003 y F1004.

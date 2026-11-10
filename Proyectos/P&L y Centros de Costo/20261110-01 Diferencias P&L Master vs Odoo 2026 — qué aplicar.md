---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-11-10
temas: [pnl, odoo, cierre-mensual]
metricas: []
---

# Diferencias P&L Master vs Odoo 2026 — qué aplicar

**Pregunta**: De las diferencias entre el P&L Master y Odoo que quedaron en la página de Notion [Diferencias P&L Master vs Odoo 2026 (para revisión)](https://app.notion.com/p/agendapro/Diferencias-P-L-Master-vs-Odoo-2026-para-revisi-n-410ab34c233b8061aa23c6d9cb16b902), ¿cuáles conviene aplicar en el Master y cuáles no?

## Conclusión
> **Recomendación de Claude, sin aplicar** (9-nov). Joaquín todavía no aprobó la lista de cambios.
> - **Aplicar ahora:** A2 a A11, con A8 corregido, más los gastos comunes de Edificio Leones de B3. Efecto en USD: **EBITDA −55.172 y Net income −95.269**.
> - **A1 (Google Ads de México) no se agrega.** El gasto ya está completo en el Master, cargado en USA con centro de costo México. Odoo lo tiene dos veces, en Inc y en México, así que agregarlo sumaría unos 32,7 mil USD de gasto duplicado. Lo que sí se puede hacer es moverlo de país, cuando Contabilidad reverse la parte de México en Inc.
> - **A8 (ICA y 4×1000 de Colombia), con un ajuste a la página.** El 4×1000 de mayo a agosto ya está en el Master en G4003, porque así se llamaba la cuenta 53051801 antes del 3-sep. Corresponde reclasificar esas filas a G4013 y agregar solo lo que de verdad falta. El efecto en Taxes es −6.094, no los −7.588 que dice la página.
> - **B4 (Chargebee USA) no se aplica, porque el error está en Odoo.** El devengo mensual nunca se reversa y, además, las facturas se registran como gasto. Desde junio el gasto está dos veces.
> - **B1 (Transbank Chile) y B5 (nómina USA) probablemente sí, pero hay que confirmarlos con [[Nicolás Astudillo]].** B2 (nómina Chile) y B6 (México de enero a mayo) necesitan una conciliación línea por línea antes de tocarlos.
> - **Reservo:** la fila 24913 de México de octubre está duplicada y va en 0. El AI Revenue de octubre en Chile es igual al de septiembre, así que hay que preguntarle a Christopher Lagos. El capítulo C queda igual.

| | EBITDA | Net income |
|---|---:|---:|
| A2, A3, A4 | −34.070 | −34.070 |
| B3, solo Edificio Leones | −22.596 | −22.596 |
| A8, 4×1000 de G4003 a G4013 | +1.494 | 0 |
| A5, A6, A7, A9, A10, A11 y lo que falta de A8 | – | −38.603 |
| **Total** | **−55.172** | **−95.269** |

USD a la tasa del Master de cada mes. Odoo al 9-nov; Master, foto del 6-nov.

## Método
- **Master:** Financials DB, foto del 6-nov (`Diferencias P&L Master vs Odoo 2026 (2026-11-06).xlsx`, en la carpeta de Octubre del cierre). La comparación es por país, código y mes.
- **Odoo:** XML-RPC de solo lectura, el 9-nov. El código G sale primero de la analítica del plan Centros y, si no hay, del nombre de la cuenta, igual que en `agent_pl_export`. Para cada diferencia se revisaron los asientos, los partners y el `create_date`.
- **Prueba de «registrado después del cierre»:** si el asiento en Odoo se creó después de la fecha de cierre del mes en el Master, la diferencia se explica sola. Las fechas de cierre están en [[P&L Master — cargar ajustes de Odoo en Financials DB]].
- **Antes de dar por buena una diferencia, se buscó si el Master ya la tenía con otro código o si Odoo la tenía dos veces.** Así aparecieron A1, A8 y B4.

## Evidencia

### Capítulo A
| Punto | Veredicto | Efecto en USD | Evidencia |
|---|---|---:|---|
| A1 · Google Ads México, abr–may | No agregar; mover de país es opcional | 0 (≈+490 en mayo si se mueve) | Inc registró el consumo completo: 21.064 en abril (OTADJ/2026/04/0021) y 20.972 en mayo (OTADJ/2026/05/0025), contra la 2.4.3. Están en el Master en las filas 19840 y 20386. México además registró 213.087 MXN en abril (MISC/2026/04/0023) y 354.532 en mayo, después del cierre. |
| A2 · App Store / Play Store USA, ene–abr | Aplicar | −19.353 | Devengos OTADJ de Apple y Play creados el 2-jun: ene 2.788; feb 864 + 3.435; mar 920 + 4.901; abr 969 + 3.308. Abril es igual a mayo: parece estimado. Además, PayPal de marzo: 2.305 en Odoo (creado el 2-may) contra −130 en el Master (fila 19117). |
| A3 · Mercado Pago Propay AR, mayo | Aplicar | −10.200 | OP/2026/05/0011, −14.261.459 ARS, creado el 3-jul. La fila 20080 del Master está en 0. |
| A4 · KPMG USA, agosto | Aplicar | −4.517 | Devengo mensual «Transfer pricing… 6 months» de 4.516,67. MISC/2026/08/0006 se creó el 21-sep. El Master tiene julio, septiembre y octubre. |
| A5 · ISN México, ene–may | Aplicar | −10.453 | 601.29.001: ene 35.665, feb 37.606 y mar 34.087 MXN (CARGA del 4-jun); abr 36.484 y may 38.909 (21-sep). El Master solo tiene de junio a agosto (filas 25002–25004). |
| A6 · IIBB Argentina, ene–mar | Aplicar | −9.637 | IBCM de ene 3.957.944 ARS (24-feb), feb 5.223.186 (30-mar) y mar 4.472.534 (4-may): se presenta después de fin de mes. Opcional: abril +453 y mayo −21. El Master tiene 5.223.186 en los dos meses, que es la cifra de febrero; Odoo ajustó el 16-sep. |
| A7 · Recargos SAT, agosto | Aplicar; anotar que es de 2025 | −8.359 | MISC/2026/08/0004 «Devengo SAT Vigilancia Profunda 2025»: es IVA de 2025 que determinó la auditoría. Rafael Rodríguez pasó la línea de 142.662 MXN a recargos el 21-sep. |
| A8 · ICA y 4×1000 Colombia | Aplicar con el ajuste | EBITDA +1.494; Taxes −6.094 | El 4×1000 de mayo a agosto ya está en el Master en G4003 (filas 20508, 20913, 21409 y 22170, ≈1.494). Lo que falta: 4×1000 de ene–abr; ICA de feb (4.099.000 COP) y may (6.035.000), creados el 25-jun; complemento de ICA de agosto, 6.486.000 (Vario/2026/08/0025, creado el 16-oct); «impuestos asumidos» de ene 88.323, mar 21.602 y jun 142.340. |
| A9 · Intereses del plan de facilidades AR, agosto | Aplicar | +5.804 | DLOCA/2026/00612, «Resarcimiento intereses»: dLocal devolvió 8.699.773 ARS, con crédito a G4007. Rafa la modificó el 20-oct. El devengo de −16.267.607 ya está en el Master (fila 21664). |
| A10 · Intereses Chile, mayo | Aplicar | −4.988 | BancoEstado 3.389.339 CLP (MISC/2026/05/0005–0006, creados el 29-may, antes del cierre) + Grenke 1.082.871. En el Master, mayo solo tiene filas en 0 (20547 y 20549). |
| A11 · Financial income USA, ene–feb | Aplicar | −4.877 | Las filas manuales de FFMM del Master son el rendimiento bruto. Odoo descuenta el cobro de Chase por el sweep (≈3.000 al mes) y ajusta el devengo con el monto real. Desde marzo, el Master ya es igual a Odoo. |

### Capítulo B
| Punto | Veredicto | Efecto en USD | Evidencia |
|---|---|---:|---|
| B1 · Transbank Chile, ene–mar | Probable; confirmar con Nico | −21.870 | Odoo de ene a abr solo tiene la «Provisión Transbank Propay», creada el 4-may: 36,99, 33,77, 38,85 y 38,12 MM CLP. El Master tiene créditos manuales de +15.951.410 en enero (fila 16628) y +20.488.937 en marzo (fila 18522); falta saber qué eran. De mayo a octubre, el Master es igual a Odoo. |
| B2 · Nómina Chile | Todavía no | — | La recarga con la analítica nueva movió los códigos de toda la nómina, con diferencias de ±100 mil por código. Lo que queda neto (mar −18,0k, abr −20,8k, jun +10,7k, jul −5,5k) hay que cuadrarlo contra el libro de remuneraciones. |
| B3 · Gastos comunes Chile, ene–may | Aplicar Edificio Leones | −22.596 | MISC/2026/01/0031, 02/0029, 03/0032, 04/0025 y 05/0024, todos creados el 2-jul, en G3100: 4,38, 3,96, 3,95, 3,83 y 3,99 MM CLP. El Master los tiene desde junio (fila 20761). El resto (G3101 contra G3104 y Clean Partners) es solo cambio de código o de mes. |
| B4 · Chargebee USA | No aplicar; Odoo cuenta dos veces | — | El devengo mensual de 4.649 (6.267 en sep y oct) va contra la 2.4.0 y nunca se reversa. Las facturas también se cargan a gasto contra la 211000: BILL/2026/06/0007, 288; BILL/2026/07/0009, 7.411; BILL/2026/07/0008, 1.662. MISC/2026/08/0005 es un ajuste de −3.151. |
| B5 · Nómina USA, jun–jul | Probable; confirmar con Nico | +20.330 | Casi todo es la provisión de vacaciones en la 5.4.0, rehecha el 1-sep (SLR/2026/06/0001–0002 y 07/0001–0002). Junio da −1.301 neto en Odoo contra −7.050 en el Master (fila 21025). En julio la provisión baja de 54.367 a 29.570 (+24.798); el Master no tiene fila de vacaciones en julio. |
| B6 · México, ene–may | Todavía no | — | El Q1 en Odoo tiene ~56–64k al mes de nómina sin código G. Además, las comisiones de pago están en G0009 en el Master (~+25k) y en G4003 en Odoo (~−17k al mes). |

### Reservo
- **México, octubre:** la fila 24738 trae el Recurring (254.758 MXN) y la 24913, que viene del export de Odoo (401.01.001 I0000), trae otros 43.142 MXN. Los demás meses solo tienen el Recurring. La 24913 va en 0. Afecta solo el P&L de Reservo.
- **Chile, octubre:** el AI Revenue de la fila 24741 (14.853.228 CLP) es igual al de septiembre (fila 23357). Recurring y Payment Processing sí cambiaron.

## Caveats
- Los números de fila son de la foto del 6-nov. En la planilla en vivo pueden estar corridos, así que hay que ubicar cada fila por país, detalle, fecha y monto.
- Los efectos usan la tasa del Master de cada mes. Al aplicar, medir el antes y el después en P&L (current FX).
- **Seguimiento con Contabilidad** (no mueve el Master todavía):
  - A1: reversar en Inc la parte de México en la 2.4.3; registrar el cargo intercompañía de 102.200 MXN pagado con Brex; aclarar la transferencia de Global66 de 90.000 MXN del 20-may.
  - B4: aplicar las facturas de Chargebee contra la 2.4.0 y revisar el saldo de esa cuenta.
- Queda abierto en [[Aplicar en el P&L Master las diferencias con Odoo de 2026 (revisión del 6-nov)]].

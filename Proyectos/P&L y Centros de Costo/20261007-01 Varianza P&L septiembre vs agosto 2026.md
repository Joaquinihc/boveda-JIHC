---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-10-07
temas: [pnl, cierre-mensual, headcount, payments]
metricas: []
---

# Varianza P&L septiembre vs agosto 2026

**Pregunta**: ¿Qué explica la variación del EBITDA de septiembre 2026 contra agosto, separando lo operativo del tipo de cambio?

Página publicada (versión completa, con detalle por línea y respuestas a la revisión): [Análisis de varianza P&L](https://app.notion.com/p/agendapro/An-lisis-de-varianza-P-L-3f2ab34c233b80b29971e6046b22e377), en Notion › Cierre septiembre 2026 › Revisión P&L septiembre 2026.

## Conclusión
> Corte: planilla *P&L AgendaPro - Master 2026 (5).xlsx* (hojas Financials DB y Mappings), 7-oct-2026. Ese mismo día el DWH (`finance.profit_and_loss`) ya la tenía cargada y coincidía fila por fila. Miles de USD; costos en negativo.
> - **EBITDA sep-26: −254,2k (−22,6% del revenue)**, contra −244,3k en agosto: **empeora −9,9k** = **−16,3k operativo** + **+6,4k tipo de cambio** (CLP +3,4%).
> - La versión previa del DWH mostraba +11,3k. La diferencia (−21,2k) son dos ajustes de la planilla: finiquito de Product Argentina registrado como payroll (−16,0k) y pérdida por diferencia de cambio en Colombia (−5,6k).
> - **Revenue plano en USD (+1,2k), pero +23,1k (+2,1%) al TC de agosto**: AI +12,4k (+22%), Software +10,3k (+1,2%), Payments +0,4k.
> - **Partidas puntuales: −27k netos.** A favor: no se repiten finiquitos y bonos de Sales MX/CO de agosto (+31,0k). En contra: eventos (−22,9k), finiquito Product AR (−12,2k), diferencia de cambio contable (−11,9k), IMSS (−11,6k), provisión de vacaciones de Chile (−8,5k). **Variación de fondo ≈ +11k.**
> - **Margen bruto 69,3% → 66,6% (−2,7pp)**: Expansion +10 personas (−18,2k, −1,4pp), Payments pierde 3,1pp de margen (33,6% → 30,5%) y los modelos IA pasan a Platform (−6,0k).
> - **Payroll al TC de agosto: sube −14,4k.** En USD baja +4,5k, pero solo por TC (+19,1k, de los cuales +17,0k Chile). Payroll de fondo crece ≈ −20k/mes (+2,7%), casi todo por las altas de Expansion.

## Método
- Planilla reconstruida con la estructura de la tabla del DWH: Account name y P&L Category por código en Mappings (primera coincidencia, como el VLOOKUP), sección por categoría, USD = moneda local ÷ TC de cierre de la hoja Exchange rates. 0 diferencias contra la planilla en ene–sep 2026.
- EBITDA = Revenue + Service + Acquisition + Corporate (como Financial Performance); sin Financing, Depreciation ni filas `#N/A` (Taxes).
- Operativo = septiembre en moneda local al TC de agosto (CLP 915,5 · MXN 17,07 · COP 3.120 · ARS 1.495); FX = el resto.
- Puente con grupos excluyentes (cada fila del P&L en un solo grupo) y "Resto"; cuadra exacto.

## Evidencia

**Puente de EBITDA (al TC de agosto)**

| Driver | Impacto |
|---|---:|
| EBITDA agosto | −244,3 |
| Revenue AI / Software / Payments | +12,4 / +10,3 / +0,4 |
| Costo de procesamiento de pagos (Chile −6,1; México −1,3) | −7,2 |
| Expansion (+10 HC: Chile +9, México +1) | −18,2 |
| Finiquitos y bonos de Sales de agosto que no se repiten | +31,0 |
| Eventos (Brand México 13,0 + evento 18-sep Chile 9,9) | −22,9 |
| Diferencia de cambio contable (Colombia, Argentina) | −11,9 |
| Payroll Product Argentina (finiquito 23,7 MM ARS) | −12,2 |
| IMSS México (patrón bimestral) | −11,6 |
| Provisión de vacaciones Chile | −8,5 |
| Asesorías legales, contables y de auditoría | +8,0 |
| Paid media sin eventos | +5,0 |
| Nube, gateways y herramientas SaaS | −2,2 |
| Reclasificación de modelos IA (neto, a validar) | +2,7 |
| Resto (payroll −6,3; no-payroll +15,2) | +8,9 |
| **Variación operativa** | **−16,3** |
| Tipo de cambio (revenue −21,8; costos +28,2) | +6,4 |
| **EBITDA septiembre** | **−254,2** |

**Payroll (−14,4k al TC de agosto)**: indemnizaciones y bonos de agosto que no se repiten +26,7 · finiquito Product AR −12,2 · Expansion −18,7 (Chile 18 → 27, México 1 → 2) · vacaciones y provisión −8,7 · variable (ISR en MX/CL/CO) −5,6 · Argentina resto +3,0 · sueldo base y cargas +1,1.
- **Onboarding (+8,2k operativo) no es TC** (TC solo +0,9k): una persona de Chile pasó a Retention (no hubo salida), comisiones Colombia −75% y México −30%, y agosto traía pagos de salidas de jul–ago.
- **Sales**: ISR (G1108) sube −15,4k (comisiones y variable); Payroll Sales (G1100) baja +35,1k (pagos de salida de agosto).

**Payments**: en México el costo sube con transacciones −4,0% (DEMERGE +12,0%, OEL +8,3% en MXN; OEL 16% sobre el modelo por procesador): no es volumen. En Chile el volumen crece +4,0%, pero el take rate cae 4 bps (crece POS Transbank, el canal más caro) y la Provisión Haulmer sube +5,6% contra +0,7% del modelo. Volumen desde `dwh.company_sales_months`.

## Caveats
- La separación puntual vs recurrente ("≈ +11k de fondo") es juicio propio; el puente sí cuadra exacto.
- La reclasificación de modelos IA deja 2,7k acreditados sin contraparte: si es error, el EBITDA de septiembre está sobreestimado en 2,7k.
- El desglose de payroll por concepto existe recién desde agosto 2026; la provisión de vacaciones de Chile tiene dos meses de historia.
- Calidad de datos de la planilla (detalle en Notion): finiquito AR como sueldo; eventos de México en G1006 Google Ads (inflaría Paid Search MX +55% en Marketing Spend y board book); G4004 (diferencias de cambio) dentro del EBITDA; Taxes `#N/A` sin sección (−30,3k ago, −6,7k sep); códigos duplicados en Mappings, ver [[Claude Code — Códigos G repetidos en la Master del P&L]].
- Acciones sugeridas a contabilidad (en Notion, decisiones de otros): facturas OEL/DEMERGE de México, ajuste de la Provisión Haulmer contra factura, provisión Stripe MX de 150.000 MXN a ajustar en octubre, posible doble cobro de CodeRabbit (1.710 = 2 × 855), Chargebee 6.267 (overage o cambio de plan), IMSS provisionado mensual, provisión de auditoría (ver con [[Nicolás Astudillo]]). Octubre: mes completo de las 10 altas de Expansion.

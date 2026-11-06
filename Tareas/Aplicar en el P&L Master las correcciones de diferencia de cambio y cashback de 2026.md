---
type: tarea
estado: en-revision
prioridad: media
tiempo:
tipo: cuadratura
origen: manual
proyecto: "[[_P&L y Centros de Costo]]"
temas:
  - pnl
  - cierre-mensual
  - odoo
fuente:
fecha_limite:
creado: 2026-11-05
revisar: false
id: T-117
---

# Aplicar en el P&L Master las correcciones de diferencia de cambio y cashback de 2026

Llevar al P&L Master 2026 el plan de diferencia de cambio acordado el 5-nov: F1003 y F1004 en Financial expenses, G4004 de vuelta a cashback y las filas de 2026 con los montos de Odoo. Odoo ya quedó alineado: lo hizo [[Nicolás Astudillo]] el 5-nov. Detalle en [[20261105-01 Diferencia de cambio y cashback en el P&L Master 2026 (F1003, F1004 y G4004)]].

## Subtareas
- [x] Aplicar en Mappings (filas 68, 167 y 168) y en Financials DB (15 filas modificadas, 9 eliminadas y 54 agregadas), y comparar el EBITDA y Financial expenses antes y después. Listo cuando F1003 + F1004 cuadren con Odoo por país y mes, y en 2026 G4004 quede solo con cashback.
  - 2026-11-05: aplicado, junto con el IMSS, la nómina de México y el Brex. Diferencia de cambio: 18 filas modificadas y 76 agregadas, sin eliminar filas existentes. F1003 + F1004 cuadran con Odoo por país y mes. Detalle en [[20261105-01 Diferencia de cambio y cashback en el P&L Master 2026 (F1003, F1004 y G4004)]] y [[20261105-02 IMSS y nómina de México en el P&L Master 2026]].
- [x] Mandarle a [[Nicolás Astudillo]] los pendientes de Odoo:
  - el rubro de consolidación de la 6.0.00.02 (Chile) y de la 641000 (Inc);
  - la etiqueta «CFM: Exchange Rate» de la 5.3.0.00.10 (Argentina);
  - que no registre el Brex de febrero sin avisar;
  - que el ajuste Inc–México de Rule 57 no caiga en octubre.
  - 2026-11-05: borrador en el hilo de #finance con los puntos 1 a 3, los rubros de cashback, SC3 y Software Reservo sin rubros, y el Brex de febrero. Falta que Joaquín lo envíe.
  - 2026-11-05: enviado; Nico respondió.

## Notas
- **Documento para el equipo:** Notion, [Ajustes P&L cierre octubre](https://app.notion.com/p/agendapro/Ajustes-P-L-cierre-octubre-40fab34c233b803f8850e2432b10ee9c) (5-nov). Explica cada ajuste y su efecto por línea y mes en EBITDA, Financial expenses y Net income.
- **Bitácora de cambios por etapa:** `Bitácora cambios P&L Master 2026-11-05.md` y `Bitácora cambios P&L Master 2026-11-06.md` (finiquitos de mayo), en la carpeta de Octubre.
- **Plan y filas a pegar:** `/Users/joaquinherreracabrejos/Documents/Cierre de mes/P&L/Octubre/`, archivos `Correcciones diferencia de cambio 2026.md` y `Ajustes diferencia de cambio 2026 v2.xlsx`. Usan la numeración del Master (6). En la planilla en vivo las filas están corridas, así que hay que ubicarlas por país, detalle, fecha y monto.
- **Fila 19988 de la planilla en vivo** (Argentina, abril, 5.3.0.00.07): también cambia el monto, de −303.270,30 a −222.398,22, según Odoo.

### Pendientes menores
- 2026-11-05 · IMSS México: Pablo decidió el 4-nov ajustarse a Odoo. El plan está en `Correcciones IMSS Mexico 2026.md`, en la misma carpeta. Confirmar si ya se aplicó en el Master. → 2026-11-05: aplicado ([[20261105-02 IMSS y nómina de México en el P&L Master 2026]]).
- 2026-11-05 · México: el costo de cambio de Global 66 de enero (+5.093,22 MXN) y de marzo (−5.774,75) quedó en F1004. En febrero, el mismo concepto se reclasificó a G4003. El neto es −681,53 MXN (~37 USD).
- 2026-11-05 · Repo `mcp-odoo-agendapro`: el PR #908 de Nico (abierto) alinea Rule 57, el pre-cierre y las guías de cierre con F1003 y F1004. Revisarlo y decidir el merge.
- 2026-11-05 · Verificar que `finance.profit_and_loss` (DWH) tome F1003 y F1004 en Financial expenses en la próxima carga del Master, para que los tableros muestren el EBITDA nuevo.
- 2026-11-05 · Brex de febrero 2026 (2.508,78 USD): la fila manual del Master (G4004, febrero) se mantiene. En Odoo el canje está sin efecto en resultados: el UPLOA/2026/02/0072 tiene las dos líneas en la 5.2.35 G2106. No aparece en otro mes ni en la 6.1.1 ni en la 5.2.33. **Sacar la fila manual solo si Contabilidad lo registra en Odoo como cashback**, porque si no, el export lo trae de nuevo y queda dos veces.
- 2026-11-05 · Nómina de México de julio: al rehacerla en Odoo el 1-sep (SLR/2026/07/0003), Sales y Sales ISR bajaron 167.392 MXN en total. La consulta a [[Nicolás Astudillo]] está en borrador en su DM; falta que Joaquín la envíe. → 2026-11-06: resuelto en Odoo, sin esperar la respuesta. Son finiquitos de mayo que Contabilidad pasó a mayo al rehacer julio (SLR/2026/05/0002, 167.391,68 MXN). Se agregaron en mayo del Master junto con dos finiquitos de mayo registrados el 2-oct: 272.139,97 MXN en total, EBITDA de mayo −15.720 USD ([[20261105-02 IMSS y nómina de México en el P&L Master 2026]]).
- 2026-11-06 · Nómina de México de enero a mayo: Odoo rehízo completas la de abril (21-sep) y la de mayo (1-sep), después del cierre del Master. Solo se comparó Sales e ISR; falta comparar el resto de las áreas contra el Master.
- 2026-11-05 · Marketing de México de julio difiere de Odoo en 1.383 MXN (≈79 USD). No se ajustó.
- 2026-11-05 · Financials DB, filas 4150 y 4151 (DLA Piper Stock Options): la fecha tiene #REF! y no caen en ningún mes.
- 2026-11-05 · P&L (AP+Reservo): sus fórmulas filtran solo Type «Real», así que no suma Reservo.

## Historial
- 2026-11-05 · Creada desde Claude Code (Joaquín en conversación)
- 2026-11-05 · Avance desde Claude Code: borrador a Nico con las cuentas sin asignar; Brex de febrero anotado (la fila manual se queda mientras Odoo no lo registre como cashback)
- 2026-11-05 · Avance desde Claude Code: Master alineado con Odoo (diferencia de cambio, IMSS y nómina de México, Brex); documentado en Notion; consulta a Nico por Sales e ISR de julio en borrador; estado → en-revision (pedido de Joaquín)
- 2026-11-06 · Avance desde Claude Code: la baja de Sales e ISR de julio eran finiquitos de mayo; se agregaron en mayo del Master (5 filas, 272.140 MXN) y se actualizó Notion
- 2026-11-06 · Mencionada de nuevo en [[2026-11-05 Diferencia de tipo de cambio y cierre del P&L de octubre (Pablo)]] (devolver la cuenta de comisiones bancarias a su nombre, ajustar los registros de diferencia de cambio, cuenta nueva intercompany vs externos y documentar los cambios del mes)

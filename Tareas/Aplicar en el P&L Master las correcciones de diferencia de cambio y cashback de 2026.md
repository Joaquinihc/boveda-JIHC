---
type: tarea
estado: pendiente
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
- [ ] Aplicar en Mappings (filas 68, 167 y 168) y en Financials DB (15 filas modificadas, 9 eliminadas y 54 agregadas), y comparar el EBITDA y Financial expenses antes y después. Listo cuando F1003 + F1004 cuadren con Odoo por país y mes, y en 2026 G4004 quede solo con cashback.
- [ ] Mandarle a [[Nicolás Astudillo]] los pendientes de Odoo:
  - el rubro de consolidación de la 6.0.00.02 (Chile) y de la 641000 (Inc);
  - la etiqueta «CFM: Exchange Rate» de la 5.3.0.00.10 (Argentina);
  - que no registre el Brex de febrero sin avisar;
  - que el ajuste Inc–México de Rule 57 no caiga en octubre.

## Notas
- **Plan y filas a pegar:** `/Users/joaquinherreracabrejos/Documents/Cierre de mes/P&L/Octubre/`, archivos `Correcciones diferencia de cambio 2026.md` y `Ajustes diferencia de cambio 2026 v2.xlsx`. Usan la numeración del Master (6). En la planilla en vivo las filas están corridas, así que hay que ubicarlas por país, detalle, fecha y monto.
- **Fila 19988 de la planilla en vivo** (Argentina, abril, 5.3.0.00.07): también cambia el monto, de −303.270,30 a −222.398,22, según Odoo.

### Pendientes menores
- 2026-11-05 · IMSS México: Pablo decidió el 4-nov ajustarse a Odoo. El plan está en `Correcciones IMSS Mexico 2026.md`, en la misma carpeta. Confirmar si ya se aplicó en el Master.
- 2026-11-05 · México: el costo de cambio de Global 66 de enero (+5.093,22 MXN) y de marzo (−5.774,75) quedó en F1004. En febrero, el mismo concepto se reclasificó a G4003. El neto es −681,53 MXN (~37 USD).
- 2026-11-05 · Repo `mcp-odoo-agendapro`: el PR #908 de Nico (abierto) alinea Rule 57, el pre-cierre y las guías de cierre con F1003 y F1004. Revisarlo y decidir el merge.
- 2026-11-05 · Verificar que `finance.profit_and_loss` (DWH) tome F1003 y F1004 en Financial expenses en la próxima carga del Master, para que los tableros muestren el EBITDA nuevo.

## Historial
- 2026-11-05 · Creada desde Claude Code (Joaquín en conversación)

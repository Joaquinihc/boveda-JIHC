---
type: proyecto
estado: activo
temas: [pnl, centros-de-costo, odoo, board]
inicio: 2026
responsable: "[[Joaquín Herrera]]"
---

# P&L y Centros de Costo

**Objetivo**: Reestructurar el P&L (visión SaaS multiproducto), dejar los CECOs de Odoo consistentes y llevar P&L por vertical al Board Book.

## Contexto
- CECOs: división+área en el CECO, país aparte; PIT-TEC (G2100) / PIT-PRO (G2000); Revenue Sales (G1100); regla de privacidad (ningún CECO de 1 persona). Retroactivo enero→hoy vía agente de Mario.
- Provisiones/ingresos diferidos: impacto P&L = delta de provisión; tokens IA se redistribuyen por vertical con el informe de tarjetas (asiento mensual de Juanjo).
- 2026-10-02 · Costos de IA por API key: cuentas G nuevas G0030–G0035 (servicio por equipo, Platform), G0205 (soporte Marketplace) y G2112–G2116 (tecnología por vertical, Development). Reglas y aprobación en [[Imputación contable de costos de IA (API keys)]]; montos en [[20261002-01 Reclasificación contable de costos de API keys de IA]].
- P&L SaaS: benchmarkear Shopify/Fresha; armar Notion estructura actual vs propuesta; costos de vendedores hoy castigan margen.
- Guía: [[Guía clasificación de personas (headcount)]]. Tareas Notion: Restructurar P&L, P&L verticales a Board Book, Headcount + clasificación, costos API Keys.
- 2026-10-29 · Reservo entró a Odoo el 19-oct-2026 como dos sociedades, SC3 SpA (Chile) y SOFTWARE RESERVO (México), con planes de cuentas clonados de AgendaPro Chile y México y cargadas por [[Nicolás Astudillo]] el 22 y 23-oct; sin nómina 2026 ni cost center HQ/país, y varias cuentas cajón con criterio distinto. Detalle en [[20261029-01 Revisión del P&L de Reservo en Odoo vs estructura AgendaPro]]; seguimiento en [[Revisar P&L de Reservo gasto por gasto (cuentas Odoo)]].
- 2026-11-02 · Costos de IA en Odoo: desde octubre, servicio y desarrollo van separados. El desarrollo usa la analítica de su vertical sobre 5.2.25 G2104, y septiembre se queda con G2104 ([[Imputación contable de costos de IA (API keys)]]). Para los códigos de remuneraciones repetidos, la Master quedó recodificada con G1207–G1210 y G1128, a la espera de los que defina Contabilidad en Odoo ([[Claude Code — Códigos G repetidos en la Master del P&L]]).
- 2026-11-03 · Contabilidad ([[Nicolás Astudillo]]) renombró en Odoo las cuentas y analíticas de remuneraciones de verticales con los códigos propuestos (G1207–G1210 y G1128); la Master ya cuadra, sin recodificar de nuevo. El G-code de la nómina llega desde la analítica del plan Centros, no desde la cuenta contable. Descuadres menores y pendientes en [[Claude Code — Códigos G repetidos en la Master del P&L]].
- 2026-11-05 · Revenue de Reservo en la planilla de conciliación y el P&L: tres flujos, por moneda en que Reservo cobra. Software = pagos de suscripción por moneda de pago; Vambe = 25 % de los planes activos al día 1 del mes siguiente, factura en CLP con IVA dentro, neto por mes de servicio; Mercado Pago = solo Chile, variación del saldo total de la cuenta mientras no haya liquidaciones. Detalle y pendientes en [[20261105-01 Revenue share de Vambe y comisiones de Mercado Pago en el cierre de octubre de Reservo]].
- 2026-11-04 · Criterios de la revisión del P&L de octubre:
  - Las terminales POS de México se difieren a 6 meses como costo de venta, igual que la compra anterior. Lo aprobó Julio Guzmán: BILL/2026/10/0020, 6 cuotas de 23.600 MXN de oct-26 a mar-27 en 501.08.003 G0012.
  - Marketplace Chile: la 4.0.00.11 Ingresos Marketplace queda en 0 por diseño, porque el cofinanciamiento del banco se compensa en la misma cuenta. G1011 Marketing Marketplace es el costo por fraude en la alianza con bancos.
- 2026-11-05 · Diferencia de cambio y cashback (decisión de [[Pablo Lucero]]):
  - La diferencia de cambio va siempre debajo del EBITDA, en Financial expenses, con F1003 (intercompañía) y F1004 (terceros, código nuevo). Las comisiones bancarias, los cashbacks y otros van dentro del EBITDA.
  - G4004 vuelve a «Cashback tarjetas corporativas» (Other Corporate), que es su uso en el Master desde 2021. G4005 nunca se usó.
  - [[Nicolás Astudillo]] alineó Odoo el 5-nov, y el Master quedó alineado el mismo día: diferencia de cambio, IMSS y nómina de México, y Brex. El 6-nov se agregaron en mayo los finiquitos de México (272.140 MXN) que Odoo había pasado de julio a mayo, más dos registrados el 2-oct. Efecto enero–octubre, en USD: EBITDA +17.495, Financial expenses −146.900, Taxes −6.895 y Net income −136.295. IMSS y finiquitos en [[20261105-02 IMSS y nómina de México en el P&L Master 2026]]; para el equipo, en Notion: [Ajustes P&L cierre octubre](https://app.notion.com/p/agendapro/Ajustes-P-L-cierre-octubre-40fab34c233b803f8850e2432b10ee9c).
  - Criterio general (Pablo, 4-nov, a propósito del IMSS de México): cuando el Master y Odoo difieren, manda Odoo.
  - Detalle en [[20261105-01 Diferencia de cambio y cashback en el P&L Master 2026 (F1003, F1004 y G4004)]].

## Próximos pasos
- [ ] Notion con estructura actual del P&L vs propuesta + benchmark de públicos.
- [ ] Cerrar distribución de costos de plataforma entre verticales (G-1000).
- [ ] Seguimiento del P&L retroactivo con nuevos CECOs (agente de Mario).

## Notas de este proyecto
Los análisis viven en esta carpeta con ID de fecha (`YYYYMMDD-NN Título`). Las reuniones relacionadas se linkean desde `Reuniones/` vía frontmatter `proyectos:`.


## Tareas de este proyecto
```base
filters:
  and:
    - type == "tarea"
    - file.inFolder("Tareas")
    - estado != "hecho"
    - estado != "descartada"
    - proyecto.contains("_P&L y Centros de Costo")
formulas:
  orden_prioridad: if(prioridad == "alta", 1, if(prioridad == "media-alta", 2, if(prioridad == "media", 3, if(prioridad == "media-baja", 4, if(prioridad == "baja", 5, 6)))))
  orden_tiempo: if(tiempo == 1, 1, if(tiempo == 2, 2, if(tiempo == 3, 3, if(tiempo == 4, 4, if(tiempo == 5, 5, 9)))))
  tiempo_txt: if(tiempo == 1, "⏱ ≤2 h", if(tiempo == 2, "⏱ ½ día", if(tiempo == 3, "⏱ 1 día", if(tiempo == 4, "⏱ 2–3 días", if(tiempo == 5, "⏱ 1 sem+", "")))))
properties:
  formula.tiempo_txt:
    displayName: Tiempo
views:
  - type: table
    name: Vivas
    order:
      - id
      - file.name
      - estado
      - prioridad
      - formula.tiempo_txt
      - fecha_limite
    sort:
      - property: formula.orden_prioridad
        direction: ASC
      - property: formula.orden_tiempo
        direction: ASC
```

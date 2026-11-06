---
type: sugerencia
origen: slack
temas: [headcount, dwh]
criterio: "(b) cambio acordado en tablas del DWH: finance.headcount suma a Reservo con una columna nueva"
fuente: https://agendapro.slack.com/archives/D095G9DL97T/p1793733451084779?thread_ts=1793733451.084779&cid=D095G9DL97T
canal: "DM con Pablo Lucero + DM con Israel Gutiérrez + DM con Nicolás Astudillo"
fecha_hilo: 2026-11-03
creado: 2026-11-06
revisar: false
---

# Slack — finance.headcount suma a Reservo con columna employer (y la carga del cash flow pierde amount_usd)

- **Diseño (3-nov, Pablo responde «super!»)**: las 51 personas de Reservo (nómina Chile, CLP) entran a `finance.headcount` con una columna nueva `employer` = AgendaPro / Reservo; las filas de AgendaPro no cambian. En «Headcount - Raw», pestaña «Reservo CL» con el área de Reservo y su equivalente AgendaPro por cargo (Ejecutivo comercial → ISR, Customer success analyst → Retención, Desarrollador → Platform Tech), que viaja como `division_area_subarea`. Consultas sin filtro `employer = 'AgendaPro'` cambian: Chile pasa de 127 a 178 activos. Joaquín deja los reportes de Evidence (board book, OKR, budget follow-up, People) en AgendaPro y agrega una vista de grupo.
- **Orden pactado con Israel**: columna I en «2026 DWH Loading» › Headcount (al inicio todo AgendaPro) → stream `finance.raw__headcount` hasta la I → PR de dbt → recién ahí entran las filas de Reservo. A Nicolás se le avisó con el nombre `company` (no `employer`): ojo con el nombre final; sus agentes `agent_payroll_review` y `agent_bu_reviewer` leen la tabla sin filtro de empresa, y Nicolás dice que esos agentes los creó Pablo.
- **Incidente relacionado (5-nov)**: al sumar Reservo a la pestaña cash_flow, se perdió `amount_usd` (los dashboards de caja del Board Book quedaron sin USD desde el 4-nov 16:27) y apareció `company`. Joaquín repuso la columna; Israel pidió ampliar el importrange de `Dataset!A1:H25000` a `Dataset!A1:H` porque quedaban sin USD casi todo México, Inc, SC3 y Reservo. Sin confirmación en el hilo de que quedó corregido.
- Relacionada: [[Definir cómo clasificar a las personas de Reservo en la planilla de headcount]].

Hilos: https://agendapro.slack.com/archives/D095G9DL97T/p1793733451084779 · https://agendapro.slack.com/archives/D09C9SSG4HJ/p1793733597904209 · https://agendapro.slack.com/archives/D0A3UHZ8ABW/p1793732713051569 · https://agendapro.slack.com/archives/D09C9SSG4HJ/p1793895377576109

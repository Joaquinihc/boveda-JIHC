---
type: sugerencia
origen: slack
temas: [dwh, gmv, definiciones, board, reportes]
criterio: "(a) cómo se cuentan las reservas y el GMV reportados + (c) explica por qué baja un mes ya cerrado"
fuente: https://agendapro.slack.com/archives/C029RSSC69X/p1775237852058589
canal: "#data (hilo de abr-26 retomado por Joaquín)"
fecha_hilo: 2026-11-20
creado: 2026-11-20
revisar: false
---

# Slack — Reservas, fct_bookings_created_monthly cuenta 31 % más que el Informasayuno y company_sales_months baja después del cierre

- Dos conteos en el DWH: `fct_bookings_created_monthly` (propuesta de Israel en abril) cuenta todas las reservas creadas en el mes, en cualquier estado, incluidas sesiones de paquete sin agendar; `company_sales_months` (fuente del GMV y de las reservas del Informasayuno y del Board Book) cuenta solo las creadas en el mes que hoy siguen vigentes y agendadas (reservada, confirmada, asiste, en espera). Octubre al 19-nov: 5,20 M contra 3,98 M (+31 %: canceladas 660k, sesiones sin agendar 266k, no asiste 222k, pendientes 49k, filtro de ticket alto 26k; importadas −23k).
- Por qué baja un mes cerrado: cada reserva se cuenta con su estado de hoy y dbt recalcula los últimos 90 días; las creadas en el mes con cita posterior salen al cancelarse o quedar «no asiste». Octubre pasó de 4,01 M (Informasayuno 13-nov) a 3,98 M; junio bajó 4,2 % en reservas y 4,7 % en GMV, julio 3,4 % y 4,6 %, y luego se congelan. El GMV baja por lo mismo.
- Propuesta de Joaquín: guardar una foto al cierre por compañía y mes (reservas, GMV y GPV) y reportar esa. Pendiente: con Pablo Lucero definir qué número va al Board Book (sin fecha).
- Paquetes = servicios con sesiones: al venderse se crea una reserva por sesión con la fecha de compra; en octubre ~550 mil reservas de sesión, la mitad sin agendar.

Hilo: https://agendapro.slack.com/archives/C029RSSC69X/p1775237852058589
Mensaje de Joaquín (20-nov): https://agendapro.slack.com/archives/C029RSSC69X/p1795180810043699?thread_ts=1775237852.058589&cid=C029RSSC69X

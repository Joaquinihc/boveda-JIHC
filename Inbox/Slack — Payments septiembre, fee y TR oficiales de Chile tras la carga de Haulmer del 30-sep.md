---
type: sugerencia
origen: slack
temas: [payments, revenue, cierre-mensual, dwh]
criterio: "(c) conclusión que explica un número"
fuente: https://agendapro.slack.com/archives/D09C9SSG4HJ/p1790887570866009
canal: "DM con Israel Gutiérrez; hilo en #data"
fecha_hilo: 2026-10-01
creado: 2026-10-02
revisar: false
---

# Slack — Payments septiembre, fee y TR oficiales de Chile tras la carga de Haulmer del 30-sep

- El 1-oct el cierre de Payments se atrasó: según Israel, por recursos y concurrencia la conexión de Airbyte no alcanzó a extraer las transacciones de Haulmer (propay); varias ejecuciones del día no trajeron nada. Pablo Lucero recordó en #data que habían quedado en que esto no se rompiera el día 1 de cada mes.
- Valor oficial de septiembre (Chile, `dwh.company_sales_months`, solo transacciones con compañía asignada), ya actualizado: fee $147.656.230 · TR $59.854.455. Reemplaza el estimado previo ($148.766.364 / $60.004.553).
- Por qué el estimado venía inflado: suponía que faltaban todas las transacciones Haulmer del 30-sep, pero ~70% ya estaban contadas porque se cobraron por la App de AgendaPro (entran por la pasarela sin esperar el archivo de Haulmer).
- Ojo: el mes en curso aparece con TR negativo mientras el día no termina de procesarse (costos cargados, fees no).

Hilo: https://agendapro.slack.com/archives/D09C9SSG4HJ/p1790887570866009
#data: https://agendapro.slack.com/archives/C029RSSC69X/p1790882320472889?thread_ts=1790881552.002339&cid=C029RSSC69X

---
type: sugerencia
origen: slack
temas: [dwh, ventas]
criterio: "(c) conclusión que explica un número"
fuente: https://agendapro.slack.com/archives/C029RSSC69X/p1789657255944409
canal: "#data"
fecha_hilo: 2026-09-17
creado: 2026-09-23
revisar: false
---

# Slack — Carga de Chargebee al DWH corre una vez al día (05:00 Chile)

- Matías Ulloa reportó que `dwh.merchants_segments` mostraba casi sin cierres del día. Joaquín concluyó que el proceso no está caído: la carga de Chargebee al DWH corre **una sola vez al día, 05:00 Chile (08:00 UTC)**, y luego dbt recalcula `merchants_segments`; los cierres del día aparecen a la mañana siguiente (corrigió su primer mensaje, que decía 7:00/8:00 y 13:00).
- `start_date_pago_mx` no dejó de completarse: es la misma data que `start_date_pago` en hora México, y es la variable correcta para contar cierres por día de negocio.
- Ese día el DWH mostraba 3 cierres (0 B2B3) vs 12 (7 B2B3) en Chargebee directo.
- Para ver cierres del mismo día habría que agregar una segunda sincronización; dueño del proceso: Israel.

Hilo: https://agendapro.slack.com/archives/C029RSSC69X/p1789657255944409

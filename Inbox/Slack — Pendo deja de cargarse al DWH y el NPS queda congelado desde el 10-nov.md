---
type: sugerencia
origen: slack
temas: [dwh, churn, okrs]
criterio: "(b) cambio de tablas del DWH (fuente dada de baja; tablas de NPS congeladas)"
fuente: https://agendapro.slack.com/archives/C029RSSC69X/p1794406359046749
canal: "#data + DM con Emilio Latorre"
fecha_hilo: 2026-11-11
creado: 2026-11-13
revisar: false
---

# Slack — Pendo deja de cargarse al DWH y el NPS queda congelado desde el 10-nov

- Israel (#data, 11-nov): Pendo se dio de baja y se deshabilitó su conexión en Airbyte; desde el 10-nov no llegan datos nuevos. La historia queda, sin actualizarse.
- Afectado: `raw.pendo__poll_events` (Redshift), schema `pendo` (Postgres), `dwh.nps_responses` y el NPS dentro de `dwh.company_churn_features` y del health score por empresa. Si un dashboard muestra NPS congelado desde esa fecha, es por la baja, no una falla.
- Joaquín medía el NPS desde ahí. Emilio Latorre: Pablo está armando una pieza para volver a tener NPS; por ahora no hay fuente. PostHog cubre product analytics, no reemplaza el NPS de Pendo.

Anuncio: https://agendapro.slack.com/archives/C029RSSC69X/p1794406359046749
DM Emilio: https://agendapro.slack.com/archives/D0B83VD95JT

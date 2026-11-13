---
type: sugerencia
origen: slack
temas: [dwh, mrr, churn, waterfall]
criterio: "(b) cambio de modelo del DWH acordado + (c) explica un número (mueve meses cerrados de dwh.mrr)"
fuente: https://agendapro.slack.com/archives/D09C9SSG4HJ/p1794238748558479
canal: "DM con Israel Gutiérrez (+ #pull-request-merge)"
fecha_hilo: 2026-11-09
creado: 2026-11-13
revisar: false
---

# Slack — Chargebee, el mensual deja de arrastrar suscripciones borradas (agendapro-dbt 467) y mueve dwh.mrr de meses cerrados

- PR #467 (Israel, Redshift): `staging.stg_chargebee__subscriptions_monthly` arrastraba el último snapshot sin mirar `deleted`, así que una suscripción borrada seguía viva y, por el dedup, tapaba a la suscripción real. Ahora hay corte de borrado terminal + filtro `not coalesce(deleted, false)` antes del dedup, y dos tests singulares. Postgres (`master`) no tiene el modelo.
- **Joaquín dio el OK el 9-nov** («estamos reportando cifras falsas para ciertos clientes») con la condición de no mergear antes del 11-nov, para que el Board Book enviado calzara. Mergeado el 12-nov; rebuild histórico con `lookback=71` (sin `--full-refresh`, para no botar la vista `dwh.chargebee_subscriptions_snapshot_addons`) corrido el 12-nov 20:24 y propagado a `dwh.mrr` y waterfalls el 13-nov.
- **Efecto en cifras**: estimación del PR, oct-26 (cerrado) +1.548 USD (+0,17 %, 19 empresas). Totales de `dwh.mrr` tras el rebuild según Israel: 893.079 (ago), 900.738 (sep), 892.537 (oct). Caso 20007 (Corporel, MX): queda la suscripción real de 900 MXN y el churn pasa a −900 en octubre (antes −5.947 en noviembre); desaparece el downgrade falso de septiembre.
- Pendiente según Israel: sacar 80 suscripciones de prueba de ene-2021 del staging. Las 3 fuentes de Evidence que leen el staging directo pierden ~192 customers/mes.

Hilo: https://agendapro.slack.com/archives/D09C9SSG4HJ/p1794238748558479
PR en Slack: https://agendapro.slack.com/archives/C02KY47GTHV/p1794230723834149

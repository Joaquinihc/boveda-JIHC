---
type: sugerencia
origen: slack
temas: [dwh]
criterio: "(b) cambio de modelos del DWH acordado (capa PII y espejos _sin_pii)"
fuente: https://agendapro.slack.com/archives/D095EKG3V37/p1793276108023609
canal: "DM con Pablo Marambio"
fecha_hilo: 2026-10-29
creado: 2026-10-30
revisar: false
---

# Slack — Capa PII del DWH (agendapro-dbt 437, 447, 448 y 450), revisión de Joaquín y plan de despliegue

- Proyecto de Pablo Marambio en `agendapro-dbt`: clasificación PII de columnas y vistas por perfil. F1 (#437, solo metadatos) aprobado por Joaquín el 25-oct y mergeado. F2a aplicado en producción (Postgres 26-oct, Redshift 27-oct). F2b: #447 (Postgres) aprobado el 28-oct; #448 (Redshift) se mergeó antes de la aprobación. Joaquín pidió que F3 y F4, que tocan permisos, esperen aprobación antes del merge.
- Acordado en F2b: el mapa cliente Chargebee → compañía y las compañías se materializan; el detector de T2.4 trabaja con límite y filas rotativas; `pii_tipo` obligatorio cuando `pii: true`. Queda para un PR aparte: deduplicar los chequeos T1 (solo en Jinja) y extender el filtro de la capa a `no_existe_en_la_base`.
- #450 (aprobado 29-oct): saca de la capa de Postgres los espejos `_sin_pii` de 22 fuentes cuyo cargador de Airbyte recrea la tabla con DROP (tipos de cambio de google_sheets, kyc, intercom, ai_whatsapp, facebook, julia_core, churnkey, exclusiones de leads). Plan de Joaquín: mergear antes del run-dbt de las 14:40 y borrar las 22 vistas a mano, sin CASCADE, antes de las syncs de 00:08, 00:31 y 06:31. No hay confirmación en Slack del merge ni del borrado.
- Aviso para la rama Redshift: `dbt-ci.yml` usa `ubuntu-latest` con Python 3.9 (sin soporte); fijar `ubuntu-24.04`.

Aprobación #450 y plan: https://agendapro.slack.com/archives/D095EKG3V37/p1793276108023609
Revisión #447/#448: https://agendapro.slack.com/archives/D095EKG3V37/p1793039911620969

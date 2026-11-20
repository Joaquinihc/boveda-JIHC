---
type: sugerencia
origen: slack
temas: [dwh, definiciones]
criterio: "(b) arreglo acordado en el DWH (permisos que dbt vuelve a dar en cada corrida) y backfill pendiente"
fuente: https://agendapro.slack.com/archives/D09C9SSG4HJ/p1794845931226879
canal: "DM con Israel Gutiérrez"
fecha_hilo: 2026-11-16
creado: 2026-11-20
revisar: false
---

# Slack — Clasificador IA de macro nicho (n8n) no carga desde marzo por permisos perdidos; ~10.600 empresas sin clasificar

- Joaquín detectó que el workflow de n8n «Classify Companies Macro Niche Using AI» (fuente IA de `company_attribute_macro_niche`, tabla `external_ai.company_macro_niche_ai`) no carga desde el 25-mar-2026: desde el 26-mar debieron clasificarse 10.652 empresas y solo aparecen 2; Redshift tiene ~1.180 empresas menos que Postgres. Sin esa carga, las empresas nuevas caen a HubSpot o al sector declarado.
- Israel (16-nov) confirmó la causa: `external_ai_user` perdió el permiso de lectura sobre `merchants_segments` y, en Redshift, sobre `stg_agendapro__services` y `stg_agendapro__bookings`, probablemente cuando dbt recreó esas tablas. Fix: que dbt vuelva a dar el permiso en cada corrida, más un grant manual para la corrida de esa noche.
- El fix no recupera las ~10.600 empresas saltadas (el workflow solo toma las que pagaron por primera vez hace exactamente 90 días): requiere un backfill aparte, con costo en OpenAI, a ver con Valeria Riquelme / Nokko. Sin confirmación en el DM de que la corrida del 16-nov clasificó.

DM: https://agendapro.slack.com/archives/D09C9SSG4HJ/p1794845931226879

---
type: sugerencia
origen: slack
temas: [dwh, evidence]
criterio: "(b) cambio de modelo/infraestructura del DWH acordado: Reservo entra a Redshift como esquema externo y tiene sitio Evidence propio"
fuente: https://agendapro.slack.com/archives/C0CA9BA5K45/p1792694285064049
canal: "#data-reservo (hilo de Pablo Lucero, respuestas de Israel Gutiérrez)"
fecha_hilo: 2026-11-02
creado: 2026-11-06
revisar: false
---

# Slack — DWH de Reservo ya se lee en Redshift (reservo_ext) y sus dashboards van en un Evidence aparte

- **Arquitectura acordada**: el DWH de Reservo (reservo_dw en Glue/Athena, 11 tablas) se lee desde Redshift como tablas externas sobre Parquet en `s3://ap-dwh-reservo` (schema `reservo_ext`), sin tocar Evidence; Joaquín apoyó importar a nuestro DWH. Validado el 2-nov con una foto manual: los números cuadran exacto con su DWH. Modelos dbt y consultas (comercial: ventas y demos; pagos) listos, a producción después del cierre de mes; actualización automática 4 veces al día.
- **Decisión de Israel (2-nov)**: los dashboards de Reservo viven en un sitio Evidence propio (`evidence-reservo.agendaprops.com`, repo `agendapro/bizops-evidence-reservo`), con deploy y accesos separados; reglas del repo: solo datos de Reservo, nada por cliente, páginas en inglés. Deploy apagado hasta tener control de acceso.
- **Pendientes**: Christopher Lagos debe pasar el ARN del rol de sus Lambdas y agregar la segunda escritura de Parquet; Israel le preguntó a Joaquín dos definiciones (si una demo «Llegó» cuenta como asistida y qué es `fecha_hasta` en fact_pagos, con pagos hasta el año 2200). El 6-nov Joaquín preguntó qué gana tener dos Evidence separados y si se puede consultar Reservo desde el Evidence de siempre: sin respuesta aún.
- Relacionada: [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]].

Hilo: https://agendapro.slack.com/archives/C0CA9BA5K45/p1792694285064049

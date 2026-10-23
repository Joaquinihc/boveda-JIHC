---
type: reunion
fecha: 2026-10-22
titulo: "Data AgendaPro — arquitectura de datos para Reservo"
personas: []
proyectos: []
temas: [dwh, definiciones, coordinacion]
fuente: https://app.notion.com/p/401ab34c233b80b1821cc27d9fff810f
verificado: false
---

# Data AgendaPro — arquitectura de datos para Reservo

**Acuerdos** (máx 5)
- Reunión técnica impulsada por Joaquín: Yonathan e Israel explicaron a Reservo (Christopher Lagos y Pablo Saintard) el stack de AgendaPro: ELT con capas medallion (bronce → plata → marts), Airbyte (más DMS y loaders en Python), Redshift, dbt (con masking), Prefect con alertas a Slack, Evidence y un MCP por el que ~90% de los usuarios consulta el DW con Claude.
- Funciona bien: Airbyte, permisos por esquema y onboarding/offboarding con VPN. A mejorar: observabilidad, fragilidad de dbt con más de 300 modelos, frescura de los datos y permisos aún manuales. Israel revisa todos los PR al DW; los pedidos entran por un tablero de Notion.
- Recomendación a Reservo: un DW acotado a indicadores de negocio; lo operacional se consulta en la réplica.
- Clasificación de comercios: sector económico (~40) → macro nicho (6) → vertical (4), con IA para reclasificar; Reservo tiene ~12 rubros. Hay que alinear rubros, macro nichos y verticales entre las dos empresas.
- Próximos pasos de otros: Israel documenta la infraestructura para Reservo; accesos a réplica y DW (con Sebastián Concha) y a los repos (Nicolás o Israel); Christopher y Pablo Saintard revisan el esquema del DW y marcan los indicadores que no se puedan construir.

**Mis acciones**
- [ ] Compartir con Reservo el Notion de definiciones de negocio (trabajado con Cristóbal)

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/401ab34c233b80b1821cc27d9fff810f).

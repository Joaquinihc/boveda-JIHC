---
type: borrador
origen: Claude Code
fecha: 2026-10-22
temas: [leads, definiciones]
revisar: true
---

# Claude Code — Lead: el board book cuenta reconversiones

- **Qué dice la skill** (`finance-metric-definitions`, card Lead, espejo en [[Lead]]): 1 lead = 1 contacto de HubSpot, contado por el mes de su primera conversión (`first_conversion_date`).
- **Qué hace el board book**: lee `dwh.fct_lead_conversions`, que tiene una fila por contacto y fecha de conversión (la creación del contacto más cada propiedad `reconversion{mes}{año}` de HubSpot desde feb-2025, contactos fusionados remapeados al ganador), y cuenta `COUNT(DISTINCT contact_id)` por mes. Un contacto puede ser lead en varios meses, nunca dos veces en el mismo. Flags `is_effective` e `is_budget_meta` encima.
- **De dónde salió**: repo `agendapro-dashboards-evidence`, rama `main`, `sources/dwh/marketing/leads.sql` y `leads_channel_brand.sql`; dbt `models/marts/marketing/fct_lead_conversions.sql` en `origin/refactor/postgres-to-redshift`. Revisado el 22-oct-2026 al redactar la hoja de Notion «Definiciones grupo: AgendaPro vs Reservo».
- **Por qué importa**: la card de la skill describe la lente de productividad por ejecutivo; el titular de leads del board book y la hoja compartida con Reservo usan la de reconversiones. Conviene actualizar la card (o dejar las dos lentes etiquetadas). Pendiente menor en [[Definiciones grupo AgendaPro vs Reservo]].

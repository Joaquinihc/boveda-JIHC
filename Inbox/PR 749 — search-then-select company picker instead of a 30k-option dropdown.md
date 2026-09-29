---
type: borrador-pr
repo: agendapro/agendapro-dashboards-evidence
pr: 749
url: https://github.com/agendapro/agendapro-dashboards-evidence/pull/749
autor: Joaquinihc
merge: 2026-09-22
temas: [margenes, evidence]
creado: 2026-09-23
revisar: true
---

# PR 749 — fix Company Margin: buscador + selector en vez de dropdown de 30k opciones

> Borrador cosechado por la tarea PRs (vie). Repo de tableros Evidence, rama `main`. Continúa el PR 748.

**Proyecto**: sin link (candidato: Márgenes por Cliente y Producto). `revisar: true`.

## Qué cambió
- El drill-down de `/finance/company-margin` pasa a dos pasos: un `TextInput` filtra por nombre (ILIKE) o prefijo de `company_id` hasta 50 coincidencias ordenadas por revenue 12 meses (top 50 si está vacío, respeta filtro de país); el `Dropdown` lista solo esas.
- Según el resumen del PR, además: KPIs por stream muestran siempre Software/Payments/AI (con ceros antes del lanzamiento), precisión consistente en montos y %, links a la metodología completa y etiquetas de gráficos rotadas.
- Parquets y queries de datos sin cambios.

## Por qué
- Abrir la página colgaba Chrome: el dropdown recibía ~30k compañías y Evidence las renderiza todas en el DOM. El commit se había empujado a la rama del 748 después del merge, por eso se re-aterriza aquí.

## Link
https://github.com/agendapro/agendapro-dashboards-evidence/pull/749

## Biblioteca que podría quedar desactualizada
- Ninguna (cambio de interfaz del tablero).

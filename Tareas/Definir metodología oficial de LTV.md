---
type: tarea
estado: pendiente
prioridad: media
tiempo:
tipo: definicion
origen: manual
proyecto:
temas:
  - definiciones
  - board
  - retencion
  - cac
fuente: https://agendapro.slack.com/archives/D0B83VD95JT/p1791492989057769
fecha_limite:
creado: 2026-10-19
revisar: false
id: T-095
---

# Definir metodología oficial de LTV

No hay LTV oficial en la empresa. Nace del cálculo del LTV self-serve para Emilio Latorre (8-oct): «Debiéramos de definir bien la metodología».

## Subtareas
- [ ] Calcular el LTV de B2B3 y del total con el mismo método (listo cuando: tabla de 6 a 60 meses por segmento, en revenue y margen bruto)
- [ ] Proponer la definición a [[Pablo Lucero]]: margen bruto, 5 años, por cohortes, por segmento, junto a LTV/CAC y payback (listo cuando: él la valide)
- [ ] Documentar la definición en `docs/METRIC_DEFINITIONS.md` del repo agendapro-dashboards-evidence

## Notas
- Análisis base: [[20261008-01 LTV self-serve por cohortes]]. Método y SQL: [[LTV por cohortes (nrr_cohorts) — método y trampas]].

### Pendientes menores
- 2026-10-19 · El CAC self-serve no está medido: el reporte Yield on CAC usa US$50 como supuesto (el P&L no separa la adquisición por segmento).
- 2026-10-19 · Verificar con Chargebee por qué la oferta de US$1 no aparece en las cohortes 2022–23 de `dwh.nrr_cohorts`.
- 2026-10-19 · El LTV del `ceo-dashboard` (borrador en Evidence) usa la fórmula clásica (ARPA × margen ÷ churn) y no separa por segmento.

## Historial
- 2026-10-19 · Creada desde Claude Code: pendiente de definir la metodología de LTV tras el cálculo self-serve.

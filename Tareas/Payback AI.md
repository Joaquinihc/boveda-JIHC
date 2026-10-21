---
type: tarea
estado: en-curso
prioridad: alta
tiempo:
tipo: analisis
origen: notion
proyecto:
temas:
  - ai
notion_id: 3ffab34c233b803fb437d66cbebc8bc9
notion_estado: In progress
notion_prioridad: High
notion_actualizado: 2026-10-20T14:03Z
fuente: https://app.notion.com/3ffab34c233b803fb437d66cbebc8bc9
fecha_limite:
creado: 2026-10-20
revisar: false
id: T-098
---

# Payback AI

Calcular el payback de los productos de inteligencia artificial (Julia, Agente AI, Sofía), separado del payback de adquisición (SaaS + POS ISR) y del de pagos a la base ([[Payback Payments]]). Quedó como prioridad en la FP&A Weekly del 20-oct-2026 junto con margen por producto, waterfall de one-time charges y payback de pagos.

## Subtareas
- [ ] 1. Revisar el draft de Pablo (PR #812, «AI Unit Economics by Vertical») y acordar con él si el payback AI parte de ese dashboard o se rehace por cohorte.

## Notas

### 2026-10-21 · Punto de partida
- **Lo que existe**: Pablo Lucero dejó el 14-oct un dashboard en draft, PR #812 del repo `agendapro-dashboards-evidence` (rama `feat/ai-unit-economics-by-vertical`, ruta `/finance/pl-by-vertical/unit-economics/`): CAC, Gross Margin, Activation Rate y Payback de IA por vertical. Al 21-oct sigue abierto.
  - CAC = payroll comercial de la vertical (KAM/AM + onboarding, cuentas G1121–G1123) ÷ todas las compañías nuevas con IA (`mrr_ai.new_companies_ai`). Solo existe desde jun-2026.
  - Gross Margin = el mismo modelo del AI P&L (`partials/ai-pl-base.md`).
  - Activación = % de nuevas con Julia/Agente AI que llegan a 15 reservas por IA en 30 días (Sofía no reserva).
  - Payback simple = CAC ÷ (ASP × GM), sin churn. Selector LM / L3M / YTD y gráficos mensuales desde marzo.
  - Quedó en draft porque faltaba el costo de IA de septiembre (facturas de modelos hasta agosto en la semilla `ai_cost_detail.csv`, `ai-pl-base.md` y `finance.ai_costs_manual_monthly`).
  - Puntos abiertos que dejó Pablo: (1) G1127 «Contractors Beauty» (~USD 5–8k/mes) fuera del CAC, ¿es costo comercial?; (2) payroll comercial de Health en agosto (USD 2,8k vs ~6–8k en meses vecinos), ¿es real?; (3) modo Allocated en mar–may reparte Azure por revenue y castiga a Health/Medspa.
- **Criterio de la reunión del 20-oct con Pablo** (aplicable también acá): payback estimado al inicio con supuestos vs payback real de la cohorte mes a mes, con dos versiones de CAC (con y sin sueldos).
- Relación con otras tareas: el costo de IA del P&L (reclasificación hacia atrás que Nicolás debe hacer desde septiembre) condiciona el margen que entra al denominador.

## Historial
- 2026-10-21 · Importada desde Notion (In progress)
- 2026-10-21 · Avance desde Claude Code: descripción, punto de partida (draft de Pablo, PR #812) y 1 subtarea.

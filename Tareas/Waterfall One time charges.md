---
type: tarea
id: T-042
estado: en-curso
prioridad: alta
tipo: tabla-dwh
origen: notion
proyecto: "[[_Waterfall One-Time Charges]]"
temas: [waterfall, otc]
notion_id: 394ab34c233b8097bd53f205989384b4
notion_estado: "In progress"
notion_prioridad: "High"
notion_actualizado: "2026-09-09T17:28Z"
fuente: https://app.notion.com/394ab34c233b8097bd53f205989384b4
creado: 2026-07-05
revisar: false
---

# Waterfall One time charges

Hay una propuesta de Claude pendiente de leer (artifact linkeado en Notion).

## Subtareas
- [x] Leer la propuesta de Claude
- [ ] Presentar la propuesta n−1 a Pablo Lucero y cerrar las decisiones 2 y 3 (reparto igual o por días; OTC manda o máximo)

## Notas

### 2026-10-19 · Propuesta al CFO (sesiones del 3, 21, 28 y 29-sep)
- Auditoría del flujo y benchmark de empresas públicas: [[20260903-01 Setup AI en el Waterfall — auditoría y benchmark]]. El 1,5x comercial no está en los datos; nadie suma lo único al ARR.
- Comercial cambió el modelo: el OTC se cobra al precio de la mensualidad por el período de instalación y luego entra el recurrente. Propuesta de reconocerlo como MRR con la regla n−1, ejecutada contra el DWH en solo lectura: [[20260928-01 OTC de AI como MRR — regla n−1]]. Método en [[Reconocimiento de OTC de AI — regla n−1]]. Artefacto para Pablo: https://claude.ai/code/artifact/LeuoFwcCdJwbkznx4LamXs (privado hasta compartirlo).
- Efecto: AI revenue publicado baja 6–14 % por mes (ago-26: 57.086 → 48.936 USD) porque deja de sumar caja y MRR forward-looking; waterfall AI con 35 `new` más y 33 compañías que hoy no existen.
- Vigencia de las invoices: 34 de 158 líneas mal cargadas en Chargebee: [[20260928-02 Auditoría de vigencia de las invoices OTC de AI]].
- La subtarea original "Definir implementación OTC separado del MRR" quedó superada: la propuesta vigente es OTC como MRR, no separado.

### Pendientes menores
- 2026-10-19 · Pedir a comercial corregir en Chargebee las 17 líneas sin período y las 13 parciales, y anular el duplicado de 461283 (ver auditoría).
- 2026-10-19 · `docs/AI_ONE_TIME_CHARGES_WATERFALL_PLAN.md` y `docs/superpowers/{specs,plans}/2026-09-28-otc-ai-regla-n-menos-1*` están solo en el working tree de `agendapro-dbt-redshift`, sin commit.
- 2026-10-19 · Decisiones 4 y 5 para Pablo: el P&L contable sigue con devengo diario; restatement desde marzo 2026 con tabla de diferencias antes del switch.
- 2026-10-19 · Si se implementa: cerrar primero el cambio de graduación que está en el working tree de `int_subscription_mrr_breakdown_changes.sql`.

## Historial
- 2026-09-22 · Importada desde Notion (In progress).
- 2026-10-19 · Avance desde Claude Code: propuesta n−1 al CFO documentada (3 análisis + metodología); subtarea de implementación reemplazada por presentar y cerrar decisiones.

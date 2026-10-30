---
type: sugerencia
origen: slack
temas: [pos, payments, definiciones]
criterio: "(a) el dueño de la SoT valida la definición de canal + compromiso (sin fecha)"
fuente: https://agendapro.slack.com/archives/D0BNJK4CXUN/p1792430096160429
canal: "DM con Ignacio Embry"
fecha_hilo: 2026-10-29
creado: 2026-10-30
revisar: false
---

# Slack — POS, canal por deal validado por Ignacio (New POS = venta de adquisición ISR) y deals ISR faltantes en México

- 29-oct · Ignacio revisó los PRs #832, #856 y #860: todo bien. Confirma que el mapeo por deal de `pos_sales_sot` (ISR → New, KAM → Stock, sin deal → Others) es la definición correcta. Ver [[PR 832 — POS sales, attachment and payback on dwh.pos_sales_sot (board book)]].
- Pide cambiar la nota del #856: **New POS = POS de la venta de adquisición (deal ISR)**, sin el argumento de los 30 días.
- Compromiso de Ignacio (sin fecha): crear los deals ISR que faltan; hoy esas ventas caen en Others, sobre todo México ene–abr. Cuando lo haga, suben las unidades ISR de México y cambia el payback.
- Pregunta abierta del mismo día (sin respuesta al 30-oct): si las 38 compañías `new` con canal Others en 2026 pasan por onboarding de POS y deben contar en la dimensión de onboarder de las cohortes de Evidence. https://agendapro.slack.com/archives/D0BNJK4CXUN/p1793280761856469

Hilo: https://agendapro.slack.com/archives/D0BNJK4CXUN/p1792430096160429
Respuesta de Ignacio: https://agendapro.slack.com/archives/D0BNJK4CXUN/p1793290688812839

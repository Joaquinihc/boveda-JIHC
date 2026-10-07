---
type: sugerencia
origen: slack
temas: [activacion, definiciones, evidence, okrs, board, fitness]
proyecto: "[[_Activación]]"
criterio: "(a) cambia cómo se mide + (c) conclusión que explica un número"
fuente: https://agendapro.slack.com/archives/D095G9DL97T/p1790648940336049
canal: "DM con Pablo Lucero"
fecha_hilo: 2026-09-28
creado: 2026-10-02
revisar: false
---

# Slack — Activation Velocity y tableros de activación con universo New Merchants (PR 769 y 772)

- Pablo Lucero cambió [[Activation Velocity]] tras conciliarla con el semanal de CX (Pablo Santa Inés), PR 769 de agendapro-dashboards-evidence (mergeado): universo = New Merchants B2B3 (`start_date_pago_mx`, primer mes pagado en `dwh.mrr`, sin cuentas test ni filtro contra `first_paying_date`), reservas + clases de fitness, y eje "Weeks since first payment". Se mantiene la ponderación por sedes; no se adopta la regla de cobros de CX.
- Explica el movimiento: antes entraban 8–21 empresas B2B3 por mes que nunca pagaron. B2B3 semana 9: jul-26 52,7% → 54,8%, abr-26 46,2% → 47,8%, ene-26 51,7% → 53,6%. **Se mueven las cifras de activación del OKR y del board book.**
- PR 772 (29-sep, abierto en ese momento) lleva las mismas reglas a Activation → Retention, Lead → Activación B2B3, PMF Fitness, R1 por país/persona y slide 2.3c del board book (re-expresa meses ya presentados). Activation → Retention baja de 60,8% a 47,3% porque entran 4.501 empresas que pagaron y nunca reservaron (61% B2C Colombia, lote de fraude de USD 1).
- Relación: el PR 415 de dbt (Inbox) también toca las cohortes de velocity (sin reservas importadas).

Hilo: https://agendapro.slack.com/archives/D095G9DL97T/p1790648940336049
PR 772: https://agendapro.slack.com/archives/D095G9DL97T/p1790707382286089

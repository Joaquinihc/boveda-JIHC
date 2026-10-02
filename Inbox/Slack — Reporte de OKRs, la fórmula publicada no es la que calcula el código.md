---
type: sugerencia
origen: slack
temas: [okrs, evidence, definiciones, reportes]
criterio: "(c) conclusión de análisis que explica un número + (a) cómo se mide"
fuente: https://agendapro.slack.com/archives/D0950MM8UM8/p1790683407080389
canal: "DM de Joaquín consigo mismo (nota de revisión)"
fecha_hilo: 2026-09-29
creado: 2026-10-02
revisar: false
---

# Slack — Reporte de OKRs, la fórmula publicada no es la que calcula el código

- Revisión del tablero de OKRs en Evidence (`pages/general/okr/`): `definitions.md` publica Progress % = (Current − Baseline)/(Target − Baseline), pero el código calcula real/target; el baseline no se usa. Con ARR en USD 14M, O1:KR1 ($17.8M) da 79% en el reporte y 42% con la fórmula publicada. Los KRs de techo también divergen (presupuesto de churn gastado justo: doc 0% rojo, reporte 100% verde).
- La cadencia "sin meta este mes" se infiere porque `evaluacion` llega NULL (casteada a numérico) en 2.076 de 2.076 filas; `unidad` también llega NULL, así que no se normaliza fracción → puntos (O4:KR10.3 ago-26: 0,87 vs meta 80 se lee como 1%).
- Otros hallazgos: el peso de cada KR depende de la granularidad (O1 pesa 52,9% de la nota de compañía, O3 6,6%); un cero en KR de techo puntúa 100%; redondeo antes del cap (99,5% pinta verde); botón "Pace mensual" documentado que no existe.
- Relación: el PR 413 de dbt (Inbox) deja `evaluacion` como texto, que es el arreglo de fondo del punto 5.

Hilo: https://agendapro.slack.com/archives/D0950MM8UM8/p1790683407080389

---
type: sugerencia
origen: slack
temas: [board, reportes, cierre-mensual]
criterio: "(a) cambia cómo se mide + (c) conclusión que explica un número"
fuente: https://agendapro.slack.com/archives/D095G9DL97T/p1790297286387219
canal: "DM con Pablo Lucero; aviso en DM con Rafael Rodriguez"
fecha_hilo: 2026-09-24
creado: 2026-09-25
revisar: false
---

# Slack — Cash Flow (Causal) pasa a tipo de cambio de cierre mensual

- Joaquín cambió la hoja *Exchange Rates* del GSheet "Cash flow (Causal)": deja de importar el tipo de cambio **promedio** del mes y pasa a importar el de **cierre** para convertir todas las monedas a USD. Afecta lo que se reporta al board, el board book y lo que se carga a Causal (avisado a Rafael Rodriguez).
- Causa del descuadre del Cash Flow por quarter: estaba a tasa promedio, pero el efecto tipo de cambio se calcula a tasa de cierre, así que no explicaba la diferencia entre períodos. A tasa de cierre cada quarter abre con el cierre del anterior y cuadra al dólar; la línea "FX effect on opening balance" pasa a "FX rate effect" con residual cero. Saldos finales se mueven menos de 1,1% vs lo enviado.
- Sigue abierta la diferencia con el Balance Sheet al 30-jun (BS USD 14,98M vs CF USD 13,37M, ≈USD 1,6M: fuentes distintas y faltan saldos de pasarelas). El 25-sep Joaquín propuso a Pablo y Nicolas Astudillo un plan (perímetro único de cash, remapeos en consolidación, control mensual); pendiente de decisión del CFO.
- Joaquín pidió a Pablo visto bueno sobre cómo explicarle a Mariana (Riverwood) la diferencia antes de enviar.

Hilo: https://agendapro.slack.com/archives/D095G9DL97T/p1790297286387219
Aviso del cambio: https://agendapro.slack.com/archives/D0950RBS4DD/p1790282745095159

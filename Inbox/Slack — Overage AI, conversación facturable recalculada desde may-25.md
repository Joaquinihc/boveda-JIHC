---
type: sugerencia
origen: slack
temas: [ai, definiciones, pricing]
criterio: "(a) define cómo se mide + (c) conclusión que explica un número"
fuente: https://agendapro.slack.com/archives/D0BB3GT39M1/p1790215303919049
canal: "DM con Rodrigo Novoa"
fecha_hilo: 2026-09-23
creado: 2026-09-25
revisar: false
---

# Slack — Overage AI, conversación facturable recalculada desde may-25

- Para estimar el overage no cobrado por cliente y mes, las **sesiones** no sirven: el cupo se consume en **conversaciones facturables** (ventana fija de 24 h desde el mensaje que la abre, al menos una respuesta de Julia, sin simulador). En sep-26 el total calza (25,6k facturables vs 22,3k sesiones, 47 clientes), pero cliente a cliente va de 0,2× a 5× (Fine Mens: 505 sesiones vs 1.798 facturables con cupo 300).
- Rodrigo Novoa recalculó las facturables hacia atrás con esa regla, por company_id y mes calendario (hora local), desde may-25. Validación vs conteo oficial de Cortex del ciclo en curso: 38.416 vs 37.713 en 93 companies (+1,9%). Antes de la migración al agente nuevo (hasta abr/may-26) los mensajes enviados por el comercio desde WhatsApp quedaban como de Julia: se excluyeron, esos meses son reconstrucción.
- Cupo: Notion primero, Cortex si no hay. Nailkery: se usa 3.000 (Notion, calza con lo cobrado ~US$936/mes); Cortex dice 10.000 sin respaldo → confirmar con Catalina. Tarifa por defecto = precio del plan ÷ conversaciones incluidas (reemplazable en hoja *Parámetros*).
- Salvedades: mes calendario (Cortex cuenta por ciclo), cupos vigentes hoy, septiembre parcial; en legacy el excedente puede estar sobrestimado. Entregó CSV + Excel v2 (24-sep).

Hilo: https://agendapro.slack.com/archives/D0BB3GT39M1/p1790215303919049

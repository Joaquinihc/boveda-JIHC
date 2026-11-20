---
type: reunion
fecha: 2026-11-17
titulo: "Monitoreo de gastos de API Keys de IA (Rodrigo)"
personas: ["[[Rodrigo Novoa]]"]
proyectos: []
temas: [ai]
fuente: https://app.notion.com/p/41bab34c233b80298e93f81417820008
verificado: false
---

# Monitoreo de gastos de API Keys de IA (Rodrigo)

**Acuerdos** (máx 5)
- Sabri quiere empezar ya el monitoreo de gastos de las API Keys, idealmente diario o semanal. Hoy Joaquín lo lleva mensual: descarga el resumen de cada key en las consolas de OpenAI, Claude y Azure, la atribuye a un equipo y carga la tabla en el DWH los primeros días de cada mes (esquema de finanzas, privado).
- Se parte con el reporte mensual: Joaquín le manda la tabla en CSV a [[Rodrigo Novoa]] (no cambia hasta el día 1) y Rodrigo trabaja con ella.
- Para un monitoreo más frecuente: exportar registros diarios desde las consolas o una tarea programada que los extraiga cada día; requiere permiso de Jonathan para entrar a las consolas. Rodrigo investiga cómo llegar a algo casi diario.
- Rodrigo tiene un monitoreo propio de tokens de input y output de Claude, pero es un estimado. La plataforma de monitoreo que hizo Jonathan tenía errores.
- Otras verticales: Carlos Izquierdo (Medspa) probablemente monitorea desde las consolas y optimizó costos de modelos; Rodrigo le pregunta. Health migró de Claude a OpenAI (unas 10 veces más barato) y quizás no lo ha trabajado.

**Mis acciones**
- [ ] Exportar a CSV la tabla mensual de consumo de API Keys y enviársela a Rodrigo Novoa

Sin dueño claro: pedirle a Jonathan permiso para acceder a las consolas de OpenAI, Claude y Azure y exportar los registros de forma periódica (el resumen de Notion se la asigna a Joaquín, pero en la transcripción el que necesita el permiso parece ser Rodrigo; Joaquín ya tiene acceso).

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/41bab34c233b80298e93f81417820008). Instant meeting sin título; la transcripción no nombra a los hablantes y se atribuyó según el resumen de Notion y el contexto. Dice «Cloud» (Claude), «Metspa» (Medspa) y «Happy Keys»/«apiquí» (API Keys). Una key de Beauty Onboarding Agent mostró cerca de un dólar de gasto; Joaquín dice que tendrá costo cero. Sabri, Jonathan y Carlos Izquierdo no tienen ficha. Relacionada: [[Pisar el P&L costos de API Keys, distribuir entre equipos]].

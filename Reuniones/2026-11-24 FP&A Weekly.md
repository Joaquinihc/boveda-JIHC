---
type: reunion
fecha: 2026-11-24
titulo: "FP&A Weekly"
personas: ["[[Ignacio Embry]]"]
proyectos: ["[[_Márgenes por Cliente y Producto]]", "[[_OKRs Finanzas]]"]
temas: [odoo, dwh, margenes, ai, okrs, board]
fuente: https://app.notion.com/p/422ab34c233b802da786d33f7b363c57
verificado: false
---

# FP&A Weekly

**Acuerdos** (máx 5)
- Prioridades de la semana, en orden: 1) cash flow, 2) métricas de Reservo (MRR primero), 3) lo demás, incluido el margen por cliente. Tomar un tema hasta terminarlo y llevar las prioridades en un solo tablero (puede ser el de Joaquín, con visibilidad de qué se toma y qué no).
- Cash flow en Odoo (~80–85 % listo; Nicole hace ajustes): no seguir cuadrándolo contra el de Evidence (planillas de Rafa); las diferencias relevantes saldrán en el entregable final. Siguiente paso: el agente exportador del repo de Odoo MCP; se exporta y se carga al DWH en una tabla nueva, en paralelo a la actual, a mano en cada cierre de mes. La automatización con Israel (Airbyte o tarea local) y una carga diaria para proyectar el mes quedan para después. Abierto: si se exporta consolidado o por país (Reservo entra al consolidado desde octubre).
- Margen por cliente: versión 1 en el esquema de Finanzas, documentada en Notion y con reporte en Evidence. Por ahora los planes van a SaaS y los add-ons de IA y el plan Julia a IA (se mantiene lo de las «10 lucas» para los add-ons del asistente). Los costos de plataforma de IA se vieron como poco relevantes por ser casi fijos; lo que agrega valor es separar costos variables por cliente (modelos de IA por conversación y Payments).
- Reservo: el MRR es la métrica principal. Chris compartió su repo (scripts de Python sobre la réplica) y Joaquín ya tiene acceso a la réplica para empezar los modelos. Interesa saber pronto la velocidad de migración de «Bambe» para ver si se llega a fin de año. Board Book: en Evidence, tres botones (AgendaPro, AgendaPro + Reservo, solo Reservo); qué se imprime en el PDF queda sin cierre.
- OKR: cerrados; se usarán en la evaluación de desempeño. Para el cierre se pide un PDF con comentarios por área. El OKR 2027 (definición y cálculo) lo lideraría Finanzas, después del presupuesto.

**Mis acciones**
- [ ] Trabajar el agente exportador del repo de Odoo MCP, exportar el cash flow y cargarlo a mano al DWH en una tabla nueva, sin borrar la actual
- [ ] Hablar con Christopher sobre dónde se registra el estado de suscripción de los clientes de Reservo, para armar los modelos sobre la réplica
- [ ] Pedir en el canal a Beauty que cargue al DWH la cantidad de conversaciones por cliente y por mes (Medspa y Health ya las tienen)
- [ ] Acordar con Jonathan los costos de plataforma de cada equipo vertical de IA (lo que muestra su plataforma no cuadra con su estimación)
- [ ] Agregar al reporte de margen por cliente el diagrama de cómo se consideran los costos (ya está en Notion)
- [ ] Sacar un PDF de los OKR con un párrafo y una lista de comentarios, para revisarlo e iterarlo

Sin dueño claro: preparar la comparación final entre el cash flow antiguo (Evidence, planillas de Rafa) y el de Odoo, con las diferencias explicadas, para entregarla a «Riverboot» (quien la pide dice que él ya hizo una versión que falta ordenar).

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/422ab34c233b802da786d33f7b363c57) (página sin nombre en Notion). La transcripción no nombra a los hablantes y Pablo Lucero no se nombra, por eso no se linkeó. Se interpretó «Hadoop» como Odoo, «EIDES/EDIOS/AIDES/Avions» como Evidence, «casco/Castro» como cash flow, «Cloda/club» como Claude y «Agenda Club» como AgendaPro; «Bambe» se dejó tal cual (el 17-nov se leyó como Vambe) y «Riverboot» no se identificó. «Jonathan/Jonas» no tiene ficha. Joaquín mencionó que tiene pendiente el payback por stream que planteó [[Ignacio Embry]] (payback de pagos). Relacionadas: [[Cash flow en Odoo]], [[Integrar los datos de revenue de Reservo (vistas AgendaPro, Reservo y combinada)]], [[Análisis margen por cliente y por producto]], [[OKR]], [[Payback Payments]].

---
type: reunion
fecha: 2026-11-24
titulo: "Bot de finanzas en Slack — demo, permisos y fuentes (Pablo y Nicolás)"
personas: ["[[Pablo Lucero]]"]
proyectos: []
temas: [ai, evidence, dwh]
fuente: https://app.notion.com/p/422ab34c233b80d1adb1ea029e38eaf6
---

# Bot de finanzas en Slack — demo, permisos y fuentes (Pablo y Nicolás)

**Acuerdos** (máx 5)
- Joaquín mostró el bot «finanzas» sobre su plataforma BizOps: registra conversaciones y herramientas llamadas, permite convertir una conversación en pedido a un proyecto (DBT y Evidence AgendaPro, cada uno con owner técnico y de negocio; Pablo es owner de negocio de Evidence). Joaquín lo estima «80 % listo»; falta que el equipo lo pruebe.
- Fuente de datos: el repo BizOps Finance MCP hoy solo lee los archivos de Evidence en producción, no el DWH. Criterio: meses cerrados → Evidence (ahí están las queries reales); mes en curso → datos frescos del DWH solo con queries específicas, sin acceso libre a Redshift para evitar cálculos inventados o distintos.
- Permisos: se filtran en el MCP, que recibe el correo de quien consulta; las reglas por usuario están por definir. Abierto: conectar Redshift en modo lectura y con qué restricciones.
- El bot solo interviene cuando lo mencionan en un hilo («está bien así»); funciona en canales públicos o privados donde esté la app, no en mensajes directos.
- Costos: API de OpenAI bajo el equipo de Joaquín, presupuesto de 300 con opción de bajarlo y tope por conversación. Una consulta tomó 26 iteraciones y 100 s: hay que optimizar.

**Mis acciones**
- [ ] Compartir con Pablo Lucero y Nicolás Cofre la URL de la plataforma, los repos (BizOps Finance MCP y DBT/Evidence) y la guía para actualizarlos y agregar herramientas
- [ ] Revisar cómo funciona la invocación del bot en grupos y confirmar que responda en canales públicos
- [ ] Dibujar cómo funciona el sistema de permisos (bot → MCP → Evidence)

Sin dueño claro: Pablo Lucero y Nicolás Cofre prueban el bot, enriquecen el repo Finance MCP (herramientas, definiciones, reglas de consulta) y reportan a Joaquín las fallas (p. ej., que no lea el hilo completo al ser mencionado).

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/422ab34c233b80d1adb1ea029e38eaf6) (página sin nombre; título de la nota: «Pablo / Nicolás»). Nicolás Cofre no tiene ficha en Personas. Se leyó «MSP/MSB» como MCP, «BISOPS» como BizOps y «Edians» como Evidence.

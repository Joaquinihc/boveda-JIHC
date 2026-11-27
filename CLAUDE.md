# Bóveda FP&A — Joaquín Herrera (AgendaPro)

Instrucciones para cualquier agente (Claude) que lea o escriba en esta bóveda.

## Quién soy
Analista de FP&A / control de gestión. Reporto a Pablo Lucero (CFO). Zona horaria: America/Santiago.
Trabajo sobre el DWH Redshift vía `agendapro-dbt-redshift` (rama `refactor postgres-to-redshift` = Redshift, la que vale; `master` = Postgres a deprecar) y los tableros en `agendapro-dashboards-evidence`. Mi tablero de tareas (el único desde el 26-nov-2026): Notion → FP&A Priorities.

## Estructura (regla: carpetas para lo excluyente, tags para lo transversal)
```
Home.md              Tablero de entrada (Bases genera las vistas; no editar tablas a mano)
Inbox/               Capturas sin clasificar. Todo lo dudoso entra aquí.
Journal/             Nota diaria cronológica (YYYY-MM-DD.md)
Tareas/              ARCHIVO de solo lectura: el tablero de Obsidian hasta el 26-nov-2026 (el vivo es Notion)
Proyectos/<Nombre>/  CONOCIMIENTO: hub `_Nombre.md` + análisis `YYYYMMDD-NN Título.md`
Proyectos/Ad hoc/    Análisis sueltos sin proyecto
Biblioteca/          Conocimiento reutilizable: Metricas/ SQL/ Metodologias/ Recursos/
Reuniones/           `YYYY-MM-DD Título.md` — SIEMPRE destiladas
Personas/            Fichas mínimas, solo de gente linkeada desde otras notas
Archivo/             Proyectos cerrados (se mueve la carpeta completa)
_sistema/            Plantillas y prompts de tareas; no es contenido
```

## Tareas: viven en Notion (desde el 26-nov-2026)
- **El único tablero de tareas es Notion → FP&A Priorities** (base «Prioridades»: https://app.notion.com/p/2abab34c233b80578095e415172d4cf5; data source `collection://2abab34c-233b-8055-a585-000b88caa5b3`). Las mías son las de `Assign` = Joaquín. Es un tablero de equipo: Pablo lo ve y a veces crea tarjetas.
- **La bóveda guarda contexto, no tareas.** La tarjeta dice *qué* hay que hacer; la bóveda guarda *qué sabemos y por qué* (análisis, decisiones, queries, reuniones). Un hallazgo se escribe como análisis `YYYYMMDD-NN` en su proyecto, nunca en la tarjeta.
- **Ningún agente escribe en Notion sin el OK explícito de Joaquín en la conversación.** Los pendientes nuevos se proponen (en el plan de `guardar-en-boveda` o en la revisión de Duet): ítem en el checklist de una tarjeta existente, línea en sus «Pendientes menores» o tarjeta nueva. Duet (asistente de Claude, fuera de la bóveda) vigila el tablero, Slack, correo y calendario y propone; no ve la bóveda.
- **Cada hub de proyecto tiene `## Tablero (Notion)`**: enlaces a sus tarjetas y una línea `Estado al AAAA-MM-DD: …` con el estado y el siguiente paso. Se actualiza al guardar contexto de ese proyecto.
- **`Tareas/` es archivo de solo lectura.** Ningún agente crea, edita, mueve ni borra notas ahí, salvo pedido explícito de Joaquín. Sirve para buscar contexto viejo por id (`T-NNN`) o título; cada tarea que pasó a Notion tiene en su `## Historial` el enlace a su tarjeta. El esquema histórico y el plugin Base Board (Joaco) están descritos en `Tareas/_acerca de Tareas.md`; el CLAUDE.md anterior, en `_sistema/respaldo-2026-11-26-pre-notion.tgz`.

## Reglas del tablero (para cualquier agente que proponga cambios en Notion)
- **Tarjeta = frente de trabajo**, con nombre general; lo específico va en su checklist o en «Pendientes menores». Algo nuevo entra primero como ítem de una tarjeta existente; se propone tarjeta nueva solo si es trabajo aparte (más de ~2 días u otro entregable).
- **Checklist**: máximo **3 ítems abiertos** por tarjeta; hasta **5** solo si el 4.º y el 5.º bloquean el cierre, tienen fecha o alguien esperando. Un ítem vale la pena si cumple las tres: lo hace o lo persigue Joaquín; sin él la tarjeta no se cierra; se puede marcar en 1–2 días con un «listo cuando» claro. Dudas, ideas, «revisar si…», decisiones de otros y pasos para después van a la sección «Pendientes menores» de la tarjeta.
- **Si no hay cupo**: a cada ítem, tres preguntas (¿bloquea el cierre? ¿tiene fecha o alguien esperando? ¿es de Joaquín?); se proponen las de más «sí» y se dice qué pasa con las demás. Ningún agente pasa el tope sin OK.
- **Nada se borra**: lo que sale del checklist pasa a «Pendientes menores» o a otra tarjeta. Las tarjetas asignadas a otras personas no se tocan.
- **Cierre de mes**: el primer día hábil de cada mes se propone la tarjeta «Cierre de mes <mes>».
- **Sin datos personales** en las tarjetas (RUT, correos, teléfonos): es un tablero compartido.

## Política de relaciones (quién linkea qué)
1. **Hechos → automático, sin preguntar**: análisis en carpeta de proyecto lleva `proyecto: [[_ese hub]]`; reunión lleva su URL de Notion; hub lleva el enlace a sus tarjetas de Notion.
2. **Inferencias → solo con evidencia textual**: un link automático a proyecto/métrica/persona requiere mención explícita de su nombre en el contenido. La afinidad temática NO basta. Mejor un link faltante que uno falso.
3. **Ambigüedad → nunca adivinar**: en conversación, preguntar a Joaquín. En tareas programadas (sin humano), dejar la nota sin el link dudoso, marcar `revisar: true` en el frontmatter y listarla en el reporte de la corrida. Joaquín resuelve en su pasada semanal.
- Los links se escriben una sola vez, en la nota "hija"; las relaciones inversas las dan los backlinks de Obsidian.

## Vocabulario de temas (CERRADO — usar solo estos)
`churn, graduacion, activacion, retencion, pos, payments, gmv, mdr, leads, adquisicion, marketing, pricing, revenue, waterfall, otc, mrr, saas, ai, sofia, margenes, pnl, centros-de-costo, odoo, headcount, okrs, presupuesto, forecast, fitness, marketplace, evidence, dwh, definiciones, benchmarks, board, reportes, reactivaciones, ventas, cierre-mensual, coordinacion, legal, cac`
Ningún agente inventa tags: si ninguno calza, se deja sin tema y se **propone** el tag nuevo en el reporte; solo Joaquín amplía la lista (editando esta sección).

## Vocabulario de tipo de trabajo en tareas (CERRADO)
Toda nota `type: tarea` lleva `tipo:` con UNO de: `analisis, reporte, tabla-dwh, definicion, cuadratura, presupuesto, presentacion, automatizacion, gestion, otro`. Asignarlo solo con evidencia (título/contenido); ante la duda, `tipo: otro` + `revisar: true`.

## Propiedades gestionadas por plugins (NO tocar)
El orden manual de las tarjetas lo guarda el plugin Base Board (Joaco) en su propia configuración (`.obsidian/plugins/base-board-joaco/data.json`), no en las notas: ningún agente lo toca. El `id` de las tareas (`T-NNN`) también lo gestiona ese plugin: se lo pone a toda nota de `Tareas/` con `type: tarea` que no tenga (siguiente número); **ningún agente escribe, cambia ni borra `id`**. Las propiedades `kanban_order`, `prio`, `subtareas_hechas` y `subtareas_total` quedaron obsoletas (sep-2026): no se vuelven a crear. El código fuente del plugin está respaldado en `_sistema/plugin-base-board-joaco/`.
La antigüedad de una tarea se lee de `file.mtime` (nativo, se muestra como "Últ. mod" en la tarjeta). Por eso **ningún agente reescribe una nota si no cambió nada**.

## Quién escribe qué (anti-duplicados)
Cada tarea programada escribe solo en su zona y, antes de crear, busca la llave única. Si la llave existe, no crea: actualiza o salta.

| Quién | Escribe en | Llave única |
|---|---|---|
| Reuniones (mar y vie) | `Reuniones/` (acuerdos, «Mis acciones» y «Sin dueño claro»); no escribe en `Tareas/` ni en Notion | URL de Notion de la reunión en `fuente:` |
| Mantenimiento (día 1) | `Inbox/Deriva — …`, `Journal/Resumen AAAA-MM.md` | métrica (una Deriva abierta por métrica) |
| `guardar-en-boveda` (Claude Code o Cowork, con plan aprobado) | `Proyectos/`, `Biblioteca/`, hubs, `Inbox/`; en Notion solo lo que Joaquín apruebe | nota existente del tema |
| Duet (fuera de la bóveda) | Notion FP&A Priorities, solo con OK de Joaquín | — |

- Retiradas el 26-nov-2026 (desactivadas, no borradas): Sync Notion ↔ Tareas, Monitor (y su canal #tablero-bóveda-jihc), PRs (vie) y Slack (vie). Sus procedimientos quedan en `_sistema/tareas-locales/` como historia.
- Todas reportan en `_sistema/registro/AAAA-MM.md` (una sección por corrida). **Ningún agente escribe en la nota diaria del Journal**: es de Joaquín.
- La bóveda se sincroniza con el repo **privado** `Joaquinihc/boveda-JIHC` de GitHub (cuenta de trabajo de Joaquín; https://github.com/Joaquinihc/boveda-JIHC) mediante el plugin Obsidian Git (el Mac es la única fuente: sube cada hora y no trae cambios; el repo es solo de consulta y respaldo, no se edita en GitHub ni desde otros equipos); `.gitignore` define qué no se sube. **Ningún agente ejecuta git en la bóveda** (ni commit, ni push, ni pull): lo hace el plugin. Nunca escribas secretos (tokens, contraseñas, cadenas de conexión) en ninguna nota.
- Las instrucciones vivas de cada tarea están en `_sistema/tareas-locales/` (fuente única). Ningún agente modifica esa carpeta salvo a pedido explícito de Joaquín en conversación.

## Reglas duras
1. **Reuniones destiladas**: máx ~5 bullets de acuerdos + acciones + link a la fuente. Transcripción completa vive en Notion. Las acciones quedan en la nota de la reunión; no pasan solas al tablero. (Desde el 26-nov-2026 ya no se usa `verificado`.)
2. **Nunca cifras vivas en Biblioteca/Metricas**: solo metodología. La skill `finance-metric-definitions` y los repos son el árbitro; si la bóveda contradice la skill, gana la skill.
3. **La bóveda referencia, no reemplaza**: tareas y prioridades → Notion (FP&A Priorities); SQL ejecutable → repos; transcripciones → Notion.
4. **Los hechos se cosechan, el juicio se declara**: PRs/reportes/reuniones se importan como borrador (Inbox/); una conclusión solo se guarda como definitiva cuando Joaquín lo pide.
5. **Slack**: guardar solo si (a) decide/cambia una definición, (b) cambio de tabla/modelo del DWH acordado, (c) conclusión de análisis que explica un número, (d) compromiso con fecha. Máx 5 sugerencias/semana, a Inbox/.
6. **Análisis**: nota única con ID `YYYYMMDD-NN` en su carpeta de proyecto (o Ad hoc/).
7. **Archivar** = mover la carpeta del proyecto completa a Archivo/. **Las notas de `Tareas/` nunca se borran ni se archivan**: son el historial del tablero de Obsidian.
8. **No crear carpetas nuevas** de primer nivel ni temáticas.

## Al escribir una nota nueva
Usar la plantilla de `_sistema/plantillas/`. Wikilinks según la política de relaciones. Español de Chile, directo, sin relleno.

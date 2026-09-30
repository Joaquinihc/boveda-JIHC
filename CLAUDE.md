# Bóveda FP&A — Joaquín Herrera (AgendaPro)

Instrucciones para cualquier agente (Claude) que lea o escriba en esta bóveda.

## Quién soy
Analista de FP&A / control de gestión. Reporto a Pablo Lucero (CFO). Zona horaria: America/Santiago.
Trabajo sobre el DWH Redshift vía `agendapro-dbt-redshift` (rama `refactor postgres-to-redshift` = Redshift, la que vale; `master` = Postgres a deprecar) y los tableros en `agendapro-dashboards-evidence`. Mi tablero de prioridades de equipo: Notion → FP&A Priorities.

## Estructura (regla: carpetas para lo excluyente, tags para lo transversal)
```
Home.md              Tablero de entrada (Bases genera las vistas; no editar tablas a mano)
Inbox/               Capturas sin clasificar. Todo lo dudoso entra aquí.
Journal/             Nota diaria cronológica (YYYY-MM-DD.md)
Tareas/              TRABAJO EN VUELO: una nota por tarea; estado en frontmatter; efímero
Proyectos/<Nombre>/  CONOCIMIENTO: hub `_Nombre.md` + análisis `YYYYMMDD-NN Título.md`
Proyectos/Ad hoc/    Análisis sueltos sin proyecto
Biblioteca/          Conocimiento reutilizable: Metricas/ SQL/ Metodologias/ Recursos/
Reuniones/           `YYYY-MM-DD Título.md` — SIEMPRE destiladas
Personas/            Fichas mínimas, solo de gente linkeada desde otras notas
Archivo/             Proyectos cerrados (se mueve la carpeta completa)
_sistema/            Plantillas y prompts de tareas; no es contenido
```

## Tareas (flujo) vs Proyectos (conocimiento)
- `Tareas/` = qué hay que hacer. `Proyectos/` = qué aprendimos/decidimos. **Una tarea nunca guarda conclusiones**: el hallazgo se escribe como análisis `YYYYMMDD-NN` en la carpeta del proyecto y la tarea solo lo linkea antes de cerrarse.
- Nota de tarea: frontmatter `type: tarea`, `id: T-NNN` (id estable; lo asigna solo el plugin del tablero, correlativo por fecha `creado`; nunca se cambia ni se reutiliza), `estado: propuesta|pendiente|en-curso|en-revision|bloqueada|hecho|descartada`, `prioridad: alta|media-alta|media|media-baja|baja` (o vacía si no tiene), `tiempo: 1-5` (opcional, tiempo estimado: 1 = 2 h o menos · 2 = medio día · 3 = 1 día · 4 = 2–3 días · 5 = 1 semana o más; lo pone solo Joaquín), `tipo:`, `origen: notion|reunion|manual`, `proyecto: [[_Hub]]` (opcional), `temas:`, `fuente:`, `fecha_limite:` (opcional), `creado:`, `revisar:`. Si viene de FP&A Priorities además: `notion_id:`, `notion_estado:` (último Status visto en Notion), `notion_prioridad:` (último Priority visto en Notion) y `notion_actualizado:` (última edición en Notion) — son la memoria del sync, solo el sync los escribe. Cuerpo: descripción + `## Subtareas` (checkboxes) + `## Notas` + `## Historial` (solo agregar líneas fechadas; lo escriben el sync, la tarea de reuniones y el Monitor al aplicar lo que pide Joaquín).
- **La completitud se marca SOLO en la tarea/tablero**, jamás en las notas de reunión. El sync refleja tablero → nota de origen (marca `- [x]` en "Mis acciones"), nunca al revés.
- Solo se traen de Notion las filas **asignadas a Joaquín o sin asignar**.
- Tareas con `notion_id`: Notion manda en el título. **Prioridad**: Notion trae `alta`/`media`/`baja` (High/Medium/Low); `media-alta` y `media-baja` los pone solo Joaquín a mano. El sync solo cambia la prioridad cuando cambia en Notion (compara contra `notion_prioridad`); ahí vuelve al valor de Notion y lo anota en `## Historial`. El `estado` se puede mover en el tablero: el sync solo lo cambia cuando Notion cambió de verdad (compara contra `notion_estado`) y lo anota en `## Historial`; si ambos cambiaron, no toca nada y marca `revisar: true`. Los títulos se comparan normalizados (sin puntuación, tildes ni mayúsculas). Si una fila desaparece de Notion, la nota se conserva con `revisar: true`. Cuando aquí pasa a `hecho`, el sync marca Done en Notion. Subtareas y notas personales NUNCA suben a Notion.
- Las tareas que nacen de reuniones (las crea un agente) entran en `estado: propuesta` y esperan que Joaquín las apruebe (le responde al Monitor en Slack en lenguaje natural, o las arrastra a `pendiente` en el tablero). `descartada` = rechazada: se oculta del tablero y **nunca se borra**.
- `revisar`: `false` en tareas creadas a mano (tuyas o de Notion) y en las aprobadas; `true` solo para dudas de clasificación y conflictos del sync.
- El tablero es `Tareas/Tablero de Tareas.base` (vista kanban del plugin **Base Board (Joaco)**, agrupada por `estado`; dentro de cada columna ordena por prioridad alta → media-alta → media → media-baja → baja → sin prioridad; dentro de la misma prioridad, por `tiempo` de más corta a más larga, sin estimar al final; después el orden manual). Cada tarjeta muestra un **número de orden** (1, 2, 3… recorriendo en-revision → en-curso → pendiente → bloqueada → propuesta; `hecho` no se numera): es solo visual, lo calcula el plugin y no se guarda en ninguna nota. Distinto del `id`, que sí es fijo.

## Política de relaciones (quién linkea qué)
1. **Hechos → automático, sin preguntar**: tarea extraída de reunión lleva `fuente: [[esa reunión]]`; tarea de Notion lleva `notion_id` + URL; análisis en carpeta de proyecto lleva `proyecto: [[_ese hub]]`; reunión lleva su URL de Notion.
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
| Sync Notion ↔ Tareas (L-V) | `Tareas/` (origen notion) + Done en Notion | `notion_id` (adopta una tarea de reunión equivalente en vez de duplicarla) |
| Reuniones (mar y vie) | `Reuniones/` + `Tareas/` (origen reunion, en `propuesta`) | reunión: URL de Notion en `fuente:`; tarea: misma acción ya existente en `Tareas/` (de cualquier origen) |
| PRs (vie) | `Inbox/PR …` | repo + número de PR |
| Slack (vie) | `Inbox/Slack — …` | permalink del hilo |
| Monitor (L-V 10:00) | Slack #tablero-bóveda-jihc (todo mensaje suyo empieza con `🤖 *Monitor bóveda* ·`); en `Tareas/` aplica lo que Joaquín pide en lenguaje natural en ese canal, solo con los cambios de su catálogo (ver su procedimiento): aplica lo claro, pregunta lo ambiguo y propone lo que pide criterio; nunca escribe en Notion, ni borra o renombra notas | mensaje de propuestas (su ts queda en el registro); un pedido se procesa una vez (queda con respuesta del Monitor) |
| Mantenimiento (día 1) | `Inbox/Deriva — …`, `Journal/Resumen AAAA-MM.md` | métrica (una Deriva abierta por métrica) |

- Todas reportan en `_sistema/registro/AAAA-MM.md` (una sección por corrida). **Ningún agente escribe en la nota diaria del Journal**: es de Joaquín.
- La bóveda se sincroniza con el repo **privado** `Joaquinihc/boveda-JIHC` de GitHub (cuenta de trabajo de Joaquín; https://github.com/Joaquinihc/boveda-JIHC) mediante el plugin Obsidian Git (el Mac es la única fuente: sube cada hora y no trae cambios; el repo es solo de consulta y respaldo, no se edita en GitHub ni desde otros equipos); `.gitignore` define qué no se sube. **Ningún agente ejecuta git en la bóveda** (ni commit, ni push, ni pull): lo hace el plugin. Nunca escribas secretos (tokens, contraseñas, cadenas de conexión) en ninguna nota.
- Las instrucciones vivas de cada tarea están en `_sistema/tareas-locales/` (fuente única). Ningún agente modifica esa carpeta salvo a pedido explícito de Joaquín en conversación.

## Reglas duras
1. **Reuniones destiladas**: máx ~5 bullets de acuerdos + acciones + link a la fuente. Transcripción completa vive en Notion. Importadas con `verificado: false` hasta que Joaquín las revise.
2. **Nunca cifras vivas en Biblioteca/Metricas**: solo metodología. La skill `finance-metric-definitions` y los repos son el árbitro; si la bóveda contradice la skill, gana la skill.
3. **La bóveda referencia, no reemplaza**: prioridades de equipo → Notion (FP&A Priorities); SQL ejecutable → repos; transcripciones → Notion.
4. **Los hechos se cosechan, el juicio se declara**: PRs/reportes/reuniones se importan como borrador (Inbox/); una conclusión solo se guarda como definitiva cuando Joaquín lo pide.
5. **Slack**: guardar solo si (a) decide/cambia una definición, (b) cambio de tabla/modelo del DWH acordado, (c) conclusión de análisis que explica un número, (d) compromiso con fecha. Máx 5 sugerencias/semana, a Inbox/.
6. **Análisis**: nota única con ID `YYYYMMDD-NN` en su carpeta de proyecto (o Ad hoc/).
7. **Archivar** = mover la carpeta del proyecto completa a Archivo/. **Las tareas `hecho` nunca se borran ni se archivan**: son el historial (en el tablero están todas, en la columna `hecho` colapsada; también en la vista de tabla Hecho).
8. **No crear carpetas nuevas** de primer nivel ni temáticas.

## Al escribir una nota nueva
Usar la plantilla de `_sistema/plantillas/`. Wikilinks según la política de relaciones. Español de Chile, directo, sin relleno.

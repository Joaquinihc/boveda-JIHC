---
type: recurso
---
# Tareas

**Trabajo en vuelo, no conocimiento.** Una nota por tarea; el estado vive en el frontmatter (`estado: propuesta | pendiente | en-curso | en-revision | bloqueada | hecho | descartada`) y se cambia arrastrando la tarjeta en el tablero (o editando la propiedad). Dentro de cada nota: descripción, `## Subtareas` (checkboxes, visibles y marcables en la tarjeta), `## Notas` libres y, en las que vienen de Notion, `## Historial` (lo escribe el sync).

Reglas:
- La completitud se marca **solo aquí** (tablero/nota), nunca en las notas de reunión.
- Una tarea nunca guarda conclusiones: el hallazgo se escribe como análisis en su proyecto y la tarea lo linkea antes de cerrarse.
- Tareas con `notion_id` vienen de FP&A Priorities (solo las tuyas o sin asignar). Notion manda en el título. La prioridad llega como alta/media/baja; tú puedes afinarla a media-alta o media-baja (o cambiarla) y el sync la respeta hasta que alguien la cambie en Notion. El estado lo puedes mover aquí y el sync lo respeta (solo lo cambia si Notion cambió de verdad, y lo anota con fecha en `## Historial`). Al pasar a `hecho` aquí, el sync marca Done allá. Subtareas y notas personales nunca suben a Notion.
- Tareas nacidas de reuniones llegan a la columna **propuesta**: apruébalas respondiendo al monitor en Slack (#tablero-bóveda-jihc) o arrastrándolas a `pendiente`. Las descartadas quedan en `descartada` (ocultas, no se borran; vista de tabla **Descartadas**).
- `revisar: true` = clasificación dudosa o conflicto del sync; revisar en la pasada semanal.
- Prioridad en 5 niveles: alta → media-alta → media → media-baja → baja. Sin prioridad = va al final de su columna.
- `tiempo` (opcional, lo pones tú): 1 = 2 h o menos · 2 = medio día · 3 = 1 día · 4 = 2–3 días · 5 = 1 semana o más. Dentro de la misma prioridad, las más cortas van primero y las sin estimar al final; después cuenta tu orden manual (arrastrar solo reordena entre tarjetas con la misma prioridad y tiempo).
- **Número de orden** (círculo en la tarjeta): 1, 2, 3… desde arriba de `en-revision`, siguiendo por `en-curso`, `pendiente`, `bloqueada` y `propuesta`. Es solo visual y se recalcula al mover tarjetas; no se guarda en las notas. Ignora el filtro de tipo (cada tarjeta mantiene su número del plan completo).
- **Id** (`T-042`, al pie de la tarjeta): fijo, por fecha de creación. Lo pone el plugin a las tareas nuevas; nunca cambia ni se reutiliza. Las subtareas son líneas dentro de una tarea y no tienen id.
- También puedes cambiar tareas desde Slack (#tablero-bóveda-jihc) escribiendo en lenguaje natural («aprueba la 1, la 2 va como subtarea de T-053, sube T-044 a alta»). El Monitor lo aplica en su corrida siguiente (L-V 10:00): hace lo claro, te pregunta lo ambiguo y, si pides que evalúe algo, te propone cambios y espera tu «ok». Sus mensajes empiezan con «🤖 Monitor bóveda ·». Pregúntale qué puede hacer para ver la lista.
- La columna `hecho` (colapsada) muestra todas las tareas cerradas; también están en la vista de tabla **Hecho**, ordenadas de la más reciente a la más antigua. Las tareas cerradas nunca se borran: son el historial.

El tablero es `Tablero de Tareas.base` (plugin **Base Board (Joaco)**). Detalles de instalación: `_sistema/Instalar tablero kanban.md`.

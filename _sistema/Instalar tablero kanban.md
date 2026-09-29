---
type: recurso
---
# Tablero kanban de tareas

La vista kanban de `Tareas/Tablero de Tareas.base` la dibuja el plugin propio **Base Board (Joaco)** (`.obsidian/plugins/base-board-joaco/`), un fork del plugin Base Board (MIT). No está en la tienda de Obsidian, así que ninguna actualización lo pisa.

- Activo: **Base Board (Joaco)**. El Base Board original debe quedar **desactivado** (registran la misma vista y no pueden convivir).
- **Task Count** ya no se usa (desinstalado el 23-sep-2026).
- Qué agrega el fork: subtareas de `## Subtareas` visibles y marcables en cada tarjeta; chip de `prioridad` coloreado (click derecho = elegir color); orden alta → media-alta → media → media-baja → baja → sin prioridad y, dentro de cada prioridad, por `tiempo` estimado (más cortas primero; v1.2.0); filtro fijo por `tipo`; "Últ. mod" al pie de cada tarjeta. Todo se configura en el menú de opciones de la vista (grupos Subtareas, Prioridad, Tiempo estimado, Numeración e id, Filtro fijo). Desde v1.3.0: número de orden visual en cada tarjeta e id estable (`T-NNN`) que el plugin asigna a las tareas nuevas.
- Columnas: pendiente, en-curso, en-revision, bloqueada, hecho (colapsada; todas las cerradas). Antes de estas va `propuesta` (tareas de reuniones por aprobar); `descartada` no se muestra.
- El orden manual (arrastrar dentro de una columna) se guarda en la configuración del plugin, no en las notas (v1.1.0): arrastrar no cambia la "Últ. mod" de ninguna tarjeta.
- Código fuente y cómo reconstruirlo: `_sistema/plugin-base-board-joaco/LEEME.md`.
- Si el plugin se desinstala, el `.base` y las notas siguen funcionando con las vistas de tabla.
- Recomendado: clic derecho sobre `Tablero de Tareas.base` → **Pin** para tenerlo siempre a mano.

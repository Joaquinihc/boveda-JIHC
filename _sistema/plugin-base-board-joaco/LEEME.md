---
type: recurso
---
# Código fuente del plugin Base Board (Joaco)

Respaldo del código del plugin propio que dibuja el tablero de `Tareas/`. El plugin instalado (lo que Obsidian ejecuta) está en `.obsidian/plugins/base-board-joaco/`; aquí está el **código fuente** para poder modificarlo en el futuro.

- `base-board-joaco-src.tgz`: repositorio completo (con historial git, sin `node_modules`). Rama `fork-joaco` sobre Base Board 2.5.1 de Michael DeRazon (licencia MIT, incluida en el paquete).
- Versión actual: **1.5.0** (7-oct-2026).

## Qué cambia respecto del original
- 1.0.0: subtareas de `## Subtareas` visibles y marcables en la tarjeta; chip de `prioridad` coloreado; orden por prioridad; filtro fijo por `tipo`; "Últ. mod" al pie.
- 1.1.0: el orden manual de las tarjetas se guarda en la configuración del plugin (`data.json`), **no en las notas**. Arrastrar una tarjeta ya no modifica ninguna nota (salvo el `estado` de la que cambia de columna), así la "Últ. mod" sigue siendo real.
- 1.2.0: 5 niveles de prioridad por defecto (alta → media-alta → media → media-baja → baja → sin prioridad) con colores para los dos nuevos; dentro de cada prioridad, orden por `tiempo` estimado (más cortas primero, sin estimar al final) y después el orden manual. Se configura en las opciones de la vista, grupo "Tiempo estimado".
- 1.3.0: número de orden visual en cada tarjeta (1, 2, 3… recorriendo en-revision → en-curso → pendiente → bloqueada → propuesta; global, ignora filtros; no se guarda en las notas) e id estable de la tarea (`id: T-NNN`) en el pie. El plugin asigna el id a las tareas de `Tareas/` que no tienen (siguiente número, por fecha `creado`); nunca cambia ni reutiliza uno. Opciones de la vista: grupo "Numeración e id"; ajustes del asignador en `data.json` → `taskIds`.
- 1.3.1: el asignador de ids solo corre en la app de escritorio (`taskIds.desktopOnly: true`), para que el celular no asigne números al mismo tiempo que el Mac cuando la bóveda se sincroniza con Git.
- 1.3.2: la vista se registra con un id propio, `tablero-joaquin`, y se llama **«Tablero Joaquín»** en el selector de diseños. Obsidian 1.14 (2-sep-2026) trajo un diseño «Kanban» nativo en Bases con el id `kanban`, el mismo que usaba el plugin, y lo tapaba: el tablero se veía con el kanban nativo. Por eso la vista del `.base` debe decir `type: tablero-joaquin` (si un `.base` dice `type: kanban`, abre el kanban nativo de Obsidian, no este plugin).
- 1.4.0: **columnas dobles**. Una columna configurada (en este tablero, `en-curso`) que tiene más de N tarjetas visibles (aquí 10) se ensancha y muestra las tarjetas en dos columnas, en zigzag (1 | 2, 3 | 4…). Con N o menos vuelve a una columna. Arrastrar funciona en las dos mitades. Se configura en las opciones de la vista, grupo "Columnas dobles" (`doubleColumns` y `doubleColumnsThreshold` en el `.base`). En el celular siempre es una columna.
- 1.4.1: el ancho de las columnas sale de la variable CSS `--base-board-column-width` (por defecto 280px). El snippet `.obsidian/snippets/tablero-ancho.css` la fija en 320px. La columna doble mide el doble (`2 × ancho − 10px`), así cada mitad conserva el ancho de una columna normal. Para cambiar el ancho, edita solo la variable del snippet.
- 1.5.0: la columna doble se arma como **mosaico**: cada tarjeta, en orden, va a la mitad que esté más corta, pegada a la de arriba, sin huecos verticales (antes era zigzag por filas y quedaban huecos bajo las tarjetas cortas). Se reacomoda sola cuando una tarjeta cambia de alto. Al arrastrar, la tarjeta queda antes de la tarjeta sobre cuya parte de arriba se suelta, en la mitad elegida; después se acomoda en el espacio libre.

## Cómo reconstruirlo
```bash
tar xzf base-board-joaco-src.tgz && cd base-board-joaco
npm install
npm test          # tests de subtareas, orden, tiempo, ids y numeración
npm run build     # genera main.js
```
Copiar `main.js`, `manifest.json` y `styles.css` a `.obsidian/plugins/base-board-joaco/` y recargar Obsidian.

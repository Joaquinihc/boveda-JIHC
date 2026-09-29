---
type: recurso
---
# Código fuente del plugin Base Board (Joaco)

Respaldo del código del plugin propio que dibuja el tablero de `Tareas/`. El plugin instalado (lo que Obsidian ejecuta) está en `.obsidian/plugins/base-board-joaco/`; aquí está el **código fuente** para poder modificarlo en el futuro.

- `base-board-joaco-src.tgz`: repositorio completo (con historial git, sin `node_modules`). Rama `fork-joaco` sobre Base Board 2.5.1 de Michael DeRazon (licencia MIT, incluida en el paquete).
- Versión actual: **1.3.1** (29-sep-2026).

## Qué cambia respecto del original
- 1.0.0: subtareas de `## Subtareas` visibles y marcables en la tarjeta; chip de `prioridad` coloreado; orden por prioridad; filtro fijo por `tipo`; "Últ. mod" al pie.
- 1.1.0: el orden manual de las tarjetas se guarda en la configuración del plugin (`data.json`), **no en las notas**. Arrastrar una tarjeta ya no modifica ninguna nota (salvo el `estado` de la que cambia de columna), así la "Últ. mod" sigue siendo real.
- 1.2.0: 5 niveles de prioridad por defecto (alta → media-alta → media → media-baja → baja → sin prioridad) con colores para los dos nuevos; dentro de cada prioridad, orden por `tiempo` estimado (más cortas primero, sin estimar al final) y después el orden manual. Se configura en las opciones de la vista, grupo "Tiempo estimado".
- 1.3.0: número de orden visual en cada tarjeta (1, 2, 3… recorriendo en-revision → en-curso → pendiente → bloqueada → propuesta; global, ignora filtros; no se guarda en las notas) e id estable de la tarea (`id: T-NNN`) en el pie. El plugin asigna el id a las tareas de `Tareas/` que no tienen (siguiente número, por fecha `creado`); nunca cambia ni reutiliza uno. Opciones de la vista: grupo "Numeración e id"; ajustes del asignador en `data.json` → `taskIds`.
- 1.3.1: el asignador de ids solo corre en la app de escritorio (`taskIds.desktopOnly: true`), para que el celular no asigne números al mismo tiempo que el Mac cuando la bóveda se sincroniza con Git.

## Cómo reconstruirlo
```bash
tar xzf base-board-joaco-src.tgz && cd base-board-joaco
npm install
npm test          # tests de subtareas, orden, tiempo, ids y numeración
npm run build     # genera main.js
```
Copiar `main.js`, `manifest.json` y `styles.css` a `.obsidian/plugins/base-board-joaco/` y recargar Obsidian.

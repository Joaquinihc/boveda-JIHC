# Tareas locales de la bóveda (v3 — fuente única)

**Estos archivos SON las instrucciones vivas.** Cada tarea programada (y la skill `/sync-tareas`) solo dice "lee y ejecuta el archivo X de esta carpeta". Para cambiar lo que hace una tarea, se edita su archivo aquí; no hace falta tocar la tarea programada. Ningún agente modifica esta carpeta salvo a pedido explícito de Joaquín en una conversación.

| Tarea | Frecuencia (hora Chile) | Archivo | Escribe en |
|---|---|---|---|
| Sync Notion ↔ Tareas | L-V 09:15 | `Tarea — Sync Notion-Tareas (diaria).md` | `Tareas/` (+ Done en Notion) |
| Reuniones → vault | mar y vie 09:40 | `Tarea — Reuniones (martes y viernes).md` | `Reuniones/`, `Tareas/` |
| PRs → borradores | vie 12:00 | `Tarea — PRs (viernes).md` | `Inbox/` |
| Slack → sugerencias | vie 12:20 | `Tarea — Slack (viernes).md` | `Inbox/` |
| Monitor | L-V 10:00 | `Tarea — Monitor (diaria).md` | Slack #tablero-bóveda-jihc, registro; en `Tareas/` solo lo que Joaquín pide en el canal (lenguaje natural, con catálogo cerrado de cambios) |
| Mantenimiento | día 1 10:00 | `Tarea — Mantenimiento (mensual).md` | `Inbox/`, `Journal/Resumen` (solo reporta) |

Requisitos para que corran: el Mac encendido, con internet y la app de Claude abierta a esa hora, y **la carpeta de la bóveda asociada a cada tarea programada** (si una tarea se recrea, hay que indicarle la carpeta; sin ella el agente no puede entrar y termina sin hacer nada). Los horarios se guardan en UTC: cuando Chile pase a horario de invierno (abril) correrán una hora antes y conviene moverlos.

Todas reportan en `_sistema/registro/AAAA-MM.md` (nunca en la nota diaria). Los horarios están escalonados para que nunca corran dos a la vez. Las llaves anti-duplicado están en `CLAUDE.md` → "Quién escribe qué".

Los archivos `Tarea — Ingesta semanal.md` y `Tarea — Mantenimiento mensual.md` son obsoletos (v1) y se pueden borrar.

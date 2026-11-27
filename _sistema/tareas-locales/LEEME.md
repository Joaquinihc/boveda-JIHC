# Tareas locales de la bóveda (v4 — fuente única)

**Estos archivos SON las instrucciones vivas.** Cada tarea programada solo dice "lee y ejecuta el archivo X de esta carpeta". Para cambiar lo que hace una tarea, se edita su archivo aquí; no hace falta tocar la tarea programada. Ningún agente modifica esta carpeta salvo a pedido explícito de Joaquín en una conversación.

Desde el 26-nov-2026 el tablero de tareas vive en Notion (FP&A Priorities) y lo vigila Duet. En la bóveda quedan solo dos tareas programadas, las que guardan contexto:

| Tarea | Frecuencia (hora Chile) | Archivo | Escribe en |
|---|---|---|---|
| Reuniones → vault | mar y vie 09:40 | `Tarea — Reuniones (martes y viernes).md` | `Reuniones/` (v5: no escribe en `Tareas/` ni en Notion) |
| Mantenimiento | día 1 10:00 | `Tarea — Mantenimiento (mensual).md` | `Inbox/`, `Journal/Resumen` (solo reporta) |

Retiradas el 26-nov-2026 (tareas programadas desactivadas, no borradas; sus archivos quedan como historia con un aviso al inicio): Sync Notion ↔ Tareas, Monitor, PRs (viernes) y Slack (viernes).

Requisitos para que corran: el Mac encendido, con internet y la app de Claude abierta a esa hora, y **la carpeta de la bóveda asociada a cada tarea programada** (si una tarea se recrea, hay que indicarle la carpeta). Los horarios se guardan en UTC: cuando Chile pase a horario de invierno (abril) correrán una hora antes y conviene moverlos.

Todas reportan en `_sistema/registro/AAAA-MM.md` (nunca en la nota diaria). Las llaves anti-duplicado están en `CLAUDE.md` → "Quién escribe qué".

Los archivos `Tarea — Ingesta semanal.md` y `Tarea — Mantenimiento mensual.md` son obsoletos (v1) y se pueden borrar.

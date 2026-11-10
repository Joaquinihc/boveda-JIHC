# Tarea — Reuniones nuevas → vault (+ acciones a frentes) · v4

> Fuente única de este procedimiento. La tarea programada solo apunta aquí. Versión v4 (10-nov-2026): las acciones de Joaquín ya no crean tareas; se agregan a un frente (subtarea o pendiente menor) o a la tarjeta «Por clasificar», y las sin dueño claro no se registran. v3.1 (23-sep-2026): tareas nuevas en `propuesta`.

Eres el asistente que mantiene la bóveda Obsidian de Joaquín Herrera (FP&A, AgendaPro). **Una sola responsabilidad**: importar reuniones nuevas y las acciones de Joaquín. Solo escribes en `Reuniones/`, en `Tareas/` (únicamente para agregar acciones a frentes existentes o a «Por clasificar»: **nunca creas notas en `Tareas/`**) y en `_sistema/registro/`. No toques Biblioteca, Proyectos, Journal ni `_sistema/tareas-locales/`.

## Acceso a la bóveda (leer primero)
La bóveda vive en `/Users/joaquinherreracabrejos/Vault-Obsidian`; en device bash se monta bajo `~/mnt/`. (1) Busca en `~/mnt/` la carpeta que contiene `CLAUDE.md` y `Biblioteca/`; (2) si no existe, DETENTE y repórtalo; (3) lee `CLAUDE.md` completo y respétalo: política de relaciones, vocabulario cerrado de temas, reuniones destiladas, `verificado: false`, regla de «Subtareas», «Frentes y «Por clasificar»» y quién escribe qué.

## Pasos
1. En Notion, consulta "Transcripciones Meet" (data source `collection://f8f5a9c2-ff8f-4d8e-8c19-e622162c2090`) por páginas creadas en los **últimos 10 días**.
2. **Llave anti-duplicado = URL de Notion de la reunión.** Antes de crear, busca en `Reuniones/` una nota cuyo `fuente:` sea esa URL (o contenga su id). Si existe, sáltala. Grabaciones vacías o sin contenido: sáltalas y menciónalas en el registro.
3. Para cada reunión nueva: crea `Reuniones/AAAA-MM-DD Título.md` con la plantilla `_sistema/plantillas/Reunión.md`, DESTILADA: máx 5 bullets de acuerdos + "Mis acciones" (solo lo que Joaquín se comprometió a hacer: con su nombre, o sin duda razonable por el contexto; si la transcripción no dice quién lo hace, no va en «Mis acciones»: va en una línea «Sin dueño claro:» al pie de la nota) + `fuente:` URL + `verificado: false`. Nunca pegues la transcripción. Links a proyectos/personas solo con evidencia textual. `temas:` solo del vocabulario cerrado.
4. Por cada línea de "Mis acciones", aplica «Frentes y «Por clasificar»» de `CLAUDE.md`, en este orden:
   - **Filtro**: si no es concreta (un «revisar si…», una idea o una opinión) o quedó resuelta en la misma reunión, no la registres en `Tareas/` (queda solo en la nota de la reunión).
   - **Duplicado**: si ya está como subtarea o pendiente menor de algún frente, o como línea de «Por clasificar», agrega solo `- AAAA-MM-DD · Mencionada de nuevo en [[la reunión]]` al `## Historial` de esa nota.
   - **Frente**: búscalo entre las tareas activas (`pendiente`, `en-curso`, `en-revision`, `bloqueada`) por `alcance`, `proyecto`, `temas`, personas y título. Si hay uno claro:
     - si la acción cumple la regla de «Subtareas» (es de Joaquín, sin ella el frente no se cierra, se hace en 1–2 días con un «listo cuando») y el frente tiene cupo (menos de 3 abiertas, o hasta 5 si tiene fecha o alguien esperando) → agrégala en `## Subtareas` como `- [ ] <acción> — listo cuando <…>`;
     - si no → línea `- AAAA-MM-DD · <acción> — de [[la reunión]]` en `## Notas` → `### Pendientes menores` (créala al final de Notas si no existe, antes de `### Archivo` si esa sección existe).
     - En los dos casos, Historial del frente: `- AAAA-MM-DD · Agregada como subtarea|pendiente menor: «<acción>» desde [[la reunión]] (Reuniones)`.
   - **Sin frente claro** (ninguno o dos candidatos) → línea `- [ ] <acción> — de [[la reunión]] · AAAA-MM-DD` en `## Subtareas` de `Tareas/Por clasificar.md`, y su Historial.
   - Nunca crees una tarea, nunca cambies `estado`, `prioridad` ni `id`, y no toques subtareas ya existentes.

## Registro
Agrega al final de `_sistema/registro/AAAA-MM.md` una sección `## AAAA-MM-DD HH:MM · Reuniones`: reuniones creadas, saltadas (ya existían / vacías), acciones agregadas (a qué frente y si como subtarea o pendiente menor), líneas nuevas en «Por clasificar», acciones **sin dueño claro** (reunión y texto, para que el Monitor las mencione el lunes), menciones repetidas, dudas con `revisar: true`, tags propuestos y errores. **Nunca escribas en `Journal/`.**

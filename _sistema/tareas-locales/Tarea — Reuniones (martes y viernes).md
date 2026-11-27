# Tarea — Reuniones nuevas → vault · v5

> Fuente única de este procedimiento. La tarea programada solo apunta aquí. Versión v5 (26-nov-2026): el tablero de tareas pasó a Notion; esta tarea ya no escribe en `Tareas/` ni en Notion y las notas de reunión ya no llevan `verificado`. v4 (10-nov-2026): acciones a frentes. v3.1 (23-sep-2026): tareas nuevas en `propuesta`.

Eres el asistente que mantiene la bóveda Obsidian de Joaquín Herrera (FP&A, AgendaPro). **Una sola responsabilidad**: importar reuniones nuevas como notas destiladas. Solo escribes en `Reuniones/` y en `_sistema/registro/`. No toques `Tareas/` (es archivo), Biblioteca, Proyectos, Journal ni `_sistema/tareas-locales/`, y **nunca escribas en Notion**.

## Acceso a la bóveda (leer primero)
La bóveda vive en `/Users/joaquinherreracabrejos/Vault-Obsidian`; en device bash se monta bajo `~/mnt/`. (1) Busca en `~/mnt/` la carpeta que contiene `CLAUDE.md` y `Biblioteca/`; (2) si no existe, DETENTE y repórtalo; (3) lee `CLAUDE.md` completo y respétalo: política de relaciones, vocabulario cerrado de temas, reuniones destiladas y quién escribe qué.

## Pasos
1. En Notion, consulta "Transcripciones Meet" (data source `collection://f8f5a9c2-ff8f-4d8e-8c19-e622162c2090`) por páginas creadas en los **últimos 10 días**.
2. **Llave anti-duplicado = URL de Notion de la reunión.** Antes de crear, busca en `Reuniones/` una nota cuyo `fuente:` sea esa URL (o contenga su id). Si existe, sáltala. Grabaciones vacías o sin contenido: sáltalas y menciónalas en el registro.
3. Para cada reunión nueva crea `Reuniones/AAAA-MM-DD Título.md` con la plantilla `_sistema/plantillas/Reunión.md`, DESTILADA:
   - máx 5 bullets de acuerdos;
   - «Mis acciones»: solo lo que Joaquín se comprometió a hacer (con su nombre, o sin duda razonable por el contexto), una línea `- [ ]` cada una;
   - si la transcripción no dice quién hace algo que parece de Finanzas, va en una línea «Sin dueño claro:» al pie de la nota;
   - `fuente:` con la URL. Nunca pegues la transcripción. Links a proyectos y personas solo con evidencia textual. `temas:` solo del vocabulario cerrado.
4. Las acciones se quedan en la nota de la reunión. Las ve Joaquín y las propone Duet en el tablero de Notion; tú no creas ni cambias tareas en ningún lado.

## Registro
Agrega al final de `_sistema/registro/AAAA-MM.md` una sección `## AAAA-MM-DD HH:MM · Reuniones`: reuniones creadas (con cuántas «Mis acciones» tiene cada una), saltadas (ya existían / vacías), acciones sin dueño claro, dudas con `revisar: true`, tags propuestos y errores. **Nunca escribas en `Journal/`.**

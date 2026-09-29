---
type: recurso
---

# Cómo funciona esta bóveda (versión humana)

**La regla de oro**: las carpetas dicen *qué tipo de cosa* es una nota o *a qué proyecto pertenece*. Los temas (churn, pos, fitness…) van como tags en el frontmatter, nunca como carpetas.

- Algo nuevo y dudoso → **Inbox/**. Una vez por semana se vacía.
- Lo que pasó hoy (contexto, decisiones chicas) → nota diaria en **Journal/**.
- Un análisis con nombre y fecha → carpeta de su proyecto en **Proyectos/**, con ID `YYYYMMDD-NN`. Sin proyecto → `Proyectos/Ad hoc/`.
- Cada proyecto tiene su hub `_Nombre.md`: objetivo, estado, próximos pasos. Es lo primero que se lee al retomar.
- **Biblioteca/** es lo reutilizable: fichas de métricas (metodología, sin cifras), SQL de referencia, metodologías, recursos.
- **Reuniones/** guarda solo el destilado (≤5 acuerdos + link a Notion). `verificado: false` = aún no revisé la transcripción.
- Proyecto terminado → arrastrar su carpeta a **Archivo/**. Listo.

- Algo **por hacer** → una nota en **Tareas/** (el tablero kanban es una vista de esa carpeta; arrastrar tarjeta = cambiar `estado` en la nota). Lo aprendido al hacerla → análisis en su proyecto. La completitud se marca solo en el tablero, nunca en las reuniones.

**Home.md** se arma solo con Bases: si el frontmatter está bien, el tablero está bien.

Los prompts de las tareas automáticas están en `_sistema/tareas-locales/` y las plantillas en `_sistema/plantillas/` (Ctrl/Cmd+P → *Templates: Insert template*).

---
type: tarea
estado: en-curso
prioridad: media
tiempo:
tipo: definicion
origen: manual
proyecto:
temas:
  - evidence
  - definiciones
fuente: https://claude.ai/artifact/As7D2PGNucDbVdp6foxiAz
fecha_limite:
creado: 2026-11-06
revisar: false
id: T-120
---

# Reorganizar la documentación del repo Evidence

Ordenar la documentación de `agendapro-dashboards-evidence` con una regla: cada tipo de conocimiento tiene un solo lugar. Así lo plantea la propuesta de Joaquín del 19-oct-2026 ([doc](https://claude.ai/artifact/As7D2PGNucDbVdp6foxiAz)). Hoy la documentación está repartida en 12 lugares, y en 3 de ellos solo la ve quien la escribió.

## Subtareas
- [ ] Cerrar con el equipo las 6 decisiones pendientes del doc: idioma, planes de Superpowers, día de la mudanza, diccionarios dbt, check de CI y responsable de la revisión mensual. Listo cuando las 6 tengan estado acordado en el doc.
- [ ] PR de la fase 1, con las reglas y el índice único de `docs/`. Listo cuando esté mergeado.
- [ ] PR de la mudanza: los 78 archivos a sus carpetas, con las rutas actualizadas (script de reemplazo y `git grep` sin rutas viejas). Listo cuando esté mergeado.

## Notas

### 2026-11-06 · La propuesta
- Doc: [Propuesta: reorganización de la documentación del repo Evidence](https://claude.ai/artifact/As7D2PGNucDbVdp6foxiAz), del 19-oct-2026. Secciones: cómo está hoy, problemas, estructura propuesta, cambios a realizar, agente docs-steward y decisiones pendientes.
- Estructura: ocho carpetas bajo `docs/`, con un índice único y una tabla que dice dónde va cada tipo de documento.
- Mudanza: 78 archivos commiteados, sin tocar dashboards ni queries. Unos 127 archivos citan esas rutas, en unos 380 lugares. El plan es de 4 fases, con un PR por fase.
- Quién decide, según el doc:
  - Idioma, día de la mudanza y responsable de la revisión mensual: todos.
  - Planes de Superpowers: Alex.
  - Diccionarios de dbt: Pablo con el equipo de data.
  - Check de CI: Israel.
- Relacionadas: [[Definiciones en MD dentro del repo de Evidence]] (T-027, antecedente: `docs/metrics`) · [[Apagar o enfriar reportes poco consultados en Evidence]] (T-110).

### Pendientes menores
- 2026-11-06 · Fases siguientes:
  - El agente `docs-steward`, que deja cada hallazgo en su lugar antes de cada PR.
  - El check de CI de ubicación de documentos: solo avisa el primer mes y después bloquea, si se aprueba la decisión 5.

## Historial
- 2026-11-06 · Creada desde Cowork, con la propuesta del 19-oct adjunta; en curso y con prioridad media (Joaquín en conversación).

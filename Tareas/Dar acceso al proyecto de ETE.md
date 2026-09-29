---
type: tarea
estado: hecho
prioridad:
tiempo:
tipo: gestion
origen: reunion
proyecto:
temas:
  - pos
fuente: "[[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]"
fecha_limite:
creado: 2026-09-24
revisar: false
id: T-073
---

# Dar acceso a Claude al proyecto dbt (ETE)

Acción de [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]. «ETE» era el proyecto dbt (transformación de datos): se trataba de darle a Claude acceso a ese contexto, para que pueda consultar el flujo de transformación y las tablas que creó Ignacio (aclarado por Joaquín el 25-sep).

Quedó resuelto así: Claude lee el repo `agendapro/agendapro-dbt` en GitHub con Code Explorer (en cualquier chat) y la copia local `~/Documents/GitHub/agendapro-dbt-redshift/` (el acceso a esa carpeta se da por sesión: en otro chat hay que volver a darlo). Las rutas quedaron en el hub [[_SoT Ventas POS]].

## Subtareas
- [ ]

## Notas

## Historial
- 2026-09-24 · Creada desde [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]] (propuesta, con duda)
- 2026-09-25 · Aclarada por Joaquín (Slack): «ETE» es el proyecto dbt; se trataba de dar acceso a Claude a ese contexto
- 2026-09-25 · Hecha: acceso de lectura vía Code Explorer (GitHub) y a la copia local del repo; rutas anotadas en el hub

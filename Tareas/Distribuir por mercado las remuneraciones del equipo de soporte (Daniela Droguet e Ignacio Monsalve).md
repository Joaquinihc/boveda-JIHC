---
type: tarea
estado: hecho
prioridad:
tiempo:
tipo: otro
origen: reunion
proyecto:
temas:
  - headcount
fuente: "[[2026-11-02 New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)]]"
fecha_limite:
creado: 2026-11-02
revisar: false
id: T-116
---

# Distribuir por mercado las remuneraciones del equipo de soporte (Daniela Droguet e Ignacio Monsalve)

Para contabilizar las remuneraciones según el mercado que atiende cada persona, Joaquín necesitaba saber a qué país atienden Daniela Droguet e Ignacio Monsalve, del equipo de soporte. Pablo cree que Daniela atiende Argentina e Ignacio México. Acción de [[2026-11-02 New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)]].

## Subtareas
- [ ] Asignar las remuneraciones de Daniela Droguet a Argentina y las de Ignacio Monsalve a México — listo cuando queden distribuidas en la planilla de remuneraciones

## Notas
- `revisar: true`: la reunión no dice si son personas de Reservo ni en qué planilla va la distribución; podría calzar como subtarea de T-112 (clasificar a las personas de Reservo en la planilla de headcount); se deja sin link hasta que Joaquín decida.
- 2026-11-09 · En el DWH ya quedó resuelto: `finance.headcount` clasifica a Daniela Droguett como Customer Support con BU Argentina (REV-SUPP-AR) y a Ignacio Monsalve con BU Mexico (REV-SUPP-MX); ambos códigos entraron al seed `headcount_cost_center_mapping` con el PR #482 de dbt. Ver [[Definir cómo clasificar a las personas de Reservo en la planilla de headcount]].

## Historial
- 2026-11-03 · Creada desde [[2026-11-02 New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)]] (propuesta)
- 2026-11-10 · Avance desde Claude Code: los dos quedaron en finance.headcount con BU Argentina y Mexico (PR #482 de dbt, 9-nov).
- 2026-11-10 · estado: propuesta → hecho. Resuelta en el DWH: `finance.headcount` deja a Daniela Droguett en BU Argentina y a Ignacio Monsalve en BU Mexico (dbt PR #482, 9-nov). (Joaquín en conversación)

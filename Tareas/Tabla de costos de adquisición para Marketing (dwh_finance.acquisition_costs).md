---
type: tarea
estado: en-curso
prioridad:
tiempo:
tipo: tabla-dwh
origen: manual
proyecto: "[[_P&L y Centros de Costo]]"
temas:
  - adquisicion
  - cac
  - dwh
  - marketing
fuente:
fecha_limite:
creado: 2026-11-26
revisar: false
alcance: "Tabla de costos de adquisición para Carlos Rojas (Marketing): modelo dbt finance.int_acquisition_costs_allocated y dwh_finance.acquisition_costs, regla de mínimo 2 personas por línea, permisos de Marketing sobre dwh_finance y la reunión de presupuesto de marketing"
id: T-128
---

# Tabla de costos de adquisición para Marketing (dwh_finance.acquisition_costs)

Facilitarle a Carlos Rojas (Marketing) una tabla de costos de adquisición en el DWH; es lo prioritario de lo que Joaquín conversa con Carlos. Cómo está armada: [[dwh_finance.acquisition_costs — modelo intermedio, regla de mínimo 2 personas y permisos]].

## Subtareas
- [ ] Decidir cuáles de las 3 mejoras se aplican (personas por línea, marca de reasignación, prueba unitaria en dbt) y abrir el PR de `feat/dwh-finance-acquisition-costs`. Listo cuando el PR esté abierto con las mejoras elegidas.
- [ ] Conseguir lectura de Carlos sobre `dwh_finance` (incluidas tablas futuras): pedido a Israel por DM el 26-nov 13:08. Listo cuando Israel dé el acceso o diga cómo se hace.

## Notas
### 2026-11-26 · Estado al cerrar el canal de trabajo
- Rama `feat/dwh-finance-acquisition-costs` en `agendapro-dbt` (commit `fd3b4c71`), reemplaza al PR #531 (cerrado). Trabajo técnico en la sesión de Claude Code «Tabla acquisition costs en Redshift».
- Revisión de la regla de mínimo 2 personas: [[20261126-02 Costos de adquisición — revisión de la regla de mínimo 2 personas por línea]].
- Reunión «Presupuesto marketing» con Carlos y [[Pablo Lucero]]: martes 1-dic, 17:00. Doc de preparación en Claude Docs (gasto de adquisición y CAC de octubre).

### Pendientes menores
- 2026-11-26 · Verificar que Buk (por BU) y el P&L (por centro de costo) coincidan persona a persona.
- 2026-11-26 · Con el cierre de noviembre, confirmar en real que Onboarding de Reservo (5 personas en Buk) vuelve a su propia línea.
- 2026-11-26 · Permisos: la propuesta de Pablo «Permisos de datos por perfil» sigue en estado propuesta (decisión de Pablo).

## Historial
- 2026-11-26 · Creada desde Cowork con el resumen del canal de trabajo de acquisition costs (Joaquín en conversación)

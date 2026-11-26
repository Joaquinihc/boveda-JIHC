---
type: recurso
temas: [adquisicion, cac, dwh, marketing, centros-de-costo]
proyecto: "[[_P&L y Centros de Costo]]"
autor: "[[Joaquín Herrera]]"
fecha: 2026-11-26
---

# dwh_finance.acquisition_costs — modelo intermedio, regla de mínimo 2 personas y permisos

**Propósito**: referencia de la tabla de costos de adquisición que se le prepara a Carlos Rojas (Marketing) en el DWH: dónde vive el código, qué hace cada capa y cómo se protege el sueldo de una persona. Estado al 26-nov-2026; la tabla **todavía no existe en el DWH** (rama sin PR). Seguimiento en [[Tabla de costos de adquisición para Marketing (dwh_finance.acquisition_costs)]]; revisión de la regla en [[20261126-02 Costos de adquisición — revisión de la regla de mínimo 2 personas por línea]].

## Código
- Repo `agendapro-dbt`, rama `feat/dwh-finance-acquisition-costs` (commit `fd3b4c71`). Reemplaza al PR #531, que se cerró.
- El trabajo técnico vive en la sesión de Claude Code «Tabla acquisition costs en Redshift».

## Dos capas
- **`finance.int_acquisition_costs_allocated`** (modelo intermedio, esquema restringido, no se publica): concentra toda la lógica (mínimo de 2 personas por línea, reparto fijo/variable, pagos por salida) y guarda cuántas personas hay detrás de cada línea de sueldos.
- **`dwh_finance.acquisition_costs`** (tabla publicada): el intermedio sin pagos por salida y sin número de personas.

## Regla de mínimo 2 personas por línea
- Cada mes, por separado, cuenta personas de Buk por línea (área + BU).
- Si una línea tiene menos de 2, su sueldo se suma a **Sales HQ** de la misma empresa. Si Sales HQ sigue bajo 2, todo Ventas de esa empresa va en una línea **Total**.
- La línea de destino nunca queda con 1 persona.
- **No es pegajosa**: el código no mira el mes anterior. Si una línea tiene 1 persona un mes (va a HQ) y 2 al siguiente, esas 2 vuelven juntas a su propia línea.
- No verificado: que Buk (por BU) y el P&L (por centro de costo) coincidan persona a persona.

## Decisiones
- 2026-11-26 · La diferencia de remuneraciones entre un mes y otro, que en teoría podría revelar un sueldo, **no es una preocupación**: se asume, porque si no habría demasiadas restricciones para reportar. Se descartaron una alerta mes a mes y agregar más gente a Reservo (decisión de Joaquín).

## Mejoras propuestas (pendiente decidir)
1. **Personas por línea**: publicar el número de personas en la tabla final, para costo por vendedor y CAC por país.
2. **Marca de reasignación**: columna sí/no cuando una línea incluye gente de otra (ej. Sales HQ de Reservo en oct-26 incluye a Onboarding).
3. **Prueba unitaria en dbt**: casos sintéticos (1 persona un mes, 2 al siguiente) para que la regla no se vuelva pegajosa.

## Permisos
- Propuesta de [[Pablo Lucero]] «Permisos de datos por perfil» (en estado propuesta al 26-nov): no abrir `finance`; usar `dwh_finance` como capa intermedia con tablas agregadas; permisos por grupo de perfil (Carlos en Dirección).
- 2026-11-26 13:08 · Joaquín le pidió a Israel (Data Engineer) por DM en Slack lectura de Carlos sobre `dwh_finance`, cubriendo también tablas futuras. Sin respuesta al 26-nov.

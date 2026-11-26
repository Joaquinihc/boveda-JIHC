---
type: analisis
proyecto: "[[_P&L y Centros de Costo]]"
estado: activo
fecha: 2026-11-26
temas: [adquisicion, cac, centros-de-costo, headcount]
metricas: []
---

# Costos de adquisición — revisión de la regla de mínimo 2 personas por línea

**Pregunta**: ¿La regla de mínimo 2 personas por línea de `dwh_finance.acquisition_costs` evita que se vea el sueldo de una persona, sin dejar a nadie solo ni quedarse pegada?

## Conclusión
> - **Nadie queda solo.** En AgendaPro ninguna línea baja de 2 personas entre nov-2025 y oct-2026. Único caso: Onboarding de Reservo en oct-2026 (1 persona), que pasa a Sales HQ de Reservo.
> - **La regla no es pegajosa**: cada mes se evalúa por separado. Probado con datos sintéticos; con el cierre de noviembre debería verse en real, porque Onboarding de Reservo ya tiene 5 personas en Buk.
> - **La diferencia de sueldos entre meses se acepta** (decisión de Joaquín, 26-nov): no se agrega alerta mes a mes ni más gente a Reservo.

## Método
- Consultas de solo lectura contra Buk y el P&L, periodo nov-2025 a oct-2026 (la tabla aún no existe en el DWH). Reglas del modelo en [[dwh_finance.acquisition_costs — modelo intermedio, regla de mínimo 2 personas y permisos]].
- Prueba de «no pegajosa» con datos sintéticos (1 persona un mes, 2 al siguiente).

## Evidencia
- Conteo mensual por línea (área + BU) en Buk: AgendaPro sin líneas bajo 2; Reservo con Onboarding en 1 en oct-2026.
- Destino: la línea receptora nunca queda con 1; si Sales HQ no llega a 2, todo Ventas de la empresa se junta en Total.

## Caveats
- No verificado que Buk (por BU) y el P&L (por centro de costo) coincidan persona a persona.
- Fuente: resumen del canal de trabajo del 26-nov-2026 (las queries viven en la sesión de Claude Code «Tabla acquisition costs en Redshift», no en el repo).

---
type: tarea
estado: en-curso
prioridad: media
tiempo:
tipo: definicion
origen: manual
proyecto: "[[_Activación]]"
temas:
  - activacion
  - definiciones
fuente:
fecha_limite:
creado: 2026-10-07
revisar: false
id: T-089
---

# Arreglar definición de activación

Revisar y corregir cómo se define y se mide la activación, para que las métricas que la usan (como Activation Velocity) cuenten bien los bookings y los enrollments.

## Subtareas
- [ ] Agregar Activation Velocity con enrollments al tablero de fitness
- [ ] Mergear el PR 415 de dbt y verificarlo después de la corrida nocturna
- [ ] Decidir si la activación cuenta solo bookings activos o todos los estados
- [ ] Llevar a Evidence la exclusión de reservas importadas (camino A o B)
- [ ] Dejar una sola definición oficial de activación (Evidence o dbt)
- [ ] Unificar la definición de enrollments (fecha, estados y orígenes)
- [ ] Definir qué hacer con el fraude de Colombia (USD 1) en Activation → Retention
- [ ] Confirmar con CX la ponderación (sedes o cuentas) y la regla de cobros
- [ ] Decidir la re-expresión de meses ya presentados y avisarle a [[Pablo Santa Inés]]
- [ ] Actualizar la tarjeta de `finance-metric-definitions` y después las fichas [[Activation]] y [[Activation Velocity]]
- [ ] Pedirle a Producto un origen propio para las importaciones (`csv_import`)
- [ ] Calcular el impacto de las importadas en las cohortes 2025
- [ ] Publicar el impacto de las importadas en el hilo de Slack de Otros LATAM

## Notas
- Subtarea «Agregar Activation Velocity…»: agregar al tablero de fitness la métrica Activation Velocity (% de clientes que llegan a 100 bookings desde que pagan) contando todos los enrollments, igual que la de la sección de Customer Experience. Meta mencionada: sobre 60%. Venía de la propuesta [[Agregar Activation Velocity con enrollments al tablero de fitness]] (T-082), de [[2026-09-29 FP&A Weekly]].
- Relacionadas: [[Bookings vs enrollments]] y [[Dashboard de bookings y enrollments desde cero]].

### 2026-10-07 · Estado de la definición (desde Claude Code)
- Análisis: [[20260929-01 Validación PR 769 y fuentes de activación]] · [[20260930-01 Reservas importadas — impacto en activación y waterfalls]] · [[20261001-01 Activation Velocity — Evidence vs tablas dbt]]. Referencia: [[Activación — fuentes y definiciones vigentes]] y [[SQL — Activación (importadas y puente Evidence vs dbt)]].
- Hecho en dbt (`agendapro-dbt`, rama `refactor/postgres-to-redshift`): PR 412 (velocity en la corrida diaria) y PR 414 (`bookings_count_demand`, con full refresh el 30-sep), los dos mergeados. El PR 415 (activación y velocity sin importadas) está abierto, con el CI en verde: https://github.com/agendapro/agendapro-dbt/pull/415
- **Fitness**: [[Pablo Lucero]] ya agregó la curva de Activation Velocity de Fitness en el PR 774 de Evidence (29-sep), en `/acquisition/pmf-fitness/` §3. Por defecto cuenta **todas** las inscripciones (selector `enrollment_scope`: `all` o `valid` = criterio CX). Revisar si eso ya cubre la primera subtarea.
- **Caminos para la exclusión de importadas en Evidence**. A: agregar al fact un contador de bookings activos sin importadas y mover las extracciones a ese contador. B: migrar Evidence a las tablas dbt de velocity. B depende de la decisión "activos o todos" y del universo (ver 20261001-01).

## Historial
- 2026-10-07 · Creada por Joaquín en conversación, en curso con prioridad media, como tarea madre de «Agregar Activation Velocity con enrollments al tablero de fitness»
- 2026-10-07 · Avance desde Claude Code: 12 subtareas nuevas con los pendientes de la definición de activación; enlaza los análisis 20260929-01, 20260930-01 y 20261001-01
- 2026-10-07 · Asignada al proyecto [[_Activación]] desde Claude Code, a pedido de Joaquín

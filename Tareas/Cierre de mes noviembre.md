---
type: tarea
estado: en-curso
prioridad: media
tiempo:
tipo: gestion
origen: manual
proyecto: "[[_P&L y Centros de Costo]]"
temas:
  - cierre-mensual
  - pnl
  - headcount
  - centros-de-costo
fuente:
fecha_limite:
creado: 2026-11-18
revisar: false
alcance: "Todo lo del cierre de noviembre 2026: cambios de CECOs desde la nómina de noviembre (Buk, Headcount - Raw, seed de dbt, Odoo), P&L y headcount de Reservo del mes, métricas que se reportan; el cash flow del cierre va en T-100"
id: T-126
---

# Cierre de mes noviembre

Lo que falta para cerrar noviembre de 2026. Nace con el frente de centros de costo: los cambios acordados con [[Pablo Lucero]] el 11-nov rigen desde la nómina de noviembre y el registro de cada cambio vive en Notion, en [Cambios CECOS noviembre 2026](https://app.notion.com/p/415ab34c233b8099855bcc149329b18f) (lo que se aplica) y [Cambios CECOS hasta octubre 2026](https://app.notion.com/p/41aab34c233b8140b176f7385959b4c9) (propuestas en espera, incluida la migración de códigos legacy del seed).

## Subtareas
- [ ] 1. Cambios de CECOs desde noviembre 2026 (Buk, headcount y seed). Listo cuando el PR #507 esté mergeado (hecho el 18-nov), la carga de diciembre traiga los ajustes pedidos a Gabriela y Nicolás tenga definido el reparto de Reservo (REV-MDR, REV-AE, G1107).
- [ ] 2. Contabilización de Ruelas (primer contratado de Reservo en México, pagado por AgendaPro México). Listo cuando Nicolás elija el mecanismo (analítica con marca Reservo en AgendaPro México, refacturación intercompañía o cuenta aparte) y quede escrito en Reglas de asignación de CECOS.

## Notas

### 1. Cambios de CECOs desde noviembre 2026 (Buk, headcount y seed)
Tres cosas, en este orden:
- **Seed y DWH**: mergear el [PR #507](https://github.com/agendapro/agendapro-dbt/pull/507) (`agendapro/agendapro-dbt`, rama `feat/headcount-ceco-reservo-mdr-ae`, contra `refactor/postgres-to-redshift`) y volver a correr `dbt seed -s headcount_cost_center_mapping --target redshift` y `dbt run -s headcount --target redshift` desde un worktree en la punta de origin. El seed ya está cargado en Redshift desde la rama (17-nov); el merge solo lo fija.
- **Buk, para la carga de diciembre** (pedido a Gabriela Ríos por DM el 17-nov, borrador): Ángel Díaz vuelve a PAY-OPS-CL y Payments Ops (sigue como Ops Analyst; la carga de noviembre lo trajo en PAY-EXPA-CL por error); Guillermo Romero Figueredo es baja, con fila de noviembre en CL Data y fila en Inactive Headcount; Ruelas va solo en la pestaña Reservo CL con MXN y REV-AE-MX; Aucan Sierra a PAY-OPS-CL con cargo Payments Consultant Junior cuando lo confirme [[Pablo Santa Inés]] (Tábata lo ubica en Payments Ops de soporte para Chile desde el 2-nov).
- **Odoo, con [[Nicolás Astudillo]]**: crear REV-MDR y REV-AE en el plan Centros de SC3 SpA y SOFTWARE RESERVO (no existían al 16-nov); las remuneraciones de MDR y AE de Reservo van a G1107 (en octubre quedaron en REV-ISR, G1108); y definir un centro de costo en la sociedad AgendaPro México con marca Reservo para Ruelas, personal de Reservo que se formaliza por AgendaPro México porque SOFTWARE RESERVO no tiene nómina ahí. Es una excepción al sistema: la sociedad que paga no es la dueña del costo.

### 2026-11-17 · Carga de noviembre de Headcount - Raw revisada contra los cambios acordados
- `finance.raw__headcount` de noviembre: 208 filas de AgendaPro y 52 de Reservo (llegó el 17-nov 13:00). `finance.headcount` se reconstruyó a mano desde el worktree `agendapro-worktrees/dbt-headcount-rebuild`.
- Aplicado en Buk (✅ en Notion): Janet Lara, Silcris Jaramillo, Karla Lucero, Carla Revello, Sabrina Orellana y Catalina Orbez (CECO, área y cargo); todo Reservo (MDR, AE, analistas de implementación con BU por país, Abadía y Piñeiro, Contreras y Cruzat), cargado por Gabriela desde las filas que FP&A dejó en Reservo CL.
- Pendiente (🔴 en Notion): Ángel Díaz (People lo movió a PAY-EXPA-CL; se queda en PAY-OPS-CL), Aucan Sierra (Buk lo bajó a Customer Support Trainee en REV-SUPP-HQ), Romero Figueredo (baja sin registrar), Ruelas (aún no entra a la carga). Saintard queda 🟡: solo en ADM-HQ dentro de Reservo; parche posible, Christopher Lagos (BizOps Analyst) a ADM-HQ, con cambio de línea de Development a Administration; decisión de Pablo.
- Hallazgos de la carga: la recarga de People borró las fórmulas J:L (Country, Gender, Status) de Consolidated data en las filas 7623–7882 (ARS desde mayo y Reservo), lo que dejaba a Argentina sin activos desde mayo; Joaquín las extendió el 17-nov y el DWH volvió a 0 filas sin status. Los 10 MDR y AE de Reservo llegaron sin categoría de P&L hasta que el seed tuvo los 4 códigos nuevos. Los espejos de Buk MX y CO (Portugueis, Santa Inés, Lucero Ramos, Cofré) y el RUT de Federico Alonso quedaron 🟡 para la carga de diciembre.
- Cómo operar la carga y qué se aprendió: [[finance.headcount — employer, clave de persona y carga desde la planilla]]. Decisiones de clasificación: [[Guía clasificación de personas (headcount)]], §8.

### 2. Contabilización de Ruelas
Se le paga por AgendaPro México (Buk México AgendaPro, CECO REV-AE-MX) pero el costo es de Reservo; Odoo y la Master separan AgendaPro de Reservo por sociedad, así que su sueldo caería en el P&L de AgendaPro. Borrador del 19-nov en el grupo de People y Finanzas con tres opciones: (1) analítica nueva en el plan Centros de AgendaPro México con marca Reservo (p. ej. RSV-REV-AE-MX), la recomendada; (2) refacturación intercompañía mensual a SOFTWARE RESERVO; (3) cuenta de remuneraciones aparte. [[Nicolás Astudillo]] no está en ese grupo: hay que agregarlo o abrir uno nuevo antes de mandar. Antecedentes: Francisca preguntó el 10-nov cómo se sabría que entra a costos de Reservo con el mismo CECO; [[Pablo Lucero]] respondió el 11-nov «sí, CECO Reservo».

### 2026-11-19 · Planilla corregida y altas de Reservo
- Gabriela corrigió Headcount - Raw el 19-nov (avisó a las 11:19 y volvió a tocar después de las 14:00). Verificado contra el DWH (extracción 14:01 Chile) y la planilla: Ángel Díaz en PAY-OPS-CL / Payments Ops; Romero Figueredo con fila de noviembre Inactive y fila en Inactive Headcount (2026-11-01); Portugueis MX en PAY-EXPA-CL; Cofré CO en Bizops Product; Aucan Sierra en PAY-OPS-CL, Payments Ops, Payments Consultant Junior (confirmado por [[Pablo Santa Inés]] el 18-nov). Solo cambio cosmético adicional: los 6 ISR de Colombia pasaron a «ISR Inside Sales Representative».
- Altas de Reservo en Reservo CL (filas 106 y 107, fecha 2026-11-01) y en Consolidated data (7884 y 7885), con Country, Gender y Status: Benjamín Mauricio Mancilla Vera, Desarrollador, P&T-TECH-HQ, CLP; Daniel Ruelas Molinas, Ejecutivo comercial, REV-AE-MX, MXN. Noviembre queda en 263 filas (209 AgendaPro, 54 Reservo). Decisión de Joaquín: Ruelas se queda en Reservo CL y no en MX Data mientras su CECO sea de México.
- Espejos de Pablo Santa Inés (MX y CO) y Pablo Lucero Ramos (MX): no vienen en Headcount - Raw desde octubre (solo las filas de Chile); preguntado a Gabriela, sin respuesta.
- Federico Alonso: su número de documento no es un RUT válido, pero no importa: `finance.raw__headcount` no trae la columna Tax Id y la persona se identifica por nombre y empleador; en la planilla solo sirve para la fórmula de Status. Cerrado sin acción.
- Mensajes: a Gabriela enviado (Aucan, altas, espejos, Alonso); a [[Pablo Lucero]] en borrador (si sigue la regla «nadie solo en un CECO dentro de una sociedad», cómo escribirla en Reglas, y el parche de Christopher Lagos a ADM-HQ en SC3 SpA por Saintard); al grupo de People y Finanzas en borrador (Ruelas, ver subtarea 2).
- Gabriela propuso el día 25 como fecha límite para cerrar los CECOs del mes sin modificaciones, mandando la planilla antes; Joaquín quiere cerrarlo antes de que los libros de remuneraciones lleguen a Nicolás. Falta responder.
- Notion [Cambios CECOS noviembre 2026](https://app.notion.com/p/415ab34c233b8099855bcc149329b18f) al día con todo lo anterior.

### Pendientes menores
- 2026-11-17 · Pablo Saintard solo en ADM-HQ (Reservo): decisión de [[Pablo Lucero]] entre aceptar la excepción, REV-SALES-HQ con Abadía y Piñeiro, o el parche de mover a Christopher Lagos a ADM-HQ. 2026-11-19: pregunta a Pablo en borrador de Slack (regla «nadie solo» por sociedad, cómo documentarla, parche de Lagos).
- 2026-11-17 · Migrar los 23 códigos legacy de julio 2026 del seed a los 5 vigentes (columna `reemplazado_por` y `cost_center_sheet` en `headcount.sql`); PR aparte, después del merge del #507. Propuesta en la sección 6 de la página de Notion «Cambios CECOS hasta octubre 2026». Los 7 códigos usados hasta octubre no se migran: son reclasificaciones de personas, no renombres.
- 2026-11-17 · La guía del repo dbt (`guia-clasificacion-personas.md`, cuentas de Sales) dice AE → G1100 y MDR → G1107; para Reservo se decidió AE y MDR → G1107. Alinear la guía o dejar la excepción escrita.
- 2026-11-17 · Repo `agendapro-dbt-redshift`: la copia principal está 72 commits detrás de origin y con un diff viejo del seed (filas de ago–sep que origin ya tiene); actualizarla sin perder nada. Borrar el worktree `agendapro-worktrees/dbt-headcount-rebuild` (ya no hace falta) y, tras el merge, `dbt-ceco-reservo-seed`.
- 2026-11-17 · Fecha de corte mensual con People: hasta qué día se actualiza Buk y qué día Finanzas revisa antes del cierre de nómina (Gabriela: ingresos hasta el 20). 2026-11-19: Gabriela propone el día 25 como límite; responderle.
- 2026-11-19 · Espejos de Pablo Santa Inés (MX y CO) y Pablo Lucero Ramos (MX) desaparecidos de Headcount - Raw desde octubre: esperar la explicación de Gabriela (¿se sacaron de Buk MX/CO o quedaron fuera de la carga?).
- 2026-11-17 · José Zalem (KAM) solo cambia si pasa a Customer Success; Silcris Jaramillo define cargo y equipo al volver del posnatal.

## Historial
- 2026-11-18 · Creada para agrupar el cierre de noviembre, con el frente de CECOs como subtarea 1 (Joaquín en conversación)
- 2026-11-18 · Avance desde Claude Code: carga de noviembre revisada, PR #507 abierto y cargado en Redshift, Notion al día con el estado de cada cambio, borrador a Gabriela con los 3 ajustes.
- 2026-11-19 · Avance desde Claude Code: planilla corregida por Gabriela verificada (Ángel, Romero, Portugueis, Cofré, Aucan), altas de Mancilla Vera y Ruelas en Reservo CL, Notion al día; mensaje a Gabriela enviado, borradores a Pablo Lucero y al grupo; subtarea 2 (contabilización de Ruelas) agregada en conversación.
- 2026-11-26 · Pasada a Notion: [Cierre de mes noviembre](https://app.notion.com/p/424ab34c233b8128bb92d968c8b00fcd). Desde hoy se sigue allá; esta nota queda como archivo (Joaquín en conversación)

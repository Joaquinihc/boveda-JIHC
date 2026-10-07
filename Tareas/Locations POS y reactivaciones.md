---
type: tarea
id: T-053
estado: en-curso
prioridad: media
tipo: cuadratura
origen: notion
proyecto: "[[_SoT Ventas POS]]"
temas: [pos, reactivaciones]
notion_id: 3ceab34c233b8075b476f3e8047975ba
notion_estado: "In progress"
notion_prioridad: "Medium"
notion_actualizado: "2026-09-11T18:39Z"
fuente: https://app.notion.com/3ceab34c233b8075b476f3e8047975ba
creado: 2026-09-01
revisar: false
---

# Locations POS y reactivaciones

Conversar con Ignacio para llegar a las mismas métricas: (1) compañías, locations y units; (2) ventas ISR vs Chargebee (yo registro más por reactivaciones y nuevas invoices). Doc: 'Inconsistencias registro venta POS Ignacio y Joaco' en Notion.

## Subtareas
- [ ] 1. Cuadrar compañías/locations/units con Ignacio
- [ ] 2. Explicar diferencia ventas ISR vs Chargebee
- [ ] 3. Revisar tabla y documentación que envió Ignacio: [purchase_attribution_model](https://app.notion.com/p/agendapro/purchase_attribution_model-3e3ab34c233b819db3d0e15fd1920b3c) · `SELECT * FROM dwh.pos_sales_sot;`
- [ ] 4. Corregir query: usar el vendedor de la factura cuando falta el agente
- [ ] 5. Corregir query: revisar también los deals del pipeline Sales ISR
- [ ] 6. Acordar con Ignacio si el cambio arriendo → cuotas cuenta como venta
- [ ] 7. Aclarar con Payments las ventas revertidas con factura pagada
- [ ] 8. Alinear con Ignacio el gráfico «POS activated per week»
- [ ] 9. Sugerir a Ignacio ajustes a su deck (semana parcial y corte)
- [ ] 10. Auditar casos borde de las tablas SoT de ventas POS
- [ ] 11. Incorporar las tablas SoT de ventas POS a dbt y Evidence
- [ ] 12. Presentación a Riverwood: ordenar las subtareas y editar el board book
- [ ] 13. Migrar el payback a las tablas nuevas de ventas POS y medir el impacto

## Notas

### Dónde está el detalle
- **Referencia técnica** (dónde vive el número del board book, queries, tablas y cómo reproducir a nivel company_id): [[SQL — SoT Ventas POS#Dónde vive el número de ventas POS del Board Book]]
- [[20260910-01 Cuadre ventas POS ISR jul–sep 2026]]: el cuadre +9 caso por caso (corte 10-sep), el contraste con `dwh.pos_sales_sot` (24-sep) y los arreglos propuestos. Base de las subtareas 2 a 7.
- [[20260909-01 POS activated per week — board book vs deck de Ignacio]]: por qué el mismo gráfico da distinto en los dos reportes. Base de las subtareas 1, 8 y 9.
- [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]: Ignacio explicó cómo funcionan sus tablas nuevas.
- [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]]: cómo están construidas esas tablas.
- [[20260924-01 Métricas POS del board book vs tablas SoT]] y [[20260924-02 Auditoría de casos borde de las tablas SoT de ventas POS]]: qué reporta hoy el board book y los casos borde de las tablas (24-sep).

### Novedades del 24-sep (reunión y auditoría)
- **1**: las tablas de Ignacio están a nivel compañía, no location; bajar a location exige un deal por location o que ISR cambie su flujo (reunión). La auditoría no la toca.
- **2, 5 y 6**: la auditoría las deja resueltas en parte; el detalle está en su sección «Estado de las subtareas de T-053».
- **3**: queda cubierta con la reunión y la auditoría.
- **4, 7, 8 y 9**: sin cambios. La 7 (Payments) quedó también como acción de la reunión.

Cada subtarea de arriba tiene aquí su descripción con el mismo número y título.

### 1. Cuadrar compañías/locations/units con Ignacio
- **Qué es**: llegar con Ignacio a las mismas cifras de compañías, locations (sedes) y units (máquinas POS), que hoy se cuentan distinto en cada reporte.
- **Por qué cuesta**: el board book trabaja a nivel **local** y reparte las máquinas entre locales con una heurística, porque ninguna fuente ata máquina ↔ local; Ignacio trabaja a nivel **empresa**. En el cuadre de ventas, los company_id que coinciden calzan también en unidades, así que la diferencia está en qué se cuenta, no en cómo se suma.
- **Qué hacer**: acordar con Ignacio en qué grano se reporta cada métrica (empresa, local o máquina) y cómo se pasa de uno a otro.
- **Resultado esperado**: las mismas cifras de compañías, locations y units en los dos reportes. Detalle en [[20260909-01 POS activated per week — board book vs deck de Ignacio]].

### 2. Explicar diferencia ventas ISR vs Chargebee
- **Qué es**: explicar por qué yo registro más ventas ISR que Ignacio / el listado (la descripción de la tarea lo atribuye a reactivaciones y facturas nuevas).
- **Avance**: el cuadre del 10-sep ya lo explica caso por caso: en jul–sep el board book tiene 87 unidades contra 78 del listado (+9), que son +13 contadas de más y −4 de menos, cada una con su causa. Las causas más grandes se atacan en las subtareas 4 a 7.
- **Qué falta**: presentarle a Ignacio ese cuadre y validar las causas con él.
- **Resultado esperado**: que los dos entendamos y aceptemos de dónde sale cada unidad de diferencia. Detalle en [[20260910-01 Cuadre ventas POS ISR jul–sep 2026]].

### 3. Revisar tabla y documentación que envió Ignacio
- **Qué es**: revisar lo que envió Ignacio, la documentación [purchase_attribution_model](https://app.notion.com/p/agendapro/purchase_attribution_model-3e3ab34c233b819db3d0e15fd1920b3c) y la tabla nueva `dwh.pos_sales_sot` (`SELECT * FROM dwh.pos_sales_sot;`).
- **Avance (24-sep)**: ya se contrastó la tabla contra el cuadre. Resuelve parte de la diferencia por diseño: separa `new` / `pos_additional` / `reactivation`, filtra facturas impagas y clasifica bien 569393. Todavía no resuelve: deja en `Others` el 67% de las filas de toda la historia (28% en 2026) y sigue contando como `new` las dos ventas revertidas. (Corregido el 24-sep: la tabla cubre toda la historia, desde 2023, no solo desde jun-2026.)
- **Qué falta**: leer la documentación y decidir si el board book puede pasar a usar esa tabla directamente.
- **Resultado esperado**: saber cuándo reemplazar los arreglos de las subtareas 4 a 6 por consumir `dwh.pos_sales_sot`. Detalle en [[20260910-01 Cuadre ventas POS ISR jul–sep 2026]].

### 4. Corregir query: usar el vendedor de la factura cuando falta el agente
- **Problema**: en el cuadre, dos ventas ISR (504406 y 535145) salieron en el board book como Stock/KAM, o sea, se contaron de menos. Son ventas solo de setup (`setup-smart-pos_2025-cl`, sin addon de arriendo ni cuotas) y en `pos_sales2` vienen sin agente. Sin agente, la regla de canal de `country_pos_sot.sql` agota sus niveles y termina decidiendo por la antigüedad de la empresa (más de 30 días desde su creación → KAM).
- **Dato que falta usar**: la factura en `pos_sales` sí trae el vendedor (`sales_user`, Bárbara Sánchez) y HubSpot las marca *Ganado con POS*, pero el SoT no lee ese campo.
- **Qué hacer**: en `country_pos_sot.sql`, cuando `pos_sales2` no traiga agente, tomar `pos_sales.sales_user` antes de caer en la regla de antigüedad.
- **Resultado esperado**: esas 2 ventas pasan a ISR, y el arreglo cubre cualquier venta de solo setup a futuro.

### 5. Corregir query: revisar también los deals del pipeline Sales ISR
- **Problema**: la venta 569393 (31-ago) la cerró monika@ con un deal ganado en el pipeline *Sales ISR*, pero el board book la contó como Stock. `pos_sales2` completó el agente con alvaro@, que es el KAM dueño del contacto (vía `hubspot_owner_fallback`) y no quien vendió; como alvaro@ está en la lista fija de KAM, la venta cayó en Stock.
- **Por qué no se corrige sola**: el chequeo de deals del SoT solo mira el pipeline *Seguimiento POS Expansión*, así que nunca ve el deal de *Sales ISR* que la habría clasificado bien.
- **Qué hacer**: ampliar ese chequeo al pipeline *Sales ISR*, para que un deal ganado el día de la venta mande sobre el agente completado.
- **Resultado esperado**: 569393 y los casos iguales pasan a ISR. La tabla nueva de Ignacio (`dwh.pos_sales_sot`) ya la clasifica bien, así que esto solo hace falta mientras el board book siga usando `country_pos_sot.sql`.

### 6. Acordar con Ignacio si el cambio arriendo → cuotas cuenta como venta
- **Contexto**: es la causa más grande de la diferencia: 5 de las 13 unidades que el board book cuenta de más (494549 en julio; 373415, 453116, 511752 y 520667 en agosto). En Chargebee se ve el mismo día `added smart-pos-cuotas` + `removed smart-pos-arriendo`: el cliente cambia la forma de pago del mismo terminal, no compra un equipo nuevo.
- **Hoy**: el board book las cuenta como venta nueva y el listado no. La tabla de Ignacio no las trata como `new` sino como `purchase_type = 'pos_additional'`.
- **Qué hacer**: decidir con Ignacio el criterio. Si no cuentan como venta, el board book tiene que excluirlas (se detectan por ese par de movimientos el mismo día y en la misma empresa), lo que equivale a adoptar su `pos_additional`.
- **Resultado esperado**: una misma definición de "venta nueva" en los dos reportes.

### 7. Aclarar con Payments las ventas revertidas con factura pagada
- **Contexto**: 562659 y 563154 (agosto) tienen la factura pagada, pero el addon POS se quitó y en el listado aparecen como POS = NO. El board book las cuenta (+2) y la tabla de Ignacio también las deja como `new` válidas.
- **Lo que no sabemos**: si la venta sí ocurrió y lo que falla es el registro del addon, o si se revirtió y la factura quedó mal.
- **Qué hacer**: preguntarle a Payments qué pasó con esas dos ventas, antes de tocar la query.
- **Resultado esperado**: saber si se cuentan o se excluyen, y si hay que corregir algo en Chargebee.

### 8. Alinear con Ignacio el gráfico «POS activated per week»
- **Contexto**: el board book (slide 2.4) y el deck de Ignacio (slide 5) tienen el mismo gráfico con el mismo título, pero miden cosas distintas. El board cuenta **locales**, solo con días de ticket promedio sobre un umbral, reinicia la suma cada mes y filtra cohortes de los últimos 12 meses. Ignacio cuenta **empresas nuevas**, con transacciones brutas acumuladas desde la venta y sin filtro de cohorte. En 24 semanas dan 150 vs 160: se parecen porque los efectos se compensan, no porque midan lo mismo.
- **Qué conversar**: (1) local vs empresa: la asignación de máquina a local del board es una heurística, porque ninguna fuente ata máquina ↔ local (es el mismo punto que la subtarea 1); (2) el reinicio mensual de la suma, que hace que el día de activación dependa del mes en que cae el cruce; (3) el filtro de 12 meses, que oculta activaciones tardías reales.
- **Resultado esperado**: una sola definición de "POS activado" y decidir cuál de los dos gráficos se corrige.

### 9. Sugerir a Ignacio ajustes a su deck (semana parcial y corte)
- **Contexto**: en su deck la última barra es la semana en curso, que está incompleta y no se marca. Además, el panel «Evidence hoy» es una captura del board con **julio** seleccionado, así que no se puede comparar barra a barra con su serie, que llega al 7-sep.
- **Qué hacer**: pedirle que marque la semana parcial y que el panel «Evidence hoy» use el mismo corte que su serie.
- **Resultado esperado**: que la comparación del deck no induzca a error. Es menor; podría ir dentro de la subtarea 8.

### 10. Auditar casos borde de las tablas SoT de ventas POS
- **Qué es**: antes de que `dwh.pos_sales_sot` pase a ser la única fuente de los gráficos, revisar los casos borde de las tablas de Ignacio (`pos_ncro`, `pos_resales`, `pos_sales_sot`) y validar que calcen con lo que muestra hoy Evidence. Si algún caso obliga a cambiar una query, avisarle a Ignacio, porque puede cambiar resultados anteriores. Acordado en [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]].
- **Avance (24-sep)**: la auditoría ya está hecha, en [[20260924-02 Auditoría de casos borde de las tablas SoT de ventas POS]]. Tiene 17 hallazgos, y la tabla todavía no puede reemplazar a los gráficos.
- **Pasos**: (1) revisar el informe y marcar con qué hallazgos estás de acuerdo; (2) enviarle a Ignacio los casos que obligan a cambiar la query (el informe trae un borrador de mensaje); (3) acordar con él las definiciones abiertas: si manda la cantidad del deal o la de la factura, si la ventana deal↔factura es ±45 días o solo hacia adelante, si un cambio arriendo → cuotas o una baja de terminal cuenta como venta (`addon_restart`), si en `qty_up` se cuenta el aumento o el total, y qué hacer con `Others`.
- **Resultado esperado**: una lista validada de arreglos y criterios acordados con Ignacio.
- Venía de la tarea [[Auditar casos borde de las tablas SoT de ventas POS]] (T-071), que pasó a ser esta subtarea el 25-sep.

### 11. Incorporar las tablas SoT de ventas POS a dbt y Evidence
- **Qué es**: dejar las tablas de Ignacio como parte oficial del proyecto dbt y usarlas en el board book. Hoy existen solo en la rama `refactor/postgres-to-redshift` del repo `agendapro/agendapro-dbt` (PRs #396, #400 y #402; no están en `master`) y ningún tablero las usa. Detalle en [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]].
- **Pasos**: (1) confirmar con Ignacio si los modelos se mergean a la rama principal y quién los mantiene (tests, documentación, frecuencia); (2) agregar la tabla como fuente en `sources/dwh/` de Evidence; (3) reactualizar los gráficos POS del board book para que usen la tabla consolidada como única fuente (va después de la subtarea 10).
- **Decisiones antes de migrar los gráficos** (del análisis [[20260924-01 Métricas POS del board book vs tablas SoT]]): qué hacer con `Others` (25% de las unidades de ene–ago), sobre todo contra el budget; si «New / Stock» es canal o tipo de compra; si se adopta `is_valid_purchase` (factura pagada + formulario), que cambia el criterio del 7-sep; la exclusión de Tabata y cómo se cuentan las reactivaciones; si «POS activated per week» pasa a grano compañía o se renombra a locales (y el reinicio mensual y el filtro de 12 meses); el payback, que sigue en `country_pos`.
- **Resultado esperado**: el board book leyendo `dwh.pos_sales_sot` como única fuente.
- Venía de las tareas [[Incorporar las tablas SoT de ventas POS a dbt y Evidence]] (T-072) y [[Reactualizar los gráficos POS del board book con la tabla consolidada]] (T-074), unidas el 25-sep.

### 12. Presentación a Riverwood: ordenar las subtareas y editar el board book
- **Qué es**: preparar la presentación de la reunión trimestral con Riverwood. Para eso hay que ordenar las demás subtareas de esta tarea (qué queda resuelto antes de presentar y qué se presenta como pendiente) y con eso editar la presentación del board book y completar los archivos pedidos.
- **Pasos**: (1) revisar el calendario de Pablo y agendar la presentación (se habló del día 14); (2) ordenar las subtareas 1 a 11 y 13 según lo que entra en la presentación; (3) editar la presentación del board book y completar los archivos pedidos.
- **Resultado esperado**: presentación a Riverwood agendada y con los números de ventas POS ordenados y explicados.
- Venía de la propuesta [[Coordinar presentación con Riverwood (revisar calendario de Pablo)]] (T-075), de [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]; se mencionó de nuevo en [[2026-10-05 Payback y migración a tablas POS]]. Pasó a ser esta subtarea el 7-oct.

### 13. Migrar el payback a las tablas nuevas de ventas POS y medir el impacto
- **Qué es**: pasar el cálculo del payback a la tabla nueva de ventas POS (columna de compra válida) y medir cuánto cambia la cantidad de POS que considera. Es la parte del payback que la subtarea 11 dejó pendiente (hoy sigue en `country_pos`).
- **Pasos**: (1) verificar qué POS toma hoy el cálculo (solo primera venta o también adicionales y reactivaciones); (2) comparar la cantidad de POS del registro actual vs el nuevo y el efecto en el payback; (3) decidir formalmente si reactivaciones y ventas adicionales entran al payback.
- **Resultado esperado**: payback calculado con la tabla nueva y el impacto medido.
- Venía de la propuesta [[Migrar el payback a las tablas nuevas de ventas POS y medir el impacto]] (T-088), de [[2026-10-05 Payback y migración a tablas POS]]. Pasó a ser esta subtarea el 7-oct.

## Historial
- 2026-09-10 · Cuadratura jul–sep 2026 (Chile, ISR) a nivel company_id contra el board book: +9 unidades, con causa identificada caso por caso. 17 company_id investigados en `pos_sales2`, facturas, addons de Chargebee, planilla KAM y HubSpot.
- 2026-09-22 · Importada desde Notion (In progress).
- 2026-09-24 · Contrastado contra `dwh.pos_sales_sot`: separa `pos_additional` de `new` y filtra impagas, pero deja 58% del canal en `Others` y solo cubre desde el 12-jun-2026. Julio calza exacto (27) sumando ISR + Others.
- 2026-09-24 · Documentada la diferencia de metodología del gráfico "POS activated per week" (board book slide 2.4 vs deck de Ignacio slide 5): grano local vs empresa, trx qualifying vs brutas, acumulación por mes relativo vs desde la venta, filtro de cohorte 12m. Análisis del 9-sep-2026.
- 2026-09-24 · Corrección del contraste con `dwh.pos_sales_sot`: la tabla cubre desde 2023 (no desde el 12-jun-2026) y `Others` es 67% del total y 28% en 2026 (no 58%). Ver [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]].
- 2026-09-24 · Mencionada de nuevo en [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]] (aclarar con Payments las ventas revertidas con factura pagada)
- 2026-09-25 · Subtarea agregada: «Auditar casos borde de las tablas SoT de ventas POS» (de [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]) (Joaquín por Slack, aplicado en conversación)
- 2026-09-25 · Subtarea agregada: «Incorporar las tablas SoT de ventas POS a dbt y Evidence», con «Reactualizar los gráficos POS del board book…» como paso 3 (Joaquín por Slack, aplicado en conversación)
- 2026-10-07 · Subtarea agregada: «Coordinar presentación con Riverwood (revisar calendario de Pablo)», como subtarea 12 (ordenar las subtareas y editar la presentación del board book) (de [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]) (Joaquín en conversación)
- 2026-10-07 · Subtarea agregada: «Migrar el payback a las tablas nuevas de ventas POS y medir el impacto», como subtarea 13 (de [[2026-10-05 Payback y migración a tablas POS]]) (Joaquín en conversación)

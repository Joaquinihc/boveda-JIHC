---
type: metodologia
temas: [pos, ventas, payments]
proyecto: "[[_SoT Ventas POS]]"
fuente: trabajo propio
fecha: 2026-09
---

# POS · Fuentes de verdad (SoT): ventas y transacciones

> **Propósito**: dejar definida y explicada UNA fuente de verdad (*source of truth*, SoT) para las
> dos preguntas básicas del negocio POS — **cuánto vendemos** (Parte I: qué se vendió, quién,
> cuándo y a quién) y **cuánto transaccionan los comercios** (Parte II: GPV y activación). Todas
> las métricas de activación, GPV y performance de canal deben calcularse sobre estas fuentes.
>
> **Query de ventas**: [`POS_VENTAS_SOT.sql`](POS_VENTAS_SOT.sql) — corre tal cual en Redshift.
> **Generado**: 2026-08-04 · **Actualizado**: 2026-08-19 — se agregó la Parte II (SoT de
> transacciones), el estado vivo del bug de Haulmer (verificado en el DWH el 19-ago) y el plan de
> migración de la planilla KAM al deal de HubSpot.
> **Owner**: Ignacio Embry (PM ProPay).
> Respaldo técnico: `PROPAY_CONTEXT.md` · `CATASTRO_TABLAS_TRANSACCIONALES_POS.md` ·
> `POS_ACTIVACION_REACTIVACIONES.md` · `2026-08-04-handoff-tablero-ventas-kam-fuentes.md`.

---

## 0. Resumen ejecutivo (para leer sin contexto técnico)

El negocio POS se mide con dos preguntas distintas, y cada una tiene su propia fuente de verdad:

| Pregunta del negocio | SoT | De qué se compone |
|---|---|---|
| ¿Cuántos POS vendimos, quién los vendió y cuándo? | **SoT de ventas** (Parte I) | Chargebee (facturación) **unido con** la planilla del equipo KAM — y en adelante, el deal de HubSpot |
| ¿Cuánto transaccionan los comercios? (GPV, activación) | **SoT de transacciones** (Parte II) | `dwh.propay_transactions` (cobros vía app) **más** los archivos de conciliación de cada proveedor (cobros directos en el terminal) |

**Glosario mínimo** (los términos que usa todo el documento):

| Término | Qué es |
|---|---|
| **GPV** | *Gross Payment Volume*: la plata que los comercios cobran a través de nuestros POS |
| **Chargebee** | Nuestro sistema de suscripciones y facturación. Toda venta que se factura deja huella acá |
| **Planilla KAM** (`dwh.pos_v2`) | El registro manual de ventas del equipo KAM, sincronizado al warehouse |
| **ISR** | *Inside Sales Rep*: vende POS a clientes nuevos, dentro del flujo de venta del software |
| **KAM** | *Key Account Manager*: vende POS a la cartera de clientes existentes |
| **Reactivación** | Comercio que ya tenía POS, dejó de usarlo, y el equipo lo hace volver a transaccionar |
| **DWH** | El data warehouse (Redshift), donde viven las tablas `dwh.*` |
| **Transacción de prueba** | El cobro de prueba que hace el técnico al instalar (100 CLP / ~1 MXN) |
| **Activación** | Que un comercio recién vendido empiece a transaccionar de verdad (hitos ≥1 / ≥5 / ≥10 / ≥30 trx) |
| **Evento** | La unidad de la SoT de ventas: una empresa que compró en un mes (aunque lleve 2+ máquinas) |
| **New / Stock** | Etiquetas del tablero de ventas: *New* = venta a un cliente nuevo del software (≈ ISR); *Stock* = venta a la cartera existente (≈ KAM) |
| **Cohorte** | El grupo de comercios vendidos en un mismo mes, que se sigue en el tiempo |
| **M0, M1…** | Meses desde la venta: M0 = el mismo mes del evento, M1 = el mes siguiente |
| **Gateway** | El sistema de AgendaPro por el que pasa un cobro cuando se hace desde la app |
| **Archivo de conciliación** | El reporte que cada proveedor de terminales envía después, con el detalle de todos los cobros que procesó |
| **Evidence** | La herramienta donde viven los dashboards oficiales (`fintech/pos-ventas`, Board Book) |
| **Trx** | Transacción |

Las seis ideas del documento:

1. **Ninguna fuente de ventas ve el universo completo, por eso la SoT es una unión.** Chargebee ve
   todo lo que se factura (ISR + KAM) pero **no ve reactivaciones**, porque reactivar un POS que el
   cliente ya tiene no genera factura nueva. La planilla del equipo KAM es el único lugar donde
   consta el tipo de venta y la reactivación, pero **no ve nada de ISR**. En julio-2026, 12 de 69
   ventas del mes (17%) eran reactivaciones invisibles para el dashboard oficial (el gráfico *POS vendidos por mes*
   de `fintech/pos-ventas`, en Evidence). La unión se
   verificó contra ambas fuentes por separado y **cuadra sin residuo** (§2b, §2c).
2. **Para transacciones, la fuente a nivel de detalle es `dwh.propay_transactions`** (el gateway de
   AgendaPro): es la única tabla con monto exacto y empresa siempre identificada. El GPV oficial
   suma eso más los cobros "externos" — los tecleados directo en el terminal — que llegan por los
   archivos de conciliación de cada proveedor (§7).
3. **La activación se calcula excluyendo la transacción de prueba de instalación** (100 CLP exactos
   en Chile, ~1 MXN en México — la hace el técnico al encender la máquina). Con las transacciones
   crudas, "≥1 transacción en 30 días" da ~90%; excluyendo la prueba, ~54%. La versión cruda mide
   que el técnico instaló la máquina, no que el comercio vendió (§9).
4. **A la métrica de activación entran solo las "ventas nuevas puras"**: primera compra del comercio
   y cero transacciones previas. Las reactivaciones y recompras se miden aparte, porque el comercio
   ya transaccionaba y contamina la métrica — un solo caso de recompra llegó a explicar el 94% del
   GPV de su cohorte (§10).
5. **La etiqueta "Reactivación" de la planilla se valida antes de usarse**: solo cuenta como
   reactivación real un comercio con **≥60 días sin transaccionar**. Con la etiqueta cruda, el GPV
   atribuido a reactivaciones se infla 94% — casi todo venía de comercios que nunca se habían
   enfriado (§6).
6. **Estado de la calidad de datos**: el bug que hacía perder transacciones de Haulmer (detectado
   en julio: 39% de sus cobros externos sin dueño, ≈ USD 257 K/mes de GPV sin contar) **ya fue
   corregido por el equipo de Data** — verificado en vivo el 19-ago, el residuo actual es ~6%
   ≈ USD 25 K/mes. Queda trabajo de mantención, no de corrección (§8). Y la planilla KAM tiene
   reemplazo decidido: el registro oficial de la venta pasa al **deal de HubSpot** (§11).


---

# Parte I — La SoT de ventas

## 1. El problema que resuelve

Ninguna de las dos fuentes disponibles ve el universo completo:

| Fuente | Qué ve | Qué NO ve |
|---|---|---|
| `dwh.pos_sales2` (Chargebee) | Todas las ventas **facturadas**: ISR + KAM | **Reactivaciones** (no generan factura nueva) y un puñado de ventas que nunca se facturan (11 en 2026 — §2c) |
| `dwh.pos_v2` (planilla del equipo KAM) | Todas las ventas **del equipo KAM**, con `tipo_venta` (Venta Nueva / Reactivación) | **ISR** por completo |

Consecuencia práctica: el gráfico *POS Vendidos por mes (canal)* del dashboard
`fintech/pos-ventas` se alimenta solo de Chargebee, así que **el trabajo de reactivación del
equipo KAM es invisible** en el reporte oficial.

### Evidencia del solape (medida 2026-08-04, cohortes 2026)

| Tipo declarado en la planilla | Eventos | Con factura en Chargebee (mismo mes) |
|---|---|---|
| Venta Nueva | 169 | 157 (**93%**) |
| Reactivación | 22 | 4 (**18%**) |

O sea: las ventas nuevas se cruzan bien entre las dos fuentes, y **las reactivaciones
prácticamente no existen en el DWH**. La tolerancia de ±1 mes en el match agrega solo 1 caso,
así que el mes calendario exacto es criterio suficiente.

En la dirección inversa, 12 a 28 eventos por mes están en Chargebee y no en la planilla: casi
todo ISR, más algún KAM que no registró la fila.

*(Los conteos de esta sección son del snapshot del 4-ago; el censo de §2c, dos días después,
difiere levemente — 174 ventas nuevas vs 169 — por ser fotos distintas de una planilla viva.)*

---

## 2. Qué agrega la fuente única

Eventos por mes (2026): la columna **Δ** es lo que hoy no se ve en ningún dashboard.

| Mes | SoT | Solo Chargebee (visión actual) | Δ | De los Δ, reactivaciones |
|---|---|---|---|---|
| ene-26 | 40 | 37 | +3 | 0 |
| feb-26 | 28 | 28 | 0 | 0 |
| mar-26 | 45 | 45 | 0 | 0 |
| abr-26 | 54 | 53 | +1 | 0 |
| may-26 | 56 | 51 | +5 | 0 |
| jun-26 | 58 | 54 | +4 | 2 |
| **jul-26** | **69** | **57** | **+12** | **12** |
| ago-26 (parcial) | 8 | 3 | +5 | 4 |

Julio es el mes revelador: **12 de 69 eventos (17%) son reactivaciones sin factura**. El
registro de `tipo_venta='Reactivación'` empieza en jun-2026, así que antes de esa fecha no se
puede reconstruir — no es que no hubiera reactivaciones, es que no se anotaban.

⚠ Nota de cuadratura: esta serie mensual es el snapshot del 4-ago, previo al fix de doble conteo
de §2c. El total 2026 auditado post-fix (6-ago) es **380 eventos**; las verificaciones de §2c y
§3c corren sobre ese universo — por eso sus totales (387 pre-fix / 380 post-fix) no calzan uno a
uno con esta tabla.

Totales desde ene-2025: **1.116 eventos · 1.085 empresas**.

| Canal | Tipo | Eventos |
|---|---|---|
| ISR | Venta Nueva | 691 |
| KAM | Venta Nueva | 395 |
| KAM | Reactivación (planilla) | 22 |
| KAM | Reactivación (derivada) | 1 |
| KAM | POS adicional | 7 |

| Origen del evento | Eventos | Qué son |
|---|---|---|
| Ambas fuentes | 303 | Venta KAM registrada y facturada — el caso sano |
| Solo Chargebee | 773 (691 ISR + 82 KAM) | Facturadas sin registro en la planilla |
| Solo planilla | 40 | 18 reactivaciones + 22 ventas KAM que nunca se facturaron (§2b) |

---

## 2b. Cuadratura con el dashboard de Evidence (verificada, residuo cero)

El gráfico *POS vendidos por mes (canal)* cuenta **unidades** (`SUM(item_qty)`); la SoT cuenta
**eventos** (empresa × mes). La diferencia se descompone exacta, mes a mes:

```
eventos_SoT  =  unidades_Evidence  +  unidades_sin_factura  −  exceso_multi-máquina
     1.116   =        1.161        +          47            −         92
```

| Concepto | Valor | Qué es |
|---|---|---|
| Unidades en Evidence (CL+MX, desde ene-25) | 1.161 | réplica exacta de la consulta que alimenta el dashboard (`pos_sold_monthly.sql`) — coincide mes a mes |
| + unidades que la unión agrega | +47 | 40 eventos: 18 reactivaciones + 22 ventas sin factura registrada |
| − exceso multi-máquina | −92 | 65 eventos llevaron más de 1 terminal (157 unidades en 65 eventos) |
| = eventos de la SoT | **1.116** | ✔ cuadra sin residuo |

**No falta ningún comercio.** Se comparó par por par (`company_id` × mes): los 1.161
eventos-unidad de Evidence están **todos** en la SoT; el conjunto de faltantes es vacío.

Dos hipótesis descartadas en el camino:

- **No es un tema de país**: en esta ventana no hay ventas POS en Argentina, Colombia ni otros
  países — el total de Evidence es exactamente Chile + México.
- **No es el colapso a mes**: ninguna empresa registró dos compras en días distintos del mismo
  mes (0 casos), así que agrupar por mes no fusionó ventas separadas.

El [artefacto interactivo](https://claude.ai/code/artifact/c385cf56-5401-438f-8dd8-e76eb17a4b76)
trae un selector **"Métrica de volumen: eventos / unidades"** para comparar directo
contra el dashboard, y una tabla puente con esta descomposición por mes.

---

## 2c. Verificación de completitud contra la planilla KAM (6-ago-2026)

Premisa: los KAM registran **todas** sus ventas en `pos_v2`, así que Stock + Reactivación de la
SoT debería calzar con la planilla.

### Dirección 1 — ¿encontramos todas las filas de la planilla? Sí, el 100%

Censo de `pos_v2` para 2026: **196 filas / 216 unidades / cero nulos** en `company_id`, `ano`, `mes`.

| País | Venta Nueva | Reactivación | Total |
|---|---:|---:|---:|
| Chile | 106 (119 uds) | 21 (25 uds) | **127** |
| México | 68 (71 uds) | 1 (1 ud) | **69** |

Las 196 aparecen en la SoT (el nivel `planilla` de `canal_origen` da exactamente 196 —
la jerarquía de niveles de canal se define en §3c). Desglose
por fuente, que cuadra fila por fila con el censo:

| País | Cat | En ambas | Solo planilla | Solo Chargebee |
|---|---|---:|---:|---:|
| Chile | Stock | 102 | 4 | **12** |
| Chile | Reactivación | 5 | 16 | — |
| México | Stock | 61 | 7 | **9** |
| México | Reactivación | — | 1 | — |

Chile 102+4+5+16 = 127 ✅ · México 61+7+1 = 69 ✅

**Dato de negocio**: 16 de las 21 reactivaciones de Chile (76%) y 11 ventas nuevas (4 CL + 7 MX)
**no tienen factura en Chargebee**. Sin la planilla serían invisibles.

### Dirección 2 — ¿tenemos ventas KAM que la planilla no tiene? 21, y 13 son de fecha

De los 21 eventos que clasificamos Stock sin match en la planilla del mismo mes:

| Caso | Eventos | Qué es |
|---|---:|---|
| **Nunca en la planilla** | **4** | Ventas KAM sin registrar → acción para el equipo |
| En la planilla pero en **otro mes** | 13 | El KAM registró el mes de cierre, Chargebee facturó otro |
| Vía owner del deal (nivel 1) | 4 | Clasificadas KAM por HubSpot |

Los 4 sin registrar: `64193` y `470847` (ene-26, sin agente) · `397356` (abr-26, rocio) ·
`420644` (abr-26, janet).

### 🔴 Bug encontrado y corregido: doble conteo por desfase de mes

El `FULL OUTER JOIN` por empresa + **mes exacto** creaba **dos eventos para una sola venta** cuando
el KAM registraba el mes de cierre y Chargebee facturaba el siguiente. Medido en 2026: **6 pares a
1 mes de distancia** (los 4 pares a 3+ meses sí son ventas distintas; el fix elimina 7 eventos en
total — el séptimo cae en el mismo rescate). Ejemplo: empresa `91759`,
**5 unidades**, nuestra fecha jul-26 y la planilla dice jun-26.

**Corrección aplicada**: primero se cruzan las ventas del mismo mes; las que quedan sueltas se
emparejan una a una solo si están a un mes de distancia (en la query: CTEs `cb_libre` /
`sh_libre` / `pares` / `sh_ali`). El campo `sale_month_sheet`
conserva el mes que declaró el KAM.

| | Eventos 2026 |
|---|---:|
| Antes del fix | 387 |
| Después del fix | **380** |
| Dobles residuales a 1 mes | **0** |

⚠ Los números por canal reportados antes de este fix estaban inflados en **7 eventos (1,8%)**.

---

## 3. Reglas de consolidación

**Grano**: 1 fila por **evento** = `company_id × mes de venta`.

**Cohorte**: `cohort_month` es **siempre el mes del evento**, nunca la primera compra histórica
del comercio. En una reactivación el reloj se reinicia — es lo que mide el impacto del equipo
hoy. ⚠ Esto difiere de los análisis previos de activación, que cohortizaban por "primera compra
de la vida"; los números **no son comparables 1:1** con esos.

**Autoridad por campo** (orden de precedencia):

| Campo | Regla |
|---|---|
| `canal` | Jerarquía por **ownership**, con el tiempo como último recurso — ver §3c |
| `canal_origen` | Cuál de los 5 niveles de la jerarquía decidió (`planilla` / `deal_hubspot` / `agente_chargebee` / `kam_pos_owner (inferido)` / `ventana_tiempo (inferido)`) |
| `tipo_venta` | 1) De la planilla (única fuente real). 2) Derivado: primera compra = `Venta Nueva`; con compra previa y ≥60 días sin transaccionar = `Reactivación (derivada)`; con actividad reciente = `POS adicional` |
| `units` | Chargebee (facturación); si no existe, la planilla |
| `pais` | `dwh.companies.country_name`; fallback al `pais` de la planilla |
| `sale_day` | 1) `addon_added_at` (qué es un addon: §3b) si es **plausible**; 2) **fecha exacta de la factura**; 3) día de la planilla; 4) día 1 del mes |
| `fecha_origen` | `addon` / `factura` / `sheet` / `mes` — cuál de las cuatro se usó |
| `fecha_confiable` | 1 si `fecha_origen` es `addon`, `factura`, o `sheet` ≥ 21-jul-2026 |

**Por qué la planilla manda en el canal**: es la única evidencia directa de quién cerró la
venta. Verificado que la regla canónica clasificaría como ISR a 12 eventos de 2026 que la
planilla registra como KAM — la unión corrige esa atribución.

---

## 3c. El canal: ownership primero, tiempo como último recurso

**Principio**: para atribuir un evento histórico solo sirven campos **point-in-time** (que quedaron
fijos al momento de la venta). Los de **estado actual** contaminan hacia atrás.

| Nivel | Fuente | Tipo | Qué puede decidir |
|---|---|---|---|
| 0 | Planilla KAM (`pos_v2`) | point-in-time | Solo KAM (es el registro propio del equipo) |
| 1 | Owner del **deal** ganado en HubSpot | **point-in-time** | KAM o ISR · solo desde abr-2026 |
| 2 | `pos_sale_agent` de Chargebee | **point-in-time** | KAM o ISR · 79% de cobertura histórica |
| 3 | `contacts.kam_pos_owner` | estado actual | Solo KAM, **marcado como inferido** |
| 4 | Ventana ≤30 d (≥jun-26) / ≤90 d (antes) | inferencia | Último recurso |

`canal_origen` expone cuál disparó. Resultado sobre los 387 eventos de 2026 (⚠ medición previa
al fix de doble conteo de §2c; post-fix el universo es 380 y los porcentajes se mueven
marginalmente):

| Nivel | Canal | Eventos |
|---|---|---:|
| 0 · planilla | KAM | 196 |
| 1 · deal HubSpot | KAM | 5 |
| 2 · agente Chargebee | ISR | 142 |
| 2 · agente Chargebee | KAM | 9 |
| 3 · `kam_pos_owner` *(inferido)* | KAM | 7 |
| 4 · ventana *(inferido)* | ISR | 20 |
| 4 · ventana *(inferido)* | KAM | 8 |

**91% se clasifica con dato duro.** La ventana decide 28 eventos (7,2%) y `kam_pos_owner` solo 7
(1,8%). Por eso deja de importar si la ventana es de 30 o de 90 días.

### Roster de KAM (validado 6-ago-2026)

`rocio` · `janet` · `jaime` · `fernandobarra` · `alvaro` · `tabata`

**`cristobal@` NO va** — era el líder del área, no vendedor de cartera. Aparece en
`contacts.kam_pos_owner` (1 contacto) y hay que descartarlo.

### Por qué `kam_pos_owner` va en el nivel 3 y no más arriba

Es un campo de asignación de **prospección**, no de atribución de venta, y no tiene columna de
fecha (compárese con `optimization_owner_date`), así que no se puede reconstruir cuándo se asignó.
Error medido si se usara como autoridad, sobre 2026:

- **37 falsos positivos** — ventas de no-KAM en contactos que *hoy* son propiedad KAM (84% marcadas "new")
- **34 falsos negativos** — ventas de KAM sin la marca (rocio tiene 17 de sus 41 así)
- **Total 71 de 371 eventos evaluados = 19% mal clasificadas**

Además está incompleto en los dos sentidos: **no trae a tabata** y **sí trae a cristobal**.

### ⚠ Esta regla ya no cuadra con el board, a propósito

El board usa solo `roster hardcodeado → ventana → is_new_merchant`. Para reproducirlo está
`POS_RECALCULO_TABLEROS.sql`, que mantiene el canal congelado justamente para poder aislar qué
parte de cada diferencia viene del grano, de la fuente y de la clasificación.

### 🔴 Deuda abierta: el roster está hardcodeado y no existe roster de ISR

Hoy `kam_roster` es una lista de 6 emails en un `WHERE`, y **ISR = "el que no es KAM"**. Eso es la
misma falla lógica que tenía la regla vieja, solo movida un paso: un account manager, alguien de
soporte o un KAM fuera del roster que ejecute el cambio en Chargebee queda clasificado como ISR
sin haber vendido nada.

**Impacto medido** (auditoría 6-ago-2026 de los 12 eventos de Chile que pasaron de Stock a New):

| | Eventos | |
|---|---:|---|
| Atribución limpia | 5 | sin rastro de KAM en planilla, deal ni `kam_pos_owner` |
| `leonardoruiz` | 5 | perfil **no** parece ISR: vende a comercios de 127 a 2.367 días de antigüedad y solo operó ene-feb 2026 |
| Reclamadas por un KAM en otro mes | 2 | empresas `7743` (Fernando, abr-26) y `430053` (Álvaro, may-26) |

En neto, esta regla le agrega +10 eventos New al conteo de Chile; descontando los 7 casos
dudosos (5 de `leonardoruiz` y 2 reclamadas por un KAM en otro mes), **el +10 podría ser +3**.
Los 196 eventos que vienen de la planilla no se ven afectados: siguen clasificados por dato duro.

### Solución acordada: planilla maestra de vendedores

Cargarla al warehouse con el patrón que ya existe para Haulmer
(`stg_google_sheets__haulmer_raw_pos2` → `dwh.haulmer_pos2`). Columnas mínimas:

```
email · rol (KAM / ISR / otro) · pais · valid_from · valid_to
```

**`valid_from` / `valid_to` es lo que hace la diferencia**: permite atribuir según el rol que la
persona tenía **en la fecha de la venta** (point-in-time), no el que tiene hoy. Sin eso, cada cambio
de equipo reescribe la historia — es exactamente el bug de `contacts.kam_pos_owner` por el que lo
bajamos al nivel 3.

Cuando exista, el cambio en la SoT son ~4 líneas: reemplazar los dos CTEs de roster por lecturas de
esa tabla y exigir en el nivel 2 `rol = 'ISR'` en vez de "no está en el roster KAM". El resto de la
jerarquía no se toca.

*(Nota: el pipeline de dbt corre sobre Redshift, no Snowflake — la planilla debe entrar por el
staging de Google Sheets que ya está montado.)*

Los 5 nombres inequívocos para arrancar la lista: `danielasuarez` (30 ventas), `barbara` (21),
`julialyz` (17), `matias` (14), `naily` (12) — los cuatro primeros con 100% de sus ventas a clientes
recién registrados. Los otros 21 tienen 1-2 ventas cada uno y hay que revisarlos uno por uno.

---

## 3b. La fecha de venta: dos fallas estructurales, ninguna de calidad de datos

**`addon_added_at` es la fecha conceptualmente correcta.** Es el acto comercial —el momento en
que la máquina se cuelga de la suscripción— independiente del ciclo de facturación, de las cuotas
y de la cobranza. No fue una mala elección de diseño. Falla por dos razones estructurales.

### Falla 1 · Solo existe si el POS se arrienda o financia

Los únicos addons de POS que existen en Chargebee son `smart-pos-arriendo_2025-cl` y
`terminal-oel-renta_2025-mx` — **los dos de arriendo**. Una venta directa es un ítem único de
factura: no crea addon, así que **no hay fecha por construcción**.

| Modalidad | Filas 2025-26 | Con `addon_added_at` |
|---|---:|---:|
| Arriendo / cuotas | 743 | **743 (100%)** |
| Venta directa | 376 | **20 (5,3%)** |

Consecuencia importante: **el "deterioro" de jul-26 no es un bug de datos, es cambio de mix.**
La correlación es casi perfecta e inversa (`% con addon ≈ 100 − % venta directa`):

| Mes | % venta directa | % con addon |
|---|---:|---:|
| feb-26 | 11 | 89 |
| abr-26 | 14 | 91 |
| jun-26 | 25 | 77 |
| jul-26 | **35** | **67** |

Y entre 50% y 100% de la venta directa es de México, así que **la vista semanal está sesgada
sistemáticamente contra México**, no aleatoriamente incompleta.

### Falla 2 · `pos_sales2.addon_added_at` no es la fecha del evento, es un atributo de empresa

El modelo aplica `addon_sale_rank = 1` dentro de un CTE llamado literalmente
**`first_pos_sale_per_company`**: la primera vez que esa empresa tuvo un POS. Se construyó para
atribuir al **vendedor original** (`pos_sale_agent`) y la fecha vino de arrastre.

Por eso en una recompra o reactivación devuelve la compra original: 43 filas con gap de hasta
**−676 días** → ventas de 2026 dibujadas en semanas de 2024-25. De **54 recompras/reactivaciones
facturadas desde ene-25, solo 9 (17%) quedan fechadas en el mes real.**

**Pero el dato bueno existe.** 24 de esas 43 filas tienen un cambio de addon real cerca del mes
de venta en `int_chargebee__subscription_addon_changes` (gap mediano +11 días). El `rank = 1` lo
descarta. Por eso la SoT va a la tabla de cambios y **no usa `pos_sales2.addon_added_at`**.

### La jerarquía que usa la SoT

1. **Cambio de addon POS más cercano al mes de venta** (CTE `addon_near`) — el acto comercial
2. **`invoice_created_at`** — emisión, no cobranza. Único dato disponible en venta directa
3. **Día de la planilla KAM** (≥ 21-jul-2026)
4. Día 1 del mes — guarda

Resultado: **cero ventas sin fecha real en todos los meses de 2026**. En comparación, el tablero
semanal de ventas del board solo tiene fecha real para entre el 67% y el 91% de las ventas, según
el mes.

Por qué `invoice_created_at` y no `invoice_paid_at`: el 71% se paga el mismo día, pero **9,5%
tarda más de 7 días (máximo 338)** y 117 facturas nunca se pagaron.

### ¿Y si hay meses gratis? Verificado que no ocurre

La objeción es válida en principio: si se regalan 3 meses, la factura llegaría tarde y fecharía
mal la venta. **No se materializa en la data.** Contra el cierre de deal en HubSpot, de 34 ventas
directas: **0 casos** con la factura más de 15 días después del cierre (gap máximo +14 días), y
**0 casos** con `start_date_pago` diferido más de 45 días.

### HubSpot: buena fuente de atribución, mala fuente de fecha

`stg_hubspot__deals` (pipeline `Seguimiento POS Expansión`, stage `Ganado`) arranca en abr-2026 y
cubre **~80% de las ventas directas sin addon** desde may-26. Pero **el deal se marca ganado
después de facturar** (gap hasta −32 días, mediana −22 en jun) y **9 de 34 casos caerían en otro
mes**. No sirve como fecha.

Donde sí es valioso: **atribución**. `kam_pos_owner` / `hubspot_owner_id` es una fuente mucho
mejor de canal que el roster hardcodeado de 6 emails que usa la regla actual.

### Por qué el fallback del mensual no se puede copiar tal cual al semanal

El mensual usa `COALESCE(addon_added_at, month)` → día 1. En el mes correcto da igual, pero en la
semanal **apila todo en la semana que contiene el día 1**: la semana del 01-jun pasaría de 9 a 22
unidades y la del 29-jun de 11 a 33. Ante eso, el board tenía dos opciones: mostrar solo lo
fechado (filtrar — lo que hace hoy) o apilar lo sin fecha en el día 1. Filtrar era lo correcto
entre esas dos; la tercera opción — reconstruir la fecha con la jerarquía de arriba — es la que
usa esta SoT.

### ⚠ Lo que esta corrección deliberadamente NO toca

La regla de `canal` sigue usando la fecha vieja (`sale_day_legacy`) para que el split ISR/KAM
cuadre con el board. Con la fecha buena, la ventana de recencia reclasificaría **19 filas de ISR
a KAM** (5% de las sin-addon). Es una mejora real, pero mueve el gráfico New/Stock: acordarla con
el dueño del tablero antes de aplicarla.

---

## 4. Campos de salida

`company_id · cohort_month · cohort_week · sale_day · fecha_origen · fecha_confiable · nro_evento · fuente ·
units · pais · canal · tipo_venta · tipo_venta_origen · kam_declarado · producto ·
mes_evento_previo · ult_trx_previa · trx_previas · dias_observados`

`producto` viene de `pos_v2.tipo` normalizado a `POS` / `POS+FE` / `POS+FE+SPLIT` — solo
disponible para eventos de la planilla (~30% del total, concentrado en 2026).

---

## 5. Límites (leer antes de reportar)

1. **El GPV es a nivel empresa, no por terminal.** Si el comercio ya tenía otro POS, su GPV
   incluye las máquinas viejas. En reactivaciones esto es dominante por diseño: el GPV que se
   ve **no es incremental** de esa venta. No hay dato por dispositivo en el DWH.
2. **`tipo_venta` solo existe desde jun-2026** y solo en la planilla. Las reactivaciones
   anteriores no se pueden reconstruir.
3. **La derivación de reactivación casi no dispara** (1 caso): una reactivación sin factura y
   sin fila en la planilla no genera ningún evento en ninguna fuente, así que no hay nada que
   derivar. La planilla es irreemplazable para esto.
4. **Ventanas de activación solapadas**: si una empresa tiene dos eventos cercanos, el segundo
   hereda transacciones del primero. Usar `trx_previas` y `ult_trx_previa` para detectarlo.
5. **La planilla es registro manual**: 17 `company_id` sin datos en el DWH y 1 fila sin
   `company_id` (excluida). El `dia` es default 1 hasta el 20-jul-2026.
6. **No corre por el MCP compartido** (~15 s de timeout): la query tarda más. Usar conexión
   directa (venv de dbt + psycopg2 con las credenciales de `~/.dbt/profiles.yml`), que requiere
   VPN.

---

## 6. ⚠ La etiqueta "Reactivación" no es confiable — validarla siempre

Al cruzar las 23 reactivaciones (22 declaradas en la planilla + 1 derivada de la data) contra la
última transacción POS previa al evento:

| Grupo | Eventos | Días en frío (mediana) | GPV M0 | % del GPV M0 del grupo |
|---|---|---|---|---|
| **Frías reales** (≥60 días sin transaccionar) | 15 | 112 d | $5.046 | 6% |
| **No estaban frías** (<60 días; varias 1-4 días) | 5 | 3 d | **$74.064** | **93%** |
| **Nunca transaccionaron** en su vida | 3 | — | $890 | 1% |

Lecturas:

1. **El GPV atribuido a reactivación se concentra en las mal etiquetadas.** Los 5 comercios que no
   estaban fríos aportan el 93% del GPV M0 del grupo — ese GPV no es incremental, el comercio ya
   estaba transaccionando cuando se registró la "reactivación".
2. **Reactivar de verdad es difícil**: las 15 frías reales llevaban una mediana de 112 días sin
   transaccionar y generan casi nada de GPV en el mes del evento. El volumen que reporta el equipo
   es real; convertirlo en GPV todavía no funciona.
3. Va en la misma dirección que `POS_ACTIVACION_REACTIVACIONES.md` §6: de las 14 reactivaciones
   declaradas en jul-26, 5 nunca habían transaccionado y 2 eran POS adicional; 7 califican como
   reactivación real con el criterio de comisiones (frío = 3 meses), y de esas solo 1 registró
   GPV en el mes del evento.

**Regla recomendada**: no usar `tipo_venta='Reactivación'` en crudo. Validar con
`DATEDIFF(day, ult_trx_previa, sale_day) >= 60` y reclasificar el resto a Venta Nueva — sigue siendo
trabajo del equipo, pero no se le atribuye GPV de más. El artefacto trae el control
"Reactivaciones: como las declara el KAM / solo frías reales", y la diferencia es enorme:
GPV M0 del grupo de $80,0K a $5,0K (−94%).

---

# Parte II — La SoT de transacciones

Las ventas (Parte I) dicen *a quién le entregamos una máquina*. Las transacciones dicen *si ese
comercio la usa*. Sobre esta segunda fuente se calculan el GPV y todas las métricas de activación
— y tiene sus propias trampas, que es lo que documenta esta parte.

## 7. De dónde sale el GPV y cuál es la fuente a nivel transacción

El GPV oficial (el de los tableros y el Board Book) se construye sumando dos mundos:

```
gpv_pos = INTERNO + EXTERNO

INTERNO = cobros que pasan por la app de AgendaPro (el "gateway")
          → dwh.propay_transactions, provider_id IN (1,3,6,7,34,35,36,67,68,69,100,166)
            (Transbank, Haulmer, Getnet, Klap, Clip, NetPay, SumUp, OEL)
          = 88-89% del monto · 0% de transacciones sin empresa

EXTERNO = cobros tecleados directo en el terminal, SIN pasar por la app
          → los reporta cada proveedor después, en su archivo de conciliación
          → dwh.{haulmer,sumup,oel}_augmented_transactions WHERE is_external
          = 11-12% del monto
```

Ambos mundos se unen en `dwh.propay_daily_transactions` (grano día × empresa × local) y de ahí
salen los agregados oficiales `dwh.company_sales_days` / `company_sales_months`.

**Regla práctica: cada pregunta tiene su tabla.**

| Pregunta | Fuente correcta | Por qué |
|---|---|---|
| GPV de un comercio / país / mes | `dwh.company_sales_days` / `_months` | Es el agregado oficial: interno + externo ya unidos y deduplicados |
| Análisis a nivel transacción (activación, ticket, prueba de instalación) | `dwh.propay_transactions` con el filtro de providers de arriba | Única tabla con monto exacto y `company_id` siempre presente. Cubre ~89% de las transacciones y el **100% de las internas** — y la transacción de prueba es interna por definición, así que están todas ahí |
| Fees y costos reales por proveedor (P&L) | Tablas `*_augmented_transactions` post-conciliación | El costo real se conoce recién al liquidar |

⚠ **Nunca cuadrar GPV desde las tablas augmented**: no son la base del volumen. Cuando se
intentó (catastro, v1), en CLP dio 110% del agregado — por sumar encima la tabla de Transbank,
que por diseño no alimenta el GPV (existe para el P&L post-liquidación, y así está comentado en
el modelo) — y en MXN dio 85%, porque Getnet no tiene archivo de conciliación y su GPV sale
entero del gateway. La única suma que cierra en ambos países es la de arriba:
`propay_transactions` (88-89%) + externas de conciliación (11-12%).

**Definición de "transacción real"** — la que usan las métricas de activación del weekly:
fila de `dwh.propay_transactions` con el filtro de providers de arriba, **excluyendo la
transacción de prueba de instalación** (§9):

```
dwh.propay_transactions
  WHERE provider_id IN (1,3,6,7,34,35,36,67,68,69,100,166)   -- providers POS del gateway
  -- excluyendo la prueba de instalación (§9):
  --   monto exacto 100 en Chile (CLP) · monto ≤ 1 en México (MXN)
```

⚠ **Decisión consciente**: la activación se mide sobre el uso del **gateway** (la parte interna).
Es la única fuente con la empresa siempre identificada, disponible al día (lo externo llega
después, por conciliación, y arrastraba el hueco de mapeo de §8) y donde la prueba de instalación
es separable. El costo asumido: un comercio que solo cobre tecleando en el terminal no contaría
como activado — se asume marginal en los primeros 30 días de vida; supuesto razonable, aún no
cuantificado.

Documentación oficial del canal de datos en Notion:
[DWH documentation](https://app.notion.com/p/agendapro/DWH-documentaiton-3b2ab34c233b806eaea8cd1d1909a395)

---

## 8. El caso Haulmer: transacciones externas sin dueño (detectado jul-26, corregido ago-26)

El mejor ejemplo de por qué esta SoT necesita mantención y monitoreo, no solo definición.

**Qué pasaba.** Los cobros externos de Haulmer se atribuyen al comercio a través del número de
serie del terminal (`pos_serial`). Cuando el serial no estaba mapeado a una empresa, la
transacción no quedaba "sin dueño": **desaparecía del GPV** (se cae del `GROUP BY company_id`).
Al medirlo en julio-2026: **39% de las transacciones externas de Haulmer sin empresa ≈ USD 257 K
al mes de GPV real que no se estaba contando**, con tendencia a empeorar ~2 puntos por mes. No era
una estimación: es exactamente la brecha entre el archivo de conciliación de Haulmer (USD ~740 K
de externo en julio) y lo que registraba el agregado (USD 480 K).

**La causa raíz fue doble** (diagnóstico 6/7-ago-2026 junto al equipo de Data):

1. **El dato para mapear siempre estuvo llegando.** Haulmer (TUU) envía el RUT del comercio en el
   100% de las filas del archivo de conciliación. Pero el modelo del warehouse tomaba el RUT desde
   un **Google Sheet manual** en vez del archivo — y ese sheet dejó de completarse. El pipeline
   estaba sano; la fuente manual se quedó atrás.
2. Por eso el arreglo fue un **join de respaldo por RUT**: si el serial no está en el sheet, se
   mapea por el RUT que viene en el propio archivo.

**Estado actual — verificado en vivo contra el DWH el 19-ago-2026:**

| Mes 2026 | % externas sin empresa (al catastro, 3-ago) | % hoy | GPV perdido hoy (MM CLP) |
|---|---:|---:|---:|
| enero | 25,1% | 3,8% | 11,0 |
| abril | 35,0% | 4,0% | 12,2 |
| julio | **39,2%** | **5,7%** | 23,1 (≈ USD 25 K) |
| agosto (parcial) | — | 6,8% | 24,2 |

El fix recuperó también el histórico (el respaldo por RUT se aplica hacia atrás), así que las
series de GPV ya quedaron corregidas. Aclaración importante para leer métricas históricas: **el
bug afectaba solo la parte externa del GPV — las métricas de activación nunca se contaminaron**,
porque se calculan sobre las transacciones internas del gateway (§7, §9).

Dos matices de honestidad sobre la cifra original: parte del
monto "perdido" podía no ser nuestro (algunos seriales aparecen con plan TUU sin marca *Partner* —
posibles clientes directos de Haulmer), así que los USD 257 K eran cota superior; y el deterioro
que persiste en filas **internas** sin serial mapeado no afecta el GPV (el interno entra por
`propay_transactions`), pero sí limita el análisis por terminal.

**Lo que queda abierto — por qué esto no está 100% cerrado:**

- El respaldo por RUT es un **parche estructuralmente incompleto para comercios nuevos**: funciona
  cuando el RUT ya existe en el diccionario (típico en segundas máquinas), pero la primera terminal
  de un comercio nuevo trae un RUT que nadie ha mapeado. Medido: cubre el 75% del **monto** del
  backlog, pero solo ~29% de las **terminales nuevas**.
- Por eso el cierre real es de proceso: **volver a mantener la fuente de mapeo y dejar un test
  automático** (alarma de dbt si el % de externas sin empresa supera un umbral). El residuo actual
  (~6% ≈ USD 25 K/mes) es exactamente lo que el parche no puede cubrir.

---

## 9. La transacción de prueba de instalación — por qué se excluye de activación

Al instalar un POS, el técnico hace un cobro de prueba para verificar que la máquina funciona:
**100 CLP exactos en Chile, ~1 MXN en México**. El 99% de las ventas nuevas tiene como primer
movimiento exactamente ese monto.

**Consecuencia si no se excluye**: el hito "≥1 transacción en 30 días" no mide que el comercio
vendió — mide que el técnico encendió la máquina. Sobre las ventas nuevas de 2026, con
transacciones crudas ese hito da **~90%**; excluyendo la prueba, **~54%**. La mediana de días
hasta la primera venta *real* aproximadamente se duplica. Los hitos profundos (≥30 trx) casi no se
mueven, porque 2-9 transacciones de prueba no alcanzan a cruzarlos — el daño se concentra justo en
la métrica que más se mira.

**Cómo se excluye (regla de la SoT):**

- **A nivel transacción** (la forma correcta, la que usa el weekly): en `dwh.propay_transactions`,
  descartar los cobros de **exactamente 100 CLP** en Chile y de **≤1 MXN** en México. Es medible
  porque la prueba la hace el técnico desde la app → siempre es interna → siempre está en esa
  tabla. ⚠ En México el umbral es ≤1 MXN y no 100: **100 MXN es un precio legítimo** de venta.
  La exclusión es permanente — todo cobro de exactamente 100 CLP se descarta, tenga la fecha que
  tenga. El falso positivo (una venta legítima de exactamente $100) es despreciable: el histograma
  de montos en Chile muestra 2.532 cobros de $100 contra 7 de $150 — en la práctica, ese monto
  solo existe como prueba.
- **A nivel agregado** (proxy para series históricas): descartar el *día* cuyo ticket promedio es
  de nivel prueba (≤150 CLP / ≤3 MXN). Se usa cuando se necesita comparabilidad hacia atrás,
  porque las tablas de detalle externas pierden cobertura en el pasado; su límite es que no
  distingue un día mixto de prueba + venta real.

---

## 10. Qué ventas entran al cálculo de activación — y por qué las reactivaciones no

La activación responde: *un comercio que recibió su POS, ¿empezó a usarlo?* Para que la respuesta
sea limpia, la población debe ser comercios que **parten de cero**. Regla de la SoT:

> **Población de activación = "ventas nuevas puras"**: `tipo_venta = 'Venta Nueva'` **y** es el
> primer evento del comercio (`nro_evento = 1`) **y** cero transacciones previas
> (`trx_previas = 0`).

Tres razones, todas medidas:

1. **Una reactivación o recompra no parte de cero.** El comercio ya transaccionaba (o transaccionó
   hace poco), así que "se activó en 30 días" no significa nada: hereda su propia historia. La SoT
   expone `trx_previas` y `ult_trx_previa` justamente para poder filtrar.
2. **Un solo caso contamina una cohorte entera.** Una recompra (empresa con 3.288 transacciones
   previas) aportaba **USD 34,8 K — el 94% del GPV M0 de la cohorte de abril-2026**. Sin el filtro
   de venta nueva pura, la celda decía que la cohorte volaba; con el filtro, era una cohorte normal.
3. **La etiqueta "Reactivación" cruda infla el GPV 94%** (§6): la mayoría del GPV "reactivado"
   venía de comercios que nunca se enfriaron. Por eso una reactivación solo cuenta como tal con
   **≥60 días sin transaccionar** — y aun así se reporta como población separada, nunca mezclada
   con activación.

Esto además es **consistente con cómo se pagan las comisiones**: el motor expulsa las
reactivaciones del lote del KAM antes de calcular activación, y su hito (30 trx acumuladas por
local) nunca se resetea — no existe la noción de "volver a activarse". Ojo: esa compuerta es la
**única** protección contra un re-pago; si se fuerza una reactivación al lote, el motor pagaría
la activación de inmediato apoyado en transacciones de años atrás
(`POS_ACTIVACION_REACTIVACIONES.md` §4). ⚠ Deudas abiertas ahí: conviven dos umbrales de
"activado" (30 trx en el motor de comisiones vs 5 trx en la planilla KAM) y dos definiciones de
"frío" (≥60 días en esta SoT vs 3 meses decidido para comisiones) — está pendiente unificar
ambas.

**La receta completa de la métrica de activación del weekly** (definiciones exactas — otras
variantes no reproducen los números publicados):

| Componente | Definición |
|---|---|
| Población | Ventas nuevas puras de la SoT (Parte I), cohorte = mes del evento, tablas Chile y México separadas |
| Hito | ≥1 / ≥5 / ≥10 / ≥30 transacciones **reales** (§7 y §9: providers del gateway, sin la trx de prueba) |
| Ventana | ≤30 días desde la venta (día 0 a 30 inclusive); solo cohortes con ≥31 días de observación |
| Fuente | `dwh.propay_transactions` |
| Fuera de la población | Reactivaciones (se reportan aparte, validadas con ≥60 días en frío), recompras / POS adicional |

⚠ Las recompras y el POS adicional hoy no tienen métrica propia (deuda abierta): se identifican
en la SoT con `nro_evento > 1` / `trx_previas > 0`, y no deben sumarse ni a activación ni a
reactivaciones.

---

## 11. Cómo seguir

### 11a. La planilla KAM se reemplaza: el deal de HubSpot pasa a ser el registro de la venta

Decisión tomada el 10-ago-2026. La planilla fue clave para descubrir todo lo de la Parte I, pero
es registro manual: sin control de calidad, sin trazabilidad, y con el día exacto de la venta
registrado recién desde el 21-jul-2026. El deal de HubSpot ya lo llenan los KAM en su flujo normal
y trae historial de cambios. Reglas de la migración:

- **Pipeline `82319002` "Seguimiento POS Expansión", etapa Ganado** (el pipeline "Venta POS" es
  legacy 2023-25 y no cuenta).
- **La fecha oficial es la propiedad `fecha_de_venta_pos`**, no el Close Date de HubSpot:
  HubSpot pre-llena Close Date con el último día del mes y solo lo corrige si el deal transiciona
  a una etapa cerrada; medido sobre los deals ganados, el 36% quedó con esa fecha de relleno. Es la misma clase de trampa que `addon_added_at` (§3b): un campo que parece la fecha y
  no lo es.
- **La llave comercio→deal es el `external_id` del contacto asociado** (= company_id de
  AgendaPro), copiado por workflow a `company_id_agendapro`. Medido: 0% de ambigüedad cuando el
  dato existe; el 93% de los deals lo tiene y el resto requiere completarse a mano (hoy el rescate
  es por el ID escrito en el nombre del deal).
- Estado al 19-ago: 9 propiedades creadas y obligatorias al ganar el deal; backfill histórico
  construido (`pos-deal-backfill/`); pendiente importar los ~180 deals históricos restantes y
  exponer las propiedades en dbt.

Para la SoT el cambio es acotado: el deal reemplaza a la planilla como nivel 0 de canal y como
fuente de `tipo_venta`/fecha declarada — la lógica de unión con Chargebee se mantiene.

### 11b. Dónde debe vivir la SoT de ventas

- **Camino corto**: dejar `POS_VENTAS_SOT.sql` como extracción de Evidence
  (`sources/dwh/payments_weekly/`) y reapuntar el gráfico *POS Vendidos por mes (canal)* de
  `fintech/pos-ventas` para que muestre las tres poblaciones.
- **Camino correcto**: modelarla en `agendapro-dbt` como `int_pos_sales_unified`, con tests de
  unicidad `(company_id, cohort_month)` y de cobertura de `tipo_venta`.
- **Roster de vendedores como dato, no como lista en el código** (§3c): planilla maestra
  `email · rol · país · valid_from · valid_to` cargada al warehouse, para atribuir por el rol que
  la persona tenía en la fecha de la venta.
- **Query canónica de la Parte II**: dejar en el repo el filtro de "transacción real" y el cálculo
  de hitos del weekly como `.sql` reproducible, igual que `POS_VENTAS_SOT.sql` lo es para ventas.

### 11c. Proceso, no dato

- **Reactivaciones**: registrar el 100% (hoy la planilla/deal es el único testigo) y validar la
  etiqueta automáticamente al cargarla (≥60 días sin transaccionar, §6) para que deje de inflar
  el GPV.
- **Haulmer**: mantener viva la fuente de mapeo serial→comercio y dejar el test de umbral en dbt
  (§8) — el parche por RUT no cubre comercios nuevos.
- **Unificar la definición de "activado"** entre el motor de comisiones (30 trx) y la planilla KAM
  (5 trx) (§10).

**Artefacto interactivo** (los números de la Parte I, con los controles eventos/unidades y
reactivaciones declaradas/frías):
[Performance del canal POS sobre la fuente única](https://claude.ai/code/artifact/c385cf56-5401-438f-8dd8-e76eb17a4b76).

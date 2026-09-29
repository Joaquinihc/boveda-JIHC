---
type: metodologia
temas: [pos, ventas, payments, activacion]
proyecto: "[[_SoT Ventas POS]]"
fuente: https://claude.ai/code/artifact/201054f6-2379-4028-87f3-e2329d1f8e0a
autor: "[[Ignacio Embry]]"
fecha: 2026-09-23
---

# Reporte SoT ProPay de Ignacio (board book recalculado)

> Artifact de [[Ignacio Embry]]: «Board Book · Payments recalculado con SoT ProPay». Replica el capítulo Payments del board-book de Evidence recalculando cada gráfico con la fuente única de ventas POS (SoT) y las definiciones acordadas. **Datos generados el 2026-09-23**. Todavía **no usa las tablas nuevas** (`dwh.pos_sales_sot` y relacionadas): su fuente de ventas es `POS_VENTAS_SOT.sql` (ver [[SQL — SoT Ventas POS]]). La versión canónica de este texto vive en Notion («SoT Propay temporal», colgada de la doc oficial DWH documentation).
>
> Esta nota copia las páginas **Explicación extendida** y **Cómo actualizar** del artifact (24-sep-2026), para tenerlas como contexto. Comparación de su lámina 5 con el board book: [[20260909-01 POS activated per week — board book vs deck de Ignacio]].

## Láminas del reporte
| # | Lámina | Qué muestra |
|---|---|---|
| 0 | Parque POS | Clientes con POS, % de la base activa, GPV POS, GPV por cliente (toda la base) |
| 1 | Payments Revenue | Ingresos POS + Online por país |
| 1b | GMV por canal | GMV promedio pre-venta de los comercios que compran/reactivan, por canal |
| 1c | GPV por cohorte | GPV M0/M1/M2 por cohorte de venta |
| 2 | POS Sales | Ventas Inbound (New) vs Stock (+ Reactivación) |
| 3 | B2B3 POS Attachment Rate | Unidades POS / clientes nuevos B2B3 |
| 4 | Payments Activation | % de la cohorte que alcanza ≥ T trx en el mes relativo |
| 5 | POS activated per week | Empresas que cruzan 30 trx acumuladas por semana |
| 6 | Payments Retention (+ GPV) | Comercios activados retenidos mes a mes |
| 7 | POS Sales vs Budget 2026 | SoT vs presupuesto |
| 8 | Tarifas vs competencia | Comisión por transacción (estática) |

## Explicación extendida — por qué cambia cada gráfico

### Lámina 0 — Parque POS · toda la base (agregada el 1-sep-2026, pedido de Miguel)
Es la única lámina que no usa la SoT de ventas: mira el parque completo — toda empresa con transacciones POS en el mes, venga de cohortes nuevas, stock, reactivaciones o clientes históricos. Fuente: `dwh.company_sales_months`, grano empresa-mes.
- Cliente con POS = `pos_payment_count > 0` o `gpv_pos_usd_m > 0` en el mes. Base activa (denominador de la línea %) = empresas con `gmv_usd_m > 0` ese mes.
- GPV vs GMV: el GMV aplica la regla oficial de outliers (se excluye todo company-mes con `gmv_usd_m` ≥ USD 1M, guard de FX corrupto de `PROPAY_CONTEXT.md`); el GPV no se filtra — pedido explícito — y el GPV que generan los comercios excluidos se baja aparte (`gpv_de_outliers`): es ~0 en toda la serie, así que la asimetría no distorsiona.
- GPV por cliente: promedio = GPV total / clientes POS; la mediana (misma escala) se calcula sobre el GPV por comercio (`parque_pos_companias.csv`; los pares duplicados company-mes de la fuente se suman, igual que en la lámina 1b).
- USD según la conversión de `company_sales_months` (Chile usa FX fijo de tabla, no dólar corriente): los niveles pueden diferir del USD de mercado, las razones % no.

### 0b · La fuente única de ventas (lo que cambia todo lo demás)
El board arma todas sus vistas de ventas POS desde `dwh.pos_sales2` (Chargebee = facturación). La SoT (`POS_VENTAS_SOT.sql` en el repo raíz) une Chargebee ∪ planilla KAM (`dwh.pos_v2`) con match por empresa + mes (y rescate ±1 mes), porque ninguna fuente ve el universo completo:

| Fuente | Ve | No ve |
|---|---|---|
| `pos_sales2` (Chargebee) | Todo lo facturado (ISR + KAM) | Reactivaciones — solo 18% factura |
| `pos_v2` (planilla KAM) | Ventas del equipo KAM con tipo de venta | ISR por completo |

Además la SoT corrige la fecha de venta: el campo del board (`addon_added_at`) solo existe si el POS se arrienda (venta directa = 5% de cobertura) y en recompras devuelve la compra original (gaps de hasta −676 días). La SoT resuelve en cascada addon del evento → fecha de factura → día de planilla y llega a cero ventas sin fecha real en 2026. Y cambia la atribución de canal: primero ownership (quién cerró la venta: planilla → owner del deal ganado en HubSpot → agente de Chargebee), y la ventana de tiempo queda como último recurso (decide solo ~7% de los eventos). Grano = 1 evento = empresa × mes; las unidades (máquinas) van en la columna `units`.

### 1 · Payments Revenue (POS + Online por país)
No cambia la definición — este recálculo usa la misma fuente del board (`company_sales_months.fee_pos/fee_web` + hardware de `int_pos_terminals_monthly`, FX mensual corriente) y reproduce Evidence al dígito. El punto a comunicar es otro: la corrección de Haulmer ya está dentro de la base. El GPV/fee de las máquinas Haulmer se atribuye a cada comercio vía un mapeo serial→empresa que salía de un Google Sheet que dejó de actualizarse; llegó a perderse el 39% de las transacciones externas (~USD 180–257K/mes de GPV, unos USD 4–5K/mes de fee). El equipo de data implementó el join de respaldo por RUT (la conciliación de TUU trae el RUT al 100%): hoy el `match_type='rut_fallback'` re-atribuye ~200–215 MM CLP/mes y las externas sin mapear quedaron en ~3%. Salvedad: el RUT-fallback recupera sobre todo comercios que ya conocíamos (segunda máquina); para terminales de comercios nuevos sigue dependiendo del alta en la fuente — no reemplaza arreglar el proceso.

### 2 · POS Sales — Inbound (New) vs Stock
Tres diferencias contra Evidence, en orden de impacto:

| # | Cambio | Efecto |
|---|---|---|
| 1 | Fuente: Chargebee ∪ planilla; aparece la categoría Reactivación | +15–17 comercios/mes recientes que el board no ve o fuerza a «Stock»; los totales suben, nada se pierde |
| 2 | Grano: comercios en vez de máquinas (toggle POS para la vista 1:1) | Una empresa con 3 máquinas cuenta 1; multi-máquina explica la mayor parte del delta unidades−comercios |
| 3 | Canal por ownership en vez de ventana de tiempo | México «New» cae fuerte (su New venía del fallback `is_new_merchant` de 92 días que nunca migró a 30); Chile casi no se mueve |

El país acá es `companies.country_name`; el board usa la moneda de la factura — puede diferir en casos borde. La fecha usa la cascada de la SoT: por eso algunas ventas caen en otro mes que en el board (ej. facturas de fin de mes).

### 3 · B2B3 POS Attachment Rate
Qué mide exactamente (definición canónica del MBR): numerador = unidades POS facturadas en el mes a clientes clasificados «New» por la regla del board — de todos los segmentos (B2B3+B2B2+B2C); denominador = clientes nuevos B2B3 del waterfall de software (`revenue_waterfall_software.change_category='new'`). No re-filtra el numerador a B2B3, y el mes del numerador es el de facturación, no el de instalación. El salto de julio en Chile (9% → 24%) es del numerador: 7 → 19 unidades vendidas a clientes nuevos con denominador plano (78 → 78) — coincide con el rebote de venta inbound de julio. Bajo el numerador SoT (comercios ISR por ownership) Chile queda casi igual; México cae a ~1–3% porque su «New» del board es mayormente el fallback de 92 días. Decisión pendiente: si el numerador oficial pasa a comercios-ISR-SoT, el nombre correcto sería «POS attachment de nuevos clientes» y México deja de aparentar 6–8%.

### 4 · Payments Activation — Deep dive
Reconstruido con la SoT y grano empresa. Cambios respecto del board y sus porqués:
- Población «ventas nuevas puras» (1er POS de la empresa y 0 trx previas): una recompra o reactivación ya transacciona — se «activaría» en el mes 1 sin hacer nada. El board las mezcla; acá se excluyen y se reportan aparte (la fila de cada cohorte muestra el neto medible).
- Grano empresa, no local: `pos_sales2` no tiene columna de local; el board infiere qué local activó rankeando por velocidad y cortando en las unidades vendidas. Como ninguna fuente ata máquina↔local, medir por empresa es lo honesto. El toggle POS pondera cada empresa por sus máquinas para acercar la comparación.
- Umbrales 1/5/30 sobre trx brutas + toggle de trx de prueba: al instalar, el técnico hace una trx de ~$100 CLP (o ~$1 MXN). Con umbral 1, «con prueba» mide que la máquina se encendió (≈76% M1); «sin prueba» (se descarta el día con ticket promedio ≤$150 CLP/≤$3 MXN) mide la primera venta real (≈45% M1). El board usa además el filtro «qualifying» (ticket ≥$10.000 CLP/día), que es más agresivo; con 30 trx los números convergen porque las pruebas nunca llegan a 30.
- Cohorte semanal (nueva): mismas reglas, cohortes = semana de venta (16 últimas), columnas = semanas desde la venta, criterio acumulado (≥T trx en ≤7·i días).

Curvas por canal (8-sep-2026): debajo de las curvas por cohorte va el mismo gráfico partido por el canal de la venta según la SoT — KAM (venta a un cliente de la base, «Stock») a la izquierda e ISR (inbound, «New») a la derecha — con los mismos toggles. Los paneles quedan chicos (10-40 comercios por cohorte, menos con el filtro de país), así que una empresa mueve varios puntos y las cohortes que no llegan a 5 comercios con ventana vivida no se dibujan; el n va en la leyenda. El canal inferido se reclasifica a ISR cuando el comercio está en la planilla del equipo.

Filtro de meses (14-sep-2026): las tres curvas dibujan por defecto los últimos 6 meses de cohorte y el control Cohortes permite elegir cuáles mostrar (atajos últimos 6 / 10 / todos, o un chip por mes). La historia disponible llega a 14 meses antes del mes en curso; las trx diarias de esas cohortes viejas se bajan solo para las curvas — las matrices de activación, la semanal, los activados por semana y la retención mantienen su población desde 2025-11 y no cambian con este filtro.

### 5 · POS activated per week
El board fecha la activación por local con suma corrida de trx qualifying dentro del mes relativo en que activó; acá es el día en que la empresa cruza 30 trx acumuladas desde la venta. Sube en semanas recientes por dos vías: la SoT ve más ventas (planilla + venta directa sin addon) y el conteo bruto adelanta el cruce vs el criterio qualifying. Alerta explícita: «POS activados» del board no es un conteo real de máquinas — es la heurística de reparto por local; nuestra vista POS es «unidades de los comercios activados» (cota superior).

### 6 · Payments Retention — Deep dive (+ GPV)
Misma mecánica del board (M1=100%; solo celdas con ventana cerrada cuentan como observadas), con tres cambios: población = empresas activadas de ventas nuevas puras SoT; actividad = ≥30 trx brutas en el mes relativo (toggle de prueba re-fecha activaciones marginales); y la vista POS pondera por máquinas. Se agrega GPV Retention: GPV POS USD (mensual conciliado, `gpv_pos_usd_m`) del mes calendario N vs el mes de activación. El mes de activación es parcial → M2 suele superar 100%; la lectura correcta es la forma de la rampa y dónde se aplana, no «retención <100 = fuga». Panel por celda (espejo de `pos_observed` del board): cada celda N solo suma las empresas cuyo mes N ya cerró, y ese panel crece con el calendario (entran los activados tardíos) — el n de cada celda está en su tooltip. Por eso la última celda de cada cohorte tiene panel chico y cambia de valor cuando cierra un mes sin que la data se haya movido (7-sep-2026: 2026-04 M4 pasó de 23,6 a 144,3 con el GPV mensual idéntico). No es restatement: es la definición heredada; leer las columnas con n cercano a la base «activated».

### 7 · POS Sales vs Budget
El budget 2026 (`dwh.budget_2026_revenue`: «New POS Sold», Inbound + Field Sales/Stock, CL+MX) está en unidades. Contra la SoT en unidades el gap real es menor que el publicado (el board subcuenta ventas); en comercios la línea es la meta implícita. Nota de gobierno: las reactivaciones no existen en el budget — decidir si cuentan para la meta comercial o se reportan aparte (hoy las mostramos como categoría propia).

### Salvedades transversales (para preguntas del board)
- Roster de vendedores hardcodeado: KAM = 6 emails; «ISR» = quien no está en la lista (no existe roster ISR con vigencias). El plan es una planilla maestra email·rol·país·valid_from/valid_to en el warehouse.
- La etiqueta «Reactivación» de la planilla no es 100% confiable: parte de las declaradas no estaban frías (validamos con ≥60 días sin transaccionar, campo `reactivacion_validada` de la SoT). El GPV de reactivaciones no se debe leer como incremental.
- Toggle «Población» (S4 y S5): la población base de activación son las ventas nuevas puras — ISR y KAM por igual; el canal nunca fue filtro. La variante «+ React. frías» agrega las reactivaciones con `reactivacion_validada = 1` (≥60 días sin transaccionar antes del evento), ancladas a la fecha de su reactivación: el reloj de activación se reinicia, y las transacciones previas del comercio no cuentan. La etiqueta de reactivación existe solo desde jun-2026, así que las cohortes anteriores son idénticas en ambas vistas. Una empresa puede aparecer dos veces (compró, se enfrió, se reactivó): son dos eventos con relojes separados, según define la SoT. La retención (S6) no usa este toggle: sigue solo sobre nuevas puras.
- El SoT migra al deal de HubSpot (pipeline «Seguimiento POS Expansión», fecha oficial `fecha_de_venta_pos`, llave `external_id`): cuando el backfill esté en dbt, esta misma vista se recalcula desde ahí sin cambiar definiciones.
- El mes cerrado más reciente avanza el día 5 de cada mes (antes el mes recién terminado sigue asentándose en el DWH); el mes en curso aparece marcado con * y su corte de datos al pie de cada lámina, y las celdas inmaduras van vacías.

### Lámina 8 — Tarifas vs competencia
Comparación estática (no sale del DWH): costo de comisión por transacción al ticket elegido con el slider (default $39.000, ticket promedio POS). ProPay cobra solo porcentual — 0,99% débito / 1,79% crédito, sin cargo fijo ni arriendo. Los competidores comparados cobran porcentual + cargo fijo por transacción expresado en UF:

| Adquirente | Débito | Crédito |
|---|---|---|
| Transbank tarifario antiguo | 0,69% + 0,00050 UF | 1,60% + 0,00051 UF |
| Getnet POS (Smart/Móvil) | 0,58% + 0,001668 UF | 1,46% + 0,001792 UF |

Se compara contra el tarifario antiguo de Transbank porque es el que conserva toda la base afiliada antes del 20-may-2026; desde esa fecha Transbank cobra a comercios nuevos un porcentual puro (1,75% débito / 2,35% crédito, mínimos 0,002260 / 0,003515 UF), más caro que ProPay en ambos medios. Tarifas netas (+ IVA) y referenciales por rubro (MCC). UF usada: $40.858 (19-ago-2026), hardcodeada como `const UF` en el template — actualizarla al recalcular. No incluye arriendos mensuales de equipo (Transbank 0,35–0,55 UF/mes; Getnet cobra arriendo en UF; ProPay no tiene).

## Cómo actualizar este tablero
Cualquier persona puede refrescarlo entregándole esta página a Claude (Claude Code) junto con el prompt de abajo. Las instrucciones también viven en Notion: SoT Propay temporal.

### Fuentes de datos (todas en Redshift, db dwh)
| Qué | Fuente | Nota |
|---|---|---|
| Eventos de venta (SoT) | `Repos/POS_VENTAS_SOT.sql` — corre tal cual | Une `dwh.pos_sales2` ∪ `dwh.pos_v2`; salida clave: `cohort_month, sale_day, units, pais, canal, tipo_venta, venta_nueva_pura` |
| Transacciones diarias | `dwh.company_sales_days` (`pos_payment_count`, `gpv_pos`) | Día de prueba = ticket promedio ≤ $150 CLP / ≤ $3 MXN |
| GPV mensual USD | `dwh.company_sales_months.gpv_pos_usd_m` | Solo meses cerrados |
| Revenue por país | `company_sales_months.fee_pos/fee_web` + `int_pos_terminals_monthly` + `int_exchange_rates_unpivoted` | Idéntico a `sources/dwh/payments/country_pos_online_revenue.sql` de Evidence; FX corriente |
| Attachment | `pos_sales2` (numerador) + `revenue_waterfall_software` × `int_company_dimensions` (denominador B2B3) | Réplica de `pos_attachment_segment.sql` |
| Budget | `dwh.budget_2026_revenue` (métrica «New POS Sold», canales Inbound / Field Sales / Stock) | Meses como columnas — despivotear |
| Check Haulmer | `dwh.haulmer_augmented_transactions` (`pos_company_match_type`) | `rut_fallback` = lo re-atribuido por el fix |

### Pipeline reproducible (ya está escrito)
```bash
cd /Users/agendapro/Desktop/AgendaPro/Repos/pos-boardbook-sot
# 1. extraer (requiere VPN; usa ~/.dbt/profiles.yml)
agendapro-dbt/.venv/bin/python sot_boardbook_pull.py     # → pulls_direct/*.csv
agendapro-dbt/.venv/bin/python pull_extras.py             # → gmv_gpv_sot_monthly.csv + gmv_total_monthly.csv (S1b/S1c)
# 2. computar datasets
python3 build_datasets.py                                 # → boardbook_data.json
# 3. inyectar al HTML y publicar
python3 inject.py                                         # → boardbook_sot.html
```
Los tres scripts + el template viven en `Repos/pos-boardbook-sot/` (en el computador de Ignacio). El paso 3 reemplaza `__DATA__` del template por el JSON nuevo; después se republica el artefacto sobre la misma URL.

### Prompt sugerido para Claude (del reporte)
> Actualiza el artefacto "Board Book · Payments recalculado con SoT ProPay" (mantén la misma URL). Pipeline en Repos/pos-boardbook-sot/: corre sot_boardbook_pull.py con el venv de dbt (necesita VPN), luego build_datasets.py, luego inject.py, y republica boardbook_sot.html. Si cambió POS_VENTAS_SOT.sql, úsalo tal cual está en el repo raíz. Actualiza también los valores "Evidence hoy" sacándolos del board-book si cambiaron (pantallazos o re-corriendo las queries de sources/dwh/payments/ del repo agendapro-dashboards-evidence).

### Definiciones que NO hay que cambiar sin acordarlo
- Población de activación/retención = `venta_nueva_pura = 1` (1er POS y 0 trx previas).
- Activación mensual = ≥ T trx dentro de un mes relativo (anclado al día de venta); semanal y «per week» = T acumuladas.
- Trx de prueba = día con ticket promedio ≤ $150 CLP / ≤ $3 MXN (los dos estados se precalculan; el toggle solo alterna).
- Madurez: una celda se muestra solo si su ventana cerró (mensual: cohorte+rel+1 ≤ hoy; GPV: mes calendario ≤ último cerrado).
- Categoría Reactivación = declarada en planilla (`tipo_venta='Reactivacion'` origen sheet).

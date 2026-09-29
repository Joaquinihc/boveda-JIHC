---
type: analisis
proyecto: "[[_SoT Ventas POS]]"
estado: activo
fecha: 2026-09-24
temas: [pos, ventas, dwh]
metricas: []
---

# Auditoría de casos borde de las tablas SoT de ventas POS

**Pregunta**: ¿Qué casos borde de `dwh.pos_ncro`, `dwh.pos_resales` y `dwh.pos_sales_sot` hay que arreglar (o decidir) antes de que `pos_sales_sot` sea la única fuente de los gráficos de ventas POS, y cómo cuadra hoy, compañía por compañía, con lo que muestra el board book de Evidence en jul–sep 2026?

Acción de [[2026-09-24 Reunión — Tablas SoT de ventas POS (Ignacio)]]. Referencia de las tablas: [[Tablas SoT de ventas POS (pos_sales_sot, pos_ncro, pos_resales)]]. Contexto: [[20260924-01 Métricas POS del board book vs tablas SoT]], [[20260910-01 Cuadre ventas POS ISR jul–sep 2026]] (con su nota de corrección del 24-sep), [[20260909-01 POS activated per week — board book vs deck de Ignacio]]. Tarea: [[Locations POS y reactivaciones]] (T-053).

**Corte de los datos**: tablas recreadas el 24-sep-2026 entre 17:11 y 17:12 (hora Chile; `pos_sales` 17:10, `pos_sales2` 17:11, `propay_transactions` 16:36, `pos_cl_form_assignments` 16:20, `int_chargebee__subscription_addon_changes` 15:50, según `pg_class_info`). Consultas SELECT del 24-sep entre las 18:03 y las 18:33 (hora Chile). Filas al corte: 2.292 / 2.019 / 273, igual que el perfil de las 17:31. Código leído en `agendapro/agendapro-dbt`, rama `refactor/postgres-to-redshift`, y `agendapro/agendapro-dashboards-evidence`, rama `main`. Documentación leída: Notion *purchase_attribution_model* (editada el 23-sep 17:52 UTC).

## Conclusión
> - **Todavía no puede ser la única fuente.** En jul–sep el total se parece (Chile 171 u en el board vs 176 en la SoT; México 56 vs 45), pero se parece por compensación: la SoT suma ecos de `addon_restart`, ventas de Tabata y reventas que el board no ve, y resta ventas que todavía no validan.
> - **Bloquean tres errores de query**: `addon_restart` cuenta como venta los cambios arriendo → cuotas y las bajas, y los cuenta dos veces por un `BETWEEN` inclusivo (22 filas válidas y 25 u en 2026, ninguna suma un terminal); `qty_up` guarda el total y no el incremento (30 u contra 16 reales); y el canal ISR es quien cargó el addon (barbara@ en 143 de 171 filas ISR), mientras la factura (`pos_sales.sales_user`) trae al vendedor real y podría sacar de `Others` 60 de las 84 primeras ventas de 2026.
> - **Bloquean dos decisiones**: la validez no es estable (septiembre queda con 27 inválidas, 20 de ellas con la ventana del formulario todavía abierta, y el corte del 1-sep hace que ene–ago no sea comparable: el 84% de las válidas KAM/Others de Chile no tiene formulario); y la fecha oficial (la SoT usa el pago; Finanzas acordó la emisión el 7-sep).
> - **Además hay que decidir**: cantidad deal vs factura (6 eventos), Tabata (15 u en 2026), soft sin 15 trx o sobre comercios activos (9 y 13 de 26) y ventas revertidas con factura pagada (15 candidatas).
> - **Ya está bien**: no hay llaves duplicadas, el país coincide con la moneda de la factura en todas las filas, 569393 es ISR y el caso de México 557935 queda en una sola fila de 1 u.

## Hallazgos

17 hallazgos: 5 cambian la query, 6 son decisiones de definición, 3 son dato de origen y 3 son solo para documentar. Impacto en 2026, por fecha declarada, salvo que se indique otra cosa.

| # | Caso | Clasificación | Impacto 2026 (filas / compañías / unidades) | Ejemplos |
|---|---|---|---|---|
| 1 | `addon_restart` cuenta swaps y bajas como venta, con un eco al mes siguiente | Cambia la query | 42 filas (CL 32, MX 10), 33 compañías; 22 válidas = 25 u (CL 20, MX 5). 14 filas son eco | 494549, 38388, 1566, 158955 |
| 2 | `qty_up` guarda la cantidad total, no el incremento | Cambia la query | 13 filas válidas, 13 compañías: 30 u contra 16 de incremento (+14: CL +10, MX +4) | 32381, 104127, 498146 |
| 3 | ISR = usuario que cargó el addon; `sales_user` de la factura no se usa | Cambia la query (interino) | ISR: 171 filas, 143 con barbara@ (113 con otro vendedor en la factura). Others NCRO: 84 filas (CL 53, MX 31), 61 u válidas; 60 tienen vendedor ISR en la factura (48 u) | 562659, 563154, 504406, 535145 |
| 4 | Validez inestable y corte de reglas del 1-sep | Decisión de definición | Sep: 27 inválidas (CL 18 de 54, MX 9 de 16), 20 con ventana abierta. Ene–ago CL: 148 de 176 válidas KAM/Others sin formulario. Pagadas inválidas: CL 39, MX 22 | 529415, 557935, 360327, 70530 |
| 5 | Fecha oficial: pago (SoT) vs emisión (acuerdo del 7-sep) | Decisión de definición | 19 válidas cambian de mes entre fecha declarada y efectiva (CL 14 = 23 u, MX 5 = 5 u); 8 eventos del board caen en otro mes en jul–sep | 311692, 416679, 553259, 120849 |
| 6 | Deal > factura y ventana deal ↔ factura de ±45 d | Decisión de definición | 19 filas KAM con factura antes del deal (MX 16, CL 3), 5 a más de 15 d; 12 con deal y factura en meses distintos; 4 NCRO fechados por trx antes de la factura | 529415, 557935, 120849, 531844 |
| 7 | Cantidad: manda el deal (código) vs la factura (reunión) | Decisión de definición | 6 eventos KAM; si manda la factura: +7 u (sumando facturas) o +6 u (con máximo) | 388767, 84364, 518906 |
| 8 | Reactivación soft: meta de 15 trx solo desde sep, comercios activos y soft con factura | Decisión de definición | 26 filas (jul 9, ago 13, sep 4). 9 válidas con menos de 15 trx; 13 con menos de 30 d sin actividad; 1 con factura pagada | 506857, 459098, 501961 |
| 9 | Ventas revertidas con factura pagada | Dato de origen | 15 filas / 15 compañías / 16 u válidas con baja del addon a 1–45 d, sin re-alta y con menos de 15 trx (CL 14, MX 1) | 562659, 563154, 477722 |
| 10 | Tabata: la SoT la incluye, el board la excluye | Decisión de definición | 12 filas, 10 compañías; 9 válidas = 15 u (KAM 4 u, Others 11 u); +11 u en CL jul–sep | 32381, 104127, 38388 |
| 11 | Primera venta ISR partida en dos filas cuando el POS transacciona más de 45 d antes de la factura | Cambia la query | 1 caso en 2026 (452621) y 1 en 2025 (339880): NCRO ISR inválida + reventa Others válida | 452621, 339880 |
| 12 | Llaves, facturas compartidas y company_id de relleno | Cambia la query (menor) | 0 llaves duplicadas; 1 caso de facturas compartidas (19517, 2024, 3 + 1 u); `company_id` −1 y 0 en `pos_ncro` | 19517 |
| 13 | Deals KAM ganados sin factura | Dato de origen | 10 filas (CL 8, MX 2), todas inválidas por `no_invoice` | 7743, 430053, 430263 |
| 14 | Mapping de roles: KAM con rol ISR/MDR y addons cargados por API | Dato de origen | rocio@ figura ISR-2 y janet@ MDR_MX en `hubspot_mapping`; 19 NCRO Others con addon de `full_access_key_v1` (MX 13, CL 6) | 554317, 557751, 544602 |
| 15 | `purchase_type = 'new'` no es lo mismo que primera venta | Solo documentar | 9 reventas KAM con deal «Venta Nueva» (CL 5, MX 4), 5 válidas = 6 u; 1 NCRO con deal «Reactivación» | 487289, 170863, 529431 |
| 16 | Compra MX pasada a arriendo con abono | Solo documentar | 1 fila, 1 u en la SoT; el board la cuenta 2 u | 557935 |
| 17 | País por `companies` vs moneda de la factura | Solo documentar | 0 diferencias en 1.882 filas con factura; 4 filas fuera de CL/MX, ninguna de 2026 | 78605, 201822 |

### 1. `addon_restart` cuenta swaps y bajas como venta (cambia la query)
- **Evidencia.** El CTE `rent_restart` (idéntico en `pos_ncro.sql` y `pos_resales.sql`) marca como venta una línea de renta si hubo cualquier baja de addon POS `BETWEEN l.prev_line_date AND l.invoice_created_at`. No exige un alta después de la baja y el `BETWEEN` incluye el día de la renta anterior. En 2026 hay 42 filas `addon_restart`: 20 son swaps del mismo día (`added smart-pos-cuotas` + `removed smart-pos-arriendo`; en MX, `terminal-getnet-renta` → `terminal-oel-renta`), 14 son bajas sin alta y 8 tienen otro patrón. En ninguna sube la cantidad frente a la renta anterior.
- **El eco.** En 14 filas (CL 12, MX 2; 8 válidas = 10 u) la única baja cae justo el día de la renta anterior, así que es la misma baja que ya abrió la fila del mes anterior. 494549: swap el 20-jul, línea de cuotas el 20-jul (fila válida) y línea de cuotas el 20-ago (otra fila válida); la del 20-sep ya no dispara. Lo mismo pasa con 38388 (jun y jul), 11787, 373415, 453116 y 520667 (ago y sep), 1566 (dos filas en mar) y 158955 (dos en feb, MX).
- **Impacto.** 22 filas válidas = 25 u en 2026: CL ene 3 u, feb 1, mar 2, may 1, jun 2, jul 2, ago 9; MX ene 1, feb 3, abr 1. En septiembre las 9 filas `addon_restart` de Chile son inválidas, pero no por la regla: a 7 les falta el formulario (`form_not_sent`) y 2 además están impagas. En el cruce con el board, 494549 suma 1 u de más en agosto (eco) y las 5 conversiones del cuadre del 10-sep siguen contando como `pos_additional` válidas.
- **Por qué importa.** Infla `pos_additional` y, antes de septiembre, casi todo valida. 75776 y 490058, que el PR #396 usaba como control de «swap no es venta», hoy tienen fila.
- **Propuesta** (en los dos modelos; esbozo, sin probar en dbt):
```sql
pos_addon_chg AS (
    SELECT company_id, occurred_at::date AS d, change_type, addon_id
    FROM {{ ref('int_chargebee__subscription_addon_changes') }}
    WHERE addon_id ~* 'pos|getnet|terminal' AND change_type IN ('added', 'removed')
),
swap_days AS (   -- baja y alta de otro addon POS el mismo día = cambio de forma de pago, no churn
    SELECT DISTINCT r.company_id, r.d
    FROM pos_addon_chg r
    JOIN pos_addon_chg a ON a.company_id = r.company_id AND a.d = r.d
                        AND a.change_type = 'added' AND a.addon_id <> r.addon_id
    WHERE r.change_type = 'removed'
),
rent_restart AS (
    SELECT DISTINCT l.invoice_id
    FROM pos_invoice_lines l
    JOIN pos_addon_chg rm ON rm.company_id = l.company_id::varchar AND rm.change_type = 'removed'
                         AND rm.d >  l.prev_line_date          -- antes: BETWEEN (incluía el día de la renta anterior)
                         AND rm.d <= l.invoice_created_at
    JOIN pos_addon_chg ad ON ad.company_id = rm.company_id AND ad.change_type = 'added'
                         AND ad.d >  rm.d AND ad.d <= l.invoice_created_at   -- re-arriendo = alta después de la baja
    LEFT JOIN swap_days sw ON sw.company_id = rm.company_id AND sw.d = rm.d
    WHERE l.is_recurring AND l.prev_line_date IS NOT NULL
      AND sw.company_id IS NULL
)
```
  Avisar a Ignacio: cambia filas de toda la historia (74 filas `addon_restart`, 49 válidas = 56 u).

### 2. `qty_up` guarda la cantidad total (cambia la query)
- **Evidencia.** En `pos_resales`, `invoice_sales.pos_quantity = MAX(item_qty)` del grupo. Para `qty_up`, `item_qty` es la cantidad total de la suscripción. En 2026 hay 13 filas `qty_up`, todas válidas: suman 30 u y el incremento real contra la renta anterior es 16. CL: 13232, 401246, 410483, 478925, 481969, 21064 y 528199 (+1 cada una, cuentan 2–3); 32381 y 104127 (Tabata, 3 contra +2); 416679 (2). MX: 444582, 498146 (3 contra +1) y 3001 (2 contra +1).
- **Por qué importa.** Suma 14 u de más en 2026 (CL +10, MX +4). 32381 llega a 7 u para unos 3 terminales, si se suman el deal de junio, `qty_up` y `addon_restart` (el `addon_restart` del 31-ago es además un eco del swap del 20-ago).
- **Propuesta:**
```sql
-- pos_resales.sql, invoice_sales
MAX(CASE WHEN line_evidence = 'qty_up' THEN item_qty - prev_line_qty ELSE item_qty END)::int AS pos_quantity
```

### 3. ISR = usuario que cargó el addon; la factura trae al vendedor (cambia la query, interino)
- **Evidencia.** `ncro_agent` toma el usuario Chargebee que agregó el addon a ±31 d de la primera factura y lo cruza con `hubspot_mapping`. En 2026, 143 de las 171 filas ISR son de barbara@ (todas en Chile), justo lo que la reunión dijo que no sirve. `pos_sales.sales_user` sí trae al vendedor: en esas 143 filas, 113 tienen otro nombre (Daniela Suarez 47, Julializ Riobueno 26, Naily Vasquez 22, Monika Andrade 12…). Ejemplos: 562659 y 563154 (addon barbara@, factura Daniela Suarez) y 569917 (factura Julializ Riobueno).
- **Others.** Las 84 primeras ventas Others de 2026 (CL 53, MX 31; 61 u válidas: CL 40, MX 21) no tienen deal ni addon ISR: 36 no tienen usuario de addon, 19 son de `full_access_key_v1` y 29 son de usuarios que no están en el equipo ISR del mapping (leonardoruiz, alexander, matias, rocio…). **60 tienen en la factura un vendedor del equipo ISR/AE/MDR del mapping** (CL 33, MX 27; 48 u válidas; 2 más son de Silcris, AM): 504406 y 535145 (Barbara Sanchez), y en México Tonatiuh Gonzalez, Pablo Molina y Jorge Bribiesca. `hubspot_mapping` tiene `user_name`, así que el cruce por nombre es directo.
- **Por qué importa.** En jul–sep, 15 u que el board tiene como New en Chile (y 6 en México) quedan en Others. El attachment B2B3 y la barra Inbound dependen de este canal. La propia doc de Notion dice que `sales_user` «es la única pista y no se usa aún».
- **Propuesta** (interina, hasta «contra topos»): en `pos_ncro`, si no hay deal KAM ni addon ISR, usar el `sales_user` de las facturas del NCRO cuando cruce con `hubspot_mapping.user_name` en el equipo ISR/AE/MDR y no esté en el roster KAM. Y exponer `channel_source` (`deal` / `addon_user` / `invoice_sales_user`) para auditar.
```sql
ncro_invoice_seller AS (
    SELECT i.company_id, MAX(LOWER(m.email)) AS seller_email
    FROM ncro_invoices_lines i            -- líneas de las facturas del NCRO
    JOIN {{ ref('stg_google_sheets__hubspot_mapping') }} m ON LOWER(m.user_name) = LOWER(i.sales_user)
    WHERE m.role IN ('AE','ISR','ISR-1','ISR-2','MDR') OR m.equipo_regla_comisiones SIMILAR TO '(ISR|MDR|AE)%'
    GROUP BY 1
)
-- channel_attribution: KAM si deal · ISR si addon ISR o ncro_invoice_seller no KAM · si no Others
```

### 4. Validez inestable y corte de reglas del 1-sep (decisión de definición)
- **Septiembre.** 27 de 70 filas son inválidas (CL 18 de 54, MX 9 de 16). 20 tienen el formulario todavía dentro de plazo (ventana CL de −15/+60 d, MX de −15/+90 d): CL 13, 3 de ellas además impagas, y MX 7. En México, las 7 pagadas con `form_not_completed` (529415, 458124, 557935, 574554, 580562, 544651, 450004) no tienen fila en *Control terminales* ni trx todavía.
- **Pagadas marcadas inválidas en 2026**: CL 39 filas / 39 u (sep 11), MX 22 / 24 u (sep 7). La mayoría son KAM/Others con factura pagada y sin formulario o sin trx.
- **Antes del 1-sep no es comparable.** El formulario POS CL existe desde may-26 (2, 6, 21, 83 y 39 asignaciones por mes, de mayo a septiembre). De las 176 ventas válidas KAM/Others de Chile en ene–ago, 148 (84%) no tienen formulario: con la regla de septiembre serían inválidas. 22 de ellas validan sin formulario y sin trx reales, solo con la prueba de instalación (`has_any_trx_post` cuenta la trx de 100 CLP). En México, 11 de 104 no tienen fila en *Control terminales*.
- **Restatement.** La doc de Notion dice que las ventas efectivas «no se restatean hacia atrás», pero `is_valid_purchase` sí cambia: una venta pagada en septiembre valida cuando llega el formulario, hasta 60–90 d después. La cifra del mes sube después del cierre.
- **Por qué importa.** Septiembre no es comparable con agosto, y lo que se muestre del mes en curso va a cambiar.
- **Propuesta.** Decidir: (a) si el gráfico usa `is_valid_purchase` o «pagada», con la validez como checklist aparte; (b) si se congela el mes al cierre o se acepta el restatement; (c) si se aplica la regla estricta hacia atrás (en Chile no se puede: no hay formularios antes de mayo) o se marca el quiebre de serie en el gráfico. Mientras tanto, exponer `is_pending` (ventana abierta) para no mezclar «todavía no» con «no».

### 5. Fecha oficial: pago vs emisión (decisión de definición)
- **Evidencia.** `effective_sale_date` es el primer pago de las facturas del evento (Notion, 21-sep: «fecha oficial = pago»). El hub del proyecto registra lo acordado el 7-sep, «fecha de venta = emisión de factura», y el board fecha por cambio de addon → emisión. En 2026, 19 ventas válidas caen en otro mes entre la fecha declarada y la efectiva (CL 14 filas = 23 u, MX 5 = 5 u), y 9 tienen la fecha efectiva *antes* de la declarada (CL 3, MX 6; p. ej. 2590: deal 1-sep, pago 21-ago).
- **En jul–sep**, 8 eventos del board caen en otro mes en la SoT: 553259, 554050 y 554347 (jul → ago), 311692 (declarada 20-ago, pagada 15-sep), 416679 y 570598 (ago → sep), 120849 y 529415 (MX, ago → sep). En Chile el neto del trimestre es 0, pero mueve meses sueltos (jul −4, sep +4).
- **Propuesta.** Una sola regla escrita (pago o emisión) para board, budget y comisiones. Si gana el pago, actualizar el hub y avisar a Payments; si gana la emisión, la SoT necesita una tercera fecha (`invoice_created_date` ya existe).

### 6. Deal > factura y ventana de ±45 d (decisión de definición)
- **Evidencia.** Con deal, `declared_sale_date` es la `fecha_de_venta_pos` aunque la factura sea anterior. En 2026 hay 19 filas KAM con factura antes del deal (MX 16, CL 3), 5 de ellas a más de 15 d: 529415 (−25 d), 557935 (−32), 120849 (−30), 541887 (−16) y 529431 (−16). Así 529415 y 120849 pasan de agosto (board) a septiembre, y 529415 y 557935 quedan bajo las reglas estrictas aunque la factura es de agosto. Hay 12 filas con deal y factura en meses distintos (CL 5, MX 7) y 19 deals fechados el día 1 (CL 15, MX 4; 2590 y 543563 lejos de su factura).
- **Ventana.** La reunión habló de «~45 d hacia adelante»; el código usa ±45 d. Si fuera solo hacia adelante, esas 19 facturas quedarían fuera del deal: el deal sería `no_invoice` y la factura una fila Others aparte (doble conteo en México). La ventana hacia atrás hace falta.
- **Trx antes de factura.** Sin deal, el NCRO se fecha por el evento más temprano, no por la factura: 4 NCRO de Chile en 2026 quedan fechados por una trx anterior a la factura (531844 13 d, 448888 10 d, 528843 8 d, 471138 2 d).
- **Propuesta.** Mantener ±45 d y documentarlo. Decidir si la fecha declarada KAM es el deal o la factura cuando la factura es anterior. Pedirle a KAM MX que marque el deal ganado con la fecha real de venta (dato de origen).

### 7. Cantidad: deal vs factura (decisión de definición)
- **Evidencia.** `COALESCE(deal, MAX(item_qty), 1)`: manda el deal. En 2026 hay 6 eventos KAM donde difieren: 388767 (deal 1 / factura 3), 84364 (1 / 3; el board le pone 3), 471474 (1 / 2), 2590 (1 / 2), 417441 (1 / 2) y 518906 (deal 2 / dos facturas de 1 cada una).
- **Por qué importa.** Si manda la factura, la SoT suma 7 u (o 6 si se toma el máximo por factura, que en 518906 daría 1 y estaría mal). La regla de la reunión necesita precisar si se suma entre facturas de compra y se toma el máximo en rentas.
- **Aparte, a favor de la SoT**: el board toma la cantidad de `pos_sales2`, que en 29705 (1 contra 2 en factura y deal) y 162282 (2 contra 3) queda corta. Duda abierta: en `smart-pos-cuotas`, ¿`item_qty` son terminales? Todo indica que sí (32381: 3 en cuotas tras 1 en arriendo).

### 8. Reactivación soft (decisión de definición)
- **Evidencia.** 26 filas, todas KAM (CL 24, MX 2): jul 9, ago 13, sep 4. Antes del 1-sep valen como se declararon: 9 son válidas con menos de 15 trx hasta el plazo (0 trx: 338864, 203659, 506857, 3451, 621, 161984, 190960; 2 en 9223; 5 en 23445). 13 de 26 se hicieron sobre comercios que tuvieron actividad POS menos de 30 d antes (459098, 120798 y 19534 con 1 d; 506857 con 1 d y 0 trx después). 501961 es soft con facturas pagadas (678653, 680248) y formulario completo. En 1724 la fecha efectiva es la declarada (1-sep), aunque llegó a 17 trx después. 417761 valida justo con 15.
- **Por qué importa.** El board cuenta 1 u por soft (planilla) y la SoT 0. Y la definición de reactivación del 7-sep (comercial = 60 d sin actividad) no se aplica: la SoT solo expone `days_inactive`.
- **Propuesta.** Decidir: (a) si las soft de jul–ago se revalidan con 15 trx o se quedan como declaradas; (b) si se exige frialdad (≥ 60 d) para llamarlas reactivación; (c) fecha efectiva = día en que llega a 15 trx. Corregir 501961 en HubSpot (cantidad ≥ 1) o tratar como `reactivation_with_purchase` toda soft con factura pagada.

### 9. Ventas revertidas con factura pagada (dato de origen)
- **Evidencia.** 562659 y 563154 siguen como ISR `new` válidas. Con el mismo patrón (venta válida de 2026, baja del addon POS entre 1 y 45 d después, sin re-alta y con menos de 15 trx reales) hay 15 filas / 16 u: CL ISR 473906, 476204, 477722, 484027, 507671, 516931, 521611, 530479, 548686, 553259 (2 u), 559296, 562659, 563154; CL Others 362886; MX ISR 462569. 484027 reaparece como reactivación en julio.
- **Por qué importa.** Si son reversiones, sobran unidades en ISR. No se puede saber sin Payments: puede que la venta ocurriera y lo que falle sea el addon.
- **Propuesta.** Llevar la lista a Payments (subtarea 7 de T-053). Si se confirman, hay dos opciones: marcar la factura en `pos_omitted_invoices` o agregar `invalid_reason = 'addon_removed_no_use'`.

### 10. Tabata (decisión de definición)
- **Evidencia.** El roster KAM de la SoT tiene 9 emails e incluye a Tabata; el board la excluye (acuerdo Payments/Ignacio, ago-26). En 2026 hay 12 filas suyas (10 compañías), 9 válidas = 15 u: KAM 4 u (388767, 45612, 471474, 417441) y Others 11 u (38388, 104127, 32381, casi todo swaps y `qty_up`). En jul–sep suma 11 u a la SoT de Chile que el board no tiene, y dos reactivaciones en julio.
- **Propuesta.** Decidir si se excluye en la SoT (`seller_email`) o si el board la incorpora. Si entra, primero hay que arreglar los hallazgos 1 y 2, que explican la mayor parte de sus unidades.

### 11. Primera venta ISR partida en dos filas (cambia la query)
- **Evidencia.** Si el POS transacciona más de 45 d antes de la primera factura, el NCRO queda fechado por la trx y sin factura (inválido, `no_invoice`), y la primera factura entra a `pos_resales` como `first_invoice` → `pos_additional` Others (en reventas nunca hay ISR). 452621: trx el 10-dic-25, factura el 18-feb-26 (70 d), addon agustin@. 339880: 56 d en 2025, addon barbara@. En toda la historia hay 9 filas `first_invoice` (2 en 2026).
- **Propuesta.** Cuando `ncro_evidence = 'trx'` y la primera factura llega después de la ventana, que el NCRO tome la primera factura (o ampliar la ventana del NCRO a 90 d solo para trx). Impacto chico, pero cambia el canal de primeras ventas.

### 12. Llaves, facturas compartidas y company_id de relleno (cambia la query, menor)
- **Evidencia.** 0 `sale_id` repetidos y ningún deal del NCRO repetido en `pos_resales`. En 2026 no hay pares de deals KAM a 45 d o menos. El único caso de facturas compartidas es 19517: dos deals (ago y sep-2024, 3 y 1 u) absorben las facturas 264985 y 275928, y las dos filas son válidas. Hay 2 comercios con dos filas el mismo mes (1566, eco del hallazgo 1; 498146) y 11 comercios con 2 o más filas `new` válidas (sobre todo reventas «Venta Nueva», hallazgo 15). `pos_ncro` trae `company_id` −1 y 0 (trx sin comercio, 2021–22).
- **Propuesta.** Agregar un test de que un `invoice_id` aparece en una sola fila (el deal más cercano se queda con la factura) y filtrar `company_id > 0`. No cambia 2026.

### 13. Deals KAM ganados sin factura (dato de origen)
- **Evidencia.** 10 filas KAM de 2026 sin factura (CL 8, MX 2). 7743 y 430053: deal «Venta Nueva» fechado el día 1 sobre una renta que ya se cobraba (facturas el 23 de cada mes, fuera de la ventana). 430263, 490565 y 364789: sin ninguna factura POS. 144708 y 131134: reactivación con máquina sin factura. 10119 (Tabata), 381847 y 317361: deal sin factura en la ventana.
- **Propuesta.** Lista para KAM: facturar o corregir el deal. La regla de la SoT está bien (quedan inválidas).

### 14. Mapping de roles y addons cargados por API (dato de origen)
- **Evidencia.** `stg_google_sheets__hubspot_mapping` tiene a rocio@ como ISR-2 y a janet@ como MDR_MX. Hoy el roster KAM los rescata, pero cualquier KAM fuera del roster caería en ISR. 19 primeras ventas de 2026 tienen el addon cargado por `full_access_key_v1` (MX 13: 554317, 557751, 567096, 568486…; CL 6: 544602, 511752…), sin persona. Vendedores que cargan addons sin estar en el mapping: juanmorales, alexander, leonardoruiz, yeilyn, pedro, cristobal, nicolascontador.
- **Propuesta.** Completar el mapping con rol y vigencia (el `pos_sales_roster` con `valid_from` / `valid_to` que ya estaba pendiente) y averiguar qué integración usa la API key en México.

### 15. `purchase_type = 'new'` no es primera venta (solo documentar)
- **Evidencia.** 9 reventas KAM de 2026 con deal «Venta Nueva» (CL: 7743, 430053, 341790, 10119, 487289; MX: 370509, 161934, 25280, 170863), 5 válidas = 6 u. El board las tiene como Stock (487289 en ago, 170863 y 25280 en jul). Al revés, 529431 es un NCRO de mayo con deal «Reactivación».
- **Propuesta.** Para primeras ventas usar `JOIN dwh.pos_ncro` o anti-join contra `pos_resales`, nunca `purchase_type = 'new'`. Pedir a KAM que tipifique bien el deal.

### 16. Compra MX pasada a arriendo con abono (solo documentar)
- 557935: compra `terminal-oel` el 10-ago (pagada el 11-sep) y renta el 15-sep (impaga), con deal de Janet el 11-sep. La SoT la deja en una fila KAM de 1 u (correcto, y así lo decidió Ignacio el 23-sep). El board la cuenta 2 u en septiembre. Hoy es inválida por `form_not_completed` (proxy MX) y está pagada porque basta una factura pagada del evento.

### 17. País (solo documentar)
- En las 1.882 filas con factura, el país de `companies` coincide con la moneda (CL/CLP 1.275, MX/MXN 607). Fuera de CL/MX quedan 78605 (Venezuela), 201822 (Nicaragua), −1 y 0, todas por trx y ninguna de 2026. Las cohortes de activación del board usan la moneda de la factura; para ventas da lo mismo.

## Reconciliación con el board-book jul–sep

Board = `country_pos_sot.sql` (slide 8) corrido hoy en Redshift a nivel compañía, New + Stock, sin reactivaciones, Tabata excluida, CL y MX. Los totales calzan con [[20260924-01 Métricas POS del board book vs tablas SoT]]. SoT = `pos_sales_sot` con `is_valid_purchase`, `purchase_type` `new` + `pos_additional`, por mes de `effective_sale_date`. El cruce es por `company_id` × mes: 206 eventos New/Stock y 35 de reactivación del board contra las filas de la SoT del mismo comercio.

**Unidades de compra (board → ajuste por causa → SoT)**

| Mes | País | Board | Validez | Fecha: sale | Fecha: entra | Cantidad | Eco `addon_restart` | Tabata | Reventas que el board no ve | SoT |
|---|---|---|---|---|---|---|---|---|---|---|
| jul | CL | 57 | −1 | −4 | 0 | 0 | 0 | +1 | +2 | **55** |
| ago | CL | 75 | −2 | −4 | +4 | +1 | +1 | +6 | +2 | **83** |
| sep | CL | 39 | −9 | 0 | +4 | 0 | 0 | +4 | 0 | **38** |
| jul | MX | 21 | −1 | 0 | 0 | 0 | 0 | 0 | 0 | **20** |
| ago | MX | 22 | −2 | −2 | 0 | 0 | 0 | 0 | 0 | **18** |
| sep | MX | 13 | −7 | 0 | +1 | 0 | 0 | 0 | 0 | **7** |
| **jul–sep** | **CL** | **171** | **−12** | **−8** | **+8** | **+1** | **+1** | **+11** | **+4** | **176** |
| **jul–sep** | **MX** | **56** | **−10** | **−2** | **+1** | **0** | **0** | **0** | **0** | **45** |

Detalle de cada columna:
- **Validez** (evento del board con fila inválida en la SoT): jul CL 70530 (`no_trx_pre_sep26_form_proxy`); ago CL 430263 (`no_invoice`) y 568306 (sin trx); sep CL 75776 (−2), 360327, 490058 y 582616 (`form_not_sent`), 574696 (impaga), 553259 y 554032 (impaga + sin formulario), 490565 (`no_invoice`); jul MX 25280; ago MX 114394 y 536592 (sin trx); sep MX 450004, 458124, 544651, 557935 (−2), 574554 y 580562 (`form_not_completed`). Los 14 eventos de septiembre (16 u) siguen dentro de plazo (formulario, pago o emisión de factura), pero 75776 y 490058 son swaps y no deberían validar.
- **Fecha**: 553259 (−2), 554050 y 554347 salen de julio y entran en agosto; 311692, 416679 (−2) y 570598 salen de agosto y entran en septiembre; en México 120849 pasa de agosto a septiembre y 529415 sale de agosto (en septiembre es inválida).
- **Cantidad**: 2590 (−1: board 2, deal 1), 29705 (+1) y 162282 (+1), en estos dos porque el board lee `pos_sales2`.
- **Eco**: 494549 tiene una segunda fila `addon_restart` válida en agosto (hallazgo 1).
- **Tabata**: 38388 (jul), 32381 (+3) y 104127 (+3) en ago, y 32381 (+3) y 417441 en sep.
- **Reventas que el board no ve**: 136771 (`purchase_with_pos`) y 531844 (NCRO por trx, sin evento en el board en jul–sep) en julio; 468030 y 510732 (`rent_resumed`) en agosto.
- 5 eventos del board que son conversiones arriendo → cuotas (494549, 373415, 453116, 511752, 520667) **calzan** en unidades, pero en la SoT son `pos_additional`. Si se corrige el hallazgo 1 salen: −5 u en la SoT.

**Composición de lo que calza (unidades SoT)**

| Board | País | u | ISR | KAM | Others | de ellas `pos_additional` |
|---|---|---|---|---|---|---|
| New | CL | 90 | 75 | 0 | 15 | 5 |
| New | MX | 9 | 3 | 0 | 6 | 0 |
| Stock | CL | 62 | 1 (569393) | 57 | 4 | 2 |
| Stock | MX | 35 | 0 | 33 | 2 | 2 (3001 `qty_up`) |

El Stock del board es casi todo primera venta KAM. Los Others del New son ventas ISR sin evidencia de addon ISR: 10 `new` (543994, 544602, 545082, 551088, 362121, 558139, 565702, 570660, 569925, 579176) más las 5 conversiones. En México son 6 primeras ventas con addon por API o sin usuario (542634, 554317, 557751, 567096, 568486, 574572).

**Reactivaciones (compañías; unidades entre paréntesis)**

| Mes | País | Board | SoT válida del mismo universo | Inválida en la SoT | Solo en la SoT (válida) | SoT total |
|---|---|---|---|---|---|---|
| jul | CL | 12 (12) | 11 (3) | 144708 (`no_invoice`) | 45612 y 471474 (Tabata), 91759 | 14 (10) |
| ago | CL | 13 (13) | 13 (2) | — | 621 (soft) | 14 (2) |
| sep | CL | 7 (9) | 5 (3) | 149087 y 501961 (soft en curso, plazo 10-oct) | — | 5 (3) |
| jul | MX | 1 (1) | 1 (0) | — | — | 1 (0) |
| ago | MX | 1 (1) | 1 (0) | — | — | 1 (0) |
| sep | MX | 1 (1) | 0 | 131134 (`no_invoice`) | — | 0 |

El universo de compañías coincide. La diferencia de unidades es la soft (0 u en la SoT) y la cantidad del deal en 84364 (board 3, SoT 1).

## Estado de las subtareas de T-053

| Subtarea | Estado con las tablas nuevas | Por qué |
|---|---|---|
| 1. Cuadrar compañías / locations / units | No la toca | Las tablas son a nivel company y no traen location. Unidades sí (`pos_quantity`), pero con los sesgos de los hallazgos 1, 2 y 7 |
| 2. Explicar diferencia ventas ISR vs Chargebee | La resuelve en parte | La reconciliación de arriba explica cada unidad jul–sep. La SoT separa `pos_additional` y validez, pero el canal ISR sigue siendo placeholder (15 u New del board quedan en Others en Chile) |
| 3. Revisar tabla y documentación de Ignacio | Hecha con esta auditoría | Doc de Notion leída. Contradice al código o a los datos en dos puntos: «las efectivas no se restatean» (la validez sí cambia, hallazgo 4) y `sales_user` «no se usa aún» (hallazgo 3) |
| 4. Usar el vendedor de la factura cuando falta el agente | No la resuelve | 504406 y 535145 pasan de Stock/KAM a Others sin vendedor; la SoT tampoco lee `sales_user`. El mismo arreglo sirve para la SoT (hallazgo 3) |
| 5. Revisar deals del pipeline Sales ISR | La resuelve en parte, por otra vía | 569393 sale ISR en la SoT porque barbara@ cargó el addon, no por el deal de monika@. La SoT no lee ese pipeline |
| 6. ¿Arriendo → cuotas cuenta como venta? | La resuelve en parte | Deja de ser `new` y pasa a `pos_additional`, pero sigue contando como venta válida, y dos veces por el eco (494549 jul y ago). La decisión sigue abierta y ahora además hay un error de query (hallazgo 1) |
| 7. Ventas revertidas con factura pagada | No la toca | 562659 y 563154 siguen ISR `new` válidas; hay 13 casos más con el mismo patrón (hallazgo 9) |
| 8. Alinear «POS activated per week» | No la toca | Las tablas no traen activación |
| 9. Ajustes al deck de Ignacio | No la toca | — |

## Mensaje para Ignacio

> Borrador para que Joaquín lo revise y lo envíe él.

Nacho, audité los casos borde de `pos_ncro` / `pos_resales` / `pos_sales_sot` (corte de hoy 17:12) y crucé jul–sep compañía por compañía contra el board. El total se parece (CL 171 vs 176 u, MX 56 vs 45), pero hay tres cosas que creo que cambian la query y que te conviene ver antes de que la usemos en los gráficos, porque mueven resultados anteriores:

1. **`addon_restart`**: cualquier baja de addon entre dos rentas abre venta, sin exigir un alta después, y el `BETWEEN` incluye el día de la renta anterior. Resultado: los swaps arriendo → cuotas cuentan como venta, y dos veces (494549 en jul y ago, 38388 en jun y jul, 1566 dos veces en mar). En 2026 son 22 filas válidas / 25 u y ninguna suma terminales. Propuesta: baja estrictamente después de la renta anterior, sin alta de otro addon POS el mismo día y con un alta después de la baja.
2. **`qty_up`** guarda la cantidad total y no el incremento: 30 u contra 16 reales en 2026 (32381, 104127, 498146).
3. **Canal ISR**: 143 de 171 ISR salen por barbara@. `pos_sales.sales_user` trae al vendedor real (562659 → Daniela Suarez) y le daría canal a 60 de los 84 Others de 2026, mientras llega «contra topos». ¿Lo usamos como interino?

Menores: 452621 queda partida en NCRO ISR inválido + reventa Others (trx 70 d antes de la factura); falta un test de `invoice_id` único entre filas (19517) y filtrar `company_id` −1 y 0.

Preguntas abiertas:
- Fecha oficial: ¿pago (tu doc, 21-sep) o emisión (lo que acordamos con Finanzas el 7-sep)?
- Cantidad: ¿deal o factura? Hay 6 casos en 2026 (388767, 84364, 471474, 2590, 417441, 518906).
- Validez: septiembre tiene 20 inválidas que pueden validar todavía (formulario a +60/+90 d). ¿Congelamos el mes al cierre o aceptamos restatement? En Chile no hay formularios antes de mayo, así que ene–ago no es comparable con septiembre.
- Soft: ¿revalidamos jul–ago con 15 trx (9 no llegan)? ¿Exigimos 60 d sin actividad (13 de 26 estaban activos)? 501961 tiene factura pagada: ¿`reactivation_with_purchase`?
- Tabata: el board la excluye y la SoT no (15 u en 2026). ¿Cuál queda?
- MX: hay deals ganados hasta 32 d después de la factura (529415, 557935), y eso los pasa a septiembre y a las reglas estrictas. ¿La fecha declarada KAM debería ser la factura cuando es anterior?

## Cómo reproducir

Todas son SELECT en Redshift (`agendapro-dwh`). La query 2 embebe la salida de la 1 porque juntas dan timeout.

**1. Board a nivel compañía** (`sources/dwh/payments/country_pos_sot.sql`, sin cambios de lógica). Para que corra por el MCP (28 s) se acotaron las fuentes sin cambiar jul–sep: `addon_evt` con `occurred_at >= '2025-10-01'`, `ps_base` con `TO_DATE(month,'YYYYMM') >= '2025-12-01' OR month IS NULL` y `sh` con `ano >= 2025`. Los totales por mes calzan con [[20260924-01 Métricas POS del board book vs tablas SoT]]. El `SELECT` final se reemplaza por:
```sql
SELECT sale_month AS date, country_group, company_id, units, sale_day, fuente, vendedor,
       CASE WHEN tipo_venta = 'Reactivacion' THEN 'Reactivacion' WHEN canal = 'ISR' THEN 'New' ELSE 'Stock' END AS category
FROM tipado
WHERE sale_month >= DATE '2026-07-01' AND sale_month < DATE '2026-10-01'
  AND COALESCE(vendedor, '') <> 'tabata' AND country_group IN ('Chile','Mexico');
```

**2. Reconciliación** (lista del board como texto `mes|país|categoría|company_id:unidades`, 241 eventos):
```sql
WITH src AS (SELECT '7CLN494549:1,7CLN529200:1,7CLN537086:1,7CLN543732:1,7CLN543994:1,7CLN544602:1,7CLN544679:1,7CLN545082:1,7CLN545483:1,7CLN545702:1,7CLN545809:1,7CLN547022:1,7CLN547114:1,7CLN548599:1,7CLN548686:1,7CLN549408:1,7CLN550046:1,7CLN550410:1,7CLN550675:1,7CLN551088:1,7CLN552096:1,7CLN552206:1,7CLN552875:1,7CLN553259:2,7CLN553321:2,7CLN554050:1,7CLN554179:1,7CLN554306:1,7CLN554347:1,7CLR9490:1,7CLR19506:1,7CLR109031:1,7CLR111874:1,7CLR144708:1,7CLR203659:1,7CLR302991:1,7CLR338864:1,7CLR389780:1,7CLR459098:1,7CLR484027:1,7CLR506857:1,7CLS750:1,7CLS1446:1,7CLS1513:5,7CLS19671:1,7CLS20783:1,7CLS32255:2,7CLS33259:1,7CLS37231:3,7CLS43708:1,7CLS54470:1,7CLS70530:1,7CLS192261:1,7CLS284905:1,7CLS397772:1,7CLS488181:1,7CLS524920:1,7CLS537052:1,7CLS543563:1,7CLS544336:1,7MXN542634:1,7MXR346119:1,7MXS3001:2,7MXS15137:1,7MXS25280:1,7MXS170863:1,7MXS172809:1,7MXS189293:1,7MXS369542:1,7MXS401195:1,7MXS405660:1,7MXS424399:1,7MXS457804:1,7MXS497320:1,7MXS512774:1,7MXS514113:1,7MXS518906:2,7MXS525623:1,7MXS526766:1,7MXS531762:1,8CLN362121:1,8CLN373415:1,8CLN416679:2,8CLN453116:1,8CLN511752:1,8CLN518460:1,8CLN520667:1,8CLN527574:1,8CLN530591:1,8CLN551286:3,8CLN557505:2,8CLN557787:1,8CLN557911:1,8CLN558139:1,8CLN558188:1,8CLN559296:1,8CLN560106:1,8CLN560120:1,8CLN560911:1,8CLN562093:1,8CLN562659:1,8CLN563125:1,8CLN563154:1,8CLN563172:1,8CLN563446:1,8CLN564283:1,8CLN564685:1,8CLN564941:1,8CLN565142:1,8CLN565624:1,8CLN565702:1,8CLN566073:1,8CLN567125:1,8CLN567436:1,8CLN567917:1,8CLN568306:1,8CLN568942:1,8CLN569264:1,8CLN569290:1,8CLN570598:1,8CLN570645:1,8CLN570660:1,8CLN570743:1,8CLR3451:1,8CLR7045:1,8CLR9223:1,8CLR19534:1,8CLR20671:1,8CLR21087:1,8CLR23445:1,8CLR120798:1,8CLR120994:1,8CLR152982:1,8CLR161984:1,8CLR190960:1,8CLR214686:1,8CLS2590:2,8CLS11787:1,8CLS12298:1,8CLS16897:1,8CLS29705:1,8CLS40694:1,8CLS47565:1,8CLS78437:1,8CLS84864:1,8CLS160110:1,8CLS162282:2,8CLS195813:1,8CLS202127:1,8CLS215275:1,8CLS311692:1,8CLS328595:1,8CLS383848:1,8CLS392383:1,8CLS404553:1,8CLS418685:1,8CLS430263:1,8CLS447138:1,8CLS487289:1,8CLS504406:1,8CLS535145:1,8CLS569393:1,8MXN554317:1,8MXN557751:1,8MXN559593:1,8MXN567096:1,8MXN568486:1,8MXR66316:1,8MXS48787:1,8MXS114394:1,8MXS120849:1,8MXS379643:1,8MXS461569:1,8MXS495233:1,8MXS497056:1,8MXS501490:1,8MXS509461:1,8MXS526634:1,8MXS529415:1,8MXS529958:1,8MXS536592:1,8MXS541887:1,8MXS550624:1,8MXS557772:1,8MXS564607:1,9CLN533993:2,9CLN553259:1,9CLN554032:1,9CLN567080:1,9CLN567896:1,9CLN569917:1,9CLN569925:1,9CLN570311:1,9CLN572087:1,9CLN572140:1,9CLN572195:1,9CLN572537:1,9CLN573834:1,9CLN574696:1,9CLN575475:1,9CLN578042:1,9CLN578315:1,9CLN579167:1,9CLN579176:1,9CLN581801:1,9CLN581877:1,9CLN582600:1,9CLN582616:1,9CLR1724:1,9CLR84364:3,9CLR149087:1,9CLR378928:1,9CLR417761:1,9CLR501961:1,9CLR518462:1,9CLS1404:1,9CLS1869:1,9CLS52173:1,9CLS75776:2,9CLS156507:1,9CLS337546:1,9CLS341107:2,9CLS360327:1,9CLS440321:1,9CLS490058:1,9CLS490565:1,9CLS546997:1,9CLS568157:1,9MXN574554:1,9MXN574572:1,9MXN574924:1,9MXN579959:1,9MXR131134:1,9MXS190575:1,9MXS368371:1,9MXS450004:1,9MXS458124:1,9MXS506951:1,9MXS544651:1,9MXS557935:2,9MXS580562:1'::varchar(4000) AS s),
n AS (SELECT ROW_NUMBER() OVER (ORDER BY sale_id) AS i FROM dwh.pos_sales_sot),
tok AS (SELECT SPLIT_PART(src.s, ',', n.i::int) AS t FROM src CROSS JOIN n WHERE n.i <= 241),
board AS (SELECT TO_DATE('2026-0' || LEFT(t,1) || '-01', 'YYYY-MM-DD') AS mes, SUBSTRING(t,2,2) AS pais, SUBSTRING(t,4,1) AS cat,
                 SPLIT_PART(SUBSTRING(t,5),':',1)::bigint AS cid, SPLIT_PART(t,':',2)::int AS u FROM tok),
sot AS (
  SELECT company_id AS cid, MAX(country) AS pais, DATE_TRUNC('month', COALESCE(effective_sale_date, declared_sale_date))::date AS mes,
         SUM(CASE WHEN is_valid_purchase AND purchase_type <> 'reactivation' THEN pos_quantity ELSE 0 END) AS uvp,
         SUM(CASE WHEN is_valid_purchase AND purchase_type = 'reactivation' THEN 1 ELSE 0 END) AS nvr,
         MAX(CASE WHEN seller_email LIKE 'tabata%' THEN 1 ELSE 0 END) AS tabata,
         MAX(CASE WHEN NOT is_valid_purchase THEN SPLIT_PART(invalid_reason,' ',1) END) AS inv_r,
         MAX(purchase_sub_type || ':' || sale_evidence) AS tipos
  FROM dwh.pos_sales_sot
  WHERE COALESCE(effective_sale_date, declared_sale_date) BETWEEN DATE '2026-04-01' AND DATE '2026-12-31'
  GROUP BY 1, 3
),
bc AS (SELECT cid, COUNT(*) AS nb FROM board GROUP BY 1),
sc AS (SELECT cid, COUNT(*) AS ns FROM sot GROUP BY 1),
s3 AS (SELECT * FROM sot WHERE mes BETWEEN DATE '2026-07-01' AND DATE '2026-09-01' AND pais IN ('CL','MX')),
j AS (
  SELECT COALESCE(b.mes, s.mes) AS mes, COALESCE(b.pais, s.pais) AS pais, COALESCE(b.cid, s.cid) AS cid,
         b.cat, COALESCE(b.u,0) AS ub, COALESCE(s.uvp,0) AS uvp, s.nvr, s.tabata, s.inv_r, s.tipos, (s.cid IS NOT NULL) AS s_here,
         COALESCE(bc.nb,0) - CASE WHEN b.cid IS NOT NULL THEN 1 ELSE 0 END AS nb_oth,
         COALESCE(sc.ns,0) - CASE WHEN s.cid IS NOT NULL THEN 1 ELSE 0 END AS ns_oth
  FROM board b FULL OUTER JOIN s3 s ON s.cid = b.cid AND s.mes = b.mes
  LEFT JOIN bc ON bc.cid = COALESCE(b.cid, s.cid)
  LEFT JOIN sc ON sc.cid = COALESCE(b.cid, s.cid)
),
c AS (
  SELECT j.*,
    CASE WHEN cat IN ('N','S') AND s_here AND uvp > 0 AND uvp = ub THEN '0 calza'
         WHEN cat IN ('N','S') AND s_here AND uvp > 0 THEN '1 cantidad'
         WHEN cat IN ('N','S') AND s_here AND nvr > 0 THEN '2 tipo: SoT reactivacion'
         WHEN cat IN ('N','S') AND s_here THEN '3 validez: ' || COALESCE(inv_r,'?')
         WHEN cat IN ('N','S') AND ns_oth > 0 THEN '4 fecha: SoT en otro mes'
         WHEN cat IN ('N','S') THEN '5 sin fila en SoT'
         WHEN cat = 'R' AND uvp > 0 THEN '6 tipo: board React, SoT compra'
         WHEN cat = 'R' THEN '9 react (fuera del puente)'
         WHEN cat IS NULL AND uvp > 0 AND nb_oth > 0 THEN '7 fecha: board en otro mes'   -- revisar a mano: 494549 ago es eco, no fecha
         WHEN cat IS NULL AND uvp > 0 AND tabata = 1 THEN '8 alcance: Tabata'
         WHEN cat IS NULL AND uvp > 0 THEN '8 alcance: no esta en board'
         ELSE '9 sin efecto' END AS causa,
    CASE WHEN cat IN ('N','S') THEN uvp - ub WHEN cat = 'R' THEN uvp WHEN cat IS NULL THEN uvp ELSE 0 END AS dlt
  FROM j
)
SELECT TO_CHAR(mes,'MM') m, pais, causa, COUNT(*) n, SUM(CASE WHEN cat IN ('N','S') THEN ub ELSE 0 END) ub, SUM(uvp) uvp, SUM(dlt) dlt,
       LISTAGG(CASE WHEN causa NOT LIKE '0%' THEN cid || '[' || COALESCE(cat,'-') || ':' || COALESCE(tipos,'') || ',' || dlt || ']' END, ' ') WITHIN GROUP (ORDER BY cid) ej
FROM c GROUP BY 1, 2, 3 ORDER BY 1, 2, 3;
```

**3. Patrón de `addon_restart`** (swap, baja sin alta, eco):
```sql
WITH ar AS (SELECT sale_id, company_id, country, declared_sale_date d, pos_quantity q, is_valid_purchase v
            FROM dwh.pos_sales_sot WHERE sale_evidence = 'addon_restart' AND declared_sale_date >= '2026-01-01'),
ch AS (SELECT a.sale_id, SUM(CASE WHEN c.change_type='added' THEN 1 ELSE 0 END) n_add,
              MAX(CASE WHEN c.change_type='removed' THEN c.occurred_at::date END) last_rem,
              MAX(CASE WHEN c.change_type='added' THEN c.occurred_at::date END) last_add
       FROM ar a JOIN dwh.int_chargebee__subscription_addon_changes c
         ON c.company_id = a.company_id::varchar AND c.addon_id ~* 'pos|getnet|terminal' AND c.change_type IN ('added','removed')
        AND c.occurred_at::date BETWEEN a.d - 45 AND a.d
       GROUP BY 1),
swap AS (SELECT DISTINCT a.sale_id
         FROM ar a
         JOIN dwh.int_chargebee__subscription_addon_changes r ON r.company_id = a.company_id::varchar AND r.change_type = 'removed'
          AND r.addon_id ~* 'pos|getnet|terminal' AND r.occurred_at::date BETWEEN a.d - 45 AND a.d
         JOIN dwh.int_chargebee__subscription_addon_changes x ON x.company_id = r.company_id AND x.change_type = 'added'
          AND x.addon_id ~* 'pos|getnet|terminal' AND x.occurred_at::date = r.occurred_at::date AND x.addon_id <> r.addon_id)
SELECT a.*, CASE WHEN s.sale_id IS NOT NULL THEN 'swap_mismo_dia' WHEN COALESCE(ch.n_add,0) = 0 THEN 'baja_sin_alta'
                 WHEN ch.last_add > ch.last_rem THEN 'alta_despues_de_baja' ELSE 'otro' END AS patron
FROM ar a LEFT JOIN ch ON ch.sale_id = a.sale_id LEFT JOIN swap s ON s.sale_id = a.sale_id;
-- Eco: unir las líneas de renta de pos_sales con LAG(invoice_created_at) por comercio y contar filas
-- cuya única baja cae en la fecha de la renta anterior (rm.occurred_at::date = prev_line_date).
```

**4. `qty_up`, total vs incremento:**
```sql
WITH qu AS (SELECT sale_id, company_id, declared_sale_date d, pos_quantity q, invoice_ids
            FROM dwh.pos_sales_sot WHERE sale_evidence = 'qty_up' AND declared_sale_date >= '2026-01-01')
SELECT qu.company_id, qu.q, MAX(p.item_qty) AS prev_q, qu.q - MAX(p.item_qty) AS incremento
FROM qu LEFT JOIN dwh.pos_sales p ON p.company_id::bigint = qu.company_id AND p.item_name ~* 'renta|arriendo|cuotas'
 AND p.invoice_created_at::date BETWEEN qu.d - 62 AND qu.d - 1
 AND ',' || qu.invoice_ids || ',' NOT LIKE '%,' || p.invoice_id || ',%'
GROUP BY 1, 2;
```

**5. Validez, pagadas inválidas y ventana abierta:**
```sql
SELECT TO_CHAR(declared_sale_date,'MM') m, country, COUNT(*) filas,
       SUM(CASE WHEN NOT is_valid_purchase THEN 1 ELSE 0 END) invalidas,
       SUM(CASE WHEN NOT is_valid_purchase AND is_invoice_paid THEN 1 ELSE 0 END) pagadas_invalidas,
       SUM(CASE WHEN NOT is_valid_purchase AND invalid_reason LIKE '%form_not_%'
                 AND declared_sale_date + CASE WHEN country = 'CL' THEN 60 ELSE 90 END >= CURRENT_DATE THEN 1 ELSE 0 END) ventana_abierta
FROM dwh.pos_sales_sot WHERE declared_sale_date >= '2026-01-01' GROUP BY 1, 2 ORDER BY 2, 1;
-- Válidas ene–ago sin formulario: CL = form_completed_date IS NULL; MX = sin fila en
-- staging.stg_google_sheets__control_terminales a −15/+90 d (fecha 'DD-MM-YYYY').
```

**6. Vendedor de la factura vs usuario del addon (NCRO 2026):**
```sql
WITH n AS (SELECT ROW_NUMBER() OVER (ORDER BY sale_id) AS i FROM dwh.pos_sales_sot),
s AS (SELECT s.* FROM dwh.pos_sales_sot s LEFT JOIN dwh.pos_resales r USING (sale_id)
      WHERE r.sale_id IS NULL AND s.declared_sale_date >= '2026-01-01'),
ex AS (SELECT s.sale_id, TRIM(SPLIT_PART(s.invoice_ids, ',', n.i::int)) inv
       FROM s JOIN n ON n.i <= REGEXP_COUNT(s.invoice_ids, ',') + 1 WHERE s.invoice_ids IS NOT NULL),
su AS (SELECT ex.sale_id, MAX(p.sales_user) sales_user FROM ex JOIN dwh.pos_sales p ON p.invoice_id = ex.inv GROUP BY 1)
SELECT s.country, s.channel_attribution, SPLIT_PART(s.seller_email,'@',1) addon_user, su.sales_user, COUNT(*) filas,
       SUM(CASE WHEN s.is_valid_purchase THEN s.pos_quantity ELSE 0 END) u_validas
FROM s LEFT JOIN su ON su.sale_id = s.sale_id
WHERE s.channel_attribution IN ('ISR','Others') GROUP BY 1, 2, 3, 4 ORDER BY 1, 2, 5 DESC;
```

**7. Otros controles usados**: soft (`purchase_sub_type = 'reactivation_soft'` con `trx_by_soft_deadline`, `days_inactive`, `is_invoice_paid`); cantidad deal vs factura (KAM 2026, `pos_quantity` contra `MAX(item_qty)` de `pos_sales` unido por `invoice_ids`); facturas compartidas (explotar `invoice_ids` y `HAVING COUNT(DISTINCT sale_id) > 1`); revertidas (válidas 2026 con `removed` de addon POS entre `declared_sale_date` y +45 d, sin `added` posterior y `trx_post_purchase < 15`); país contra `pos_sales.invoice_currency`; y deals KAM con `invoice_created_date < deal_sale_date`.

## Caveats
- El board se reprodujo desde Redshift en vivo, no desde el parquet publicado. El mes en curso es parcial, y las tablas de Ignacio y `pos_sales2` se recrean en cada corrida de dbt.
- En el cruce, el mes de la SoT es `COALESCE(effective_sale_date, declared_sale_date)`, así que las inválidas sin pago caen en su mes declarado. El puente usa solo las válidas.
- La clasificación automática marcó 494549 (ago) como «fecha»; a mano es eco del hallazgo 1 y así está en la tabla.
- Los «revertidos» del hallazgo 9 son candidatos por patrón, no casos confirmados. Algunas bajas pueden ser un cambio de suscripción.
- `sales_user` se validó por nombre contra `hubspot_mapping.user_name`. No se contrastó contra la planilla ISR ni contra HubSpot deal por deal.
- No se revisaron activación ni retención (slides 2.4 / 2.5): las tablas nuevas no las traen.

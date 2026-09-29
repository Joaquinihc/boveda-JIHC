---
type: query
temas: [pos, ventas]
proyecto: "[[_SoT Ventas POS]]"
lenguaje: sql (Redshift)
---

> Query original: `POS_VENTAS_SOT.sql` (copia literal).

```sql
-- =====================================================================================
-- ProPay · FUENTE ÚNICA DE VENTAS POS  (SoT)
-- Grano: 1 fila por EVENTO de venta = empresa × mes de venta.
-- Cubre las tres poblaciones en una sola tabla:
--   · ISR            (inbound / vendedor nuevo)      → solo existe en Chargebee
--   · KAM Venta      (venta nueva del equipo KAM)     → en Chargebee + sheet
--   · KAM Reactivación (recupera comercio frío)       → CASI SOLO en el sheet (18% factura)
--
-- Por qué la unión es obligatoria (medido 2026-08-04, cohortes 2026):
--   · Venta Nueva del sheet: 93% tiene factura en Chargebee  → se cruzan bien.
--   · Reactivación del sheet: solo 18% (4 de 22) tiene factura → invisible en pos_sales2,
--     y por lo tanto invisible en el gráfico New/Stock del dashboard pos-ventas.
--   · Al revés: 12-28 eventos/mes están en Chargebee y no en el sheet (ISR, más algún
--     KAM que no registró la fila).
--
-- COHORTE: `sale_month` es SIEMPRE el mes del evento, no la primera compra histórica.
-- En una reactivación el reloj se reinicia — es lo que mide el impacto del equipo KAM hoy.
-- (Ojo: los análisis previos de activación usaban "primera compra de la vida"; esta fuente
--  cambia ese criterio a propósito y por eso da cohortes más grandes.)
--
-- Autoridad de cada campo (orden de precedencia):
--   canal      Jerarquía por OWNERSHIP, con el tiempo como último recurso (acordado 6-ago-2026).
--              Principio: para atribuir un evento histórico solo sirven campos POINT-IN-TIME
--              (fijos al momento de la venta); los de ESTADO ACTUAL contaminan hacia atrás.
--              0) planilla KAM (registro propio del equipo)          → KAM
--              1) owner del DEAL ganado en HubSpot  [point-in-time]  → KAM o ISR · desde abr-2026
--              2) pos_sale_agent de Chargebee       [point-in-time]  → KAM o ISR · 79% cobertura
--              3) contacts.kam_pos_owner            [estado actual]  → solo KAM, marcado inferido
--              4) ventana de recencia ≤30d (≥jun-26) / ≤90d (antes)  → inferido, <20% de los casos
--              El campo `canal_origen` expone cuál de los cinco niveles disparó.
--              ⚠ Roster KAM: rocio · janet · jaime · fernandobarra · alvaro · tabata.
--                cristobal@ NO va — era el LÍDER del área. Aparece en `contacts.kam_pos_owner`
--                (1 contacto) y hay que descartarlo. Y kam_pos_owner NO trae a tabata: está
--                incompleto en los dos sentidos (37 falsos positivos + 34 negativos = 19% de
--                error si se usara como autoridad). Por eso va en el nivel 3 y no más arriba.
--              ⚠ Esta regla YA NO cuadra con el board, a propósito: el board usa solo
--                (roster hardcodeado → ventana → is_new_merchant). Para reproducir el board ver
--                POS_RECALCULO_TABLEROS.sql, que mantiene el canal congelado.
--   tipo_venta 1) del sheet (Venta Nueva / Reactivación) — ÚNICA fuente real
--              2) si no está en el sheet → derivado: primera compra histórica = 'Venta Nueva';
--                 con compra previa y ≥60 días sin transaccionar = 'Reactivación (derivada)';
--                 con compra previa y actividad reciente = 'POS adicional'
--   unidades   Chargebee (facturación) y si no el sheet
--   país       dwh.companies.country_name, fallback al `pais` del sheet
--
-- ESQUEMA DE TRANSACCIONES (para métricas de activación/GPV sobre estos eventos):
--   gpv_pos = INTERNO + EXTERNO
--   INTERNO  = dwh.propay_transactions, provider_id IN (1,3,6,7,34,35,36,67,68,69,100,166)
--              → 88-89% del monto, 0% de company_id nulo. ES LA SoT A NIVEL TRANSACCIÓN.
--   EXTERNO  = dwh.{haulmer,sumup,oel}_augmented_transactions WHERE is_external → 11-12%
--   Unidos en dwh.propay_daily_transactions (día × país × empresa × local), que luego alimenta
--   location_sales_days → company_sales_days → company_sales_months.
--   ⚠ NO usar dwh.all_augmented_transactions para cuadrar GPV: excluye Transbank a propósito y
--     da 110% en CLP / 85% en MXN. Transbank solo aporta su parte interna, por diseño.
--   🔴 Bug abierto: Haulmer pierde 39% de sus trx externas sin pos_company_id (≈USD 257K/mes).
--   Doc oficial: https://app.notion.com/p/agendapro/DWH-documentaiton-3b2ab34c233b806eaea8cd1d1909a395
--
-- FECHA DE VENTA (corregido 2026-08-06 — el board pierde 1 de cada 3 ventas en la vista semanal):
--   `addon_added_at` es la fecha CONCEPTUALMENTE CORRECTA (el acto comercial de colgar la máquina a
--   la suscripción, independiente del ciclo de facturación, las cuotas y la cobranza). No fue una
--   mala elección de diseño. Falla por dos razones ESTRUCTURALES, ninguna de calidad de datos:
--
--   (1) SOLO EXISTE SI EL POS SE ARRIENDA O FINANCIA. Los únicos addons POS en Chargebee son
--       `smart-pos-arriendo_2025-cl` y `terminal-oel-renta_2025-mx`. Una venta directa es un ítem
--       único de factura y no crea addon, así que no hay fecha por construcción. Medido 2025-26:
--         arriendo/cuotas: 743 filas → 743 con addon (100%)
--         venta directa:   376 filas →  20 con addon (5,3%)
--       Por eso el "deterioro" de jul-26 NO es un bug: la venta directa pasó de 11% a 35% del mix
--       (% con addon ≈ 100 − % venta directa, casi perfecto). Y 50-100% de la venta directa es MX,
--       así que la vista semanal está sesgada sistemáticamente contra México.
--
--   (2) `pos_sales2.addon_added_at` NO ES LA FECHA DEL EVENTO, es un atributo de EMPRESA. El modelo
--       aplica `addon_sale_rank = 1` en un CTE llamado `first_pos_sale_per_company`: la primera vez
--       que la empresa tuvo POS. Se construyó para atribuir al vendedor original (`pos_sale_agent`)
--       y la fecha vino de arrastre. En una RECOMPRA o REACTIVACIÓN devuelve la compra original:
--       43 filas con gap de hasta -676 días → ventas de 2026 dibujadas en semanas de 2024-25.
--       De 54 recompras/reactivaciones facturadas desde ene-25, solo 9 (17%) quedan en el mes real.
--       ⚠ Pero el dato bueno EXISTE: 24 de esas 43 tienen un cambio de addon real cerca del mes de
--         venta en `int_chargebee__subscription_addon_changes` (gap mediano +11 d). El `rank = 1`
--         lo descarta. Por eso esta query va a la tabla de cambios y NO usa `pos_sales2.addon_added_at`.
--
--   Prioridad que usa esta query (resultado: 0 ventas sin fecha real en todos los meses de 2026):
--     1) cambio de addon POS más CERCANO al mes de venta (CTE `addon_near`) = el acto comercial
--     2) `invoice_created_at` — emisión, NO cobranza. Único dato disponible en venta directa.
--        Se descartó `invoice_paid_at`: 9,5% se paga >7 días después (máx 338) y 117 nunca se pagaron.
--        Verificado que NO llega tarde: contra el cierre de deal en HubSpot, 0 de 34 casos tiene la
--        factura >15 días después (gap máx +14 d) y 0 tiene `start_date_pago` diferido. O sea el
--        escenario "3 meses gratis y factura tardía" no se materializa en la data.
--     3) día de la planilla KAM (≥ 21-jul-2026, antes es default 1)
--     4) día 1 del mes (solo eventos de planilla sin factura ni día)
--   `fecha_origen` expone cuál se usó y `fecha_confiable` = 1 si el día es real.
--
--   HubSpot (`stg_hubspot__deals`, pipeline 'Seguimiento POS Expansión', stage 'Ganado') NO sirve
--   como fuente de fecha: el deal se marca ganado DESPUÉS de facturar (gap hasta -32 d) y 9 de 34
--   casos caerían en otro mes. Arranca en abr-2026 y cubre ~80% de las ventas directas sin addon.
--   Donde SÍ es valioso es en ATRIBUCIÓN: `kam_pos_owner` / `hubspot_owner_id` es mejor fuente de
--   canal que el roster hardcodeado de 6 emails que usa la regla de abajo.
--
--   `sale_day_board` se expone para poder reconciliar contra los tableros (es la fecha que ellos
--   usan hoy) sin tener que rehacer esta query. La regla de canal usa `sale_day`, el bueno.
--
-- GRANO Y CLAVE (importante si esto va a ser tabla maestra):
--   Clave única = (company_id, cohort_month). Verificado 6-ago-2026: 473 filas / 473 claves, cero
--   duplicados — el emparejamiento 1-a-1 del rescate ±1 mes lo garantiza. Al llevarlo a dbt,
--   dejarlo como test `unique` sobre la combinación, no como comprobación manual.
--   ⚠ Es un grano de EVENTO, no de unidad: una empresa que compra 3 máquinas el mismo mes es 1 fila
--   con units=3. Los tableros cuentan unidades. No son intercambiables (ver §2b de POS_VENTAS_SOT.md).
--   ⚠ `units` = Chargebee y si no la planilla, con fallback silencioso a 1 cuando faltan las dos.
--   `units_chargebee` y `units_planilla` se exponen aparte para ver las discrepancias.
--
-- COSTO: el CTE `act_previa` hace LEFT JOIN contra `dwh.company_sales_days` SIN piso de fecha,
--   porque `trx_previas = 0` necesita toda la historia (con un piso, una empresa con transacciones
--   viejas parecería nueva). Es el paso más caro y hace que la query no corra por el MCP (~15 s).
--   Al materializarla en dbt, esto deja de ser un problema; mientras tanto correr por psycopg2.
--
-- Convenciones: dedup Chargebee §3.3 (1 venta por empresa+día, unidades = MAX(item_qty)).
-- El `dia` del sheet es default 1 hasta el 20-jul-2026 → solo sirve desde el 21-jul.
-- =====================================================================================
WITH addon_evt AS (                     -- TODOS los cambios de addon POS (no solo el primero)
    -- pos_sales2.addon_added_at aplica `addon_sale_rank = 1` en un CTE llamado
    -- `first_pos_sale_per_company`: es la PRIMERA vez que la empresa tuvo POS, un atributo de
    -- empresa, no del evento de venta. Acá vamos a la tabla de cambios sin ese filtro.
    SELECT company_id::int AS company_id, occurred_at::date AS addon_day
    FROM dwh.int_chargebee__subscription_addon_changes
    WHERE change_type IN ('added','initial')
      AND addon_id ~* 'pos|getnet|terminal|smart-pos'
      AND company_id IS NOT NULL AND company_id <> ''
),
addon_near AS (                         -- el addon MÁS CERCANO al mes de venta (1 por empresa+mes)
    SELECT company_id, mes_d, addon_day
    FROM (
        SELECT m.company_id, m.mes_d, ae.addon_day,
               ROW_NUMBER() OVER (PARTITION BY m.company_id, m.mes_d
                                  ORDER BY ABS(DATEDIFF(day, m.mes_d, ae.addon_day))) AS rn
        FROM (SELECT DISTINCT company_id, TO_DATE(month,'YYYYMM') AS mes_d FROM dwh.pos_sales2) m
        JOIN addon_evt ae
          ON ae.company_id = m.company_id
         AND ae.addon_day BETWEEN m.mes_d - 31 AND m.mes_d + 62
    ) x
    WHERE rn = 1
),
inv AS (                                -- fecha EXACTA de la factura (tipo date, 100% poblada)
    -- Se usa invoice_created_at (emisión) y NO invoice_paid_at (cobranza): el 71% se paga el mismo
    -- día, pero el 9,5% tarda >7 días (máx 338) y 117 facturas nunca se pagaron. La emisión está
    -- más cerca del acto comercial; la cobranza atribuiría la venta a la semana en que entró la plata.
    SELECT invoice_id,
           MIN(COALESCE(invoice_created_at, invoice_paid_at)) AS invoice_day
    FROM dwh.pos_sales
    GROUP BY 1
),
cb_raw AS (                             -- Chargebee con la fecha de venta saneada (ver bloque FECHA)
    SELECT ps.company_id,
           ps.sale_id,
           ps.item_qty,
           LOWER(COALESCE(ps.pos_sale_agent, ps.hubspot_owner_email, ps.corrected_agent_email)) AS agent_email,
           ps.is_new_merchant,
           -- 1) el addon POS más cercano a este mes = el acto comercial real (arriendos/cuotas)
           -- 2) la emisión de la factura = lo único disponible en una venta directa (no hay addon)
           -- 3) día 1 del mes como guarda
           COALESCE(an.addon_day, iv.invoice_day, TO_DATE(ps.month, 'YYYYMM'))  AS sale_day,
           CASE WHEN an.addon_day  IS NOT NULL THEN 'addon'
                WHEN iv.invoice_day IS NOT NULL THEN 'factura'
                ELSE 'mes' END                                AS fecha_origen,
           -- fecha que usan HOY el board y pos-ventas (addon crudo → día 1 del mes). Se expone
           -- como `sale_day_board` para poder reconciliar contra los tableros sin rehacer la query.
           COALESCE(ps.addon_added_at::date, TO_DATE(ps.month, 'YYYYMM')) AS sale_day_board
    FROM dwh.pos_sales2 ps
    LEFT JOIN inv iv ON iv.invoice_id = ps.first_invoice_id
    LEFT JOIN addon_near an ON an.company_id = ps.company_id
                           AND an.mes_d = TO_DATE(ps.month, 'YYYYMM')
),
cb_dedup AS (                           -- 1 fila por empresa+día (canon §3.3)
    SELECT company_id, sale_day, sale_day_board, fecha_origen,
           item_qty AS units, agent_email, is_new_merchant
    FROM (
        SELECT cb_raw.*,
               ROW_NUMBER() OVER (PARTITION BY company_id, sale_day
                                  ORDER BY item_qty DESC, sale_id) AS rn
        FROM cb_raw
    ) x
    WHERE rn = 1
),
cb AS (                                 -- colapsado a empresa+mes para poder casar con el sheet
    SELECT company_id,
           DATE_TRUNC('month', sale_day)::date AS sale_month,
           MIN(sale_day)        AS sale_day,
           MIN(sale_day_board)  AS sale_day_board,
           MIN(fecha_origen)    AS fecha_origen,   -- 'addon' < 'factura' < 'mes' = orden de prioridad
           SUM(units)           AS units,
           MAX(agent_email)     AS agent_email,
           MAX(is_new_merchant) AS is_new_merchant
    FROM cb_dedup
    GROUP BY 1, 2
),
sh AS (                                 -- registro del equipo KAM: única fuente de tipo_venta
    SELECT company_id,
           TO_DATE(ano::varchar || LEFT(mes, 2), 'YYYYMM') AS sale_month,
           MAX(CASE WHEN dia BETWEEN 1 AND 31
                    THEN TO_DATE(ano::varchar || LEFT(mes,2) || LPAD(dia::varchar, 2, '0'), 'YYYYMMDD')
               END) AS sheet_day,
           SUM(cant) AS units,
           MAX(CASE WHEN LOWER(tipo_venta) LIKE '%reactiva%' THEN 1 ELSE 0 END) AS es_react,
           MAX(kam)  AS kam,
           MAX(CASE WHEN pais = 'México' THEN 'Mexico' ELSE pais END) AS pais_sheet,
           MAX(CASE WHEN LOWER(tipo) LIKE '%split%' THEN 'POS+FE+SPLIT'
                    WHEN LOWER(tipo) LIKE '%fe%'    THEN 'POS+FE'
                    ELSE 'POS' END) AS producto
    FROM dwh.pos_v2
    WHERE company_id IS NOT NULL
    GROUP BY 1, 2
),
-- ╔═══════════════════════════════════════════════════════════════════════════════════════╗
-- ║ ⚠ DEUDA ABIERTA: EL ROSTER DE VENDEDORES ESTÁ HARDCODEADO Y NO HAY ROSTER DE ISR      ║
-- ║                                                                                       ║
-- ║ Hoy `kam_roster` es una lista de 6 emails en el WHERE de abajo, y **ISR = "el que no  ║
-- ║ es KAM"**. Eso es la misma falla lógica que tenía la regla vieja, solo movida un paso:║
-- ║ un account manager, alguien de soporte o un KAM fuera del roster que ejecute el       ║
-- ║ cambio en Chargebee queda clasificado como ISR sin haber vendido nada.                ║
-- ║                                                                                       ║
-- ║ Impacto medido (auditoría 6-ago-2026 de los 12 eventos de Chile que pasaron de Stock  ║
-- ║ a New): solo 5 tienen atribución limpia. 5 son de `leonardoruiz`, cuyo perfil NO      ║
-- ║ parece ISR (vende a comercios de 127 a 2.367 días de antigüedad, solo operó en        ║
-- ║ ene-feb 2026), y 2 (empresas 7743 y 430053) las reclama un KAM en la planilla 3-4     ║
-- ║ meses después. O sea el +10 de Chile podría ser +3.                                   ║
-- ║                                                                                       ║
-- ║ SOLUCIÓN ACORDADA: planilla maestra de vendedores cargada al warehouse, con el patrón ║
-- ║ que ya existe para Haulmer (`stg_google_sheets__haulmer_raw_pos2`). Columnas mínimas:  ║
-- ║   email · rol (KAM / ISR / otro) · pais · valid_from · valid_to                        ║
-- ║ El `valid_from`/`valid_to` es lo que hace la diferencia: permite atribuir según el rol ║
-- ║ que la persona tenía EN LA FECHA DE LA VENTA (point-in-time), no el que tiene hoy.     ║
-- ║ Sin eso, cada cambio de equipo reescribe la historia — es el mismo bug que tiene       ║
-- ║ `contacts.kam_pos_owner` y por el que lo bajamos al nivel 3.                           ║
-- ║                                                                                       ║
-- ║ Cuando exista, reemplazar los dos CTEs de abajo por lecturas de esa tabla y cambiar el ║
-- ║ nivel 2 de la regla de canal para exigir `rol = 'ISR'` en vez de "no está en el roster ║
-- ║ KAM". Es un cambio de 4 líneas: el resto de la jerarquía no se toca.                   ║
-- ╚═══════════════════════════════════════════════════════════════════════════════════════╝
kam_roster AS (                         -- roster canónico de KAM POS (validado con Ignacio 6-ago-2026)
    -- cristobal@ queda FUERA a propósito: era el LÍDER del área, no vendedor de cartera KAM.
    -- Aparece en `contacts.kam_pos_owner` (1 contacto) y hay que descartarlo.
    -- tabata@ SÍ va: está en el roster del código pero NO en kam_pos_owner, que está incompleto.
    -- ↓ FUTURO: SELECT owner_id, email FROM dwh.pos_sales_roster WHERE rol = 'KAM'
    --            AND sale_day BETWEEN valid_from AND valid_to
    SELECT o.id::varchar AS owner_id, LOWER(o.email) AS email
    FROM staging.stg_hubspot__owners o
    WHERE LOWER(o.email) SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%'
),
-- isr_roster AS (   ← NO EXISTE TODAVÍA. Mientras no exista, el nivel 2 asume ISR por exclusión.
--     SELECT owner_id, email FROM dwh.pos_sales_roster WHERE rol = 'ISR' ...
-- ),
deals AS (                              -- (1) owner del DEAL: POINT-IN-TIME, quien cerró ESA venta
    SELECT c.external_id::int AS company_id,
           d.closedate::date  AS closedate,
           CASE WHEN kr.owner_id IS NOT NULL THEN 1 ELSE 0 END AS deal_es_kam
    FROM staging.stg_hubspot__deals d
    JOIN staging.stg_hubspot__contacts c     ON c.id        = d.contact_id::bigint
    JOIN staging.stg_hubspot__deal_stages ds ON ds.stage_id = d.dealstage_id
    LEFT JOIN kam_roster kr                  ON kr.owner_id = d.hubspot_owner_id
    WHERE ds.pipeline_label = 'Seguimiento POS Expansión'
      AND ds.label = 'Ganado'
      AND c.external_id IS NOT NULL AND c.external_id <> ''
      AND d.closedate IS NOT NULL
),
kpo AS (                                -- (3) kam_pos_owner: ESTADO ACTUAL, no point-in-time
    -- ⚠ Solo como último recurso antes de la ventana. Error medido si se usa como autoridad:
    --   37 falsos positivos (ventas de no-KAM en contactos que HOY son propiedad KAM, 84% marcadas
    --   "new") + 34 falsos negativos (ventas de KAM sin la marca; rocio tiene 17 de sus 41 así)
    --   = 71 de 371 ventas 2026 mal clasificadas (19%). Es un campo de asignación de PROSPECCIÓN.
    -- No tiene columna de fecha (compárese con `optimization_owner_date`), así que no hay historia.
    SELECT c.external_id::int AS company_id, MAX(1) AS es_kam_hoy
    FROM staging.stg_hubspot__contacts c
    JOIN kam_roster kr ON kr.owner_id = c.kam_pos_owner   -- el join ya excluye a cristobal
    WHERE c.external_id IS NOT NULL AND c.external_id <> ''
    GROUP BY 1
),
-- ── MATCH ENTRE FUENTES: exacto por mes, con rescate a ±1 mes ───────────────────────────
-- El KAM a veces registra la venta en el mes en que la cerró y Chargebee factura en el mes
-- siguiente (o al revés). Con match por mes EXACTO eso produce DOS eventos para UNA sola venta.
-- Medido en 2026: 6 pares a 1 mes de distancia (ej. empresa 91759, 5 unidades: nuestra fecha
-- jul-26, la planilla dice jun-26). Los pares a 3+ meses (4 casos) SÍ son ventas distintas.
cb_libre AS (                           -- eventos de Chargebee sin match exacto en la planilla
    SELECT c.company_id, c.sale_month
    FROM cb c
    LEFT JOIN sh s ON s.company_id = c.company_id AND s.sale_month = c.sale_month
    WHERE s.company_id IS NULL
),
sh_libre AS (                           -- filas de la planilla sin match exacto en Chargebee
    SELECT s.company_id, s.sale_month
    FROM sh s
    LEFT JOIN cb c ON c.company_id = s.company_id AND c.sale_month = s.sale_month
    WHERE c.company_id IS NULL
),
pares AS (                              -- emparejamiento 1-a-1 a ±1 mes entre los que quedaron libres
    SELECT company_id, cb_month, sh_month
    FROM (
        SELECT a.company_id, a.sale_month AS cb_month, b.sale_month AS sh_month,
               ROW_NUMBER() OVER (PARTITION BY a.company_id, a.sale_month
                                  ORDER BY ABS(DATEDIFF(month, a.sale_month, b.sale_month)), b.sale_month) AS rn_cb,
               ROW_NUMBER() OVER (PARTITION BY a.company_id, b.sale_month
                                  ORDER BY ABS(DATEDIFF(month, a.sale_month, b.sale_month)), a.sale_month) AS rn_sh
        FROM cb_libre a
        JOIN sh_libre b ON b.company_id = a.company_id
                       AND ABS(DATEDIFF(month, a.sale_month, b.sale_month)) = 1
    ) x
    WHERE rn_cb = 1 AND rn_sh = 1        -- 1-a-1: ni dos planillas al mismo Chargebee ni al revés
),
sh_ali AS (                             -- la planilla, re-estampada al mes de Chargebee si hubo rescate
    SELECT s.company_id,
           COALESCE(p.cb_month, s.sale_month) AS sale_month,
           s.sale_month AS sale_month_sheet,  -- se conserva el mes que declaró el KAM
           s.sheet_day, s.units, s.es_react, s.kam, s.pais_sheet, s.producto
    FROM sh s
    LEFT JOIN pares p ON p.company_id = s.company_id AND p.sh_month = s.sale_month
),
meses AS (                              -- universo empresa × mes de venta (Chargebee ∪ planilla)
    SELECT company_id, sale_month FROM cb
    UNION
    SELECT company_id, sale_month FROM sh_ali
),
deal_near AS (                          -- el deal ganado más CERCANO a cada mes de venta
    SELECT company_id, sale_month, deal_es_kam
    FROM (
        SELECT m.company_id, m.sale_month, dl.deal_es_kam,
               ROW_NUMBER() OVER (PARTITION BY m.company_id, m.sale_month
                                  ORDER BY ABS(DATEDIFF(day, m.sale_month, dl.closedate))) AS rn
        FROM meses m
        JOIN deals dl ON dl.company_id = m.company_id
                     AND dl.closedate BETWEEN m.sale_month - 45 AND m.sale_month + 45
    ) x WHERE rn = 1
),
eventos AS (                            -- UNIÓN: todo evento de venta de cualquiera de las dos fuentes
    SELECT COALESCE(c.company_id, s.company_id)  AS company_id,
           COALESCE(c.sale_month, s.sale_month)  AS sale_month,
           COALESCE(c.sale_day, s.sheet_day, s.sale_month) AS sale_day,
           CASE WHEN c.company_id IS NOT NULL AND s.company_id IS NOT NULL THEN 'ambas'
                WHEN c.company_id IS NOT NULL THEN 'chargebee'
                ELSE 'sheet' END              AS fuente,
           COALESCE(c.sale_day_board, s.sheet_day, s.sale_month) AS sale_day_board,
           COALESCE(c.units, s.units)          AS units,
           c.units AS units_chargebee, s.units AS units_planilla,  -- para ver discrepancias
           c.agent_email, c.is_new_merchant,
           COALESCE(c.fecha_origen, CASE WHEN s.sheet_day IS NOT NULL THEN 'sheet' ELSE 'mes' END) AS fecha_origen,
           s.es_react, s.kam, s.pais_sheet, s.producto,
           s.sale_month_sheet,                 -- mes que declaró el KAM (≠ sale_month si hubo rescate ±1 mes)
           dn.deal_es_kam, k.es_kam_hoy,
           -- el día es real si Chargebee dio addon plausible o fecha de factura, o si viene del
           -- sheet a partir del 21-jul-2026 (antes el `dia` del sheet es default 1)
           CASE WHEN c.fecha_origen IN ('addon','factura') THEN 1
                WHEN s.sheet_day IS NOT NULL AND s.sheet_day >= DATE '2026-07-21' THEN 1
                ELSE 0 END                     AS fecha_confiable
    FROM cb c
    FULL OUTER JOIN sh_ali s
      ON s.company_id = c.company_id AND s.sale_month = c.sale_month
    LEFT JOIN deal_near dn ON dn.company_id = COALESCE(c.company_id, s.company_id)
                          AND dn.sale_month = COALESCE(c.sale_month, s.sale_month)
    LEFT JOIN kpo k        ON k.company_id  = COALESCE(c.company_id, s.company_id)
),
sec AS (                                -- orden histórico de eventos por empresa
    SELECT e.*,
           ROW_NUMBER() OVER (PARTITION BY e.company_id ORDER BY e.sale_month, e.sale_day) AS nro_evento,
           LAG(e.sale_month) OVER (PARTITION BY e.company_id ORDER BY e.sale_month, e.sale_day) AS mes_evento_previo
    FROM eventos e
),
act_previa AS (                         -- última transacción POS ANTES del evento (para derivar el tipo)
    SELECT s.company_id, s.sale_month,
           MAX(d.day::date) AS ult_trx_previa,
           SUM(d.pos_payment_count) AS trx_previas
    FROM sec s
    LEFT JOIN dwh.company_sales_days d
      ON d.company_id = s.company_id
     AND d.day::date < s.sale_day
     AND d.pos_payment_count > 0
    GROUP BY 1, 2
)
SELECT
    s.company_id,
    s.sale_month                                        AS cohort_month,   -- SIEMPRE el mes del evento
    DATE_TRUNC('week', s.sale_day)::date                AS cohort_week,
    s.sale_day,
    s.fecha_origen,                                     -- addon | factura | sheet | mes
    s.fecha_confiable,
    s.nro_evento,
    s.fuente,
    COALESCE(s.units, 1)                                AS units,
    CASE WHEN c.country_name IN ('Mexico','México') THEN 'Mexico'
         WHEN c.country_name IS NOT NULL THEN c.country_name
         ELSE COALESCE(s.pais_sheet, 'Otros') END       AS pais,
    -- ---------- CANAL: ownership primero, tiempo como último recurso ----------
    -- Jerarquía acordada 6-ago-2026. Principio: para atribuir un evento histórico solo sirven
    -- campos POINT-IN-TIME (fijos al momento de la venta). Los de ESTADO ACTUAL contaminan.
    CASE
        -- (0) La planilla KAM es el registro que el propio equipo lleva: si el evento está ahí, es KAM.
        WHEN s.fuente IN ('ambas','sheet')                  THEN 'KAM'
        -- (1) Owner del DEAL ganado: point-in-time, es quien cerró ESA venta. Solo desde abr-2026.
        WHEN s.deal_es_kam = 1                              THEN 'KAM'
        WHEN s.deal_es_kam = 0                              THEN 'ISR'
        -- (2) pos_sale_agent de Chargebee: point-in-time (quien ejecutó el addon ese día), 79% de
        --     cobertura en toda la historia. Ojo: es "quien ejecutó", proxy del vendedor — de ahí la
        --     cola de nombres con 1-2 ventas (soporte/administración).
        WHEN s.agent_email SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%' THEN 'KAM'
        WHEN s.agent_email IS NOT NULL
             AND s.agent_email NOT IN ('otros','full_access_key_v1')  THEN 'ISR'
        -- (3) kam_pos_owner: estado actual. Solo llega acá si no hubo planilla, deal ni agente
        --     (las ~65 ventas directas sin addon). Marcado como inferido en `canal_origen`.
        WHEN s.es_kam_hoy = 1                               THEN 'KAM'
        -- (4) Ventana de recencia desde el alta. Último recurso: <20% de los casos.
        WHEN c.created_at IS NULL
             THEN CASE WHEN s.is_new_merchant = 1 THEN 'ISR' ELSE 'KAM' END
        WHEN s.sale_month >= DATE '2026-06-01'
             THEN CASE WHEN DATEDIFF(day, c.created_at, s.sale_day) <= 30 THEN 'ISR' ELSE 'KAM' END
        ELSE CASE WHEN DATEDIFF(day, c.created_at, s.sale_day) <= 90 THEN 'ISR' ELSE 'KAM' END
    END                                                 AS canal,
    -- de dónde salió el canal: para poder auditar y para no leer una inferencia como dato duro
    CASE
        WHEN s.fuente IN ('ambas','sheet')                  THEN 'planilla'
        WHEN s.deal_es_kam IS NOT NULL                      THEN 'deal_hubspot'
        WHEN s.agent_email SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%'
          OR (s.agent_email IS NOT NULL
              AND s.agent_email NOT IN ('otros','full_access_key_v1')) THEN 'agente_chargebee'
        WHEN s.es_kam_hoy = 1                               THEN 'kam_pos_owner (inferido)'
        ELSE 'ventana_tiempo (inferido)'
    END                                                 AS canal_origen,
    -- ---------- TIPO: del sheet si existe; si no, derivado ----------
    CASE
        WHEN s.es_react = 1 THEN 'Reactivacion'
        WHEN s.es_react = 0 THEN 'Venta Nueva'
        WHEN s.nro_evento = 1 THEN 'Venta Nueva'
        WHEN a.ult_trx_previa IS NULL THEN 'Venta Nueva'
        WHEN DATEDIFF(day, a.ult_trx_previa, s.sale_day) >= 60 THEN 'Reactivacion (derivada)'
        ELSE 'POS adicional'
    END                                                 AS tipo_venta,
    CASE WHEN s.es_react IS NOT NULL THEN 'sheet' ELSE 'derivado' END AS tipo_venta_origen,
    -- ---------- VENDEDOR: quién queda atribuido (sin esto el canal no es auditable) ----------
    COALESCE(s.kam, SPLIT_PART(s.agent_email, '@', 1))  AS vendedor,
    CASE WHEN s.kam IS NOT NULL        THEN 'planilla'
         WHEN s.agent_email IS NOT NULL
              AND s.agent_email NOT IN ('otros','full_access_key_v1') THEN 'agente_chargebee'
         ELSE NULL END                                  AS vendedor_origen,
    s.kam                                               AS kam_declarado,
    s.agent_email                                       AS agente_chargebee,
    -- ---------- FLAGS DERIVADOS: se materializan para que nadie los re-derive mal ----------
    -- "Venta nueva pura" = la única población válida para activación y GPV. No basta excluir
    -- reactivaciones: las RECOMPRAS contaminan igual (un solo evento de recompra aportó $34,8K
    -- del GPV M0 de abr-26, el 94% de la celda).
    CASE WHEN s.es_react = 0 OR (s.es_react IS NULL AND s.nro_evento = 1)
              THEN CASE WHEN s.nro_evento = 1 AND COALESCE(a.trx_previas,0) = 0 THEN 1 ELSE 0 END
         ELSE 0 END                                     AS venta_nueva_pura,
    -- La etiqueta "Reactivación" de la planilla NO es confiable: de 23 declaradas, 5 no estaban
    -- frías (última trx 1-4 días antes) y concentraban el 93% del GPV del grupo. Este flag valida.
    CASE WHEN s.es_react = 1
              THEN CASE WHEN a.ult_trx_previa IS NULL THEN 0            -- nunca transaccionó
                        WHEN DATEDIFF(day, a.ult_trx_previa, s.sale_day) >= 60 THEN 1
                        ELSE 0 END
         ELSE NULL END                                  AS reactivacion_validada,
    s.producto,
    -- ---------- AUDITORÍA ----------
    s.sale_day_board,                                   -- fecha que usan hoy board y pos-ventas
    s.sale_month_sheet                                  AS mes_declarado_kam,  -- ≠ cohort_month si hubo rescate ±1 mes
    s.units_chargebee,
    s.units_planilla,
    s.mes_evento_previo,
    a.ult_trx_previa,
    COALESCE(a.trx_previas, 0)                          AS trx_previas,
    DATEDIFF(day, s.sale_day, CURRENT_DATE - 1)         AS dias_observados,
    CURRENT_DATE                                        AS calculado_at
FROM sec s
LEFT JOIN act_previa a ON a.company_id = s.company_id AND a.sale_month = s.sale_month
LEFT JOIN dwh.companies c ON c.company_id = s.company_id
WHERE s.sale_month >= DATE '2025-01-01'
  AND s.sale_month <  DATE_TRUNC('month', CURRENT_DATE + INTERVAL '1 month')
ORDER BY s.sale_month, s.company_id
```

## Dónde vive el número de ventas POS del Board Book

> Movido desde la tarea [[Locations POS y reactivaciones]] (T-053) el 24-sep-2026.

Proyecto: [[_SoT Ventas POS]]. El número que yo registro sale del **Board Book**, no de una planilla.

| Qué | Dónde |
|---|---|
| Repo | `agendapro-dashboards-evidence` |
| Reporte | https://evidence.agendaprops.com/finance/board-book — slide 8, "Payments" |
| Página | `pages/finance/board-book/index.md` |
| Gráfico | `POS Sales — units · Inbound (New) vs Stock · Chile & Mexico` (columnas `*_u`) |
| Query del dashboard | bloque SQL nombrado `pay_pos_sales_hist`, en esa misma página (DuckDB) |
| Query de extracción | `sources/dwh/payments/country_pos_sot.sql` → tabla `dwh.country_pos_sot` (PostgreSQL/Redshift) |
| Refrescar parquet | `npm run sources -- --queries "dwh.country_pos_sot"` |
| Otro consumidor | `pages/finance/board-book/quarterly/index.md` |

`country_pos_sot.sql` es el **port al repo del SoT de Ignacio** ([[SoT Ventas POS (documento base)]], raíz `POS_VENTAS_SOT.sql`), adoptado el 7-sep-2026. `country_pos.sql` quedó intacto a propósito porque lo leen otras 5 páginas y el denominador de payback.

Tablas del DWH que toca: `dwh.pos_sales2` (grano compañía × mes, trae el agente), `dwh.pos_sales` (facturas), `dwh.int_chargebee__subscription_addon_changes` (altas/bajas de addon POS), `dwh.pos_v2` (planilla KAM), `dwh.companies` y `staging.stg_hubspot__deals` / `__contacts` / `__deal_stages` / `__owners`.

**Cómo reproducir a nivel company_id**: tomar `country_pos_sot.sql`, borrar el `SELECT` agregado final y reemplazarlo por `SELECT * FROM clasificado WHERE country_group='Chile' AND date = DATE '2026-08-01'`. Para ver el porqué de cada clasificación hay que agregar `canal, fuente, agent_email, vendedor, sale_day` a la proyección del CTE `clasificado`, que por defecto no los expone. Queries relacionadas en [[SQL — SoT Ventas POS]].

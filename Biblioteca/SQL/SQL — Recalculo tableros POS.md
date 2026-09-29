---
type: query
temas: [pos, tableros]
proyecto: "[[_SoT Ventas POS]]"
lenguaje: sql (Redshift)
---

> Query original: `POS_RECALCULO_TABLEROS.sql` (copia literal).

```sql
-- =====================================================================================
-- ProPay · RECÁLCULO DE LOS DOS GRÁFICOS DE VENTAS POS  (2026-08-06)
-- Contrasta lo que muestran los tableros HOY contra la fuente única corregida.
--
-- Columnas de salida:
--   bd_u_new / bd_u_stk = réplica EXACTA de la extracción de Evidence (unidades = máquinas)
--   bd_com              = misma población del board, contada en COMERCIOS → efecto GRANO
--   sot_new/stk/rea     = fuente única en comercios, con 3ª categoría → efecto FUENTE
--   dif_com             = comercios que el tablero no está mostrando
--
-- Correcciones que aplica el bloque `sot`:
--   1) FECHA: addon POS más cercano al mes (de `int_chargebee__subscription_addon_changes`,
--      NO el `addon_added_at` de pos_sales2, que es `first_pos_sale_per_company`)
--      → `invoice_created_at` → día de planilla. Cero ventas sin fecha real en 2026.
--   2) REACTIVACIONES: FULL OUTER JOIN con dwh.pos_v2 — solo 18% de ellas genera factura.
--   3) TERCERA CATEGORÍA: New / Stock / Reactivación. El split binario del board no tiene
--      dónde poner una reactivación y hoy la fuerza a una caja que no le corresponde.
--
-- ══ DOS MODOS DE USO ═══════════════════════════════════════════════════════════════════════
-- MODO A (lo que está escrito abajo): CANAL CONGELADO. Usa `dia_canal` (fecha vieja) y la regla
--   del board (roster hardcodeado → ventana → is_new_merchant). Sirve para la reunión: las únicas
--   diferencias contra el tablero son de GRANO y de FUENTE, así que cada número es atribuible.
--
-- MODO B: CANAL NUEVO (jerarquía por ownership acordada 6-ago-2026, ver POS_VENTAS_SOT.md §3c).
--   Es el estado final recomendado. Para activarlo, agregar estos CTEs después de `sh`:
--
--     kam_roster AS (   -- roster validado; cristobal@ EXCLUIDO (era el líder del área)
--         SELECT o.id::varchar AS owner_id FROM staging.stg_hubspot__owners o
--         WHERE LOWER(o.email) SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%'),
--     deals AS (        -- owner del DEAL ganado: point-in-time
--         SELECT c.external_id::int AS company_id, d.closedate::date AS closedate,
--                CASE WHEN kr.owner_id IS NOT NULL THEN 1 ELSE 0 END AS deal_es_kam
--         FROM staging.stg_hubspot__deals d
--         JOIN staging.stg_hubspot__contacts c     ON c.id = d.contact_id::bigint
--         JOIN staging.stg_hubspot__deal_stages ds ON ds.stage_id = d.dealstage_id
--         LEFT JOIN kam_roster kr                  ON kr.owner_id = d.hubspot_owner_id
--         WHERE ds.pipeline_label='Seguimiento POS Expansión' AND ds.label='Ganado'
--           AND c.external_id IS NOT NULL AND c.external_id <> '' AND d.closedate IS NOT NULL),
--     kpo AS (          -- kam_pos_owner: estado actual, solo último recurso
--         SELECT c.external_id::int AS company_id, MAX(1) AS es_kam_hoy
--         FROM staging.stg_hubspot__contacts c
--         JOIN kam_roster kr ON kr.owner_id = c.kam_pos_owner
--         WHERE c.external_id IS NOT NULL AND c.external_id <> '' GROUP BY 1),
--     meses AS (SELECT company_id, sale_month FROM cb UNION SELECT company_id, sale_month FROM sh),
--     deal_near AS (    -- el deal ganado más cercano a cada mes de venta
--         SELECT company_id, sale_month, deal_es_kam FROM (
--             SELECT m.company_id, m.sale_month, dl.deal_es_kam,
--                    ROW_NUMBER() OVER (PARTITION BY m.company_id, m.sale_month
--                        ORDER BY ABS(DATEDIFF(day, m.sale_month, dl.closedate))) AS rn
--             FROM meses m JOIN deals dl ON dl.company_id = m.company_id
--                  AND dl.closedate BETWEEN m.sale_month - 45 AND m.sale_month + 45) x
--         WHERE rn = 1)
--
--   …unirlos en `eventos` (LEFT JOIN por company_id + mes) y reemplazar el CASE de `cat` por:
--
--     CASE WHEN e.es_react = 1                    THEN 'Reactivacion'
--          WHEN e.fuente IN ('ambas','sheet')     THEN 'Stock'
--          WHEN e.deal_es_kam = 1                 THEN 'Stock'
--          WHEN e.deal_es_kam = 0                 THEN 'New'
--          WHEN e.agent_email SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%'
--                                                 THEN 'Stock'
--          WHEN e.agent_email IS NOT NULL
--               AND e.agent_email NOT IN ('otros','full_access_key_v1') THEN 'New'
--          WHEN e.es_kam_hoy = 1                  THEN 'Stock'
--          WHEN e.created_at IS NULL THEN CASE WHEN e.is_new_merchant=1 THEN 'New' ELSE 'Stock' END
--          WHEN e.mes >= DATE '2026-06-01'
--               THEN CASE WHEN DATEDIFF(day, e.created_at, e.dia) <= 30 THEN 'New' ELSE 'Stock' END
--          ELSE CASE WHEN DATEDIFF(day, e.created_at, e.dia) <= 90 THEN 'New' ELSE 'Stock' END END
--
--   ⚠ Ojo: en MODO B usar `e.dia` (fecha corregida), NO `dia_canal`.
--   Efecto medido del MODO B en comercios, 2026: Chile New 110→120 y Stock 123→120; MÉXICO
--   New 42→32 (−24%) y Stock 57→80 (+40%). El grueso del cambio en México es que su "New" venía
--   del fallback `is_new_merchant` sin migrar (92 d sobre start_date_pago en vez de 30 d sobre
--   created_at), concentrado en la venta directa que no genera addon.
-- ═══════════════════════════════════════════════════════════════════════════════════════════
--
-- ⚠ Se agregan board y SoT POR SEPARADO y se unen con FULL OUTER JOIN a propósito: una misma
--   venta puede caer en semanas/meses distintos en cada versión — eso es justo lo que se corrige.
--
-- País: el board book usa `invoice_currency`; pos-ventas y el SoT usan `companies.country_name`.
-- Doc oficial: https://app.notion.com/p/agendapro/DWH-documentaiton-3b2ab34c233b806eaea8cd1d1909a395
-- =====================================================================================


-- #####################################################################################
-- QUERY 1 · Board Book › "POS Sales — Inbound (New) vs Stock" (Chile & Mexico)
--           Salida: MES × PAÍS.  Reemplaza sources/dwh/payments/country_pos.sql
-- #####################################################################################
WITH addon_evt AS (             -- TODOS los cambios de addon POS (sin el filtro rank = 1)
    SELECT company_id::int AS company_id, occurred_at::date AS addon_day
    FROM dwh.int_chargebee__subscription_addon_changes
    WHERE change_type IN ('added','initial')
      AND addon_id ~* 'pos|getnet|terminal|smart-pos'
      AND company_id IS NOT NULL AND company_id <> ''
),
addon_near AS (                 -- el addon más CERCANO a cada mes de venta (1 por empresa+mes)
    SELECT company_id, mes_d, addon_day
    FROM (
        SELECT m.company_id, m.mes_d, ae.addon_day,
               ROW_NUMBER() OVER (PARTITION BY m.company_id, m.mes_d
                                  ORDER BY ABS(DATEDIFF(day, m.mes_d, ae.addon_day))) AS rn
        FROM (SELECT DISTINCT company_id, TO_DATE(month,'YYYYMM') AS mes_d FROM dwh.pos_sales2) m
        JOIN addon_evt ae ON ae.company_id = m.company_id
                         AND ae.addon_day BETWEEN m.mes_d - 31 AND m.mes_d + 62
    ) x WHERE rn = 1
),
inv AS (                        -- emisión de factura (NO cobranza): tipo date, 100% poblada
    SELECT invoice_id, MIN(COALESCE(invoice_created_at, invoice_paid_at)) AS invoice_day
    FROM dwh.pos_sales GROUP BY 1
),
dedup AS (                      -- dedup canónico jul-2026: 1 venta por empresa+día
    SELECT * FROM (
        SELECT ps.company_id, ps.item_qty, ps.is_new_merchant, ps.invoice_currency,
               ps.addon_added_at, ps.month,
               LOWER(COALESCE(ps.pos_sale_agent, ps.hubspot_owner_email,
                              ps.corrected_agent_email,'otros')) AS agent_email,
               an.addon_day, iv.invoice_day,
               ROW_NUMBER() OVER (
                   PARTITION BY ps.company_id, COALESCE(ps.addon_added_at::date, TO_DATE(ps.month,'YYYYMM'))
                   ORDER BY ps.item_qty DESC, ps.addon_added_at DESC NULLS LAST, ps.sale_id) AS rn
        FROM dwh.pos_sales2 ps
        LEFT JOIN inv iv        ON iv.invoice_id = ps.first_invoice_id
        LEFT JOIN addon_near an ON an.company_id = ps.company_id
                               AND an.mes_d     = TO_DATE(ps.month,'YYYYMM')
    ) x WHERE rn = 1
),
cb AS (                         -- Chargebee con las tres fechas en paralelo
    SELECT d.company_id, d.item_qty, d.agent_email, d.is_new_merchant, d.addon_added_at,
           COALESCE(d.addon_added_at::date, TO_DATE(d.month,'YYYYMM'))       AS dia_board,
           COALESCE(d.addon_day, d.invoice_day, TO_DATE(d.month,'YYYYMM'))   AS dia_sot,
           CASE WHEN d.invoice_currency='CLP' THEN 'Chile'
                WHEN d.invoice_currency='MXN' THEN 'Mexico' ELSE 'Otros' END AS pais_board,
           CASE WHEN c.country_name IN ('Mexico','México') THEN 'Mexico'
                ELSE COALESCE(c.country_name,'Otros') END                    AS pais_sot,
           c.created_at
    FROM dedup d LEFT JOIN dwh.companies c ON c.company_id = d.company_id
),
sh AS (                         -- planilla KAM: única fuente de reactivaciones
    SELECT company_id, TO_DATE(ano::varchar || LEFT(mes,2),'YYYYMM') AS mes_sh,
           MAX(CASE WHEN dia BETWEEN 1 AND 31
                    THEN TO_DATE(ano::varchar||LEFT(mes,2)||LPAD(dia::varchar,2,'0'),'YYYYMMDD') END) AS sheet_day,
           MAX(CASE WHEN LOWER(tipo_venta) LIKE '%reactiva%' THEN 1 ELSE 0 END) AS es_react,
           MAX(CASE WHEN pais='México' THEN 'Mexico' ELSE pais END)            AS pais_sheet
    FROM dwh.pos_v2 WHERE company_id IS NOT NULL GROUP BY 1,2
),
agg_board AS (                  -- réplica del board: unidades + la misma población en comercios
    SELECT mes, pais,
           SUM(CASE WHEN cat='New'   THEN unid ELSE 0 END) AS unid_new,
           SUM(CASE WHEN cat='Stock' THEN unid ELSE 0 END) AS unid_stock,
           COUNT(DISTINCT company_id)                      AS comercios
    FROM (
        SELECT DATE_TRUNC('month', dia_board)::date AS mes, pais_board AS pais,
               company_id, item_qty AS unid,
               CASE WHEN agent_email SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%' THEN 'Stock'
                    WHEN created_at IS NULL OR addon_added_at IS NULL
                         THEN CASE WHEN is_new_merchant=1 THEN 'New' ELSE 'Stock' END
                    WHEN DATE_TRUNC('month', dia_board) >= DATE '2026-06-01'
                         THEN CASE WHEN DATEDIFF(day, created_at, addon_added_at) <= 30 THEN 'New' ELSE 'Stock' END
                    ELSE CASE WHEN DATEDIFF(day, created_at, addon_added_at) <= 90 THEN 'New' ELSE 'Stock' END
               END AS cat
        FROM cb
    ) b GROUP BY 1,2
),
eventos AS (                    -- UNIÓN Chargebee + planilla, grano empresa × mes
    SELECT COALESCE(cb.company_id, sh.company_id)                      AS company_id,
           COALESCE(DATE_TRUNC('month', cb.dia_sot)::date, sh.mes_sh)  AS mes,
           COALESCE(cb.dia_board, sh.sheet_day, sh.mes_sh)             AS dia_canal,  -- canal congelado
           cb.agent_email, cb.is_new_merchant, cb.created_at, cb.addon_added_at, sh.es_react,
           CASE WHEN cb.company_id IS NOT NULL THEN 1 ELSE 0 END       AS en_chargebee,
           COALESCE(cb.pais_sot, sh.pais_sheet, 'Otros')               AS pais
    FROM cb FULL OUTER JOIN sh ON sh.company_id = cb.company_id
                              AND sh.mes_sh     = DATE_TRUNC('month', cb.dia_sot)::date
),
agg_sot AS (
    SELECT mes, pais,
           COUNT(DISTINCT CASE WHEN cat='New'          THEN company_id END) AS com_new,
           COUNT(DISTINCT CASE WHEN cat='Stock'        THEN company_id END) AS com_stock,
           COUNT(DISTINCT CASE WHEN cat='Reactivacion' THEN company_id END) AS com_react,
           COUNT(DISTINCT company_id)                                       AS com_total
    FROM (
        SELECT e.*,
               CASE WHEN e.es_react = 1     THEN 'Reactivacion'
                    WHEN e.en_chargebee = 0 THEN 'Stock'   -- solo en planilla ⇒ es del equipo KAM
                    WHEN e.agent_email SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%' THEN 'Stock'
                    WHEN e.created_at IS NULL OR e.addon_added_at IS NULL
                         THEN CASE WHEN e.is_new_merchant=1 THEN 'New' ELSE 'Stock' END
                    WHEN e.mes >= DATE '2026-06-01'
                         THEN CASE WHEN DATEDIFF(day, e.created_at, e.dia_canal) <= 30 THEN 'New' ELSE 'Stock' END
                    ELSE CASE WHEN DATEDIFF(day, e.created_at, e.dia_canal) <= 90 THEN 'New' ELSE 'Stock' END
               END AS cat
        FROM eventos e
    ) z GROUP BY 1,2
)
SELECT TO_CHAR(COALESCE(s.mes,b.mes),'YYYY-MM')      AS mes,
       COALESCE(s.pais,b.pais)                       AS pais,
       COALESCE(b.unid_new,0)                        AS bd_u_new,
       COALESCE(b.unid_stock,0)                      AS bd_u_stk,
       COALESCE(b.comercios,0)                       AS bd_com,
       COALESCE(s.com_new,0)                         AS sot_new,
       COALESCE(s.com_stock,0)                       AS sot_stk,
       COALESCE(s.com_react,0)                       AS sot_rea,
       COALESCE(s.com_total,0)                       AS sot_total,
       COALESCE(s.com_total,0) - COALESCE(b.comercios,0) AS dif_com
FROM agg_sot s FULL OUTER JOIN agg_board b ON b.mes = s.mes AND b.pais = s.pais
WHERE COALESCE(s.mes,b.mes) >= DATE '2026-01-01'
  AND COALESCE(s.pais,b.pais) IN ('Chile','Mexico')
ORDER BY 1,2;


-- #####################################################################################
-- QUERY 2 · POS Ventas › "POS vendidos por semana (canal)"
--           Salida: SEMANA × PAÍS.  Reemplaza sources/dwh/payments_weekly/pos_sales_daily.sql
--
-- Única diferencia estructural con la Query 1: el board aquí usa `sale_date = addon_added_at::date`
-- y FILTRA `sale_date IS NOT NULL` — por eso `agg_board` lleva `WHERE dia_board IS NOT NULL`.
-- Ahí es donde se pierde el 9-35% de las ventas (y en México semanas enteras en cero).
-- #####################################################################################
WITH addon_evt AS (
    SELECT company_id::int AS company_id, occurred_at::date AS addon_day
    FROM dwh.int_chargebee__subscription_addon_changes
    WHERE change_type IN ('added','initial')
      AND addon_id ~* 'pos|getnet|terminal|smart-pos'
      AND company_id IS NOT NULL AND company_id <> ''
),
addon_near AS (
    SELECT company_id, mes_d, addon_day
    FROM (
        SELECT m.company_id, m.mes_d, ae.addon_day,
               ROW_NUMBER() OVER (PARTITION BY m.company_id, m.mes_d
                                  ORDER BY ABS(DATEDIFF(day, m.mes_d, ae.addon_day))) AS rn
        FROM (SELECT DISTINCT company_id, TO_DATE(month,'YYYYMM') AS mes_d FROM dwh.pos_sales2) m
        JOIN addon_evt ae ON ae.company_id = m.company_id
                         AND ae.addon_day BETWEEN m.mes_d - 31 AND m.mes_d + 62
    ) x WHERE rn = 1
),
inv AS (
    SELECT invoice_id, MIN(COALESCE(invoice_created_at, invoice_paid_at)) AS invoice_day
    FROM dwh.pos_sales GROUP BY 1
),
dedup AS (
    SELECT * FROM (
        SELECT ps.company_id, ps.item_qty, ps.is_new_merchant, ps.addon_added_at, ps.month,
               LOWER(COALESCE(ps.pos_sale_agent, ps.hubspot_owner_email,
                              ps.corrected_agent_email,'otros')) AS agent_email,
               an.addon_day, iv.invoice_day,
               ROW_NUMBER() OVER (
                   PARTITION BY ps.company_id, COALESCE(ps.addon_added_at::date, TO_DATE(ps.month,'YYYYMM'))
                   ORDER BY ps.item_qty DESC, ps.addon_added_at DESC NULLS LAST, ps.sale_id) AS rn
        FROM dwh.pos_sales2 ps
        LEFT JOIN inv iv        ON iv.invoice_id = ps.first_invoice_id
        LEFT JOIN addon_near an ON an.company_id = ps.company_id
                               AND an.mes_d     = TO_DATE(ps.month,'YYYYMM')
    ) x WHERE rn = 1
),
cb AS (
    SELECT d.company_id, d.item_qty, d.agent_email, d.is_new_merchant, d.addon_added_at,
           d.addon_added_at::date                                          AS dia_board,   -- puede ser NULL
           COALESCE(d.addon_added_at::date, TO_DATE(d.month,'YYYYMM'))      AS dia_canal,
           COALESCE(d.addon_day, d.invoice_day, TO_DATE(d.month,'YYYYMM'))  AS dia_sot,
           CASE WHEN c.country_name IN ('Mexico','México') THEN 'Mexico'
                ELSE COALESCE(c.country_name,'Otros') END                   AS pais,
           c.created_at
    FROM dedup d LEFT JOIN dwh.companies c ON c.company_id = d.company_id
),
sh AS (
    SELECT company_id, TO_DATE(ano::varchar || LEFT(mes,2),'YYYYMM') AS mes_sh,
           MAX(CASE WHEN dia BETWEEN 1 AND 31
                    THEN TO_DATE(ano::varchar||LEFT(mes,2)||LPAD(dia::varchar,2,'0'),'YYYYMMDD') END) AS sheet_day,
           MAX(CASE WHEN LOWER(tipo_venta) LIKE '%reactiva%' THEN 1 ELSE 0 END) AS es_react,
           MAX(CASE WHEN pais='México' THEN 'Mexico' ELSE pais END)            AS pais_sheet
    FROM dwh.pos_v2 WHERE company_id IS NOT NULL GROUP BY 1,2
),
agg_board AS (
    SELECT DATE_TRUNC('week', dia_board)::date AS semana, pais,
           SUM(CASE WHEN cat='New'   THEN unid ELSE 0 END) AS unid_new,
           SUM(CASE WHEN cat='Stock' THEN unid ELSE 0 END) AS unid_stock,
           COUNT(DISTINCT company_id)                      AS comercios
    FROM (
        SELECT company_id, item_qty AS unid, dia_board, pais,
               CASE WHEN agent_email SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%' THEN 'Stock'
                    WHEN created_at IS NULL OR addon_added_at IS NULL
                         THEN CASE WHEN is_new_merchant=1 THEN 'New' ELSE 'Stock' END
                    WHEN DATE_TRUNC('month', dia_canal) >= DATE '2026-06-01'
                         THEN CASE WHEN DATEDIFF(day, created_at, addon_added_at) <= 30 THEN 'New' ELSE 'Stock' END
                    ELSE CASE WHEN DATEDIFF(day, created_at, addon_added_at) <= 90 THEN 'New' ELSE 'Stock' END
               END AS cat
        FROM cb
        WHERE dia_board IS NOT NULL          -- ← el filtro que descarta 9-35% de las ventas
    ) w GROUP BY 1,2
),
eventos AS (
    SELECT COALESCE(cb.company_id, sh.company_id)                     AS company_id,
           COALESCE(cb.dia_sot, sh.sheet_day, sh.mes_sh)              AS dia,
           COALESCE(cb.dia_canal, sh.sheet_day, sh.mes_sh)            AS dia_canal,
           COALESCE(DATE_TRUNC('month', cb.dia_sot)::date, sh.mes_sh) AS mes,
           cb.agent_email, cb.is_new_merchant, cb.created_at, cb.addon_added_at, sh.es_react,
           CASE WHEN cb.company_id IS NOT NULL THEN 1 ELSE 0 END      AS en_chargebee,
           COALESCE(cb.pais, sh.pais_sheet, 'Otros')                  AS pais
    FROM cb FULL OUTER JOIN sh ON sh.company_id = cb.company_id
                              AND sh.mes_sh     = DATE_TRUNC('month', cb.dia_sot)::date
),
agg_sot AS (
    SELECT DATE_TRUNC('week', dia)::date AS semana, pais,
           COUNT(DISTINCT CASE WHEN cat='New'          THEN company_id END) AS com_new,
           COUNT(DISTINCT CASE WHEN cat='Stock'        THEN company_id END) AS com_stock,
           COUNT(DISTINCT CASE WHEN cat='Reactivacion' THEN company_id END) AS com_react,
           COUNT(DISTINCT company_id)                                       AS com_total
    FROM (
        SELECT e.*,
               CASE WHEN e.es_react = 1     THEN 'Reactivacion'
                    WHEN e.en_chargebee = 0 THEN 'Stock'
                    WHEN e.agent_email SIMILAR TO '(rocio|janet|jaime|fernandobarra|alvaro|tabata)@%' THEN 'Stock'
                    WHEN e.created_at IS NULL OR e.addon_added_at IS NULL
                         THEN CASE WHEN e.is_new_merchant=1 THEN 'New' ELSE 'Stock' END
                    WHEN e.mes >= DATE '2026-06-01'
                         THEN CASE WHEN DATEDIFF(day, e.created_at, e.dia_canal) <= 30 THEN 'New' ELSE 'Stock' END
                    ELSE CASE WHEN DATEDIFF(day, e.created_at, e.dia_canal) <= 90 THEN 'New' ELSE 'Stock' END
               END AS cat
        FROM eventos e
    ) z GROUP BY 1,2
)
SELECT TO_CHAR(COALESCE(s.semana,b.semana),'YYYY-MM-DD') AS semana,
       COALESCE(s.pais,b.pais)                           AS pais,
       COALESCE(b.unid_new,0)                            AS bd_u_new,
       COALESCE(b.unid_stock,0)                          AS bd_u_stk,
       COALESCE(b.comercios,0)                           AS bd_com,
       COALESCE(s.com_new,0)                             AS sot_new,
       COALESCE(s.com_stock,0)                           AS sot_stk,
       COALESCE(s.com_react,0)                           AS sot_rea,
       COALESCE(s.com_total,0)                           AS sot_total,
       COALESCE(s.com_total,0) - COALESCE(b.comercios,0) AS dif_com
FROM agg_sot s FULL OUTER JOIN agg_board b ON b.semana = s.semana AND b.pais = s.pais
WHERE COALESCE(s.semana,b.semana) >= DATE '2026-06-01'
  AND COALESCE(s.pais,b.pais) IN ('Chile','Mexico')
ORDER BY 1,2;
```

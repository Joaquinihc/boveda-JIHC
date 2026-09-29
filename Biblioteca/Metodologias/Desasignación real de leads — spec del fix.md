---
type: metodologia
temas: [leads, hubspot, adquisicion]
metricas: ["[[Lead]]"]
fuente: Carlos Rojas (2026-08-06)
fecha: 2026-08-06
---

# Desasignación real de leads — diagnóstico completo, queries y spec del fix

**Para:** Pablo Lucero · Joaquín Herrera (documento pensado para ser procesado por su Claude — incluye contexto de tablas DWH y SQL reproducible)
**Preparado:** 2026-08-06 · Carlos Rojas
**Objetivo:** que el tablero de Evidence [Asignación de Leads](https://evidence.agendaprops.com/acquisition/lead-assignment) muestre la **desasignación real**, y cerrar la definición de los casos OFFLINE/original-source-vacío.

---

## 0 · TL;DR

El "% de desasignación" del tablero (**9,6–9,8% mes en curso, 7,6% último cerrado**) está inflado por componentes que **no son desasignación del proceso**:

| Componente | Magnitud medida | ¿Es desasignación real? |
|---|---|---|
| Contactos de **prueba internos** del equipo | Ago: 8 de los 13 "sin dueño" core · Jul Chile: **30 de 41** | ❌ |
| **Otros LATAM** (fuera de MX/CL/AR/CO) | 42 de ~52 sin owner (1–5 ago) | ❌ (no rutean al ISR core) |
| Leads **en ventana de asignación** (<48h) | 2 de los 13 (creados el mismo día del corte) | ❌ todavía |
| Eco del **incidente del workflow 1–3 ago** (ya revertido) | 164 leads el 03-ago, reasignados | ❌ (foto vieja: el snapshot corre con lag) |
| **Leads reales sin asignar** | **1** (Meli Rosales, AR — ya posteada a #leads-sin-asignar) | ✅ |

**Desasignación real de agosto en los 4 países core: 1 lead.** Verificado contacto por contacto contra el estado vivo de HubSpot (sección 2, IDs en sección 6).

---

## 1 · Cómo leer el DWH para esto (contexto para Claude)

Todas las queries corren en el **Postgres del DWH**. Tablas involucradas y sus trampas:

| Tabla | Qué es | ⚠️ Trampa |
|---|---|---|
| `dwh.int_hubspot_leads_flat` | Fuente del tablero de Evidence. 1 fila por contacto-lead con `effective_owner_id = COALESCE(deal_owner_id, contact_owner_id)` | **El campo `email` viene hasheado (MD5)** → clasificar internos con `email ILIKE '%agendapro%'` da 0 matches siempre. Hay que resolver el email en otra tabla |
| `dwh.hubspot_contacts` | Contactos con **email en texto plano** | **Rezagada 1–3 días** en el sync HubSpot→DWH: contactos muy nuevos aún no existen (por eso 2 de los 13 de agosto quedan "sin resolver") |
| `dwh.stg_hubspot__contacts` | Estado **vivo/fresco** del contacto (owner, source, lifecycle) | `email` también MD5. Usarla para verificar owner actual, no para clasificar internos |
| `dwh.fct_lead_conversions` | Conversiones flaggeadas — el universo de "leads de marketing" que usan los dashboards de marketing (All Channels, Gap Analysis, Pace) | Se arma desde los flags `reconversion<mes>` — leads que no llegan a flagear no existen acá |
| `dwh.hubspot_owners` | `id` → nombre del asesor | `id` numérico; castear para joinear |

**Corte B2B3+** (el mismo en todos los universos):
```sql
UPPER(TRIM(COALESCE(numemployees,''))) IN
  ('3-5','5-16','6-15','16','16+','+16','5-25','25-50','50-100','1000+','6+')
```

**Definición de interno/test** (solo funciona con email plano, i.e. vía `dwh.hubspot_contacts`):
```sql
email ILIKE '%agendapro%' OR email ILIKE '%test%'
OR email ILIKE '%prueba%' OR email ILIKE '%yopmail%'
```
(atrapa `@agendapro.com`, `+agendapro@gmail`, `emilio+ar2@...`, etc.)

---

## 2 · Auditoría de los "sin dueño" — el gap explicado componente por componente

### 2.1 Contactos de prueba internos (el componente más grande)

**Query Q1 — sin-owner del mes en el universo del tablero:**
```sql
SELECT f.contact_id, f.country, f.sector_economico, f.numemployees,
       f.lifecyclestage, f.createdate::date, f.recent_conversion_date::date,
       f.hs_analytics_source
FROM dwh.int_hubspot_leads_flat f
WHERE f.country IN ('México','Chile','Argentina','Colombia')
  AND UPPER(TRIM(COALESCE(f.numemployees,''))) IN
      ('3-5','5-16','6-15','16','16+','+16','5-25','25-50','50-100','1000+','6+')
  AND f.recent_conversion_date >= '2026-08-01'
  AND f.effective_owner_id IS NULL;
```

**Query Q2 — resolver emails en texto plano y clasificar:**
```sql
SELECT hc.id AS contact_id, hc.email, hc.firstname, hc.lastname
FROM dwh.hubspot_contacts hc
WHERE hc.id IN (/* contact_ids de Q1 */);
```

**Resultado agosto (1–6):** 13 sin dueño en los 4 países core → **8 pruebas confirmadas** (emilio+ar2/ar3/col1/col2@agendapro, fernandoherrera@, andressureda@, alanortiz+lppilates@, diego.romero.v13+agendapro@gmail). Las fechas calzan con los tests del equipo post-incidente del workflow (§2.4).

**Resultado julio Chile (mismo método):** 41 sin dueño de 279 leads B2B3 = 14,7% según el tablero → **30 de los 41 son pruebas internas**. Los 11 reales dan **3,9%** de desasignación.

> El detalle del tablero ya trae la marca `Interno (test)` / `Externo`, pero **el % headline y las series las cuentan igual**. Ahí está el grueso del gap.

### 2.2 Otros LATAM

**Query Q3 — sin-owner por país incluyendo LATAM (universo marketing, estado vivo):**
```sql
SELECT
  CASE WHEN c.country IN ('México','Chile','Argentina','Colombia')
       THEN c.country ELSE 'Otros LATAM' END AS pais,
  COUNT(*) AS leads_b2b3,
  COUNT(*) FILTER (WHERE c.hubspot_owner_id IS NULL) AS sin_owner
FROM dwh.fct_lead_conversions lc
JOIN dwh.stg_hubspot__contacts c ON lc.contact_id = c.id
WHERE lc.conversion_date >= '2026-08-01' AND lc.conversion_date < '2026-08-06'
  AND c.country IS NOT NULL AND c.recent_conversion_date IS NOT NULL
  AND UPPER(COALESCE(c.hs_analytics_source,'')) != 'OFFLINE'
  AND UPPER(TRIM(COALESCE(c.numemployees,''))) IN
      ('3-5','5-16','6-15','16','16+','+16','5-25','25-50','50-100','1000+','6+')
GROUP BY 1;
```
**Resultado (1–5 ago):** Otros LATAM = 128 leads, **42 sin owner (33%)**; los 4 core = 621 leads, 10 sin owner (todos rubro "otro" MX, que por regla no se asigna). Otros LATAM no rutea al equipo ISR core → no debería estar en el % headline.

### 2.3 Ventana de asignación (<48h)

2 de los 13 de agosto fueron creados **el mismo día del corte** (06-ago) — están dentro de la ventana normal del workflow. Un lead de hace 2 horas sin owner no es desasignación.

### 2.4 El incidente del workflow (1–3 ago) y el lag del snapshot

Cronología (todo en #comité-adquisición): el viernes **31-jul** se cambió el workflow de asignación → el lunes **03-ago** amanecieron 244 contactos de agosto con solo 80 asignados (**164 sin asignar, todos los países** — reporte de Alan Ortiz 11:04). Alex lo revirtió ese mismo día (Alan 12:32: "ya se corrigió el branch y el workflow") y los pendientes se reasignaron. **Verificado en vivo: CL/AR/CO hoy tienen 0 leads reales sin owner.** Como el Evidence se refresca cada ~9h sobre un DWH que corre con 1–2 días de lag (Airbyte+dbt), su "mes en curso" siguió mostrando el incidente días después de resuelto.

### 2.5 Diferencia de universos (por qué marketing ve otro número)

Los dashboards de marketing cuentan leads desde `fct_lead_conversions` (excluye original source OFFLINE y, en la práctica, los no-flaggeados); el tablero de asignación cuenta desde `int_hubspot_leads_flat` (todo contacto con `recent_conversion_date`). Ninguno es "el equivocado" — pero **el % de desasignación debe calcularse sobre contactos asignables**: externos, países core, fuera de ventana.

---

## 3 · Spec del fix del tablero (lo que pedimos)

1. **Excluir contactos internos/test del numerador y denominador.** Resolver email plano vía `dwh.hubspot_contacts` (regla §1) o propagar la marca `Interno (test)` que el detalle ya tiene al % headline y a las series. Nota: si se clasifica dentro de `int_hubspot_leads_flat`, hay que agregar el email plano al modelo dbt — el campo actual está hasheado.
2. **Otros LATAM fuera del % headline** — mostrarlo como serie/fila aparte si interesa, pero no mezclado con la operación core.
3. **Leads con `createdate < 48h` como "en ventana de asignación"**, no como "sin dueño".
4. Mantener y explotar el split **"nunca asignado" vs "des-asignado"** que ya existe — son problemas distintos (routing vs pérdida de owner).

**Criterio de aceptación:** recalculando julio-Chile con las reglas 1–3, el % debe bajar de **14,7% → ~3,9%** (11 leads reales de 279). Agosto core debería dar **~0–1** sin dueño real.

---

## 4 · Problemática de fondo A: original source OFFLINE o vacío (inmutable)

HubSpot estampa el **original source (first-touch) en la creación del contacto y es inmutable**. Cuando el contacto se crea por una vía no-web *antes* de llenar un formulario, queda OFFLINE (o vacío) para siempre:

```
Intercom / CRM_UI crea el contacto  →  original source = OFFLINE (fijado)
Contacto llena form real después    →  original source NO cambia
                                       latest source SÍ lo refleja ("el latest nunca miente")
```

**Vías de creación detectadas** (jul–ago 2026, B2B3+, MX/CL/AR/CO — 17 contactos, Query Q4 en anexo):
- `INTEGRATION / 169804` y `38589` (Intercom y afines): 4
- `CRM_UI / userId:*` (asesores creando el contacto a mano — **no debería asignarse**): 12
- Sin data: 1

**Dimensión real, abierta por latest source:**

| Latest source | n | ¿Lead real? |
|---|---|---|
| `go.agendapro.com/lp/form-ventas` | 5 | ❌ Form interno del equipo de ventas |
| `www.agendapro.com/quick_add` | 3 | ⚠️ **Posible lead** — evaluar |
| `go.agendapro.com/lp/ventas-landing-cuenta` | 1 | ❌ Interno |
| `agendapro.com/co/signup` | 3 | ⚠️ **Posible lead** — evaluar |
| Nunca llenó form (CRM_UI puro) | 1 | ❌ |
| **Facebook lead ad** (colombia - estetica - saas) | **1** | ✅ **Real — y sin owner** |
| **Paid Search brand CO** (900_branding) | **1** | ✅ Real (con owner) |
| Direct sitio (`agendapro.com/ar`, `/cl`) | 2 | ✅ Reales (con owner) |

→ **~4 leads reales confirmados + 6 posibles (quick_add/signup) en 5 semanas** — entre ~2 y ~8/mes según cómo se resuelvan los posibles.

**Soluciones posibles:**

- **A · Status quo + caso a caso** *(recomendada por volumen)*: mantener la exclusión; el canal #leads-sin-asignar alertea los OFFLINE B2B3 y se evalúa uno por uno.
- **B · Rescate estrecho por latest source (solo atribución)**: **alcance duro — aplica exclusivamente cuando `original source ∈ {OFFLINE, vacío/null}`**; un original real nunca se pisa. Dentro de ese universo, contar el lead cuando `latest` trae señal de campaña/form real, bucketeado por latest, con blacklist explícita de internos (`form-ventas`, `ventas-landing-cuenta`) y `quick_add`/`signup` como posibles a evaluar. ⚠️ Un rescate amplio por "latest ≠ OFFLINE" importaría forms internos al conteo. Requiere tocar all_channels, gap_analysis y Evidence a la vez.
- **C · Fix upstream**: campo custom `fuente_efectiva` estampado por workflow con el latest del **primer form real** — sin pelear contra el original. Dashboards leen `COALESCE(fuente_efectiva, original)`. Más robusto; requiere ops.
- **D · Parche de asignación (urgente, independiente de atribución)**: contacto con original OFFLINE/vacío + form real + MQL **debe rutear** (caso Danays, §6).

## 5 · Problemática de fondo B: race condition en la asignación

Las propiedades analytics (`hs_analytics_source` y familia) se escriben **asíncronamente** — segundos a minutos después de crear el contacto:

```
Llega el lead → WF de asignación dispara YA → original source todavía vacío
             → el branch que filtra por source no matchea → NO se asigna
```

**Mitigación actual (Carlos):** delays + branches en los workflows. Es la capa correcta — la asignación solo se arregla en HubSpot. Pero ningún delay fijo gana la carrera el 100% de las veces. Afinamientos a discutir con ops:

1. **Desacoplar asignación de original source**: disparar el WF desde el **form submission** (evento explícito, sin carrera) y dejar el analytics source solo para reporting.
2. **Branch catch-all con timeout**: "source unknown tras X min → **asignar igual** por rotación + marcar `asignado_sin_source`". Un lead mal atribuido se corrige; uno sin asignar se enfría. La rama unknown sin salida es la candidata a explicar los leads reales atascados en lifecycle `lead` que nunca llegan a MQL (Chile julio: 11).
3. **Red de seguridad medible**: #leads-sin-asignar (alertas) + auditoría diaria de leads maduros >48h sin owner contra HubSpot vivo.
4. **Métrica de control**: latencia creación→asignación por país/cohorte semanal; post-fix la cola >1h debería ser ~0. Pendiente de medir.

---

## 6 · Casos testigo (accionables hoy)

**Meli Rosales** — `meliiirosales16@gmail.com` (contact `239543610993`) · Argentina · salón de belleza · **6-15 empleados** · Social Media · creada 08-03. Real, quedó en lifecycle `lead` sin llegar a MQL → nunca ruteó. **Ya posteada a #leads-sin-asignar (06-ago) para asignación manual.**

**Danays Banda** — `bandadanays991@gmail.com` (contact `236994035476`) · Colombia · manicure_pedicure · 3-5 · Creada por Intercom (169804) el 07-22 → llenó **lead ad de Meta** ("colombia - estetica - lead ad - saas") → llegó a **MQL** → **sin owner desde hace 2 semanas**. No cuenta como Paid Social en ningún dashboard y nadie la trabaja. Es el caso que motiva las soluciones B/C/D del §4.

Los 13 sin-dueño de agosto, con veredicto contacto por contacto, están en `~/Downloads/leads-sin-owner-agosto-2026.md`.

---

## 7 · Preguntas para cerrar la definición

1. **Pablo**: ¿aplicamos el spec del §3 al tablero (internos fuera, LATAM aparte, ventana <48h)? ¿El universo "leads trabajados" ya excluye los forms internos (`form-ventas`, `quick_add`)?
2. **Joaquín**: para la fuente única de verdad que pidió Julio (05-ago), ¿OFFLINE/vacío queda como exclusión dura (A), rescate estrecho (B) o campo corregido (C)? Sea cual sea, que Evidence y los dashboards de marketing usen la misma.
3. **Ambos**: ¿blacklist canónica de landings/forms internos versionada en un solo lugar (seed de dbt) compartida por todos los universos?
4. **Ops (Alan/Alex)**: cerrar el hueco D — OFFLINE/vacío + form real + MQL debe rutear. ¿Lo cubre "ISR - Rotation MQL" o necesita branch nuevo?
5. **Ops (Alan/Alex)**: sobre la race condition (§5) — ¿los delays/branches actuales tienen rama catch-all para source unknown, o esa rama muere sin asignar? ¿Podemos mover el trigger a form submission y/o agregar `asignado_sin_source`? Los CRM_UI (creados a mano por asesores) quedan explícitamente fuera del routing.

## 8 · Recomendación

**Fix del tablero (§3) + parche de asignación (D) ahora**; OFFLINE como status quo + caso a caso (A) mientras el volumen sea ~2–8/mes; **C como fix definitivo** si el patrón Intercom crece cuando escale el bot como canal de entrada.

---

## Anexo · Queries adicionales

**Q4 — universo OFFLINE-original y su apertura por latest source:**
```sql
-- vías de creación
SELECT COALESCE(c.hs_analytics_source_data_1,'(null)') || ' / ' ||
       COALESCE(c.hs_analytics_source_data_2,'(null)') AS origen_creacion,
       COUNT(*) AS contactos,
       COUNT(*) FILTER (WHERE UPPER(COALESCE(c.hs_latest_source,''))!='OFFLINE') AS luego_form_real,
       COUNT(*) FILTER (WHERE c.hubspot_owner_id IS NULL) AS sin_owner
FROM dwh.stg_hubspot__contacts c
WHERE c.createdate >= '2026-07-01'
  AND c.country IN ('México','Chile','Argentina','Colombia')
  AND UPPER(COALESCE(c.hs_analytics_source,'')) = 'OFFLINE'
  AND UPPER(TRIM(COALESCE(c.numemployees,''))) IN
      ('3-5','5-16','6-15','16','16+','+16','5-25','25-50','50-100','1000+','6+')
GROUP BY 1 ORDER BY 2 DESC;

-- apertura por latest source (cambiar SELECT por hs_latest_source_data_1/2)
```

**Q5 — julio Chile: clasificación interno vs real de los sin-owner:**
```sql
WITH sin_owner_jul AS (
  SELECT f.contact_id
  FROM dwh.int_hubspot_leads_flat f
  WHERE f.country='Chile'
    AND UPPER(TRIM(COALESCE(f.numemployees,''))) IN
        ('3-5','5-16','6-15','16','16+','+16','5-25','25-50','50-100','1000+','6+')
    AND f.recent_conversion_date >= '2026-07-01'
    AND f.recent_conversion_date <  '2026-08-01'
    AND f.effective_owner_id IS NULL
)
SELECT CASE WHEN hc.email IS NULL THEN 'no sincronizado (lag)'
            WHEN hc.email ILIKE '%agendapro%' OR hc.email ILIKE '%test%'
              OR hc.email ILIKE '%prueba%' OR hc.email ILIKE '%yopmail%'
            THEN 'interno/test' ELSE 'externo real' END AS tipo,
       COUNT(*) AS n
FROM sin_owner_jul s
LEFT JOIN dwh.hubspot_contacts hc ON hc.id = s.contact_id
GROUP BY 1;
-- Resultado 2026-08-06: interno/test 30 · externo real 11
```

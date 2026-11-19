---
type: recurso
temas: [headcount, dwh, centros-de-costo]
proyecto: "[[_P&L y Centros de Costo]]"
autor: "[[Joaquín Herrera]]"
fecha: 2026-11-09
---

# finance.headcount — employer, clave de persona y carga desde la planilla

**Propósito**: referencia de cómo llega el headcount de AgendaPro y Reservo a `finance.headcount` en Redshift desde la planilla de People, qué hace el modelo dbt y cómo se opera. Vale desde el 9-nov-2026 (PRs #481 y #482 de `agendapro/agendapro-dbt`, rama `refactor/postgres-to-redshift`). La metodología de clasificación está en [[Guía clasificación de personas (headcount)]]; el lado Evidence, en [[Reservo en Evidence — semilla xlsx, merchants, revenue y consolidación]].

## Cadena de carga
1. **«Headcount - Raw»** (Google Sheet de [[Pablo Lucero]], id `1xLVEDUvcRa3VTM64OcOXoA9mLDvSTQ6yC4XnMi-OIuo`). Una pestaña por nómina: CL, CO, MX, US, VE y AR Data con layout A:I (Date, Currency, Tax Id, Name, Amount, Position, Area, Comissions, Amount local) y el centro de costo en una columna propia de cada pestaña (CO!K, CL!P, MX!Y, US!AK, AR!P). **«Reservo CL»** tiene otro layout: A date, B nombre, C cargo en Reservo, D área en Reservo, E área equivalente de AgendaPro («División,Área,Subárea», asignada por cargo), F centro de costo, G moneda, H Tax Id; I:K deben quedar vacías.
2. **«Consolidated data»**, mismo archivo. `A1` es un QUERY que apila las pestañas de país A:I y, al final, Reservo CL reordenada: `{'Reservo CL'!A:A, G:G, Tax Id, B:B, I:I, C:C, E:E, J:J, K:K}`; el Tax Id es `H`, o `"RSV-"&nombre` mientras H esté vacío. Columnas derivadas: J Country (override por nombre en «US mapping» E:F, si no por moneda en I:J), K Gender (VLOOKUP por nombre en «Gender Mapping»), L Status (Inactive si fecha + Tax Id están en «Inactive Headcount»), M Centro de Costo (ARRAYFORMULA, VLOOKUP por fecha & Tax Id contra cada pestaña, Reservo incluida), N Employer (Reservo si fecha + nombre están en Reservo CL, si no AgendaPro). J, K y L son fórmulas fila a fila: hay que copiarlas hacia abajo cuando crece el consolidado.
3. **«2026 DWH Loading»** (id `1GMt5lB1oZ3Uk2y9YuBDAxk2zF54TrU2WrrcuvZxsy2o`), pestaña **Headcount**, columnas A:I por IMPORTRANGE: date, name, position, «division, area, subarea», country, gender, status, Cost Center, employer. Ni el RUT ni la moneda se exportan.
4. **Airbyte** → `finance.raw__headcount` (date, name, gender, status, country, employer, position, cost_center, division_area_subarea + `_airbyte_*`). El stream reescribe la tabla completa **cada hora**; no hace falta full refresh para que entre un cambio de la planilla.
5. **dbt** `models/intermediate/finance/headcount.sql` (tag `redshift_marts`) → `finance.headcount`. Lo construye la cadena de marts; a mano: `dbt run -s headcount --target redshift`. Si cambia el CSV del mapping, antes `dbt seed -s headcount_cost_center_mapping --target redshift` (la cadena no recarga seeds sola; el 9-nov el seed en Redshift quedó con 62 códigos).

## Qué hace el modelo
- **Separadores**: toda coma de `division_area_subarea` se lleva a `, ` (acepta «A, B, C», «A,B,C» y «A , B , C»); el guion « - » se traduce salvo los casos legítimos de «Operations - Payments - Onboarding». Un patrón nuevo deja `area` vacía y rompe el test.
- **employer**: `coalesce(nullif(btrim(employer), ''), 'AgendaPro')`. El modelo mira el catálogo al compilar (`adapter.get_columns_in_relation`): si la tabla raw no tuviera la columna, emite NULL y todo cae a AgendaPro. Valores: `AgendaPro` | `Reservo`.
- **Clasificación**: desde jul-2026 por `cost_center` expandido con el seed `headcount_cost_center_mapping`; ene–jun 2026 por el seed congelado `headcount_classification`. `classification_source` = sheet / seed / codigo_sin_mapear.
- **Deduplicación**: 1 fila por (date, employer, persona). Clave de persona = tokens del nombre sin comas ni acentos, en minúsculas y ordenados alfabéticamente (cada pestaña escribe el nombre en distinto orden: Chile «Nombres Apellidos», México «Apellidos, Nombres», Colombia y Argentina «Apellidos Nombres»). Prioridad: clasificación resuelta > mismo país que el mes anterior > Chile > Mexico > Argentina > Colombia > USA > Inactive antes que Active > desempate determinístico. La clave no se publica; el test singular `assert_headcount_one_row_per_person_month` la recalcula. `employer` va en el PARTITION BY pero no en el ORDER BY del `row_id` (Redshift rechaza constantes ahí).
- **business** = vertical si la persona participa de uno, si no la división traducida a la organización actual (Product & Tech / Revenue / People & Finance…). `division` conserva el valor crudo.
- **Tests en `error`**: `business` accepted_values, `area` not_null desde 2026-07, `employer` not_null y accepted_values. El resto sigue en `warn`.

## Consumidores que deben filtrar por empresa
Las extracciones de `agendapro-dashboards-evidence` que leen `finance.headcount` (board book ARR/FTE, OKR KR6, budget follow-up, people/headcount, headcount-cost, onboarding capacity, r1 by country) llevan `employer = 'AgendaPro'` en la rama `feat/board-book-reservo-b2b3-stock`; hasta que se mergee, Chile sube de 127 a 179 activos. Los agentes de revisión de nómina de `mcp-odoo-agendapro` ([[Nicolás Astudillo]]) también la leen; el filtro lo pone él.

## Consultas útiles
```sql
-- estado de la tabla raw y de la última extracción
select employer, count(*) as filas, max(_airbyte_extracted_at) as ultima_extraccion
from finance.raw__headcount group by 1;

-- un mes, por empresa, con huecos de clasificación
select employer, status, count(*) as filas,
       sum(case when classification_source is null then 1 else 0 end) as sin_clasif,
       sum(case when classification_source = 'codigo_sin_mapear' then 1 else 0 end) as cod_sin_mapear
from finance.headcount where date = '2026-10-01' group by 1, 2;

-- códigos del sheet que no están en el mapping (deberían ser 0)
select date, cost_center, count(*)
from finance.raw__headcount
where nullif(btrim(cost_center), '') is not null
  and cost_center not in (select cost_center from finance.headcount_cost_center_mapping)
group by 1, 2;
```

## 2026-11-17 · Carga de noviembre: lo que se aprendió
- **Correr dbt solo desde un worktree en la punta de `origin/refactor/postgres-to-redshift`.** La copia principal `agendapro-dbt-redshift` quedó 72 commits atrás; un `dbt run -s headcount` desde ahí regeneró `finance.headcount` con el modelo viejo (sin `employer`, división sin separar, status NULL). Se corrigió creando `agendapro-worktrees/dbt-headcount-rebuild` en la punta de origin y volviendo a correr. Antes de correr: `git fetch` y `git log -1 origin/refactor/postgres-to-redshift` contra el worktree.
- **Seed con 66 códigos.** El [PR #507](https://github.com/agendapro/agendapro-dbt/pull/507) (`feat/headcount-ceco-reservo-mdr-ae`) agrega REV-MDR-HQ, REV-AE-CL, REV-AE-AR y REV-AE-MX (Sales; BU HQ o país). Cargado en Redshift desde la rama el 17-nov con `dbt seed` + `dbt run`, 17 tests en PASS. Hasta entonces los 10 MDR y AE de Reservo venían con `classification_source = 'codigo_sin_mapear'`. 31 códigos del seed son legacy (24 solo de julio, 7 hasta octubre) y se sostienen porque los meses cargados los usan; la propuesta de migrar los 23 renombres está en Notion («Cambios CECOS hasta octubre 2026», sección 6).
- **La recarga mensual de People puede borrar las fórmulas J:L de Consolidated data.** El 17-nov las filas 7623–7882 (ARS desde mayo 2026 y Reservo) llegaron sin Country, Gender ni Status: Argentina quedó sin activos desde mayo en el DWH. Revisar después de cada carga con `select date, employer, count(*), sum(status is null) from finance.raw__headcount group by 1,2`; si hay nulos, extender las fórmulas de la fila anterior y esperar la siguiente corrida de Airbyte.
- **Bajas.** La persona lleva una fila en su último mes con Status `Inactive` (la da la pestaña Inactive Headcount: fecha + Tax Id + Employer) y desaparece desde el mes siguiente. Si no tiene fila ese mes, no queda registro de la salida (caso Romero Figueredo, nov-2026).
- **Reservo CL.** Las filas de cada mes las pega FP&A (copia del mes anterior con la fecha nueva y los cambios acordados); People carga Buk Reservo desde esa lista. Ruelas (Reservo, pagado por AgendaPro México) va solo en Reservo CL con MXN: si People lo pone además en MX Data, `employer` lo cuenta como AgendaPro y queda duplicado.
- Registro de cambios por persona y su estado: Notion, [Cambios CECOS noviembre 2026](https://app.notion.com/p/415ab34c233b8099855bcc149329b18f). Seguimiento: [[Cierre de mes noviembre]].
- 2026-11-19 · **El Tax Id no viaja al DWH.** `finance.raw__headcount` no trae las columnas Tax Id ni Currency de Consolidated data (solo date, name, gender, status, country, employer, position, cost_center, division_area_subarea); la persona se identifica por nombre normalizado y `employer`. En la planilla el Tax Id solo lo usa la fórmula de Status (COUNTIFS contra Inactive Headcount por Tax Id y fecha), así que basta con que el mismo valor esté en las dos pestañas; un número mal formado (caso Federico Alonso, nov-2026) no afecta nada.
- 2026-11-19 · **Personal de Reservo pagado por una sociedad de AgendaPro** (Ruelas, AgendaPro México) se queda en la pestaña Reservo CL con su moneda real (MXN) mientras su CECO sea de ese país; el `employer` sale de la pestaña, no de quién paga. La contabilización en Odoo se resuelve aparte (ver [[Cierre de mes noviembre]], subtarea 2).

## Historia y pendientes
- 2026-11-09 · Octubre venía roto por el separador sin espacio y Cofré duplicado; detalle en [[20261109-01 Auditoría del informe de cierre sobre finance.headcount (octubre 2026)]]. Reservo (52 personas) entra el mismo día; seguimiento en [[Definir cómo clasificar a las personas de Reservo en la planilla de headcount]].
- Pendientes: PR de Evidence con el filtro; RUT real de Reservo (hoy provisorio `RSV-`); regenerar los espejos sin PII (`perfiles/generar_espejos.py --escribir`, hoy dejan `employer` fuera); exportar `person_id` (RUT hasheado) y `payroll_entity` (moneda) en un próximo cambio de stream.
- 2026-11-17 · Carga de noviembre revisada; seed con los códigos comerciales de Reservo (PR #507, pendiente de merge); migración de los códigos legacy de julio propuesta en Notion. Pendientes nuevos en [[Cierre de mes noviembre]].

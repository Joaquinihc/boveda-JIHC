---
type: tarea
estado: hecho
prioridad:
tiempo:
tipo: gestion
origen: reunion
proyecto:
temas:
  - headcount
  - centros-de-costo
fuente: "[[2026-10-27 Planilla de headcount con People (Gabriela)]]"
fecha_limite:
creado: 2026-10-27
revisar: false
id: T-112
---

# Definir cómo clasificar a las personas de Reservo en la planilla de headcount

Las personas de Reservo están en un libro de remuneraciones aparte, sin la estructura de áreas de AgendaPro, y este mes hay que clasificarlas en la planilla de Headcount (división, área, centro de costo). Gabriela Rios (People) consulta a Javi cómo absorberlas. Acción de [[2026-10-27 Planilla de headcount con People (Gabriela)]].

## Subtareas
- [ ] Responder en el hilo de Slack sobre la clasificación de las personas de Reservo, mencionando a Javi — listo cuando la respuesta esté publicada
- [ ] Abrir y mergear el PR de Evidence desde `feat/board-book-reservo-b2b3-stock` (filtro `employer = 'AgendaPro'` en las extracciones de headcount y gráficos de grupo) — listo cuando el board book vuelva a mostrar 127 activos en Chile
- [ ] Conseguir los RUT de las 52 personas de Reservo (export BUK de Reservo, vía Gabriela o Javiera) y cargarlos en Reservo CL!H — listo cuando ningún Tax Id de Consolidated data empiece con `RSV-`

## Notas

### 2026-11-09 · `employer` en finance.headcount
- La columna quedó como `employer` (`AgendaPro` | `Reservo`; dbt PR #482 en `refactor/postgres-to-redshift`). Octubre 2026: 201 activos AgendaPro y 52 de Reservo, todos con función del P&L, CECO y negocio.
- Evidence (rama `feat/board-book-reservo-b2b3-stock`, sin PR al 10-nov): las 8 extracciones que leen `finance.headcount` filtran `employer = 'AgendaPro'`, con test sobre todo `sources/`; comparadas en Redshift contra `main`, solo salen las 52 personas de Reservo (headcount AgendaPro de oct: 253 → 201). La extracción nueva `headcount_by_employer` alimenta los gráficos AgendaPro + Reservo de las láminas 5.1g y 5.1g2 (Reservo desde oct-26).

### 2026-11-09 · Carga de Reservo a finance.headcount (planilla, Airbyte y dbt)
- Planilla «Headcount - Raw» (de [[Pablo Lucero]]): pestaña «Reservo CL» con el área de AgendaPro asignada **por cargo** (no por área de Reservo) y el centro de costo; Gender Mapping con las 52 personas (género inferido del nombre, columnas Employer y Fuente para validarlo con BUK); Consolidated data con columna N `Employer`, el QUERY de A1 apilando Reservo CL directo (sin pestaña puente, decisión de Joaquín), el VLOOKUP de centro de costo (M2) con el par de Reservo y las fórmulas J:L copiadas al bloque nuevo. «2026 DWH Loading» › Headcount, columna I `employer`.
- Tax Id: Reservo no tiene RUT cargado. Mientras la columna H esté vacía, A1 y M2 usan un provisorio `RSV-` + nombre, que desaparece solo cuando People cargue el RUT. Sin él, Status quedaba en blanco y el VLOOKUP daba a todos el centro de costo de la primera persona.
- DWH: Israel extendió el stream de Airbyte a A:I el 9-nov (la tabla raw se reescribe cada hora, sin full refresh). PRs de dbt contra `refactor/postgres-to-redshift`: #481 (separador de octubre y clave de persona) y #482 (`employer`, dedup por empresa, tests en error, seed con 9 códigos), ambos mergeados el 9-nov. Corrido a mano el 9-nov: `dbt seed -s headcount_cost_center_mapping` (62 códigos) y `dbt run -s headcount`, 13 tests en PASS; las filas de AgendaPro no cambiaron (203 en octubre).
- Códigos de Reservo: todos existían salvo REV-SUPP-AR (Daniela Droguett) y REV-SUPP-MX (Ignacio Monsalve), soporte dedicado a un mercado que también hace onboarding, confirmado con [[Pablo Saintard]] el 2-nov. Quedan en Customer Support con BU Argentina y Mexico.
- Cómo funciona hoy la cadena completa: [[finance.headcount — employer, clave de persona y carga desde la planilla]]. La auditoría del informe de cierre que motivó el #481: [[20261109-01 Auditoría del informe de cierre sobre finance.headcount (octubre 2026)]].

### Pendientes menores
- 2026-11-09 · Regenerar los espejos sin PII (`perfiles/generar_espejos.py --escribir`) ahora que `raw__headcount` produce `employer`; hoy lo dejan fuera.
- 2026-11-09 · Avisar a [[Nicolás Astudillo]] que la columna se llama `employer`, no `company` (sus agentes de revisión de nómina deben filtrar por ella).
- 2026-11-09 · Pedir a Israel, en un próximo cambio de stream, el RUT hasheado como `person_id` y la moneda de la nómina como `payroll_entity` (el país del headcount es residencia, no sociedad pagadora).
- 2026-11-09 · Cuando haya bajas de Reservo, cargarlas en Inactive Headcount con la misma llave (RUT o `RSV-`) y una columna Employer, igual que en Gender Mapping.
- 2026-11-09 · Las fórmulas J:L de Consolidated data son fila a fila: copiarlas hacia abajo cada mes que crezca el consolidado. Las columnas I:K de Reservo CL deben quedarse vacías (hacen de Amount, Comissions y Amount local en el apilado).

## Historial
- 2026-10-30 · Creada desde [[2026-10-27 Planilla de headcount con People (Gabriela)]] (propuesta)
- 2026-11-10 · Avance desde Claude Code: `employer` en vivo; Evidence filtra AgendaPro y suma gráficos de headcount AgendaPro + Reservo (rama sin PR).
- 2026-11-10 · Avance desde Claude Code: planilla y carga documentadas (Reservo CL, Gender Mapping, QUERY A1, Tax Id provisorio, PRs #481/#482, dbt run del 9-nov); 2 subtareas nuevas (PR de Evidence y RUT).
- 2026-11-10 · estado: propuesta → hecho. Reservo ya está en `finance.headcount` con `employer` (dbt PR #482, 9-nov) y el board book filtra por esa columna (Evidence PR #892, mergeado el 9-nov). El RUT de las 52 personas no es necesario para el cierre: la llave provisoria `RSV-` funciona. (Joaquín en conversación)

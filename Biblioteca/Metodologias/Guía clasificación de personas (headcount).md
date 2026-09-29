---
type: metodologia
temas: [headcount, pnl, centros-de-costo]
proyecto: "[[_P&L y Centros de Costo]]"
fuente: trabajo propio
fecha: 2026
---

# Guía de Clasificación de Personas — BU × P&L Category × Business

> **Estado: BORRADOR para validación del equipo Finance. No crear cuentas contables ni tocar Odoo hasta aprobar esta metodología.**
>
> Fuente de datos: `finance.headcount` (DWH; sheet "headcount raw" cargada vía Airbyte) — columnas `name`, `country`, `position` (cargo), `division / area / subarea` (equipo).
> Complementa a [`cost-center-classification-guide.md`](cost-center-classification-guide.md) (que cubre solo la dimensión BU, para todo gasto). Esta guía cubre **personas (payroll)** en las 3 dimensiones.

---

## 1. El problema que motiva esta guía

Cada persona debe quedar clasificada en **tres dimensiones a la vez**:

| Dimensión | Valores válidos | Pregunta que responde |
|---|---|---|
| **BU (cost center)** | Chile, Mexico, Argentina, Colombia, Other, HQ | ¿**Para quién** trabaja? (alcance geográfico) |
| **P&L Category** | ver §3 | ¿**Qué función** cumple? (naturaleza del costo) |
| **Business** | Health, Beauty, Medspa, Payments, Marketplace, `null` | ¿**En qué vertical/squad** participa? |

El error histórico fue **mezclar dimensiones**: para reflejar que alguien participa en un vertical (Business = Health) se le cambió la categoría P&L (de Development a Product), moviendo gente de tecnología a producto en el estado de resultados. La data actual del P&L muestra la consecuencia:

- `G1110 Payroll Payments`, `G1114 Payroll Payments Tech`, `G1118 Payroll Payments Sales` → hoy mapeadas a **Customer Success Expansion** (gente de tech y sales del vertical Payments cae en una categoría de CS).
- `G2109` se usa a la vez para *Health non-doctors Tech* (**Development**) y *Health non-doctors Sales* (**Sales**); ídem `G2110` con Beauty. Un mismo G-code con dos categorías rompe cualquier automatización.

## 2. Principio rector: tres dimensiones ortogonales

**Cada dimensión tiene UNA fuente y UN carril. Cambiar una dimensión JAMÁS cambia otra.**

| Dimensión | Se deriva de | NO depende de |
|---|---|---|
| P&L Category | el **cargo** (position) — qué hace la persona | el equipo/vertical donde está sentada |
| Business | la **división** (equipo) — dónde participa | el cargo |
| BU | el **alcance del rol** — a qué mercado sirve | el país de la nómina que le paga |

Tres consecuencias prácticas:

1. Un developer en *Beauty Tech* es **Development / Beauty / HQ**. Nunca "Product" por estar en un vertical de producto.
2. Una ISR de *Health non-doctors Sales* en Chile es **Sales / Health / Chile**. El vertical no la saca de Sales.
3. El país de la entidad pagadora (columna `country`) **no es la BU**: los contratados vía Deel/USA que son devs remotos van a HQ; una vendedora pagada por Argentina que vende para "Other markets" va a Other.

## 3. Dimensión P&L Category — se deriva del CARGO

Categorías que pueden llevar personas (payroll):

| p_l | Categoría | Cargos que la componen (patrones) |
|---|---|---|
| 2- Service Costs | **Platform** | (normalmente sin payroll — infra: AWS, Twilio. Si el equipo decide cargar SRE/DevOps aquí, definirlo explícito; default: DevOps → Development) |
| 2- Service Costs | **Payment Processing** | (sin payroll — costo transaccional de procesadores. La nómina de Payments Ops va en CS Expansion, decisión 2026-07-06) |
| 2- Service Costs | **Customer Support** | Customer Support (todo nivel), Head of Customer Experience* |
| 2- Service Costs | **Customer Success Retention** | Customer Success Manager, CS Adoption |
| 2- Service Costs | **Customer Success Expansion** | **Todo rol que vende al stock de clientes** (decisión 2026-07-06): Account Manager (todos, incl. AM AI y AM POS), KAM (todos), Field Sales, Payments Ops (Ops Analyst, Executive Ops, Payments Consultant), Head of Customer Success* |
| 3- Acquisition | **Marketing** | **Toda la división Marketing, cualquiera sea el cargo** (decisión 2026-07-06 — incl. Graphic Designer, UGC, SEO, Copywriter). Además: CMO, VP Marketing, Brand, Performance, Comms |
| 3- Acquisition | **Sales** | Venta nueva: CRO, Head of Sales, ISR, MDR, Account Executive, Team Leader de venta, Sales Ops |
| 3- Acquisition | **Customer Success Onboarding** | Customer Onboarding (todo nivel), Onboarding Engineer, CX Enablement |
| 4- Corporate | **Development** | CTO, VP of Engineering, Tech Lead, Software/Backend/Frontend/Mobile Developer, Software Engineer, **Product Engineer**, Data Engineer, DevOps, BizOps, QA, Domain Expert AI |
| 4- Corporate | **Product** | VP of Product, Product Manager, Product Designer, Head of Design, Designer, Product Ops |
| 4- Corporate | **Administration** | CEO (decisión 2026-07-06), CFO, CPO, Controller, Accounting, FP&A, Payroll, People/HR, Office, Legal interno, Administrative |
| 4- Corporate | **Other Corporate** | (sin payroll — gastos corporativos no funcionales) |

\* Los "Head of" que lideran varias funciones (ej. Head of Customer Experience cubre Support + Onboarding) van a la categoría de la función **dominante de su rol**, decidida una vez y anotada en el registro de excepciones (§8).

**Reglas de desempate del cargo:**

- **La línea Sales vs Expansion es el mercado, no el título**: venta a clientes NUEVOS → Sales (ISR, MDR, AE); venta/expansión al STOCK de clientes existentes → CS Expansion (Account Manager, KAM, Field Sales, Payments Ops). Decisión 2026-07-06.
- Título contiene *Developer / Engineer / Tech* → **Development**, aunque el equipo sea de un vertical y aunque el título diga "Product Engineer". Excepción: **Onboarding Engineer** → CS Onboarding (su función es implementar clientes, no construir producto).
- Título contiene *Product Manager / Designer* → **Product** (un PM en Payments Tech sigue siendo Product). **Excepción: si su división es Marketing → Marketing** (un Graphic Designer de marketing hace marketing).
- C-suite: CEO → Administration (decisión 2026-07-06: se mantiene en G3000, sin cuenta Other Corporate) · CFO/CPO → Administration · CTO → Development · CRO → Sales · CMO → Marketing · VP de un área → la categoría de su área (VP of Payments → Development: lidera ingeniería del vertical; **VP of Fintech → Sales**: lidera la función comercial de fintech — decisión CFO 2026-07-06).
- Las categorías **Recurring**, **Payment Processing** y **Payments processing fee** existen en el P&L pero **nunca llevan personas** — son revenue/costo transaccional.

## 4. Dimensión Business — se deriva de la DIVISIÓN

| División (headcount) | Business |
|---|---|
| Beauty | **Beauty** |
| Health non-doctors | **Health** |
| MedSpa | **Medspa** |
| Payments | **Payments** |
| Marketplace | **Marketplace** |
| Product & Tech (Core Experience, Platform, DevOps, BizOps, Technology) | `null` (transversal) |
| Marketing | `null` |
| Revenue (ISR, Onboarding, Support, CS, Expansion) | `null` |
| People & Finance | `null` |
| Executive Office | `null` |

- `null` **no es un error**: significa "función transversal, no participa de un squad vertical". No forzar un vertical a quien no lo tiene.
- Si una persona de un equipo transversal se dedica ≥80% a un vertical (ej. un AM POS que solo trabaja Payments), se puede sobre-escribir el Business **vía registro de excepciones (§8)** — nunca cambiando su categoría P&L.

## 5. Dimensión BU — se deriva del ALCANCE del rol

Reglas heredadas de [`cost-center-classification-guide.md`](cost-center-classification-guide.md) §3.4 (sin cambios):

| Alcance | BU | Roles típicos |
|---|---|---|
| Global / toda la compañía | **HQ** | Todo Development, Product, C-suite, People & Finance, BizOps, DevOps |
| Regional (sirve a varios países) | **HQ** | Customer Support (LATAM), Marketing (política actual), Head of Sales Latam, CX Enablement |
| Un país específico | **ese país** | ISR/MDR/AE/Field Sales (donde venden), Onboarding (país que onboardean), contabilidad local |
| Mercados menores (UY, PA, CR, PE, EC…) | **Other** | Vendedores/onboarders dedicados a esos mercados |

**Gotchas:**
- `country` en el headcount = **entidad que paga la nómina**, no la BU. USA = contractors Deel remotos (devs → HQ). Vendedoras "Other markets" pagadas por AR/USA (ej. Montero, Silvera) → **Other**.
- Expat / dual-entity (ej. CRO pagado por Chile y México) = split billing conocido, no doble conteo; la BU del rol sigue siendo HQ.
- Los verticales NO alteran la BU: Payments Ops de Chile → Chile (costo directo del negocio de pagos chileno); Beauty Tech → HQ (construyen producto global).
- **La categoría NO altera la BU**: Field Sales, AM POS y Payments Ops son CS Expansion en el P&L, pero su BU sigue siendo el país cuya cartera atienden — la consolidación de CS en HQ aplica a los roles regionales, no a los country-specific.

## 6. Orden de decisión (algoritmo)

Para cada persona, en este orden — las tres respuestas son independientes:

```
1. Business  ← división           (tabla §4; central → null)
2. Categoría ← cargo              (tabla §3 + desempates)
3. BU        ← alcance del rol    (tabla §5; ignorar país de nómina)
4. ¿El cargo contradice al equipo? (ej. "Auxiliar de Software" en ISR,
   "VP of Payments" en Beauty Tech) → NO adivinar: flag REVISAR,
   resolver con People, anotar en §8.
```

**Ejemplos completos:**

| Persona (caso real jun-2026) | Cargo | Equipo | → Categoría | Business | BU |
|---|---|---|---|---|---|
| Dev junior | Software Developer Junior | Beauty Tech | Development | Beauty | HQ |
| PM semi senior | Product Manager Semi Senior | MedSpa Tech | Product | Medspa | HQ |
| ISR México | ISR | Revenue-ISR | Sales | null | Mexico |
| Onboarding AR | Onboarding | Revenue-CX-Onboarding | CS Onboarding | null | Argentina |
| Ops de pagos | Executive Ops | Payments Ops | Payment Processing | Payments | Chile |
| Soporte AR | Customer Support | Revenue-CX-Support | Customer Support | null | HQ (regional) |
| Contador local CO | (externo, no headcount) | — | Administration | null | Colombia |
| Dev remoto Deel | Backend Senior Developer | Payments Tech | Development | Payments | HQ |

## 7. Implicancia contable — Opción A elegida (decisión 2026-07-06)

La cuenta contable es el **carril de la categoría**; el vertical y la BU viajan por otros carriles.

**Decisión**: se va por **cuentas por combinación** (`Payroll <Categoría> - <Business>`). La Opción B (segundo plan analítico) se descartó porque la analítica ya identifica el cost center (país que atienden) y no se quiere sobrecargar ese carril.

Reglas de diseño de las cuentas:

1. **Cuenta (G-code) = P&L Category** — relación 1:1 estricta. Un G-code nunca puede aparecer en dos categorías. (Los duplicados "G2109/G2110 ... Sales" que mostraba el Master eran un error de tipeo del Master: en Odoo los Sales verticales siempre fueron G1115/G1116/G1117.)
2. Cada cuenta nueva recibe un G-code propio, idealmente **dentro del rango de su categoría** (G21xx = Development, G20xx = Product, G12xx = Onboarding, etc.). Excepciones históricas ya contabilizadas (G1114 en Development, G1110/G1115-G1118 en Expansion) se aceptan y se corrigen **solo en el mapping del Master**, no renumerando Odoo.
3. Las cuentas verticales ya creadas en junio (CL/MX/Inc) se reusan tal cual; lo que cambia es su categoría en el Master cuando la gente real no coincide con el nombre (G1114 → Development; G1111/G1115/G1116/G1117 → CS Expansion).
4. **BU**: sigue en el plan analítico de cost center actual (Chile/Mexico/Argentina/Colombia/Other/HQ). Sin cambios.

Prueba de aceptación: **puedo cambiar el Business de una persona sin tocar su categoría ni su BU, y viceversa.**

### Mapa de cuentas payroll — (Categoría × Business) → cuenta contable

Matriz alineada a las cuentas **ya existentes en Odoo** (verificado 2026-07-06: Chile, México y AgendaPro Inc crearon en junio el set vertical de Product/Tech/Sales; CO y AR solo tienen el set base). Personas según headcount jun-2026.

| Categoría | Business | Cuenta contable | Estado |
|---|---|---|---|
| Development | null | G2100 - Payroll Tech | existente |
| Development | Marketplace | G2107 - Payroll Marketplace Tech | existente |
| Development | Health | G2109 - Payroll Health non-doctors Tech | existente |
| Development | Beauty | G2110 - Payroll Beauty Tech | existente |
| Development | Medspa | G2111 - Payroll Medspa Tech | existente |
| Development | Payments | G1114 - Payroll Payments Tech | existente — **re-mapear en el Master de CS Expansion → Development** |
| Product | null | G2000 - Payroll Product | existente |
| Product | Marketplace | G2006 - Payroll Marketplace Product | existente |
| Product | Health | G2007 - Payroll Health non-doctors Product | existente (ojo: en Odoo G2007=Health y G2008=Beauty) |
| Product | Beauty | G2008 - Payroll Beauty Product | existente |
| Product | Medspa | G2009 - Payroll Medspa Product | existente |
| Product | Payments | G2010 - Payroll Payments Product | existente (creada 2026-07-06 en CL, rango 5.4.00.1x) |
| Sales (venta nueva) | — | por cargo: ISR → G1108 · MDR → G1107 · AE/Head/CRO/Sales Ops → G1100 | existentes |
| Marketing | — | G1000 - Payroll Marketing | existente |
| CS Onboarding | null | G1200 - Payroll Onboarding | existente |
| CS Onboarding | Beauty | G1202 - Payroll Beauty Onboarding | existente (creada 2026-07-06 en CL, rango 5.4.00.1x) |
| CS Onboarding | Health | G1203 - Payroll Health non-doctors Onboarding | existente (creada 2026-07-06 en CL, rango 5.4.00.1x) |
| CS Onboarding | Medspa | G1204 - Payroll Medspa Onboarding | existente (creada 2026-07-06 en CL, rango 5.4.00.1x) |
| CS Onboarding | Payments | G1206 - Payroll Payments Onboarding | existente (creada 2026-07-06 en CL, rango 5.4.00.1x) |
| CS Retention | null | G0100 - Payroll Retention | existente |
| CS Expansion | null | G0105 - Payroll Expansion (AM, KAM y Field Sales sin vertical) | existente |
| Sales | (histórica) | G1111 - Payroll Field Sales — **mantiene su mapping en Sales**; desde abril 2026 los Field Sales operan como KAM (CS Expansion) y se asientan en las cuentas de Expansion, así que esta cuenta queda sin gente | existente |
| CS Expansion | Payments | G1110 - Payroll Payments (Ops) · G1118 - Payroll Payments Sales (KAM POS / Field Sales) | existentes (mapping a Expansion CORRECTO) |
| CS Expansion | Health | G1115 - Remuneraciones Health non-doctors Sales | existente — **re-mapear a CS Expansion** (su gente es AM al stock) y renombrar a "Expansion" |
| CS Expansion | Beauty | G1116 - Remuneraciones Beauty Sales | existente — ídem |
| CS Expansion | Medspa | G1117 - Remuneraciones Medspa Sales | existente — ídem (hoy sin gente) |
| Customer Support | null | G0200 - Payroll Support | existente |
| Customer Support | Health | G0202 - Payroll Health non-doctors Support | existente (creada 2026-07-06 en CL) |
| Administration | — | G3000 - Payroll Admin (incluye CEO) | existente |

**Balance real del trabajo contable** (mucho menor de lo estimado antes de revisar Odoo):

- **Cuentas creadas el 2026-07-06** en AgendaPro Chile SpA (`5.4.00.10`-`5.4.00.15`): G2010, G1202, G1203, G1204, G1206, G0202. Replicar en MX/Inc solo si algún día tienen gente de ese combo.
- **Re-mapear en el Master (sin tocar Odoo)**: G1114 → Development · G1115, G1116, G1117 → CS Expansion. **G1111 NO se re-mapea** (mantiene Sales): lo que cambia es la asignación de las personas — los Field Sales desde abril 2026 operan como KAM y van a las cuentas de Expansion. Los códigos duplicados que se veían en el Master ("G2109/G2110 ... Sales") eran un **error de tipeo del Master**: en Odoo los Sales verticales siempre fueron G1115/G1116/G1117.
- Combos sin cuenta propia (ej. Retention × vertical) usan la genérica de su categoría hasta que exista masa crítica.

## 8. Mantención y excepciones

- **Fuente de verdad**: el sheet "headcount raw" (People). Finance clasifica; People corrige equipo/cargo cuando el flag REVISAR muestre inconsistencia.
- **Registro de excepciones**: toda persona cuya clasificación no salga mecánicamente de las tablas (Heads multi-función, Account Managers ambiguos, overrides de Business) se anota con: nombre, regla que no aplicó, decisión, quién la tomó, fecha. Sin registro, la excepción no existe.

### Registro de excepciones vigente

| Persona / grupo | Decisión | Quién / fecha |
|---|---|---|
| Pablo Marambio (VP of Payments) | **Development** (lidera ingeniería de Payments) · Business Payments · HQ. Pendiente corregir su equipo en el sheet (hoy figura en Beauty Tech) | CFO, 2026-07-06 |
| Rodrigo Bitrán (Head of Growth, Marketplace) | **Product** · Marketplace · HQ | CFO, 2026-07-06 |
| Miguel Cárdenas (VP of Fintech) | **Sales** (no ISR/MDR → G1100) · BU HQ (VP, alcance global) | CFO, 2026-07-06 |
| Nicolás Cofré (Head of BizOps) | **Product** · Business null · BU HQ (el cargo BizOps mapea a Development por regla, pero su función real es Product) | CFO, 2026-07-06 |
| Stephanie Padilla Marun ("Auxiliar de Software", ISR MX) | **Sales** — es ISR; el cargo del sheet no refleja su función. Pedir a People corregir el cargo | CFO, 2026-07-06 |
| División Marketing completa (incl. Carmona García, Graphic Designer) | **Marketing** cualquiera sea el cargo | CFO, 2026-07-06 |
| Payments Ops (equipo completo) | **CS Expansion** (venden/expanden al stock) — Payment Processing queda sin payroll | CFO, 2026-07-06 |
| Account Managers, KAMs y Field Sales (todos) | **CS Expansion** (venden al stock de clientes; Sales = solo venta nueva) | CFO, 2026-07-06 |
| Field Sales — vigencia y cuenta | Aplican como KAM/Expansion **desde abril 2026** aunque el cargo en el sheet no se haya actualizado (caso Álvaro González Llanquinao, Payments Sales). La cuenta `G1111 - Payroll Field Sales` **mantiene su mapping en Sales** — cambia la asignación de las personas (a cuentas de Expansion), no la cuenta | CFO, 2026-07-06 |
| CEO | **Administration** (se queda en G3000; no se crea cuenta de payroll en Other Corporate) | CFO, 2026-07-06 |
- **Nuevo empleado**: aplicar §6. Si el cargo es nuevo y no matchea ningún patrón de §3 → REVISAR, no adivinar.
- **Cambio de equipo**: recalcular solo Business (y BU si cambió el alcance). La categoría solo cambia si cambió la **función**.
- Revisión mensual: cruzar headcount vs P&L (el `agent_bu_reviewer` audita la dimensión BU; las dimensiones Categoría y Business se auditan con el export de este mismo análisis).

> **Cómo se lleva esta clasificación a la contabilidad de cada sociedad**
> (patrones por país, plan de cuentas, reglas de privacidad y el flujo validado):
> ver [`payroll-centralization-methodology.md`](payroll-centralization-methodology.md).

---

*Generado 2026-07-06 sobre el headcount activo de junio 2026 (176 personas). Los casos REVISAR detectados están en `exports/headcount_classification_2026-06.xlsx`.*

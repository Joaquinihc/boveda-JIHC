---
type: recurso
temas: [marketplace, valor-agendapro]
fuente: discovery proyecto Valor de AgendaPro
fecha: 2026
---

---

---

## De qué se trata el proyecto

**Valor de AgendaPro** es un proyecto para que el comercio sea consciente, todo el tiempo, del valor que le entrega AgendaPro. Hoy ese valor está disperso (en reportes, en su día a día, en el monto que factura) y no lo asociamos explícitamente con la suscripción que paga. Cuando llega el momento de cancelar, el comercio decide en frío sin tener presente todo lo que recibió.

El proyecto interviene tres superficies:

- **Dashboard semanal** (la home `app.agendapro.com`, hoy de bajo uso): se transforma en un panel personalizado donde se ven los clientes nuevos, los cumpleaños y especialmente **lo que el Marketplace le está generando esta semana**.
- **Mail semanal:** resumen que recibe cada lunes con el highlight de su semana, anclado en el aporte del Marketplace.
- **Pantalla de cancelación:** cuando el comercio decide darse de baja, antes de mostrarle el modal de razones le mostramos **la historia que construimos juntos** (reservas, clientes nuevos, ventas) y un cálculo directo de cuánto le paga el Marketplace versus lo que cuesta la suscripción. El objetivo es interceptar emocionalmente la cancelación con datos reales suyos.

El proyecto se adapta por país, vertical y estado del Marketplace del comercio (activo / inactivo / fuerte / débil). El copy y los números cambian para que cada comercio vea su propia realidad, no un mensaje genérico.

## Por qué los necesito

Cada uno de estos lugares muestra números que vienen del DWH. Antes de pasarlo a desarrollo, necesito que Finanzas defina **cuál es la fuente oficial** de cada métrica financiera. El equipo de ingeniería no debe inventar definiciones: necesitamos una sola fuente de verdad por métrica para que el día de mañana, cuando alguien pregunte "¿de dónde sale ese número?", todos respondan lo mismo.

---

## 1. Métricas financieras que necesitan validación

### 1.1 Marketplace — corazón del proyecto


| #     | Métrica mostrada                                                    | Dónde aparece                                                                            | Definición propuesta                                                                                                        | Fuente DWH propuesta                                                                                         | Cadencia                                         | Validar con Finanzas                                                                                                                                                                                                 |
| ----- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.1.1 | **GMV Marketplace** (ej: "$195K al mes" / "$49K esta semana")       | Dashboard card "Tu semana en Marketplace" · Mail · Cancel flow (Story + Marketplace ROI) | Suma del `price` de bookings cuyo `demand_channel ∈ ('marketplace', 'marketplace_app', 'marketplace_google')` en el período | `dwh.int_marketplace__bookings` → `SUM(price)` filtrando por `company_id` y rango de fechas                  | Semanal (dashboard/mail) · Mensual (cancel flow) | ¿`int_marketplace__bookings` es la fuente oficial o usan `dwh_marketplace.mart_marketplace__chargeable_bookings`? · ¿Filtramos por `status_id` (excluir cancelados/no-shows)? · `marketplace_google` cuenta como MP? |
| 1.1.2 | **Clientes nuevos vía Marketplace** ("+7 clientes nuevos este mes") | Dashboard card MP · Mail · Cancel flow                                                   | Clientes cuya primera reserva con esa company es vía MP en el período                                                       | `dwh_marketplace.int_marketplace__client_first_bookings` (presunto, hay que verificar columnas)              | Semanal / Mensual                                | ¿Existe esa tabla con esa lógica? Si no, cómo calculamos "primera reserva del cliente con esta company"?                                                                                                             |
| 1.1.3 | **Reservas Marketplace** ("13 reservas/mes")                        | Dashboard card MP · Mail · Cancel flow                                                   | `COUNT(*)` de bookings del MP en el período                                                                                 | `dwh.int_marketplace__bookings` → `COUNT(*)`                                                                 | Semanal / Mensual                                | ¿Contamos todos los `status_id` o solo `attended` (3)?                                                                                                                                                               |
| 1.1.4 | **Ratio MP / Suscripción** ("3.3× cada mes")                        | Cancel flow · Banner verde "te paga la suscripción X×"                                   | `GMV_marketplace_mensual / MRR_mensual`                                                                                     | Cálculo derivado de 1.1.1 + 2.1.1                                                                            | Mensual                                          | ¿Es correcto el cálculo así o tienen otra forma de hablar del ROI del Marketplace para el comercio?                                                                                                                  |
| 1.1.5 | **Margen libre tras suscripción** ("$136K libres al mes")           | Cancel flow · "te quedan $X libres al mes"                                               | `GMV_marketplace_mensual − MRR_mensual`                                                                                     | Cálculo derivado                                                                                             | Mensual                                          | ¿Hay algún descuento que tengamos que aplicar al GMV antes de mostrar este número al comercio (impuestos, copagos, etc.)?                                                                                            |
| 1.1.6 | **Benchmark MP por país × vertical** ("promedio Beauty CL: $195K")  | Dashboard variantes Weak/Inactive · Cancel flow Weak/Inactive · Mail (potencialmente)    | Mediana o promedio de GMV mensual MP de comercios B2B3+ activos, por country × macro_niche                                  | Query agregada sobre `dwh.int_marketplace__bookings` + `dwh.companies` + `dwh.company_attribute_macro_niche` | Snapshot mensual (actualizar 1×/mes)             | ¿Usamos **mediana** (más robusta) o **promedio** (más simple)? · ¿Excluimos outliers? · ¿Cómo definimos "comercio MP activo": ≥1 booking en últimos 30/90 días?                                                      |
| 1.1.7 | **% comercios self-paying** ("70% en Beauty CL")                    | Cancel flow Inactive · "Para el X% ya paga la suscripción"                               | % de comercios con ratio ≥ 1× en el segmento país × niche                                                                   | Query agregada                                                                                               | Snapshot mensual                                 | ¿Usamos esta métrica para mostrar al cliente o solo en interno? ¿Cómo se computa "comercio similar al tuyo"?                                                                                                         |


### 1.2 Lifetime Value del comercio — Cancel flow (Story)

Los 3 stats principales del bloque emocional "Qué pena que nos dejes, Camila":


| #     | Métrica mostrada                                     | Dónde                        | Definición propuesta                                                                  | Fuente DWH propuesta                                                                                    | Validar                                                                                                                                              |
| ----- | ---------------------------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.2.1 | **Reservas gestionadas lifetime** ("4.128 reservas") | Cancel flow Story · StatCard | `COUNT(*)` de bookings de la company desde `first_paying_month` hasta hoy             | `dwh.augmented_bookings` filtrando por `company_id`                                                     | ¿Incluimos solo bookings creados desde que es cliente pagado, o desde el sign-up? ¿Filtramos por status?                                             |
| 1.2.2 | **Clientes nuevos lifetime** ("612 clientes")        | Cancel flow Story · StatCard | `COUNT(DISTINCT client_id)` que tuvieron su primera reserva en esa company            | `staging.clients` o derivada de `dwh.augmented_bookings` con `MIN(created_at) BY client_id, company_id` | ¿Hay una tabla con `first_booking_date` por (client_id, company_id)? · ¿Cómo evitamos contar clientes duplicados que llegaron por distintos canales? |
| 1.2.3 | **GMV total registrado** ("$49.7M en ventas")        | Cancel flow Story · StatCard | Suma de todas las ventas registradas en AgendaPro por la company desde que es cliente | `dwh.company_sales_months.gmv_usd` o equivalente local                                                  | Mensual: la tabla `company_sales_months` existe — ¿es la que Finanzas usa? · ¿Incluye solo ventas del calendar o también POS / pagos online?         |


### 1.3 MRR / costo de suscripción


| #     | Métrica mostrada                                                  | Dónde                                                           | Definición propuesta                          | Fuente DWH propuesta                                                | Validar                                                                                                                 |
| ----- | ----------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| 1.3.1 | **Suscripción mensual del comercio** ("$59K al mes")              | Cancel flow Marketplace ROI · "Pagas al mes $X"                 | Plan + addons (decisión de producto, ya cerrada). MRR total del comercio en moneda local. | `dwh.mrr` → `mrr` (moneda local), última `date_month` con `mrr > 0` — confirmar que ese MRR ya incluye addons o si hay que sumar de otra tabla | ¿`dwh.mrr.mrr` ya refleja plan + addons o tenemos que armarlo desde `int_subscriptions_coupons_enriched`? |
| 1.3.2 | **Suscripción total pagada lifetime** (antes "$1.6M en 27 meses") | ⚠️ Se removió del MVP — solo si Finanzas lo quiere reintroducir | Suma de MRR pagado desde `first_paying_month` | `dwh.mrr` → `SUM(mrr) WHERE company_id = X`                         | ¿Vale la pena exponer este número? Riesgo: alguien dirá "pagué $1.6M y solo me dieron $X"?                              |


### 1.4 Operativos no-financieros (no críticos, pero conviene validar definición)


| #     | Métrica                                                    | Dónde                                 | Fuente propuesta                                                                                         | Validar                                                                                    |
| ----- | ---------------------------------------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 1.4.1 | "AgendaPro te ahorró 12 horas administrativas esta semana" | Dashboard WelcomeHeader · Mail footer | **No tenemos fuente clara** — es un cálculo proxy basado en bookings × tiempo estimado de gestión manual | Definir con Finanzas/Producto: ¿qué cálculo defendemos? (ej: `bookings × 5 min ahorrados`) |
| 1.4.2 | "Recordatorios enviados" (8.240)                           | Cancel flow Story · MiniStat          | Suma de SMS + WhatsApp + email enviados                                                                  | ¿Existe una tabla agregada? Si no, hay 3 fuentes distintas a combinar                      |
| 1.4.3 | "Caída de no-shows" (38%)                                  | Cancel flow Story · MiniStat          | `(no_show_rate_primer_mes − no_show_rate_actual)`                                                        | ¿Cómo se calcula no_show_rate oficialmente? · ¿Qué pasa si el primer mes tuvo n bajo?      |


---

## 2. Métricas no-financieras que igual conviene confirmar

Estas no involucran $$ pero igual el dato viene del DWH y necesitamos fuente única:


| #   | Métrica                                            | Fuente propuesta                                             | Comentario                                                                                                      |
| --- | -------------------------------------------------- | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| 2.1 | Origen de reservas (% interna / web / marketplace) | `dwh.augmented_bookings` con campo de origen                 | ¿Existe la dimensión `origin/channel` en la tabla? Si no, ¿cómo distinguimos web del mini-sitio vs marketplace? |
| 2.2 | "Nuevos clientes esta semana" (lista)              | `dwh.augmented_bookings` con flag `is_new_client` o derivado | Validar cómo se identifica un cliente como "nuevo" (primera reserva de su vida vs primera con este comercio)    |
| 2.3 | Cumpleaños esta semana                             | `staging.clients.birthday`                                   | ¿Existe la columna? ¿Está poblada? % de clientes con birthday completado                                        |
| 2.4 | "Cliente desde marzo 2024" (tenure)                | `dwh.first_paying_date.start_date_pago`                      | OK, validar consistencia con `dwh.mrr.first_paying_month`                                                       |


---

## 3. Decisiones de producto que dependen de Finanzas

Estas son las **decisiones bloqueantes** que necesito resolver con Finanzas antes de pasar a desarrollo:

### 3.1 ¿Qué exactamente representa el `price` de los bookings de Marketplace?

**El issue:**
Hoy mostramos al comercio "Marketplace te genera $195K al mes" usando `SUM(price)` de `dwh.int_marketplace__bookings`. Marketplace **no cobra comisión** sobre los bookings, así que el comercio recibe todo. Igualmente quiero confirmar que el `price` representa el monto real que entra al comercio.

**Preguntas:**

- ¿`price` ya incluye impuestos (IVA / lo que aplique en cada país) o es pre-impuestos?
- ¿Incluye eventuales copagos, propinas o adicionales del cliente final?
- ¿Hay algún concepto que se le descuente al comercio antes de que ese dinero llegue (gateway de pago, etc.) y que deberíamos restar para mostrar el "ingreso real"?

**Recomendación:** mostrar `SUM(price)` tal cual, aclarando en disclaimer chico qué representa ("Ventas asociadas a tus reservas de Marketplace").

**Pregunta a Finanzas:** ¿están de acuerdo o hay que ajustar?

### 3.2 Benchmarks: ¿cómo definimos "comercio similar al tuyo"?

**El issue:**
"Comercios Beauty CL como tú ganan $195K/mes desde MP" — el comercio comparado es:

- Mismo `country_code`
- Mismo `macro_niche`
- ¿Mismo plan? ¿Mismo tamaño (venues, clientes)?
- ¿Mismo tenure?

**Cuanto más fino el filtro, mejor el benchmark — pero menor el n.**

**Recomendación:** filtrar por `country × macro_niche × tiene_MP_activo`. No filtrar por tamaño/plan/tenure en V1; agregar en V2 si las cohortes quedan muy grandes.

**Pregunta a Finanzas/Analytics:** ¿les parece el corte? ¿qué umbral mínimo de n requieren (ej: 30 comercios) antes de mostrar el benchmark?

### 3.3 ¿Cadencia de actualización de los benchmarks?

Los benchmarks no necesitan ser en tiempo real. Pero tampoco pueden estar atrasados meses.

**Recomendación:** snapshot mensual generado por dbt el día 1 de cada mes. El usuario ve "promedio de Beauty CL en mayo 2026" — explícito.

**Pregunta:** ¿el equipo de data puede materializar una tabla `dwh_marketplace.benchmark_country_niche` con refresh mensual?

---

## 4. Riesgos legales / compliance a chequear


| Riesgo                                                                                      | Mitigación propuesta                                                                       | Validar con      |
| ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ---------------- |
| Decir "te genera $X" puede ser interpretado como ingreso garantizado                        | Disclaimers en hover/footer. "Histórico, no garantía"                                      | Legal            |
| Comparar al comercio con su segmento puede generar quejas ("¿de dónde sacan ese promedio?") | Mostrar el corte explícito ("Promedio comercios Beauty CL con MP activo, últimos 90 días") | Producto + Legal |
| Mostrar pérdida estimada ("dejas de ganar $136K") en el cancel flow                         | Aclarar "basado en tu promedio últimos 3 meses"                                            | Legal            |


---

## 5. Qué necesito que me devuelvan

**Por cada fila de las secciones 1.1, 1.2, 1.3 y 1.4:**

1. Tabla y columna oficial (o "no tenemos, definila tú")
2. Filtros obligatorios que tengo que aplicar (ej: "siempre excluir `status_id IN (4,5)`")
3. Marcar cada fila como ✅ validada / ⚠️ requiere ajuste / 🚫 no usable

**Para las 3 decisiones bloqueantes de la sección 3:**

- 3.1 Qué representa `price` exactamente (impuestos, copagos, descuentos) → confirmación o ajustes
- 3.2 Corte para benchmarks → si les sirve `country × macro_niche × tiene_MP_activo` + umbral mínimo de n
- 3.3 Materialización de tabla de benchmarks → si data puede armarla con refresh mensual

**Cuándo lo necesito:** idealmente esta semana para no atrasar el pasaje a desarrollo. Si necesitan más tiempo o quieren que aclare algo, avisen.

---

## Anexo A — Mapeo rápido "métrica mostrada → archivo del prototipo"

Para que ingeniería pueda trazar cada número al código:


| Métrica                                   | Archivo del prototipo                                                                   |
| ----------------------------------------- | --------------------------------------------------------------------------------------- |
| GMV Marketplace semanal                   | `src/components/sections/MarketplaceImpactCard.tsx` → `weekly(profile)`                 |
| Ratio 3.3× / margen libre                 | `src/components/sections/CancellationFlow.tsx` → `VariantStrong`                        |
| Lifetime stats (reservas/clientes/ventas) | `src/data/profiles.ts` → fields `totalBookings`, `totalNewClients`, `monthlyAvgRevenue` |
| Costo suscripción                         | `src/data/profiles.ts` → `monthlySubscriptionCost`                                      |
| Benchmarks                                | `src/data/profiles.ts` → `benchmarkAvgMarketplaceRevenue`, `benchmarkPctSelfPaying`     |


---

## Anexo B — Datos del DWH ya validados (mar–may 2026)

Como referencia de orden de magnitud, ya validamos en el DWH:

- **1.721 comercios B2B3+ en Chile con MP activo** (mar–may 2026)
- **Avg GMV MP mensual: $180K CLP** (mediana ~$80K)
- **Avg MRR: $76K CLP** (B2B3+)
- **Avg ratio MP/MRR: 3.0×** en Chile
- **Distribución del ratio Chile:** 38% <1× · 25% 1-2× · 24% 2-5× · 12% ≥5×
- **% comercios self-paying:** 62% CL · 55% AR · 35% CO · 35% MX
- **Beauty CL:** 70% self-paying (mejor segmento)
- **Health Doctors:** 47% self-paying (peor segmento)

Estos números son los que sustentan el copy. Si Finanzas dice "ojo, hay que recalcular con definición distinta", todo el copy cambia.

---

*Última actualización: 2026-06-18 · Sofía Aliste*
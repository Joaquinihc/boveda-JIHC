---
type: analisis
proyecto: "[[_Márgenes por Cliente y Producto]]"
estado: activo
fecha: 2026-11-19
temas: [margenes, pnl, payments, dwh]
metricas: []
---

# Llaves de Software, país y ejecución — hallazgos versión 1

**Pregunta**: ¿con qué llave conviene repartir el costo compartido de Software, hace falta separar por país en Payments y en soporte, y cuánto cuesta correr la cadena en dbt?

## Conclusión
> El costo compartido de Software se reparte mejor con reservas y sucursales que con sucursales solas: la mensajería sigue a las reservas y el soporte a una mezcla de ambas, con tope en el percentil 95 de reservas por sucursal. En Payments y en soporte no conviene separar por país: las diferencias entre países son de contabilización, no de costo. La cadena pesa poco en las corridas de dbt (1,2%); rediseñarla sirve para dejar una sola tabla, no para ahorrar minutos.

## Método
P&L Real ene–oct 2026 (`finance.profit_and_loss`, bloque 2- Service Costs), `finance.int_company_margin_drivers_monthly` (cadena en producción), `dwh.fct_bookings_created_monthly` (`bookings_count_demand`), `staging.stg_agendapro__bookings` (marcas `should_send_whatsapp` y `should_send_whatsapp_reminder`), `dwh.company_sales_months`, `dwh.location_sales_months` y `sys_query_history` de Redshift (corridas del 11 al 18-nov). Corte: 18 y 19-nov-2026. Simulación de diez combinaciones de llaves por cuenta con cuatro formas de contar reservas, de julio a octubre. Scripts, prototipo y consultas en `_analisis/2026-11-company-margin-v1/` (carpeta GitHub). Detalle y diagramas en las hojas de Notion de la tarea [[Análisis margen por cliente y por producto|T-056]]. Continúa [[20261118-01 Costos por stream — hallazgos versión 1]].

«Desajuste» = parte del costo que caería en un grupo de compañías distinto de donde está el consumo medido (grupos por segmento y tramo de reservas; promedio jul–oct 2026).

## Evidencia
**Software: con qué llave**
- Mensajería (G0008; el 99% es WhatsApp de Meta). Cada reserva marca si debe enviar WhatsApp: en oct-2026 son el 58% de las reservas, en 13.142 de 17.367 compañías. Desajuste: 41% por sucursales, 5% por reservas. El addon de WhatsApp no sirve de llave: cubre el 40% de esas reservas.
- Soporte, contra las conversaciones de Intercom con compañía identificada (54% del total). Desajuste: 26% por sucursales, 25% por reservas, 12% mitad y mitad.
- Nube y retención no tienen medida por compañía; su llave es un supuesto.
- El consumo por reserva baja cuando la compañía agenda más. WhatsApp por reserva: 0,77 bajo la mediana de reservas por sucursal, 0,55 entre p95 y p99, 0,41 sobre p99. Conversaciones de soporte por cada 10 mil reservas: 34,9, 7,4 y 3,6.
- Tope de reservas por sucursal, oct-2026. p99 (1.905): 153 compañías, 6% de las reservas fuera, desajuste 4% en mensajería y 14% en soporte. p95 (924): 761 compañías, 15% fuera, 5% y 12%. p90 (618): 1.523 compañías, 24% fuera, 9% y 12%. El tope por compañía que trae el código (p99) trata a una cadena como un solo local.
- El tope no traslada costo a las compañías sin reservas: lo que dejan de recibir las topadas (6,8k USD en octubre con p95) se reparte entre las demás que agendan.
- Importaciones: en jul-2026 una compañía cargó 224 mil reservas de su sistema anterior (4,5% del mes); `bookings_count_demand` las excluye. Carga masiva no marcada como importación: 66.820 reservas creadas en seis días para 217 clientes (oct-2026).
- Resultado en oct-2026 con lo decidido (mensajería por reservas; nube y soporte mitad y mitad; retención y resto por sucursales; tope p95): el 57% del costo compartido (104,6k USD) va por reservas; 1,37 centavos por reserva y 2,29 USD por sucursal; margen de Software B2B3 83%, B2B2 82%, B2C 73%; cerca de 1.100 compañías con margen negativo.
- Las 2.112 compañías sin reservas (12%) generan el 1,6% de las conversaciones de soporte y nada de mensajería; reciben 2,4 USD cada una, contra 5,5 con todo por sucursales.

**Payments: país**
- Factor por país, ene–oct 2026 (costo de procesamiento y comisiones bancarias del P&L ÷ costo medido, con el país de la cuenta): Chile 0,96, México 0,94, Argentina 0,93, Colombia 0,74; un solo factor, 0,95.
- Fuera de Chile el P&L de procesadores se contabiliza con el mismo costo medido del DWH: Argentina calza al dólar en nueve de diez meses (oct-2026: 10.610 contra 10.595 USD). Solo Chile tiene facturas propias; ahí el medido sobrestimó 12% en enero y 1% en octubre.
- Asientos que faltan: Argentina sin procesadores en mayo de 2026 (10,2k USD) y Colombia sin procesadores desde mayo (unos 0,75k al mes). En abril, 42.476 USD de Transbank quedaron con centro de costo HQ y país Chile.
- Contracargos: 50,6k USD de enero a octubre (Chile 25,5k, México 25,2k), el 3,8% de la llave. No aparecen en el P&L de Payment Processing. En México fueron 8,5k, 8,5k y 4,3k de junio a agosto.
- Con un solo factor, en el año cada país queda a menos de 3% de su P&L: Chile −0,8%, México +0,9%, Argentina +2,5%.
- Terminales POS (G0012) por país de la cuenta, oct-2026: Chile 9.504 USD entre 729 sucursales con POS (13,0); México 1.334 entre 281 (4,7). Parte del arriendo de Chile (5,4k en ene–oct) está con centro de costo HQ.

**Soporte, retención y facturación electrónica**
- La nómina de soporte de Argentina figura en el centro de costo Argentina solo de abril a septiembre (6,4k a 10,4k USD al mes de abril a julio) y en HQ el resto del año: es el mismo equipo con otra etiqueta. Es cerca del 38% de la nómina de soporte; los clientes argentinos son el 19% de las conversaciones identificadas.
- Repartir según el centro de costo habría bajado 7 puntos el margen de Software de Argentina en junio y 3 el de Colombia (retención, unos 4k al mes de mayo a julio).
- Facturación electrónica (G0007): DTEmite en Chile, unos 4,0k USD al mes (en HQ de enero a marzo); Dataico en Colombia, 0,6k a 1,8k; un cargo único de 276 USD en México en marzo.

**Ejecución en dbt**
- La cadena `company_margin_*` corre en producción desde la rama base (23 reglas) seis veces al día: unos 150 segundos por corrida (modelos 64, tests 87), el 1,2% del tiempo de consulta de dbt. El 58% es planificación del SQL: la lista de reglas se repite dentro de `company_margin_monthly` y de tres tests.
- Tablas en `finance`: `int_company_margin_drivers_monthly` (43 columnas), `company_margin_monthly` (38) y `company_margin_summary` (40). Único consumidor: el tablero Company Margin de Evidence, con cinco consultas.
- El PR #407 está 283 commits atrás de la base, en conflicto y con CI en rojo. Ver [[PR 407 — costos IA por vertical desde registro de API keys (reemplaza seed ai_costs_manual_monthly)]].
- Un prototipo de reparto en una pasada, sin tabla intermedia, cuadró con el P&L en los 22 meses y tardó 13,5 segundos.

## Caveats
- La marca de WhatsApp de una reserva indica intención de envío, no mensajes cobrados por Meta.
- El 46% de las conversaciones de soporte no tiene compañía asociada.
- El reparto de la nube mitad y mitad es un supuesto.
- El prototipo se midió una sola vez, como consulta suelta y sin las llaves de Software de la versión 1.
- Los tiempos de ejecución y el prototipo vienen de una auditoría hecha por un subagente; revisé las fechas de creación de las tablas, las corridas diarias y los tiempos por pieza.
- Las cifras por vertical se mueven si una compañía cambia de vertical: la dimensión es el valor actual.
- Corrección a la nota del 18-nov: la cadena en producción tiene 23 reglas; las 27 son del PR #407.

---
type: analisis
proyecto: "[[_Márgenes por Cliente y Producto]]"
estado: activo
fecha: 2026-11-18
temas: [margenes, pnl, ai, payments]
metricas: []
---

# Costos por stream — hallazgos versión 1

**Pregunta**: ¿qué costos del P&L corresponden a cada revenue stream y con qué dato se pueden repartir a cada compañía en una primera versión simple?

## Conclusión
> El modelo no hay que rehacerlo: la complejidad estaba en las reglas (27, con 11 llaves) y no en el motor. Con 4 a 6 llaves se llega a un reparto explicable que cuadra con el P&L. Los costos de pagos ya se miden por compañía y canal; AI solo se puede repartir por lo que paga cada compañía; en Software la llave elegida cambia mucho el margen de B2C.

## Método
P&L Real ene–oct 2026 (`finance.profit_and_loss`, bloque 2- Service Costs), `dwh.company_sales_months`, `dwh.location_sales_months`, `finance.int_company_margin_drivers_monthly` (versión en producción, PR #394), Odoo (AgendaPro Inc, sep y oct 2026) y la factura de AWS de octubre adjunta en Odoo. Corte: 18-nov-2026. Detalle y diagramas en las subpáginas de Notion de la tarea [[Análisis margen por cliente y por producto|T-056]].

## Evidencia
**AWS**
- La factura de AWS de oct-2026 es de 64.490 USD. Contabilidad la registra 50% en G0001 (Service Costs) y 50% en G2004 Product platforms («AWS R&D»). El P&L no tiene desglose por servicio ni por equipo.
- El PDF de la factura sí trae desglose por servicio (RDS 10.990, Savings Plans de cómputo 10.144, EC2 6.521, transferencia de datos 5.502, ElastiCache 1.260) y por cuenta: principal 49.566, AgendaPro Payments 1.437 y unas 31 cuentas «frust…» por cerca de 13.500 USD, sin identificar.
- El portal de costos de EKS de Infra (13.786 USD al mes) no incluye todo RDS ni Valkey: solo RDS y sus Savings Plans suman 14,1k en la factura.
- Azure dentro de G0001: 17,4k USD en jul, 6,2k en sep y 1,2k en oct-2026. No hay doble conteo con G0030–G0032: la reclasificación mensual descuenta de G0001 (−1.153 en sep, −1.113 en oct).

**AI**
- El costo de modelos de Health cayó de 2.859 USD (sep) a 365 (oct). El P&L calza con el registro de API keys; es optimización de consumo del equipo, no un error contable.
- Uso medido por compañía en oct-2026: Beauty sin datos (`cortex_conversations_daily` no se actualiza desde jun-2026), Health 75 de 107 compañías, Medspa 59 de 110.
- Margen de AI con modelos más plataforma: sep-2026 86% (Beauty 86%, Health 85%, Medspa 86%); oct-2026 90% (Beauty 85%, Health 95%, Medspa 90%).

**Payments**
- `company_sales_months` separa el costo por canal: en oct-2026, POS 100k USD y pagos en línea 47k; 902 compañías con POS, 2.854 con pagos en línea, 399 con ambos.
- P&L de procesadores (G0009, G0015, G0014) ÷ costo de la tabla sin contracargos: 0,88 a 0,99 por mes; 0,98 en octubre. La tabla no incluye terminales (86k en ene–oct) ni comisiones bancarias (41k).
- Terminales POS (G0012): crece parejo de 7,1k (ene) a 10,8k (oct) y da cerca de 12 USD por compañía con POS al mes; el 82% es Transbank. Se comporta como arriendo sobre la base, no como costo de alta. El 93% de las compañías con POS opera en una sola sucursal.
- El P&L de pagos tiene meses contabilizados en otro país o en HQ (Chile abr-2026, Argentina may-2026, Colombia may–oct).

**Software** (simulación oct-2026, 17.367 compañías)
- 2.112 compañías (12%) no crearon reservas en el mes y pagan 49k USD; 512 son B2B3.
- Margen por segmento según la llave del costo compartido: por sucursales B2B3 86% / B2B2 80% / B2C 62%; por reservas 81% / 85% / 86%; mixta (nube y mensajería por reservas, soporte y retención por sucursales) 83% / 83% / 76%.
- Compañías con margen negativo: 1.224 por sucursales, 301 por reservas, 1.099 mixta. Casi todas pagan menos de unos 3 USD al mes.
- El 1% con más reservas concentra el 22% de las reservas con el 8,5% del revenue; hay un caso de 66.905 reservas en un mes con una sucursal.

## Caveats
- Las simulaciones usan la tabla de llaves en producción, anterior al PR #407; los montos por compañía son aproximados.
- La plataforma de AI es una estimación de Infra de ago-2026, no una factura.
- En la comparación de llaves de terminales POS faltó la parte de México (1,3k USD) por un cruce de nombres de país.
- Los permisos de lectura del esquema `finance` no se pudieron listar.

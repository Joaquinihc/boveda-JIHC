---
type: reunion
fecha: 2026-11-17
titulo: "FP&A Weekly"
personas: ["[[Pablo Lucero]]"]
proyectos: ["[[_Márgenes por Cliente y Producto]]"]
temas: [margenes, definiciones, pnl, ai]
fuente: https://app.notion.com/p/41bab34c233b8106ae45e73ab4735db8
verificado: false
---

# FP&A Weekly

**Acuerdos** (máx 5)
- Verticales: se mantiene la definición existente (flujo original de macro nichos); no se reabre. Se elimina el flujo paralelo de Jesús Alcántara y Company Management queda solo de lectura. Los cambios de vertical se hacen con un PR en dbt, compañía por compañía, dejando fecha, quién y de qué a qué. Joaquín esperaba la confirmación de Vale López (le quita MRR) antes de sobreescribir los ids que pidió Sabri.
- Asignación de país: los merchants usan país geográfico y New Merchants usa la moneda de pago, por lo que muchos argentinos que pagan en dólares (App Store, Google Play) caen en «other». Propuesta de Joaquín: todo a país geográfico; primero se calcula el impacto (cambiaría hasta la tasa de conversión) y después se cambia.
- Margen por cliente, primera versión simple: a nivel de cliente, revenue de SaaS, Payments e IA más tres costos, que sumados cuadren con el P&L; pocas variables para no exponer costos sensibles. Payment processing (fees adicionales de Transbank, solo hay el total del P&L) se reparte en proporción a los costos de Company Sales Month; el resto se prorratea con supuestos (IA: por pagos y sucursales, o revenue unitario por sucursal; soporte todo a software por ahora).
- Expansión (ventas de IA, soporte y onboarding) queda fuera del alcance y se reclasifica después. Pablo pidió no reabrir decisiones tomadas y anotar cada una: cómo se clasifica hoy, qué se descartó y cuál sería la forma ideal sin datos.
- Se publica una primera versión en el esquema de Finance (Pablo la revisa) y se refina con datos de las verticales; Joaquín dejó en Drive (carpeta de Reservo) el registro de clientes de la alianza con Vambe y el saldo de la cuenta de Mercado Pago.

**Mis acciones**
- [ ] Cerrar el primer cálculo de margen por cliente: corregir los costos que se duplican al repartir entre IA y software y cuadrarlo con el P&L
- [ ] Publicar el cálculo en el esquema de Finance y, si sirve, pasarlo al esquema del DWH para uso público
- [ ] Calcular el impacto de llevar la asignación de país de New Merchants al país geográfico y volver con los resultados
- [ ] Enviarle a Pablo Lucero el tablero de prioridades para que lo revise
- [ ] Sobreescribir con un PR en dbt la vertical de las compañías que pidió Sabri, una vez que confirme Vale López

Sin dueño claro: enviar el Notion con la estructura del margen a los equipos verticales, indicando qué datos se necesitan de ellos para asignar costos; pedir al equipo de BizOps que cargue esos datos en Company Management, marcados como beta.

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/41bab34c233b8106ae45e73ab4735db8). La transcripción no nombra a los hablantes; hay un cuarto participante que no se identifica. Dice «DDT» (dbt), «BAMBE» (Vambe), «BISOPS» (BizOps), «piano» (P&L) y «cuenta G1» (cuenta que Joaquín no identificó). La última acción no está en el resumen de Notion: sale de la transcripción («Hay que sobreescribir entonces esos ID que te mandaron»). Vale López, Sabri y Jesús Alcántara no tienen ficha. Relacionadas: [[Análisis margen por cliente y por producto]], [[Clasificación de verticales por compañía]].

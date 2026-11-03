---
type: reunion
fecha: 2026-11-02
titulo: "Conciliación de clientes e ingresos de Reservo (Christopher)"
personas: []
proyectos: []
temas: [ventas, revenue, dwh]
fuente: https://app.notion.com/p/40cab34c233b808097abf68d8251c9ac
verificado: false
---

# Conciliación de clientes e ingresos de Reservo (Christopher)

**Acuerdos** (máx 5)
- Corte de octubre de Reservo: 4.119 sucursales pagando, tras excluir 114 que se estaban dando de baja. Los Merchants totales ya quedaron corregidos con el nuevo archivo de Christopher; faltan los New Merchants.
- New Merchants: las 215 altas por fecha de activación no cuadran con la referencia de ~170–180 al mes. Se cuentan por primer pago, abriendo cada pago a las sucursales de su cadena (un pago puede ser varias sucursales).
- Las tablas de pagos y de cambio de estado del cliente (la maneja postventa) no están vinculadas: el histórico de activos se reconstruye por descarte y «activo» no prueba que el cliente pague (reactivaciones para descargar fichas, cuentas de prueba activas). Christopher revisa las compañías nuevas sin pago menores de 90 días (planes anuales, datos perdidos, pruebas o uso sin pagar, que se cancelaría).
- Revenue: hoy solo cuenta pagos directos de suscripciones; faltan BAMBE (sus compañías se cuentan, su revenue no; factura externa) y las comisiones de Mercado Pago. Christopher consulta a Pablo dónde se registran y se arma una serie mensual completa.
- Christopher prioriza este mes ordenar los indicadores en el DW de Reservo, con un snapshot diario del estado de cada cliente (unas semanas para validar, después mensual con corte de fin de mes). Aparte: Juan Luis Cid entró a Reservo hace ~3 semanas; por eso no estaba en la nómina usada para los centros de costo.

**Mis acciones**
- [ ] Corregir los New Merchants de octubre cruzando el primer pago de cada compañía con las sucursales de su cadena
- [ ] Armar la serie mensual de revenue de Reservo con BAMBE y las comisiones de Mercado Pago

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/40cab34c233b808097abf68d8251c9ac). Título en Notion: «Reunión». Sin asistentes registrados; participan Christopher («Cris») y un Pablo que se suma por altavoz. Sigue en [[2026-11-02 New Merchants de Reservo — primer pago y los 173 de agosto (Pablo)]].

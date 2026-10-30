---
type: sugerencia
origen: slack
temas: [revenue, board, definiciones]
criterio: "(a) define cómo se mide revenue y merchants de Reservo para el grupo"
fuente: https://agendapro.slack.com/archives/D0CAK07JT4P/p1793193695165639
canal: "DM con Christopher Lagos (Reservo) + DM con Pablo Lucero"
fecha_hilo: 2026-10-28
creado: 2026-10-30
revisar: false
---

# Slack — Reservo en el Board Book, revenue base caja (IVA por país) y merchants por sucursal según cadena

- **Revenue** de Reservo = caja reconocida linealmente en el período que cubre cada pago (no MRR; la base no guarda historia de planes). Monto de `pagoclientemes`: Chile neto de IVA; Argentina y Colombia monto final pagado (sin IVA); México con IVA 16 % cuando se pide factura. Es el monto final con cupones y descuentos ya aplicados. Para el cierre del lunes 2-nov se usa como está (la corrida excluía 3 rubros, ~4 % de los pagos) — estado enviado a Pablo el 29-oct, que responde que lo que falta es el acceso a la data.
- **Merchants** (definición propuesta por Joaquín, 28-oct): se cuentan sucursales; los pagos múltiples son sucursales de una misma cadena (`id_cadena`). Cuenta como activa una sucursal en cero si su cadena sí pagó. New merchants = sucursales de la cadena en el mes de su primer pago. Base completa, sin excluir rubros. Hoy 3.366 cuentas que pagan; por sucursal debiera rondar ~4.100 (sep-26).
- 30-oct · Christopher entregó la corrida nueva (`id_cadena`, `estado_pago`, maestro de 9.451 centros, historial de estados y todos los rubros; cambia la apertura por rubro, no el total) y dice que actualizará lo que hay en el DWH.
- Pedido abierto a Reservo: guardar una foto mensual de clientes activos × mensualidad para tener MRR real en 2-3 meses. Board Book: [[PR 874 — AgendaPro + Reservo merchants and revenue (board book)]].

Hilo (preguntas y definiciones): https://agendapro.slack.com/archives/D0CAK07JT4P/p1793193695165639
Estado a Pablo: https://agendapro.slack.com/archives/D095G9DL97T/p1793296679406059

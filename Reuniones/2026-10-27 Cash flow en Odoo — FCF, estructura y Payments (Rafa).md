---
type: reunion
fecha: 2026-10-27
titulo: "Cash flow en Odoo — FCF, estructura y Payments (Rafa)"
personas: ["[[Nicolás Astudillo]]"]
proyectos: []
temas: [odoo, payments, definiciones]
fuente: https://app.notion.com/p/406ab34c233b81a3a345cdc051effbb4
verificado: false
---

# Cash flow en Odoo — FCF, estructura y Payments (Rafa)

**Acuerdos** (máx 5)
- Free Cash Flow: se confirma la práctica estándar de excluir levantamiento de capital y deuda (Loan, ESOP): solo Cash from Operations y Cash from Investing. Los intercompany se netean en el consolidado y se omiten en la vista por país. La estructura que venían usando Rafa y Joaquín iba bien.
- Estructura propuesta (hecha con Claude): arriba el bloque equivalente a Evidence y abajo un bloque ProPay/Payments (saldo inicial de caja propia, pasarelas y diferencias netas de cuentas ProPay) para llegar a un FCF total. Pendiente: si las pasarelas se muestran como saldo o como movimiento; el reporte debe dejar claro qué es saldo y qué es movimiento.
- Deuda neta ProPay = cuentas por pagar a comercios − cuentas por cobrar a pasarelas (transacciones del mes aún no liquidadas en el Santander); la por pagar debería ser siempre mayor. Hay que revisar si los ~USD 69k actuales están bien antes de validar el concepto.
- Odoo ya muestra saldo inicial y final de bancos y una sección Santander (por ahora fuera del FCF). En Chile las pasarelas están como cuentas por cobrar, no como banco, lo que complica incluirlas. dLocal cuadra (la diferencia son transferencias de los primeros días del mes). Dos devoluciones menores tomadas como ingreso se dejan pasar.
- Consolidado: P&L y balance ya están consolidados en Odoo; el cash flow no. Plan: cerrar el cash flow por país (moneda local, después USD) y luego consolidar con Claude o con [[Nicolás Astudillo]] (el único con el módulo). Rafa ve cómo reflejar en Odoo los gastos con tarjeta de crédito Santander que quedan en la cuenta ProPay. Objetivo: consolidado listo el viernes 30-oct.

**Mis acciones**
- [ ] Revisar los estados financieros de dLocal para validar la estructura del cash flow propuesta
- [ ] Coordinar con Rafa durante la semana el cierre del cash flow por país y luego el consolidado (objetivo: viernes 30-oct)

> Detalle completo: [Transcripción en Notion](https://app.notion.com/p/406ab34c233b81a3a345cdc051effbb4). La transcripción dice «UDU»; es Odoo. Sigue a [[2026-10-26 Cash flow en Odoo — diferencias con Evidence (Rafa y Pablo)]].
